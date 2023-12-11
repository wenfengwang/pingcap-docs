---
title: ADMIN RESUME DDL JOBS
summary: TiDBデータベースでのADMIN RESUME DDLの使用概要。

# ADMIN RESUME DDL JOBS

`ADMIN RESUME DDL`を使用すると、一時停止したDDLジョブを再開できます。[`ADMIN SHOW DDL JOBS`](/sql-statements/sql-statement-admin-show-ddl.md)を実行して`job_id`を見つけることができます。

この文を使用して、一時停止したDDLジョブを再開できます。再開が完了すると、DDLジョブを実行するSQL文は引き続き実行されていると表示されます。既に完了したDDLジョブを再開しようとすると、「`DDL Job:90 not found`」エラーが`RESULT`列に表示されます。これは、ジョブがDDL待機キューから削除されたことを示します。

## 概要

```ebnf+diagram
AdminStmt ::=
    'ADMIN' ( 'SHOW' ( 'DDL' ( 'JOBS' Int64Num? WhereClauseOptional | 'JOB' 'QUERIES' NumList )? | TableName 'NEXT_ROW_ID' | 'SLOW' AdminShowSlow ) | 'CHECK' ( 'TABLE' TableNameList | 'INDEX' TableName Identifier ( HandleRange ( ',' HandleRange )* )? ) | 'RECOVER' 'INDEX' TableName Identifier | 'CLEANUP' ( 'INDEX' TableName Identifier | 'TABLE' 'LOCK' TableNameList ) | 'CHECKSUM' 'TABLE' TableNameList | 'CANCEL' 'DDL' 'JOBS' NumList | 'PAUSE' 'DDL' 'JOBS' NumList | 'RESUME' 'DDL' 'JOBS' NumList | 'RELOAD' ( 'EXPR_PUSHDOWN_BLACKLIST' | 'OPT_RULE_BLACKLIST' | 'BINDINGS' ) | 'PLUGINS' ( 'ENABLE' | 'DISABLE' ) PluginNameList | 'REPAIR' 'TABLE' TableName CreateTableStmt | ( 'FLUSH' | 'CAPTURE' | 'EVOLVE' ) 'BINDINGS' )

NumList ::=
    Int64Num ( ',' Int64Num )*
```

## 例

`ADMIN RESUME DDL JOBS`は現在一時停止しているDDLジョブを再開し、ジョブが再開されたかどうかを返します。

```sql
ADMIN RESUME DDL JOBS job_id [, job_id] ...;
```

再開に失敗した場合、失敗の具体的な理由が表示されます。

<CustomContent platform="tidb">

> **注意:**
>
> + クラスタのアップグレード中、進行中のDDLジョブは一時停止され、アップグレード中に開始されたDDLジョブも一時停止されます。アップグレード後、すべての一時停止したDDLジョブが再開されます。アップグレード中の一時停止および再開操作は自動的に行われます。詳細については、[TiDBスムーズアップグレード](/smooth-upgrade-tidb.md)を参照してください。
> + この文は複数のDDLジョブを再開できます。DDLジョブの`job_id`を取得するには [`ADMIN SHOW DDL JOBS`](/sql-statements/sql-statement-admin-show-ddl.md) 文を使用できます。
> + `一時停止`以外の状態のDDLジョブは再開できず、再開操作は失敗します。
> + ジョブを複数回再開しようとすると、TiDBは `Error Number: 8261` エラーを報告します。

</CustomContent>
<CustomContent platform="tidb-cloud">

> **注意:**
>
> + クラスタのアップグレード中、進行中のDDLジョブは一時停止され、アップグレード中に開始されたDDLジョブも一時停止されます。アップグレード後、すべての一時停止したDDLジョブが再開されます。アップグレード中の一時停止および再開操作は自動的に行われます。詳細については、[TiDBスムーズアップグレード](https://docs.pingcap.com/tidb/stable/smooth-upgrade-tidb)を参照してください。
> + この文は複数のDDLジョブを再開できます。DDLジョブの`job_id`を取得するには [`ADMIN SHOW DDL JOBS`](/sql-statements/sql-statement-admin-show-ddl.md) 文を使用できます。
> + `一時停止`以外の状態のDDLジョブは再開できず、再開操作は失敗します。
> + ジョブを複数回再開しようとすると、TiDBは `Error Number: 8261` エラーを報告します。

</CustomContent>

## MySQL互換性

この文はMySQL構文のTiDB拡張です。

## 関連情報

* [`ADMIN SHOW DDL [JOBS|QUERIES]`](/sql-statements/sql-statement-admin-show-ddl.md)
* [`ADMIN CANCEL DDL`](/sql-statements/sql-statement-admin-cancel-ddl.md)
* [`ADMIN PAUSE DDL`](/sql-statements/sql-statement-admin-pause-ddl.md)