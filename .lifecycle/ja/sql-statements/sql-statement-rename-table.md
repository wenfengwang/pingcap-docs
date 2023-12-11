---
title: RENAME TABLE | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースのRENAME TABLEの使用法の概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-rename-table/', '/docs/dev/reference/sql/statements/rename-table/']
---

# RENAME TABLE

このステートメントは、既存のテーブルを新しい名前に変更します。

## 構文

```ebnf+diagram
RenameTableStmt ::=
    'RENAME' 'TABLE' TableToTable ( ',' TableToTable )*

TableToTable ::=
    TableName 'TO' TableName
```

## 例

```sql
mysql> CREATE TABLE t1 (a int);
Query OK, 0 rows affected (0.12 sec)

mysql> SHOW TABLES;
+----------------+
| Tables_in_test |
+----------------+
| t1             |
+----------------+
1 row in set (0.00 sec)

mysql> RENAME TABLE t1 TO t2;
Query OK, 0 rows affected (0.08 sec)

mysql> SHOW TABLES;
+----------------+
| Tables_in_test |
+----------------+
| t2             |
+----------------+
1 row in set (0.00 sec)
```

## MySQL互換性

TiDBの`RENAME TABLE`ステートメントはMySQLと完全に互換性があります。互換性に違いがある場合は、[バグを報告してください](https://docs.pingcap.com/tidb/stable/support)。

## 関連項目

* [CREATE TABLE](/sql-statements/sql-statement-create-table.md)
* [SHOW TABLES](/sql-statements/sql-statement-show-tables.md)
* [ALTER TABLE](/sql-statements/sql-statement-alter-table.md)