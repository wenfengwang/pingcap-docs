---
title: ADMIN PAUSE DDL ジョブ
summary: TiDBデータベースでのADMIN PAUSE DDL ジョブの使用の概要。

---

# ADMIN PAUSE DDL ジョブ

`ADMIN PAUSE DDL`を使用すると、実行中のDDLジョブを一時停止できます。`job_id`は、[`ADMIN SHOW DDL JOBS`](/sql-statements/sql-statement-admin-show-ddl.md)を実行して見つけることができます。

このステートメントを使用して、発行されたDDLジョブがまだ完了していない場合に一時停止できます。一時停止後、DDLジョブを実行するSQLステートメントはすぐには戻らず、まだ実行中のように見えます。すでに完了しているDDLジョブを一時停止しようとすると、「DDL Job:90 not found」エラーが「RESULT」列に表示され、ジョブがDDL待機キューから削除されたことを示します。

## 構文

```ebnf+diagram
AdminStmt ::=
    'ADMIN' ( 'SHOW' ( 'DDL' ( 'JOBS' Int64Num? WhereClauseOptional | 'JOB' 'QUERIES' NumList )? | TableName 'NEXT_ROW_ID' | 'SLOW' AdminShowSlow ) | 'CHECK' ( 'TABLE' TableNameList | 'INDEX' TableName Identifier ( HandleRange ( ',' HandleRange )* )? ) | 'RECOVER' 'INDEX' TableName Identifier | 'CLEANUP' ( 'INDEX' TableName Identifier | 'TABLE' 'LOCK' TableNameList ) | 'CHECKSUM' 'TABLE' TableNameList | 'CANCEL' 'DDL' 'JOBS' NumList | 'PAUSE' 'DDL' 'JOBS' NumList | 'RESUME' 'DDL' 'JOBS' NumList | 'RELOAD' ( 'EXPR_PUSHDOWN_BLACKLIST' | 'OPT_RULE_BLACKLIST' | 'BINDINGS' ) | 'PLUGINS' ( 'ENABLE' | 'DISABLE' ) PluginNameList | 'REPAIR' 'TABLE' TableName CreateTableStmt | ( 'FLUSH' | 'CAPTURE' | 'EVOLVE' ) 'BINDINGS' )

NumList ::=
    Int64Num ( ',' Int64Num )*
```

## 例

`ADMIN PAUSE DDL ジョブ`は現在実行中のDDLジョブを一時停止し、ジョブが正常に一時停止されたかどうかを返します。このジョブは`ADMIN RESUME DDL ジョブ`で再開できます。

```sql
ADMIN PAUSE DDL ジョブ job_id [, job_id] ...;
```

一時停止に失敗した場合、失敗の特定の理由が表示されます。

<CustomContent platform="tidb">

> **注意:**
>
> + このステートメントはDDLジョブを一時停止できますが、他の操作や環境の変更（マシンの再起動やクラスターの再起動など）は、クラスターアップグレードを除いて、DDLジョブを一時停止しません。
> + クラスターアップグレード中は、進行中のDDLジョブが一時停止され、アップグレード中に開始されたDDLジョブも一時停止されます。アップグレード後、すべての一時停止したDDLジョブが再開されます。アップグレード中の一時停止と再開の操作は自動的に行われます。詳細については、「[TiDBスムーズアップグレード](/smooth-upgrade-tidb.md)」を参照してください。
> + このステートメントは複数のDDLジョブを一時停止できます。DDLジョブの`job_id`を取得するには、[`ADMIN SHOW DDL JOBS`](/sql-statements/sql-statement-admin-show-ddl.md)ステートメントを使用できます。

</CustomContent>
<CustomContent platform="tidb-cloud">

> **注意:**
>
> + このステートメントはDDLジョブを一時停止できますが、他の操作や環境の変更（マシンの再起動やクラスターの再起動など）は、クラスターアップグレードを除いて、DDLジョブを一時停止しません。
> + クラスターアップグレード中は、進行中のDDLジョブが一時停止され、アップグレード中に開始されたDDLジョブも一時停止されます。アップグレード後、すべての一時停止したDDLジョブが再開されます。アップグレード中の一時停止と再開の操作は自動的に行われます。詳細については、「[TiDBスムーズアップグレード](https://docs.pingcap.com/tidb/stable/smooth-upgrade-tidb)」を参照してください。
> + このステートメントは複数のDDLジョブを一時停止できます。DDLジョブの`job_id`を取得するには、[`ADMIN SHOW DDL JOBS`](/sql-statements/sql-statement-admin-show-ddl.md)ステートメントを使用できます。

</CustomContent>

## MySQLの互換性

このステートメントはMySQL構文のTiDB拡張です。

## 関連情報

* [`ADMIN SHOW DDL [JOBS|QUERIES]`](/sql-statements/sql-statement-admin-show-ddl.md)
* [`ADMIN CANCEL DDL`](/sql-statements/sql-statement-admin-cancel-ddl.md)
* [`ADMIN RESUME DDL`](/sql-statements/sql-statement-admin-resume-ddl.md)