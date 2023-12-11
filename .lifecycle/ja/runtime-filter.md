---
title: ランタイムフィルター
summary: ランタイムフィルターの動作原理と使用方法を学びます。

# ランタイムフィルター

ランタイムフィルターは、TiDB v7.3で導入された新機能であり、MPPシナリオにおけるハッシュ結合のパフォーマンスを改善することを目的としています。ハッシュ結合のデータを事前にフィルタリングすることで、データのスキャン量とハッシュ結合の計算量を削減し、クエリのパフォーマンスを向上させることができます。

## コンセプト

- ハッシュ結合: 結合演算を実装する方法の1つです。片側にハッシュテーブルを構築し、他方の側のハッシュテーブルと連続して一致させることで結合の結果を取得します。
- ビルド側: ハッシュ結合の片側でハッシュテーブルを構築するために使用されます。本文書では、ハッシュ結合の右側のテーブルをデフォルトでビルド側と呼びます。
- プローブ側: ハッシュ結合の片側でハッシュテーブルを連続して一致させるために使用されます。本文書では、ハッシュ結合の左側のテーブルをデフォルトでプローブ側と呼びます。
- フィルター: 本文書では、フィルター条件のことを指します。

## ランタイムフィルターの動作原理

ハッシュ結合は、右側のテーブルを基にハッシュテーブルを構築し、左側のテーブルを連続してハッシュテーブルに探ることで結合操作を行います。プローブ中に結合キーの値がハッシュテーブルにヒットしない場合、そのデータは右側のテーブルに存在せず、最終的な結合結果に表示されません。そのため、TiDBがスキャン中に結合キーデータを事前にフィルタリングできれば、スキャン時間とネットワークオーバーヘッドを削減し、結合の効率を大幅に改善できます。

ランタイムフィルターは、クエリ計画段階で生成される**動的述語**です。この述語は、TiDB Selectionオペレータ内の他の述語と同様の機能を持ちます。これらの述語は、述語と一致しない行をフィルタリングするためにTable Scanオペレーションに適用されます。唯一の違いは、ランタイムフィルター内のパラメータ値がハッシュ結合構築プロセスで生成された結果から来ていることです。

### 例

`store_sales`テーブルと`date_dim`テーブルの間の結合クエリを考えます。結合方法はハッシュ結合です。`store_sales`は主に店舗の売上データを保持するファクトテーブルであり、行数は100万です。`date_dim`は日付情報を主に保持する時間次元テーブルです。年2001の売上データをクエリしたいため、365行の`date_dim`テーブルが結合操作に関与します。

```sql
SELECT * FROM store_sales, date_dim
WHERE ss_date_sk = d_date_sk
    AND d_year = 2001;
```

ハッシュ結合の実行計画は通常以下のようになります。

```
                 +-------------------+
                 | PhysicalHashJoin  |
        +------->|                   |<------+
        |        +-------------------+       |
        |                                    |
        |                                    |
  100w  |                                    | 365
        |                                    |
        |                                    |
+-------+-------+                   +--------+-------+
| TableFullScan |                   | TableFullScan  |
|  store_sales  |                   |    date_dim    |
+---------------+                   +----------------+
```

*(上記の図は、エクスチェンジノードやその他のノードを省略しています。)*

ランタイムフィルターの実行過程は以下の通りです。

1. `date_dim`テーブルのデータをスキャンします。
2. `PhysicalHashJoin`は、ビルド側のデータに基づいてフィルタ条件を計算します（例: `date_dim in (2001/01/01～2001/12/31)`）。
3. 待機している`store_sales`をスキャンする`TableFullScan`オペレータにフィルタ条件を送信します。
4. フィルタ条件が`store_sales`に適用され、フィルタリングされたデータが`PhysicalHashJoin`に渡され、それによりプローブ側のスキャン量とハッシュテーブルの一致計算量が減少します。

```
                         2. ランタイムフィルターの値を構築
            +-------->+-------------------+
            |         |PhysicalHashJoin   |<-----+
            |    +----+                   |      |
4. フィルタ後  |    |3. ランタイムフィルタ   |      | 1. テーブル2をスキャン
    5000    |    |データ送信                  |      365
            |    |データ                       |
            |    |                              |
      +-----+----v------+               +--------+--------+
      |  TableFullScan  |               | TabelFullScan  |
      |  store_sales    |               |    date_dim    |
      +-----------------+               +----------------+
```

*(RFはランタイムフィルターの略です)*

上記の2つの図から見ると、`store_sales`のスキャン量が100万から5000に削減されています。`TableFullScan`のスキャン量を削減することで、ランタイムフィルターはハッシュテーブルとの一致回数を減らし、不必要なI/Oやネットワーク転送を避け、結合操作の効率を大幅に向上させることができます。

## ランタイムフィルターの使用

ランタイムフィルターを使用するには、TiFlashのレプリカを持つテーブルを作成し、[`tidb_runtime_filter_mode`](/system-variables.md#tidb_runtime_filter_mode-new-in-v720)を`LOCAL`に設定する必要があります。

TPC-DSデータセットを例に、`catalog_sales`テーブルと`date_dim`テーブルを使用して結合操作の効率を向上させるランタイムフィルターの使用方法を説明します。

### Step 1. 結合するテーブルにTiFlashレプリカを作成

`catalog_sales`テーブルと`date_dim`テーブルそれぞれにTiFlashレプリカを追加します。

```sql
ALTER TABLE catalog_sales SET tiflash REPLICA 1;
ALTER TABLE date_dim SET tiflash REPLICA 1;
```

2つのテーブルのTiFlashレプリカが準備完了になるまで待ちます。つまり、それぞれのレプリカの`AVAILABLE`と`PROGRESS`フィールドが共に`1`になるまでです。

```sql
SELECT * FROM INFORMATION_SCHEMA.TIFLASH_REPLICA WHERE TABLE_NAME='catalog_sales';
+--------------+---------------+----------+---------------+-----------------+-----------+----------+
| TABLE_SCHEMA | TABLE_NAME    | TABLE_ID | REPLICA_COUNT | LOCATION_LABELS | AVAILABLE | PROGRESS |
+--------------+---------------+----------+---------------+-----------------+-----------+----------+
| tpcds50      | catalog_sales |     1055 |             1 |                 |         1 |        1 |
+--------------+---------------+----------+---------------+-----------------+-----------+----------+

SELECT * FROM INFORMATION_SCHEMA.TIFLASH_REPLICA WHERE TABLE_NAME='date_dim';
+--------------+------------+----------+---------------+-----------------+-----------+----------+
| TABLE_SCHEMA | TABLE_NAME | TABLE_ID | REPLICA_COUNT | LOCATION_LABELS | AVAILABLE | PROGRESS |
+--------------+------------+----------+---------------+-----------------+-----------+----------+
| tpcds50      | date_dim   |     1015 |             1 |                 |         1 |        1 |
+--------------+------------+----------+---------------+-----------------+-----------+----------+
```

### Step 2. ランタイムフィルターを有効にする

ランタイムフィルターを有効にするには、システム変数[`tidb_runtime_filter_mode`](/system-variables.md#tidb_runtime_filter_mode-new-in-v720)の値を`LOCAL`に設定します。

```sql
SET tidb_runtime_filter_mode="LOCAL";
```

変更が成功したかどうかを確認します。

```sql
SHOW VARIABLES LIKE "tidb_runtime_filter_mode";
+--------------------------+-------+
| Variable_name            | Value |
+--------------------------+-------+
| tidb_runtime_filter_mode | LOCAL |
+--------------------------+-------+
```

システム変数の値が`LOCAL`であれば、ランタイムフィルターが有効になっています。

### Step 3. クエリを実行する

クエリを実行する前に、`EXPLAIN`文を使用して実行計画を表示し、ランタイムフィルターが効果を発揮しているかどうかを確認します。

```sql
EXPLAIN SELECT cs_ship_date_sk FROM catalog_sales, date_dim
WHERE d_date = '2002-2-01' AND
     cs_ship_date_sk = d_date_sk;
```

ランタイムフィルターが有効になっている場合、対応するランタイムフィルターが`HashJoin`ノードと`TableScan`ノードにマウントされており、ランタイムフィルターが正常に適用されていることを示します。

```
TableFullScan: runtime filter:0[IN] -> tpcds50.catalog_sales.cs_ship_date_sk
HashJoin: runtime filter:0[IN] <- tpcds50.date_dim.d_date_sk |
```

完全なクエリ実行計画は以下のようになります。

```
+----------------------------------------+-------------+--------------+---------------------+-----------------------------------------------------------------------------------------------------------------------------------------------+
| id                                     | estRows     | task         | access object       | operator info                                                                                                                                 |
+----------------------------------------+-------------+--------------+---------------------+-----------------------------------------------------------------------------------------------------------------------------------------------+
| TableReader_53                         | 37343.19    | root         |                     | MppVersion: 1, data:ExchangeSender_52                                                                                                         |
| └─ExchangeSender_52                    | 37343.19    | mpp[tiflash] |                     | ExchangeType: PassThrough                                                                                                                     |
|   └─Projection_51                      | 37343.19    | mpp[tiflash] |                     | tpcds50.catalog_sales.cs_ship_date_sk                                                                                                         |
```
|     └─HashJoin_48                      | 37343.19    | mpp[tiflash] |                     | inner join, equal:[eq(tpcds50.date_dim.d_date_sk, tpcds50.catalog_sales.cs_ship_date_sk)], runtime filter:0[IN] <- tpcds50.date_dim.d_date_sk |
|       ├─ExchangeReceiver_29(Build)     | 1.00        | mpp[tiflash] |                     |                                                                                                                                               |
|       │ └─ExchangeSender_28            | 1.00        | mpp[tiflash] |                     | ExchangeType: Broadcast, Compression: FAST                                                                                                    |
|       │   └─TableFullScan_26           | 1.00        | mpp[tiflash] | table:date_dim      | pushed down filter:eq(tpcds50.date_dim.d_date, 2002-02-01 00:00:00.000000), keep order:false                                                  |
|       └─Selection_31(Probe)            | 71638034.00 | mpp[tiflash] |                     | not(isnull(tpcds50.catalog_sales.cs_ship_date_sk))                                                                                            |
|         └─TableFullScan_30             | 71997669.00 | mpp[tiflash] | table:catalog_sales | pushed down filter:empty, keep order:false, runtime filter:0[IN] -> tpcds50.catalog_sales.cs_ship_date_sk                                     |
+----------------------------------------+-------------+--------------+---------------------+-----------------------------------------------------------------------------------------------------------------------------------------------+
9 rows in set (0.01 sec)

```sql
SELECT cs_ship_date_sk FROM catalog_sales, date_dim
WHERE d_date = '2002-2-01' AND
     cs_ship_date_sk = d_date_sk;
```

### 第4段. パフォーマンス比較

この例では、50GBのTPC-DSデータを使用しています。ランタイムフィルターを有効にした後、クエリの実行時間が0.38秒から0.17秒に短縮され、効率が50%向上しました。ランタイムフィルターが効果を発揮した後の各オペレータの実行時間を表示するには、`ANALYZE` ステートメントを使用できます。

以下は、ランタイムフィルターが無効の場合のクエリの実行情報です:

```sql
EXPLAIN ANALYZE SELECT cs_ship_date_sk FROM catalog_sales, date_dim WHERE d_date = '2002-2-01' AND cs_ship_date_sk = d_date_sk;
+----------------------------------------+-------------+----------+--------------+---------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------------+---------+------+
| id                                     | estRows     | actRows  | task         | access object       | execution info                                                                                                                                                                                                                                                                                                                                                                                    | operator info                                                                                | memory  | disk |
+----------------------------------------+-------------+----------+--------------+---------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------------+---------+------+
| TableReader_53                         | 37343.19    | 59574    | root         |                     | time:379.7ms, loops:83, RU:0.000000, cop_task: {num: 48, max: 0s, min: 0s, avg: 0s, p95: 0s, copr_cache_hit_ratio: 0.00}                                                                                                                                                                                                                                                                          | MppVersion: 1, data:ExchangeSender_52                                                        | 12.0 KB | N/A  |
| └─ExchangeSender_52                    | 37343.19    | 59574    | mpp[tiflash] |                     | tiflash_task:{proc max:377ms, min:375.3ms, avg: 376.1ms, p80:377ms, p95:377ms, iters:1160, tasks:2, threads:16}                                                                                                                                                                                                                                                                                   | ExchangeType: PassThrough                                                                    | N/A     | N/A  |
|   └─Projection_51                      | 37343.19    | 59574    | mpp[tiflash] |                     | tiflash_task:{proc max:377ms, min:375.3ms, avg: 376.1ms, p80:377ms, p95:377ms, iters:1160, tasks:2, threads:16}                                                                                                                                                                                                                                                                                   | tpcds50.catalog_sales.cs_ship_date_sk                                                        | N/A     | N/A  |
|     └─HashJoin_48                      | 37343.19    | 59574    | mpp[tiflash] |                     | tiflash_task:{proc max:377ms, min:375.3ms, avg: 376.1ms, p80:377ms, p95:377ms, iters:1160, tasks:2, threads:16}                                                                                                                                                                                                                                                                                   | inner join, equal:[eq(tpcds50.date_dim.d_date_sk, tpcds50.catalog_sales.cs_ship_date_sk)]    | N/A     | N/A  |
|       ├─ExchangeReceiver_29(Build)     | 1.00        | 2        | mpp[tiflash] |                     | tiflash_task:{proc max:291.3ms, min:290ms, avg: 290.6ms, p80:291.3ms, p95:291.3ms, iters:2, tasks:2, threads:16}                                                                                                                                                                                                                                                                                  |                                                                                              | N/A     | N/A  |
|       │ └─ExchangeSender_28            | 1.00        | 1        | mpp[tiflash] |                     | tiflash_task:{proc max:290.9ms, min:0s, avg: 145.4ms, p80:290.9ms, p95:290.9ms, iters:1, tasks:2, threads:1}                                                                                                                                                                                                                                                                                      | ExchangeType: Broadcast, Compression: FAST                                                   | N/A     | N/A  |
|       │   └─TableFullScan_26           | 1.00        | 1        | mpp[tiflash] | table:date_dim      | tiflash_task:{proc max:3.88ms, min:0s, avg: 1.94ms, p80:3.88ms, p95:3.88ms, iters:1, tasks:2, threads:1}, tiflash_scan:{dtfile:{total_scanned_packs:2, total_skipped_packs:12, total_scanned_rows:16384, total_skipped_rows:97625, total_rs_index_load_time: 0ms, total_read_time: 0ms}, total_create_snapshot_time: 0ms, total_local_region_num: 1, total_remote_region_num: 0}                  | pushed down filter:eq(tpcds50.date_dim.d_date, 2002-02-01 00:00:00.000000), keep order:false | N/A     | N/A  |
|       └─Selection_31(Probe)            | 71638034.00 | 71638034 | mpp[tiflash] |                     | tiflash_task:{proc max:47ms, min:34.3ms, avg: 40.6ms, p80:47ms, p95:47ms, iters:1160, tasks:2, threads:16}                                                                                                                                                                                                                                                                                        | not(isnull(tpcds50.catalog_sales.cs_ship_date_sk))                                           | N/A     | N/A  |
|         └─TableFullScan_30             | 71997669.00 | 71997669 | mpp[tiflash] | table:catalog_sales | tiflash_task:{proc max:34ms, min:17.3ms, avg: 25.6ms, p80:34ms, p95:34ms, iters:1160, tasks:2, threads:16}, tiflash_scan:{dtfile:{total_scanned_packs:8893, total_skipped_packs:4007, total_scanned_rows:72056474, total_skipped_rows:32476901, total_rs_index_load_time: 8ms, total_read_time: 579ms}, total_create_snapshot_time: 0ms, total_local_region_num: 194, total_remote_region_num: 0} | pushed down filter:empty, keep order:false                                                   | N/A     | N/A  |
+----------------------------------------+-------------+----------+--------------+---------------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------------+---------+------+
9 rows in set (0.38 sec)
```

以下は、ランタイムフィルターが有効の場合のクエリの実行情報です:

```sql
EXPLAIN ANALYZE SELECT cs_ship_date_sk FROM catalog_sales, date_dim
    -> WHERE d_date = '2002-2-01' AND
    ->      cs_ship_date_sk = d_date_sk;
+----------------------------------------+-------------+---------+--------------+---------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------+---------+------+
| TableReader_53                                                                                              | 37343.19    | 59574   | root         |                     | time:162.1ms, loops:82, RU:0.000000, cop_task: {num: 47, max: 0s, min: 0s, avg: 0s, p95: 0s, copr_cache_hit_ratio: 0.00}                                                                                                                                                                                                                                                                             | MppVersion: 1, data:ExchangeSender_52                                                                                                         | 12.7 KB | N/A  |
| └─ExchangeSender_52                                                                                          | 37343.19    | 59574   | mpp[tiflash] |                     | tiflash_task:{proc max:160.8ms, min:154.3ms, avg: 157.6ms, p80:160.8ms, p95:160.8ms, iters:86, tasks:2, threads:16}                                                                                                                                                                                                                                                                                  | ExchangeType: PassThrough                                                                                                                     | N/A     | N/A  |
|   └─Projection_51                                                                                            | 37343.19    | 59574   | mpp[tiflash] |                     | tiflash_task:{proc max:160.8ms, min:154.3ms, avg: 157.6ms, p80:160.8ms, p95:160.8ms, iters:86, tasks:2, threads:16}                                                                                                                                                                                                                                                                                  | tpcds50.catalog_sales.cs_ship_date_sk                                                                                                         | N/A     | N/A  |
|     └─HashJoin_48                                                                                            | 37343.19    | 59574   | mpp[tiflash] |                     | tiflash_task:{proc max:160.8ms, min:154.3ms, avg: 157.6ms, p80:160.8ms, p95:160.8ms, iters:86, tasks:2, threads:16}                                                                                                                                                                                                                                                                                  | inner join, equal:[eq(tpcds50.date_dim.d_date_sk, tpcds50.catalog_sales.cs_ship_date_sk)], runtime filter:0[IN] <- tpcds50.date_dim.d_date_sk | N/A     | N/A  |
|       ├─ExchangeReceiver_29(Build)                                                                           | 1.00        | 2       | mpp[tiflash] |                     | tiflash_task:{proc max:132.3ms, min:130.8ms, avg: 131.6ms, p80:132.3ms, p95:132.3ms, iters:2, tasks:2, threads:16}                                                                                                                                                                                                                                                                                   |                                                                                                                                               | N/A     | N/A  |
|       │ └─ExchangeSender_28                                                                                 | 1.00        | 1       | mpp[tiflash] |                     | tiflash_task:{proc max:131ms, min:0s, avg: 65.5ms, p80:131ms, p95:131ms, iters:1, tasks:2, threads:1}                                                                                                                                                                                                                                                                                                | ExchangeType: Broadcast, Compression: FAST                                                                                                    | N/A     | N/A  |
|       │   └─TableFullScan_26                                                                                | 1.00        | 1       | mpp[tiflash] | table:date_dim      | tiflash_task:{proc max:3.01ms, min:0s, avg: 1.51ms, p80:3.01ms, p95:3.01ms, iters:1, tasks:2, threads:1}, tiflash_scan:{dtfile:{total_scanned_packs:2, total_skipped_packs:12, total_scanned_rows:16384, total_skipped_rows:97625, total_rs_index_load_time: 0ms, total_read_time: 0ms}, total_create_snapshot_time: 0ms, total_local_region_num: 1, total_remote_region_num: 0}                     | pushed down filter:eq(tpcds50.date_dim.d_date, 2002-02-01 00:00:00.000000), keep order:false                                                  | N/A     | N/A  |
|       └─Selection_31(Probe)                                                                                 | 71638034.00 | 5308995 | mpp[tiflash] |                     | tiflash_task:{proc max:39.8ms, min:24.3ms, avg: 32.1ms, p80:39.8ms, p95:39.8ms, iters:86, tasks:2, threads:16}                                                                                                                                                                                                                                                                                       | not(isnull(tpcds50.catalog_sales.cs_ship_date_sk))                                                                                            | N/A     | N/A  |
|         └─TableFullScan_30                                                                                  | 71997669.00 | 5335549 | mpp[tiflash] | table:catalog_sales | tiflash_task:{proc max:36.8ms, min:23.3ms, avg: 30.1ms, p80:36.8ms, p95:36.8ms, iters:86, tasks:2, threads:16}, tiflash_scan:{dtfile:{total_scanned_packs:660, total_skipped_packs:12451, total_scanned_rows:5335549, total_skipped_rows:100905778, total_rs_index_load_time: 2ms, total_read_time: 47ms}, total_create_snapshot_time: 0ms, total_local_region_num: 194, total_remote_region_num: 0} | pushed down filter:empty, keep order:false, runtime filter:0[IN] -> tpcds50.catalog_sales.cs_ship_date_sk                                     | N/A     | N/A  |
+-----------------------------------------------+-------------+---------+--------------+---------------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------+---------+------+
9行セット (0.17秒)

両方のクエリの実行情報を比較することで、以下の改善点が見つかります：

* IOの削減：TableFullScanオペレータの`total_scanned_rows`を比較することで、Runtime Filterが有効化された後、`TableFullScan`のスキャン量が3分の2に削減されたことが分かります。
* Hash Joinのパフォーマンス改善：`HashJoin`オペレータの実行時間が376.1ミリ秒から157.6ミリ秒に短縮されました。

### ベストプラクティス

Runtime Filterは、大規模なテーブルと小規模なテーブルが結合されるようなシナリオに適しており、事実テーブルと次元テーブルの結合クエリなどに適用できます。次元テーブルが少量のヒットデータを持つ場合、つまりフィルタが少ない値を持つことを意味し、そのため、ファクトテーブルは条件に適合しないデータをより効果的にフィルタリングできます。従来のシナリオで全体のファクトテーブルがスキャンされる状況と比較すると、これによりクエリのパフォーマンスが大幅に向上します。

TPC-DSの`Sales`テーブルと `date_dim`テーブルの結合操作はその典型的な例です。

## Runtime Filterの設定

Runtime Filterを使用する際は、Runtime Filterのモードと述部タイプを設定できます。

### Runtime Filterモード

Runtime Filterのモードは、**フィルタ送信オペレータ**と**フィルタ受信オペレータ**の関係です。`OFF`、`LOCAL`、`GLOBAL`の3つのモードがあります。v7.3.0では、`OFF`と`LOCAL`モードのみがサポートされます。Runtime Filterのモードはシステム変数[`tidb_runtime_filter_mode`](/system-variables.md#tidb_runtime_filter_mode-new-in-v720)で制御されます。

- `OFF`: Runtime Filterが無効化されます。無効化した後は、クエリの動作は以前のバージョンと同じです。
- `LOCAL`: ローカルモードでRuntime Filterが有効化されます。ローカルモードでは、**フィルタ送信オペレータ**と**フィルタ受信オペレータ**は同じMPPタスクにあります。つまり、Runtime FilterはHashJoinオペレータとTableScanオペレータが同じタスクにあるシナリオに適用できます。現在、Runtime Filterはローカルモードのみをサポートしています。このモードを有効化するには、`LOCAL`に設定してください。
- `GLOBAL`: 現在、グローバルモードはサポートされていません。Runtime Filterをこのモードに設定することはできません。

### Runtime Filterタイプ

Runtime Filterのタイプは、生成されたフィルタオペレータで使用される述部のタイプです。現在、`IN`のみがサポートされています。これは、生成された述部が`k1 in (xxx)`と類似していることを意味します。Runtime Filterのタイプはシステム変数[`tidb_runtime_filter_type`](/system-variables.md#tidb_runtime_filter_type-new-in-v720)で制御されます。

- `IN`: デフォルトのタイプです。生成されたRuntime Filterが`IN`タイプの述部を使用することを意味します。

## 制限

- Runtime FilterはMPPアーキテクチャの最適化であり、TiFlashにプッシュダウンされたクエリにのみ適用できます。
- 結合タイプ: 左外部結合、フル外部結合、アンチ結合 (左側のテーブルがプローブ側の場合) はRuntime Filterをサポートしません。Runtime Filterは結合に関与するデータを事前にフィルタリングするため、これらの結合タイプでは適合しないデータを破棄しませんので、Runtime Filterを使用できません。
- 等しい結合の式: 等しい結合式のプローブ列が複雑な式である場合、またはプローブ列のタイプがJSON、Blob、Array、または他の複雑なデータタイプである場合、Runtime Filterは生成されません。主な理由は、これらの種類の列が結合列としてほとんど使用されないことです。フィルタが生成されても、通常フィルタリング率は低いです。
```
For the preceding limitations, if you need to confirm whether Runtime Filter is generated correctly, you can use the [`EXPLAIN` statement](/sql-statements/sql-statement-explain.md) to verify the execution plan.
```

上記の制限については、ランタイムフィルタが正しく生成されているかどうかを確認する必要がある場合は、[`EXPLAIN`ステートメント](/sql-statements/sql-statement-explain.md)を使用して実行計画を確認できます。