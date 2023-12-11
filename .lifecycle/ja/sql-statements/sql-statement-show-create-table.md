---
title: SHOW CREATE TABLE | TiDB SQLステートメントリファレンス
summary: TiDBデータベースのSHOW CREATE TABLEの使用方法の概要。
aliases: ['/docs/dev/sql-statements/sql-statement-show-create-table/','/docs/dev/reference/sql/statements/show-create-table/']
---

# SHOW CREATE TABLE

このステートメントは、既存のテーブルを再作成するための正確なステートメントをSQLで表示します。

## 概要

**ShowCreateTableStmt:**

![ShowCreateTableStmt](/media/sqlgram/ShowCreateTableStmt.png)

**TableName:**

![TableName](/media/sqlgram/TableName.png)

## 例

```sql
mysql> CREATE TABLE t1 (a INT);
Query OK, 0 rows affected (0.12 sec)

mysql> SHOW CREATE TABLE t1;
+-------+------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                               |
+-------+------------------------------------------------------------------------------------------------------------+
| t1    | CREATE TABLE `t1` (
  `a` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin |
+-------+------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

## MySQL互換性

TiDBの`SHOW CREATE TABLE`ステートメントはMySQLと完全に互換性があります。互換性に違いがある場合は、[バグを報告してください](https://docs.pingcap.com/tidb/stable/support)。

## 関連項目

* [CREATE TABLE](/sql-statements/sql-statement-create-table.md)
* [DROP TABLE](/sql-statements/sql-statement-drop-table.md)
* [SHOW TABLES](/sql-statements/sql-statement-show-tables.md)
* [SHOW COLUMNS FROM](/sql-statements/sql-statement-show-columns-from.md)