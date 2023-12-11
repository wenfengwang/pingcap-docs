---
title: TiDBのOOMのトラブルシューティング
summary: TiDBのOOM（メモリ不足）の問題を診断して解決する方法を学びます。

# TiDBのOOMのトラブルシューティング

このドキュメントでは、TiDBのOOM（メモリ不足）の問題のトラブルシューティング方法について、現象、原因、解決策、および診断情報について説明します。

## 典型的なOOMの現象

以下は、いくつかの典型的なOOMの現象です。

- クライアント側で次のエラーが報告されます：`SQL error, errno = 2013, state = 'HY000': Lost connection to MySQL server during query`。

- Grafanaのダッシュボードには次の情報が表示されます：
    - **TiDB** > **Server** > **Memory Usage** には、`process/heapInUse` メトリックが継続的に増加し、しきい値に達した後に突然ゼロになります。
    - **TiDB** > **Server** > **Uptime** が突然ゼロになります。
    - **TiDB-Runtime** > **Memory Usage** には、`estimate-inuse` メトリックが継続的に増加しています。

- `tidb.log` をチェックすると、次のログエントリが見つかります:
    - OOMに関する警告：`[WARN] [memory_usage_alarm.go:139] ["tidb-server has the risk of OOM. Running SQLs and heap profile will be recorded in record path"]`。詳細は[`memory-usage-alarm-ratio`](/system-variables.md#tidb_memory_usage_alarm_ratio)を参照してください。
    - 再起動に関するログエントリ：`[INFO] [printer.go:33] ["Welcome to TiDB."]`。

## トラブルシューティング全般のプロセス

OOMの問題をトラブルシューティングする際は、次のプロセスに従ってください:

1. OOMの問題であるかを確認します。

    次のコマンドを実行してオペレーティングシステムのログを確認します。問題が発生したタイミングで `oom-killer` のログがある場合、それがOOMの問題であることを確認できます。

    ```shell
    dmesg -T | grep tidb-server
    ```

    次は、`oom-killer` を含むログの例です:

    ```shell
    ......
    Mar 14 16:55:03 localhost kernel: tidb-server invoked oom-killer: gfp_mask=0x201da, order=0, oom_score_adj=0
    Mar 14 16:55:03 localhost kernel: tidb-server cpuset=/ mems_allowed=0
    Mar 14 16:55:03 localhost kernel: CPU: 14 PID: 21966 Comm: tidb-server Kdump: loaded Not tainted 3.10.0-1160.el7.x86_64 #1
    Mar 14 16:55:03 localhost kernel: Hardware name: QEMU Standard PC (i440FX + PIIX, 1996), BIOS rel-1.14.0-0-g155821a1990b-prebuilt.qemu.org 04/01/2014
    ......
    Mar 14 16:55:03 localhost kernel: Out of memory: Kill process 21945 (tidb-server) score 956 or sacrifice child
    Mar 14 16:55:03 localhost kernel: Killed process 21945 (tidb-server), UID 1000, total-vm:33027492kB, anon-rss:31303276kB, file-rss:0kB, shmem-rss:0kB
    Mar 14 16:55:07 localhost systemd: tidb-4000.service: main process exited, code=killed, status=9/KILL
    ......

2. OOMの問題であることが確認できたら、OOMがデプロイまたはデータベースの問題によるものかをさらに調査できます。

    - もしデプロイメントの問題によるOOMである場合、リソース構成やハイブリッドデプロイメントの影響を調査する必要があります。
    - もしデータベースの問題によるOOMである場合、次のいくつかの可能な原因が考えられます:
        - TiDBが大きなデータトラフィック（大きなクエリ、大きな書き込み、およびデータのインポート）を処理している。
        - TiDBが高並列シナリオで、複数のSQLステートメントがリソースを同時に消費するか、オペレータの並列実行が高い。
        - TiDBにメモリリークがあり、リソースが解放されていない。

    具体的なトラブルシューティング方法については、次のセクションを参照してください。

## 典型的な原因と解決策

OOMの問題は通常、次のような理由によって引き起こされます:

- [デプロイメントの問題](#deployment-issues)
- [データベースの問題](#database-issues)
- [クライアント側の問題](#client-side-issues)

### デプロイメントの問題

不適切なデプロイメントによるOOMの原因として、次の点が考えられます:

- オペレーティングシステムのメモリ容量が不十分である。
- TiUP構成の [`resource_control`](/tiup/tiup-cluster-topology-reference.md#global) が適切でない。
- ハイブリッドデプロイメントの場合（TiDBおよび他のアプリケーションが同じサーバーに展開されている場合）、リソースが不足して `oom-killer` によってTiDBが不慮の事故により終了することがあります。

### データベースの問題

このセクションでは、データベースの問題によるOOMの原因と解決策について説明します。

> **注意:**
>
> [`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query) を設定している場合、エラーが発生します：`ERROR 1105 (HY000): Out of Memory Quota![conn_id=54]`。これはデータベースのメモリ使用制御の挙動によるものであり、正常な挙動です。

#### SQLステートメントの実行によるメモリの消費が過剰

OOMの問題の異なる原因に応じて、以下の措置を取ることでSQLステートメントのメモリ使用量を削減できます。

- SQLの実行プランが最適でない場合、例えば適切なインデックスがない、古い統計情報、または最適化プログラムのバグにより、誤ったSQLの実行プランが選択される可能性があります。その結果、膨大な中間結果セットがメモリに蓄積されます。この場合、次の対策を検討してください:
    - 適切なインデックスを追加します。
    - 実行演算子に対する [ディスクスパイル](/configure-memory-usage.md#disk-spill) 機能を使用します。
    - テーブル間のJOIN順序を調整します。
    - ヒントを使用してSQLステートメントを最適化します。

- 一部のオペレータや関数はストレージレベルにプッシュダウンされることがサポートされておらず、膨大な中間結果セットが発生します。この場合、SQLステートメントを洗練するか、ヒントを使用して最適化し、プッシュダウンをサポートする関数やオペレータを使用します。

- 実行プランにオペレータHashAggが含まれています。HashAggは複数のスレッドで並行実行されますが、これによりメモリがより多く消費されます。代わりに `STREAM_AGG()` を使用できます。

- 同時に読み取るリージョンの数を減らすか、オペレータの並行性を減らして、高並行性によるメモリ問題を回避します。対応するシステム変数には次のものがあります:
    - [`tidb_distsql_scan_concurrency`](/system-variables.md#tidb_distsql_scan_concurrency)
    - [`tidb_index_serial_scan_concurrency`](/system-variables.md#tidb_index_serial_scan_concurrency)
    - [`tidb_executor_concurrency`](/system-variables.md#tidb_executor_concurrency-new-in-v50)

- 問題が発生する時点でのセッションの並列性が高すぎる場合は、TiDBクラスターをスケールアウトして、より多くのTiDBノードを追加することを検討してください。

#### 大規模なトランザクションや大規模な書き込みによるメモリの消費が過剰

メモリ容量の計画が必要です。トランザクションを実行する際、TiDBプロセスのメモリ使用量は、トランザクションサイズと比較して2〜3倍以上にスケーリングアップされます。

単一の大規模なトランザクションを複数の小さなトランザクションに分割することができます。

#### 統計情報の収集およびロードプロセスによるメモリの消費が過剰

TiDBノードは起動後に統計情報をメモリにロードする必要があります。統計情報を収集する際にTiDBはメモリを消費します。以下の方法でメモリ使用量を制御できます:

- サンプリング率を指定し、特定の列のみ統計情報を収集し、 `ANALYZE` の並列性を減少させます。
- TiDB v6.1.0以降では、システム変数 [`tidb_stats_cache_mem_quota`](/system-variables.md#tidb_stats_cache_mem_quota-new-in-v610) を使用して統計情報のメモリ使用量を制御できます。
- TiDB v6.1.0以降では、システム変数 [`tidb_mem_quota_analyze`](/system-variables.md#tidb_mem_quota_analyze-new-in-v610) を使用して、TiDBが統計情報を更新する際の最大メモリ使用量を制御できます。

詳細については、[統計情報の紹介](/statistics.md) ページを参照してください。

#### プリペアドステートメントの過剰な使用

クライアント側がプリペアドステートメントを作成し続けて `deallocate prepare stmt` を実行しないため、メモリ消費が継続的に増加し、最終的にTiDB OOMがトリガーされます。プリペアドステートメントが占有するメモリは、セッションが閉じられるまで解放されません。これは特に長時間の接続セッションにとって重要です。

問題を解決するため、次の措置を検討してください:

- セッションのライフサイクルを調整します。
- 接続プールの `wait_timeout` および `max_execution_time` を調整します。
- システム変数 [`max_prepared_stmt_count`](/system-variables.md#max_prepared_stmt_count) を使用してセッション内のプリペアドステートメントの最大数を制御します。

#### `tidb_enable_rate_limit_action` が適切に構成されていない場合
システム変数[`tidb_enable_rate_limit_action`](/system-variables.md#tidb_enable_rate_limit_action)は、SQLステートメントがデータを読み取る場合にメモリ使用量を効果的に制御します。この変数が有効になっており、計算操作（例: 結合または集計操作）が必要な場合、メモリ使用量は[`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query)の制御下にない可能性があり、OOMのリスクが高まります。

このシステム変数を無効にすることをお勧めします。TiDB v6.3.0から、このシステム変数はデフォルトで無効になっています。

### クライアント側の問題

クライアント側でOOMが発生した場合は、以下を調査してください:

- **Grafana TiDB Details** > **Server** > **Client Data Traffic**でトレンドと速度をチェックし、ネットワークのブロックがあるかどうかを確認してください。
- JDBC構成パラメータによるアプリケーションOOMが発生していないかを確認してください。例えば、ストリーミング読み取り用の`defaultFetchSize`パラメータが誤って構成されていると、クライアント側でデータが大量に蓄積される原因になります。

## OOM問題のトラブルシューティングに収集すべき診断情報

OOM問題の原因を特定するには、以下の情報を収集する必要があります:

- オペレーティングシステムのメモリ関連の設定を収集する:
    - TiUP設定: `resource_control.memory_limit`
    - オペレーティングシステムの設定:
        - メモリ情報: `cat /proc/meminfo`
        - カーネルパラメータ: `vm.overcommit_memory`
    - NUMA情報:
        - `numactl --hardware`
        - `numactl --show`

- データベースのバージョン情報とメモリ関連の設定を収集する:
    - TiDBバージョン
    - `tidb_mem_quota_query`
    - `memory-usage-alarm-ratio`
    - `mem-quota-query`
    - `oom-action`
    - `tidb_enable_rate_limit_action`
    - `tidb_server_memory_limit`
    - `oom-use-tmp-storage`
    - `tmp-storage-path`
    - `tmp-storage-quota`
    - `tidb_analyze_version`

- GrafanaダッシュボードのTiDBメモリの日ごとの使用状況をチェックする: **TiDB** > **Server** > **Memory Usage**.

- より多くのメモリを消費するSQLステートメントをチェックする:
    - TiDBダッシュボードでSQLステートメントの解析、遅いクエリ、およびメモリ使用量を表示します。
    - `INFORMATION_SCHEMA`の`SLOW_QUERY`および`CLUSTER_SLOW_QUERY`をチェックする。
    - 各TiDBノードでの`tidb_slow_query.log`をチェックする。
    - `grep "expensive_query" tidb.log`を実行して、対応するログエントリをチェックします。
    - `EXPLAIN ANALYZE`を実行して、オペレータのメモリ使用量をチェックします。
    - `SELECT * FROM information_schema.processlist;`を実行して、`MEM`列の値をチェックします。

- メモリ使用量が高いときにTiDBプロファイル情報を収集するには、次のコマンドを実行します:

    ```shell
    curl -G http://{TiDBIP}:10080/debug/zip?seconds=10" > profile.zip
    ```

- `grep "tidb-server has the risk of OOM" tidb.log`を実行して、TiDB Serverが収集したアラートファイルのパスをチェックします。以下は例です:

    ```shell
    ["tidb-server has the risk of OOM. Running SQLs and heap profile will be recorded in record path"] ["is tidb_server_memory_limit set"=false] ["system memory total"=14388137984] ["system memory usage"=11897434112] ["tidb-server memory usage"=11223572312] [memory-usage-alarm-ratio=0.8] ["record path"="/tmp/0_tidb/MC4wLjAuMDo0MDAwLzAuMC4wLjA6MTAwODA=/tmp-storage/record"]
    ```

## 関連情報

- [TiDB Memory Control](/configure-memory-usage.md)
- [Tune TiKV Memory Parameter Performance](/tune-tikv-memory-performance.md)