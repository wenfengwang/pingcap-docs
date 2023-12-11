---
title: 使用 | TiDB SQL 文字リファレンス
summary: TiDBデータベースのUSEの使用法の概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-use/','/docs/dev/reference/sql/statements/use/']
---

# USE

`USE` ステートメントは、ユーザーセッションのための現在のデータベースを選択します。

## 概要

**UseStmt:**

![UseStmt](/media/sqlgram/UseStmt.png)

**DBName:**

![DBName](/media/sqlgram/DBName.png)

## 例

```sql
mysql> USE mysql;
テーブルとカラム名の補完のためのテーブル情報を読み込む
この機能を無効にして、-Aでより速い起動を得ることができます

Database changed
mysql> SHOW TABLES;
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
20 rows in set (0.01 sec)

mysql> CREATE DATABASE newtest;
クエリが実行されました: 0 行が影響しました (0.10 秒)

mysql> USE newtest;
Database changed
mysql> SHOW TABLES;
Empty set (0.00 sec)

mysql> CREATE TABLE t1 (a int);
クエリが実行されました: 0 行が影響しました (0.10 秒)

mysql> SHOW TABLES;
+-------------------+
| Tables_in_newtest |
+-------------------+
| t1                |
+-------------------+
1 row in set (0.00 sec)
```

## MySQL 互換性

TiDBの`USE`ステートメントはMySQLと完全互換です。互換性の違いを見つけた場合は、[バグを報告してください](https://docs.pingcap.com/tidb/stable/support)。

## 関連項目

* [CREATE DATABASE](/sql-statements/sql-statement-create-database.md)
* [SHOW TABLES](/sql-statements/sql-statement-show-tables.md)