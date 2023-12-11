---
title: DROP TABLE | TiDB SQLステートメントリファレンス
summary: TiDBデータベースのDROP TABLEの使用方法の概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-drop-table/', '/docs/dev/reference/sql/statements/drop-table/']
---

# DROP TABLE

このステートメントは、現在選択されているデータベースからテーブルを削除します。 `IF EXISTS`修飾子が使用されていない限り、テーブルが存在しない場合はエラーが返されます。

## 概要

```ebnf+diagram
DropTableStmt ::=
    'DROP' OptTemporary TableOrTables IfExists TableNameList RestrictOrCascadeOpt

OptTemporary ::=
    ( 'TEMPORARY' | ('GLOBAL' 'TEMPORARY') )?

TableOrTables ::=
    'TABLE'
|   'TABLES'

TableNameList ::=
    TableName ( ',' TableName )*
```

## 一時テーブルを削除する

次の構文を使用して通常のテーブルと一時テーブルを削除できます。

- `DROP TEMPORARY TABLE`を使用してローカル一時テーブルを削除します。
- `DROP GLOBAL TEMPORARY TABLE`を使用してグローバル一時テーブルを削除します。
- `DROP TABLE`を使用して通常のテーブルまたは一時テーブルを削除します。

## 例

```sql
mysql> CREATE TABLE t1 (a INT);
クエリがOKです、0行が影響を受けました (0.11秒)

mysql> DROP TABLE t1;
クエリがOKです、0行が影響を受けました (0.22秒)

mysql> DROP TABLE table_not_exists;
ERROR 1051 (42S02): テーブル 'test.table_not_exists' がありません

mysql> DROP TABLE IF EXISTS table_not_exists;
クエリがOKです、0行が影響を受けました、1件の警告があります (0.01秒)

mysql> SHOW WARNINGS;
+-------+------+---------------------------------------+
| Level | Code | Message                               |
+-------+------+---------------------------------------+
| Note  | 1051 | テーブル 'test.table_not_exists' がありません |
+-------+------+---------------------------------------+
1行がセットされました (0.01秒)

mysql> CREATE VIEW v1 AS SELECT 1;
クエリがOKです、0行が影響を受けました (0.10秒)

mysql> DROP TABLE v1;
クエリがOKです、0行が影響を受けました (0.23秒)
```

## MySQLの互換性

現在、`RESTRICT`および`CASCADE`は構文的にのみサポートされています。

## 関連項目

* [CREATE TABLE](/sql-statements/sql-statement-create-table.md)
* [SHOW CREATE TABLE](/sql-statements/sql-statement-show-create-table.md)
* [SHOW TABLES](/sql-statements/sql-statement-show-tables.md)