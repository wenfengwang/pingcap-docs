---
title: ディスティンクト最適化
summary: TiDBクエリ最適化における`distinct`最適化を紹介します。

# ディスティンクト最適化

このドキュメントでは、TiDBクエリ最適化における`distinct`最適化について紹介します。これには`SELECT DISTINCT`および集約関数での`DISTINCT`も含まれます。

## `SELECT`文での`DISTINCT`修飾子

`DISTINCT`修飾子は、結果セットから重複する行を削除することを指定します。例えば、`SELECT DISTINCT`は`GROUP BY`に変換されます。

```sql
mysql> explain SELECT DISTINCT a from t;
+--------------------------+---------+-----------+---------------+-------------------------------------------------------+
| id                       | estRows | task      | access object | operator info                                         |
+--------------------------+---------+-----------+---------------+-------------------------------------------------------+
| HashAgg_6                | 2.40    | root      |               | group by:test.t.a, funcs:firstrow(test.t.a)->test.t.a |
| └─TableReader_11         | 3.00    | root      |               | data:TableFullScan_10                                 |
|   └─TableFullScan_10     | 3.00    | cop[tikv] | table:t       | keep order:false, stats:pseudo                        |
+--------------------------+---------+-----------+---------------+-------------------------------------------------------+
3 rows in set (0.00 sec)
```

## 集約関数での`DISTINCT`オプション

通常、`DISTINCT`オプションを持つ集約関数は、TiDBレイヤーで単一スレッドの実行モデルで実行されます。

<CustomContent platform="tidb">

[`tidb_opt_distinct_agg_push_down`](/system-variables.md#tidb_opt_distinct_agg_push_down)システム変数またはTiDBの[`distinct-agg-push-down`](/tidb-configuration-file.md#distinct-agg-push-down)構成項目によって、ディスティンクト集約クエリのリライトとTiKVまたはTiFlashコプロセッサへのプッシュを制御します。

</CustomContent>

<CustomContent platform="tidb-cloud">

TiDBの[`tidb_opt_distinct_agg_push_down`](/system-variables.md#tidb_opt_distinct_agg_push_down)システム変数によって、ディスティンクト集約クエリのリライトとTiKVまたはTiFlashコプロセッサへのプッシュを制御します。

</CustomContent>

この最適化の例として以下のクエリを取り上げます。`tidb_opt_distinct_agg_push_down`はデフォルトでは無効になっており、これにより集約関数はTiDBレイヤーで実行されます。この最適化を有効にして値を`1`に設定すると、`count(distinct a)`の`distinct a`部分がTiKVまたはTiFlashコプロセッサにプッシュされます。
TiKVコプロセッサには列aの重複値を削除するためのHashAgg_5があります。これによりTiDBレイヤーの`HashAgg_8`の計算オーバーヘッドが削減される可能性があります。

```sql
mysql> desc select count(distinct a) from test.t;
+-------------------------+----------+-----------+---------------+------------------------------------------+
| id                      | estRows  | task      | access object | operator info                            |
+-------------------------+----------+-----------+---------------+------------------------------------------+
| StreamAgg_6             | 1.00     | root      |               | funcs:count(distinct test.t.a)->Column#4 |
| └─TableReader_10        | 10000.00 | root      |               | data:TableFullScan_9                     |
|   └─TableFullScan_9     | 10000.00 | cop[tikv] | table:t       | keep order:false, stats:pseudo           |
+-------------------------+----------+-----------+---------------+------------------------------------------+
3 rows in set (0.01 sec)

mysql> set session tidb_opt_distinct_agg_push_down = 1;
Query OK, 0 rows affected (0.00 sec)

mysql> desc select count(distinct a) from test.t;
+---------------------------+----------+-----------+---------------+------------------------------------------+
| id                        | estRows  | task      | access object | operator info                            |
+---------------------------+----------+-----------+---------------+------------------------------------------+
| HashAgg_8                 | 1.00     | root      |               | funcs:count(distinct test.t.a)->Column#3 |
| └─TableReader_9           | 1.00     | root      |               | data:HashAgg_5                           |
|   └─HashAgg_5             | 1.00     | cop[tikv] |               | group by:test.t.a,                       |
|     └─TableFullScan_7     | 10000.00 | cop[tikv] | table:t       | keep order:false, stats:pseudo           |
+---------------------------+----------+-----------+---------------+------------------------------------------+
4 rows in set (0.00 sec)
```