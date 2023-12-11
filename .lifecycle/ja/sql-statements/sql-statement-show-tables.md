---
title: SHOW [FULL] TABLES | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースにおけるSHOW [FULL] TABLESの使用概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-show-tables/','/docs/dev/reference/sql/statements/show-tables/']
---

# SHOW [FULL] TABLES

このステートメントは、現在選択されているデータベース内のテーブルとビューのリストを表示します。オプションのキーワード`FULL`は、テーブルが`BASE TABLE`または`VIEW`のタイプであるかを示します。

別のデータベース内のテーブルを表示するには、`SHOW TABLES IN DatabaseName`を使用します。

## 概要

**ShowTablesStmt:**

![ShowTablesStmt](/media/sqlgram/ShowTablesStmt.png)

**OptFull:**

![OptFull](/media/sqlgram/OptFull.png)

**ShowDatabaseNameOpt:**

![ShowDatabaseNameOpt](/media/sqlgram/ShowDatabaseNameOpt.png)

**ShowLikeOrWhereOpt:**

![ShowLikeOrWhereOpt](/media/sqlgram/ShowLikeOrWhereOpt.png)

## 例

```sql
mysql> CREATE TABLE t1 (a int);
クエリが正常に完了しました。 (0.12 sec)

mysql> CREATE VIEW v1 AS SELECT 1;
クエリが正常に完了しました。 (0.10 sec)

mysql> SHOW TABLES;
+----------------+
| Tables_in_test |
+----------------+
| t1             |
| v1             |
+----------------+
2 rows in set (0.00 sec)

mysql> SHOW FULL TABLES;
+----------------+------------+
| Tables_in_test | Table_type |
+----------------+------------+
| t1             | BASE TABLE |
| v1             | VIEW       |
+----------------+------------+
2 rows in set (0.00 sec)

mysql> SHOW TABLES IN mysql;
+-------------------------+
| Tables_in_mysql         |
+-------------------------+
| GLOBAL_VARIABLES        |
| bind_info               |
| columns_priv            |
| db                      |
| default_roles           |
| expr_pushdown_blacklist |
| gc_delete_range         |
| gc_delete_range_done    |
| global_priv             |
| help_topic              |
| opt_rule_blacklist      |
| role_edges              |
| stats_buckets           |
| stats_feedback          |
| stats_histograms        |
| stats_meta              |
| stats_top_n             |
| tables_priv             |
| tidb                    |
| user                    |
+-------------------------+
20 rows in set (0.00 sec)
```

## MySQL互換性

TiDBにおける`SHOW [FULL] TABLES`ステートメントは、MySQLと完全に互換性があります。互換性に違いがある場合は、[バグを報告してください](https://docs.pingcap.com/tidb/stable/support)。

## 関連項目

* [CREATE TABLE](/sql-statements/sql-statement-create-table.md)
* [DROP TABLE](/sql-statements/sql-statement-drop-table.md)
* [SHOW CREATE TABLE](/sql-statements/sql-statement-show-create-table.md)