---
title: 遅いクエリの特定
summary: 遅いクエリログを使用して問題のあるSQLステートメントを特定します。
aliases: ['/docs/dev/identify-slow-queries/','/docs/dev/how-to/maintain/identify-abnormal-queries/identify-slow-queries/','/docs/dev/how-to/maintain/identify-slow-queries']
---

# 遅いクエリの特定

遅いクエリを特定し、SQLの実行パフォーマンスを分析して改善するために、TiDBは[`tidb_slow_log_threshold`](/system-variables.md#tidb_slow_log_threshold)（デフォルト値は300ミリ秒）を超える実行時間を持つステートメントを[遅いクエリファイル](/tidb-configuration-file.md#slow-query-file)（デフォルト値は "tidb-slow.log"）に出力します。

TiDBは、遅いクエリログをデフォルトで有効にしています。システム変数[`tidb_enable_slow_log`](/system-variables.md#tidb_enable_slow_log)を変更することで、この機能を有効または無効にすることができます。

## 使用例

```sql
# Time: 2019-08-14T09:26:59.487776265+08:00
# Txn_start_ts: 410450924122144769
# User@Host: root[root] @ localhost [127.0.0.1]
# Conn_ID: 3086
# Exec_retry_time: 5.1 Exec_retry_count: 3
# Query_time: 1.527627037
# Parse_time: 0.000054933
# Compile_time: 0.000129729
# Rewrite_time: 0.000000003 Preproc_subqueries: 2 Preproc_subqueries_time: 0.000000002
# Optimize_time: 0.00000001
# Wait_TS: 0.00001078
# Process_time: 0.07 Request_count: 1 Total_keys: 131073 Process_keys: 131072 Prewrite_time: 0.335415029 Commit_time: 0.032175429 Get_commit_ts_time: 0.000177098 Local_latch_wait_time: 0.106869448 Write_keys: 131072 Write_size: 3538944 Prewrite_region: 1
# DB: test
# Is_internal: false
# Digest: 50a2e32d2abbd6c1764b1b7f2058d428ef2712b029282b776beb9506a365c0f1
# Stats: t:pseudo
# Num_cop_tasks: 1
# Cop_proc_avg: 0.07 Cop_proc_p90: 0.07 Cop_proc_max: 0.07 Cop_proc_addr: 172.16.5.87:20171
# Cop_wait_avg: 0 Cop_wait_p90: 0 Cop_wait_max: 0 Cop_wait_addr: 172.16.5.87:20171
# Cop_backoff_regionMiss_total_times: 200 Cop_backoff_regionMiss_total_time: 0.2 Cop_backoff_regionMiss_max_time: 0.2 Cop_backoff_regionMiss_max_addr: 127.0.0.1 Cop_backoff_regionMiss_avg_time: 0.2 Cop_backoff_regionMiss_p90_time: 0.2
# Cop_backoff_rpcPD_total_times: 200 Cop_backoff_rpcPD_total_time: 0.2 Cop_backoff_rpcPD_max_time: 0.2 Cop_backoff_rpcPD_max_addr: 127.0.0.1 Cop_backoff_rpcPD_avg_time: 0.2 Cop_backoff_rpcPD_p90_time: 0.2
# Cop_backoff_rpcTiKV_total_times: 200 Cop_backoff_rpcTiKV_total_time: 0.2 Cop_backoff_rpcTiKV_max_time: 0.2 Cop_backoff_rpcTiKV_max_addr: 127.0.0.1 Cop_backoff_rpcTiKV_avg_time: 0.2 Cop_backoff_rpcTiKV_p90_time: 0.2
# Mem_max: 525211
# Disk_max: 65536
# Prepared: false
# Plan_from_cache: false
# Succ: true
# Plan: tidb_decode_plan('ZJAwCTMyXzcJMAkyMAlkYXRhOlRhYmxlU2Nhbl82CjEJMTBfNgkxAR0AdAEY1Dp0LCByYW5nZTpbLWluZiwraW5mXSwga2VlcCBvcmRlcjpmYWxzZSwgc3RhdHM6cHNldWRvCg==')
use test;
insert into t select * from t;
```

## フィールドの説明

> **注意:**
>
> 遅いクエリログ内の以下の時間フィールドの単位は**「秒」**です。

遅いクエリの基本情報:

* `Time`: ログの出力時間。
* `Query_time`: ステートメントの実行時間。
* `Parse_time`: ステートメントの解析にかかる時間。
* `Compile_time`: クエリの最適化にかかる時間。
* `Optimize_time`: 実行計画の最適化にかかる時間。
* `Wait_TS`: トランザクションのタイムスタンプを取得するためにステートメントが待機する時間。
* `Query`: SQLステートメント。`Query`は遅いクエリログには表示されませんが、対応するフィールドは、遅いログがメモリーテーブルにマッピングされた後に`Query`と呼ばれます。
* `Digest`: SQLステートメントのフィンガープリント。
* `Txn_start_ts`: トランザクションの開始タイムスタンプと一意のID。この値を使用してトランザクション関連のログを検索できます。
* `Is_internal`: SQLステートメントがTiDB内部で実行されるかどうか。`true`はSQLステートメントがTiDB内部で実行されることを示し、`false`はユーザーによってSQLステートメントが実行されることを示します。
* `Index_names`: ステートメントで使用されるインデックス名。
* `Stats`: このクエリの実行中に使用される統計情報の健康状態、内部バージョン、合計行数、変更された行数、およびロード状況を示す。`pseudo`は統計情報が不健康であることを示します。最適化プログラムが完全にロードされていない一部の統計情報を使用しようとする場合、内部状態も出力されます。例えば、`t1:439478225786634241[105000;5000][col1:allEvicted][idx1:allEvicted]`の意味は次のように理解できます:
    - `t1`: クエリの最適化中にテーブル`t1`の統計情報が使用される。
    - `439478225786634241`: 内部バージョン。
    - `105000`: 統計情報内の総行数。
    - `5000`: 最後に統計情報を収集してからの変更行数。
    - `col1:allEvicted`: 列`col1`の統計情報が完全にロードされていない。
    - `idx1:allEvicted`: インデックス`idx1`の統計情報が完全にロードされていない。
* `Succ`: ステートメントが正常に実行されたかどうか。
* `Backoff_time`: ステートメントが再試行を要するエラーに遭遇した場合の再試行前の待機時間。一般的なエラーには`lock occurs`、`Region split`、`tikv server is busy`などが含まれる。
* `Plan`: ステートメントの実行計画。具体的な実行計画を解析するには`SELECT tidb_decode_plan('xxx...')`ステートメントを実行します。
* `Binary_plan`: バイナリエンコードされたステートメントの実行計画。具体的な実行計画を解析するには`SELECT tidb_decode_binary_plan('xxx...')`ステートメントを実行します。`Plan`と`Binary_plan`フィールドは同じ情報を持っています。ただし、2つのフィールドから解析された実行計画の形式は異なります。
* `Prepared`: このステートメントが`Prepare`または`Execute`リクエストであるかどうか。
* `Plan_from_cache`: このステートメントが実行計画キャッシュにヒットしたかどうか。
* `Plan_from_binding`: このステートメントがバインドされた実行計画を使用したかどうか。
* `Has_more_results`: このステートメントがユーザーによって取得される結果がさらにあるかどうか。
* `Rewrite_time`: このステートメントのクエリのリライトにかかる時間。
* `Preproc_subqueries`: 事前に実行されるサブクエリ（ステートメント内にある）の数。例えば、`where id in (select if from t)`サブクエリが事前に実行されるかもしれません。
* `Preproc_subqueries_time`: このステートメントのサブクエリの事前実行にかかる時間。
* `Exec_retry_count`: このステートメントのリトライ回数。このフィールドは通常、ステートメントがロックに失敗した場合にリトライされる悲観的トランザクションに使用されます。
* `Exec_retry_time`: このステートメントの実行リトライ時間。例えば、ステートメントが合計3回（最初の2回は失敗）実行された場合、`Exec_retry_time`は最初の2回の実行の合計期間を意味します。最後の実行の期間は、`Query_time`から`Exec_retry_time`を引いたものです。
* `KV_total`: このステートメントによるTiKVまたはTiFlashへのすべてのRPCリクエストに費やされる時間。
* `PD_total`: このステートメントによるPDへのすべてのRPCリクエストに費やされる時間。
* `Backoff_total`: このステートメントの実行中に発生したすべてのバックオフに費やされる時間。
* `Write_sql_response_total`: このステートメントによるクライアントへの結果の送信にかかる時間。
* `Result_rows`: クエリ結果の行数。
* `IsExplicitTxn`: このステートメントが明示的トランザクションの中にあるかどうか。値が`false`の場合、トランザクションは`autocommit=1`であり、ステートメントは自動的に実行後にコミットされます。
* `Warnings`: このステートメントの実行中に生成されたJSON形式の警告。これらの警告は一般的に[`SHOW WARNINGS`](/sql-statements/sql-statement-show-warnings.md)ステートメントの出力と一貫していますが、追加の診断情報を提供する追加の警告が含まれることがあります。これらの追加の警告は`IsExtra: true`とマークされます。

次のフィールドはトランザクションの実行に関連しています:

* `Prewrite_time`: 2フェーズのトランザクションコミットの最初のフェーズ（prewrite）の期間。
* `Commit_time`: 2フェーズのトランザクションコミットの2番目のフェーズ（commit）の期間。
* `Get_commit_ts_time`: 2フェーズのトランザクションコミットの2番目のフェーズ（commit）中に`commit_ts`を取得するのにかかった時間。
* `Local_latch_wait_time`: 2フェーズのトランザクションコミットの2番目のフェーズ（commit）の前にTiDBがロックを待機していた時間。
* `Write_keys`: トランザクションがTiKVのWrite CFに書き込むキーの数。
* `Write_size`: トランザクションがコミットするときに書き込まれるキーまたは値の総サイズ。
* `Prewrite_region`: 2フェーズのトランザクションコミットの最初のフェーズ（prewrite）に関与するTiKVリージョンの数。各リージョンはリモート手続き呼び出しをトリガーします。
* `Wait_prewrite_binlog_time`: トランザクションがコミットされるときにバイナリログを書き込むために使用された時間。
* `Resolve_lock_time`: トランザクションコミット中にロックを解決するかロックの期限切れを待機する時間。

メモリ使用量フィールド:

* `Mem_max`: SQLステートメントの実行期間中に使用される最大メモリスペース（単位はバイト）。

ハードディスクフィールド:

* `Disk_max`: SQLステートメントの実行期間中に使用される最大ディスクスペース（単位はバイト）。

ユーザー項目:

* `User`: このステートメントを実行するユーザーの名前。
* `Host`: このステートメントのホスト名。
* `Conn_ID`: コネクションID（セッションID）。たとえば、セッションIDが`3`であるログを検索するために、キーワード`con:3`を使用できます。
* `DB`: 現在のデータベース。

TiKV Coprocessorタスクフィールド:

* `Request_count`: ステートメントが送信するCoprocessorリクエストの数。
* `Total_keys`: Coprocessorがスキャンしたキーの数。
* `Process_time`: TiKVでのSQLステートメントの総処理時間。データはTiKVに同時に送信されるため、この値は`Query_time`を超える場合があります。
* `Wait_time`: TiKVでのステートメントの総待ち時間。TiKVのCoprocessorが限られた数のスレッドで実行されるため、すべてのCoprocessorのスレッドが作業しているとリクエストがキューイングされるかもしれません。キュー内のリクエストの処理に時間がかかると、その後のリクエストの待ち時間が増加します。
* `Process_keys`: Coprocessorが処理したキーの数。`processed_keys`は古いMVCCのバージョンを含みませんが、`total_keys`と比較すると`processed_keys`、大きな違いがある場合は多くの古いバージョンが存在することを示します。
* `Num_cop_tasks`: このステートメントによって送信されたCoprocessorタスクの数。
* `Cop_proc_avg`: コプタスクの平均実行時間を含む待ち時間など、カウントされない待ち時間のようなものを含みます、例えばRocksDB内のミューテックス。
* `Cop_proc_p90`: コプタスクのP90実行時間。
* `Cop_proc_max`: コプタスクの最大実行時間。
* `Cop_proc_addr`: 最長の実行時間を持つコプタスクのアドレス。
* `Cop_wait_avg`: コプタスクの平均待ち時間を含む、リクエストキューイングやスナップショット取得の時間。
* `Cop_wait_p90`: コプタスクの待ち時間のP90。
* `Cop_wait_max`: コプタスクの最大待ち時間。
* `Cop_wait_addr`: 待ち時間が最も長いコプタスクのアドレス。
* `Rocksdb_delete_skipped_count`: RocksDBの読み取り中に削除されたキーのスキャン回数。
* `Rocksdb_key_skipped_count`: スキャンデータ時にRocksDBが遭遇する削除済み（墓石）キーの数。
* `Rocksdb_block_cache_hit_count`: RocksDBがブロックキャッシュからデータを読み取った回数。
* `Rocksdb_block_read_count`: RocksDBがファイルシステムからデータを読み取った回数。
* `Rocksdb_block_read_byte`: RocksDBがファイルシステムから読み取ったデータ量。
* `Rocksdb_block_read_time`: RocksDBがファイルシステムからデータを読み取るのにかかった時間。
* `Cop_backoff_{backoff-type}_total_times`: エラーによって引き起こされたリトライの合計回数。
* `Cop_backoff_{backoff-type}_total_time`: エラーによって引き起こされたリトライの総時間。
* `Cop_backoff_{backoff-type}_max_time`: エラーによって引き起こされたリトライの最長時間。
* `Cop_backoff_{backoff-type}_max_addr`: エラーによって引き起こされたリトライの時間が最も長いコプタスクのアドレス。
* `Cop_backoff_{backoff-type}_avg_time`: エラーによって引き起こされたリトライの平均時間。
* `Cop_backoff_{backoff-type}_p90_time`: エラーによって引き起こされたリトライのP90パーセンタイル時間。

`backoff-type` には一般的に以下のタイプが含まれます:

* `tikvRPC`: TiKVへのRPCリクエストの送信に失敗したことによるリトライ。
* `tiflashRPC`: TiFlashへのRPCリクエストの送信に失敗したことによるリトライ。
* `pdRPC`: PDへのRPCリクエストの送信に失敗したことによるリトライ。
* `txnLock`: ロックの競合によるリトライ。
* `regionMiss`: リージョンが分割されたり統合された後にTiDBリージョンキャッシュ情報が古い場合、リクエストの処理が失敗したことによるリトライ。
* `regionScheduling`: リージョンがスケジュールされており、リーダーが選択されていないときにTiDBがリクエストを処理できないことによるリトライ。
* `tikvServerBusy`: TiKVの負荷が高すぎて新しいリクエストを処理できないことによるリトライ。
* `tiflashServerBusy`: TiFlashの負荷が高すぎて新しいリクエストを処理できないことによるリトライ。
* `tikvDiskFull`: TiKVのディスクがいっぱいであることによるリトライ。
* `txnLockFast`: データの読み取り中にロックが発生したことによるリトライ。

## 関連するシステム変数

* [`tidb_slow_log_threshold`](/system-variables.md#tidb_slow_log_threshold): 遅いログのしきい値を設定します。このしきい値を超える実行時間のSQLステートメントは遅いログに記録されます。デフォルト値は300 (ms) です。
* [`tidb_query_log_max_len`](/system-variables.md#tidb_query_log_max_len): 遅いログに記録されるSQLステートメントの最大長を設定します。デフォルト値は4096 (バイト) です。
* [`tidb_redact_log`](/system-variables.md#tidb_redact_log): 遅いログに記録されたSQLステートメントでユーザーデータを`?`で非表示にするかどうかを決定します。デフォルト値は`0`で、この機能が無効になっています。
* [`tidb_enable_collect_execution_info`](/system-variables.md#tidb_enable_collect_execution_info): 実行プランの各演算子の物理的な実行情報を記録するかどうかを決定します。デフォルト値は`1`です。この機能はパフォーマンスに約3%の影響を与えます。この機能を有効にした後、以下のように`Plan`情報を表示できます:

    ```sql
    > select tidb_decode_plan('jAOIMAk1XzE3CTAJMQlmdW5jczpjb3VudChDb2x1bW4jNyktPkMJC/BMNQkxCXRpbWU6MTAuOTMxNTA1bXMsIGxvb3BzOjIJMzcyIEJ5dGVzCU4vQQoxCTMyXzE4CTAJMQlpbmRleDpTdHJlYW1BZ2dfOQkxCXQRSAwyNzY4LkgALCwgcnBjIG51bTogMQkMEXMQODg0MzUFK0hwcm9jIGtleXM6MjUwMDcJMjA2HXsIMgk1BWM2zwAAMRnIADcVyAAxHcEQNQlOL0EBBPBbCjMJMTNfMTYJMQkzMTI4MS44NTc4MTk5MDUyMTcJdGFibGU6dCwgaW5kZXg6aWR4KGEpLCByYW5nZTpbLWluZiw1MDAwMCksIGtlZXAgb3JkZXI6ZmFsc2UJMjUBrgnQVnsA');
    +------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```markdown
    | tidb_decode_plan('jAOIMAk1XzE3CTAJMQlmdW5jczpjb3VudChDb2x1bW4jNyktPkMJC/BMNQkxCXRpbWU6MTAuOTMxNTA1bXMsIGxvb3BzOjIJMzcyIEJ5dGVzCU4vQQoxCTMyXzE4CTAJMQlpbmRleDpTdHJlYW1BZ2dfOQkxCXQRSAwyNzY4LkgALCwgcnBjIG51bTogMQkMEXMQODg0MzUFK0hwcm9jIGtleXM6MjUwMDcJMjA2HXsIMg |
    +------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
    |     id                    task    estRows               operator info                                                  actRows    execution info                                                                  memory       disk                              |
    |     StreamAgg_17          root    1                     funcs:count(Column#7)->Column#5                                1          time:10.931505ms, loops:2                                                       372 Bytes    N/A                               |
    |     └─IndexReader_18      root    1                     index:StreamAgg_9                                              1          time:10.927685ms, loops:2, rpc num: 1, rpc time:10.884355ms, proc keys:25007    206 Bytes    N/A                               |
    |       └─StreamAgg_9       cop     1                     funcs:count(1)->Column#7                                       1          time:11ms, loops:25                                                             N/A          N/A                               |
    |         └─IndexScan_16    cop     31281.857819905217    table:t, index:idx(a), range:[-inf,50000), keep order:false    25007      time:11ms, loops:25                                                             N/A          N/A                               |
    +------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```
```markdown
If you are conducting a performance test, you can disable the feature of automatically collecting the execution information of operators:

{{< copyable "sql" >}}

```sql
set @@tidb_enable_collect_execution_info=0;
```

The returned result of the `Plan` field has roughly the same format with that of `EXPLAIN` or `EXPLAIN ANALYZE`. For more details of the execution plan, see [`EXPLAIN`](/sql-statements/sql-statement-explain.md) or [`EXPLAIN ANALYZE`](/sql-statements/sql-statement-explain-analyze.md).

For more information, see [TiDB specific variables and syntax](/system-variables.md).

## Memory mapping in slow log

スロークエリログのコンテンツは、`INFORMATION_SCHEMA.SLOW_QUERY` テーブルをクエリすることで取得できます。このテーブル内の各列名は、スロークエリログの1つのフィールド名に対応します。テーブル構造については、[Information Schema](/information-schema/information-schema-slow-query.md) の `SLOW_QUERY` テーブルに関する紹介をご覧ください。

> **注意:**
>
> `SLOW_QUERY` テーブルをクエリするたびに、TiDB は現在のスロークエリログを読み取り、パースします。

TiDB 4.0 では、`SLOW_QUERY` はローテートされたスロークエリログファイルを含む任意の時間帯のスロークエリログをクエリする機能をサポートしています。スロークエリログファイルをパースするには、`TIME` 範囲を指定する必要があります。`TIME` 範囲を指定しない場合、TiDB は現在のスロークエリログファイルのみをパースします。例:

* 時間範囲を指定しない場合、TiDB は TiDB がスロークエリログファイルに書き込んでいるスロークエリデータのみをパースします:

  {{< copyable "sql" >}}

  ```sql
  select count(*),
        min(time),
        max(time)
  from slow_query;
  ```

  ```
  +----------+----------------------------+----------------------------+
  | count(*) | min(time)                  | max(time)                  |
  +----------+----------------------------+----------------------------+
  | 122492   | 2020-03-11 23:35:20.908574 | 2020-03-25 19:16:38.229035 |
  +----------+----------------------------+----------------------------+
  ```

* 時間範囲を指定する場合、たとえば `2020-03-10 00:00:00` から `2020-03-11 00:00:00` まで、TiDB はまず指定された時間範囲のスロークエリログファイルを特定し、それからスロークエリ情報をパースします:

  {{< copyable "sql" >}}

  ```sql
  select count(*),
        min(time),
        max(time)
  from slow_query
  where time > '2020-03-10 00:00:00'
    and time < '2020-03-11 00:00:00';
  ```

  ```
  +----------+----------------------------+----------------------------+
  | count(*) | min(time)                  | max(time)                  |
  +----------+----------------------------+----------------------------+
  | 2618049  | 2020-03-10 00:00:00.427138 | 2020-03-10 23:00:22.716728 |
  +----------+----------------------------+----------------------------+
  ```

> **注意:**
>
> 指定された時間範囲のスロークエリログファイルが削除された場合、またはスロークエリが存在しない場合、クエリは NULL を返します。

TiDB 4.0 では、すべての TiDB ノードのスロークエリ情報をクエリするための [`CLUSTER_SLOW_QUERY`](/information-schema/information-schema-slow-query.md#cluster_slow_query-table) システムテーブルが追加されています。`CLUSTER_SLOW_QUERY` テーブルのテーブル構造は `SLOW_QUERY` テーブルのそれと異なり、`CLUSTER_SLOW_QUERY` に `INSTANCE` 列が追加されています。`INSTANCE` 列はスロークエリに関する行情報の TiDB ノードアドレスを表します。`CLUSTER_SLOW_QUERY` を使用する方法は、[`SLOW_QUERY`](/information-schema/information-schema-slow-query.md) と同様です。

`CLUSTER_SLOW_QUERY` テーブルをクエリする場合、TiDB は計算と判断を他のノードに移動させます。これにより、他のノードからのすべてのスロークエリ情報を取得し、1つの TiDB ノードで操作を実行する必要がなくなります。

## `SLOW_QUERY` / `CLUSTER_SLOW_QUERY` 使用例

### Top-N スロークエクエリ

ユーザーの Top 2 スロークエリをクエリします。`Is_internal=false` は TiDB 内のスロークエリを除き、ユーザーのスロークエリのみをクエリすることを意味します。

{{< copyable "sql" >}}

```sql
select query_time, query
from information_schema.slow_query
where is_internal = false
order by query_time desc
limit 2;
```

出力例:

```
+--------------+------------------------------------------------------------------+
| query_time   | query                                                            |
+--------------+------------------------------------------------------------------+
| 12.77583857  | select * from t_slim, t_wide where t_slim.c0=t_wide.c0;          |
|  0.734982725 | select t0.c0, t1.c1 from t_slim t0, t_wide t1 where t0.c0=t1.c0; |
+--------------+------------------------------------------------------------------+
```

### `test` ユーザーの Top-N スロークエリをクエリ

以下の例では、`test` ユーザーが実行したスロークエリをクエリし、最初の2つの結果を実行時間の逆順で表示します。

{{< copyable "sql" >}}

```sql
select query_time, query, user
from information_schema.slow_query
where is_internal = false
  and user = "test"
order by query_time desc
limit 2;
```

出力例:

```
+-------------+------------------------------------------------------------------+----------------+
| Query_time  | query                                                            | user           |
+-------------+------------------------------------------------------------------+----------------+
| 0.676408014 | select t0.c0, t1.c1 from t_slim t0, t_wide t1 where t0.c0=t1.c1; | test           |
+-------------+------------------------------------------------------------------+----------------+
```

### 同じ SQL フィンガープリントを持つ類似のスロークエリをクエリ

Top-N SQL 文を取得した後、同じフィンガープリントを使用して類似のスロークエリを続けてクエリします。

1. Top-N スロークエリおよびそれに対応する SQL フィンガープリントを取得します。

    {{< copyable "sql" >}}

    ```sql
    select query_time, query, digest
    from information_schema.slow_query
    where is_internal = false
    order by query_time desc
    limit 1;
    ```

    出力例:

    ```
    +-------------+-----------------------------+------------------------------------------------------------------+
    | query_time  | query                       | digest                                                           |
    +-------------+-----------------------------+------------------------------------------------------------------+
    | 0.302558006 | select * from t1 where a=1; | 4751cb6008fda383e22dacb601fde85425dc8f8cf669338d55d944bafb46a6fa |
    +-------------+-----------------------------+------------------------------------------------------------------+
    ```

2. フィンガープリントを使用して類似のスロークエリをクエリします。

    {{< copyable "sql" >}}

    ```sql
    select query, query_time
    from information_schema.slow_query
    where is_internal = false
      and digest = '4751cb6008fda383e22dacb601fde85425dc8f8cf669338d55d944bafb46a6fa';
    ```
```
    digest="4751cb6008fda383e22dacb601fde85425dc8f8cf669338d55d944bafb46a6fa" となります。

出力例：

```
+-----------------------------+-------------+
| query                       | query_time  |
+-----------------------------+-------------+
| select * from t1 where a=1; | 0.302558006 |
| select * from t1 where a=2; | 0.401313532 |
+-----------------------------+-------------+
```

## `stats` と疑似の `stats` で遅いクエリを検索

{{< copyable "sql" >}}

```sql
select query, query_time, stats
from information_schema.slow_query
where is_internal = false
  and stats like '%pseudo%';
```

出力例：

```
+-----------------------------+-------------+---------------------------------+
| query                       | query_time  | stats                           |
+-----------------------------+-------------+---------------------------------+
| select * from t1 where a=1; | 0.302558006 | t1:pseudo                       |
| select * from t1 where a=2; | 0.401313532 | t1:pseudo                       |
| select * from t1 where a>2; | 0.602011247 | t1:pseudo                       |
| select * from t1 where a>3; | 0.50077719  | t1:pseudo                       |
| select * from t1 join t2;   | 0.931260518 | t1:407872303825682445,t2:pseudo |
+-----------------------------+-------------+---------------------------------+
```

### 実行計画が変化した遅いクエリを検索

同じカテゴリのSQL文の実行計画が変更されると、統計情報が古くなっていたり、実データの分布を正確に反映していなかったりするため、実行が遅くなります。次のSQL文を使用して、異なる実行計画を持つSQL文を検索できます。

{{< copyable "sql" >}}

```sql
select count(distinct plan_digest) as count,
       digest,
       min(query)
from cluster_slow_query
group by digest
having count > 1
limit 3\G
```

出力例：

```
***************************[ 1. row ]***************************
count      | 2
digest     | 17b4518fde82e32021877878bec2bb309619d384fca944106fcaf9c93b536e94
min(query) | SELECT DISTINCT c FROM sbtest25 WHERE id BETWEEN ? AND ? ORDER BY c [arguments: (291638, 291737)];
***************************[ 2. row ]***************************
count      | 2
digest     | 9337865f3e2ee71c1c2e740e773b6dd85f23ad00f8fa1f11a795e62e15fc9b23
min(query) | SELECT DISTINCT c FROM sbtest22 WHERE id BETWEEN ? AND ? ORDER BY c [arguments: (215420, 215519)];
***************************[ 3. row ]***************************
count      | 2
digest     | db705c89ca2dfc1d39d10e0f30f285cbbadec7e24da4f15af461b148d8ffb020
min(query) | SELECT DISTINCT c FROM sbtest11 WHERE id BETWEEN ? AND ? ORDER BY c [arguments: (303359, 303458)];
```

次に、上記のクエリ結果のSQL指紋を使用して異なる計画をクエリできます：

{{< copyable "sql" >}}

```sql
select min(plan),
       plan_digest
from cluster_slow_query
where digest='17b4518fde82e32021877878bec2bb309619d384fca944106fcaf9c93b536e94'
group by plan_digest\G
```

出力例：

```
*************************** 1. row ***************************
  min(plan):    Sort_6                  root    100.00131380758702      sbtest.sbtest25.c:asc
        └─HashAgg_10            root    100.00131380758702      group by:sbtest.sbtest25.c, funcs:firstrow(sbtest.sbtest25.c)->sbtest.sbtest25.c
          └─TableReader_15      root    100.00131380758702      data:TableRangeScan_14
            └─TableScan_14      cop     100.00131380758702      table:sbtest25, range:[502791,502890], keep order:false
plan_digest: 6afbbd21f60ca6c6fdf3d3cd94f7c7a49dd93c00fcf8774646da492e50e204ee
*************************** 2. row ***************************
  min(plan):    Sort_6                  root    1                       sbtest.sbtest25.c:asc
        └─HashAgg_12            root    1                       group by:sbtest.sbtest25.c, funcs:firstrow(sbtest.sbtest25.c)->sbtest.sbtest25.c
          └─TableReader_13      root    1                       data:HashAgg_8
            └─HashAgg_8         cop     1                       group by:sbtest.sbtest25.c,
              └─TableScan_11    cop     1.2440069558121831      table:sbtest25, range:[472745,472844], keep order:false
plan_digest: 17b4518fde82e32021877878bec2bb309619d384fca944106fcaf9c93b536e94
```

### クラスタ内の各TiDBノードにおける遅いクエリの数を検索

{{< copyable "sql" >}}

```sql
select instance, count(*) from information_schema.cluster_slow_query where time >= "2020-03-06 00:00:00" and time < now() group by instance;
```

出力例：

```
+---------------+----------+
| instance      | count(*) |
+---------------+----------+
| 0.0.0.0:10081 | 124      |
| 0.0.0.0:10080 | 119771   |
+---------------+----------+
```

### 異常な時間帯に発生する遅いログをクエリ

`2020-03-10 13:24:00` から `2020-03-10 13:27:00` までの時間帯にQPSが減少したりレイテンシーが増加したりするなどの問題が発生した場合、大きなクエリが発生している可能性があります。異常な時間帯にのみ発生する遅いログをクエリするには、次のSQL文を実行します。`2020-03-10 13:20:00` から `2020-03-10 13:23:00` までの時間帯は通常の時間帯を指します。

{{< copyable "sql" >}}

```sql
SELECT * FROM
    (SELECT /*+ AGG_TO_COP(), HASH_AGG() */ count(*),
         min(time),
         sum(query_time) AS sum_query_time,
         sum(Process_time) AS sum_process_time,
         sum(Wait_time) AS sum_wait_time,
         sum(Commit_time),
         sum(Request_count),
         sum(process_keys),
         sum(Write_keys),
         max(Cop_proc_max),
         min(query),min(prev_stmt),
         digest
    FROM information_schema.CLUSTER_SLOW_QUERY
    WHERE time >= '2020-03-10 13:24:00'
            AND time < '2020-03-10 13:27:00'
            AND Is_internal = false
    GROUP BY  digest) AS t1
WHERE t1.digest NOT IN
    (SELECT /*+ AGG_TO_COP(), HASH_AGG() */ digest
    FROM information_schema.CLUSTER_SLOW_QUERY
    WHERE time >= '2020-03-10 13:20:00'
            AND time < '2020-03-10 13:23:00'
    GROUP BY  digest)
ORDER BY  t1.sum_query_time DESC limit 10\G
```

出力例：

```
***************************[ 1. row ]***************************
count(*)           | 200
min(time)          | 2020-03-10 13:24:27.216186
sum_query_time     | 50.114126194
sum_process_time   | 268.351
sum_wait_time      | 8.476
sum(Commit_time)   | 1.044304306
sum(Request_count) | 6077
sum(process_keys)  | 202871950
sum(Write_keys)    | 319500
max(Cop_proc_max)  | 0.263
min(query)         | delete from test.tcs2 limit 5000;
min(prev_stmt)     |
digest             | 24bd6d8a9b238086c9b8c3d240ad4ef32f79ce94cf5a468c0b8fe1eb5f8d03df
```

### 他のTiDB遅いログファイルの解析

TiDBはセッション変数 `tidb_slow_query_file` を使用して、`INFORMATION_SCHEMA.SLOW_QUERY` をクエリする際に読み込むファイルを制御します。セッション変数の値を変更することで、他の遅いクエリログファイルの内容をクエリできます。

{{< copyable "sql" >}}

```sql
set tidb_slow_query_file = "/path-to-log/tidb-slow.log"
```
### 「pt-query-digest」を使用してTiDBの遅いログを解析する

`pt-query-digest`を使用してTiDBの遅いクエリログを解析します。

> **注意:**
>
> `pt-query-digest` 3.0.13またはそれ以降のバージョンを使用することをお勧めします。

例:

{{< copyable "shell-regular" >}}

```shell
pt-query-digest --report tidb-slow.log
```

出力例:

```
# 320msのユーザータイム、20msのシステムタイム、27.00Mのrss、221.32Mのvsz
# 現在の日付: 2019年3月18日月曜日 13:18:51
# ホスト名: localhost.localdomain
# ファイル: tidb-slow.log
# 全体: 1.02k合計、21ユニーク、0 QPS、0x並行_________________
# 時間範囲: 2019-03-18-12:22:16から2019-03-18-13:08:52
# 属性          合計     最小     最大     平均     95%  標準偏差  中央値
# ============     ======= ======= ======= ======= ======= ======= =======
# 実行時間           218秒    10ms     13秒   213ms    30ms      1秒    19ms
# クエリーサイズ       175.37k       9   2.01k  175.89  158.58  122.36  158.58
# コミット時間         46ms     2ms     7ms     3ms     7ms     1ms     3ms
# コネクションID               71       1      16    8.88   15.25    4.06    9.83
# プロセスキー     581.87k       2 103.15k  596.43  400.73   3.91k  400.73
# プロセス時間         31秒     1ms     10秒    32ms    19ms   334ms    16ms
# リクエスト数       1.97k       1      10    2.02    1.96    0.33    1.96
# 合計キー       636.43k       2 103.16k  652.35  793.42   3.97k  400.73
# トランザクション開始時刻     374.38E       0  16.00E 375.48P   1.25P  89.05T   1.25P
# 待機時間          943ms     1ms    19ms     1ms     2ms     1ms   972us
.
.
.
```

## 問題のあるSQLステートメントを特定する

すべての`SLOW_QUERY`ステートメントが問題を抱えているわけではありません。`process_time`が非常に長いステートメントのみがクラスタ全体に圧力をかけます。

`wait_time`が非常に長く、`process_time`が非常に短いステートメントは通常問題ではありません。これは、ステートメントが実際の問題のあるステートメントによってブロックされ、実行待ちキューで待たなければならないため、はるかに長い応答時間が発生するからです。

### `ADMIN SHOW SLOW`コマンド

TiDBログファイルに加えて、`ADMIN SHOW SLOW`コマンドを実行して遅いクエリを特定することもできます:

{{< copyable "sql" >}}

```sql
ADMIN SHOW SLOW recent N
ADMIN SHOW SLOW TOP [internal | all] N
```

`recent N`は直近のN個の遅いクエリレコードを表示します。例:

{{< copyable "sql" >}}

```sql
ADMIN SHOW SLOW recent 10
```

`top N`は最近（数日以内）の最も遅いN件のクエリレコードを表示します。`internal`オプションが指定された場合、返される結果はシステムによって実行された内部SQLです。`all`オプションが指定された場合、返される結果はユーザのSQLと内部SQLが結合されたものです。それ以外の場合、このコマンドはユーザのSQLからの遅いクエリレコードのみを返します。

{{< copyable "sql" >}}

```sql
ADMIN SHOW SLOW top 3
ADMIN SHOW SLOW top internal 3
ADMIN SHOW SLOW top all 5
```

TiDBはメモリの制約により、遅いクエリレコードの数を限られた数しか保存しません。クエリコマンドの`N`の値が記録件数よりも大きい場合、返されるレコード数は`N`よりも小さくなります。

次の表は出力の詳細を示しています:

| 列名 | 説明 |
|:------|:---- |
| start | SQL実行の開始時刻 |
| duration | SQL実行の期間 |
| details | SQL実行の詳細 |
| succ | SQLステートメントの実行が成功したかどうか。`1`は成功、`0`は失敗を意味します。 |
| conn_id | セッションの接続ID |
| transaction_ts | トランザクションのコミット時の`commit ts` |
| user | ステートメントの実行を行ったユーザ名 |
| db | ステートメントの実行に関与したデータベース |
| table_ids | SQLステートメントの実行に関与したテーブルのID |
| index_ids | SQLステートメントの実行に関与したインデックスのID |
| internal | これはTiDB内部のSQLステートメントです |
| digest | SQLステートメントのフィンガープリント |
| sql | 実行中または実行されたSQLステートメント |