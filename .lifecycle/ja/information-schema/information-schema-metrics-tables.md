---
title: METRICS_TABLES
summary: `METRICS_TABLES` システムテーブルの使用方法を学ぶ。
aliases: ['/docs/dev/system-tables/system-table-metrics-tables/','/docs/dev/reference/system-databases/metrics-tables/','/tidb/dev/system-table-metrics-tables/']
---

# METRICS_TABLES

`METRICS_TABLES` テーブルは、[`METRICS_SCHEMA`](/metrics-schema.md) データベース内の各ビューの PromQL（Prometheus クエリ言語）定義を提供します。

> **注意:**
>
> このテーブルは、TiDB Self-Hosted にのみ適用され、[TiDB Cloud](https://docs.pingcap.com/tidbcloud/) では利用できません。

```sql
USE INFORMATION_SCHEMA;
DESC METRICS_TABLES;
```

出力は以下のようになります:

```sql
+------------+--------------+------+------+---------+-------+
| Field      | Type         | Null | Key  | Default | Extra |
+------------+--------------+------+------+---------+-------+
| TABLE_NAME | varchar(64)  | YES  |      | NULL    |       |
| PROMQL     | varchar(64)  | YES  |      | NULL    |       |
| LABELS     | varchar(64)  | YES  |      | NULL    |       |
| QUANTILE   | double       | YES  |      | NULL    |       |
| COMMENT    | varchar(256) | YES  |      | NULL    |       |
+------------+--------------+------+------+---------+-------+
```

フィールドの説明:

* `TABLE_NAME`: `METRICS_SCHEMA` 内のテーブル名に対応します。
* `PROMQL`: モニタリングテーブルの動作原理は、SQL 文を `PromQL` にマッピングし、Prometheus の結果を SQL クエリ結果に変換することです。このフィールドは `PromQL` の式テンプレートです。モニタリングテーブルのデータをクエリすると、クエリ条件がこのテンプレート内の変数を書き換え、最終的なクエリ式を生成します。
* `LABELS`: モニタリングアイテムのラベルです。各ラベルは、モニタリングテーブルの列に対応します。SQL 文に対応する列のフィルターが含まれている場合、対応する `PromQL` も変更されます。
* `QUANTILE`: パーセンタイルです。ヒストグラム型の監視データの場合、デフォルトのパーセンタイルが指定されます。このフィールドの値が `0` の場合、監視テーブルに対応する監視アイテムがヒストグラムではないことを意味します。
* `COMMENT`: モニタリングテーブルに関するコメントです。

```sql
SELECT * FROM metrics_tables LIMIT 5\G
```

出力は以下のようになります:

```sql
*************************** 1. row ***************************
TABLE_NAME: abnormal_stores
    PROMQL: sum(pd_cluster_status{ type=~"store_disconnected_count|store_unhealth_count|store_low_space_count|store_down_count|store_offline_count|store_tombstone_count"})
    LABELS: instance,type
  QUANTILE: 0
   COMMENT:
*************************** 2. row ***************************
TABLE_NAME: etcd_disk_wal_fsync_rate
    PROMQL: delta(etcd_disk_wal_fsync_duration_seconds_count{$LABEL_CONDITIONS}[$RANGE_DURATION])
    LABELS: instance
  QUANTILE: 0
   COMMENT: パーシステントストレージに WAL を書き込む速度
*************************** 3. row ***************************
TABLE_NAME: etcd_wal_fsync_duration
    PROMQL: histogram_quantile($QUANTILE, sum(rate(etcd_disk_wal_fsync_duration_seconds_bucket{$LABEL_CONDITIONS}[$RANGE_DURATION])) by (le,instance))
    LABELS: instance
  QUANTILE: 0.99
   COMMENT: パーシステントストレージに WAL を書き込むためにかかる分位時間
*************************** 4. row ***************************
TABLE_NAME: etcd_wal_fsync_total_count
    PROMQL: sum(increase(etcd_disk_wal_fsync_duration_seconds_count{$LABEL_CONDITIONS}[$RANGE_DURATION])) by (instance)
    LABELS: instance
  QUANTILE: 0
   COMMENT: パーシステントストレージに WAL を書き込む総数
*************************** 5. row ***************************
TABLE_NAME: etcd_wal_fsync_total_time
    PROMQL: sum(increase(etcd_disk_wal_fsync_duration_seconds_sum{$LABEL_CONDITIONS}[$RANGE_DURATION])) by (instance)
    LABELS: instance
  QUANTILE: 0
   COMMENT: パーシステントストレージに WAL を書き込む総時間
5 rows in set (0.00 sec)
```