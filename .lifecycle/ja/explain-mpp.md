---
title: MPPモードでのステートメントの説明
summary: TiDBのEXPLAINステートメントによって返される実行計画情報について学ぶ

# MPPモードでのステートメントの説明

TiDBは[MPPモード](/tiflash/use-tiflash-mpp-mode.md)を使用してクエリを実行することをサポートしています。MPPモードでは、TiDBの最適化プログラムはMPP向けの実行計画を生成します。なお、MPPモードは[TiFlash](/tiflash/tiflash-overview.md)にレプリカがあるテーブルにのみ適用できます。

このドキュメントの例は、以下のサンプルデータに基づいています。

{{< copyable "sql" >}}

```sql
CREATE TABLE t1 (id int, value int);
INSERT INTO t1 values(1,2),(2,3),(1,3);
ALTER TABLE t1 set tiflash replica 1;
ANALYZE TABLE t1;
SET tidb_allow_mpp = 1;
```

## MPPクエリフラグメントとMPPタスク

MPPモードでは、クエリは論理的に複数のクエリフラグメントに分割されます。次のステートメントを例に取ると：

{{< copyable "sql" >}}

```sql
EXPLAIN SELECT COUNT(*) FROM t1 GROUP BY id;
```

このクエリはMPPモードでは2つのフラグメントに分かれます。1つは最初の段階の集約のためのもので、もう1つは2段階目の集約および最終的な集約のためのものです。このクエリが実行されると、各クエリフラグメントは1つ以上のMPPタスクに具体化されます。

## Exchange演算子

`ExchangeReceiver`および`ExchangeSender`はMPP実行計画専用の2つのExchange演算子です。 `ExchangeReceiver`演算子は下流のクエリフラグメントからデータを読み取り、`ExchangeSender`演算子は下流のクエリフラグメントから上流のクエリフラグメントにデータを送信します。MPPモードでは、各MPPクエリフラグメントのルート演算子は`ExchangeSender`であり、クエリフラグメントは`ExchangeSender`演算子によって区切られています。

以下は簡単なMPP実行計画です：

{{< copyable "sql" >}}

```sql
EXPLAIN SELECT COUNT(*) FROM t1 GROUP BY id;
```

```sql
+------------------------------------+---------+-------------------+---------------+----------------------------------------------------+
| id                                 | estRows | task              | access object | operator info                                      |
+------------------------------------+---------+-------------------+---------------+----------------------------------------------------+
| TableReader_31                     | 2.00    | root              |               | data:ExchangeSender_30                             |
| └─ExchangeSender_30                | 2.00    | batchCop[tiflash] |               | ExchangeType: PassThrough                          |
|   └─Projection_26                  | 2.00    | batchCop[tiflash] |               | Column#4                                           |
|     └─HashAgg_27                   | 2.00    | batchCop[tiflash] |               | group by:test.t1.id, funcs:sum(Column#7)->Column#4 |
|       └─ExchangeReceiver_29        | 2.00    | batchCop[tiflash] |               |                                                    |
|         └─ExchangeSender_28        | 2.00    | batchCop[tiflash] |               | ExchangeType: HashPartition, Hash Cols: test.t1.id |
|           └─HashAgg_9              | 2.00    | batchCop[tiflash] |               | group by:test.t1.id, funcs:count(1)->Column#7      |
|             └─TableFullScan_25     | 3.00    | batchCop[tiflash] | table:t1      | keep order:false                                   |
+------------------------------------+---------+-------------------+---------------+----------------------------------------------------+
```
|     └─HashJoin_43                      | 9.00    | cop[tiflash] |               | inner join, equal:[eq(test.t1.id, test.t1.id)] |
|       ├─ExchangeReceiver_20(Build)     | 6.00    | cop[tiflash] |               |                                                |
|       │ └─ExchangeSender_19            | 6.00    | cop[tiflash] |               | ExchangeType: Broadcast                        |
|       │   └─Selection_18               | 6.00    | cop[tiflash] |               | not(isnull(test.t1.id))                        |
|       │     └─TableFullScan_17         | 6.00    | cop[tiflash] | table:a       | keep order:false                               |
|       └─Selection_22(Probe)            | 6.00    | cop[tiflash] |               | not(isnull(test.t1.id))                        |
|         └─TableFullScan_21             | 6.00    | cop[tiflash] | table:b       | keep order:false                               |
+----------------------------------------+---------+--------------+---------------+------------------------------------------------+

上記の実行計画において：

* クエリフラグメント `[TableFullScan_17, Selection_18, ExchangeSender_19]` は小さなテーブル（テーブル a）からデータを読み込み、そのデータを大きなテーブル（テーブル b）からデータを含む各ノードにブロードキャストします。
* クエリフラグメント `[TableFullScan_21, Selection_22, ExchangeReceiver_20, HashJoin_43, ExchangeSender_46]` は全てのデータを結合し、TiDB に返します。

## MPPモードにおける `EXPLAIN ANALYZE` ステートメント

`EXPLAIN ANALYZE` ステートメントは `EXPLAIN` に似ていますが、ランタイム情報も出力します。

以下は単純な `EXPLAIN ANALYZE` の例の出力です：

{{< copyable "sql" >}}

```sql
EXPLAIN ANALYZE SELECT COUNT(*) FROM t1 GROUP BY id;
```

```sql
+------------------------------------+---------+---------+-------------------+---------------+---------------------------------------------------------------------------------------------------+----------------------------------------------------------------+--------+------+
| id                                 | estRows | actRows | task              | access object | execution info                                                                                    | operator info                                                  | memory | disk |
+------------------------------------+---------+---------+-------------------+---------------+---------------------------------------------------------------------------------------------------+----------------------------------------------------------------+--------+------+
| TableReader_31                     | 4.00    | 2       | root              |               | time:44.5ms, loops:2, cop_task: {num: 1, max: 0s, proc_keys: 0, copr_cache_hit_ratio: 0.00}       | data:ExchangeSender_30                                         | N/A    | N/A  |
| └─ExchangeSender_30                | 4.00    | 2       | batchCop[tiflash] |               | tiflash_task:{time:16.5ms, loops:1, threads:1}                                                    | ExchangeType: PassThrough, tasks: [2, 3, 4]                    | N/A    | N/A  |
|   └─Projection_26                  | 4.00    | 2       | batchCop[tiflash] |               | tiflash_task:{time:16.5ms, loops:1, threads:1}                                                    | Column#4                                                       | N/A    | N/A  |
|     └─HashAgg_27                   | 4.00    | 2       | batchCop[tiflash] |               | tiflash_task:{time:16.5ms, loops:1, threads:1}                                                    | group by:test.t1.id, funcs:sum(Column#7)->Column#4             | N/A    | N/A  |
|       └─ExchangeReceiver_29        | 4.00    | 2       | batchCop[tiflash] |               | tiflash_task:{time:14.5ms, loops:1, threads:20}                                                   |                                                                | N/A    | N/A  |
|         └─ExchangeSender_28        | 4.00    | 0       | batchCop[tiflash] |               | tiflash_task:{time:9.49ms, loops:0, threads:0}                                                    | ExchangeType: HashPartition, Hash Cols: test.t1.id, tasks: [1] | N/A    | N/A  |
|           └─HashAgg_9              | 4.00    | 0       | batchCop[tiflash] |               | tiflash_task:{time:9.49ms, loops:0, threads:0}                                                    | group by:test.t1.id, funcs:count(1)->Column#7                  | N/A    | N/A  |
|             └─TableFullScan_25     | 6.00    | 0       | batchCop[tiflash] | table:t1      | tiflash_task:{time:9.49ms, loops:0, threads:0}, tiflash_scan:{dtfile:{total_scanned_packs:1,...}} | keep order:false                                               | N/A    | N/A  |
+------------------------------------+---------+---------+-------------------+---------------+---------------------------------------------------------------------------------------------------+----------------------------------------------------------------+--------+------+
```

`EXPLAIN` の出力と比較して、`ExchangeSender` の `operator info` 列には `tasks` も表示されており、これはクエリフラグメントが実体化されるMPPタスクのIDを記録しています。さらに、各MPPオペレータには `execution info` 列で `threads` フィールドがあり、これはTiDBがこのオペレータを実行する際の並列処理を記録しています。クラスタが複数のノードで構成されている場合、この並列処理はすべてのノードの並列処理を合計した結果です。

## MPPバージョンとExchangeデータの圧縮

v6.6.0から、MPP実行計画に新しいフィールド `MPPVersion` と `Compression` が追加されています。

- `MppVersion`: MPP実行計画のバージョン番号で、システム変数 [`mpp_version`](/system-variables.md#mpp_version-new-in-v660) を介して設定できます。
- `Compression`: `Exchange` オペレータのデータ圧縮モードであり、システム変数 [`mpp_exchange_compression_mode`](/system-variables.md#mpp_exchange_compression_mode-new-in-v660) を介して設定できます。データ圧縮が有効になっていない場合、このフィールドは実行計画に表示されません。

次の例を参照してください：

```sql
mysql > EXPLAIN SELECT COUNT(*) AS count_order FROM lineitem GROUP BY l_returnflag, l_linestatus ORDER BY l_returnflag, l_linestatus;

+----------------------------------------+--------------+--------------+----------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| id                                     | estRows      | task         | access object  | operator info                                                                                                                                                                                                                                                                        |
+----------------------------------------+--------------+--------------+----------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Projection_6                           | 3.00         | root         |                | Column#18                                                                                                                                                                                                                                                                            |
| └─Sort_8                               | 3.00         | root         |                | tpch100.lineitem.l_returnflag, tpch100.lineitem.l_linestatus                                                                                                                                                                                                                         |
|   └─TableReader_36                     | 3.00         | root         |                | MppVersion: 1, data:ExchangeSender_35                                                                                                                                                                                                                                                |
|     └─ExchangeSender_35                | 3.00         | mpp[tiflash] |                | ExchangeType: PassThrough                                                                                                                                                                                                                                                            |
|       └─Projection_31                  | 3.00         | mpp[tiflash] |                | Column#18, tpch100.lineitem.l_returnflag, tpch100.lineitem.l_linestatus                                                                                                                                                                                                              |
|         └─HashAgg_32                   | 3.00         | mpp[tiflash] |                | group by:tpch100.lineitem.l_linestatus, tpch100.lineitem.l_returnflag, funcs:sum(Column#23)->Column#18, funcs:firstrow(tpch100.lineitem.l_returnflag)->tpch100.lineitem.l_returnflag, funcs:firstrow(tpch100.lineitem.l_linestatus)->tpch100.lineitem.l_linestatus, stream_count: 20 |
|           └─ExchangeReceiver_34        | 3.00         | mpp[tiflash] |                | stream_count: 20                                                                                                                                                                                                                                                                     |
|             └─ExchangeSender_33        | 3.00         | mpp[tiflash] |                | ExchangeType: HashPartition, Compression: FAST, Hash Cols: [name: tpch100.lineitem.l_returnflag, collate: utf8mb4_bin], [name: tpch100.lineitem.l_linestatus, collate: utf8mb4_bin], stream_count: 20                                                                                |
|               └─HashAgg_14             | 3.00         | mpp[tiflash] |                | group by:tpch100.lineitem.l_linestatus, tpch100.lineitem.l_returnflag, funcs:count(1)->Column#23                                                                                                                                                                                     |
|                 └─TableFullScan_30     | 600037902.00 | mpp[tiflash] | table:lineitem | keep order:false                                                                                                                                                                                                                                                                     |
+----------------------------------------+--------------+--------------+----------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```
```
In the preceding execution plan result, TiDB uses an MPP execution plan of version `1` to build `TableReader`. The `ExchangeSender` operator of the `HashPartition` type uses the `FAST` data compression mode. Data compression is not enabled for the `ExchangeSender` operator of the `PassThrough` type.
```