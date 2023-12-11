---
title: トランザクション
summary: TiDB でトランザクションを学ぶ。
aliases: ['/docs/dev/transaction-overview/','/docs/dev/reference/transactions/overview/']
---

# トランザクション

TiDB は、[悲観的](/pessimistic-transaction.md) または [楽観的](/optimistic-transaction.md) トランザクションモードを使用して分散トランザクションをサポートしています。TiDB 3.0.8 から、TiDB はデフォルトで悲観的トランザクションモードを使用します。

このドキュメントでは、一般的に使用されるトランザクション関連のステートメント、明示的および暗黙的トランザクション、分離レベル、遅延チェック制約、およびトランザクションサイズを紹介します。

一般的な変数には、[`autocommit`](#autocommit)、[`tidb_disable_txn_auto_retry`](/system-variables.md#tidb_disable_txn_auto_retry)、[`tidb_retry_limit`](/system-variables.md#tidb_retry_limit)、および[`tidb_txn_mode`](/system-variables.md#tidb_txn_mode)が含まれます。

> **注記:**
>
> [`tidb_disable_txn_auto_retry`](/system-variables.md#tidb_disable_txn_auto_retry) および [`tidb_retry_limit`](/system-variables.md#tidb_retry_limit) 変数は楽観的トランザクションにのみ適用され、悲観的トランザクションには適用されません。

## 一般的なステートメント

### トランザクションの開始

ステートメント [`BEGIN`](/sql-statements/sql-statement-begin.md) および [`START TRANSACTION`](/sql-statements/sql-statement-start-transaction.md) は、新しいトランザクションを明示的に開始するために相互に使用できます。

構文:

{{< copyable "sql" >}}

```sql
BEGIN;
```

{{< copyable "sql" >}}

```sql
START TRANSACTION;
```

{{< copyable "sql" >}}

```sql
START TRANSACTION WITH CONSISTENT SNAPSHOT;
```

{{< copyable "sql" >}}

```sql
START TRANSACTION WITH CAUSAL CONSISTENCY ONLY;
```

これらのステートメントのいずれかを実行すると、現在のセッションがトランザクションの途中にある場合、TiDB は新しいトランザクションを開始する前に自動的に現在のトランザクションをコミットします。

> **注記:**
>
> MySQL とは異なり、TiDB は、上記のステートメントを実行した後、現在のデータベースのスナップショットを取得します。 MySQL の `BEGIN` および `START TRANSACTION` は、トランザクションが開始された後に InnoDB からデータを読み取る最初の `SELECT` ステートメント（`SELECT FOR UPDATE` ではない）の実行後にスナップショットを取得します。 `START TRANSACTION WITH CONSISTENT SNAPSHOT` は、ステートメントの実行中にスナップショットを取得します。その結果、`BEGIN`、`START TRANSACTION`、および `START TRANSACTION WITH CONSISTENT SNAPSHOT` は MySQL の `START TRANSACTION WITH CONSISTENT SNAPSHOT` と等価です。

### トランザクションのコミット

ステートメント [`COMMIT`](/sql-statements/sql-statement-commit.md) は、現在のトランザクションで行われたすべての変更を適用するよう TiDB に指示します。

構文:

{{< copyable "sql" >}}

```sql
COMMIT;
```

> **ヒント:**
>
> [楽観的トランザクション](/optimistic-transaction.md) を有効にする前に、`COMMIT` ステートメントがエラーを返す可能性があることを正しく処理するために、アプリケーションが正しく処理されることを確認してください。これについての自信がない場合は、[悲観的トランザクション](/pessimistic-transaction.md) のデフォルトを使用することをお勧めします。

### トランザクションのロールバック

ステートメント [`ROLLBACK`](/sql-statements/sql-statement-rollback.md) は、現在のトランザクションで行われたすべての変更を取り消します。

構文:

{{< copyable "sql" >}}

```sql
ROLLBACK;
```

クライアント接続が中断または閉じられた場合、トランザクションも自動的にロールバックされます。

## 自動コミット

MySQL 互換性のため、TiDB はデフォルトでステートメントを即座に _自動コミット_ します。

例:

```sql
mysql> CREATE TABLE t1 (
     id INT NOT NULL PRIMARY KEY auto_increment,
     pad1 VARCHAR(100)
    );
Query OK, 0 rows affected (0.09 sec)

mysql> SELECT @@autocommit;
+--------------+
| @@autocommit |
+--------------+
| 1            |
+--------------+
1 row in set (0.00 sec)

mysql> INSERT INTO t1 VALUES (1, 'test');
Query OK, 1 row affected (0.02 sec)

mysql> ROLLBACK;
Query OK, 0 rows affected (0.01 sec)

mysql> SELECT * FROM t1;
+----+------+
| id | pad1 |
+----+------+
|  1 | test |
+----+------+
1 row in set (0.00 sec)
```

上記の例では、`ROLLBACK` ステートメントは効果がありません。これは、`INSERT` ステートメントが自動コミットで実行されたためです。つまり、これは次の単一ステートメントトランザクションと同等でした。

```sql
START TRANSACTION;
INSERT INTO t1 VALUES (1, 'test');
COMMIT;
```

トランザクションが明示的に開始されている場合は自動コミットは適用されません。次の例では、`ROLLBACK` ステートメントは `INSERT` ステートメントを正常に元に戻します。

```sql
mysql> CREATE TABLE t2 (
     id INT NOT NULL PRIMARY KEY auto_increment,
     pad1 VARCHAR(100)
    );
Query OK, 0 rows affected (0.10 sec)

mysql> SELECT @@autocommit;
+--------------+
| @@autocommit |
+--------------+
| 1            |
+--------------+
1 row in set (0.00 sec)

mysql> START TRANSACTION;
Query OK, 0 rows affected (0.00 sec)

mysql> INSERT INTO t2 VALUES (1, 'test');
Query OK, 1 row affected (0.02 sec)

mysql> ROLLBACK;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT * FROM t2;
Empty set (0.00 sec)
```

[`autocommit`](/system-variables.md#autocommit) システム変数は、グローバルまたはセッションベースで [変更できます](/sql-statements/sql-statement-set-variable.md)。

例:

{{< copyable "sql" >}}

```sql
SET autocommit = 0;
```

{{< copyable "sql" >}}

```sql
SET GLOBAL autocommit = 0;
```

## 明示的および暗黙的トランザクション

> **注記:**
>
> 一部のステートメントは暗黙的にコミットされます。たとえば、`[BEGIN|START TRANSACTION]` を実行すると、最後のトランザクションが暗黙的にコミットされ、新しいトランザクションが開始されます。この動作は MySQL 互換性のために必要です。詳細については、[implicit commit](https://dev.mysql.com/doc/refman/8.0/en/implicit-commit.html) を参照してください。

TiDB は、明示的トランザクション（ `[BEGIN|START TRANSACTION]` および `COMMIT` を使用してトランザクションの開始と終了を定義する）と暗黙的トランザクション（`SET autocommit = 1`）をサポートしています。

`autocommit` の値を `1` に設定し、 `[BEGIN|START TRANSACTION]` ステートメントを使用して新しいトランザクションを開始した場合、トランザクションは明示的になります。

DDL ステートメントの場合、トランザクションは自動的にコミットされ、ロールバックはサポートされません。現在のセッションでトランザクションが進行中の状態で DDL ステートメントを実行すると、DDL ステートメントは現在のトランザクションがコミットされた後に実行されます。

## 制約の遅延チェック

デフォルトでは、楽観的トランザクションは、DML ステートメントの実行時に [主キー](/constraints.md#primary-key) または [一意制約](/constraints.md#unique-key) をチェックしません。これらのチェックは、トランザクション `COMMIT` の際に実行されます。

例:

{{< copyable "sql" >}}

```sql
CREATE TABLE t1 (id INT NOT NULL PRIMARY KEY);
INSERT INTO t1 VALUES (1);
BEGIN OPTIMISTIC;
INSERT INTO t1 VALUES (1); -- MySQL ではエラーが返されますが、TiDB は成功を返します。
INSERT INTO t1 VALUES (2);
COMMIT; -- これは MySQL では正常にコミットされますが、TiDB はエラーを返し、トランザクションをロールバックします。
SELECT * FROM t1; -- MySQL では 1 2 を返しますが、TiDB は 1 を返します。
```

```sql
mysql> CREATE TABLE t1 (id INT NOT NULL PRIMARY KEY);
Query OK, 0 rows affected (0.10 sec)

mysql> INSERT INTO t1 VALUES (1);
Query OK, 1 row affected (0.02 sec)

mysql> BEGIN OPTIMISTIC;
Query OK, 0 rows affected (0.00 sec)

mysql> INSERT INTO t1 VALUES (1); -- MySQL ではエラーが返されますが、TiDB は成功を返します。
Query OK, 1 row affected (0.00 sec)

mysql> INSERT INTO t1 VALUES (2);
Query OK, 1 row affected (0.00 sec)

mysql> COMMIT; -- これは MySQL では正常にコミットされますが、TiDB はエラーを返し、トランザクションをロールバックします。
ERROR 1062 (23000): Duplicate entry '1' for key 't1.PRIMARY'
mysql> SELECT * FROM t1; -- MySQL では 1 2 を返しますが、TiDB は 1 を返します。
+----+
| id |
+----+
|  1 |
+----+
1 row in set (0.01 sec)
```

遅延チェックの最適化は、制約チェックをバッチ処理してネットワーク通信を削減することによりパフォーマンスを向上させます。この動作は、[`tidb_constraint_check_in_place=ON`](/system-variables.md#tidb_constraint_check_in_place) を設定することで無効にできます。

> **注記:**
>
> + この最適化は楽観的トランザクションにのみ適用されます。
> + この最適化は、`INSERT IGNORE` および `INSERT ON DUPLICATE KEY UPDATE` には影響しませんが、通常の `INSERT` 文にのみ影響します。

## ステートメントのロールバック

TiDBは、ステートメントの実行失敗後にアトミックなロールバックをサポートしています。ステートメントにエラーが発生した場合、その変更は反映されません。トランザクションはオープンのままであり、`COMMIT` 文または `ROLLBACK` 文を発行する前に追加の変更を行うことができます。

{{< copyable "sql" >}}

```sql
CREATE TABLE test (id INT NOT NULL PRIMARY KEY);
BEGIN;
INSERT INTO test VALUES (1);
INSERT INTO tset VALUES (2);  -- "test" が "tset" と誤って入力されたため、このステートメントは影響しません。
INSERT INTO test VALUES (1),(2);  -- このステートメント全体が PRIMARY KEY 制約に違反するため、影響しません。
INSERT INTO test VALUES (3);
COMMIT;
SELECT * FROM test;
```

```sql
mysql> CREATE TABLE test (id INT NOT NULL PRIMARY KEY);
Query OK, 0 rows affected (0.09 sec)

mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)

mysql> INSERT INTO test VALUES (1);
Query OK, 1 row affected (0.02 sec)

mysql> INSERT INTO tset VALUES (2);  -- "test" が "tset" と誤って入力されたため、このステートメントは影響しません。
ERROR 1146 (42S02): Table 'test.tset' doesn't exist
mysql> INSERT INTO test VALUES (1),(2);  -- このステートメント全体が PRIMARY KEY 制約に違反するため、影響しません。
ERROR 1062 (23000): Duplicate entry '1' for key 'test.PRIMARY'
mysql> INSERT INTO test VALUES (3);
Query OK, 1 row affected (0.00 sec)

mysql> COMMIT;
Query OK, 0 rows affected (0.01 sec)

mysql> SELECT * FROM test;
+----+
| id |
+----+
|  1 |
|  3 |
+----+
2 rows in set (0.00 sec)
```

上記の例では、失敗した `INSERT` ステートメントの後、トランザクションはオープンのままとなります。その後、最終的な挿入ステートメントは成功し、変更がコミットされます。

## トランザクションサイズ制限

TiDBは、基礎ストレージエンジンの制約により、1つのローが6 MBを超えないように制限されています。1つのローのすべての列は、データ型に応じてバイトに変換され、1つのローのサイズを推定するために合計されます。

TiDBは楽観的トランザクションと悲観的トランザクションの両方をサポートしており、楽観的トランザクションは悲観的トランザクションの基盤となっています。楽観的トランザクションはまずプライベートメモリに変更をキャッシュするため、TiDBは単一トランザクションのサイズを制限しています。

デフォルトでは、TiDBは単一トランザクションの総サイズを100 MBを超えないように設定しています。このデフォルト値は構成ファイルの `txn-total-size-limit` を介して変更することができます。`txn-total-size-limit` の最大値は1 TBです。個々のトランザクションサイズ制限は、サーバで利用可能な残りのメモリのサイズに依存します。これは、トランザクションが実行されると、TiDBプロセスのメモリ使用量が、トランザクションサイズの2倍から3倍以上、それ以上に拡大するためです。

TiDBは以前、単一トランザクションのキーバリューペアの総数を30万に制限していました。この制限はTiDB v4.0で削除されました。

> **注意:**
>
> 通常、TiDB バイナリログはダウンストリームへのデータレプリケーションを有効にするために使用されます。一部のシナリオでは、Kafkaなどのメッセージミドルウェアが、ダウンストリームにレプリケートされたバイナログを消費するために使用されます。
>
> Kafkaを例として挙げると、Kafkaの単一メッセージ処理能力の上限は1 GBです。したがって、`txn-total-size-limit` を1 GBより大きな値に設定した場合、TiDBでトランザクションが正常に実行されても、ダウンストリームのKafkaでエラーが報告される可能性があります。この状況を避けるためには、`txn-total-size-limit` の実際の値を、最終消費者の制限に応じて決定する必要があります。たとえば、Kafkaがダウンストリームで使用される場合、`txn-total-size-limit` は1 GBを超えてはなりません。

## 因果的整合性

> **注意:**
>
> 因果的整合性を持つトランザクションは、非同期コミットとワンフェーズコミットの機能が有効になっている場合にのみ有効です。これらの機能の詳細については、[`tidb_enable_async_commit`](/system-variables.md#tidb_enable_async_commit-new-in-v50) および [`tidb_enable_1pc`](/system-variables.md#tidb_enable_1pc-new-in-v50) を参照してください。

TiDBは、トランザクションに対して因果的整合性を有効にすることをサポートしています。因果的整合性を持つトランザクションは、コミット時にPDからタイムスタンプを取得する必要がなく、コミットの待ち時間が短くなります。因果的整合性を有効にする構文は次のようになります:

{{< copyable "sql" >}}

```sql
START TRANSACTION WITH CAUSAL CONSISTENCY ONLY;
```

デフォルトでは、TiDBはリニア整合性を保証しています。リニア整合性の場合、トランザクション1がコミットされた後にトランザクション2がコミットされると、論理的にはトランザクション2はトランザクション1の後に発生するはずです。因果的整合性はリニア整合性よりも弱いです。因果的整合性の場合、データベースによって認識されるトランザクション1およびトランザクション2によってロックが設定されたデータまたは書き込まれたデータに因果関係がある場合のみ、2つのトランザクションのコミット順序と発生順序が一貫していることが保証されます。現在のところ、TiDBは外部因果関係を渡すことをサポートしていません。

因果的整合性が有効になっている2つのトランザクションには、次の特性があります:

+ [潜在的な因果関係を持つトランザクションは一貫した論理的順序と物理的コミット順序を持つ](#potential-causal-relationship)
+ [因果関係を持たないトランザクションは一貫した論理的順序と物理的コミット順序を保証しない](#no-causal-relationship)
+ [ロックなしの読み取りは因果関係を作成しません](#reads-without-lock)

### 潜在的な因果関係を持つトランザクションは一貫した論理的順序と物理的コミット順序を持つ

トランザクション1とトランザクション2が因果的整合性を採用し、次のステートメントを実行したと仮定します:

| トランザクション 1 | トランザクション 2 |
|-------|-------|
| START TRANSACTION WITH CAUSAL CONSISTENCY ONLY | START TRANSACTION WITH CAUSAL CONSISTENCY ONLY |
| x = SELECT v FROM t WHERE id = 1 FOR UPDATE | |
| UPDATE t set v = $(x + 1) WHERE id = 2 | |
| COMMIT | |
| | UPDATE t SET v = 2 WHERE id = 1 |
| | COMMIT |

上記の例では、トランザクション1は `id = 1` レコードをロックし、トランザクション2が `id = 1` レコードを変更しています。したがって、トランザクション1とトランザクション2には潜在的な因果関係があります。因果的整合性が有効になっていても、トランザクション2がトランザクション1の後に正常にコミットされた場合、論理的にはトランザクション2はトランザクション1の後に発生するはずです。そのため、トランザクション1が`id = 2`のレコードの変更を読み取らずにトランザクション2の変更を読み取ることはありません。

### 因果関係を持たないトランザクションは一貫した論理的順序と物理的コミット順序を保証しない

`id = 1` および `id = 2`の初期値がどちらも `0`と仮定します。トランザクション1とトランザクション2が因果的整合性を採用し、次のステートメントを実行したと仮定します:

| トランザクション 1 | トランザクション 2 | トランザクション 3 |
|-------|-------|-------|
| START TRANSACTION WITH CAUSAL CONSISTENCY ONLY | START TRANSACTION WITH CAUSAL CONSISTENCY ONLY | |
| UPDATE t set v = 3 WHERE id = 2 | | |
| | UPDATE t SET v = 2 WHERE id = 1 | |
| | | BEGIN |
| COMMIT | | |
| | COMMIT | |
| | | SELECT v FROM t WHERE id IN (1, 2) |

上記の例では、トランザクション1は `id = 1` レコードを読み取らないため、トランザクション1とトランザクション2にはデータベースが認識する因果関係はありません。これらのトランザクションに因果的整合性が有効になっている場合、物理時間順でトランザクション1がコミットされた後にトランザクション2がコミットされても、TiDBはトランザクション2が論理的にトランザクション1の後に発生することを保証しません。

もしトランザクション3がトランザクション1がコミットされる前に開始され、トランザクション2がコミットされた後に `id = 1` および `id = 2` のレコードを読み取る場合、トランザクション3は `id = 1` の値を `2`、`id = 2` の値を `0` と読み取る可能性があります。

### ロックなしの読み取りは因果関係を作成しません

トランザクション1とトランザクション2が因果的整合性を採用し、次のステートメントを実行したと仮定します:

| トランザクション 1 | トランザクション 2 |
|-------|-------|
| START TRANSACTION WITH CAUSAL CONSISTENCY ONLY | START TRANSACTION WITH CAUSAL CONSISTENCY ONLY |
| | UPDATE t SET v = 2 WHERE id = 1 |
| SELECT v FROM t WHERE id = 1 | |
| UPDATE t set v = 3 WHERE id = 2 | |
| | COMMIT |
| COMMIT | |

上記の例では、ロックなしの読み取りは因果関係を作成しません。トランザクション1とトランザクション2は書き込みのスキューを作成しています。この場合、2つのトランザクションがまだ因果的な関係を持っているというのは筋違いです。したがって、因果的整合性が有効になっているこれらの2つのトランザクションには明確な論理的順序がありません。