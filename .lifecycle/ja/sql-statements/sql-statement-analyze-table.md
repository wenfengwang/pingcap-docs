---
title: ANALYZE | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースのANALYZEの使用方法の概要。
aliases: ['/docs/dev/sql-statements/sql-statement-analyze-table/','/docs/dev/reference/sql/statements/analyze-table/']
---

# ANALYZE

このステートメントは、TiDBがテーブルやインデックスに構築する統計情報を更新します。大規模なバッチ更新やレコードのインポートを実行した後、またはクエリ実行計画が最適化されていないことに気付いた時に、`ANALYZE`を実行することを推奨します。

TiDBは、自身の推定値と一貫性が取れていないことを発見すると、統計情報も自動的に更新します。

現在、TiDBは`ANALYZE TABLE`ステートメントを使用して統計情報を完全に収集します。詳細は、[統計情報の紹介](/statistics.md)を参照してください。

## 構文

```ebnf+diagram
AnalyzeTableStmt ::=
    'ANALYZE' ( 'TABLE' ( TableNameList ( 'ALL COLUMNS' | 'PREDICATE COLUMNS' ) | TableName ( 'INDEX' IndexNameList? | AnalyzeColumnOption | 'PARTITION' PartitionNameList ( 'INDEX' IndexNameList? | AnalyzeColumnOption )? )? ) | 'INCREMENTAL' 'TABLE' TableName ( 'PARTITION' PartitionNameList )? 'INDEX' IndexNameList? ) AnalyzeOptionListOpt

AnalyzeOptionListOpt ::=
( WITH AnalyzeOptionList )?

AnalyzeOptionList ::=
AnalyzeOption ( ',' AnalyzeOption )*

AnalyzeOption ::=
( NUM ( 'BUCKETS' | 'TOPN' | ( 'CMSKETCH' ( 'DEPTH' | 'WIDTH' ) ) | 'SAMPLES' ) ) | ( FLOATNUM 'SAMPLERATE' )

AnalyzeColumnOption ::=
( 'ALL COLUMNS' | 'PREDICATE COLUMNS' | 'COLUMNS' ColumnNameList )

TableNameList ::=
    TableName (',' TableName)*

TableName ::=
    Identifier ( '.' Identifier )?

ColumnNameList ::=
    Identifier ( ',' Identifier )*

IndexNameList ::=
    Identifier ( ',' Identifier )*

PartitionNameList ::=
    Identifier ( ',' Identifier )*
```

## 例

```sql
mysql> CREATE TABLE t1 (id INT NOT NULL PRIMARY KEY AUTO_INCREMENT, c1 INT NOT NULL);
Query OK, 0 rows affected (0.11 sec)
```

```sql
mysql> INSERT INTO t1 (c1) VALUES (1),(2),(3),(4),(5);
Query OK, 5 rows affected (0.03 sec)
Records: 5  Duplicates: 0  Warnings: 0
```

```sql
mysql> ALTER TABLE t1 ADD INDEX (c1);
Query OK, 0 rows affected (0.30 sec)
```

```sql
mysql> EXPLAIN SELECT * FROM t1 WHERE c1 = 3;
+------------------------+---------+-----------+------------------------+---------------------------------------------+
| id                     | estRows | task      | access object          | operator info                               |
+------------------------+---------+-----------+------------------------+---------------------------------------------+
| IndexReader_6          | 10.00   | root      |                        | index:IndexRangeScan_5                      |
| └─IndexRangeScan_5     | 10.00   | cop[tikv] | table:t1, index:c1(c1) | range:[3,3], keep order:false, stats:pseudo |
+------------------------+---------+-----------+------------------------+---------------------------------------------+
2 rows in set (0.00 sec)
```

現在の統計情報の状態は`pseudo`であり、統計情報が不正確であることを示しています。

```sql
mysql> ANALYZE TABLE t1;
Query OK, 0 rows affected (0.13 sec)

mysql> EXPLAIN SELECT * FROM t1 WHERE c1 = 3;
+------------------------+---------+-----------+------------------------+-------------------------------+
| id                     | estRows | task      | access object          | operator info                 |
+------------------------+---------+-----------+------------------------+-------------------------------+
| IndexReader_6          | 1.00    | root      |                        | index:IndexRangeScan_5        |
| └─IndexRangeScan_5     | 1.00    | cop[tikv] | table:t1, index:c1(c1) | range:[3,3], keep order:false |
+------------------------+---------+-----------+------------------------+-------------------------------+
2 rows in set (0.00 sec)
```

統計情報は正しく更新され、ロードされました。

## MySQL互換性

TiDBは、クエリ実行中に統計情報をどのように収集し、利用するかについて、MySQLとは異なる点があります。このステートメントは、構文的にはMySQLに類似していますが、以下のような違いがあります。

+ TiDBは`ANALYZE TABLE`を実行する際、最近コミットされた変更を含めない場合があります。行のバッチ更新後、統計情報更新がこれらの変更を反映するようにするには、`ANALYZE TABLE`を実行する前に`sleep(1)`する必要があります。[#16570](https://github.com/pingcap/tidb/issues/16570)を参照してください。
+ `ANALYZE TABLE`は、TiDBでMySQLよりも実行に時間がかかる可能性があります。

## 関連項目

* [EXPLAIN](/sql-statements/sql-statement-explain.md)
* [EXPLAIN ANALYZE](/sql-statements/sql-statement-explain-analyze.md)