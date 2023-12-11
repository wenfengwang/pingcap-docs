---
title: TiDB トランザクション分離レベル
summary: TiDBのトランザクション分離レベルについて学ぶ。
aliases: ['/docs/dev/transaction-isolation-levels/','/docs/dev/reference/transactions/transaction-isolation/']
---

# TiDB トランザクション分離レベル

<CustomContent platform="tidb">

トランザクション分離は、データベーストランザクション処理の基礎の一つです。分離はトランザクションの4つの主要な特性の一つです（一般的に[ACID](/glossary.md#acid)と呼ばれます）。

</CustomContent>

<CustomContent platform="tidb-cloud">

トランザクション分離は、データベーストランザクション処理の基礎の一つです。分離はトランザクションの4つの主要な特性の一つです（一般的に[tidb-cloud-glossary.md#acid](/tidb-cloud/tidb-cloud-glossary.md#acid)と呼ばれます）。

</CustomContent>

SQL-92標準では、4つのトランザクション分離レベルが定義されています：Read Uncommitted、Read Committed、Repeatable Read、および Serializable。詳細については、以下の表を参照してください。

| 分離レベル        | Dirty Write   | Dirty Read | Fuzzy Read     | Phantom |
| :----------- | :------------ | :------------- | :----------| :-------- |
| READ UNCOMMITTED | 不可能 | 可能     | 可能     | 可能     |
| READ COMMITTED   | 不可能 | 不可能 | 可能     | 可能     |
| REPEATABLE READ  | 不可能 | 不可能 | 不可能 | 可能     |
| SERIALIZABLE     | 不可能 | 不可能 | 不可能 | 不可能 |

TiDBでは、Snapshot Isolation（SI）の整合性を実装しており、これをMySQLとの互換性のために `REPEATABLE-READ` として宣伝しています。これは[ANSI Repeatable Read分離レベル](#difference-between-tidb-and-ansi-repeatable-read)および[MySQL Repeatable Readレベル](#difference-between-tidb-and-mysql-repeatable-read)とは異なります。

> **注意:**
>
> TiDB v3.0から、トランザクションの自動リトライはデフォルトで無効になっています。自動リトライを有効にすることは**トランザクションの分離レベルを破損する可能性**があるため、推奨されません。詳細については、[Transaction Retry](/optimistic-transaction.md#automatic-retry)を参照してください。
>
> TiDB v3.0.8から、新しく作成されたTiDBクラスターはデフォルトで[pessimistic transaction mode](/pessimistic-transaction.md)を使用します。現在の読み取り（`for update` read）は**non-repeatable read**です。詳細については、[pessimistic transaction mode](/pessimistic-transaction.md)を参照してください。

## Repeatable Read分離レベル

Repeatable Read分離レベルは、トランザクション開始前にコミットされたデータのみを参照し、トランザクション実行中に並行トランザクションによってコミットされたデータや未コミットのデータを一切参照しません。ただし、トランザクション文は、まだコミットされていないデータに影響を受けます。

異なるノードで実行されるトランザクションの開始およびコミットの順序は、PDから取得したタイムスタンプの順序に依存します。

Repeatable Read分離レベルのトランザクションは、同じ行を同時に更新することはできません。コミット時、トランザクションは始動後に他のトランザクションによって行が更新されたことを検知した場合、トランザクションはロールバックします。例：

```sql
create table t1(id int);
insert into t1 values(0);

start transaction;              |               start transaction;
select * from t1;               |               select * from t1;
update t1 set id=id+1;          |               update t1 set id=id+1; -- 悲観的トランザクションでは、`update`文が後に実行されても、行ロックが解放されるまで待機します。
commit;                         |
                                |               commit; -- トランザクションのコミットが失敗し、ロールバックされます。悲観的トランザクションではコミットに成功することができます。
```

### TiDBとANSI Repeatable Readの違い

TiDBのRepeatable Read分離レベルは、ANSI Repeatable Read分離レベルとは異なりますが、同じ名前を共有しています。[A Critique of ANSI SQL Isolation Levels](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/tr-95-51.pdf)論文で記述された標準によると、TiDBはSnapshot Isolationレベルを実装しています。この分離レベルは厳密なphantom（A3）を許可しませんが、広範なphantom（P3）とwrite skewを許可します。一方、ANSI Repeatable Read分離レベルはphantom readsを許可しますが、write skewを許可しません。

### TiDBとMySQL Repeatable Readの違い

TiDBのRepeatable Read分離レベルは、MySQLのそれとは異なります。MySQLのRepeatable Read分離レベルは、更新時に現在のバージョンが見えるかどうかをチェックしません。つまり、トランザクションが開始した後に行が更新されても更新を続けることができます。一方、TiDBの楽観的トランザクションは、行がトランザクション開始後に更新された場合、ロールバックされて再試行されます。TiDBの楽観的並行制御でのトランザクションの再試行が失敗する可能性があり、最終的にはトランザクションが完全に失敗するかもしれません。一方、TiDBの悲観的並行制御とMySQLでは、更新トランザクションは成功することができます。

## Read Committed分離レベル

TiDB v4.0.0-beta以降、TiDBはRead Committed分離レベルをサポートしています。

歴史的な理由から、現在の主要なデータベースのRead Committed分離レベルは基本的にはOracleによって定義された[Consistent Read isolation level](https://docs.oracle.com/cd/B19306_01/server.102/b14220/consist.htm)です。この状況に適合するため、TiDBの悲観的トランザクションのRead Committed分離レベルも基本的には一貫した読み取り動作です。

> **注意:**
>
> Read Committed分離レベルは、[pessimistic transaction mode](/pessimistic-transaction.md)でのみ有効です。[optimistic transaction mode](/optimistic-transaction.md)では、トランザクション分離レベルを`Read Committed`に設定しても効果がありませんし、トランザクションは引き続きRepeatable Read分離レベルを使用します。

v6.0.0以降、TiDBでは、稀な読み込み-書き込み競合のシナリオでタイムスタンプ取得を最適化するために、[`tidb_rc_read_check_ts`](/system-variables.md#tidb_rc_read_check_ts-new-in-v600)システム変数を使用できます。この変数を有効にすると、`SELECT`が実行される際に前の有効なタイムスタンプを使用してデータを読もうとします。この変数の初期値はトランザクションの`start_ts`です。

- TiDBが読み取りプロセス中にデータ更新に遭遇しない場合、TiDBは結果をクライアントに返し、`SELECT`文は正常に実行されます。
- TiDBが読み取りプロセス中にデータ更新に遭遇した場合：
    - クライアントに結果をまだ送信していない場合、TiDBは新しいタイムスタンプを取得し、この文を再試行します。
    - クライアントに部分的なデータを送信済みの場合、TiDBはクライアントにエラーを報告します。クライアントへの送信データ量は、`tidb_init_chunk_size`および`tidb_max_chunk_size`で制御されます。

`READ-COMMITTED`分離レベルを使用し、`SELECT`文が多く、読み書き競合が稀な場合、この変数を有効にすることで、グローバルタイムスタンプの取得の遅延やコストを回避することができます。

v6.3.0以降、TiDBは、稀なPoint-Write競合のシナリオでタイムスタンプ取得を最適化するために、システム変数[`tidb_rc_write_check_ts`](/system-variables.md#tidb_rc_write_check_ts-new-in-v630)を使用します。この変数を有効にすると、Point-Write文の実行時にTiDBは現在のトランザクションの有効なタイムスタンプを使用してデータを読み取り、ロックします。[`tidb_rc_read_check_ts`](/system-variables.md#tidb_rc_read_check_ts-new-in-v600)が有効な場合と同じように、TiDBはエラーをクライアントに報告します。Point-Write文の読み取りプロセス全体でデータ更新バージョンに遭遇しない場合、TiDBは引き続き現在のトランザクションのタイムスタンプを使用してデータをロックします。
- Point-Write文に対する読み取りプロセス全体で古いタイムスタンプによる書き込み競合が発生した場合、TiDBは最新のグローバルタイムスタンプを取得してロック取得プロセスを再試行します。
- ポイント書き込み文の読み取りプロセス中に更新されたデータバージョンに遭遇した場合、TiDBは新しいタイムスタンプを取得してこの文を再試行します。

`READ-COMMITTED`分離レベルで多くのPoint-Write文が含まれ、Point-Write競合が少ないトランザクションの場合、この変数を有効にすると、グローバルタイムスタンプの取得の遅延や負荷を回避することができます。

MySQLのRead Committed分離レベルは、ほとんどのケースでConsistent Read機能と一致します。[semi-consistent read](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html)などの例外もあります。TiDBでは、この特別な動作はサポートされていません。