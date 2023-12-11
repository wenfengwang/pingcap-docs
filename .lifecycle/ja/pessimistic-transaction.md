---
title: TiDB Pessimistic Transaction Mode
summary: TiDBの悲観的トランザクションモードについて学びます。
aliases: ['/docs/dev/pessimistic-transaction/','/docs/dev/reference/transactions/transaction-pessimistic/']
---

# TiDB Pessimistic Transaction Mode

TiDBを従来のデータベースに近づけ、移行コストを削減するために、v3.0から、TiDBは楽観的トランザクションモデルの上に悲観的トランザクションモードをサポートします。このドキュメントでは、TiDBの悲観的トランザクションモードの機能について説明します。

> **注意:**
>
> v3.0.8から、新しく作成されたTiDBクラスタはデフォルトで悲観的トランザクションモードを使用します。ただし、これは既存のクラスタに影響しません。それはv3.0.7またはそれ以前からv3.0.8またはそれ以降にアップグレードした場合です。つまり、**新しく作成されたクラスタのみがデフォルトで悲観的トランザクションモードを使用します**。

## トランザクションモードの切り替え

[`tidb_txn_mode`](/system-variables.md#tidb_txn_mode) システム変数を設定することで、トランザクションモードを設定できます。次のコマンドは、クラスタ内で新しく作成されたセッションによって実行されるすべての明示的なトランザクション（すなわち、自動コミットトランザクションではないトランザクション）を、悲観的トランザクションモードに設定します:

{{< copyable "sql" >}}

```sql
SET GLOBAL tidb_txn_mode = 'pessimistic';
```

また、次のSQL文を実行することで、明示的に悲観的トランザクションモードを有効にすることもできます:

{{< copyable "sql" >}}

```sql
BEGIN PESSIMISTIC;
```

{{< copyable "sql" >}}

```sql
BEGIN /*T! PESSIMISTIC */;
```

`BEGIN PESSIMISTIC;` および `BEGIN OPTIMISTIC;` 文は、`tidb_txn_mode` システム変数を無視し、悲観的および楽観的トランザクションモードの両方を使用することをサポートするために、これらの2つの文を使用して開始されたトランザクションは、システム変数を無視します。

## 動作

TiDBの悲観的トランザクションは、MySQLのものと似た動作をします。MySQL InnoDBとの細かい違いについては[Difference with MySQL InnoDB](#difference-with-mysql-innodb)を参照してください。

- 悲観的トランザクションでは、スナップショットリードとカレントリードが導入されています。

    - スナップショットリード: トランザクション開始前にコミットされたバージョンを読むロックされていないリードです。`SELECT` 文のリードはスナップショットリードです。
    - カレントリード: 最新のコミットされたバージョンを読むロックされたリードです。`UPDATE`、`DELETE`、`INSERT`、または `SELECT FOR UPDATE` 文のリードはカレントリードです。

    次の例では、スナップショットリードとカレントリードについて詳しく説明します。

    | セッション 1 | セッション 2 | セッション 3 |
    | :----| :---- | :---- |
    | CREATE TABLE t (a INT); |  |  |
    | INSERT INTO T VALUES(1); |  |  |
    | BEGIN PESSIMISTIC; |  |
    | UPDATE t SET a = a + 1; |  |
    |  | BEGIN PESSIMISTIC; |  |
    |  | SELECT * FROM t;  -- この時、スナップショットリードを使用して、現在のトランザクションが開始される前にコミットされたバージョンを読みます。結果は a=1 となります。 |  |
    |  |  | BEGIN PESSIMISTIC;
    |  |  | SELECT * FROM t FOR UPDATE; -- カレントリードを使用します。ロックを待ちます。
    | COMMIT; -- ロックを解放します。セッション 3 の SELECT FOR UPDATE 操作がロックを取得し、TiDBはカレントリードを使用して最新のコミットされたバージョンを読み込みます。結果は a=2 となります。 |  |  |
    |  | SELECT * FROM t; -- この時、スナップショットリードを使用して、現在のトランザクションが開始される前にコミットされたバージョンを読みます。結果は a=1 となります。 |  |

- `UPDATE`、`DELETE`、または `INSERT` 文を実行すると、 **最新** にコミットされたデータが読まれ、データが変更され、変更された行に悲観的ロックが適用されます。

- `SELECT FOR UPDATE` 文の場合、悲観的ロックがコミットされたデータの最新バージョンに適用されます。変更された行には適用されません。

- トランザクションがコミットされるかロールバックされるとロックが解放されます。データを変更しようとする他のトランザクションはブロックされ、ロックが解放されるのを待たなければなりません。データを読もうとするトランザクションはブロックされません。なぜなら、TiDBはマルチバージョン同時実行制御（MVCC）を使用しています。

- 一意制約のチェックで悲観的ロックをスキップするかどうかを制御するために、システム変数[`tidb_constraint_check_in_place_pessimistic`](/system-variables.md#tidb_constraint_check_in_place_pessimistic-new-in-v630)を設定できます。詳細は[constraints](/constraints.md#pessimistic-transactions)を参照してください。

- いくつかのトランザクションがお互いのロックを取得しようとすると、デッドロックが発生します。これは自動的に検出され、トランザクションの1つがランダムに終了し、MySQL互換のエラーコード `1213` が返されます。

- 新しいロックを取得するためにトランザクションが `innodb_lock_wait_timeout` 秒（デフォルト: 50）待ちます。このタイムアウトが達すると、MySQL互換のエラーコード `1205` が返されます。同じロックを待っている複数のトランザクションがある場合、優先順位はおおよそトランザクションの `start ts` に基づいています。

- TiDBは同じクラスタ内で楽観的トランザクションモードと悲観的トランザクションモードの両方をサポートしています。トランザクションの実行モードを指定することができます。

- TiDBは `FOR UPDATE NOWAIT` 構文をサポートし、ロックを待ちます。代わりに、MySQL互換のエラーコード `3572` が返されます。

- `Point Get` および `Batch Point Get` の演算子はデータを読まない場合でも、指定された主キーまたは一意キーをロックし、他のトランザクションが同じ主キーまたは一意キーに対してのロックや書き込みをブロックします。

- TiDBは `FOR UPDATE OF TABLES` 構文をサポートします。複数のテーブルを結合するステートメントの場合、TiDBは `OF TABLES` で関連付けられたテーブルの行にのみ悲観的ロックを適用します。

## MySQL InnoDB との違い

1. TiDBは、`WHERE` 句で範囲を使用したDMLまたは `SELECT FOR UPDATE` ステートメントを実行するとき、範囲内の並行したDMLステートメントはブロックされません。

    例:

    ```sql
    CREATE TABLE t1 (
     id INT NOT NULL PRIMARY KEY,
     pad1 VARCHAR(100)
    );
    INSERT INTO t1 (id) VALUES (1),(5),(10);
    ```

    ```sql
    BEGIN /*T! PESSIMISTIC */;
    SELECT * FROM t1 WHERE id BETWEEN 1 AND 10 FOR UPDATE;
    ```

    ```sql
    BEGIN /*T! PESSIMISTIC */;
    INSERT INTO t1 (id) VALUES (6); -- MySQLではブロックされます
    UPDATE t1 SET pad1='new value' WHERE id = 5; -- MySQLとTiDBの両方で待機します
    ```

    この動作は、TiDBは現在ギャップロックをサポートしていないためです。

2. TiDBは `SELECT LOCK IN SHARE MODE` をサポートしていません。

    `SELECT LOCK IN SHARE MODE` を実行すると、ロックがない状態と同じ効果があります。そのため、他のトランザクションの読み込みや書き込み操作はブロックされません。

3. DDLは悲観的トランザクションのコミット失敗を引き起こす場合があります。

    MySQLでDDLが実行されると、実行中のトランザクションによってブロックされる可能性があります。しかし、このシナリオでは、TiDBではDDL操作がブロックされず、悲観的トランザクションのコミットが失敗する可能性があります: `ERROR 1105 (HY000): Information schema is changed. [try again later]`。TiDBはトランザクションの実行中に `TRUNCATE TABLE` ステートメントを実行することがあり、これにより `table doesn't exist` エラーが発生する可能性があります。

4. `START TRANSACTION WITH CONSISTENT SNAPSHOT` を実行した後、MySQLでは他のトランザクションで後で作成された表を読むことができますが、TiDBではできません。

5. オートコミットトランザクションは、楽観的ロックを優先します。

    悲観的モデルを使用すると、オートコミットトランザクションはまず、オーバーヘッドの少ない楽観的モデルを使用してステートメントをコミットしようとします。書き込みの競合が発生した場合、悲観的モデルを使用してトランザクションを再試行します。したがって、`tidb_retry_limit` が `0` に設定されている場合、オートコミットトランザクションは、書き込みの競合が発生した場合にも 「Write Conflict」 エラーを報告します。

    オートコミット `SELECT FOR UPDATE` ステートメントはロックを待ちません。

6. ステートメント内の `EMBEDDED SELECT` によって読み取られたデータはロックされません。

7. TiDBのオープンなトランザクションはガベージコレクション（GC）をブロックしません。デフォルトでは、悲観的トランザクションの最大実行時間は1時間に制限されています。これは、TiDB設定ファイルの `[performance]` の `max-txn-ttl` を編集することで変更できます。

## 分離レベル

TiDBは、悲観的トランザクションモードでデフォルトでMySQLと同じく [Repeatable Read](/transaction-isolation-levels.md#repeatable-read-isolation-level) をサポートします。

> **注意:**
>
> この分離レベルでは、DML操作は最新のコミットされたデータに基づいて実行されます。これはMySQLと同じですが、TiDBの楽観的トランザクションモードとは異なります。[Difference between TiDB and MySQL Repeatable Read](/transaction-isolation-levels.md#difference-between-tidb-and-mysql-repeatable-read)を参照してください。
- [Read Committed](/transaction-isolation-levels.md#read-committed-isolation-level). You can set this isolation level using the [`SET TRANSACTION`](/sql-statements/sql-statement-set-transaction.md) statement.

## 悲観的なトランザクションのコミットプロセス

トランザクションのコミットプロセスでは、悲観的なトランザクションと楽観的なトランザクションは同じロジックを持っています。両方のトランザクションは2段階コミット（2PC）モードを採用しています。悲観的なトランザクションの重要な適応は DML の実行です。

![TiDBの悲観的なトランザクションのコミットプロセス](/media/pessimistic-transaction-commit.png)

悲観的なトランザクションは2PCの前に `悲観的ロックを取得` フェーズを追加します。このフェーズには次の手順が含まれます。

1. （楽観的なトランザクションモードと同じ）TiDB はクライアントから `begin` リクエストを受信し、現在のタイムスタンプがこのトランザクションの start_ts です。
2. TiDB サーバーがクライアントからの書き込みリクエストを受信すると、TiDB サーバーは TiKV サーバーに悲観的ロックリクエストを開始し、そのロックは TiKV サーバーに永続化されます。
3. （楽観的なトランザクションモードと同じ）クライアントがコミットリクエストを送信すると、TiDB は楽観的なトランザクションモードと同様に2段階コミットを実行し始めます。

![TiDBの悲観的トランザクション](/media/pessimistic-transaction-in-tidb.png)

## パイプラインロック処理

悲観的なロックを追加するには、TiKV にデータを書き込む必要があります。ロックを正常に追加する応答は、コミットおよび Raft を介した適用後に TiDB に返されます。したがって、楽観的なトランザクションと比較して、悲観的なトランザクションモードは必然的により高いレイテンシーを持ちます。

Locking のオーバーヘッドを減らすために、TiKV はパイプラインロック処理を実装しています：データがロックの要件を満たした場合、TiKV はすぐに TiDB に後続のリクエストを実行し、悲観的ロックに非同期で書き込みます。このプロセスにより、大部分のレイテンシが削減され、悲観的なトランザクションのパフォーマンスが大幅に向上します。ただし、TiKV でネットワーク分断が発生した場合や、TiKV ノードがダウンした場合、悲観的なロックへの非同期書き込みが失敗し、以下の側面に影響を与える可能性があります。

* 同じデータを変更する他のトランザクションはブロックされません。アプリケーションロジックがロックまたはロック待機メカニズムに依存する場合、アプリケーションロジックの正確性が影響を受けます。

* トランザクションのコミットが失敗する可能性は低いが、トランザクションの正確性には影響しません。

<CustomContent platform="tidb">

アプリケーションロジックがロック取得またはロック待機メカニズムに依存している場合、または TiKV クラスタに異常が発生してもトランザクションのコミットが成功する可能性をできるだけ保証したい場合は、パイプラインロック機能を無効にする必要があります。

![パイプライン悲観的ロック](/media/pessimistic-transaction-pipelining.png)

この機能はデフォルトで有効です。これを無効にするには、TiKV の構成を変更します。

```toml
[pessimistic-txn]
pipelined = false
```

TiKV クラスタが v4.0.9 以降の場合、[TiKV 構成を動的に変更](/dynamic-config.md#modify-tikv-configuration-dynamically)してこの機能を無効にすることもできます。

{{< copyable "sql" >}}

```sql
set config tikv pessimistic-txn.pipelined='false';
```

</CustomContent>

<CustomContent platform="tidb-cloud">

アプリケーションロジックがロック取得またはロック待機メカニズムに依存している場合、または TiKV クラスタに異常が発生してもトランザクションのコミットが成功する可能性をできるだけ保証したい場合は、[TiDB Cloud サポート](/tidb-cloud/tidb-cloud-support.md)に連絡して、パイプラインロック機能を無効にできます。

</CustomContent>

## インメモリ悲観的ロック

v6.0.0 で、TiKV はインメモリ悲観的ロックの機能を導入しました。この機能が有効になっているとき、悲観的ロックは通常、リージョンリーダーのメモリにのみ保存され、ディスクに永続化されることなく Raft を介した他のレプリカに複製されません。この機能により、悲観的ロックの取得のオーバーヘッドが大幅に削減され、悲観的なトランザクションのスループットが向上します。

インメモリ悲観的ロックのメモリ使用量がリージョンや TiKV ノードのメモリしきい値を超えると、悲観的ロックの取得に [パイプラインロック処理](#pipelined-locking-process) に切り替わります。リージョンがマージされたり、リーダーが転送されたりすると、悲観的ロックを失わないために、TiKV はインメモリ悲観的ロックをディスクに書き込み、他のレプリカに複製します。

インメモリ悲観的ロックは、クラスタが正常な状態であるときにロックの取得に影響を与えることはありません。ただし、TiKV でネットワーク分離が発生した場合や、TiKV ノードがダウンした場合、取得した悲観的ロックは失われる可能性があります。

アプリケーションロジックがロック取得またはロック待機メカニズムに依存している場合、またはクラスタが異常な状態であってもトランザクションのコミットの成功率をできるだけ保証したい場合は、**インメモリ悲観的ロック** 機能を **無効に** する必要があります。

この機能はデフォルトで有効です。これを無効にするには、TiKV の構成を変更します。

```toml
[pessimistic-txn]
in-memory = false
```

この機能を動的に無効にするには、TiKV の構成を動的に変更します。

{{< copyable "sql" >}}

```sql
set config tikv pessimistic-txn.in-memory='false';
```