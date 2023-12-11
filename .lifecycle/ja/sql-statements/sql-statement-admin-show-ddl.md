---
title: ADMIN SHOW DDL [JOBS|JOB QUERIES] | TiDB SQLステートメントリファレンス
summary: TiDBデータベースのADMINの使用法の概要。

# ADMIN SHOW DDL [JOBS|JOB QUERIES]

`ADMIN SHOW DDL [JOBS|JOB QUERIES]`ステートメントは、実行中および最近完了したDDLジョブに関する情報を表示します。

## 概要

```ebnf+diagram
AdminStmt ::=
    'ADMIN' ( 'SHOW' ( 'DDL' ( 'JOBS' Int64Num? WhereClauseOptional | 'JOB' 'QUERIES' NumList | 'JOB' 'QUERIES' 'LIMIT' m 'OFFSET' n )? | TableName 'NEXT_ROW_ID' | 'SLOW' AdminShowSlow ) | 'CHECK' ( 'TABLE' TableNameList | 'INDEX' TableName Identifier ( HandleRange ( ',' HandleRange )* )? ) | 'RECOVER' 'INDEX' TableName Identifier | 'CLEANUP' ( 'INDEX' TableName Identifier | 'TABLE' 'LOCK' TableNameList ) | 'CHECKSUM' 'TABLE' TableNameList | 'CANCEL' 'DDL' 'JOBS' NumList | 'RELOAD' ( 'EXPR_PUSHDOWN_BLACKLIST' | 'OPT_RULE_BLACKLIST' | 'BINDINGS' ) | 'PLUGINS' ( 'ENABLE' | 'DISABLE' ) PluginNameList | 'REPAIR' 'TABLE' TableName CreateTableStmt | ( 'FLUSH' | 'CAPTURE' | 'EVOLVE' ) 'BINDINGS' )

NumList ::=
    Int64Num ( ',' Int64Num )*

WhereClauseOptional ::=
    WhereClause?
```

## 例

### `ADMIN SHOW DDL`

現在実行中のDDLジョブのステータスを表示するには、`ADMIN SHOW DDL`を使用します。出力には、現在のスキーマバージョン、DDL IDおよび所有者のアドレス、実行中のDDLジョブおよびSQLステートメント、現在のTiDBインスタンスのDDL IDが含まれます。

{{< copyable "sql" >}}

```sql
ADMIN SHOW DDL;
```

```sql
mysql> ADMIN SHOW DDL;
+------------+--------------------------------------+---------------+--------------+--------------------------------------+-------+
| SCHEMA_VER | OWNER_ID                             | OWNER_ADDRESS | RUNNING_JOBS | SELF_ID                              | QUERY |
+------------+--------------------------------------+---------------+--------------+--------------------------------------+-------+
|         26 | 2d1982af-fa63-43ad-a3d5-73710683cc63 | 0.0.0.0:4000  |              | 2d1982af-fa63-43ad-a3d5-73710683cc63 |       |
+------------+--------------------------------------+---------------+--------------+--------------------------------------+-------+
1 row in set (0.00 sec)
```

### `ADMIN SHOW DDL JOBS`

`ADMIN SHOW DDL JOBS`ステートメントは、現在のDDLジョブキューのすべての結果を表示します。これには、実行中およびキューイングされたタスク、および完了したDDLジョブキューの最新の10件の結果が含まれます。 返される結果のフィールドは以下のように説明されています：

<CustomContent platform="tidb">

- `JOB_ID`: 各DDL操作にはDDLジョブが対応します。 `JOB_ID`はグローバルに一意です。
- `DB_NAME`: DDL操作が行われているデータベースの名前。
- `TABLE_NAME`: DDL操作が行われているテーブルの名前。
- `JOB_TYPE`: DDL操作のタイプ。一般的なジョブタイプには以下が含まれます：
    - `ingest`: [`tidb_ddl_enable_fast_reorg`](/system-variables.md#tidb_ddl_enable_fast_reorg-new-in-v630)で構成されたアクセラレーテッドインデックスバックフィリングを伴う挿入。
    - `txn`: 基本的なトランザクションバックフィル。
    - `txn-merge`: 一時インデックスを使用し、バックフィルが完了すると元のインデックスとマージされるトランザクションバックフィル。
- `SCHEMA_STATE`: DDLが作用するスキーマオブジェクトの現在の状態。`JOB_TYPE`が`ADD INDEX`の場合、インデックスの状態です。`JOB_TYPE`が`ADD COLUMN`の場合、列の状態です。`JOB_TYPE`が`CREATE TABLE`の場合、テーブルの状態です。 一般的な状態には以下が含まれます：
    - `none`: 存在しないことを示します。一般的に`DROP`操作の後、または`CREATE`操作が失敗してロールバックされた後、`none`の状態になります。
    - `delete only`, `write only`, `delete reorganization`, `write reorganization`: これらの4つの状態は中間状態です。詳細な意味については、[TiDBでのオンラインDDL非同期変更の動作](/ddl-introduction.md#how-the-online-ddl-asynchronous-change-works-in-tidb)を参照してください。中間状態への変換が速いため、これらの状態は一般的に操作中には見えません。 `ADD INDEX`操作を実行すると、`write reorganization`状態が見える場合があります。これは、インデックスデータが追加されていることを示します。
    - `public`: 存在し、ユーザーが使用可能であることを示します。`CREATE TABLE`および`ADD INDEX`（または`ADD COLUMN`）の操作が完了した後、`public`状態になります。これは、新しく作成されたテーブル、列、およびインデックスを通常通りに読み書きできることを示します。
- `SCHEMA_ID`: DDL操作が行われているデータベースのID。
- `TABLE_ID`: DDL操作が行われているテーブルのID。
- `ROW_COUNT`: `ADD INDEX`操作を実行すると、追加されたデータ行の数です。
- `START_TIME`: DDL操作の開始時刻。
- `STATE`: DDL操作の状態。一般的な状態には以下が含まれます：
    - `queueing`: 操作ジョブがDDLジョブキューに入っており、以前のDDLジョブの完了を待っているため実行されていません。 別の理由は、`DROP`操作の後には`none`状態になりますが、すぐに`synced`状態に更新され、すべてのTiDBインスタンスがその状態に同期されたことを示します。
    - `running`: 操作が実行中であることを示します。
    - `synced`: 操作が正常に実行され、すべてのTiDBインスタンスがこの状態に同期されたことを示します。
    - `rollback done`: 操作が失敗し、ロールバックが完了したことを示します。
    - `rollingback`: 操作が失敗し、ロールバック中であることを示します。
    - `cancelling`: 操作がキャンセルされていることを示します。 この状態は、[`ADMIN CANCEL DDL JOBS`](/sql-statements/sql-statement-admin-cancel-ddl.md)コマンドを使用してDDLジョブをキャンセルするときにのみ表示されます。
    - `paused`: 操作が一時停止されていることを示します。 この状態は、[`ADMIN PAUSED DDL JOBS`](/sql-statements/sql-statement-admin-pause-ddl.md)コマンドを使用してDDLジョブを一時停止したときにのみ表示されます。 [`ADMIN RESUME DDL JOBS`](/sql-statements/sql-statement-admin-resume-ddl.md)コマンドを使用してDDLジョブを再開できます。

</CustomContent>

<CustomContent platform="tidb-cloud">

- `JOB_ID`: 各DDL操作にはDDLジョブが対忋します。 `JOB_ID`はグローバルに一意です。
- `DB_NAME`: DDL操作が行われているデータベースの名前。
- `TABLE_NAME`: DDL操作が行われているテーブルの名前。
- `JOB_TYPE`: DDL操作のタイプ。
- `SCHEMA_STATE`: DDLが作用するスキーマオブジェクトの現在の状態。`JOB_TYPE`が`ADD INDEX`の場合、インデックスの状態です。`JOB_TYPE`が`ADD COLUMN`の場合、列の状態です。`JOB_TYPE`が`CREATE TABLE`の場合、テーブルの状態です。 一般的な状態には以下が含まれます：
    - `none`: 存在しないことを示します。 一般的に`DROP`操作の後、または`CREATE`操作が失敗してロールバックされた後、`none`の状態になります。
    - `delete only`, `write only`, `delete reorganization`, `write reorganization`: これらの4つの状態は中間状態です。詳細な意味については、[TiDBでのオンラインDDL非同期変更の動作](https://docs.pingcap.com/tidb/stable/ddl-introduction#how-the-online-ddl-asynchronous-change-works-in-tidb)を参照してください。中間状態への変換が速いため、これらの状態は一般的に操作中には見えません。 `ADD INDEX`操作を実行すると、`write reorganization`状態が見える場合があります。これは、インデックスデータが追加されていることを示します。
    - `public`: 存在し、ユーザーが使用可能であることを示します。`CREATE TABLE`および`ADD INDEX`（または`ADD COLUMN`）の操作が完了した後、`public`状態になります。これは、新しく作成されたテーブル、列、およびインデックスを通常通りに読み書きできることを示します。
- `SCHEMA_ID`: DDL操作が行われているデータベースのID。
- `TABLE_ID`: DDL操作が行われているテーブルのID。
- `ROW_COUNT`: `ADD INDEX`操作を実行すると、追加されたデータ行の数です。
- `START_TIME`: DDL操作の開始時刻。
- `STATE`: DDL操作の状態。一般的な状態には以下が含まれます：
    - `queueing`: 操作ジョブがDDLジョブキューに入っており、以前のDDLジョブの完了を待っているため実行されていません。 別の理由は、`DROP`操作の後には`none`状態になりますが、すぐに`synced`状態に更新され、すべてのTiDBインスタンスがその状態に同期されたことを示します。
    - `running`: 操作が実行中であることを示します。
    - `synced`: 操作が正常に実行され、すべてのTiDBインスタンスがこの状態に同期されたことを示します。
    - `rollback done`: 操作が失敗し、ロールバックが完了したことを示します。
    - `rollingback`: 操作が失敗し、ロールバック中であることを示します。
    - `cancelling`: 操作がキャンセルされていることを示します。 この状態は、[`ADMIN CANCEL DDL JOBS`](/sql-statements/sql-statement-admin-cancel-ddl.md)コマンドを使用してDDLジョブをキャンセルするときにのみ表示されます。
    - `paused`: 操作が一時停止されていることを示します。 この状態は、[`ADMIN PAUSED DDL JOBS`](/sql-statements/sql-statement-admin-pause-ddl.md)コマンドを使用してDDLジョブを一時停止したときにのみ表示されます。 [`ADMIN RESUME DDL JOBS`](/sql-statements/sql-statement-admin-resume-ddl.md)コマンドを使用してDDLジョブを再開できます。

</CustomContent>
    - `queueing`: オペレーションジョブがDDLジョブキューに入ったことを示し、まだ実行されていないため、以前のDDLジョブの完了を待っています。もう1つの理由は、`DROP`オペレーションを実行した後、`none`状態になりますが、すぐに`synced`状態に更新され、すべてのTiDBインスタンスがその状態に同期されたことを示しています。
    - `running`: オペレーションが実行されていることを示します。
    - `synced`: オペレーションが正常に実行され、すべてのTiDBインスタンスがこの状態に同期されていることを示します。
    - `rollback done`: オペレーションが失敗し、ロールバックが完了したことを示します。
    - `rollingback`: オペレーションが失敗し、ロールバックが行われています。
    - `cancelling`: オペレーションがキャンセルされていることを示します。この状態は、[`ADMIN CANCEL DDL JOBS`](/sql-statements/sql-statement-admin-cancel-ddl.md)コマンドを使用してDDLジョブをキャンセルする場合のみ表示されます。
    - `paused`: オペレーションが一時停止されていることを示します。この状態は、[`ADMIN PAUSED DDL JOBS`](/sql-statements/sql-statement-admin-pause-ddl.md)コマンドを使用してDDLジョブを一時停止する場合のみ表示されます。[`ADMIN RESUME DDL JOBS`](/sql-statements/sql-statement-admin-resume-ddl.md)コマンドを使用してDDLジョブを再開できます。

</CustomContent>

以下の例は、`ADMIN SHOW DDL JOBS`の結果を示しています:

{{< copyable "sql" >}}

```sql
ADMIN SHOW DDL JOBS;
```

```sql
mysql> ADMIN SHOW DDL JOBS;
+--------+---------+--------------------+--------------+----------------------+-----------+----------+-----------+---------------------+---------------------------------------------+---------------------+---------+
| JOB_ID | DB_NAME | TABLE_NAME         | JOB_TYPE     | SCHEMA_STATE         | SCHEMA_ID | TABLE_ID | ROW_COUNT | CREATE_TIME         | START_TIME          | END_TIME            | STATE   |
+--------+---------+--------------------+--------------+----------------------+-----------+----------+-----------+---------------------+---------------------+---------------------+---------+
|     59 | test    | t1                 | add index    | write reorganization |         1 |       55 |     88576 | 2020-08-17 07:51:58 | 2020-08-17 07:51:58 | NULL                | running |
|     60 | test    | t2                 | add index    | none                 |         1 |       57 |         0 | 2020-08-17 07:51:59 | 2020-08-17 07:51:59 | NULL                | none    |
|     58 | test    | t2                 | create table | public               |         1 |       57 |         0 | 2020-08-17 07:41:28 | 2020-08-17 07:41:28 | 2020-08-17 07:41:28 | synced  |
|     56 | test    | t1                 | create table | public               |         1 |       55 |         0 | 2020-08-17 07:41:02 | 2020-08-17 07:41:02 | 2020-08-17 07:41:02 | synced  |
|     54 | test    | t1                 | drop table   | none                 |         1 |       50 |         0 | 2020-08-17 07:41:02 | 2020-08-17 07:41:02 | 2020-08-17 07:41:02 | synced  |
|     53 | test    | t1                 | drop index   | none                 |         1 |       50 |         0 | 2020-08-17 07:35:44 | 2020-08-17 07:35:44 | 2020-08-17 07:35:44 | synced  |
|     52 | test    | t1                 | add index    | public               |         1 |       50 |    451010 | 2020-08-17 07:34:43 | 2020-08-17 07:34:43 | 2020-08-17 07:35:16 | synced  |
|     51 | test    | t1                 | create table | public               |         1 |       50 |         0 | 2020-08-17 07:34:02 | 2020-08-17 07:34:02 | 2020-08-17 07:34:02 | synced  |
|     49 | test    | t1                 | drop table   | none                 |         1 |       47 |         0 | 2020-08-17 07:34:02 | 2020-08-17 07:34:02 | 2020-08-17 07:34:02 | synced  |
|     48 | test    | t1                 | create table | public               |         1 |       47 |         0 | 2020-08-17 07:33:37 | 2020-08-17 07:33:37 | 2020-08-17 07:33:37 | synced  |
|     46 | mysql   | stats_extended     | create table | public               |         3 |       45 |         0 | 2020-08-17 06:42:38 | 2020-08-17 06:42:38 | 2020-08-17 06:42:38 | synced  |
|     44 | mysql   | opt_rule_blacklist | create table | public               |         3 |       43 |         0 | 2020-08-17 06:42:38 | 2020-08-17 06:42:38 | 2020-08-17 06:42:38 | synced  |
+--------+---------+--------------------+--------------+----------------------+-----------+----------+-----------+---------------------+---------------------+---------------------+---------+
12 rows in set (0.00 sec)
```

上記の出力から:

- ジョブ59は現在進行中です(`STATE`が`running`です)。スキーマの状態は現在`write reorganization`ですが、タスクが完了すると`public`に切り替わり、変更がユーザーセッションで公開されることが確認できます。また、`end_time`列も`NULL`であり、ジョブの完了時間が現在わかっていません。

- ジョブ60は`add index`ジョブで、現在はジョブ59の完了を待ってキューに入っています。ジョブ59が完了すると、ジョブ60の`STATE`は`running`に切り替わります。

- インデックスの削除やテーブルの削除などの破壊的な変更の場合、ジョブが完了すると`SCHEMA_STATE`が`none`に変わります。追加変更の場合、`SCHEMA_STATE`が`public`に変わります。

行の数を制限するには、番号とwhere条件を指定します:

```sql
ADMIN SHOW DDL JOBS [NUM] [WHERE where_condition];
```

* `NUM`: 完了したDDLジョブキュー内の最後の`NUM`結果を表示します。指定されていない場合、`NUM`はデフォルトで10です。
* `WHERE`: フィルタ条件を追加するために使用します。

### `ADMIN SHOW DDL JOB QUERIES`

`job_id`に対応するDDLジョブの元のSQLステートメントを表示するには、`ADMIN SHOW DDL JOB QUERIES`を使用します:

{{< copyable "sql" >}}

```sql
ADMIN SHOW DDL JOBS;
ADMIN SHOW DDL JOB QUERIES 51;
```

```sql
mysql> ADMIN SHOW DDL JOB QUERIES 51;
+--------------------------------------------------------------+
| QUERY                                                        |
+--------------------------------------------------------------+
| CREATE TABLE t1 (id INT NOT NULL PRIMARY KEY auto_increment) |
+--------------------------------------------------------------+
1 row in set (0.02 sec)
```

`job_id`に対応する実行中のDDLジョブを、DDL実行履歴ジョブキューの直近10の結果内で検索することができます。

### `ADMIN SHOW DDL JOB QUERIES LIMIT m OFFSET n`

`job_id`に対応するDDLジョブの元のSQLステートメントを指定された範囲`[n+1, n+m]`で表示するには、`ADMIN SHOW DDL JOB QUERIES LIMIT m OFFSET n`を使用します:

{{< copyable "sql" >}}

```sql
 ADMIN SHOW DDL JOB QUERIES LIMIT m;  # 最初のm行を取得
 ADMIN SHOW DDL JOB QUERIES LIMIT n, m;  # 行[n+1, n+m]を取得
 ADMIN SHOW DDL JOB QUERIES LIMIT m OFFSET n;  # 行[n+1, n+m]を取得
 ```

`n`と`m`は0以上の整数です。

 ```sql
 ADMIN SHOW DDL JOB QUERIES LIMIT 3;  # 最初の3行を取得
 +--------+--------------------------------------------------------------+
 | JOB_ID | QUERY                                                        |
 +--------+--------------------------------------------------------------+
 |     59 | ALTER TABLE t1 ADD INDEX index2 (col2)                       |
 |     60 | ALTER TABLE t2 ADD INDEX index1 (col1)                       |
```
|     58 | CREATE TABLE t2 (id INT NOT NULL PRIMARY KEY auto_increment) |
+--------+--------------------------------------------------------------+
3 rows in set (0.00 sec)
```

```sql
ADMIN SHOW DDL JOB QUERIES LIMIT 6, 2;  # 7～8行目の行を取得
+--------+----------------------------------------------------------------------------+
| JOB_ID | QUERY                                                                      |
+--------+----------------------------------------------------------------------------+
|     52 | ALTER TABLE t1 ADD INDEX index1 (col1)                                     |
|     51 | CREATE TABLE IF NOT EXISTS t1 (id INT NOT NULL PRIMARY KEY auto_increment) |
+--------+----------------------------------------------------------------------------+
3 rows in set (0.00 sec)
```

```sql
ADMIN SHOW DDL JOB QUERIES LIMIT 3 OFFSET 4;  # 5～7行目の行を取得
+--------+----------------------------------------+
| JOB_ID | QUERY                                  |
+--------+----------------------------------------+
|     54 | DROP TABLE IF EXISTS t3                |
|     53 | ALTER TABLE t1 DROP INDEX index1       |
|     52 | ALTER TABLE t1 ADD INDEX index1 (col1) |
+--------+----------------------------------------+
3 rows in set (0.00 sec)
```

任意のDDL履歴ジョブ列の結果範囲内で、`job_id` に対応する実行中のDDLジョブを検索できます。この構文は、`ADMIN SHOW DDL JOB QUERIES` の過去10個の結果の制限がありません。

## MySQL 互換性

このステートメントは、TiDBのMySQL構文に対する拡張です。

## 関連情報

* [ADMIN CANCEL DDL](/sql-statements/sql-statement-admin-cancel-ddl.md)
* [ADMIN PAUSE DDL](/sql-statements/sql-statement-admin-pause-ddl.md)
* [ADMIN RESUME DDL](/sql-statements/sql-statement-admin-resume-ddl.md)
```