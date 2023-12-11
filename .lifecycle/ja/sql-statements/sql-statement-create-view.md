---
title: CREATE VIEW | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースでCREATE VIEWの使用法の概要。
aliases: ['/docs/dev/sql-statements/sql-statement-create-view/','/docs/dev/reference/sql/statements/create-view/']
---

# CREATE VIEW

`CREATE VIEW`ステートメントは、`SELECT`ステートメントをクエリ可能なオブジェクトとして保存し、テーブルと同様に動作します。TiDBのビューは非物質化されています。これは、ビューがクエリされる際に、TiDBが内部的にクエリを再構成してビュー定義をSQLクエリと組み合わせることを意味します。

## 概要

```ebnf+diagram
CreateViewStmt ::=
    'CREATE' OrReplace ViewAlgorithm ViewDefiner ViewSQLSecurity 'VIEW' ViewName ViewFieldList 'AS' CreateViewSelectOpt ViewCheckOption

OrReplace ::=
    ( 'OR' 'REPLACE' )?

ViewAlgorithm ::=
    ( 'ALGORITHM' '=' ( 'UNDEFINED' | 'MERGE' | 'TEMPTABLE' ) )?

ViewDefiner ::=
    ( 'DEFINER' '=' Username )?

ViewSQLSecurity ::=
    ( 'SQL' 'SECURITY' ( 'DEFINER' | 'INVOKER' ) )?

ViewName ::= TableName

ViewFieldList ::=
    ( '(' Identifier ( ',' Identifier )* ')' )?

ViewCheckOption ::=
    ( 'WITH' ( 'CASCADED' | 'LOCAL' ) 'CHECK' 'OPTION' )?
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

mysql> INSERT INTO t1 (c1) VALUES (6);
Query OK, 1 row affected (0.01 sec)

mysql> SELECT * FROM v1;
+----+----+
| id | c1 |
+----+----+
|  3 |  3 |
|  4 |  4 |
|  5 |  5 |
|  6 |  6 |
+----+----+
4 rows in set (0.00 sec)

mysql> INSERT INTO v1 (c1) VALUES (7);
ERROR 1105 (HY000): insert into view v1 is not supported now.
```

## MySQL互換性

* 現在、TiDBのビューには挿入したり更新したりすることはできません（つまり、`INSERT VIEW`、`UPDATE VIEW`はサポートされていません）。`WITH CHECK OPTION`は構文上互換性があるものの、効果はありません。
* 現在、TiDBのビューは`ALTER VIEW`をサポートしていませんが、代わりに`CREATE OR REPLACE`を使用できます。
* 現在、TiDBでは`ALGORITHM`フィールドは構文上互換性がありますが、効果はありません。TiDBは現在、MERGEアルゴリズムのみをサポートしています。

## 関連項目

* [DROP VIEW](/sql-statements/sql-statement-drop-view.md)
* [CREATE TABLE](/sql-statements/sql-statement-create-table.md)
* [SHOW CREATE TABLE](/sql-statements/sql-statement-show-create-table.md)
* [DROP TABLE](/sql-statements/sql-statement-drop-table.md)