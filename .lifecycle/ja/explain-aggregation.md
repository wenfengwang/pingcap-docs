---
title: 集約を使用したステートメントの説明
summary: TiDBの`EXPLAIN`ステートメントによって返される実行計画情報について学びます。

# 集約を使用したステートメントの説明

データを集約する際、SQLオプティマイザはHash AggregationまたはStream Aggregation演算子のいずれかを選択します。クエリの効率を向上させるために、集約はコプロセッサとTiDBの両方で実行されます。次の例を考慮してください：

{{< copyable "sql" >}}

```sql
CREATE TABLE t1 (id INT NOT NULL PRIMARY KEY auto_increment, pad1 BLOB, pad2 BLOB, pad3 BLOB);
INSERT INTO t1 SELECT NULL, RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024) FROM dual;
INSERT INTO t1 SELECT NULL, RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024) FROM t1 a JOIN t1 b JOIN t1 c LIMIT 10000;
INSERT INTO t1 SELECT NULL, RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024) FROM t1 a JOIN t1 b JOIN t1 c LIMIT 10000;
INSERT INTO t1 SELECT NULL, RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024) FROM t1 a JOIN t1 b JOIN t1 c LIMIT 10000;
--（省略）
SELECT SLEEP(1);
ANALYZE TABLE t1;
```

次に示す[`SHOW TABLE REGIONS`](/sql-statements/sql-statement-show-table-regions.md)の出力から、このテーブルが複数のリージョンに分割されていることがわかります：

{{< copyable "sql" >}}

```sql
SHOW TABLE t1 REGIONS;
```

```sql
+-----------+--------------+--------------+-----------+-----------------+-------+------------+---------------+------------+----------------------+------------------+
| REGION_ID | START_KEY    | END_KEY      | LEADER_ID | LEADER_STORE_ID | PEERS | SCATTERING | WRITTEN_BYTES | READ_BYTES | APPROXIMATE_SIZE(MB) | APPROXIMATE_KEYS |
+-----------+--------------+--------------+-----------+-----------------+-------+------------+---------------+------------+----------------------+------------------+
|        64 | t_64_        | t_64_r_31766 |        65 |               1 | 65    |          0 |          1325 |  102033520 |                   98 |            52797 |
|        66 | t_64_r_31766 | t_64_r_63531 |        67 |               1 | 67    |          0 |          1325 |   72522521 |                  104 |            78495 |
|        68 | t_64_r_63531 | t_64_r_95296 |        69 |               1 | 69    |          0 |          1325 |          0 |                  104 |            95433 |
|         2 | t_64_r_95296 |              |         3 |               1 | 3     |          0 |          1501 |          0 |                   81 |            63211 |
+-----------+--------------+--------------+-----------+-----------------+-------+------------+---------------+------------+----------------------+------------------+
4 行が返されました (0.00 秒)
```

以下の集約ステートメントで`EXPLAIN`を使用すると、TiKV内の各リージョンで最初に`└─StreamAgg_8`が実行され、その後、各リージョンからTiDBに1行が送信され、TiDBで各リージョンのデータが`StreamAgg_16`で集約されます：

{{< copyable "sql" >}}

```sql
EXPLAIN SELECT COUNT(*) FROM t1;
```

```sql
+----------------------------+-----------+-----------+---------------+---------------------------------+
| id                         | estRows   | task      | access object | operator info                   |
+----------------------------+-----------+-----------+---------------+---------------------------------+
| StreamAgg_16               | 1.00      | root      |               | funcs:count(Column#7)->Column#5 |
| └─TableReader_17           | 1.00      | root      |               | data:StreamAgg_8                |
|   └─StreamAgg_8            | 1.00      | cop[tikv] |               | funcs:count(1)->Column#7        |
|     └─TableFullScan_15     | 242020.00 | cop[tikv] | table:t1      | keep order:false                |
+----------------------------+-----------+-----------+---------------+---------------------------------+
4 行が返されました (0.00 秒)
```

`EXPLAIN ANALYZE`でこれを観察するのが最も簡単です。`TableFullScan`が使用されており、二次索引がないため、`actRows`が`SHOW TABLE REGIONS`のリージョンの数と一致することがわかります：

```sql
EXPLAIN ANALYZE SELECT COUNT(*) FROM t1;
```

```sql
+----------------------------+-----------+---------+-----------+---------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------------------+-----------+------+
| id                         | estRows   | actRows | task      | access object | execution info                                                                                                                                                                                                                                  | operator info                   | memory    | disk |
+----------------------------+-----------+---------+-----------+---------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------------------+-----------+------+
| StreamAgg_16               | 1.00      | 1       | root      |               | time:12.609575ms, loops:2                                                                                                                                                                                                                       | funcs:count(Column#7)->Column#5 | 372 Bytes | N/A  |
| └─TableReader_17           | 1.00      | 4       | root      |               | time:12.605155ms, loops:2, cop_task: {num: 4, max: 12.538245ms, min: 9.256838ms, avg: 10.895114ms, p95: 12.538245ms, max_proc_keys: 31765, p95_proc_keys: 31765, tot_proc: 48ms, rpc_num: 4, rpc_time: 43.530707ms, copr_cache_hit_ratio: 0.00} | data:StreamAgg_8                | 293 Bytes | N/A  |
|   └─StreamAgg_8            | 1.00      | 4       | cop[tikv] |               | proc max:12ms, min:12ms, p80:12ms, p95:12ms, iters:122, tasks:4                                                                                                                                                                                 | funcs:count(1)->Column#7        | N/A       | N/A  |
|     └─TableFullScan_15     | 242020.00 | 121010  | cop[tikv] | table:t1      | proc max:12ms, min:12ms, p80:12ms, p95:12ms, iters:122, tasks:4                                                                                                                                                                                 | keep order:false                | N/A       | N/A  |
4 行が返されました (0.00 秒)
```
```
      +----------------------------+-----------+---------+-----------+---------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------------------+-----------+------+
4 行のセット (0.01 秒)

## ハッシュ集約

ハッシュ集約アルゴリズムは、集約を行う間に中間結果を格納するためにハッシュテーブルを使用します。複数のスレッドを使用して並列で実行されますが、Stream Aggregation よりも多くのメモリを消費します。

以下は `HashAgg` 演算子の例です。

{{< copyable "sql" >}}

```sql
EXPLAIN SELECT /*+ HASH_AGG() */ count(*) FROM t1;
```

```sql
+---------------------------+-----------+-----------+---------------+---------------------------------+
| id                        | estRows   | task      | access object | operator info                   |
+---------------------------+-----------+-----------+---------------+---------------------------------+
| HashAgg_9                 | 1.00      | root      |               | funcs:count(Column#6)->Column#5 |
| └─TableReader_10          | 1.00      | root      |               | data:HashAgg_5                  |
|   └─HashAgg_5             | 1.00      | cop[tikv] |               | funcs:count(1)->Column#6        |
|     └─TableFullScan_8     | 242020.00 | cop[tikv] | table:t1      | keep order:false                |
+---------------------------+-----------+-----------+---------------+---------------------------------+
4 行のセット (0.00 秒)

`operator info` によると、データを集約するために使用されるハッシュ関数は `funcs:count(1)->Column#6` です。

## ストリーム集約

ストリーム集約アルゴリズムは、通常、ハッシュ集約よりも少ないメモリを消費します。ただし、この演算子ではデータが順序付けられて送信される必要があり、到着した値に対して集約を適用できるようにするためにストリームされます。

次の例を考えてみましょう。

{{< copyable "sql" >}}

```sql
CREATE TABLE t2 (id INT NOT NULL PRIMARY KEY, col1 INT NOT NULL);
INSERT INTO t2 VALUES (1, 9),(2, 3),(3,1),(4,8),(6,3);
EXPLAIN SELECT /*+ STREAM_AGG() */ col1, count(*) FROM t2 GROUP BY col1;
```

```sql
Query OK, 0 行が影響を受けました (0.11 秒)

Query OK, 5 行が影響を受けました (0.01 秒)
Records: 5  Duplicates: 0  Warnings: 0

+------------------------------+----------+-----------+---------------+---------------------------------------------------------------------------------------------+
| id                           | estRows  | task      | access object | operator info                                                                               |
+------------------------------+----------+-----------+---------------+---------------------------------------------------------------------------------------------+
| Projection_4                 | 8000.00  | root      |               | test.t2.col1, Column#3                                                                      |
| └─StreamAgg_8                | 8000.00  | root      |               | group by:test.t2.col1, funcs:count(1)->Column#3, funcs:firstrow(test.t2.col1)->test.t2.col1 |
|   └─Sort_13                  | 10000.00 | root      |               | test.t2.col1                                                                                |
|     └─TableReader_12         | 10000.00 | root      |               | data:TableFullScan_11                                                                       |
|       └─TableFullScan_11     | 10000.00 | cop[tikv] | table:t2      | keep order:false, stats:pseudo                                                              |
+------------------------------+----------+-----------+---------------+---------------------------------------------------------------------------------------------+
5 行のセット (0.00 秒)

この例では、`└─Sort_13` 演算子を `col1` にインデックスを追加することで削除できます。インデックスが追加されると、データは順序で読み取ることができ、`└─Sort_13` 演算子が削除されます。

{{< copyable "sql" >}}

```sql
ALTER TABLE t2 ADD INDEX (col1);
EXPLAIN SELECT /*+ STREAM_AGG() */ col1, count(*) FROM t2 GROUP BY col1;
```

```sql
Query OK, 0 行が影響を受けました (0.28 秒)

+------------------------------+---------+-----------+----------------------------+----------------------------------------------------------------------------------------------------+
| id                           | estRows | task      | access object              | operator info                                                                                      |
+------------------------------+---------+-----------+----------------------------+----------------------------------------------------------------------------------------------------+
| Projection_4                 | 4.00    | root      |                            | test.t2.col1, Column#3                                                                             |
| └─StreamAgg_14               | 4.00    | root      |                            | group by:test.t2.col1, funcs:count(Column#4)->Column#3, funcs:firstrow(test.t2.col1)->test.t2.col1 |
|   └─IndexReader_15           | 4.00    | root      |                            | index:StreamAgg_8                                                                                  |
|     └─StreamAgg_8            | 4.00    | cop[tikv] |                            | group by:test.t2.col1, funcs:count(1)->Column#4                                                    |
|       └─IndexFullScan_13     | 5.00    | cop[tikv] | table:t2, index:col1(col1) | keep order:true, stats:pseudo                                                                      |
+------------------------------+---------+-----------+----------------------------+----------------------------------------------------------------------------------------------------+
5 行のセット (0.00 秒)

## ROLLUP を使用した多次元データの集約

v7.4.0 から、TiDB の `GROUP BY` 句は `WITH ROLLUP` 修飾子をサポートしています。

`GROUP BY` 句では、1 つ以上の列をグループリストとして指定し、リストの後に `WITH ROLLUP` 修飾子を追加することができます。すると、TiDB はグループリストの列に基づいて多次元の降順グルーピングを行い、出力で各グループの集計結果を提供します。

> **注意:**
>
> 現時点では、TiDB は Cube 構文をサポートしておらず、TiDB は TiFlash MPP モードでの `WITH ROLLUP` 構文に対してのみ有効な実行計画の生成をサポートしています。

```sql
explain SELECT year, month, grouping(year), grouping(month), SUM(profit) AS profit FROM bank GROUP BY year, month WITH ROLLUP;
+----------------------------------------+---------+--------------+---------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| id                                     | estRows | task         | access object | operator info                                                                                                                                                                                                                        |
+----------------------------------------+---------+--------------+---------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| TableReader_44                         | 2.40    | root         |               | MppVersion: 2, data:ExchangeSender_43                                                                                                                                                                                                |
| └─ExchangeSender_43                    | 2.40    | mpp[tiflash] |               | ExchangeType: PassThrough                                                                                                                                                                                                            |
|   └─Projection_8                       | 2.40    | mpp[tiflash] |               | Column#6->Column#12, Column#7->Column#13, grouping(gid)->Column#14, grouping(gid)->Column#15, Column#9->Column#16                                                                                                                    |
|     └─Projection_38                    | 2.40    | mpp[tiflash] |               | Column#9, Column#6, Column#7, gid                                                                                                                                                                                                    |
|       └─HashAgg_36                     | 2.40    | mpp[tiflash] |               | group by:Column#6, Column#7, gid, funcs:sum(test.bank.profit)->Column#9, funcs:firstrow(Column#6)->Column#6, funcs:firstrow(Column#7)->Column#7, funcs:firstrow(gid)->gid, stream_count: 8                                           |
|         └─ExchangeReceiver_22          | 3.00    | mpp[tiflash] |               | stream_count: 8                                                                                                                                                                                                                      |
|           └─ExchangeSender_21          | 3.00    | mpp[tiflash] |               | ExchangeType: HashPartition, Compression: FAST, Hash Cols: [name: Column#6, collate: binary], [name: Column#7, collate: utf8mb4_bin], [name: gid, collate: binary], stream_count: 8                                                  |
|             └─Expand_20                | 3.00    | mpp[tiflash] |               | level-projection:[test.bank.profit, <nil>->Column#6, <nil>->Column#7, 0->gid],[test.bank.profit, Column#6, <nil>->Column#7, 1->gid],[test.bank.profit, Column#6, Column#7, 3->gid]; schema: [test.bank.profit,Column#6,Column#7,gid] |
|               └─Projection_16          | 3.00    | mpp[tiflash] |               | test.bank.profit, test.bank.year->Column#6, test.bank.month->Column#7                                                                                                                                                                |
|                 └─TableFullScan_17     | 3.00    | mpp[tiflash] | table:bank    | keep order:false, stats:pseudo                                                                                                                                                                                                       |
+----------------------------------------+---------+--------------+---------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
10 行のセット (0.05 秒)
```
```
According to the `GROUP BY year, month WITH ROLLUP` syntax in the preceding statement, the SQL aggregation results for this statement can be calculated and concatenated in three groups: `{year, month}`, `{year}`, and `{}` respectively.

For more information, see [GROUP BY modifiers](/functions-and-operators/group-by-modifier.md).
```