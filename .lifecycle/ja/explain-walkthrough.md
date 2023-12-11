---
title: EXPLAINの操作手順
summary: 実行計画を解析するEXPLAINの使用方法を学びます

# `EXPLAIN`の操作手順

SQLは宣言型の言語であるため、クエリが効率的に実行されているかどうかは自動的には分かりません。まず、[EXPLAIN](/sql-statements/sql-statement-explain.md)ステートメントを使用して現在の実行計画を把握する必要があります。

<CustomContent platform="tidb">

[bikeshareのサンプルデータベース](/import-example-data.md)から次のステートメントは、2017年7月1日に行われたトリップの数をカウントしています：

</CustomContent>

<CustomContent platform="tidb-cloud">

[bikeshareのサンプルデータベース](/tidb-cloud/import-sample-data.md)から次のステートメントは、2017年7月1日に行われたトリップの数をカウントしています：

</CustomContent>

{{< copyable "sql" >}}

```sql
EXPLAIN SELECT count(*) FROM trips WHERE start_date BETWEEN '2017-07-01 00:00:00' AND '2017-07-01 23:59:59';
```

```sql
+------------------------------+----------+-----------+---------------+------------------------------------------------------------------------------------------------------------------------+
| id                           | estRows  | task      | access object | operator info                                                                                                          |
+------------------------------+----------+-----------+---------------+------------------------------------------------------------------------------------------------------------------------+
| StreamAgg_20                 | 1.00     | root      |               | funcs:count(Column#13)->Column#11                                                                                      |
| └─TableReader_21             | 1.00     | root      |               | data:StreamAgg_9                                                                                                       |
|   └─StreamAgg_9              | 1.00     | cop[tikv] |               | funcs:count(1)->Column#13                                                                                              |
|     └─Selection_19           | 250.00   | cop[tikv] |               | ge(bikeshare.trips.start_date, 2017-07-01 00:00:00.000000), le(bikeshare.trips.start_date, 2017-07-01 23:59:59.000000) |
|       └─TableFullScan_18     | 10000.00 | cop[tikv] | table:trips   | keep order:false, stats:pseudo                                                                                         |
+------------------------------+----------+-----------+---------------+------------------------------------------------------------------------------------------------------------------------+
5 rows in set (0.00 sec)
```

From the child operator `└─TableFullScan_18` back, you can see its execution process as follows, which is currently suboptimal:

1. コプロセッサ（TiKV）は、`TableFullScan`操作として`trips`テーブル全体を読み取ります。次に、それが読み取った行を、まだTiKV内にある`Selection_19`演算子に渡します。
2. その後、`Selection_19`演算子で`WHERE start_date BETWEEN ..`述語がフィルタリングされます。この選択条件に合致する行の数はおおよそ`250`行と推定されています。この数値は統計情報と演算子の論理に基づいて推定されます。`└─TableFullScan_18`演算子は`stats:pseudo`と表示されており、テーブルに実際の統計情報がないことを意味します。`ANALYZE TABLE trips`を実行して統計情報を収集した後には、統計情報がより正確になることが期待されます。
3. 選択基準に合致する行には、`count`関数が適用されます。この処理もまた、まだTiKV内にある`StreamAgg_9`演算子内で完了されます（`cop[tikv]`）。TiKVコプロセッサは、多くのMySQL組み込み関数を実行できますが、その内の1つが`count`です。
4. `StreamAgg_9`からの結果は、今度はTiDBサーバー（`root`のタスク）内にある`TableReader_21`演算子に送られます。この演算子の`estRows`列の値は`1`であり、演算子はアクセスされる各TiKVリージョンから1行を受け取るということを意味します。これらのリクエストに関する詳細情報については、[`EXPLAIN ANALYZE`](/sql-statements/sql-statement-explain-analyze.md)を参照してください。
5. `StreamAgg_20`演算子は、`└─TableReader_21`演算子の各行に`count`関数を適用し、[`SHOW TABLE REGIONS`](/sql-statements/sql-statement-show-table-regions.md)で表示することができる約56行になります。これはルート演算子であるため、それから結果をクライアントに戻します。

> **注意:**
>
> テーブルが持つリージョンの一般的な表示を取得するには、[`SHOW TABLE REGIONS`](/sql-statements/sql-statement-show-table-regions.md)を実行します。

## 現在のパフォーマンスを評価します

`EXPLAIN`はクエリの実行計画を返すだけであり、クエリを実行しません。実際の実行時間を取得するには、クエリを実行するか`EXPLAIN ANALYZE`を使用できます：

{{< copyable "sql" >}}

```sql
EXPLAIN ANALYZE SELECT count(*) FROM trips WHERE start_date BETWEEN '2017-07-01 00:00:00' AND '2017-07-01 23:59:59';
```

```sql
+------------------------------+----------+----------+-----------+---------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------+-----------+------+
| id                           | estRows  | actRows  | task      | access object | execution info                                                                                                                                                                                                                                    | operator info                                                                                                          | memory    | disk |
+------------------------------+----------+----------+-----------+---------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------+-----------+------+
| StreamAgg_20                 | 1.00     | 1        | root      |               | time:1.031417203s, loops:2                                                                                                                                                                                                                        | funcs:count(Column#13)->Column#11                                                                                      | 632 Bytes | N/A  |
| └─TableReader_21             | 1.00     | 56       | root      |               | time:1.031408123s, loops:2, cop_task: {num: 56, max: 782.147269ms, min: 5.759953ms, avg: 252.005927ms, p95: 609.294603ms, max_proc_keys: 910371, p95_proc_keys: 704775, tot_proc: 11.524s, tot_wait: 580ms, rpc_num: 56, rpc_time: 14.111932641s} | data:StreamAgg_9                                                                                                       | 328 Bytes | N/A  |
|   └─StreamAgg_9              | 1.00     | 56       | cop[tikv] |               | proc max:640ms, min:8ms, p80:276ms, p95:480ms, iters:18695, tasks:56                                                                                                                                                                              | funcs:count(1)->Column#13                                                                                              | N/A       | N/A  |
|     └─Selection_19           | 250.00   | 11409    | cop[tikv] |               | proc max:640ms, min:8ms, p80:276ms, p95:476ms, iters:18695, tasks:56                                                                                                                                                                              | ge(bikeshare.trips.start_date, 2017-07-01 00:00:00.000000), le(bikeshare.trips.start_date, 2017-07-01 23:59:59.000000) | N/A       | N/A  |
|       └─TableFullScan_18     | 10000.00 | 19117643 | cop[tikv] | table:trips   | proc max:612ms, min:8ms, p80:248ms, p95:460ms, iters:18695, tasks:56                                                                                                                                                                              | keep order:false, stats:pseudo                                                                                         | N/A       | N/A  |
+------------------------------+----------+----------+-----------+---------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------+-----------+------+
5 rows in set (1.03 sec)
```

上記のクエリ例は、`1.03`秒で実行され、理想的なパフォーマンスです。

上記の`EXPLAIN ANALYZE`の結果から、`actRows`は、いくつかの推定値(`estRows`)が不正確であることを示しており、それはすでに`└─TableFullScan_18`の`operator info`(`stats:pseudo`)で示されています. [`ANALYZE TABLE`](/sql-statements/sql-statement-analyze-table.md)を実行してから`EXPLAIN ANALYZE`を再度実行すると、推定がはるかに近くなることがわかります：

{{< copyable "sql" >}}

```sql
ANALYZE TABLE trips;
EXPLAIN ANALYZE SELECT count(*) FROM trips WHERE start_date BETWEEN '2017-07-01 00:00:00' AND '2017-07-01 23:59:59';
```

```sql
Query OK, 0 rows affected (10.22 sec)

+------------------------------+-------------+----------+-----------+---------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------+-----------+------+
| id                           | estRows     | actRows  | task      | access object | execution info                                                                                                                                                                                                                                   | operator info                                                                                                          | memory    | disk |
```
```
+------------------------------+-------------+----------+-----------+---------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------+-----------+------+
| StreamAgg_20                 | 1.00        | 1        | root      |               | time:926.393612ms, loops:2                                                                                                                                                                                                                       | funcs:count(Column#13)->Column#11                                                                                      | 632 Bytes | N/A  |
| └─TableReader_21             | 1.00        | 56       | root      |               | time:926.384792ms, loops:2, cop_task: {num: 56, max: 850.94424ms, min: 6.042079ms, avg: 234.987725ms, p95: 495.474806ms, max_proc_keys: 910371, p95_proc_keys: 704775, tot_proc: 10.656s, tot_wait: 904ms, rpc_num: 56, rpc_time: 13.158911952s} | data:StreamAgg_9                                                                                                       | 328 Bytes | N/A  |
|   └─StreamAgg_9              | 1.00        | 56       | cop[tikv] |               | proc max:592ms, min:4ms, p80:244ms, p95:480ms, iters:18695, tasks:56                                                                                                                                                                             | funcs:count(1)->Column#13                                                                                              | N/A       | N/A  |
|     └─Selection_19           | 432.89      | 11409    | cop[tikv] |               | proc max:592ms, min:4ms, p80:244ms, p95:480ms, iters:18695, tasks:56                                                                                                                                                                             | ge(bikeshare.trips.start_date, 2017-07-01 00:00:00.000000), le(bikeshare.trips.start_date, 2017-07-01 23:59:59.000000) | N/A       | N/A  |
|       └─TableFullScan_18     | 19117643.00 | 19117643 | cop[tikv] | table:trips   | proc max:564ms, min:4ms, p80:228ms, p95:456ms, iters:18695, tasks:56                                                                                                                                                                             | keep order:false                                                                                                       | N/A       | N/A  |
+------------------------------+-------------+----------+-----------+---------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------+-----------+------+
5 rows in set (0.93 sec)
```

`ANALYZE TABLE`を実行した後、`└─TableFullScan_18`オペレータの推定行数は正確であり、`└─Selection_19`の推定行数もはるかに近いことがわかります。上記の2つのケースでは、実行計画（このクエリを実行するためにTiDBが使用するオペレータのセット）が変わらなくても、しばしば最適でない実行計画は古い統計情報によって引き起こされます。

`ANALYZE TABLE`に加えて、TiDBは背景で[`tidb_auto_analyze_ratio`](/system-variables.md#tidb_auto_analyze_ratio)の閾値を超えた後に統計情報を自動的に再生成します。この閾値にTiDBがどれくらい近いか（TiDBが統計情報をどれだけ健康的と見なしているか）は、[`SHOW STATS_HEALTHY`](/sql-statements/sql-statement-show-stats-healthy.md)ステートメントを実行することで確認できます。

{{< copyable "sql" >}}

```sql
SHOW STATS_HEALTHY;
```

```sql
+-----------+------------+----------------+---------+
| Db_name   | Table_name | Partition_name | Healthy |
+-----------+------------+----------------+---------+
| bikeshare | trips      |                |     100 |
+-----------+------------+----------------+---------+
1行の結果 (0.00 秒)
```

## 最適化の特定

現在の実行計画は、以下の点で効率的です。

* 作業のほとんどはTiKVコプロセッサ内で処理されています。処理する必要がある行は56行だけで、これらの行のそれぞれは短く、選択に一致する行数のみを含んでいます。

* TiDB（`StreamAgg_20`）とTiKV（`└─StreamAgg_9`）の両方で行数を集計することで、ストリーム集約が非常に効率的にメモリを使用しています。

しかし、現在の実行計画の最大の問題は、述語`start_date BETWEEN '2017-07-01 00:00:00' AND '2017-07-01 23:59:59'`が直ちに適用されないことです。すべての行が最初に`TableFullScan`オペレータで読み込まれ、その後に選択が適用されます。この原因は`SHOW CREATE TABLE trips`の出力から特定できます。

{{< copyable "sql" >}}

```sql
SHOW CREATE TABLE trips\G
```

```sql
*************************** 1. row ***************************
       Table: trips
Create Table: CREATE TABLE `trips` (
  `trip_id` bigint(20) NOT NULL AUTO_INCREMENT,
  `duration` int(11) NOT NULL,
  `start_date` datetime DEFAULT NULL,
  `end_date` datetime DEFAULT NULL,
  `start_station_number` int(11) DEFAULT NULL,
  `start_station` varchar(255) DEFAULT NULL,
  `end_station_number` int(11) DEFAULT NULL,
  `end_station` varchar(255) DEFAULT NULL,
  `bike_number` varchar(255) DEFAULT NULL,
  `member_type` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`trip_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin AUTO_INCREMENT=20477318
1行の結果 (0.00 秒)
```

`start_date`には**インデックスがありません**。この述語をインデックスリーダーオペレータに押し込むためには、インデックスが必要です。インデックスを以下のように追加します。

{{< copyable "sql" >}}

```sql
ALTER TABLE trips ADD INDEX (start_date);
```

```sql
クエリは実行されました（0件のデータが影響を受けました） (2分10.23秒)
```

> **注記:**
>
> DDLジョブの進捗状況は、[`ADMIN SHOW DDL JOBS`](/sql-statements/sql-statement-admin-show-ddl.md)コマンドを使用してモニタリングできます。TiDBではデフォルトの値が注意深く選択されており、インデックスの追加が本番のワークロードにあまり影響を与えないようにしています。テスト環境では、[`tidb_ddl_reorg_batch_size`](/system-variables.md#tidb_ddl_reorg_batch_size)と[`tidb_ddl_reorg_worker_cnt`](/system-variables.md#tidb_ddl_reorg_worker_cnt)の値を増やすことを検討してください。参照システムでは、バッチサイズを`10240`、ワーカー数を`32`に設定すると、デフォルトより10倍のパフォーマンス向上が得られます。

インデックスを追加した後、`EXPLAIN`でクエリを繰り返すことができます。次の出力では、新しい実行計画が選択され、`TableFullScan`および`Selection`オペレータが除外されたことがわかります。

{{< copyable "sql" >}}

```sql
EXPLAIN SELECT count(*) FROM trips WHERE start_date BETWEEN '2017-07-01 00:00:00' AND '2017-07-01 23:59:59';
```

```sql
+-----------------------------+---------+-----------+-------------------------------------------+-------------------------------------------------------------------+
| id                          | estRows | task      | access object                             | operator info                                                     |
+-----------------------------+---------+-----------+-------------------------------------------+-------------------------------------------------------------------+
| StreamAgg_17                | 1.00    | root      |                                           | funcs:count(Column#13)->Column#11                                 |
| └─IndexReader_18            | 1.00    | root      |                                           | index:StreamAgg_9                                                 |
|   └─StreamAgg_9             | 1.00    | cop[tikv] |                                           | funcs:count(1)->Column#13                                         |
|     └─IndexRangeScan_16     | 8471.88 | cop[tikv] | table:trips, index:start_date(start_date) | range:[2017-07-01 00:00:00,2017-07-01 23:59:59], keep order:false |
+-----------------------------+---------+-----------+-------------------------------------------+-------------------------------------------------------------------+
4行の結果 (0.00 秒)
```

実際の実行時間を比較するためには、再度[`EXPLAIN ANALYZE`](/sql-statements/sql-statement-explain-analyze.md)を使用できます。

{{< copyable "sql" >}}

```sql
EXPLAIN ANALYZE SELECT count(*) FROM trips WHERE start_date BETWEEN '2017-07-01 00:00:00' AND '2017-07-01 23:59:59';
```

```sql
+-----------------------------+---------+---------+-----------+-------------------------------------------+------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+-----------+------+
| id                          | estRows | actRows | task      | access object                             | execution info                                                                                                   | operator info                                                     | memory    | disk |
+-----------------------------+---------+---------+-----------+-------------------------------------------+------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+-----------+------+
```
| StreamAgg_17                | 1.00    | 1       | root      |                                           | time:4.516728ms, loops:2                                                                                         | funcs:count(Column#13)->Column#11                                 | 372 Bytes | N/A  |
| └─IndexReader_18            | 1.00    | 1       | root      |                                           | time:4.514278ms, loops:2, cop_task: {num: 1, max:4.462288ms, proc_keys: 11409, rpc_num: 1, rpc_time: 4.457148ms} | index:StreamAgg_9                                                 | 238 Bytes | N/A  |
|   └─StreamAgg_9             | 1.00    | 1       | cop[tikv] |                                           | time:4ms, loops:12                                                                                               | funcs:count(1)->Column#13                                         | N/A       | N/A  |
|     └─IndexRangeScan_16     | 8471.88 | 11409   | cop[tikv] | table:trips, index:start_date(start_date) | time:4ms, loops:12                                                                                               | range:[2017-07-01 00:00:00,2017-07-01 23:59:59], keep order:false | N/A       | N/A  |
+-----------------------------+---------+---------+-----------+-------------------------------------------+------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+-----------+------+
4 rows in set (0.00 sec)

前述の結果から、クエリの実行時間が1.03秒から0.0秒に短縮されました。

> **注:**
>
> ここで適用されているもう1つの最適化は、並列処理キャッシュです。インデックスを追加できない場合は、[coprocessor cache](/coprocessor-cache.md)を有効にすることを検討してください。これを有効にすると、リージョンが最後に演算子が実行された後に変更されていない限り、TiKVはキャッシュから値を返します。これにより、高コストな`TableFullScan`および`Selection`演算子の多くのコストも削減されます。

## サブクエリの早期実行の無効化

クエリの最適化中、TiDBは直接計算できるサブクエリを事前に実行します。たとえば:

```sql
CREATE TABLE t1(a int);
INSERT INTO t1 VALUES(1);
CREATE TABLE t2(a int);
EXPLAIN SELECT * FROM t2 WHERE a = (SELECT a FROM t1);
```

```sql
+---------------------------+----------+-----------+---------------+--------------------------------+
| id                        | estRows  | task      | access object | operator info                  |
+---------------------------+----------+-----------+---------------+--------------------------------+
| TableReader_14            | 10.00    | root      |               | data:Selection_13              |
| └─Selection_13            | 10.00    | cop[tikv] |               | eq(test.t2.a, 1)               |
|   └─TableFullScan_12      | 10000.00 | cop[tikv] | table:t2      | keep order:false, stats:pseudo |
+---------------------------+----------+-----------+---------------+--------------------------------+
3 rows in set (0.00 sec)
```

前述の例では、 `a = (SELECT a FROM t1)` のサブクエリは最適化中に計算され、 `t2.a=1` に書き換えられます。これにより、定数伝搬や折りたたみなどの最適化がより可能となります。ただし、`EXPLAIN` 文の実行時間に影響を与えます。 サブクエリ自体の実行に時間がかかる場合、`EXPLAIN` 文が完了しない可能性があり、オンライントラブルシューティングに影響を与える可能性があります。

v7.3.0から、TiDBは[`tidb_opt_enable_non_eval_scalar_subquery`](/system-variables.md#tidb_opt_enable_non_eval_scalar_subquery-new-in-v730)システム変数を導入し、`EXPLAIN`でこのようなサブクエリの事前実行を無効にするかどうかを制御します。この変数のデフォルト値は`OFF`であり、サブクエリを事前に計算します。この変数を`ON`に設定すると、サブクエリの事前実行を無効にすることができます:

```sql
SET @@tidb_opt_enable_non_eval_scalar_subquery = ON;
EXPLAIN SELECT * FROM t2 WHERE a = (SELECT a FROM t1);
```

```sql
+---------------------------+----------+-----------+---------------+---------------------------------+
| id                        | estRows  | task      | access object | operator info                   |
+---------------------------+----------+-----------+---------------+---------------------------------+
| Selection_13              | 8000.00  | root      |               | eq(test.t2.a, ScalarQueryCol#5) |
| └─TableReader_15          | 10000.00 | root      |               | data:TableFullScan_14           |
|   └─TableFullScan_14      | 10000.00 | cop[tikv] | table:t2      | keep order:false, stats:pseudo  |
| ScalarSubQuery_10         | N/A      | root      |               | Output: ScalarQueryCol#5        |
| └─MaxOneRow_6             | 1.00     | root      |               |                                 |
|   └─TableReader_9         | 1.00     | root      |               | data:TableFullScan_8            |
|     └─TableFullScan_8     | 1.00     | cop[tikv] | table:t1      | keep order:false, stats:pseudo  |
+---------------------------+----------+-----------+---------------+---------------------------------+
7 rows in set (0.00 sec)
```

ご覧の通り、スカラーサブクエリは実行中に展開されないため、このようなSQLの具体的な実行プロセスを理解しやすくなります。

> **注:**
>
> [`tidb_opt_enable_non_eval_scalar_subquery`](/system-variables.md#tidb_opt_enable_non_eval_scalar_subquery-new-in-v730)は`EXPLAIN`文の動作にのみ影響し、`EXPLAIN ANALYZE`文は引き続きサブクエリを事前に実行します。