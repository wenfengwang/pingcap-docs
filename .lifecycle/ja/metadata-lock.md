---
title: メタデータロック
summary: TiDBにおけるメタデータロックの概念、原則、および実装の詳細を紹介します。

# メタデータロック

このドキュメントでは、TiDBにおけるメタデータロックについて紹介します。

## 概念

TiDBはオンライン非同期スキーマ変更アルゴリズムを使用してメタデータオブジェクトの変更をサポートしています。トランザクションが実行されると、トランザクション開始時に対応するメタデータスナップショットを取得します。トランザクション中にメタデータが変更された場合、データの整合性を確保するため、TiDBは「Information schema is changed」というエラーを返し、トランザクションはコミットに失敗します。

この問題を解決するため、TiDB v6.3.0ではメタデータロックをオンラインDDLアルゴリズムに導入しています。ほとんどのDMLエラーを避けるために、TiDBはテーブルメタデータの変更中にDMLとDDLの優先度を調整し、古いメタデータを持つDMLのコミットを待ってからDDLを実行します。

## シナリオ

TiDBのメタデータロックは、次のようなすべてのDDLステートメントに適用されます:

- [`ADD INDEX`](/sql-statements/sql-statement-add-index.md)
- [`ADD COLUMN`](/sql-statements/sql-statement-add-column.md)
- [`DROP COLUMN`](/sql-statements/sql-statement-drop-column.md)
- [`DROP INDEX`](/sql-statements/sql-statement-drop-index.md)
- [`DROP PARTITION`](/partitioned-table.md#partition-management)
- [`TRUNCATE TABLE`](/sql-statements/sql-statement-truncate.md)
- [`EXCHANGE PARTITION`](/partitioned-table.md#partition-management)
- [`CHANGE COLUMN`](/sql-statements/sql-statement-change-column.md)
- [`MODIFY COLUMN`](/sql-statements/sql-statement-modify-column.md)

メタデータロックを有効にすると、TiDBのDDLタスクの実行にいくらかのパフォーマンスへの影響がある場合があります。その影響を軽減するために、次に示すシナリオではメタデータロックは必要ありません:

+ 自動コミットが有効な`SELECT`クエリ
+ スタールリードが有効
+ 一時テーブルへのアクセス

## 使用法

v6.5.0からは、TiDBはデフォルトでメタデータロックを有効にします。v6.4.0以前の既存のクラスタをv6.5.0以降にアップグレードすると、TiDBは自動的にメタデータロックを有効にします。メタデータロックを無効にするには、システム変数[`tidb_enable_metadata_lock`](/system-variables.md#tidb_enable_metadata_lock-new-in-v630)を`OFF`に設定できます。

## 影響

- DMLに対して、メタデータロックはその実行をブロックしたり、デッドロックを引き起こしたりしません。
- メタデータロックが有効になっている場合、トランザクション内のメタデータオブジェクトの情報は最初のアクセス時に決定され、その後は変更されません。
- DDLに対して、メタデータの状態を変更する場合、古いトランザクションによってDDLがブロックされることがあります。以下に例を示します:

    | セッション1                                   | セッション2                                |
    |:----------------------------------------------|:-------------------------------------------|
    | `CREATE TABLE t (a INT);`                     |                                           |
    | `INSERT INTO t VALUES(1);`                    |                                           |
    | `BEGIN;`                                      |                                           |
    |                                               | `ALTER TABLE t ADD COLUMN b INT;`         |
    | `SELECT * FROM t;`（テーブル`t`の現在のメタデータバージョンを使用。(`a=1, b=NULL`)を返し、テーブル`t`をロック。） |                                           |
    |                                               | `ALTER TABLE t ADD COLUMN c INT;` (セッション1によってブロックされます)|

    繰り返し読み取り（repeatable read）分離レベルでは、トランザクション開始からテーブルのメタデータが決定されるまでの時間に、インデックスの追加やカラムタイプの変更などのデータの変更を必要とするDDLが実行されると、以下のようなエラーが発生します:

    | セッション1                                   | セッション2                                |
    |:----------------------------------------------|:-------------------------------------------|
    | `CREATE TABLE t (a INT);`                     |                                           |
    | `INSERT INTO t VALUES(1);`                    |                                           |
    | `BEGIN;`                                      |                                           |
    |                                               | `ALTER TABLE t ADD INDEX idx(a);`          |
    | `SELECT * FROM t;`（インデックス`idx`が利用できません）|                                           |
    | `COMMIT;`                                     |                                           |
    | `BEGIN;`                                      |                                           |
    |                                               | `ALTER TABLE t MODIFY COLUMN a CHAR(10);` |
    | `SELECT * FROM t;`（`Error 8028: Information schema is changed`を返します） | |

## 観察

TiDB v6.3.0では、現在ブロックされているDDLの情報を取得するために `mysql.tidb_mdl_view` ビューが導入されています。

> **注記:**
>
> `mysql.tidb_mdl_view` ビューを選択するには、[`PROCESS` 権限](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_process)が必要です。

次に、テーブル`t`にインデックスを追加することを例に考えてみましょう。DDLステートメント `ALTER TABLE t ADD INDEX idx(a)` があるとします:

```sql
SELECT * FROM mysql.tidb_mdl_view\G
*************************** 1. row ***************************
    JOB_ID: 141
   DB_NAME: test
TABLE_NAME: t
     QUERY: ALTER TABLE t ADD INDEX idx(a)
SESSION ID: 2199023255957
  TxnStart: 08-30 16:35:41.313(435643624013955072)
SQL_DIGESTS: ["begin","select * from `t`"]
1 row in set (0.02 sec)
```

前述の出力から、`SESSION ID` が `2199023255957` のトランザクションが `ADD INDEX` DDL をブロックしていることがわかります。`SQL_DIGEST` にはこのトランザクションによって実行されたSQLステートメントが表示されており、``["begin","select * from `t`"]`` です。ブロックされているDDLを続行させるには、次のグローバルな `KILL` ステートメントを使用して `2199023255957` のトランザクションを終了させることができます:

```sql
mysql> KILL 2199023255957;
Query OK, 0 rows affected (0.00 sec)
```

トランザクションを終了させた後、再度 `mysql.tidb_mdl_view` ビューを選択できます。この時点では、前述のトランザクションは出力に表示されないため、DDLがブロックされていないことを示します。

```sql
SELECT * FROM mysql.tidb_mdl_view\G
Empty set (0.01 sec)
```

## 原則

### 問題の説明

TiDBにおけるDDL操作はオンラインDDLモードです。DDLステートメントの実行中、変更される定義オブジェクトのメタデータバージョンは複数のマイナーバージョン変更を経る可能性があります。オンライン非同期メタデータ変更アルゴリズムは、二つの隣接するマイナーバージョンが互換性があること、つまり、DDLによって変更されるオブジェクトのデータ整合性が壊れないことを確立します。

テーブルにインデックスを追加する場合、DDLステートメントの状態は次のように変化します: None -> Delete Only, Delete Only -> Write Only, Write Only -> Write Reorg, Write Reorg -> Public.

以下のトランザクションのコミットプロセスが前述の制約を破る:

| トランザクション | トランザクションで使用されるバージョン | クラスタ内の最新バージョン | バージョン差分 |
|:-------------|:----------------------------|:-------------------------|:------------|
| txn1         | None                        | None                     | 0           |
| txn2         | DeleteOnly                  | DeleteOnly               | 0           |
| txn3         | WriteOnly                   | WriteOnly                | 0           |
| txn4         | None                        | WriteOnly                | 2           |
| txn5         | WriteReorg                  | WriteReorg               | 0           |
| txn6         | WriteOnly                   | WriteReorg               | 1           |
| txn7         | Public                      | Public                   | 0           |

上記の表では、`txn4` がコミットされた際に使用されるメタデータバージョンは、クラスタ内の最新バージョンと2つのバージョンが異なります。これによりデータの不整合が発生する可能性があります。

### 実装の詳細

メタデータロックは、TiDBクラスタ内のすべてのトランザクションによって使用されるメタデータバージョンが、最大で差分バージョンが1つであることを保証します。この目標を達成するために、TiDBは次の2つのルールを実装しています:

- DMLを実行する際、TiDBは、トランザクションのコンテキスト（テーブル、ビュー、および関連するメタデータバージョンなど）にアクセスしたメタデータオブジェクトを記録します。これらの記録は、トランザクションがコミットされるときにクリーンアップされます。
- DDLステートメントが状態を変更すると、最新のメタデータバージョンがすべてのTiDBノードにpushされます。TiDBノード上のこのステート変更に関連するすべてのトランザクションによって使用されるメタデータバージョンと現在のメタデータバージョンの差が2未満の場合、TiDBノードはメタデータオブジェクトのメタデータロックを取得したと見なされます。次の状態変更は、クラスタ内のすべてのTiDBノードがメタデータオブジェクトのメタデータロックを取得した後にのみ実行できます。