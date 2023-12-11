---
title: ADD COLUMN | TiDB SQLステートメントリファレンス
summary: TiDBデータベースのADD COLUMNの使用方法の概要。
aliases: ['/docs/dev/sql-statements/sql-statement-add-column/','/docs/dev/reference/sql/statements/add-column/']
---

# ADD COLUMN

`ALTER TABLE.. ADD COLUMN`ステートメントは、既存のテーブルに列を追加します。この操作はTiDBのオンライン動作であり、つまり、テーブルへの読み取りや書き込みが列の追加によってブロックされることはありません。

## 概要

```ebnf+diagram
AlterTableStmt
         ::= 'ALTER' 'IGNORE'? 'TABLE' TableName AddColumnSpec ( ',' AddColumnSpec )*

TableName ::=
    Identifier ('.' Identifier)?

AddColumnSpec
         ::= 'ADD' 'COLUMN' 'IF NOT EXISTS'? ColumnName ColumnType ColumnOption+ ( 'FIRST' | 'AFTER' ColumnName )?

ColumnType
         ::= NumericType
           | StringType
           | DateAndTimeType
           | 'SERIAL'

ColumnOption
         ::= 'NOT'? 'NULL'
           | 'AUTO_INCREMENT'
           | 'PRIMARY'? 'KEY' ( 'CLUSTERED' | 'NONCLUSTERED' )?
           | 'UNIQUE' 'KEY'?
           | 'DEFAULT' ( NowSymOptionFraction | SignedLiteral | NextValueForSequence )
           | 'SERIAL' 'DEFAULT' 'VALUE'
           | 'ON' 'UPDATE' NowSymOptionFraction
           | 'COMMENT' stringLit
           | ( 'CONSTRAINT' Identifier? )? 'CHECK' '(' Expression ')' ( 'NOT'? ( 'ENFORCED' | 'NULL' ) )?
           | 'GENERATED' 'ALWAYS' 'AS' '(' Expression ')' ( 'VIRTUAL' | 'STORED' )?
           | 'REFERENCES' TableName ( '(' IndexPartSpecificationList ')' )? Match? OnDeleteUpdateOpt
           | 'COLLATE' CollationName
           | 'COLUMN_FORMAT' ColumnFormat
           | 'STORAGE' StorageMedia
           | 'AUTO_RANDOM' ( '(' LengthNum ')' )?
```

## 例

```sql
mysql> CREATE TABLE t1 (id INT NOT NULL PRIMARY KEY AUTO_INCREMENT);
クエリ OK, 0行が影響を受けました (0.11 sec)

mysql> INSERT INTO t1 VALUES (NULL);
クエリ OK, 1行が影響を受けました (0.02 sec)

mysql> SELECT * FROM t1;
+----+
| id |
+----+
|  1 |
+----+
1 行が返されました (0.00 sec)

mysql> ALTER TABLE t1 ADD COLUMN c1 INT NOT NULL;
クエリ OK, 0行が影響を受けました (0.28 sec)

mysql> SELECT * FROM t1;
+----+----+
| id | c1 |
+----+----+
|  1 |  0 |
+----+----+
1 行が返されました (0.00 sec)

mysql> ALTER TABLE t1 ADD c2 INT NOT NULL AFTER c1;
クエリ OK, 0行が影響を受けました (0.28 sec)

mysql> SELECT * FROM t1;
+----+----+----+
| id | c1 | c2 |
+----+----+----+
|  1 |  0 |  0 |
+----+----+----+
1 行が返されました (0.00 sec)
```

## MySQL互換性

* 新しい列を追加してそれを`PRIMARY KEY`に設定することはサポートされていません。
* 新しい列を追加してそれを`AUTO_INCREMENT`に設定することはサポートされていません。
* 生成された列を追加する際にはいくつかの制限があります。[生成列の制限](/generated-columns.md#limitations)を参照してください。

## 関連情報

* [ADD INDEX](/sql-statements/sql-statement-add-index.md)
* [CREATE TABLE](/sql-statements/sql-statement-create-table.md)