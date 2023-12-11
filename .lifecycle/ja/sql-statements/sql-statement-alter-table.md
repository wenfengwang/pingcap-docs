---
title: ALTER TABLE | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースのALTER TABLEの使用方法の概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-alter-table/','/docs/dev/reference/sql/statements/alter-table/']
---

# ALTER TABLE

このステートメントは、既存のテーブルを新しいテーブル構造に合わせるように変更します。`ALTER TABLE`ステートメントは、以下のように使用できます。

- インデックスを[`ADD`](/sql-statements/sql-statement-add-index.md)、[`DROP`](/sql-statements/sql-statement-drop-index.md)、または[`RENAME`](/sql-statements/sql-statement-rename-index.md)する
- カラムを[`ADD`](/sql-statements/sql-statement-add-column.md)、[`DROP`](/sql-statements/sql-statement-drop-column.md)、[`MODIFY`](/sql-statements/sql-statement-modify-column.md)、または[`CHANGE`](/sql-statements/sql-statement-change-column.md)する
- テーブルデータを[`COMPACT`](/sql-statements/sql-statement-alter-table-compact.md)する

## 概要

```ebnf+diagram
AlterTableStmt ::=
    'ALTER' IgnoreOptional 'TABLE' TableName (
        AlterTableSpecListOpt AlterTablePartitionOpt |
        'ANALYZE' 'PARTITION' PartitionNameList ( 'INDEX' IndexNameList )? AnalyzeOptionListOpt |
        'COMPACT' ( 'PARTITION' PartitionNameList )? 'TIFLASH' 'REPLICA'
    )

TableName ::=
    Identifier ('.' Identifier)?

AlterTableSpec ::=
    TableOptionList
|   'SET' 'TIFLASH' 'REPLICA' LengthNum LocationLabelList
|   'CONVERT' 'TO' CharsetKw ( CharsetName | 'DEFAULT' ) OptCollate
|   'ADD' ( ColumnKeywordOpt IfNotExists ( ColumnDef ColumnPosition | '(' TableElementList ')' ) | Constraint | 'PARTITION' IfNotExists NoWriteToBinLogAliasOpt ( PartitionDefinitionListOpt | 'PARTITIONS' NUM ) )
|   ( ( 'CHECK' | 'TRUNCATE' ) 'PARTITION' | ( 'OPTIMIZE' | 'REPAIR' | 'REBUILD' ) 'PARTITION' NoWriteToBinLogAliasOpt ) AllOrPartitionNameList
|   'COALESCE' 'PARTITION' NoWriteToBinLogAliasOpt NUM
|   'DROP' ( ColumnKeywordOpt IfExists ColumnName RestrictOrCascadeOpt | 'PRIMARY' 'KEY' |  'PARTITION' IfExists PartitionNameList | ( KeyOrIndex IfExists | 'CHECK' ) Identifier | 'FOREIGN' 'KEY' IfExists Symbol )
|   'EXCHANGE' 'PARTITION' Identifier 'WITH' 'TABLE' TableName WithValidationOpt
|   ( 'IMPORT' | 'DISCARD' ) ( 'PARTITION' AllOrPartitionNameList )? 'TABLESPACE'
|   'REORGANIZE' 'PARTITION' NoWriteToBinLogAliasOpt ReorganizePartitionRuleOpt
|   'ORDER' 'BY' AlterOrderItem ( ',' AlterOrderItem )*
|   ( 'DISABLE' | 'ENABLE' ) 'KEYS'
|   ( 'MODIFY' ColumnKeywordOpt IfExists | 'CHANGE' ColumnKeywordOpt IfExists ColumnName ) ColumnDef ColumnPosition
|   'ALTER' ( ColumnKeywordOpt ColumnName ( 'SET' 'DEFAULT' ( SignedLiteral | '(' Expression ')' ) | 'DROP' 'DEFAULT' ) | 'CHECK' Identifier EnforcedOrNot | 'INDEX' Identifier IndexInvisible )
|   'RENAME' ( ( 'COLUMN' | KeyOrIndex ) Identifier 'TO' Identifier | ( 'TO' | '='? | 'AS' ) TableName )
|   LockClause
|   AlgorithmClause
|   'FORCE'
|   ( 'WITH' | 'WITHOUT' ) 'VALIDATION'
|   'SECONDARY_LOAD'
|   'SECONDARY_UNLOAD'
|   ( 'AUTO_INCREMENT' | 'AUTO_ID_CACHE' | 'AUTO_RANDOM_BASE' | 'SHARD_ROW_ID_BITS' ) EqOpt LengthNum
|   ( 'CACHE' | 'NOCACHE' )
|   (
        'TTL' EqOpt TimeColumnName '+' 'INTERVAL' Expression TimeUnit (TTLEnable EqOpt ( 'ON' | 'OFF' ))?
        | 'REMOVE' 'TTL'
        | TTLEnable EqOpt ( 'ON' | 'OFF' )
        | TTLJobInterval EqOpt stringLit
    )
|   PlacementPolicyOption

PlacementPolicyOption ::=
    "PLACEMENT" "POLICY" EqOpt PolicyName
|   "PLACEMENT" "POLICY" (EqOpt | "SET") "DEFAULT"
```

## 例

初期データを含むテーブルを作成します。

{{< copyable "sql" >}}

```sql
CREATE TABLE t1 (id INT NOT NULL PRIMARY KEY AUTO_INCREMENT, c1 INT NOT NULL);
INSERT INTO t1 (c1) VALUES (1),(2),(3),(4),(5);
```

```sql
Query OK, 0 rows affected (0.11 sec)

Query OK, 5 rows affected (0.03 sec)
Records: 5  Duplicates: 0  Warnings: 0
```

次のクエリは、列c1がインデックスされていないため、フルテーブルスキャンが必要です。

{{< copyable "sql" >}}

```sql
EXPLAIN SELECT * FROM t1 WHERE c1 = 3;
```

```sql
+-------------------------+----------+-----------+---------------+--------------------------------+
| id                      | estRows  | task      | access object | operator info                  |
+-------------------------+----------+-----------+---------------+--------------------------------+
| TableReader_7           | 10.00    | root      |               | data:Selection_6               |
| └─Selection_6           | 10.00    | cop[tikv] |               | eq(test.t1.c1, 3)              |
|   └─TableFullScan_5     | 10000.00 | cop[tikv] | table:t1      | keep order:false, stats:pseudo |
+-------------------------+----------+-----------+---------------+--------------------------------+
3 rows in set (0.00 sec)
```

ステートメント[`ALTER TABLE .. ADD INDEX`](/sql-statements/sql-statement-add-index.md)を使用して、テーブルt1にインデックスを追加できます。 `EXPLAIN`によって、元のクエリが現在インデックス範囲スキャンを使用していることが確認され、これはより効率的です。

{{< copyable "sql" >}}

```sql
ALTER TABLE t1 ADD INDEX (c1);
EXPLAIN SELECT * FROM t1 WHERE c1 = 3;
```

```sql
Query OK, 0 rows affected (0.30 sec)

+------------------------+---------+-----------+------------------------+---------------------------------------------+
| id                     | estRows | task      | access object          | operator info                               |
+------------------------+---------+-----------+------------------------+---------------------------------------------+
| IndexReader_6          | 10.00   | root      |                        | index:IndexRangeScan_5                      |
| └─IndexRangeScan_5     | 10.00   | cop[tikv] | table:t1, index:c1(c1) | range:[3,3], keep order:false, stats:pseudo |
+------------------------+---------+-----------+------------------------+---------------------------------------------+
2 rows in set (0.00 sec)
```

TiDBは、DDLの変更が特定の`ALTER`アルゴリズムを使用すると主張する機能をサポートしています。これは単なる主張であり、テーブルを変更する際に実際に使用されるアルゴリズムを変更するものではありません。クラスタのピーク時間中にインスタントDDL変更のみを許可したい場合に役立ちます。

{{< copyable "sql" >}}

```sql
ALTER TABLE t1 DROP INDEX c1, ALGORITHM=INSTANT;
```

```sql
Query OK, 0 rows affected (0.24 sec)
```

`INPLACE`アルゴリズムを必要とする操作に`ALGORITHM=INSTANT`主張を使用すると、ステートメントエラーが発生します。

{{< copyable "sql" >}}

```sql
ALTER TABLE t1 ADD INDEX (c1), ALGORITHM=INSTANT;
```

```sql
ERROR 1846 (0A000): ALGORITHM=INSTANT is not supported. Reason: Cannot alter table by INSTANT. Try ALGORITHM=INPLACE.
```

ただし、`INPLACE`操作に対して`ALGORITHM=COPY`主張を使用すると、エラーの代わりに警告が生成されます。これは、TiDBが主張を「このアルゴリズムまたはそれ以上」と解釈するためです。この動作の違いは、TiDBが使用するアルゴリズムがMySQLと異なる可能性があるため、MySQLとの互換性のために便利です。

{{< copyable "sql" >}}

```sql
ALTER TABLE t1 ADD INDEX (c1), ALGORITHM=COPY;
SHOW WARNINGS;
```

```sql
Query OK, 0 rows affected, 1 warning (0.25 sec)

+-------+------+---------------------------------------------------------------------------------------------+
| Level | Code | Message                                                                                     |
+-------+------+---------------------------------------------------------------------------------------------+
| Error | 1846 | ALGORITHM=COPY is not supported. Reason: Cannot alter table by COPY. Try ALGORITHM=INPLACE. |
+-------+------+---------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

## MySQLの互換性

TiDBの`ALTER TABLE`には次の重要な制限が適用されます。

- 1つの`ALTER TABLE`ステートメントで複数のスキーマオブジェクトを変更する場合:
    - 複数の変更で同じオブジェクトを変更することはサポートされていません。
    - TiDBは実行前に表のスキーマに基づいてステートメントを検証します。例えば、`ALTER TABLE t ADD COLUMN c1 INT, ADD COLUMN c2 INT AFTER c1;` のようなステートメントは、表に列`c1`が存在しないため、実行時にエラーが返されます。
    - `ALTER TABLE`ステートメントでは、左から右に一つずつ変更が実行されますが、これは一部の場合においてMySQLとは互換性がないことになります。

- 主キー列の[Reorg-Data](/sql-statements/sql-statement-modify-column.md#reorg-data-change)タイプの変更はサポートされていません。

- パーティションされた表の列タイプの変更はサポートされていません。

- 生成列の列タイプの変更はサポートされていません。

- 一部のデータ型（例: TIME、Bit、Set、Enum、JSONなど）の変更は、`CAST`関数の動作の互換性の問題によりサポートされていません。

- 空間データ型はサポートされていません。

- `ALTER TABLE t CACHE | NOCACHE`はMySQL構文にTiDB拡張機能です。詳細については、[Cached Tables](/cached-tables.md)を参照してください。

その他の制限事項については、[MySQL 互換性](/mysql-compatibility.md#ddl-operations)を参照してください。

## 関連項目

- [MySQL 互換性](/mysql-compatibility.md#ddl-operations)
- [ADD COLUMN](/sql-statements/sql-statement-add-column.md)
- [DROP COLUMN](/sql-statements/sql-statement-drop-column.md)
- [ADD INDEX](/sql-statements/sql-statement-add-index.md)
- [DROP INDEX](/sql-statements/sql-statement-drop-index.md)
- [RENAME INDEX](/sql-statements/sql-statement-rename-index.md)
- [ALTER INDEX](/sql-statements/sql-statement-alter-index.md)
- [CREATE TABLE](/sql-statements/sql-statement-create-table.md)
- [DROP TABLE](/sql-statements/sql-statement-drop-table.md)
- [SHOW CREATE TABLE](/sql-statements/sql-statement-show-create-table.md)