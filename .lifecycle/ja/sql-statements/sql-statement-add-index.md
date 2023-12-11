---
title: インデックスの追加 | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースでのADD INDEXの使用方法の概要。
aliases: ['/docs/dev/sql-statements/sql-statement-add-index/','/docs/dev/reference/sql/statements/add-index/']
---

# インデックスの追加

`ALTER TABLE.. ADD INDEX` ステートメントは既存のテーブルにインデックスを追加します。この操作はTiDBではオンラインで行われるため、テーブルへの読み取りまたは書き込みがインデックスの追加によってブロックされることはありません。

<CustomContent platform="tidb">

> **注意:**
>
> - クラスターでDDLステートメントが実行されている場合（通常は`ADD INDEX`や列の型変更などの時間のかかるDDLステートメントの場合）、TiDBクラスターをアップグレードしないでください。
> - アップグレードの前に、[`ADMIN SHOW DDL`](/sql-statements/sql-statement-admin-show-ddl.md)コマンドを使用してTiDBクラスターに実行中のDDLジョブがあるかどうかを確認することをお勧めします。クラスターにDDLジョブがある場合、クラスターをアップグレードするには、DDLの実行が終了するのを待つか、[`ADMIN CANCEL DDL`](/sql-statements/sql-statement-admin-cancel-ddl.md)コマンドを使用してDDLジョブをキャンセルしてからクラスターをアップグレードしてください。
> - また、クラスターアップグレード中は**いかなるDDLステートメントも実行しないでください**。それ以外の場合、未定義の動作の問題が発生する可能性があります。
>
> TiDBをv7.1.0からそれ以降のバージョンにアップグレードする場合、前述の制限を無視できます。詳細は、[TiDBスムーズアップグレードの制限事項](/smooth-upgrade-tidb.md)を参照してください。

</CustomContent>

## 構文

```ebnf+diagram
AlterTableStmt
         ::= 'ALTER' 'IGNORE'? 'TABLE' TableName AddIndexSpec ( ',' AddIndexSpec )*

AddIndexSpec
         ::= 'ADD' ( ( 'PRIMARY' 'KEY' | ( 'KEY' | 'INDEX' ) 'IF NOT EXISTS'? | 'UNIQUE' ( 'KEY' | 'INDEX' )? ) ( ( Identifier? 'USING' | Identifier 'TYPE' ) IndexType )? | 'FULLTEXT' ( 'KEY' | 'INDEX' )? IndexName ) '(' IndexPartSpecification ( ',' IndexPartSpecification )* ')' IndexOption*

IndexPartSpecification
         ::= ( ColumnName ( '(' LengthNum ')' )? | '(' Expression ')' ) ( 'ASC' | 'DESC' )

IndexOption
         ::= 'KEY_BLOCK_SIZE' '='? LengthNum
           | IndexType
           | 'WITH' 'PARSER' Identifier
           | 'COMMENT' stringLit
           | 'VISIBLE'
           | 'INVISIBLE'

IndexType
         ::= 'BTREE'
           | 'HASH'
           | 'RTREE'
```

## 例

```sql
mysql> CREATE TABLE t1 (id INT NOT NULL PRIMARY KEY AUTO_INCREMENT, c1 INT NOT NULL);
クエリ OK、0行が変更されました (0.11 秒)

mysql> INSERT INTO t1 (c1) VALUES (1),(2),(3),(4),(5);
クエリ OK、5行が変更されました (0.03 秒)
Records: 5  Duplicates: 0  警告: 0

mysql> EXPLAIN SELECT * FROM t1 WHERE c1 = 3;
+-------------------------+----------+-----------+---------------+--------------------------------+
| id                      | estRows  | task      | access object | operator info                  |
+-------------------------+----------+-----------+---------------+--------------------------------+
| TableReader_7           | 10.00    | root      |               | data:Selection_6               |
| └─Selection_6           | 10.00    | cop[tikv] |               | eq(test.t1.c1, 3)              |
|   └─TableFullScan_5     | 10000.00 | cop[tikv] | table:t1      | keep order:false, stats:pseudo |
+-------------------------+----------+-----------+---------------+--------------------------------+
3 rows in set (0.00 秒)

mysql> ALTER TABLE t1 ADD INDEX (c1);
クエリ OK、0行が変更されました (0.30 秒)

mysql> EXPLAIN SELECT * FROM t1 WHERE c1 = 3;
+------------------------+---------+-----------+------------------------+---------------------------------------------+
| id                     | estRows | task      | access object          | operator info                               |
+------------------------+---------+-----------+------------------------+---------------------------------------------+
| IndexReader_6          | 0.01    | root      |                        | index:IndexRangeScan_5                      |
| └─IndexRangeScan_5     | 0.01    | cop[tikv] | table:t1, index:c1(c1) | range:[3,3], keep order:false, stats:pseudo |
+------------------------+---------+-----------+------------------------+---------------------------------------------+
2 rows in set (0.00 秒)
```

## MySQLとの互換性

* `FULLTEXT`、`HASH`、`SPATIAL`インデックスはサポートされていません。
* 降順インデックスはサポートされていません（MySQL 5.7と同様）。
* `CLUSTERED`型の主キーをテーブルに追加することはサポートされていません。`CLUSTERED`型の主キーの詳細については、[clustered index](/clustered-indexes.md)を参照してください。

## 関連項目

* [インデックスの選択](/choose-index.md)
* [間違ったインデックスの解決方法](/wrong-index-solution.md)
* [CREATE INDEX](/sql-statements/sql-statement-create-index.md)
* [DROP INDEX](/sql-statements/sql-statement-drop-index.md)
* [RENAME INDEX](/sql-statements/sql-statement-rename-index.md)
* [ALTER INDEX](/sql-statements/sql-statement-alter-index.md)
* [ADD COLUMN](/sql-statements/sql-statement-add-column.md)
* [CREATE TABLE](/sql-statements/sql-statement-create-table.md)
* [EXPLAIN](/sql-statements/sql-statement-explain.md)