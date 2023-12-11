---
title: SHOW STATS_META
summary: TiDBデータベースのSHOW STATS_METAの使用方法の概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-show-stats-meta/']
---

# SHOW STATS_META

`SHOW STATS_META`を使用すると、テーブル内の行数とそのテーブルで変更された行数を表示できます。このステートメントを使用する際は、`ShowLikeOrWhere`句で必要な情報をフィルタリングできます。

現在、`SHOW STATS_META`ステートメントは6つの列を出力します。

| 列名 | 説明            |
| -------- | ------------- |
| db_name  |  データベース名    |
| table_name | テーブル名 |
| partition_name| パーティション名 |
| update_time | 最終更新時間 |
| modify_count | 変更された行数 |
| row_count | 総行数 |

> **注意:**
>
> `update_time`はTiDBが`MODIFY_COUNT`および`ROW_COUNT`フィールドをDMLステートメントに基づいて更新するときに更新されます。そのため`update_time`は`ANALYZE`ステートメントの最終実行時間ではありません。

## 概要

**ShowStmt**

![ShowStmt](/media/sqlgram/ShowStmt.png)

**ShowTargetFiltertable**

![ShowTargetFilterable](/media/sqlgram/ShowTargetFilterable.png)

**ShowLikeOrWhereOpt**

![ShowLikeOrWhereOpt](/media/sqlgram/ShowLikeOrWhereOpt.png)

## 例

{{< copyable "sql" >}}

```sql
show stats_meta;
```

```sql
+---------+------------+----------------+---------------------+--------------+-----------+
| Db_name | Table_name | Partition_name | Update_time         | Modify_count | Row_count |
+---------+------------+----------------+---------------------+--------------+-----------+
| test    | t0         |                | 2020-05-15 16:58:00 |            0 |         0 |
| test    | t1         |                | 2020-05-15 16:58:04 |            0 |         0 |
| test    | t2         |                | 2020-05-15 16:58:11 |            0 |         0 |
| test    | s          |                | 2020-05-22 19:46:43 |            0 |         0 |
| test    | t          |                | 2020-05-25 12:04:21 |            0 |         0 |
+---------+------------+----------------+---------------------+--------------+-----------+
5 rows in set (0.00 sec)
```

{{< copyable "sql" >}}

```sql
show stats_meta where table_name = 't2';
```

```sql
+---------+------------+----------------+---------------------+--------------+-----------+
| Db_name | Table_name | Partition_name | Update_time         | Modify_count | Row_count |
+---------+------------+----------------+---------------------+--------------+-----------+
| test    | t2         |                | 2020-05-15 16:58:11 |            0 |         0 |
+---------+------------+----------------+---------------------+--------------+-----------+
1 row in set (0.00 sec)
```

## MySQL互換性

このステートメントはMySQLの構文に対するTiDBの拡張です。

## 関連項目

* [ANALYZE](/sql-statements/sql-statement-analyze-table.md)
* [統計情報の紹介](/statistics.md)