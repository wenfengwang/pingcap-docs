---
title: ADMIN CANCEL DDL | TiDB SQLステートメントリファレンス
summary: TiDBデータベースのADMIN CANCEL DDLの使用方法の概要です。
category: リファレンス
---

# ADMIN CANCEL DDL

`ADMIN CANCEL DDL` ステートメントは、実行中のDDLジョブをキャンセルすることができます。`job_id` は、[`ADMIN SHOW DDL JOBS`](/sql-statements/sql-statement-admin-show-ddl.md) を実行して見つけることができます。

`ADMIN CANCEL DDL` ステートメントは、コミットされたがまだ実行が完了していないDDLジョブをキャンセルすることもできます。キャンセル後、DDLジョブを実行するSQLステートメントは `ERROR 8214 (HY000): Cancelled DDL job` エラーを返します。すでに完了したDDLジョブをキャンセルすると、`RESULT` 列に `DDL Job:90 not found` エラーが表示され、ジョブがDDL待機キューから削除されたことを示します。

## 概要

```ebnf+diagram
AdminStmt ::=
    'ADMIN' ( 'SHOW' ( 'DDL' ( 'JOBS' Int64Num? WhereClauseOptional | 'JOB' 'QUERIES' NumList )? | TableName 'NEXT_ROW_ID' | 'SLOW' AdminShowSlow ) | 'CHECK' ( 'TABLE' TableNameList | 'INDEX' TableName Identifier ( HandleRange ( ',' HandleRange )* )? ) | 'RECOVER' 'INDEX' TableName Identifier | 'CLEANUP' ( 'INDEX' TableName Identifier | 'TABLE' 'LOCK' TableNameList ) | 'CHECKSUM' 'TABLE' TableNameList | 'CANCEL' 'DDL' 'JOBS' NumList | 'RELOAD' ( 'EXPR_PUSHDOWN_BLACKLIST' | 'OPT_RULE_BLACKLIST' | 'BINDINGS' ) | 'PLUGINS' ( 'ENABLE' | 'DISABLE' ) PluginNameList | 'REPAIR' 'TABLE' TableName CreateTableStmt | ( 'FLUSH' | 'CAPTURE' | 'EVOLVE' ) 'BINDINGS' )

NumList ::=
    Int64Num ( ',' Int64Num )*
```

## 例

現在実行中のDDLジョブをキャンセルし、対応するジョブが正常にキャンセルされたかどうかを返すには、`ADMIN CANCEL DDL JOBS` を使用します。

{{< copyable "sql" >}}

```sql
ADMIN CANCEL DDL JOBS job_id [, job_id] ...;
```

ジョブのキャンセルに失敗した場合、具体的な理由が表示されます。

> **注意:**
>
> - この操作だけがDDLジョブをキャンセルできます。他の操作や環境の変更（マシンの再起動やクラスタの再起動など）ではこれらのジョブをキャンセルできません。
> - この操作では複数のDDLジョブを同時にキャンセルできます。DDLジョブのIDは、[`ADMIN SHOW DDL JOBS`](/sql-statements/sql-statement-admin-show-ddl.md) ステートメントを使用して取得できます。
> - キャンセルしたいジョブが完了している場合、キャンセル操作は失敗します。

## MySQL互換性

このステートメントはMySQL構文へのTiDBの拡張です。

## 関連情報

* [`ADMIN SHOW DDL [JOBS|JOB QUERIES]`](/sql-statements/sql-statement-admin-show-ddl.md)