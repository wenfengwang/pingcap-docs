---
title: DROP VIEW | TiDB SQLステートメントリファレンス
summary: TiDBデータベースでのDROP VIEWの使用方法についての概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-drop-view/','/docs/dev/reference/sql/statements/drop-view/']
---

# DROP VIEW

このステートメントは、現在選択されているデータベースからビューオブジェクトを削除します。ビューが参照しているベーステーブルには影響を与えません。

## 構文

```ebnf+diagram
DropViewStmt ::=
    'DROP' 'VIEW' ( 'IF' 'EXISTS' )? TableNameList RestrictOrCascadeOpt

TableNameList ::=
    TableName ( ',' TableName )*

TableName ::=
    Identifier ('.' Identifier)?
```

## 例

```sql
mysql> CREATE TABLE t1 (id INT NOT NULL PRIMARY KEY AUTO_INCREMENT, c1 INT NOT NULL);
Query OK, 0 rows affected (0.11 sec)

mysql> INSERT INTO t1 (c1) VALUES (1),(2),(3),(4),(5);
Query OK, 5 rows affected (0.03 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> CREATE VIEW v1 AS SELECT * FROM t1 WHERE c1 > 2;
Query OK, 0 rows affected (0.11 sec)

mysql> SELECT * FROM t1;
+----+----+
| id | c1 |
+----+----+
|  1 |  1 |
|  2 |  2 |
|  3 |  3 |
|  4 |  4 |
|  5 |  5 |
+----+----+
5 rows in set (0.00 sec)

mysql> SELECT * FROM v1;
+----+----+
| id | c1 |
+----+----+
|  3 |  3 |
|  4 |  4 |
|  5 |  5 |
+----+----+
3 rows in set (0.00 sec)

mysql> DROP VIEW v1;
Query OK, 0 rows affected (0.23 sec)

mysql> SELECT * FROM t1;
+----+----+
| id | c1 |
+----+----+
|  1 |  1 |
|  2 |  2 |
|  3 |  3 |
|  4 |  4 |
|  5 |  5 |
+----+----+
5 rows in set (0.00 sec)
```

## MySQL互換性

TiDBの`DROP VIEW`ステートメントはMySQLと完全に互換性があります。互換性に違いがある場合は、[バグを報告](https://docs.pingcap.com/tidb/stable/support)してください。

## 関連情報

* [CREATE VIEW](/sql-statements/sql-statement-create-view.md)
* [DROP TABLE](/sql-statements/sql-statement-drop-table.md)