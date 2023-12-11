---
title: ADMIN | TiDB SQL ステートメントリファレンス
summary: TiDB データベースの ADMIN の使用方法の概要
aliases: ['/docs/dev/sql-statements/sql-statement-admin/','/docs/dev/reference/sql/statements/admin/']
---

# ADMIN

このステートメントは TiDB の拡張シンタックスであり、TiDB の状態を表示し、TiDB のテーブルのデータを確認するために使用されます。このドキュメントでは、以下の `ADMIN` 関連のステートメントを紹介します。

- [`ADMIN RELOAD`](#admin-reloadステートメント)
- [`ADMIN PLUGINS`](#admin-plugins関連ステートメント)
- [`ADMIN ... BINDINGS`](#admin-bindings関連ステートメント)
- [`ADMIN REPAIR`](#admin-repairステートメント)
- [`ADMIN SHOW NEXT_ROW_ID`](#admin-show-next_row_idステートメント)
- [`ADMIN SHOW SLOW`](#admin-show-slowステートメント)

## DDL 関連ステートメント

<CustomContent platform="tidb-cloud">

| ステートメント                                                             | 説明                                  |
|--------------------------------------------------------------------------|---------------------------------------|
| [`ADMIN CANCEL DDL JOBS`](/sql-statements/sql-statement-admin-cancel-ddl.md) | 現在実行中の DDL ジョブをキャンセルします。 |
| [`ADMIN PAUSE DDL JOBS`](/sql-statements/sql-statement-admin-pause-ddl.md) | 現在実行中の DDL ジョブを一時停止します。 |
| [`ADMIN RESUME DDL JOBS`](/sql-statements/sql-statement-admin-resume-ddl.md) | 一時停止した DDL ジョブを再開します。 |
| [`ADMIN CHECKSUM TABLE`](/sql-statements/sql-statement-admin-checksum-table.md) | テーブルのすべての行とインデックスの CRC64 を計算します。 |
| [<code>ADMIN CHECK [TABLE\|INDEX]</code>](/sql-statements/sql-statement-admin-check-table-index.md) | テーブルまたはインデックスの整合性をチェックします。 |
| [<code>ADMIN SHOW DDL [JOBS\|QUERIES]</code>](/sql-statements/sql-statement-admin-show-ddl.md) | 現在実行中または最近完了した DDL ジョブに関する詳細を表示します。 |

</CustomContent>

<CustomContent platform="tidb">

| ステートメント                                                             | 説明                                  |
|--------------------------------------------------------------------------|---------------------------------------|
| [`ADMIN CANCEL DDL JOBS`](/sql-statements/sql-statement-admin-cancel-ddl.md) | 現在実行中の DDL ジョブをキャンセルします。 |
| [`ADMIN CHECKSUM TABLE`](/sql-statements/sql-statement-admin-checksum-table.md) | テーブルのすべての行とインデックスの CRC64 を計算します。 |
| [<code>ADMIN CHECK [TABLE\|INDEX]</code>](/sql-statements/sql-statement-admin-check-table-index.md) | テーブルまたはインデックスの整合性をチェックします。 |
| [<code>ADMIN SHOW DDL [JOBS\|QUERIES]</code>](/sql-statements/sql-statement-admin-show-ddl.md) | 現在実行中または最近完了した DDL ジョブに関する詳細を表示します。 |
| [`ADMIN SHOW TELEMETRY`](/sql-statements/sql-statement-admin-show-telemetry.md) | テレメトリ機能の一部として PingCAP に報告される情報を表示します。 |

</CustomContent>

## `ADMIN RELOAD` ステートメント

{{< copyable "sql" >}}

```sql
ADMIN RELOAD expr_pushdown_blacklist;
```

上記のステートメントは、式によって押し込まれたブロックリストを再読み込みするために使用されます。

{{< copyable "sql" >}}

```sql
ADMIN RELOAD opt_rule_blacklist;
```

上記のステートメントは、ロジック最適化ルールのブロックリストを再読み込みするために使用されます。

## `ADMIN PLUGINS` 関連ステートメント

> **注意:**
>
> この機能は [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) クラスターでは利用できません。

```sql
ADMIN PLUGINS ENABLE plugin_name [, plugin_name] ...;
```

上記のステートメントは、`plugin_name` プラグインを有効にするために使用されます。

```sql
ADMIN PLUGINS DISABLE plugin_name [, plugin_name] ...;
```

上記のステートメントは、`plugin_name` プラグインを無効にするために使用されます。

## `ADMIN BINDINGS` 関連ステートメント

{{< copyable "sql" >}}

```sql
ADMIN FLUSH BINDINGS;
```

上記のステートメントは、SQL プランのバインディング情報を永続化するために使用されます。

{{< copyable "sql" >}}

```sql
ADMIN CAPTURE BINDINGS;
```

上記のステートメントは、複数回発生する `SELECT` ステートメントからの SQL プランのバインディングを生成するために使用されます。

{{< copyable "sql" >}}

```sql
ADMIN EVOLVE BINDINGS;
```

自動バインディング機能が有効になっている場合、SQL プランのバインディング情報の進化が `bind-info-leave` (デフォルト値は `3s`) でトリガーされます。上記のステートメントは、この進化を積極的にトリガーするために使用されます。

{{< copyable "sql" >}}

```sql
ADMIN RELOAD BINDINGS;
```

上記のステートメントは、SQL プランのバインディング情報を再読み込みするために使用されます。

## `ADMIN REPAIR` ステートメント

<CustomContent platform="tidb-cloud">

> **注意:**
>
> この TiDB ステートメントは TiDB Cloud では適用されません。

</CustomContent>

信頼性のない方法で格納されたテーブルのメタデータを上書きするために、`ADMIN REPAIR TABLE` を使用します:

{{< copyable "sql" >}}

```sql
ADMIN REPAIR TABLE tbl_name CREATE TABLE STATEMENT;
```

<CustomContent platform="tidb">

ここでの「信頼性のない」は、元のテーブルのメタデータが `CREATE TABLE STATEMENT` 操作によってカバーされることを手動で確認する必要があるという意味です。この `REPAIR` ステートメントを使用するには、[`repair-mode`](/tidb-configuration-file.md#repair-mode) 構成項目を有効にし、修復するテーブルが [`repair-table-list`](/tidb-configuration-file.md#repair-table-list) にリストされていることを確認してください。

</CustomContent>

## `ADMIN SHOW NEXT_ROW_ID` ステートメント

```sql
ADMIN SHOW t NEXT_ROW_ID;
```

上記のステートメントは、テーブルの特定の列の詳細を表示するために使用されます。出力は [SHOW TABLE NEXT_ROW_ID](/sql-statements/sql-statement-show-table-next-rowid.md) と同じです。

## `ADMIN SHOW SLOW` ステートメント

> **注意:**
>
> この機能は [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) クラスターでは利用できません。

```sql
ADMIN SHOW SLOW RECENT N;
```

```sql
ADMIN SHOW SLOW TOP [INTERNAL | ALL] N;
```

<CustomContent platform="tidb">

詳細については、[`ADMIN SHOW SLOW` コマンド](/identify-slow-queries.md#admin-show-slow-command) を参照してください。

</CustomContent>

## 概要

```ebnf+diagram
AdminStmt ::=
    'ADMIN' ( 'SHOW' ( 'DDL' ( 'JOBS' Int64Num? WhereClauseOptional | 'JOB' 'QUERIES' NumList )? | TableName 'NEXT_ROW_ID' | 'SLOW' AdminShowSlow ) | 'CHECK' ( 'TABLE' TableNameList | 'INDEX' TableName Identifier ( HandleRange ( ',' HandleRange )* )? ) | 'RECOVER' 'INDEX' TableName Identifier | 'CLEANUP' ( 'INDEX' TableName Identifier | 'TABLE' 'LOCK' TableNameList ) | 'CHECKSUM' 'TABLE' TableNameList | 'CANCEL' 'DDL' 'JOBS' NumList | 'RELOAD' ( 'EXPR_PUSHDOWN_BLACKLIST' | 'OPT_RULE_BLACKLIST' | 'BINDINGS' ) | 'PLUGINS' ( 'ENABLE' | 'DISABLE' ) PluginNameList | 'REPAIR' 'TABLE' TableName CreateTableStmt | ( 'FLUSH' | 'CAPTURE' | 'EVOLVE' ) 'BINDINGS' )
```

## 例

次のコマンドを実行して、現在実行中の DDL ジョブキューの最後に完了した 10 件の DDL ジョブを表示します。`NUM` が指定されていない場合、デフォルトで最後に完了した 10 件の DDL ジョブのみが表示されます。

{{< copyable "sql" >}}

```sql
ADMIN SHOW DDL JOBS;
```

```
+--------+---------+------------+---------------------+----------------+-----------+----------+-----------+-----------------------------------+-----------------------------------+---------------+
| JOB_ID | DB_NAME | TABLE_NAME | JOB_TYPE            | SCHEMA_STATE   | SCHEMA_ID | TABLE_ID | ROW_COUNT | START_TIME                        | END_TIME                          | STATE         |
+--------+---------+------------+---------------------+----------------+-----------+----------+-----------+-----------------------------------+-----------------------------------+---------------+
| 45     | test    | t1         | add index     | write reorganization | 32        | 37       | 0         | 2019-01-10 12:38:36.501 +0800 CST |                                   | running       |
| 44     | test    | t1         | add index     | none                 | 32        | 37       | 0         | 2019-01-10 12:36:55.18 +0800 CST  | 2019-01-10 12:36:55.852 +0800 CST | rollback done |
```
| 43     | test    | t1         | add index     | public               | 32        | 37       | 6         | 2019-01-10 12:35:13.66 +0800 CST  | 2019-01-10 12:35:14.925 +0800 CST | synced        |
| 42     | test    | t1         | drop index    | none                 | 32        | 37       | 0         | 2019-01-10 12:34:35.204 +0800 CST | 2019-01-10 12:34:36.958 +0800 CST | synced        |
| 41     | test    | t1         | add index     | public               | 32        | 37       | 0         | 2019-01-10 12:33:22.62 +0800 CST  | 2019-01-10 12:33:24.625 +0800 CST | synced        |
| 40     | test    | t1         | drop column   | none                 | 32        | 37       | 0         | 2019-01-10 12:33:08.212 +0800 CST | 2019-01-10 12:33:09.78 +0800 CST  | synced        |
| 39     | test    | t1         | add column    | public               | 32        | 37       | 0         | 2019-01-10 12:32:55.42 +0800 CST  | 2019-01-10 12:32:56.24 +0800 CST  | synced        |
| 38     | test    | t1         | create table  | public               | 32        | 37       | 0         | 2019-01-10 12:32:41.956 +0800 CST | 2019-01-10 12:32:43.956 +0800 CST | synced        |
| 36     | test    |            | drop table    | none                 | 32        | 34       | 0         | 2019-01-10 11:29:59.982 +0800 CST | 2019-01-10 11:30:00.45 +0800  CST | synced        |
| 35     | test    |            | create table  | public               | 32        | 34       | 0         | 2019-01-10 11:29:40.741 +0800 CST | 2019-01-10 11:29:41.682 +0800 CST | synced        |
| 33     | test    |            | create schema | public               | 32        | 0        | 0         | 2019-01-10 11:29:22.813 +0800 CST | 2019-01-10 11:29:23.954 +0800 CST | synced        |
+--------+---------+------------+---------------------+----------------+-----------+----------+-----------+-----------------------------------+-----------------------------------+---------------+
```

現在実行中のDDLジョブキューで最後に完了した5つのDDLジョブを表示するには、次のコマンドを実行します。

```sql
ADMIN SHOW DDL JOBS 5;
```

```
+--------+---------+------------+---------------------+----------------+-----------+----------+-----------+-----------------------------------+-----------------------------------+---------------+
| JOB_ID | DB_NAME | TABLE_NAME | JOB_TYPE            | SCHEMA_STATE   | SCHEMA_ID | TABLE_ID | ROW_COUNT | START_TIME                        | END_TIME                          | STATE         |
+--------+---------+------------+---------------------+----------------+-----------+----------+-----------+-----------------------------------+-----------------------------------+---------------+
| 45     | test    | t1         | add index     | write reorganization | 32        | 37       | 0         | 2019-01-10 12:38:36.501 +0800 CST |                                   | running       |
| 44     | test    | t1         | add index     | none                 | 32        | 37       | 0         | 2019-01-10 12:36:55.18 +0800 CST  | 2019-01-10 12:36:55.852 +0800 CST | rollback done |
| 43     | test    | t1         | add index     | public               | 32        | 37       | 6         | 2019-01-10 12:35:13.66 +0800 CST  | 2019-01-10 12:35:14.925 +0800 CST | synced        |
| 42     | test    | t1         | drop index    | none                 | 32        | 37       | 0         | 2019-01-10 12:34:35.204 +0800 CST | 2019-01-10 12:34:36.958 +0800 CST | synced        |
| 41     | test    | t1         | add index     | public               | 32        | 37       | 0         | 2019-01-10 12:33:22.62 +0800 CST  | 2019-01-10 12:33:24.625 +0800 CST | synced        |
| 40     | test    | t1         | drop column   | none                 | 32        | 37       | 0         | 2019-01-10 12:33:08.212 +0800 CST | 2019-01-10 12:33:09.78 +0800 CST  | synced        |
+--------+---------+------------+---------------------+----------------+-----------+----------+-----------+-----------------------------------+-----------------------------------+---------------+
```

特定のテーブルの特定の列の詳細を表示するには次のコマンドを実行します。出力結果は[SHOW TABLE NEXT_ROW_ID](/sql-statements/sql-statement-show-table-next-rowid.md)と同じです。

```sql
ADMIN SHOW t NEXT_ROW_ID;
```

```sql
+---------+------------+-------------+--------------------+----------------+
| DB_NAME | TABLE_NAME | COLUMN_NAME | NEXT_GLOBAL_ROW_ID | ID_TYPE        |
+---------+------------+-------------+--------------------+----------------+
| test    | t          | _tidb_rowid |                101 | _TIDB_ROWID    |
| test    | t          | _tidb_rowid |                  1 | AUTO_INCREMENT |
+---------+------------+-------------+--------------------+----------------+
2 行が返されました (0.01 sec)
```

テストデータベース内の未完了のDDLジョブを表示するには、次のコマンドを実行します。結果には実行中のDDLジョブと完了したが失敗した最後の5つのDDLジョブが含まれます。

```sql
ADMIN SHOW DDL JOBS 5 WHERE state != 'synced' AND db_name = 'test';
```

```
+--------+---------+------------+---------------------+----------------+-----------+----------+-----------+-----------------------------------+-----------------------------------+---------------+
| JOB_ID | DB_NAME | TABLE_NAME | JOB_TYPE            | SCHEMA_STATE   | SCHEMA_ID | TABLE_ID | ROW_COUNT | START_TIME                        | END_TIME                          | STATE         |
+--------+---------+------------+---------------------+----------------+-----------+----------+-----------+-----------------------------------+-----------------------------------+---------------+
| 45     | test    | t1         | add index     | write reorganization | 32        | 37       | 0         | 2019-01-10 12:38:36.501 +0800 CST |                                   | running       |
| 44     | test    | t1         | add index     | none                 | 32        | 37       | 0         | 2019-01-10 12:36:55.18 +0800 CST  | 2019-01-10 12:36:55.852 +0800 CST | rollback done |
+--------+---------+------------+---------------------+----------------+-----------+----------+-----------+-----------------------------------+-----------------------------------+---------------+
```
* `delete only`, `write only`, `delete reorganization`, `write reorganization`: these four states are intermediate states. These states are not visible in common operations, because the conversion from the intermediate states is so quick. You can see the `write reorganization` state only in `add index` operations, which means that the index data is being added.
    * `public`: it indicates existing and usable. When operations like `create table` and `add index/column` are finished, it usually becomes the `public` state, which means that the created table/column/index can be normally read and written now.

* `SCHEMA_ID`: the ID of the database on which the DDL operations are performed.
* `TABLE_ID`: the ID of the table on which the DDL operations are performed.
* `ROW_COUNT`: the number of the data rows that have been added when running the `add index` operation.
* `START_TIME`: the start time of the DDL operations.
* `END_TIME`: the end time of the DDL operations.
* `STATE`: the state of the DDL operations. The common states include:
    * `none`: it indicates that the operation task has been put in the DDL job queue but has not been performed yet, because it is waiting for the previous tasks to complete. Another reason might be that it becomes the `none` state after running the drop operation, but it will soon be updated to the `synced` state, which means that all TiDB instances have been synced to this state.
    * `running`: it indicates that the operation is being performed.
    * `synced`: it indicates that the operation has been performed successfully and all TiDB instances have been synced to this state.
    * `rollback done`: it indicates that the operation has failed and has finished rolling back.
    * `rollingback`: it indicates that the operation has failed and is rolling back.
    * `cancelling`: it indicates that the operation is being cancelled. This state only occurs when you cancel DDL jobs using the [`ADMIN CANCEL DDL JOBS`](/sql-statements/sql-statement-admin-cancel-ddl.md) command.
    * `paused`: it indicates that the operation has been paused. This state only appears when you use the [`ADMIN PAUSED DDL JOBS`](/sql-statements/sql-statement-admin-pause-ddl.md) command to pause the DDL job. You can use the [`ADMIN RESUME DDL JOBS`](/sql-statements/sql-statement-admin-resume-ddl.md) command to resume the DDL job.

## MySQL compatibility

This statement is a TiDB extension to MySQL syntax.