---
title: パーティションを使用したステートメントの説明
summary: TiDBのEXPLAINステートメントによって返される実行プラン情報について学びます。

# パーティションを使用したステートメントの説明

`EXPLAIN`ステートメントは、クエリを実行するためにTiDBがアクセスする必要があるパーティションを表示します。[パーティションプルーニング](/partition-pruning.md)のため、表示されるパーティションは通常、全体のパーティションのサブセットになります。このドキュメントでは、一般的なパーティションテーブルの最適化および`EXPLAIN`の出力の解釈方法について説明します。

このドキュメントで使用されるサンプルデータ：

{{< copyable "sql" >}}

```sql
CREATE TABLE t1 (
 id BIGINT NOT NULL auto_increment,
 d date NOT NULL,
 pad1 BLOB,
 pad2 BLOB,
 pad3 BLOB,
 PRIMARY KEY (id,d)
) PARTITION BY RANGE (YEAR(d)) (
 PARTITION p2016 VALUES LESS THAN (2017),
 PARTITION p2017 VALUES LESS THAN (2018),
 PARTITION p2018 VALUES LESS THAN (2019),
 PARTITION p2019 VALUES LESS THAN (2020),
 PARTITION pmax VALUES LESS THAN MAXVALUE
);

INSERT INTO t1 (d, pad1, pad2, pad3) VALUES 
 ('2016-01-01', RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024)),
 ('2016-06-01', RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024)),
 ('2016-09-01', RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024)),
 ('2017-01-01', RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024)),
 ('2017-06-01', RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024)),
 ('2017-09-01', RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024)),
 ('2018-01-01', RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024)),
 ('2018-06-01', RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024)),
 ('2018-09-01', RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024)),
 ('2019-01-01', RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024)),
 ('2019-06-01', RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024)),
 ('2019-09-01', RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024)),
 ('2020-01-01', RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024)),
 ('2020-06-01', RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024)),
 ('2020-09-01', RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024));

INSERT INTO t1 SELECT NULL, a.d, RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024) FROM t1 a JOIN t1 b JOIN t1 c LIMIT 10000;
INSERT INTO t1 SELECT NULL, a.d, RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024) FROM t1 a JOIN t1 b JOIN t1 c LIMIT 10000;
INSERT INTO t1 SELECT NULL, a.d, RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024) FROM t1 a JOIN t1 b JOIN t1 c LIMIT 10000;
INSERT INTO t1 SELECT NULL, a.d, RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024) FROM t1 a JOIN t1 b JOIN t1 c LIMIT 10000;

SELECT SLEEP(1);
ANALYZE TABLE t1;
```

以下の例は、新しく作成されたパーティションテーブルに対するステートメントを示しています：

{{< copyable "sql" >}}

```sql
EXPLAIN SELECT COUNT(*) FROM t1 WHERE d = '2017-06-01';
```

```sql
+------------------------------+---------+-----------+---------------------------+-------------------------------------------+
| id                           | estRows | task      | access object             | operator info                             |
+------------------------------+---------+-----------+---------------------------+-------------------------------------------+
| StreamAgg_21                 | 1.00    | root      |                           | funcs:count(Column#8)->Column#6           |
| └─TableReader_22             | 1.00    | root      |                           | data:StreamAgg_10                         |
|   └─StreamAgg_10             | 1.00    | cop[tikv] |                           | funcs:count(1)->Column#8                  |
|     └─Selection_20           | 8.87    | cop[tikv] |                           | eq(test.t1.d, 2017-06-01 00:00:00.000000) |
|       └─TableFullScan_19     | 8870.00 | cop[tikv] | table:t1, partition:p2017 | keep order:false                          |
+------------------------------+---------+-----------+---------------------------+-------------------------------------------+
5 rows in set (0.01 sec)
```

内側から始めてルート演算子（`StreamAgg_21`）に向かって進むように、以下の手順が実行されます：

* TiDBは、アクセスする必要があるパーティションが1つだけであることを正常に特定しました（`p2017`）。これは`access object`の下でメモされています。
* パーティション自体は、演算子`└─TableFullScan_19`でスキャンされ、その後`└─Selection_20`が適用され、開始日が`2017-06-01 00:00:00.000000`である行をフィルタリングします。
* `└─Selection_20`に一致する行は、コプロセッサでストリーム集約され、`count`関数をネイティブで理解しています。
* 各コプロセッサリクエストは、TiDB内の`└─TableReader_22`に1行を返し、その後`StreamAgg_21`の下でストリームが集約され、1行がクライアントに返されます。

次の例では、パーティションプルーニングはどのパーティションも削除しません：

{{< copyable "sql" >}}

```sql
EXPLAIN SELECT COUNT(*) FROM t1 WHERE YEAR(d) = 2017;
```

```sql
+------------------------------------+----------+-----------+---------------------------+----------------------------------+
| id                                 | estRows  | task      | access object             | operator info                    |
+------------------------------------+----------+-----------+---------------------------+----------------------------------+
| HashAgg_20                         | 1.00     | root      |                           | funcs:count(Column#7)->Column#6  |
| └─PartitionUnion_21                | 5.00     | root      |                           |                                  |
|   ├─StreamAgg_36                   | 1.00     | root      |                           | funcs:count(Column#9)->Column#7  |
|   │ └─TableReader_37               | 1.00     | root      |                           | data:StreamAgg_25                |
|   │   └─StreamAgg_25               | 1.00     | cop[tikv] |                           | funcs:count(1)->Column#9         |
|   │     └─Selection_35             | 6000.00  | cop[tikv] |                           | eq(year(test.t1.d), 2017)        |
|   │       └─TableFullScan_34       | 7500.00  | cop[tikv] | table:t1, partition:p2016 | keep order:false                 |
|   ├─StreamAgg_55                   | 1.00     | root      |                           | funcs:count(Column#11)->Column#7 |
|   │ └─TableReader_56               | 1.00     | root      |                           | data:StreamAgg_44                |
|   │   └─StreamAgg_44               | 1.00     | cop[tikv] |                           | funcs:count(1)->Column#11        |
|   │     └─Selection_54             | 14192.00 | cop[tikv] |                           | eq(year(test.t1.d), 2017)        |
|   │       └─TableFullScan_53       | 17740.00 | cop[tikv] | table:t1, partition:p2017 | keep order:false                 |
|   ├─StreamAgg_74                   | 1.00     | root      |                           | funcs:count(Column#13)->Column#7 |
|   │ └─TableReader_75               | 1.00     | root      |                           | data:StreamAgg_63                |
|   │   └─StreamAgg_63               | 1.00     | cop[tikv] |                           | funcs:count(1)->Column#13        |
|   │     └─Selection_73             | 3977.60  | cop[tikv] |                           | eq(year(test.t1.d), 2017)        |
|   │       └─TableFullScan_72       | 4972.00  | cop[tikv] | table:t1、partition:p2018   | keep order:false |
|   ├─StreamAgg_93                   | 1.00     | root      |                           | funcs:count(Column#15)->Column#7  |
|   │ └─TableReader_94               | 1.00     | root      |                           | data:StreamAgg_82  |
|   │   └─StreamAgg_82               | 1.00     | cop[tikv] |                           | funcs:count(1)->Column#15  |
|   │     └─Selection_92             | 20361.60 | cop[tikv] |                           | eq(year(test.t1.d), 2017)  |
|   │       └─TableFullScan_91       | 25452.00 | cop[tikv] | table:t1、partition:p2019   | keep order:false  |
|   └─StreamAgg_112                  | 1.00     | root      |                           | funcs:count(Column#17)->Column#7  |
|     └─TableReader_113              | 1.00     | root      |                           | data:StreamAgg_101  |
|       └─StreamAgg_101              | 1.00     | cop[tikv] |                           | funcs:count(1)->Column#17  |
|         └─Selection_111            | 8892.80  | cop[tikv] |                           | eq(year(test.t1.d), 2017)  |
|           └─TableFullScan_110      | 11116.00 | cop[tikv] | table:t1、partition:pmax   | keep order:false  |
+------------------------------------+----------+-----------+---------------------------+----------------------------------+
27 rows in set (0.00 sec)

上記の出力から：

* TiDBは、述語`YEAR(d) = 2017`が[sargable](https://en.wikipedia.org/wiki/Sargable)と見なされないため、すべてのパーティション（p2016..pMax）にアクセスする必要があると考えています。この問題はTiDB固有のものではありません。
* 各パーティションがスキャンされると、`Selection`演算子が2017年に一致しない行をフィルタリングします。
* 各パーティションでストリーム集約が行われ、一致する行の数が数えられます。
* 演算子`└─PartitionUnion_21`は、各パーティションへのアクセス結果を結合します。