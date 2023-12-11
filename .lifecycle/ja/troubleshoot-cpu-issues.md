---
title: リードおよびライトの遅延のトラブルシューティング
summary: リードおよびライトの遅延の問題のトラブルシューティング方法を学びます。

# リードおよびライトの遅延のトラブルシューティング

このドキュメントでは、リードおよびライトの遅延およびジッターの可能性の原因と、これらの問題のトラブルシューティング方法について紹介します。

## 一般的な原因

### 正しくない TiDB 実行計画

クエリの実行計画が安定せず、正しくないインデックスを選択する可能性があり、それが遅延を引き起こすことがあります。

#### 現象

* クエリの実行計画がスローログに出力されている場合は、計画を直接確認できます。詳細な実行計画を解析するには、`select tidb_decode_plan('xxx...')` ステートメントを実行します。
* モニターでスキャンされるキーの数が異常に増加し、スローログで `Scan Keys` の数が多いです。
* TiDB における SQL の実行期間が MySQL などの他のデータベースと大幅に異なっています。他のデータベースの実行計画を比較できます（例: `Join Order` が異なるかどうか）。

#### 可能性のある原因

統計情報が不正確です。

#### トラブルシューティング方法

* 統計情報を更新します
    * `analyze table` を手動で実行し、`crontab` コマンドで定期的に `analyze` を実行して統計情報を正確に保ちます。
    * `auto analyze` を自動的に実行します。`analyze ratio` の閾値値を下げ、情報収集の頻度を増やし、実行の開始時間と終了時間を設定します。以下は例です:
        * `set global tidb_auto_analyze_ratio=0.2;`
        * `set global tidb_auto_analyze_start_time='00:00 +0800';`
        * `set global tidb_auto_analyze_end_time='06:00 +0800';`
* 実行計画をバインドします
    * アプリケーションの SQL ステートメントを修正し、`use index` を実行して列のインデックスを一貫して使用します。
    * 3.0 バージョンでは、アプリケーションの SQL ステートメントを修正する必要はありません。`create global binding` を使用して `force index` のバインディング SQL ステートメントを作成します。
    * 4.0 バージョンでは、[SQL Plan Management](/sql-plan-management.md) がサポートされており、安定しない実行計画によるパフォーマンスの低下を回避できます。

### PD 異常

#### 現象

PD TSO の `wait duration` メトリックが異常に増加しています。このメトリックは、PD からの要求の返信を待つ期間を表します。

#### 可能性のある原因

* ディスクの問題。PD ノードが配置されているディスクが I/O 負荷でいっぱいになっています。PD が他の高い I/O 要求を持つコンポーネントと一緒に展開されているか、ディスクの健康状態を調査します。必要に応じて、モニターメトリックを **Grafana** -> **disk performance** -> **latency**/**load** で確認したり、必要に応じて FIO ツールを使用してディスクをチェックします。

* PD ピア間のネットワークの問題。PD ログに `lost the TCP streaming connection` が表示されます。PD ノード間のネットワークに問題があるかどうかを確認し、`round trip` をモニター **Grafana** -> **PD** -> **etcd** で確認して原因を検証します。

* サーバーの負荷が高い。ログに `server is likely overloaded` と表示されます。

* PD がリーダーを選出できない: PD ログに `lease is not expired` と表示されます。この問題は v3.0.x および v2.1.19 で修正されています。

* リーダー選出が遅い。リージョンの読み込み時間が長いです。PD ログで `grep "regions cost"` を実行してこの問題を確認できます。結果が `load 460927 regions cost 11.77099s` のように秒単位の数値である場合、リージョンの読み込みが遅いことを意味します。v3.0 で `use-region-storage` を `true` に設定することでリージョンの読み込み時間を大幅に削減できます。

* TiDB と PD 間のネットワークの問題。モニター **Grafana** -> **blackbox_exporter** -> **ping latency** にアクセスして、TiDB から PD リーダーへのネットワークが正常に実行されているかどうかを確認します。

* PD が `FATAL` エラーを報告し、ログに `range failed to find revision pair` と表示されています。この問題は v3.0.8 で修正されています ([#2040](https://github.com/pingcap/pd/pull/2040))。

* `/api/v1/regions` インターフェースを使用すると、多くのリージョンが PD OOM を引き起こす可能性があります。この問題は v3.0.8 で修正されています ([#1986](https://github.com/pingcap/pd/pull/1986))。

* ローリングアップグレード中に PD が OOM になります。gRPC メッセージのサイズが制限されていないため、モニターには `TCP InSegs` が比較的大きく表示されます。この問題は v3.0.6 で修正されています ([#1952](https://github.com/pingcap/pd/pull/1952))。

* PD がパニックになります。[バグを報告](https://github.com/tikv/pd/issues/new?labels=kind/bug&template=bug-report.md)します。

* その他の原因。`curl http://127.0.0.1:2379/debug/pprof/goroutine?debug=2` を実行して goroutine を取得し、[バグを報告](https://github.com/pingcap/pd/issues/new?labels=kind%2Fbug&template=bug-report.md)します。

### TiKV 異常

#### 現象

モニターでの `KV Cmd Duration` メトリックが異常に増加しています。このメトリックは、TiDB が TiKV にリクエストを送信してから TiDB が応答を受信するまでの期間を表します。

#### 可能性のある原因

* `gRPC duration` メトリックを確認します。このメトリックは、TiKV での gRPC リクエストの総期間を表します。TiKV の `gRPC duration` と TiDB の `KV duration` を比較することで、潜在的なネットワークの問題を特定できます。TiDB の `KV duration` が長い場合、TiDB と TiKV 間のネットワークの遅延が高い可能性があるか、TiDB と TiKV 間の NIC バンド幅が完全に占有されている可能性があります。

* TiKV が再起動されるための再選出。
    * TiKV がパニックになり、`systemd` によって引き上げられて正常に動作する。TiKV ログを確認してパニックが発生したかどうかを確認できます。この問題は予期しないものですので、発生した場合は [バグを報告](https://github.com/tikv/tikv/issues/new?template=bug-report.md) します。
    * TiKV が第三者によって停止または強制終了され、その後 `systemd` によって引き上げられる。`dmesg` および TiKV ログを確認して原因を調査します。
    * TiKV が OOM になり、再起動されます。
    * TiKV は `THP`（Transparent Hugepage）の動的調整のためにハングアップしています。

* モニターを確認: TiKV RocksDB が書き込みスタールに遭遇し、再選出が発生します。モニター **Grafana** -> **TiKV-details** -> **errors** で `server is busy` が表示されているかどうかを確認します。

* ネットワークの孤立による再選出。

* `block-cache` 構成が大きすぎると、TiKV が OOM を引き起こす可能性があります。問題の原因を確認するには、モニター **Grafana** -> **TiKV-details** で対応するインスタンスを選択して RocksDB の `block cache size` を確認します。同時に、`[storage.block-cache] capacity = # "1GB"` パラメータが適切に設定されているかを確認します。デフォルトでは、TiKV の `block-cache` は物理マシンの合計メモリの `45%` に設定されます。コンテナに TiKV を展開する場合は、コンテナのメモリ制限を超える可能性があるため、このパラメータを明示的に指定する必要があります。

* コプロセッサが多くの大きなクエリを受信し、大量のデータを返します。gRPC はコプロセッサがデータを返す速度よりも迅速にデータを送信できないため、OOM が発生します。問題の原因を確認するには、モニター **Grafana** -> **TiKV-details** -> **coprocessor overview** で `response size` が `network outbound` トラフィックを超えていないかを確認します。

### TiKV スレッドのボトルネック

TiKV にはボトルネックになる可能性があるスレッドがいくつかあります。

* TiKV インスタンスのリージョンが多すぎて単一の gRPC スレッドがボトルネックになる場合（**Grafana** -> **TiKV-details** -> **Thread CPU/gRPC CPU Per Thread** メトリックを確認）、バージョン 3.x 以降では `Hibernate Region` を有効にできます。
* v3.0 より前のバージョンでは、raftstore スレッドまたは apply スレッドがボトルネックになる場合（**Grafana** -> **TiKV-details** -> **Thread CPU/raft store CPU** および **Async apply CPU** メトリックが `80%` を超える）、TiKV（v2.x）インスタンスをスケールアウトするか、マルチスレッディングにアップグレードします。

### CPU 負荷の増加

#### 現象

CPU リソースの使用がボトルネックになります。

#### 可能性のある原因

* ホットスポットの問題
* 全体的な負荷が高いです。TiDB の遅いクエリと高コストなクエリを確認します。インデックスを追加したり、クエリをバッチで実行するなど、実行クエリを最適化します。別の解決策としては、クラスタをスケールアウトすることもあります。

## その他の原因

### クラスタのメンテナンス
```
Most of each online cluster has three or five nodes. If the machine to be maintained has the PD component, you need to determine whether the node is the leader or the follower. Disabling a follower has no impact on the cluster operation. Before disabling a leader, you need to switch the leadership. During the leadership change, performance jitter of about 3 seconds will occur.

### Minority of replicas are offline

By default, each TiDB cluster has three replicas, so each Region has three replicas in the cluster. These Regions elect the leader and replicate data through the Raft protocol. The Raft protocol ensures that TiDB can still provide services without data loss even when the nodes (that are fewer than half of replicas) fail or are isolated. For the cluster with three replicas, the failure of one node might cause performance jitter but the usability and correctness in theory are not affected.

### New indexes

Creating indexes consumes a huge amount of resources when TiDB scans tables and backfills indexes. Index creation might even conflict with the frequently updated fields, which affects the application. Creating indexes on a large table often takes a long time, so you must try to balance the index creation time and the cluster performance (for example, creating indexes at the off-peak time).

**Parameter adjustment:**

Currently, you can use `tidb_ddl_reorg_worker_cnt` and `tidb_ddl_reorg_batch_size` to dynamically adjust the speed of index creation. Usually, the smaller the values, the smaller the impact on the system, with longer execution time though.

In general cases, you can first keep their default values (`4` and `256`), observe the resource usage and response speed of the cluster, and then increase the value of `tidb_ddl_reorg_worker_cnt` to increase the concurrency. If no obvious jitter is observed in the monitor, increase the value of `tidb_ddl_reorg_batch_size`. If the columns involved in the index creation are frequently updated, the many resulting conflicts will cause the index creation to fail and be retried.

In addition, you can also set the value of `tidb_ddl_reorg_priority` to `PRIORITY_HIGH` to prioritize the index creation and speed up the process. But in the general OLTP system, it is recommended to keep its default value.

### High GC pressure

The transaction of TiDB adopts the Multi-Version Concurrency Control (MVCC) mechanism. When the newly written data overwrites the old data, the old data is not replaced, and both versions of data are stored. Timestamps are used to mark different versions. The task of GC is to clear the obsolete data.

* In the phase of Resolve Locks, a large amount of `scan_lock` requests are created in TiKV, which can be observed in the gRPC-related metrics. These `scan_lock` requests call all Regions.
* In the phase of Delete Ranges, a few (or no) `unsafe_destroy_range` requests are sent to TiKV, which can be observed in the gRPC-related metrics and the **GC tasks** panel.
* In the phase of Do GC, each TiKV by default scans the leader Regions on the machine and performs GC to each leader, which can be observed in the **GC tasks** panel.
```