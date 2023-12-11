---
title: SHOW STATS_HISTOGRAMS
summary: TiDBデータベースのSHOW HISTOGRAMSの使用概要。
aliases: ['/docs/dev/sql-statements/sql-statement-show-histograms/']
---

# SHOW STATS_HISTOGRAMS

この文は、`ANALYZE`文によって収集されたヒストグラム情報を示します。

## 概要

**ShowStmt**

![ShowStmt](/media/sqlgram/ShowStmt.png)

**ShowTargetFiltertable**

![ShowTargetFiltertable](/media/sqlgram/ShowTargetFiltertable.png)

**ShowLikeOrWhereOpt**

![ShowLikeOrWhereOpt](/media/sqlgram/ShowLikeOrWhereOpt.png)

## 例

{{< copyable "sql" >}}

```sql
show stats_histograms;
```

```sql
+---------+------------+----------------+-------------+----------+---------------------+----------------+------------+--------------+-------------+
| Db_name | Table_name | Partition_name | Column_name | Is_index | Update_time         | Distinct_count | Null_count | Avg_col_size | Correlation |
+---------+------------+----------------+-------------+----------+---------------------+----------------+------------+--------------+-------------+
| test    | t          |                | a           |        0 | 2020-05-25 19:20:00 |              7 |          0 |            1 |           1 |
| test    | t2         |                | a           |        0 | 2020-05-25 19:20:01 |              6 |          0 |            8 |           0 |
| test    | t2         |                | b           |        0 | 2020-05-25 19:20:01 |              6 |          0 |         1.67 |           1 |
+---------+------------+----------------+-------------+----------+---------------------+----------------+------------+--------------+-------------+
3 rows in set (0.00 sec)
```

{{< copyable "sql" >}}

```sql
show stats_histograms where table_name = 't2';
```

```sql
+---------+------------+----------------+-------------+----------+---------------------+----------------+------------+--------------+-------------+
| Db_name | Table_name | Partition_name | Column_name | Is_index | Update_time         | Distinct_count | Null_count | Avg_col_size | Correlation |
+---------+------------+----------------+-------------+----------+---------------------+----------------+------------+--------------+-------------+
| test    | t2         |                | b           |        0 | 2020-05-25 19:20:01 |              6 |          0 |         1.67 |           1 |
| test    | t2         |                | a           |        0 | 2020-05-25 19:20:01 |              6 |          0 |            8 |           0 |
+---------+------------+----------------+-------------+----------+---------------------+----------------+------------+--------------+-------------+
2 rows in set (0.00 sec)
```

## MySQL 互換性

この文は、TiDBのMySQL構文への拡張です。

## 関連情報

* [ANALYZE](/sql-statements/sql-statement-analyze-table.md)
* [統計の概要](/statistics.md)