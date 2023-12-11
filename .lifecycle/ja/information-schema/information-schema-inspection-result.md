---
title: INSPECTION_RESULT
summary: `INSPECTION_RESULT`のダイアグノスティック結果テーブルを学びます。
aliases: ['/docs/dev/system-tables/system-table-inspection-result/','/docs/dev/reference/system-databases/inspection-result/','/tidb/dev/system-table-inspection-result/']
---

# INSPECTION_RESULT

TiDBには、システム内の障害や隠れた問題を検出するための組み込みのダイアグノスティックルールがいくつかあります。

`INSPECTION_RESULT`ダイアグノスティックテーブルは、問題を迅速に見つけて反復作業を減らすのに役立ちます。 `select * from information_schema.inspection_result` ステートメントを使用して内部のダイアグノスティクスをトリガーできます。

> **注意:**
>
> このテーブルはTiDB Self-Hostedにのみ適用され、[TiDB Cloud](https://docs.pingcap.com/tidbcloud/)では使用できません。

`information_schema.inspection_result`ダイアグノスティック結果テーブル `information_schema.inspection_result` の構造は次のようになっています：

{{< copyable "sql" >}}

```sql
USE information_schema;
DESC inspection_result;
```

```sql
+----------------+--------------+------+------+---------+-------+
| Field          | Type         | Null | Key  | Default | Extra |
+----------------+--------------+------+------+---------+-------+
| RULE           | varchar(64)  | YES  |      | NULL    |       |
| ITEM           | varchar(64)  | YES  |      | NULL    |       |
| TYPE           | varchar(64)  | YES  |      | NULL    |       |
| INSTANCE       | varchar(64)  | YES  |      | NULL    |       |
| STATUS_ADDRESS | varchar(64)  | YES  |      | NULL    |       |
| VALUE          | varchar(64)  | YES  |      | NULL    |       |
| REFERENCE      | varchar(64)  | YES  |      | NULL    |       |
| SEVERITY       | varchar(64)  | YES  |      | NULL    |       |
| DETAILS        | varchar(256) | YES  |      | NULL    |       |
+----------------+--------------+------+------+---------+-------+
9 rows in set (0.00 sec)
```

フィールドの説明:

* `RULE`: ダイアグノスティックルールの名前。現在、次のルールが利用可能です：
    * `config`: 構成が一貫しており適切かどうかをチェックします。同じ構成が異なるインスタンスで不一致の場合、「warning」のダイアグノスティック結果が生成されます。
    * `version`: バージョンの一貫性チェック。同じバージョンが異なるインスタンスで不一致の場合、「warning」のダイアグノスティック結果が生成されます。
    * `node-load`: サーバーロードをチェックします。現在のシステム負荷が高すぎる場合、対応する「warning」ダイアグノスティック結果が生成されます。
    * `critical-error`: システムの各モジュールが致命的なエラーを定義します。致命的なエラーが対応する期間内で閾値を超えると、警告のダイアグノスティック結果が生成されます。
    * `threshold-check`: ダイアグノスティックシステムが主要なメトリクスの閾値をチェックします。閾値を超えると、対応するダイアグノスティック情報が生成されます。
* `ITEM`: 各ルールは異なる項目を診断します。このフィールドは、各ルールに対応する特定の診断項目を示します。
* `TYPE`: ダイアグノスティックのインスタンスタイプ。オプションの値は `tidb`、`pd`、`tikv` です。
* `INSTANCE`: 診断されたインスタンスの具体的なアドレス。
* `STATUS_ADDRESS`: インスタンスのHTTP APIサービスアドレス。
* `VALUE`: 特定の診断アイテムの値。
* `REFERENCE`: この診断アイテムの参照値（閾値）です。`VALUE`が閾値を超える場合、対応する診断情報が生成されます。
* `SEVERITY`: 重大度レベル。オプションの値は `warning` と `critical` です。
* `DETAILS`: 診断の詳細であり、追加の診断のための SQL ステートメントまたはドキュメントリンクを含む場合があります。

## ダイアグノスティックの例

クラスタ内に現在存在する問題を診断します。

{{< copyable "sql" >}}

```sql
SELECT * FROM information_schema.inspection_result\G
```

```sql
***************************[ 1. row ]***************************
RULE      | config
ITEM      | log.slow-threshold
TYPE      | tidb
INSTANCE  | 172.16.5.40:4000
VALUE     | 0
REFERENCE | not 0
SEVERITY  | warning
DETAILS   | slow-threshold = 0 will record every query to slow log, it may affect performance
***************************[ 2. row ]***************************
RULE      | version
ITEM      | git_hash
TYPE      | tidb
INSTANCE  |
VALUE     | inconsistent
REFERENCE | consistent
SEVERITY  | critical
DETAILS   | the cluster has 2 different tidb version, execute the sql to see more detail: select * from information_schema.cluster_info where type='tidb'
***************************[ 3. row ]***************************
RULE      | threshold-check
ITEM      | storage-write-duration
TYPE      | tikv
INSTANCE  | 172.16.5.40:23151
VALUE     | 130.417
REFERENCE | < 0.100
SEVERITY  | warning
DETAILS   | max duration of 172.16.5.40:23151 tikv storage-write-duration was too slow
***************************[ 4. row ]***************************
RULE      | threshold-check
ITEM      | rocksdb-write-duration
TYPE      | tikv
INSTANCE  | 172.16.5.40:20151
VALUE     | 108.105
REFERENCE | < 0.100
SEVERITY  | warning
DETAILS   | max duration of 172.16.5.40:20151 tikv rocksdb-write-duration was too slow
```

上記の診断結果から、次の問題が検出されます：

* 最初の行は、TiDBの`log.slow-threshold`の値が `0` に設定されており、パフォーマンスに影響を与える可能性があります。
* 2番目の行は、クラスタ内に異なるTiDBバージョンが2つ存在することを示しています。
* 3番目と4番目の行は、TiKVの書き込み遅延が長すぎることを示しています。期待される遅延は0.1秒未満ですが、実際の遅延は期待よりもはるかに長いです。

指定された範囲内に存在する問題を診断することもできます、例えば "2020-03-26 00:03:00" から "2020-03-26 00:08:00" までです。時間範囲を指定するには、 `/*+ time_range() */` のSQL Hint を使用します。以下のクエリ例を参照してください：

{{< copyable "sql" >}}

```sql
select /*+ time_range("2020-03-26 00:03:00", "2020-03-26 00:08:00") */ * from information_schema.inspection_result\G
```

```sql
***************************[ 1. row ]***************************
RULE      | critical-error
ITEM      | server-down
TYPE      | tidb
INSTANCE  | 172.16.5.40:4009
VALUE     |
REFERENCE |
SEVERITY  | critical
DETAILS   | tidb 172.16.5.40:4009 restarted at time '2020/03/26 00:05:45.670'
***************************[ 2. row ]***************************
RULE      | threshold-check
ITEM      | get-token-duration
TYPE      | tidb
INSTANCE  | 172.16.5.40:10089
VALUE     | 0.234
REFERENCE | < 0.001
SEVERITY  | warning
DETAILS   | max duration of 172.16.5.40:10089 tidb get-token-duration is too slow
```

上記の診断結果から、次の問題が検出されます：

* 最初の行は、 `172.16.5.40:4009` TiDBインスタンスが `2020/03/26 00:05:45.670` に再起動されたことを示しています。
* 2番目の行は、 `172.16.5.40:10089` TiDBインスタンスの最大 `get-token-duration` 時間が0.234秒であり、期待される時間よりも0.001秒未満です。

条件を指定することもできます。例えば、`critical` レベルのダイアグノスティック結果のみをクエリする：

{{< copyable "sql" >}}

```sql
select * from information_schema.inspection_result where severity='critical';
```

`critical-error` ルールのダイアグノスティック結果のみをクエリする：

{{< copyable "sql" >}}

```sql
select * from information_schema.inspection_result where rule='critical-error';
```

## ダイアグノスティックルール

ダイアグノスティックモジュールには、一連のルールが含まれています。これらのルールは、既存の監視テーブルやクラスタ情報テーブルをクエリした後に、結果を閾値と比較します。結果が閾値を超えると、`warning` または `critical` のダイアグノスティックが生成され、`details` 列に対応する情報が提供されます。

`inspection_rules` システムテーブルをクエリすることにより、既存のダイアグノスティックルールを照会できます：

{{< copyable "sql" >}}

```sql
select * from information_schema.inspection_rules where type='inspection';
```

```sql
+-----------------+------------+---------+
| NAME            | TYPE       | COMMENT |
+-----------------+------------+---------+
| config          | inspection |         |
| version         | inspection |         |
| node-load       | inspection |         |
| critical-error  | inspection |         |
| threshold-check | inspection |         |
+-----------------+------------+---------+
```

### `config` ダイアグノスティックルール
```markdown
In the `config` diagnostic rule, the following two diagnostic rules are executed by querying the `CLUSTER_CONFIG` system table:

* Check whether the configuration values of the same component are consistent. Not all configuration items has this consistency check. The allowlist of consistency check is as follows:

```go
// The allowlist of the TiDB configuration consistency check
port
status.status-port
host
path
advertise-address
status.status-port
log.file.filename
log.slow-query-file
tmp-storage-path

// The allowlist of the PD configuration consistency check
advertise-client-urls
advertise-peer-urls
client-urls
data-dir
log-file
log.file.filename
metric.job
name
peer-urls

// The allowlist of the TiKV configuration consistency check
server.addr
server.advertise-addr
server.status-addr
log-file
raftstore.raftdb-path
storage.data-dir
storage.block-cache.capacity
```

* Check whether the values of the following configuration items are as expected.

|  Component  | Configuration item | Expected value |
|  ----  | ----  |  ----  |
| TiDB | log.slow-threshold | larger than `0` |
| TiKV | raftstore.sync-log | `true` |

### `version` diagnostic rule

The `version` diagnostic rule checks whether the version hash of the same component is consistent by querying the `CLUSTER_INFO` system table. See the following example:

{{< copyable "sql" >}}

```sql
SELECT * FROM information_schema.inspection_result WHERE rule='version'\G
```

```sql
***************************[ 1. row ]***************************
RULE      | version
ITEM      | git_hash
TYPE      | tidb
INSTANCE  |
VALUE     | inconsistent
REFERENCE | consistent
SEVERITY  | critical
DETAILS   | the cluster has 2 different tidb versions, execute the sql to see more detail: SELECT * FROM information_schema.cluster_info WHERE type='tidb'
```

### `critical-error` diagnostic rule

In `critical-error` diagnostic rule, the following two diagnostic rules are executed:

* Detect whether the cluster has the following errors by querying the related monitoring system tables in the metrics schema:

|  Component  | Error name | Monitoring table | Error description |
|  ----  | ----  |  ----  |  ----  |
| TiDB | panic-count | tidb_panic_count_total_count | Panic occurs in TiDB. |
| TiDB | binlog-error | tidb_binlog_error_total_count | An error occurs when TiDB writes binlog. |
| TiKV | critical-error | tikv_critical_error_total_coun | The critical error of TiKV. |
| TiKV | scheduler-is-busy       | tikv_scheduler_is_busy_total_count | The TiKV scheduler is too busy, which makes TiKV temporarily unavailable. |
| TiKV | coprocessor-is-busy | tikv_coprocessor_is_busy_total_count | The TiKV Coprocessor is too busy. |
| TiKV | channel-is-full | tikv_channel_full_total_count | The "channel full" error occurs in TiKV. |
| TiKV | tikv_engine_write_stall | tikv_engine_write_stall | The "stall" error occurs in TiKV. |

* Check whether any component is restarted by querying the `metrics_schema.up` monitoring table and the `CLUSTER_LOG` system table.

### `threshold-check` diagnostic rule

The `threshold-check` diagnostic rule checks whether the following metrics in the cluster exceed the threshold by querying the related monitoring system tables in the metrics schema:

|  Component  | Monitoring metric | Monitoring table | Expected value |  Description  |
|  :----  | :----  |  :----  |  :----  |  :----  |
| TiDB | tso-duration              | pd_tso_wait_duration                | < 50ms  |   The wait duration of getting the TSO of transaction. |
| TiDB | get-token-duration        | tidb_get_token_duration             | < 1ms   |  Queries the time it takes to get the token. The related TiDB configuration item is [`token-limit`](/command-line-flags-for-tidb-configuration.md#--token-limit).  |
| TiDB | load-schema-duration      | tidb_load_schema_duration           | < 1s    |   The time it takes for TiDB to update the schema metadata.|
| TiKV | scheduler-cmd-duration    | tikv_scheduler_command_duration     | < 0.1s  |  The time it takes for TiKV to execute the KV `cmd` request. |
| TiKV | handle-snapshot-duration  | tikv_handle_snapshot_duration       | < 30s   |  The time it takes for TiKV to handle the snapshot. |
| TiKV | storage-write-duration    | tikv_storage_async_request_duration | < 0.1s  |  The write latency of TiKV. |
| TiKV | storage-snapshot-duration | tikv_storage_async_request_duration | < 50ms  |  The time it takes for TiKV to get the snapshot. |
| TiKV | rocksdb-write-duration    | tikv_engine_write_duration          | < 100ms |  The write latency of TiKV RocksDB. |
| TiKV | rocksdb-get-duration | tikv_engine_max_get_duration | < 50ms |   The read latency of TiKV RocksDB. |
| TiKV | rocksdb-seek-duration | tikv_engine_max_seek_duration | < 50ms |  The latency of TiKV RocksDB to execute `seek`.   |
| TiKV | scheduler-pending-cmd-coun | tikv_scheduler_pending_commands  | < 1000 |  The number of commands stalled in TiKV.  |
| TiKV | index-block-cache-hit | tikv_block_index_cache_hit | > 0.95 |  The hit rate of index block cache in TiKV. |
| TiKV | filter-block-cache-hit | tikv_block_filter_cache_hit | > 0.95 |  The hit rate of filter block cache in TiKV. |
| TiKV | data-block-cache-hit | tikv_block_data_cache_hit | > 0.80 |  The hit rate of data block cache in TiKV. |
| TiKV | leader-score-balance | pd_scheduler_store_status  | < 0.05 |  Checks whether the leader score of each TiKV instance is balanced. The expected difference between instances is less than 5%. |
| TiKV | region-score-balance | pd_scheduler_store_status  | < 0.05 |  Checks whether the Region score of each TiKV instance is balanced. The expected difference between instances is less than 5%. |
| TiKV | store-available-balance | pd_scheduler_store_status  | < 0.2 | Checks whether the available storage of each TiKV instance is balanced. The expected difference between instances is less than 20%. |
| TiKV | region-count | pd_scheduler_store_status  | < 20000 |  Checks the number of Regions on each TiKV instance. The expected number of Regions in a single instance is less than 20,000. |
| PD | region-health | pd_region_health | < 100  |  Detects the number of Regions that are in the process of scheduling in the cluster. The expected number is less than 100 in total. |

In addition, this rule also checks whether the CPU usage of the following threads in a TiKV instance is too high:

* scheduler-worker-cpu
* coprocessor-normal-cpu
* coprocessor-high-cpu
* coprocessor-low-cpu
* grpc-cpu
* raftstore-cpu
* apply-cpu
* storage-readpool-normal-cpu
* storage-readpool-high-cpu
* storage-readpool-low-cpu
* split-check-cpu

The built-in diagnostic rules are constantly being improved. If you have more diagnostic rules, welcome to create a PR or an issue in the [`tidb` repository](https://github.com/pingcap/tidb).
```