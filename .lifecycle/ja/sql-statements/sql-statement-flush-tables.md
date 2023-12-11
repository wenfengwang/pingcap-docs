---
title: FLUSH TABLES | TiDB SQLステートメントリファレンス
summary: TiDBデータベースのFLUSH TABLESの使用方法の概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-flush-tables/','/docs/dev/reference/sql/statements/flush-tables/']
---

# FLUSH TABLES

このステートメントは、MySQLとの互換性のために含まれています。TiDBでは有効な使用方法はありません。

## 概要

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
|    LogTypeOpt 'LOGS'
|    TableOrTables TableNameListOpt WithReadLockOpt

LogTypeOpt ::=
    ( 'BINARY' | 'ENGINE' | 'ERROR' | 'GENERAL' | 'SLOW' )?

TableOrTables ::=
    'TABLE'
|   'TABLES'

TableNameListOpt ::=
    TableNameList?

WithReadLockOpt ::=
    ( 'WITH' 'READ' 'LOCK' )?
```

## 例

```sql
mysql> FLUSH TABLES;
クエリが正常終了しました、0行が変更されました (0.00 秒)

mysql> FLUSH TABLES WITH READ LOCK;
ERROR 1105 (HY000): FLUSH TABLES WITH READ LOCK is not supported.  Please use @@tidb_snapshot
```

## MySQL互換性

* TiDBにはMySQLのようなテーブルキャッシュの概念はありません。そのため、互換性のためにTiDBでは`FLUSH TABLES`は解析されますが無視されます。
* `FLUSH TABLES WITH READ LOCK`ステートメントはエラーを生成します。TiDBは現在、テーブルのロックをサポートしていないためです。代わりに、この目的には[履歴データの読み取り](/read-historical-data.md)を使用することを推奨します。

## 関連項目

* [履歴データの読み取り](/read-historical-data.md)