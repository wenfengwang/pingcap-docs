---
title: メトリクススキーマ
summary: `METRICS_SCHEMA` スキーマの学習。
aliases: ['/docs/dev/system-tables/system-table-metrics-schema/','/docs/dev/reference/system-databases/metrics-schema/','/tidb/dev/system-table-metrics-schema/']
---

# メトリクススキーマ

`METRICS_SCHEMA` は、Prometheus に保存されている TiDB のメトリクスの上にあるビューのセットです。各テーブルの PromQL（Prometheus Query Language）のソースは [`INFORMATION_SCHEMA.METRICS_TABLES`](/information-schema/information-schema-metrics-tables.md) で確認できます。

{{< copyable "sql" >}}

```sql
USE metrics_schema;
SELECT * FROM uptime;
SELECT * FROM information_schema.metrics_tables WHERE table_name='uptime'\G
```

```sql
+----------------------------+-----------------+------------+--------------------+
| time                       | instance        | job        | value              |
+----------------------------+-----------------+------------+--------------------+
| 2020-07-06 15:26:26.203000 | 127.0.0.1:10080 | tidb       | 123.60300016403198 |
| 2020-07-06 15:27:26.203000 | 127.0.0.1:10080 | tidb       | 183.60300016403198 |
| 2020-07-06 15:26:26.203000 | 127.0.0.1:20180 | tikv       | 123.60300016403198 |
| 2020-07-06 15:27:26.203000 | 127.0.0.1:20180 | tikv       | 183.60300016403198 |
| 2020-07-06 15:26:26.203000 | 127.0.0.1:2379  | pd         | 123.60300016403198 |
| 2020-07-06 15:27:26.203000 | 127.0.0.1:2379  | pd         | 183.60300016403198 |
| 2020-07-06 15:26:26.203000 | 127.0.0.1:9090  | prometheus | 123.72300004959106 |
| 2020-07-06 15:27:26.203000 | 127.0.0.1:9090  | prometheus | 183.72300004959106 |
+----------------------------+-----------------+------------+--------------------+
8 rows in set (0.00 sec)

*************************** 1. row ***************************
TABLE_NAME: uptime
    PROMQL: (time() - process_start_time_seconds{$LABEL_CONDITIONS})
    LABELS: instance,job
  QUANTILE: 0
   COMMENT: TiDB uptime since last restart(second)
1 row in set (0.00 sec)
```

{{< copyable "sql" >}}

```sql
SHOW TABLES;
```

```sql
+---------------------------------------------------+
| Tables_in_metrics_schema                          |
+---------------------------------------------------+
| abnormal_stores                                   |
| etcd_disk_wal_fsync_rate                          |
| etcd_wal_fsync_duration                           |
| etcd_wal_fsync_total_count                        |
| etcd_wal_fsync_total_time                         |
| go_gc_count                                       |
| go_gc_cpu_usage                                   |
| go_gc_duration                                    |
| go_heap_mem_usage                                 |
| go_threads                                        |
| goroutines_count                                  |
| node_cpu_usage                                    |
| node_disk_available_size                          |
| node_disk_io_util                                 |
| node_disk_iops                                    |
| node_disk_read_latency                            |
| node_disk_size                                    |
..
| tikv_storage_async_request_total_time             |
| tikv_storage_async_requests                       |
| tikv_storage_async_requests_total_count           |
| tikv_storage_command_ops                          |
| tikv_store_size                                   |
| tikv_thread_cpu                                   |
| tikv_thread_nonvoluntary_context_switches         |
| tikv_thread_voluntary_context_switches            |
| tikv_threads_io                                   |
| tikv_threads_state                                |
| tikv_total_keys                                   |
| tikv_wal_sync_duration                            |
| tikv_wal_sync_max_duration                        |
| tikv_worker_handled_tasks                         |
| tikv_worker_handled_tasks_total_num               |
| tikv_worker_pending_tasks                         |
| tikv_worker_pending_tasks_total_num               |
| tikv_write_stall_avg_duration                     |
| tikv_write_stall_max_duration                     |
| tikv_write_stall_reason                           |
| up                                                |
| uptime                                            |
+---------------------------------------------------+
626 rows in set (0.00 sec)
```

`METRICS_SCHEMA` は、[`metrics_summary`](/information-schema/information-schema-metrics-summary.md)、[`metrics_summary_by_label`](/information-schema/information-schema-metrics-summary.md)、[`inspection_summary`](/information-schema/information-schema-inspection-summary.md) などのモニタリングに関連するサマリーテーブルのデータソースとして使用されます。

## 追加の例

`metrics_schema` 内の `tidb_query_duration` 監視テーブルを例として取り上げ、この監視テーブルの使用方法と仕組みについて説明します。その他の監視テーブルの動作原理も、`tidb_query_duration` に類似しています。

`information_schema.metrics_tables` で `tidb_query_duration` テーブルに関連する情報をクエリします。

{{< copyable "sql" >}}

```sql
SELECT * FROM information_schema.metrics_tables WHERE table_name='tidb_query_duration';
```

```sql
+---------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------+----------+----------------------------------------------+
| TABLE_NAME          | PROMQL                                                                                                                                                   | LABELS            | QUANTILE | COMMENT                                      |
+---------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------+----------+----------------------------------------------+
| tidb_query_duration | histogram_quantile($QUANTILE, sum(rate(tidb_server_handle_query_duration_seconds_bucket{$LABEL_CONDITIONS}[$RANGE_DURATION])) by (le,sql_type,instance)) | instance,sql_type | 0.9      | The quantile of TiDB query durations(second) |
+---------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------+-------------------+----------+----------------------------------------------+
```

フィールドの説明:

* `TABLE_NAME`: `metrics_schema` 内のテーブル名に対応します。この例ではテーブル名は `tidb_query_duration` です。
* `PROMQL`: 監視テーブルの動作原理は、まず SQL ステートメントを `PromQL` にマップし、次に Prometheus からデータをリクエストし、Prometheus の結果を SQL クエリの結果に変換することです。このフィールドは `PromQL` の式テンプレートです。監視テーブルのデータをクエリする際に、クエリ条件がこのテンプレート内の変数を書き換えて最終的なクエリ式を生成することに使われます。
* `LABELS`: 監視項目のラベル。たとえば `tidb_query_duration` は、`instance` および `sql_type` の2つのラベルを持っています。
* `QUANTILE`: パーセンタイル。ヒストグラム型の監視データに対しては、デフォルトのパーセンタイルが指定されています。このフィールドの値が `0` の場合、監視テーブルに対応する監視項目はヒストグラムではないことを表します。
* `COMMENT`: 監視テーブルの説明。`tidb_query_duration` テーブルは、TiDB クエリの実行時間（P999/P99/P90 などのパーセンタイル時間などを含む）をクエリするために使用されることがわかります。単位は秒です。

`tidb_query_duration` テーブルのスキーマをクエリするには、次のステートメントを実行します。

{{< copyable "sql" >}}

```sql
SHOW CREATE TABLE metrics_schema.tidb_query_duration;
```

```sql
+---------------------+--------------------------------------------------------------------------------------------------------------------+
| Table               | Create Table                                                                                                       |
+---------------------+--------------------------------------------------------------------------------------------------------------------+
| tidb_query_duration | CREATE TABLE `tidb_query_duration` (                                                                               |
|                     |   `time` datetime unsigned DEFAULT CURRENT_TIMESTAMP,                                                              |
|                     |   `instance` varchar(512) DEFAULT NULL,                                                                            |
|                     |   `sql_type` varchar(512) DEFAULT NULL,                                                                            |
|                     |   `quantile` double unsigned DEFAULT '0.9',                                                                        |
|                     |   `value` double unsigned DEFAULT NULL                                                                             |
|                     | ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin COMMENT='The quantile of TiDB query durations(second)' |
+---------------------+--------------------------------------------------------------------------------------------------------------------+
```

* `time`: 監視項目の時間。
* `instance` および `sql_type`: `tidb_query_duration` 監視項目のラベル。`instance` は監視アドレスを意味します。`sql_type` は実行された SQL ステートメントのタイプを意味します。
* `quantile`: パーセンタイル。ヒストグラム型の監視項目にはこの列があります。これはクエリのパーセンタイル時間を示します。たとえば、`quantile = 0.9` は P90 の時間をクエリすることを意味します。
* `value`: 監視項目の値。

下記のステートメントは、[`2020-03-25 23:40:00`, `2020-03-25 23:42:00`] の範囲内で P99 の時間をクエリします。

{{< copyable "sql" >}}

```sql
SELECT * FROM metrics_schema.tidb_query_duration WHERE value is not null AND time>='2020-03-25 23:40:00' AND time <= '2020-03-25 23:42:00' AND quantile=0.99;
```

```sql
+---------------------+-------------------+----------+----------+----------------+
| time                | インスタンス         | sql_type | quantile | value          |
+---------------------+-------------------+----------+----------+----------------+
| 2020-03-25 23:40:00 | 172.16.5.40:10089 | Insert   | 0.99     | 0.509929485256 |
| 2020-03-25 23:41:00 | 172.16.5.40:10089 | Insert   | 0.99     | 0.494690793986 |
| 2020-03-25 23:42:00 | 172.16.5.40:10089 | Insert   | 0.99     | 0.493460506934 |
| 2020-03-25 23:40:00 | 172.16.5.40:10089 | Select   | 0.99     | 0.152058493415 |
| 2020-03-25 23:41:00 | 172.16.5.40:10089 | Select   | 0.99     | 0.152193879678 |
| 2020-03-25 23:42:00 | 172.16.5.40:10089 | Select   | 0.99     | 0.140498483232 |
| 2020-03-25 23:40:00 | 172.16.5.40:10089 | internal | 0.99     | 0.47104        |
| 2020-03-25 23:41:00 | 172.16.5.40:10089 | internal | 0.99     | 0.11776        |
| 2020-03-25 23:42:00 | 172.16.5.40:10089 | internal | 0.99     | 0.11776        |
+---------------------+-------------------+----------+----------+----------------+
```markdown
    | 2020-03-25 23:42:00 | 172.16.5.40:10089 | internal | 0.99 | 0.0629333333333 |
    +---------------------+-------------------+----------+----------+-----------------+
    ```

3. 実行計画を表示します。結果から見ると、実行計画の`PromQL`と`step`の値が30秒に変更されたことも確認できます。

    {{< copyable "sql" >}}

    ```sql
    desc select * from metrics_schema.tidb_query_duration where value is not null and time>='2020-03-25 23:40:00' and time <= '2020-03-25 23:42:00' and quantile=0.99;
    ```

    ```sql
    +------------------+----------+------+---------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    | id               | estRows  | task | access object             | operator info                                                                                                                                                                                         |
    +------------------+----------+------+---------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    | Selection_5      | 8000.00  | root |                           | not(isnull(Column#5))                                                                                                                                                                                 |
    | └─MemTableScan_6 | 10000.00 | root | table:tidb_query_duration | PromQL:histogram_quantile(0.99, sum(rate(tidb_server_handle_query_duration_seconds_bucket{}[30s])) by (le,sql_type,instance)), start_time:2020-03-25 23:40:00, end_time:2020-03-25 23:42:00, step:30s |
    +------------------+----------+------+---------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    ```
