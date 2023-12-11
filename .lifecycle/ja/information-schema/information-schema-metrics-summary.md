---
title: METRICS_SUMMARY
summary: METRICS_SUMMARY システムテーブルについて学ぶ。
aliases: ['/docs/dev/system-tables/system-table-metrics-summary/','/docs/dev/reference/system-databases/metrics-summary/','/tidb/dev/system-table-metrics-summary']
---

# METRICS_SUMMARY

TiDBクラスタには、多くのモニタリングメトリクスがあります。特異なモニタリングメトリクスを検出しやすくするために、TiDB 4.0 では以下の2つのモニタリングサマリーテーブルが導入されています。

* `information_schema.metrics_summary`
* `information_schema.metrics_summary_by_label`

> **ノート:**
>
> 上記の2つのモニタリングサマリーテーブルは、TiDB Self-Hostedにのみ適用され、[TiDB Cloud](https://docs.pingcap.com/tidbcloud/)では利用できません。

これらの2つのテーブルは、すべてのモニタリングデータをまとめて、各モニタリングメトリクスを効率的に確認するためのものです。`information_schema.metrics_summary`と比較して、`information_schema.metrics_summary_by_label`テーブルには追加の`label`列があり、異なるラベルごとに統計を実行します。

{{< copyable "sql" >}}

```sql
USE information_schema;
DESC metrics_summary;
```

```sql
+--------------+--------------+------+------+---------+-------+
| Field        | Type         | Null | Key  | Default | Extra |
+--------------+--------------+------+------+---------+-------+
| METRICS_NAME | varchar(64)  | YES  |      | NULL    |       |
| QUANTILE     | double       | YES  |      | NULL    |       |
| SUM_VALUE    | double(22,6) | YES  |      | NULL    |       |
| AVG_VALUE    | double(22,6) | YES  |      | NULL    |       |
| MIN_VALUE    | double(22,6) | YES  |      | NULL    |       |
| MAX_VALUE    | double(22,6) | YES  |      | NULL    |       |
| COMMENT      | varchar(256) | YES  |      | NULL    |       |
+--------------+--------------+------+------+---------+-------+
7 rows in set (0.00 sec)
```

フィールドの説明:

* `METRICS_NAME`: モニタリングテーブルの名前。
* `QUANTILE`: パーセンタイル。SQLステートメントを使用して`QUANTILE`を指定できます。例:
    * `select * from metrics_summary where quantile=0.99` は、0.99パーセンタイルのデータを表示することを指定します。
    * `select * from metrics_summary where quantile in (0.80, 0.90, 0.99, 0.999)` は、0.8、0.90、0.99、0.999パーセンタイルのデータを同時に表示することを指定します。
* `SUM_VALUE`、`AVG_VALUE`、`MIN_VALUE`、`MAX_VALUE`はそれぞれ合計値、平均値、最小値、最大値を意味します。
* `COMMENT`: 対応するモニタリングテーブルのコメントです。

例:

TiDBクラスタ内で平均実行時間が最も長い3つの監視項目グループを、`'2020-03-08 13:23:00', '2020-03-08 13:33:00'`時間範囲で問い合わせるには、`information_schema.metrics_summary`テーブルを直接問い合わせ、`/*+ time_range() */`ヒントで時間範囲を指定します。SQLステートメントは次のようになります。

{{< copyable "sql" >}}

```sql
SELECT /*+ time_range('2020-03-08 13:23:00','2020-03-08 13:33:00') */ *
FROM information_schema.metrics_summary
WHERE metrics_name LIKE 'tidb%duration'
 AND avg_value > 0
 AND quantile = 0.99
ORDER BY avg_value DESC
LIMIT 3\G
```

```sql
***************************[ 1. row ]***************************
METRICS_NAME | tidb_get_token_duration
QUANTILE     | 0.99
SUM_VALUE    | 8.972509
AVG_VALUE    | 0.996945
MIN_VALUE    | 0.996515
MAX_VALUE    | 0.997458
COMMENT      |  The quantile of Duration (us) for getting token, it should be small until concurrency limit is reached(second)
***************************[ 2. row ]***************************
METRICS_NAME | tidb_query_duration
QUANTILE     | 0.99
SUM_VALUE    | 0.269079
AVG_VALUE    | 0.007272
MIN_VALUE    | 0.000667
MAX_VALUE    | 0.01554
COMMENT      | The quantile of TiDB query durations(second)
***************************[ 3. row ]***************************
METRICS_NAME | tidb_kv_request_duration
QUANTILE     | 0.99
SUM_VALUE    | 0.170232
AVG_VALUE    | 0.004601
MIN_VALUE    | 0.000975
MAX_VALUE    | 0.013
COMMENT      | The quantile of kv requests durations by store
```

同様に、以下の例は`metrics_summary_by_label`モニタリングサマリーテーブルを問い合わせています:

{{< copyable "sql" >}}

```sql
SELECT /*+ time_range('2020-03-08 13:23:00','2020-03-08 13:33:00') */ *
FROM information_schema.metrics_summary_by_label
WHERE metrics_name LIKE 'tidb%duration'
 AND avg_value > 0
 AND quantile = 0.99
ORDER BY avg_value DESC
LIMIT 10\G
```

```sql
***************************[ 1. row ]***************************
INSTANCE     | 172.16.5.40:10089
METRICS_NAME | tidb_get_token_duration
LABEL        |
QUANTILE     | 0.99
SUM_VALUE    | 8.972509
AVG_VALUE    | 0.996945
MIN_VALUE    | 0.996515
MAX_VALUE    | 0.997458
COMMENT      |  The quantile of Duration (us) for getting token, it should be small until concurrency limit is reached(second)
***************************[ 2. row ]***************************
INSTANCE     | 172.16.5.40:10089
METRICS_NAME | tidb_query_duration
LABEL        | Select
QUANTILE     | 0.99
SUM_VALUE    | 0.072083
AVG_VALUE    | 0.008009
MIN_VALUE    | 0.007905
MAX_VALUE    | 0.008241
COMMENT      | The quantile of TiDB query durations(second)
***************************[ 3. row ]***************************
INSTANCE     | 172.16.5.40:10089
METRICS_NAME | tidb_query_duration
LABEL        | Rollback
QUANTILE     | 0.99
SUM_VALUE    | 0.072083
AVG_VALUE    | 0.008009
MIN_VALUE    | 0.007905
MAX_VALUE    | 0.008241
COMMENT      | The quantile of TiDB query durations(second)
```

上記のクエリ結果の2行目と3行目は、`tidb_query_duration`の`Select`および`Rollback`ステートメントが長い平均実行時間を持っていることを示しています。

上記の例に加えて、2つの期間の全リンク監視項目を比較し、最大の変化を持つモジュールをすばやく見つけ、ボトルネックを素早く特定するために、モニタリングサマリーテーブルを使用することができます。以下の例では、次の2つの期間のすべての監視項目を比較します（t1がベースラインです）。

* 期間t1: `("2020-03-03 17:08:00", "2020-03-03 17:11:00")`
* 期間t2: `("2020-03-03 17:18:00", "2020-03-03 17:21:00")`

2つの期間の監視項目は`METRICS_NAME`に従って結合され、差分値に従って並べ替えられます。`TIME_RANGE`は、クエリ時間を指定するヒントです。

{{< copyable "sql" >}}

```sql
SELECT GREATEST(t1.avg_value,t2.avg_value)/LEAST(t1.avg_value,
         t2.avg_value) AS ratio,
         t1.metrics_name,
         t1.avg_value as t1_avg_value,
         t2.avg_value as t2_avg_value,
         t2.comment
FROM
    (SELECT /*+ time_range("2020-03-03 17:08:00", "2020-03-03 17:11:00")*/ *
    FROM information_schema.metrics_summary ) t1
JOIN
    (SELECT /*+ time_range("2020-03-03 17:18:00", "2020-03-03 17:21:00")*/ *
    FROM information_schema.metrics_summary ) t2
    ON t1.metrics_name = t2.metrics_name
ORDER BY ratio DESC LIMIT 10;
```

```sql
+----------------+------------------------------------------+----------------+------------------+---------------------------------------------------------------------------------------------+
| ratio          | metrics_name                             | t1_avg_value   | t2_avg_value     | comment                                                                                     |
+----------------+------------------------------------------+----------------+------------------+---------------------------------------------------------------------------------------------+
| 5865.59537065  | tidb_slow_query_cop_process_total_time   |       0.016333 |        95.804724 | TiDBスロークエリの統計情報の総時間とスロークエリの総cop処理時間(秒) |
| 3648.74109023  | tidb_distsql_partial_scan_key_total_num  |   10865.666667 |  39646004.4394   | distsql部分スキャンキー数の合計数                                          |
|  267.002351165 | tidb_slow_query_cop_wait_total_time      |       0.003333 |         0.890008 | TiDBスロークエリの統計情報の総時間とスロークエリの総cop待機時間(秒)    |
|  192.43267836  | tikv_cop_total_response_total_size       | 2515333.66667  | 484032394.445    |                                                                             |
|  192.43267836  | tikv_cop_total_response_size_per_seconds |   41922.227778 |   8067206.57408  |                                                                             |
|  152.780296296 | tidb_distsql_scan_key_total_num          |    5304.333333 |    810397.618317 | distsqlスキャン数の合計                                                |
|  126.042290167 | tidb_distsql_execution_total_time        |       0.421622 |        53.142143 | distsql実行の合計時間(秒)                                             |
|  105.164020657 | tikv_cop_scan_details                    |     134.450733 |     14139.379665 |                                                                             |
|  105.164020657 | tikv_cop_scan_details_total              |    8067.043981 |    848362.77991  |                                                                             |
|  101.635495394 | tikv_cop_scan_keys_num                   |    1070.875    |    108838.91113  |                                                                             |
+----------------+------------------------------------------+----------------+------------------+-----------------------------------------------------------------------------+