```
---
title: FLUSH PRIVILEGES | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースのFLUSH PRIVILEGESの使用法についての概要。
aliases: ['/docs/dev/sql-statements/sql-statement-flush-privileges/','/docs/dev/reference/sql/statements/flush-privileges/']
---

# FLUSH PRIVILEGES

ステートメント`FLUSH PRIVILEGES`は、TiDBに対して特権テーブルからの特権のインメモリコピーを再読み込みするよう指示します。このステートメントは、`mysql.user`のようなテーブルを手動で編集した後に実行する必要があります。ただし、`GRANT`や`REVOKE`のような特権ステートメントを使用した後には、このステートメントを実行する必要はありません。このステートメントを実行するには、`RELOAD`特権が必要です。

## 構文

```ebnf+diagram
FlushStmt ::=
    'FLUSH' NoWriteToBinLogAliasOpt FlushOption

NoWriteToBinLogAliasOpt ::=
    ( 'NO_WRITE_TO_BINLOG' | 'LOCAL' )?

FlushOption ::=
    'PRIVILEGES'
|   'STATUS'
|    'TIDB' 'PLUGINS' PluginNameList
|    'HOSTS'
|   LogTypeOpt 'LOGS'
|   TableOrTables TableNameListOpt WithReadLockOpt
```

## 例

```sql
mysql> FLUSH PRIVILEGES;
クエリが実行されました, 0 行が変更されました (0.01 秒)
```

## MySQL互換性

TiDBの`FLUSH PRIVILEGES`ステートメントは、MySQLと完全に互換性があります。互換性に違いがある場合は、[バグを報告してください](https://docs.pingcap.com/tidb/stable/support)。

## 関連項目

* [SHOW GRANTS](/sql-statements/sql-statement-show-grants.md)

<CustomContent platform="tidb">

* [特権管理](/privilege-management.md)

</CustomContent>
```