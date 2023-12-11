---
title: Partition Pruning
summary: TiDBのパーティションプルーニングの使用シナリオについて学びます。

# パーティションプルーニング

パーティションプルーニングは、パーティションされたテーブルに適用されるパフォーマンスの最適化です。クエリ文のフィルタ条件を分析し、必要なデータが含まれていないパーティションを取り除く（_プルーニング_）ことで、TiDBはアクセスする必要のあるデータ量を減らし、クエリの実行時間を大幅に改善することができます。

以下は、例です：

{{< copyable "sql" >}}

```sql
CREATE TABLE t1 (
 id INT NOT NULL PRIMARY KEY,
 pad VARCHAR(100)
)
PARTITION BY RANGE COLUMNS(id) (
 PARTITION p0 VALUES LESS THAN (100),
 PARTITION p1 VALUES LESS THAN (200),
 PARTITION p2 VALUES LESS THAN (MAXVALUE)
);

INSERT INTO t1 VALUES (1, 'test1'),(101, 'test2'), (201, 'test3');
EXPLAIN SELECT * FROM t1 WHERE id BETWEEN 80 AND 120;
```

```sql
+----------------------------+---------+-----------+------------------------+------------------------------------------------+
| id                         | estRows | task      | access object          | operator info                                  |
+----------------------------+---------+-----------+------------------------+------------------------------------------------+
| PartitionUnion_8           | 80.00   | root      |                        |                                                |
| ├─TableReader_10           | 40.00   | root      |                        | data:TableRangeScan_9                          |
| │ └─TableRangeScan_9       | 40.00   | cop[tikv] | table:t1, partition:p0 | range:[80,120], keep order:false, stats:pseudo |
| └─TableReader_12           | 40.00   | root      |                        | data:TableRangeScan_11                         |
|   └─TableRangeScan_11      | 40.00   | cop[tikv] | table:t1, partition:p1 | range:[80,120], keep order:false, stats:pseudo |
+----------------------------+---------+-----------+------------------------+------------------------------------------------+
5 rows in set (0.00 sec)
```

## パーティションプルーニングの使用シナリオ

パーティションプルーニングの使用シナリオは、レンジパーティションテーブルとハッシュパーティションテーブルの2種類によって異なります。

### ハッシュパーティションテーブルでのパーティションプルーニングの使用

このセクションでは、ハッシュパーティションテーブルにおけるパーティションプルーニングの適用可能なおよび適用不可能な使用シナリオについて説明します。

#### ハッシュパーティションテーブルにおける適用可能なシナリオ

ハッシュパーティションテーブルでのパーティションプルーニングは、等しい比較のクエリ条件にのみ適用されます。

{{< copyable "sql" >}}

```sql
create table t (x int) partition by hash(x) partitions 4;
explain select * from t where x = 1;
```

```sql
+-------------------------+----------+-----------+-----------------------+--------------------------------+
| id                      | estRows  | task      | access object         | operator info                  |
+-------------------------+----------+-----------+-----------------------+--------------------------------+
| TableReader_8           | 10.00    | root      |                       | data:Selection_7               |
| └─Selection_7           | 10.00    | cop[tikv] |                       | eq(test.t.x, 1)                |
|   └─TableFullScan_6     | 10000.00 | cop[tikv] | table:t, partition:p1 | keep order:false, stats:pseudo |
+-------------------------+----------+-----------+-----------------------+--------------------------------+
```

上記のSQL文から、条件`x = 1`によってすべての結果が1つのパーティションに含まれることが分かります。ハッシュパーティションを通過した後、値`1`が`p1`パーティションにあることが確認されます。したがって、スキャンする必要があるのは`p2`、`p3`、`p4`パーティションではないため、実行計画からパーティションプルーニングが有効であることが確認できます。

#### ハッシュパーティションテーブルにおける適用不可能なシナリオ

このセクションでは、ハッシュパーティションテーブルにおけるパーティションプルーニングの適用不可能な使用シナリオについて説明します。

##### シナリオ1

（`in`、`between`、`>`、`<`、`>=`、`<=`などの）クエリ結果が1つのパーティションに落ちる条件が確認できない場合、パーティションプルーニングの最適化を使用することはできません。例：

{{< copyable "sql" >}}

```sql
create table t (x int) partition by hash(x) partitions 4;
explain select * from t where x > 2;
```

```sql
+------------------------------+----------+-----------+-----------------------+--------------------------------+
| id                           | estRows  | task      | access object         | operator info                  |
+------------------------------+----------+-----------+-----------------------+--------------------------------+
| Union_10                     | 13333.33 | root      |                       |                                |
| ├─TableReader_13             | 3333.33  | root      |                       | data:Selection_12              |
| │ └─Selection_12             | 3333.33  | cop[tikv] |                       | gt(test.t.x, 2)                |
| │   └─TableFullScan_11       | 10000.00 | cop[tikv] | table:t, partition:p0 | keep order:false, stats:pseudo |
| ├─TableReader_16             | 3333.33  | root      |                       | data:Selection_15              |
| │ └─Selection_15             | 3333.33  | cop[tikv] |                       | gt(test.t.x, 2)                |
| │   └─TableFullScan_14       | 10000.00 | cop[tikv] | table:t, partition:p1 | keep order:false, stats:pseudo |
| ├─TableReader_19             | 3333.33  | root      |                       | data:Selection_18              |
| │ └─Selection_18             | 3333.33  | cop[tikv] |                       | gt(test.t.x, 2)                |
| │   └─TableFullScan_17       | 10000.00 | cop[tikv] | table:t, partition:p2 | keep order:false, stats:pseudo |
| └─TableReader_22             | 3333.33  | root      |                       | data:Selection_21              |
|   └─Selection_21             | 3333.33  | cop[tikv] |                       | gt(test.t.x, 2)                |
|     └─TableFullScan_20       | 10000.00 | cop[tikv] | table:t, partition:p3 | keep order:false, stats:pseudo |
+------------------------------+----------+-----------+-----------------------+--------------------------------+
```

この場合、条件`x > 2`によって対応するハッシュパーティションが確認できないため、パーティションプルーニングは適用不可能です。

##### シナリオ2

クエリプランの生成段階でのフィルタ条件が実行段階でのみ取得可能なシナリオに対しては、パーティションプルーニングのルール最適化が実行されるため、パーティションプルーニングは適していません。例：

{{< copyable "sql" >}}

```sql
create table t (x int) partition by hash(x) partitions 4;
explain select * from t2 where x = (select * from t1 where t2.x = t1.x and t2.x < 2);
```

```sql
+--------------------------------------+----------+-----------+------------------------+----------------------------------------------+
| id                                   | estRows  | task      | access object          | operator info                                |
+--------------------------------------+----------+-----------+------------------------+----------------------------------------------+
| Projection_13                        | 9990.00  | root      |                        | test.t2.x                                    |
| └─Apply_15                           | 9990.00  | root      |                        | inner join, equal:[eq(test.t2.x, test.t1.x)] |
|   ├─TableReader_18(Build)            | 9990.00  | root      |                        | data:Selection_17                            |
|   │ └─Selection_17                   | 9990.00  | cop[tikv] |                        | not(isnull(test.t2.x))                       |
|   │   └─TableFullScan_16             | 10000.00 | cop[tikv] | table:t2               | keep order:false, stats:pseudo               |
|   └─Selection_19(Probe)              | 0.80     | root      |                        | not(isnull(test.t1.x))                       |
|     └─MaxOneRow_20                   | 1.00     | root      |                        |                                              |
|       └─Union_21                     | 2.00     | root      |                        |                                              |
|         ├─TableReader_24             | 2.00     | root      |                        | data:Selection_23                            |
|         │ └─Selection_23             | 2.00     | cop[tikv] |                        | eq(test.t2.x, test.t1.x), lt(test.t2.x, 2)   |
```
|         │   └─TableFullScan_22       | 2500.00  | cop[tikv] | table:t1, partition:p0 | keep order:false, stats:pseudo               |
|         └─TableReader_27             | 2.00     | root      |                        | data:Selection_26                            |
|           └─Selection_26             | 2.00     | cop[tikv] |                        | eq(test.t2.x, test.t1.x), lt(test.t2.x, 2)   |
|             └─TableFullScan_25       | 2500.00  | cop[tikv] | table:t1, partition:p1 | keep order:false, stats:pseudo               |
+--------------------------------------+----------+-----------+------------------------+----------------------------------------------+
```

このクエリが`t2`から1行読み込むたびに、`t1`のパーティションテーブルでクエリが実行されます。理論的には、この時点で`t1.x = val`のフィルタ条件が満たされますが、実際には、クエリプランの生成フェーズでのみパーティションプルーニングが有効で、実行フェーズで有効になるわけではありません。

### Rangeパーティションテーブルでのパーティションプルーニングの使用

このセクションでは、Rangeパーティションテーブルでのパーティションプルーニングの適用可能な使用シナリオと非適用シナリオについて説明します。

#### Rangeパーティションテーブルでの適用可能なシナリオ

このセクションでは、Rangeパーティションテーブルでのパーティションプルーニングの適用可能な使用シナリオについての3つの適用シナリオを説明します。

##### シナリオ1

パーティションプルーニングは、Rangeパーティションテーブルにおける等価比較のクエリ条件に適用されます。例：

{{< copyable "sql" >}}

```sql
create table t (x int) partition by range (x) (
    partition p0 values less than (5),
    partition p1 values less than (10),
    partition p2 values less than (15)
    );
explain select * from t where x = 3;
```

```sql
+-------------------------+----------+-----------+-----------------------+--------------------------------+
| id                      | estRows  | task      | access object         | operator info                  |
+-------------------------+----------+-----------+-----------------------+--------------------------------+
| TableReader_8           | 10.00    | root      |                       | data:Selection_7               |
| └─Selection_7           | 10.00    | cop[tikv] |                       | eq(test.t.x, 3)                |
|   └─TableFullScan_6     | 10000.00 | cop[tikv] | table:t, partition:p0 | keep order:false, stats:pseudo |
+-------------------------+----------+-----------+-----------------------+--------------------------------+
```

パーティションプルーニングは、`in`クエリ条件を使用する等価比較にも適用されます。例：

{{< copyable "sql" >}}

```sql
create table t (x int) partition by range (x) (
    partition p0 values less than (5),
    partition p1 values less than (10),
    partition p2 values less than (15)
    );
explain select * from t where x in(1,13);
```

```sql
+-----------------------------+----------+-----------+-----------------------+--------------------------------+
| id                          | estRows  | task      | access object         | operator info                  |
+-----------------------------+----------+-----------+-----------------------+--------------------------------+
| Union_8                     | 40.00    | root      |                       |                                |
| ├─TableReader_11            | 20.00    | root      |                       | data:Selection_10              |
| │ └─Selection_10            | 20.00    | cop[tikv] |                       | in(test.t.x, 1, 13)            |
| │   └─TableFullScan_9       | 10000.00 | cop[tikv] | table:t, partition:p0 | keep order:false, stats:pseudo |
| └─TableReader_14            | 20.00    | root      |                       | data:Selection_13              |
|   └─Selection_13            | 20.00    | cop[tikv] |                       | in(test.t.x, 1, 13)            |
|     └─TableFullScan_12      | 10000.00 | cop[tikv] | table:t, partition:p2 | keep order:false, stats:pseudo |
+-----------------------------+----------+-----------+-----------------------+--------------------------------+
```

上記のSQL文から、`x in(1,13)`条件で、すべての結果がいくつかのパーティションに含まれることがわかります。分析すると、`x = 1`のすべてのレコードが`p0`パーティションにあり、`x = 13`のすべてのレコードが`p2`パーティションにあることがわかります。そのため、`p0`と`p2`パーティションのみにアクセスする必要があります。

##### シナリオ2

パーティションプルーニングは、`between`、`>`、`<`、`=`、`>=`、`<=`などの間隔比較のクエリ条件に適用されます。例：

{{< copyable "sql" >}}

```sql
create table t (x int) partition by range (x) (
    partition p0 values less than (5),
    partition p1 values less than (10),
    partition p2 values less than (15)
    );
explain select * from t where x between 7 and 14;
```

```sql
+-----------------------------+----------+-----------+-----------------------+-----------------------------------+
| id                          | estRows  | task      | access object         | operator info                     |
+-----------------------------+----------+-----------+-----------------------+-----------------------------------+
| Union_8                     | 500.00   | root      |                       |                                   |
| ├─TableReader_11            | 250.00   | root      |                       | data:Selection_10                 |
| │ └─Selection_10            | 250.00   | cop[tikv] |                       | ge(test.t.x, 7), le(test.t.x, 14) |
| │   └─TableFullScan_9       | 10000.00 | cop[tikv] | table:t, partition:p1 | keep order:false, stats:pseudo    |
| └─TableReader_14            | 250.00   | root      |                       | data:Selection_13                 |
|   └─Selection_13            | 250.00   | cop[tikv] |                       | ge(test.t.x, 7), le(test.t.x, 14) |
|     └─TableFullScan_12      | 10000.00 | cop[tikv] | table:t, partition:p2 | keep order:false, stats:pseudo    |
+-----------------------------+----------+-----------+-----------------------+-----------------------------------+
```

##### シナリオ3

パーティションプルーニングは、パーティション式が`fn(col)`の単純な形で、クエリ条件が`>`、`<`、`=`、`>=`、`<=`のいずれかであるシナリオに適用され、`fn`関数が単調である場合に適用されます。

`fn`関数が単調であれば、任意の`x`と`y`について、`x > y`ならば`fn(x) > fn(y)`となります。このような`fn`関数は厳密に単調と呼ばれます。任意の`x`と`y`について、`x > y`ならば`fn(x) >= fn(y)`となります。この場合、`fn`は単調と呼べます。理論的には、すべての単調関数が、厳密でなくても、パーティションプルーニングをサポートします。現在、TiDBは次の単調関数のみをサポートしています。

* [`UNIX_TIMESTAMP()`](/functions-and-operators/date-and-time-functions.md)
* [`TO_DAYS()`](/functions-and-operators/date-and-time-functions.md)

例えば、`fn`が単調関数`to_days`の場合、パーティションプルーニングが適用されます：

{{< copyable "sql" >}}

```sql
create table t (id datetime) partition by range (to_days(id)) (
    partition p0 values less than (to_days('2020-04-01')),
    partition p1 values less than (to_days('2020-05-01')));
explain select * from t where id > '2020-04-18';
```

```sql
+-------------------------+----------+-----------+-----------------------+-------------------------------------------+
| id                      | estRows  | task      | access object         | operator info                             |
+-------------------------+----------+-----------+-----------------------+-------------------------------------------+
| TableReader_8           | 3333.33  | root      |                       | data:Selection_7                          |
| └─Selection_7           | 3333.33  | cop[tikv] |                       | gt(test.t.id, 2020-04-18 00:00:00.000000) |
|   └─TableFullScan_6     | 10000.00 | cop[tikv] | table:t, partition:p1 | keep order:false, stats:pseudo            |
+-------------------------+----------+-----------+-----------------------+-------------------------------------------+
```

#### Rangeパーティションテーブルでの非適用シナリオ

クエリプランの生成フェーズ中にルール最適化が行われるため、パーティションプルーニングは実行フェーズ中でのみフィルタ条件を取得可能なシナリオには適用されません。例：
```sql
CREATE TABLE t1 (x int) RANGE パーティション (x) でテーブル (
    パーティション p0 値は (5) より小さい,
    パーティション p1 値は (10) より小さい);
CREATE TABLE t2 (x int);
説明 SELECT * FROM t2 WHERE x < (SELECT * FROM t1 WHERE t2.x < t1.x AND t2.x < 2);
```

```
+--------------------------------------+----------+-----------+------------------------+-----------------------------------------------------------+
| id                                   | estRows  | task      | access object          | operator info                                             |
+--------------------------------------+----------+-----------+------------------------+-----------------------------------------------------------+
| Projection_13                        | 9990.00  | root      |                        | test.t2.x                                                 |
| └─Apply_15                           | 9990.00  | root      |                        | CARTESIAN inner join, other cond:lt(test.t2.x, test.t1.x) |
|   ├─TableReader_18(Build)            | 9990.00  | root      |                        | data:Selection_17                                         |
|   │ └─Selection_17                   | 9990.00  | cop[tikv] |                        | not(isnull(test.t2.x))                                    |
|   │   └─TableFullScan_16             | 10000.00 | cop[tikv] | table:t2               | keep order:false, stats:pseudo                            |
|   └─Selection_19(Probe)              | 0.80     | root      |                        | not(isnull(test.t1.x))                                    |
|     └─MaxOneRow_20                   | 1.00     | root      |                        |                                                           |
|       └─Union_21                     | 2.00     | root      |                        |                                                           |
|         ├─TableReader_24             | 2.00     | root      |                        | data:Selection_23                                         |
|         │ └─Selection_23             | 2.00     | cop[tikv] |                        | lt(test.t2.x, 2), lt(test.t2.x, test.t1.x)                |
|         │   └─TableFullScan_22       | 2.50     | cop[tikv] | table:t1, partition:p0 | keep order:false, stats:pseudo                            |
|         └─TableReader_27             | 2.00     | root      |                        | data:Selection_26                                         |
|           └─Selection_26             | 2.00     | cop[tikv] |                        | lt(test.t2.x, 2), lt(test.t2.x, test.t1.x)                |
|             └─TableFullScan_25       | 2.50     | cop[tikv] | table:t1, partition:p1 | keep order:false, stats:pseudo                            |
+--------------------------------------+----------+-----------+------------------------+-----------------------------------------------------------+
クエリの再順序化の生成フェーズでパーティションプルーニングのみが効果を持ちますが、実行フェーズでは効果がありません。
14 行セット (0.00 秒)
```