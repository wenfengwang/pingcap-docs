---
title: DROP COLUMN | TiDB SQLステートメントリファレンス
summary: TiDBデータベースのDROP COLUMNの使用方法の概要。
aliases: ['/docs/dev/sql-statements/sql-statement-drop-column/', '/docs/dev/reference/sql/statements/drop-column/']
---

# DROP COLUMN

このステートメントは、指定したテーブルから列を削除します。`DROP COLUMN`はTiDBでオンライン使用可能であり、これはつまり読み取りまたは書き込み操作をブロックしないことを意味します。

## 概要

```ebnf+diagram
AlterTableStmt
         ::= 'ALTER' 'IGNORE'? 'TABLE' TableName DropColumnSpec ( ',' DropColumnSpec )*

DropColumnSpec
         ::= 'DROP' 'COLUMN'? 'IF EXISTS'? ColumnName ( 'RESTRICT' | 'CASCADE' )?

ColumnName
         ::= Identifier ( '.' Identifier ( '.' Identifier )? )?
```

## 例

```sql
mysql> CREATE TABLE t1 (id INT NOT NULL PRIMARY KEY AUTO_INCREMENT, col1 INT NOT NULL, col2 INT NOT NULL);
クエリが成功しました。影響を受けた行: 0 (0.12 sec)

mysql> INSERT INTO t1 (col1, col2) VALUES (1, 1), (2, 2), (3, 3), (4, 4), (5, 5);
クエリが成功しました。影響を受けた行: 5 (0.02 sec)
レコード: 5  重複: 0  警告: 0

mysql> SELECT * FROM t1;
+----+------+------+
| id | col1 | col2 |
+----+------+------+
|  1 |    1 |    1 |
|  2 |    2 |    2 |
|  3 |    3 |    3 |
|  4 |    4 |    4 |
|  5 |    5 |    5 |
+----+------+------+
5 行を表示しました (0.01 秒)

mysql> ALTER TABLE t1 DROP COLUMN col1, DROP COLUMN col2;
ERROR 1105 (HY000): can't run multi schema change
mysql> SELECT * FROM t1;
+----+------+------+
| id | col1 | col2 |
+----+------+------+
|  1 |    1 |    1 |
|  2 |    2 |    2 |
|  3 |    3 |    3 |
|  4 |    4 |    4 |
|  5 |    5 |    5 |
+----+------+------+
5 行を表示しました (0.00 秒)

mysql> ALTER TABLE t1 DROP COLUMN col1;
クエリが成功しました。影響を受けた行: 0 (0.27 秒)

mysql> SELECT * FROM t1;
+----+------+
| id | col2 |
+----+------+
|  1 |    1 |
|  2 |    2 |
|  3 |    3 |
|  4 |    4 |
|  5 |    5 |
+----+------+
5 行を表示しました (0.00 秒)
```

## MySQL互換性

* プライマリキーの列や複合インデックスでカバーされている列を削除することはできません。

## 関連項目

* [ADD COLUMN](/sql-statements/sql-statement-add-column.md)
* [SHOW CREATE TABLE](/sql-statements/sql-statement-show-create-table.md)
* [CREATE TABLE](/sql-statements/sql-statement-create-table.md)