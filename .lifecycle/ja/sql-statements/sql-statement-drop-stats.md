---
title: DROP STATS
summary: TiDBデータベースでのDROP STATSの使用方法の概要。
aliases: ['/docs/dev/sql-statements/sql-statement-drop-stats/']
---

# DROP STATS

`DROP STATS`ステートメントは、選択したデータベースから選択したテーブルの統計情報を削除するために使用されます。

## 概要

```ebnf+diagram
DropStatsStmt ::=
    'DROP' 'STATS' TableNameList 

TableNameList ::=
    TableName ( ',' TableName )*

TableName ::=
    Identifier ('.' Identifier)?
```

## 例

{{< copyable "sql" >}}

```sql
CREATE TABLE t(a INT);
```

```sql
クエリ OK、0 行が変更されました (0.01 秒)
```

{{< copyable "sql" >}}

```sql
SHOW STATS_META WHERE db_name='test' and table_name='t';
```

```sql
+---------+------------+----------------+---------------------+--------------+-----------+
| Db_name | Table_name | Partition_name | Update_time         | Modify_count | Row_count |
+---------+------------+----------------+---------------------+--------------+-----------+
| test    | t          |                | 2020-05-25 20:34:33 |            0 |         0 |
+---------+------------+----------------+---------------------+--------------+-----------+
1 行が返されました (0.00 秒)
```

{{< copyable "sql" >}}

```sql
DROP STATS t;
```

```sql
クエリ OK、0 行が変更されました (0.00 秒)
```

{{< copyable "sql" >}}

```sql
SHOW STATS_META WHERE db_name='test' and table_name='t';
```

```sql
Empty set (0.00 秒)
```

## MySQL互換性

このステートメントは、TiDBにおけるMySQL構文の拡張です。

## 関連項目

* [統計情報概要](/statistics.md)