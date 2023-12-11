---
title: ADMIN CHECKSUM TABLE | TiDB SQLステートメントリファレンス
summary: TiDBデータベースのADMINの使用法の概要です。
category: リファレンス
---

# ADMIN CHECKSUM TABLE

`ADMIN CHECKSUM TABLE`ステートメントは、テーブルのデータおよびインデックスのCRC64チェックサムを計算します。このステートメントは、TiDB Lightningなどのプログラムで、インポート操作が正常に完了したことを確認するために使用されます。

## 概要

```ebnf+diagram
AdminStmt ::=
    'ADMIN' ( 'SHOW' ( 'DDL' ( 'JOBS' Int64Num? WhereClauseOptional | 'JOB' 'QUERIES' NumList )? | TableName 'NEXT_ROW_ID' | 'SLOW' AdminShowSlow ) | 'CHECK' ( 'TABLE' TableNameList | 'INDEX' TableName Identifier ( HandleRange ( ',' HandleRange )* )? ) | 'RECOVER' 'INDEX' TableName Identifier | 'CLEANUP' ( 'INDEX' TableName Identifier | 'TABLE' 'LOCK' TableNameList ) | 'CHECKSUM' 'TABLE' TableNameList | 'CANCEL' 'DDL' 'JOBS' NumList | 'RELOAD' ( 'EXPR_PUSHDOWN_BLACKLIST' | 'OPT_RULE_BLACKLIST' | 'BINDINGS' ) | 'PLUGINS' ( 'ENABLE' | 'DISABLE' ) PluginNameList | 'REPAIR' 'TABLE' TableName CreateTableStmt | ( 'FLUSH' | 'CAPTURE' | 'EVOLVE' ) 'BINDINGS' )

TableNameList ::=
    TableName ( ',' TableName )*
```

## 例

`t1`テーブルを作成する：

```sql
CREATE TABLE t1(id INT PRIMARY KEY);
```

`t1`にデータを挿入する：

```sql
INSERT INTO t1 VALUES (1),(2),(3);
```

`t1`のチェックサムを計算する：

```sql
ADMIN CHECKSUM TABLE t1;
```

出力は次のようになります：

```sql
+---------+------------+----------------------+-----------+-------------+
| Db_name | Table_name | Checksum_crc64_xor   | Total_kvs | Total_bytes |
+---------+------------+----------------------+-----------+-------------+
| test    | t1         | 10909174369497628533 |         3 |          75 |
+---------+------------+----------------------+-----------+-------------+
1 row in set (0.00 sec)
```

## MySQL互換性

このステートメントは、TiDB構文のMySQL拡張です。