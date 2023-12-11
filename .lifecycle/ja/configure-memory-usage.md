---
title: TiDB メモリ制御
summary: クエリのメモリクォータの構成方法を学び、OOM（メモリ不足）を回避します。
aliases: ['/docs/dev/configure-memory-usage/','/docs/dev/how-to/configure/memory-control/']
---

# TiDB メモリ制御

現在、TiDB は単一のSQLクエリのメモリクォータを追跡し、特定の閾値値を超えるメモリ使用量になった際にOOM（メモリ不足）を回避したり、トラブルシューティングを行うための対策を講じます。システム変数 [`tidb_mem_oom_action`](/system-variables.md#tidb_mem_oom_action-new-in-v610) は、クエリがメモリ制限に達した際に取るアクションを指定します:

- `LOG` の値は、[`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query) の制限に達した際にクエリは実行を継続しますが、TiDB はログにエントリを出力します。
- `CANCEL` の値は、[`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query) の制限に達した直後にTiDBはSQLクエリの実行を直ちに停止し、クライアントにエラーを返します。エラー情報には、SQL実行プロセスでメモリを消費する各物理実行オペレータのメモリ使用量が明確に表示されます。

## クエリのメモリクォータを構成する

システム変数 [`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query) は、クエリのバイト単位の制限を設定します。以下はいくつかの使用例です:

{{< copyable "sql" >}}

```sql
-- 単一のSQLクエリのメモリクォータの閾値値を8GBに設定:
SET tidb_mem_quota_query = 8 << 30;
```

{{< copyable "sql" >}}

```sql
-- 単一のSQLクエリのメモリクォータの閾値値を8MBに設定:
SET tidb_mem_quota_query = 8 << 20;
```

{{< copyable "sql" >}}

```sql
-- 単一のSQLクエリのメモリクォータの閾値値を8KBに設定:
SET tidb_mem_quota_query = 8 << 10;
```

## tidb-server インスタンスのメモリ使用量の閾値を構成する

v6.5.0から、システム変数 [`tidb_server_memory_limit`](/system-variables.md#tidb_server_memory_limit-new-in-v640) を使用して、tidb-server インスタンスのメモリ使用量の閾値を設定できます。

例えば、tidb-server インスタンスの総メモリ使用量を32GBに設定します:

```sql
SET GLOBAL tidb_server_memory_limit = "32GB";
```

この変数を設定すると、tidb-server インスタンスのメモリ使用量が32GBに達すると、そのインスタンスで実行中の全SQL操作のうち最もメモリを消費するものから順にSQL操作が強制的に終了され、インスタンスのメモリ使用量が32GBを下回るまで繰り返されます。強制的に終了されたSQL操作はクライアントに `Out Of Memory Quota!` エラーを返します。

現在、`tidb_server_memory_limit` によって設定されたメモリ制限は以下のSQL操作には**適用されません**:

- DDL操作
- ウィンドウ関数や共通テーブル式を含むSQL操作

> **警告:**
>
> + 起動プロセス中、TiDB は[`tidb_server_memory_limit`](/system-variables.md#tidb_server_memory_limit-new-in-v640) の制限が強制されることを保証しません。操作システムの空きメモリが不足している場合、TiDBはまだOOMに遭遇する場合があります。TiDB インスタンスに十分な利用可能なメモリがあることを確認する必要があります。
> + メモリ制御のプロセスで、TiDBの総メモリ使用量は、`tidb_server_memory_limit` によって設定された制限をわずかに超える場合があります。
> + v6.5.0以降、設定項目 `server-memory-quota` は非推奨です。互換性を確保するため、クラスタをv6.5.0またはそれ以降のバージョンにアップグレードした後、`tidb_server_memory_limit` は `server-memory-quota` の値を継承します。アップグレード前に`server-memory-quota`を構成していない場合、デフォルト値の `tidb_server_memory_limit` が使用されます（80%）。

tidb-server インスタンスのメモリ使用量が合計メモリ量の一定の割合に達した場合（割合はシステム変数 [`tidb_server_memory_limit_gc_trigger`](/system-variables.md#tidb_server_memory_limit_gc_trigger-new-in-v640) で制御されます）、tidb-server はメモリストレスを緩和するためにGolang GCをトリガしようと試みます。閾値周辺でのインスタンスメモリの変動による性能問題を引き起こす頻繁なGCを回避するため、このGCの方式は毎分最大1回のGCをトリガします。

> **ノート:**
>
> ハイブリッド・デプロイメント・シナリオでは、`tidb_server_memory_limit` は物理マシン全体のトータルメモリの閾値ではなく、単一のtidb-serverインスタンスのメモリ使用量の閾値です。

## INFORMATION_SCHEMA システムテーブルを使用して現在のtidb-server インスタンスのメモリ使用量を表示する

現在のインスタンスまたはクラスタのメモリ使用量を表示するには、システムテーブル [`INFORMATION_SCHEMA.(CLUSTER_)MEMORY_USAGE`](/information-schema/information-schema-memory-usage.md) をクエリできます。

現在のインスタンスまたはクラスタのメモリ関連の操作と実行基準を表示するには、システムテーブル [`INFORMATION_SCHEMA.(CLUSTER_)MEMORY_USAGE_OPS_HISTORY`](/information-schema/information-schema-memory-usage-ops-history.md) をクエリできます。このテーブルでは、各インスタンスについて最新50レコードが保持されます。

## 過剰なメモリ使用の警告をトリガする

tidb-server インスタンスのメモリ使用量がデフォルトで合計メモリの70%を超え、以下の条件のいずれかが検出されると、TiDBは関連する状態ファイルを記録し、警告ログを出力します。

- メモリ使用量がメモリ閾値を初めて超えた場合。
- メモリ使用量がメモリ閾値を超え、最後の警告から60秒以上経過した場合。
- メモリ使用量がメモリ閾値を超え、`(現在のメモリ使用量 - 前回の警告時のメモリ使用量) / トータルメモリ > 10%`。

システム変数 `tidb_memory_usage_alarm_ratio` を介して、警告をトリガするメモリ閾値を制御できます。

過剰なメモリ使用の警告がトリガされると、以下のアクションが実行されます:

- TiDBは、TiDBログファイル [`ファイル名`](/tidb-configuration-file.md#filename) が配置されているディレクトリに以下の情報を記録します。

    - 現在実行中のすべてのSQL文のうち、メモリ使用量が最も高いトップ10のSQL文と実行時間が最も長いトップ10のSQL文の情報
    - goroutineスタック情報
    - ヒープメモリの使用状況

- TiDBは、次のメモリ関連システム変数の値とともに、`tidb-server has the risk of OOM` というキーワードを含む警告ログを出力します。

    - [`tidb_mem_oom_action`](/system-variables.md#tidb_mem_oom_action-new-in-v610)
    - [`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query)
    - [`tidb_server_memory_limit`](/system-variables.md#tidb_server_memory_limit-new-in-v640)
    - [`tidb_analyze_version`](/system-variables.md#tidb_analyze_version-new-in-v510)
    - [`tidb_enable_rate_limit_action`](/system-variables.md#tidb_enable_rate_limit_action)

アラームの状態ファイルが過剰に蓄積されることを避けるため、TiDBはデフォルトで最近5回のアラーム時に生成された状態ファイルのみを保持します。これを調整するには、システム変数 [`tidb_memory_usage_alarm_keep_record_num`](/system-variables.md#tidb_memory_usage_alarm_keep_record_num-new-in-v640) を構成できます。

以下の例は、アラームをトリガするメモリ集約的なSQLステートメントを構築します:

1. `tidb_memory_usage_alarm_ratio` を `0.85` に設定する:

    {{< copyable "" >}}

    ```sql
    SET GLOBAL tidb_memory_usage_alarm_ratio = 0.85;
    ```

2. `CREATE TABLE t(a int);` を実行し、1000行のデータを挿入します。

3. `select * from t t1 join t t2 join t t3 order by t1.a` を実行します。このSQLステートメントは多くのメモリを消費する10億のレコードを出力し、そのためアラームがトリガされます。

4. `tidb.log` ファイルを確認します。このファイルにはシステムの総メモリ、現在のシステムメモリ使用量、tidb-server インスタンスのメモリ使用量、および状態ファイルのディレクトリが記録されています。

    ```
    [2022/10/11 16:39:02.281 +08:00] [WARN] [memoryusagealarm.go:212] ["tidb-server has the risk of OOM because of memory usage exceeds alarm ratio. Running SQLs and heap profile will be recorded in record path"] ["is tidb_server_memory_limit set"=false] ["system memory total"=33682427904] ["system memory usage"=22120655360] ["tidb-server memory usage"=21468556992] [memory-usage-alarm-ratio=0.85] ["record path"=/tiup/deploy/tidb-4000/log/oom_record]
    ```

    上記の例のログファイルのフィールドについては、次のように説明します:

    * `is tidb_server_memory_limit set` は、[`tidb_server_memory_limit`](/system-variables.md#tidb_server_memory_limit-new-in-v640) が設定されているかを示します。
    * `system memory total` は現在のシステムの総メモリを示します。

 
    * `system memory usage`は現在のシステムメモリ使用状況を示します。
    * `tidb-server memory usage`はtidb-serverインスタンスのメモリ使用状況を示します。
    * `memory-usage-alarm-ratio`はシステム変数[`tidb_memory_usage_alarm_ratio`](/system-variables.md#tidb_memory_usage_alarm_ratio)の値を示します。
    * `record path`はステータスファイルのディレクトリを示します。

5. ステータスファイルのディレクトリをチェックすることにより（先の例では、ディレクトリは`/tiup/deploy/tidb-4000/log/oom_record`です）、対応するタイムスタンプ（たとえば、`record2022-10-09T17:18:38+08:00`）が記録されたレコードディレクトリが見つかります。レコードディレクトリには、`goroutinue`、`heap`、`running_sql`という3つのファイルが含まれています。これらの3つのファイルは、ステータスファイルが記録された時間を末尾に付けたものです。それぞれ、ゴールーチンのスタック情報、ヒープメモリの使用状況、およびアラームがトリガーされた際の実行中のSQLの情報が記録されています。`running_sql`の内容については、[`expensive-queries`](/identify-expensive-queries.md)を参照してください。

## tidb-serverの他のメモリ制御動作

### フロー制御

- TiDBはデータを読む演算子のために動的なメモリ制御をサポートしています。デフォルトでは、この演算子はデータを読むために[`tidb_distsql_scan_concurrency`](/system-variables.md#tidb_distsql_scan_concurrency)が許可するスレッドの最大数を使用します。個々のSQL実行のメモリ使用量が[`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query)を超えると、データを読む演算子は1つのスレッドを停止します。

- このフロー制御の動作は、システム変数[`tidb_enable_rate_limit_action`](/system-variables.md#tidb_enable_rate_limit_action)によって制御されます。
- フロー制御の動作がトリガーされると、TiDBは`memory exceeds quota, destroy one token now`というキーワードを含むログを出力します。

### ディスクスパイル

TiDBは実行演算子のためにディスクスパイルをサポートしています。SQL実行のメモリ使用量がメモリクォータを超えると、tidb-serverは実行演算子の中間データをディスクにスパイルしてメモリ圧力を緩和できます。ディスクスパイルをサポートする演算子には、Sort、MergeJoin、HashJoin、およびHashAggが含まれます。

- ディスクスパイルの動作は以下のパラメータによって共同で制御されます：[`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query)、[`tidb_enable_tmp_storage_on_oom`](/system-variables.md#tidb_enable_tmp_storage_on_oom)、[`tmp-storage-path`](/tidb-configuration-file.md#tmp-storage-path)、[`tmp-storage-quota`](/tidb-configuration-file.md#tmp-storage-quota)。
- ディスクスパイルがトリガーされると、TiDBは`memory exceeds quota, spill to disk now`または`memory exceeds quota, set aggregate mode to spill-mode`というキーワードを含むログを出力します。
- Sort、MergeJoin、およびHashJoin演算子のディスクスパイルはv4.0.0で導入されました。HashAgg演算子のディスクスパイルはv5.2.0で導入されました。
- Sort、MergeJoin、またはHashJoinを含むSQL実行がOOMを引き起こすと、TiDBはデフォルトでディスクスパイルをトリガーします。HashAggを含むSQL実行がOOMを引き起こすと、TiDBはデフォルトではディスクスパイルをトリガーしません。HashAggのディスクスパイルをトリガーするには、システム変数`tidb_executor_concurrency = 1`を構成できます。

> **注意:**
>
> HashAggのディスクスパイルは、`DISTINCT`集約関数を含むSQL実行をサポートしていません。`DISTINCT`集約関数を含むSQL実行が大量のメモリを使用する場合、ディスクスパイルは適用されません。

次の例は、HashAggのディスクスパイル機能をデモンストレーションするために、メモリを消費するSQL文を使用しています：

1. SQL文のメモリクォータを1GB（デフォルトは1GB）に設定します：

    {{< copyable "sql" >}}

    ```sql
    SET tidb_mem_quota_query = 1 << 30;
    ```

2. 単一のテーブル`CREATE TABLE t(a int);`を作成し、異なるデータの256行を挿入します。

3. 次のSQL文を実行します：

    {{< copyable "sql" >}}

    ```sql
    [tidb]> explain analyze select /*+ HASH_AGG() */ count(*) from t t1 join t t2 join t t3 group by t1.a, t2.a, t3.a;
    ```

    このSQL文の実行がメモリを大量に使用するため、「Out of Memory Quota」エラーメッセージが返されます：

    ```sql
    ERROR 1105 (HY000): Out Of Memory Quota![conn_id=3]
    ```

4. システム変数`tidb_executor_concurrency`を1に設定します。この構成では、メモリ不足時にHashAggは自動的にディスクスパイルをトリガーしようとします。

    {{< copyable "sql" >}}

    ```sql
    SET tidb_executor_concurrency = 1;
    ```

5. 同じSQL文を実行します。この時、エラーメッセージが返されずに正常に実行されたことがわかります。以下の詳細な実行計画からも、HashAggが600MBのハードディスク容量を使用していることがわかります。

    {{< copyable "sql" >}}

    ```sql
    [tidb]> explain analyze select /*+ HASH_AGG() */ count(*) from t t1 join t t2 join t t3 group by t1.a, t2.a, t3.a;
    ```

    ```sql
    +---------------------------------+-------------+----------+-----------+---------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------+-----------+----------+
    | id                              | estRows     | actRows  | task      | access object | execution info                                                                                                                                                      | operator info                                                   | memory    | disk     |
    +---------------------------------+-------------+----------+-----------+---------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------+-----------+----------+
    | HashAgg_11                      | 204.80      | 16777216 | root      |               | time:1m37.4s, loops:16385                                                                                                                                           | group by:test.t.a, test.t.a, test.t.a, funcs:count(1)->Column#7 | 1.13 GB   | 600.0 MB |
    | └─HashJoin_12                   | 16777216.00 | 16777216 | root      |               | time:21.5s, loops:16385, build_hash_table:{total:267.2µs, fetch:228.9µs, build:38.2µs}, probe:{concurrency:1, total:35s, max:35s, probe:35s, fetch:962.2µs}         | CARTESIAN inner join                                            | 8.23 KB   | 4 KB     |
    |   ├─TableReader_21(Build)       | 256.00      | 256      | root      |               | time:87.2µs, loops:2, cop_task: {num: 1, max: 150µs, proc_keys: 0, rpc_num: 1, rpc_time: 145.1µs, copr_cache_hit_ratio: 0.00}                                       | data:TableFullScan_20                                           | 885 Bytes | N/A      |
    |   │ └─TableFullScan_20          | 256.00      | 256      | cop[tikv] | table:t3      | tikv_task:{time:23.2µs, loops:256}                                                                                                                                  | keep order:false, stats:pseudo                                  | N/A       | N/A      |
    |   └─HashJoin_14(Probe)          | 65536.00    | 65536    | root      |               | time:728.1µs, loops:65, build_hash_table:{total:307.5µs, fetch:277.6µs, build:29.9µs}, probe:{concurrency:1, total:34.3s, max:34.3s, probe:34.3s, fetch:278µs}      | CARTESIAN inner join                                            | 8.23 KB   | 4 KB     |
    |     ├─TableReader_19(Build)     | 256.00      | 256      | root      |               | time:126.2µs, loops:2, cop_task: {num: 1, max: 308.4µs, proc_keys: 0, rpc_num: 1, rpc_time: 295.3µs, copr_cache_hit_ratio: 0.00}                                    | data:TableFullScan_18                                           | 885 Bytes | N/A      |
    |     │ └─TableFullScan_18        | 256.00      | 256      | cop[tikv] | table:t2      | tikv_task:{time:79.2µs, loops:256}                                                                                                                                  | keep order:false, stats:pseudo                                  | N/A       | N/A      |
    |     └─TableReader_17(Probe)     | 256.00      | 256      | root      |               | time:211.1µs, loops:2, cop_task: {num: 1, max: 295.5µs, proc_keys: 0, rpc_num: 1, rpc_time: 279.7µs, copr_cache_hit_ratio: 0.00}                                    | data:TableFullScan_16                                           | 885 Bytes | N/A      |
    |       └─TableFullScan_16        | 256.00      | 256      | cop[tikv] | table:t1      | tikv_task:{time:71.4µs, loops:256}                                                                                                                                  | keep order:false, stats:pseudo                                  | N/A       | N/A      |
    +---------------------------------+-------------+----------+-----------+---------------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------+-----------------------------------------------------------------+-----------+----------+
    9 rows in set (1 min 37.428 sec)
    ```

## その他

### `GOMEMLIMIT` の構成による OOM イシューの緩和

GO 1.19 では、`GOMEMLIMIT` という環境変数が導入されており、GC をトリガーするメモリ制限を設定できます。

v6.1.3 <= TiDB < v6.5.0 の場合、`GOMEMLIMIT` を手動で設定することで、典型的な種類の OOM イシューを緩和できます。典型的な OOM イシューのカテゴリは、OOM 発生前に Grafana で使用される推定メモリが全体のメモリの半分しか占めない場合です（TiDB-Runtime > メモリ使用量 > 推定使用中）、次の図に示されているような状況です:

![通常の OOM ケースの例](/media/configure-memory-usage-oom-example.png)

`GOMEMLIMIT` のパフォーマンスを確認するために、特定のメモリ使用量を `GOMEMLIMIT` の構成有りと無しで比較するテストを実施します。

- TiDB v6.1.2 では、シミュレートされたワークロードがいくつかの分実行された後、TiDB サーバーで OOM (システムメモリ: 約 48 GiB) に遭遇します:

    ![v6.1.2 ワークロード oom](/media/configure-memory-usage-612-oom.png)

- TiDB v6.1.3 では、`GOMEMLIMIT` が 40000 MiB に設定されます。シミュレートされたワークロードが長時間安定して実行され、TiDB サーバーで OOM が発生せず、プロセスの最大メモリ使用量が約 40.8 GiB で安定していることがわかります:

    ![v6.1.3 ワークロード GOMEMLIMIT なしの OOM](/media/configure-memory-usage-613-no-oom.png)