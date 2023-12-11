---
title: SHOW INDEXES [FROM|IN] | TiDBのSQLステートメントリファレンス
summary: TiDBデータベースのSHOW INDEXES [FROM|IN]の使用方法の概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-show-indexes/','/docs/dev/reference/sql/statements/show-indexes/', '/tidb/dev/sql-statement-show-index/', '/tidb/dev/sql-statement-show-keys/']
---

# SHOW INDEXES [FROM|IN]

ステートメント`SHOW INDEXES [FROM|IN]`は指定したテーブルのインデックスを一覧表示します。ステートメント`SHOW INDEX [FROM|IN]`、`SHOW KEYS [FROM|IN]`はこのステートメントのエイリアスであり、MySQLとの互換性のために含まれています。

## 概要

**ShowIndexStmt:**

![ShowIndexStmt](/media/sqlgram/ShowIndexStmt.png)

**ShowIndexKwd:**

![ShowIndexKwd](/media/sqlgram/ShowIndexKwd.png)

**FromOrIn:**

![FromOrIn](/media/sqlgram/FromOrIn.png)

**TableName:**

![TableName](/media/sqlgram/TableName.png)

**ShowLikeOrWhereOpt:**

![ShowLikeOrWhereOpt](/media/sqlgram/ShowLikeOrWhereOpt.png)

## 例

```sql
mysql> CREATE TABLE t1 (id int not null primary key AUTO_INCREMENT, col1 INT, INDEX(col1));
Query OK, 0 rows affected (0.12 sec)

mysql> SHOW INDEXES FROM t1;
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
| テーブル | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment | Visible | Expression |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
| t1    |          0 | PRIMARY  |            1 | id          | A         |           0 |     NULL | NULL   |      | BTREE      |         |               | YES     | NULL       |
| t1    |          1 | col1     |            1 | col1        | A         |           0 |     NULL | NULL   | YES  | BTREE      |         |               | YES     | NULL       |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
2 行が返されました (0.00 秒)

mysql> SHOW INDEX FROM t1;
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
| テーブル | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment | Visible | Expression |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
| t1    |          0 | PRIMARY  |            1 | id          | A         |           0 |     NULL | NULL   |      | BTREE      |         |               | YES     | NULL       |
| t1    |          1 | col1     |            1 | col1        | A         |           0 |     NULL | NULL   | YES  | BTREE      |         |               | YES     | NULL       |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
2 行が返されました (0.00 秒)

mysql> SHOW KEYS FROM t1;
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
| テーブル | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment | Visible | Expression |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
| t1    |          0 | PRIMARY  |            1 | id          | A         |           0 |     NULL | NULL   |      | BTREE      |         |               | YES     | NULL       |
| t1    |          1 | col1     |            1 | col1        | A         |           0 |     NULL | NULL   | YES  | BTREE      |         |               | YES     | NULL       |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
2 行が返されました (0.00 秒)
```

## MySQL互換性

TiDBの`SHOW INDEXES [FROM|IN]`ステートメントはMySQLと完全に互換性があります。互換性に違いがある場合は、[バグを報告してください](https://docs.pingcap.com/tidb/stable/support)。

## 関連項目

* [SHOW CREATE TABLE](/sql-statements/sql-statement-show-create-table.md)
* [DROP INDEX](/sql-statements/sql-statement-drop-index.md)
* [CREATE INDEX](/sql-statements/sql-statement-create-index.md)