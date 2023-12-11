---
title: TiDBクラスターアラートルール
summary: TiDBクラスターのアラートルールについて学ぶ
aliases: ['/docs/dev/alert-rules/','/docs/dev/reference/alert-rules/']
---

<!-- markdownlint-disable MD024 -->

# TiDBクラスターアラートルール

このドキュメントでは、TiDBクラスター内の異なるコンポーネントのアラートルールについて説明し、TiDB、TiKV、PD、TiFlash、TiDB Binlog、TiCDC、Node_exporter、Blackbox_exporterのアラート項目のルールと解決策を含みます。

深刻度レベルに応じて、アラートルールは3つのカテゴリ（高、中、低）に分類されます。この深刻度レベルの分類は、以下の各コンポーネントのすべてのアラート項目に適用されます。

|  深刻度レベル |  説明   |
| :-------- | :----- |
|  高  |  サービスが利用できない最も深刻なレベルです。緊急時のアラートは、サービスやノードの障害から通常発生します。**すぐに手動での介入が必要です**。 |
|  中  |  サービスの可用性が低下しています。中のアラートでは、異常なメトリクスを注意深く監視する必要があります。 |
|  低  |  低のアラートは、問題やエラーのリマインダーです。  |

## TiDBアラートルール

このセクションでは、TiDBコンポーネントのアラートルールを示します。

### 緊急レベルアラート

#### `TiDB_schema_error`

* アラートルール：

    `increase(tidb_session_schema_lease_error_total{type="outdated"}[15m]) > 0`

* 説明：

    最新のスキーマ情報がTiDBで1つのリース内に再読み込みされません。TiDBがサービスを提供し続けることに失敗した場合、アラートがトリガーされます。

* 解決策：

    通常は、利用できないリージョンまたはTiKVのタイムアウトによって引き起こされます。TiKVの監視項目を確認して問題を特定する必要があります。

#### `TiDB_tikvclient_region_err_total`

* アラートルール：

    `increase(tidb_tikvclient_region_err_total[10m]) > 6000`

* 説明：

    TiDBサーバーは、TiKVのリージョンリーダーに、各自のキャッシュ情報に従ってアクセスします。リージョンリーダーが変更されたり、TiDBのキャッシュとTiKVのリージョン情報が一致しないと、リージョンキャッシュエラーが発生します。10分間にエラーが6000回を超えると、アラートがトリガーされます。

* 解決策：

    [**TiKV-Details** > **Cluster** dashboard](/grafana-tikv-dashboard.md#cluster)を表示して、リーダーが均等であるかどうかを確認してください。

#### `TiDB_domain_load_schema_total`

* アラートルール：

    `increase(tidb_domain_load_schema_total{type="failed"}[10m]) > 10`

* 説明：

    TiDBで最新のスキーマ情報の再読み込みに失敗した回数の合計です。10分間にリロードの失敗が10回を超えると、アラートがトリガーされます。

* 解決策：

    [`TiDB_schema_error`](#tidb_schema_error)と同じです。

#### `TiDB_monitor_keep_alive`

* アラートルール：

    `increase(tidb_monitor_keep_alive_total[10m]) < 100`

* 説明：

    TiDBプロセスがまだ存在しているかどうかを示します。`tidb_monitor_keep_alive_total`の回数が10分間に100回未満に増加すると、TiDBプロセスは既に終了している可能性があり、アラートがトリガーされます。

* 解決策：

    * TiDBプロセスがメモリ不足かどうかを確認してください。
    * マシンが再起動したかどうかを確認してください。

### 中レベルアラート

#### `TiDB_server_panic_total`

* アラートルール：

    `increase(tidb_server_panic_total[10m]) > 0`

* 説明：

    パニックしたTiDBスレッドの数です。パニックが発生すると、アラートがトリガーされます。スレッドは通常回復しますが、そうでない場合は、TiDBが頻繁に再起動します。

* 解決策：

    パニックログを収集して、問題を特定してください。

### 低レベルアラート

#### `TiDB_memory_abnormal`

* アラートルール：

    `go_memstats_heap_inuse_bytes{job="tidb"} > 1e+10`

* 説明：

    TiDBのメモリ使用量を監視します。使用量が10Gを超えると、アラートがトリガーされます。

* 解決策：

    HTTP APIを使用して、goroutineのリーク問題をトラブルシューティングしてください。

#### `TiDB_query_duration`

* アラートルール：

    `histogram_quantile(0.99, sum(rate(tidb_server_handle_query_duration_seconds_bucket[1m])) BY (le, instance)) > 1`

* 説明:

    TiDBでのリクエスト処理の遅延時間です。99パーセンタイルの遅延時間が1秒を超えると、アラートがトリガーされます。

* 解決策:

    TiDBログを表示し、遅いSQLクエリを特定するために、`SLOW_QUERY`および`TIME_COP_PROCESS`のキーワードを検索してください。

#### `TiDB_server_event_error`

* アラートルール：

    `increase(tidb_server_event_total{type=~"server_start|server_hang"}[15m]) > 0`

* 説明:

    TiDBサービスで発生するイベントの数です。以下のイベントが発生した場合、アラートがトリガーされます。

    1. start: TiDBサービスが起動します。
    2. hang: 重大なイベント（現在のところTiDBがバイナリログを書き込めないシナリオのみがあります）が発生したとき、TiDBは`hang`モードに入り、手動で停止されるまで待機します。

* 解決策:

    * サービスを回復するために、TiDBを再起動してください。
    * TiDB Binlogサービスが正常かどうかを確認してください。

#### `TiDB_tikvclient_backoff_seconds_count`

* アラートルール：

    `increase(tidb_tikvclient_backoff_seconds_count[10m]) > 10`

* 説明:

    TiDBがTiKVにアクセスできない場合のリトライ回数です。10分間にリトライ回数が10回を超えると、アラートがトリガーされます。

* 解決策:

    TiKVの監視状態を確認してください。

#### `TiDB_monitor_time_jump_back_error`

* アラートルール：

    `increase(tidb_monitor_time_jump_back_total[10m]) > 0`

* 説明:

    TiDBを保持しているマシンの時間が巻き戻ると、アラートがトリガーされます。

* 解決策:

    NTPの構成をトラブルシューティングしてください。

#### `TiDB_ddl_waiting_jobs`

* アラートルール：

    `sum(tidb_ddl_waiting_jobs) > 5`

* 説明:

    TiDBで実行待ちのDDLタスクの数が5を超えると、アラートがトリガーされます。

* 解決策:

    実行中の`admin show ddl`を実行して、実行待ちの`add index`操作があるかどうかを確認してください。

## PDアラートルール

このセクションでは、PDコンポーネントのアラートルールを示します。

### 緊急レベルアラート

#### `PD_cluster_down_store_nums`

* アラートルール：

    `(sum(pd_cluster_status{type="store_down_count"}) by (instance) > 0) and (sum(etcd_server_is_leader) by (instance) > 0)`

* 説明:

    PDは、一定時間（デフォルト構成は30分）TiKV/TiFlashからのハートビートを受信していません。

* 解決策:

    * TiKV/TiFlashプロセスが正常かどうか、ネットワークが分離されているか、負荷が高いかを確認し、可能な限りサービスを回復してください。
    * TiKV/TiFlashインスタンスを回復できない場合は、オフラインにすることができます。

### 中レベルアラート

#### `PD_etcd_write_disk_latency`

* アラートルール：

    `histogram_quantile(0.99, sum(rate(etcd_disk_wal_fsync_duration_seconds_bucket[1m])) by (instance, job, le)) > 1`

* 説明:

    fsync操作のレイテンシが1秒を超えると、etcdがデータを通常よりも低い速度でディスクに書き込むことを示します。これにより、PDのリーダータイムアウトが発生したり、TSOを時限にディスクに保存できなくなり、クラスター全体のサービスがシャットダウンする可能性があります。

* 解決策:

    * 遅い書き込みの原因を見つけてください。他のサービスがシステムを過負荷にしている可能性があります。PD自体が大量のCPUやI/Oリソースを占有していないか確認できます。
    * PDを再起動して、または別のPDに手動でリーダーを転送してサービスを回復してください。
    * 環境要因によって問題のあるPDインスタンスが回復できない場合は、オフラインにして置き換えてください。

#### `PD_miss_peer_region_count`

* アラートルール：

    `(sum(pd_regions_status{type="miss-peer-region-count"}) by (instance) > 100) and (sum(etcd_server_is_leader) by (instance) > 0)`

* 説明:

    リージョンレプリカの数が`max-replicas`の値よりも小さいです。

* 解決策:

    ダウンまたはオフラインになっているTiKVマシンがあるかどうかを確認し、`miss-peer-region-count`が連続して減少しているかどうかをリージョンヘルスパネルで確認してください。

### 低レベルアラート

#### `PD_cluster_lost_connect_store_nums`

* アラートルール：

    `(sum(pd_cluster_status{type="store_disconnected_count"}) by (instance) > 0) and (sum(etcd_server_is_leader) by (instance) > 0)`

* 説明:
    PDは20秒以内にTiKV/TiFlashのハートビートを受信していません。通常、TiKV/TiFlashのハートビートは10秒ごとに発生します。

* 解決策：

    * TiKV/TiFlashのインスタンスが再起動されていないかどうかを確認してください。
    * TiKV/TiFlashのプロセスが正常か、ネットワークが隔離されていないか、負荷が高すぎないかを確認し、サービスをできるだけ回復させてください。
    * TiKV/TiFlashのインスタンスが回復できないことが確認された場合、オフラインにすることができます。
    * TiKV/TiFlashのインスタンスが回復可能であることが確認された場合、ただし短期間ではない場合、`max-down-time`の値を増やすことを検討できます。これにより、TiKV/TiFlashのインスタンスが回復不能と見なされ、データがTiKV/TiFlashから削除されるのを防ぎます。

#### `PD_cluster_unhealthy_tikv_nums`

* アラートルール：

    `(sum(pd_cluster_status{type="store_unhealth_count"}) by (instance) > 0) and (sum(etcd_server_is_leader) by (instance) > 0)`

* 説明：

    不健全なストアがあることを示します。一定時間（[`max-store-down-time`](/pd-configuration-file.md#max-store-down-time)で構成され、デフォルトは`30m`）この状況が続くと、ストアはおそらく`オフライン`状態に変化し、[`PD_cluster_down_store_nums`](#pd_cluster_down_store_nums)アラートをトリガーします。

* 解決策：

    TiKVストアの状態を確認してください。

#### `PD_cluster_low_space`

* アラートルール：

    `(sum(pd_cluster_status{type="store_low_space_count"}) by (instance) > 0) and (sum(etcd_server_is_leader) by (instance) > 0)`

* 説明：

    TiKV/TiFlashノードの十分なスペースがないことを示します。

* 解決策：

    * クラスタ内のスペースが一般的に不足していないかを確認してください。そうであれば、その容量を増やしてください。
    * リージョンのバランススケジューリングに問題がないかを確認してください。そうであれば、データ分布が不均等になります。
    * ログ、スナップショット、コアダンプなど、多量のディスクスペースを占有しているファイルがないかを確認してください。
    * ノードのリージョンウェイトを下げて、データ量を減らしてください。
    * スペースを解放できない場合は、積極的にノードをオフラインにすることを検討してください。これにより、ディスクスペースが不足してダウンタイムが発生するのを防ぎます。

#### `PD_etcd_network_peer_latency`

* アラートルール：

    `histogram_quantile(0.99, sum(rate(etcd_network_peer_round_trip_time_seconds_bucket[1m])) by (To, instance, job, le)) > 1`

* 説明：

    PDノード間のネットワークレイテンシが高いです。これは、リーダータイムアウトやTSOディスクストレージタイムアウトにつながる可能性があり、クラスタのサービスに影響を与える可能性があります。

* 解決策：

    * ネットワークとシステムの負荷状況を確認してください。
    * 問題のあるPDインスタンスが環境要因で回復できない場合は、オフラインにして置き換えてください。

#### `PD_tidb_handle_requests_duration`

* アラートルール：

    `histogram_quantile(0.99, sum(rate(pd_client_request_handle_requests_duration_seconds_bucket{type="tso"}[1m])) by (instance, job, le)) > 0.1`

* 説明：

    PDがTSO要求を処理するのに時間がかかっています。これは通常、負荷が高いために引き起こされます。

* 解決策：

    * サーバーの負荷状況を確認してください。
    * PDのCPUプロファイルを分析するためにpprofを使用してください。
    * PDのリーダーを手動で切り替えてください。
    * 環境要因により問題のあるPDインスタンスが回復できない場合は、オフラインにして置き換えてください。

#### `PD_down_peer_region_nums`

* アラートルール：

    `(sum(pd_regions_status{type="down-peer-region-count"}) by (instance)  > 0) and (sum(etcd_server_is_leader) by (instance) > 0)`

* 説明：

    Raftリーダーによって無応答のピアを持つリージョンの数。

* 解決策：

    * ダウンしているTiKVがあるか、または再起動されたか、またはビジーであるかどうかを確認してください。
    * リージョンヘルスパネルを監視し、`down_peer_region_count`が連続して減少しているかどうかを確認してください。
    * TiKVサーバー間のネットワークを確認してください。

#### `PD_pending_peer_region_count`

* アラートルール：

    `(sum(pd_regions_status{type="pending-peer-region-count"}) by (instance) > 100) and (sum(etcd_server_is_leader) by (instance) > 0)`

* 説明：

    ラフトログが遅れているリージョンが多すぎることを示します。スケジューリングによってわずかな数のペンディングピアになることは正常ですが、その数が高いままである場合、問題がある可能性があります。

* 解決策：

    * リージョンヘルスパネルを監視し、`pending_peer_region_count`が連続して減少しているかどうかを確認してください。
    * 特に帯域幅が十分であるかどうかを含め、TiKVサーバー間のネットワークを確認してください。

#### `PD_leader_change`

* アラートルール：

    `count(changes(pd_tso_events{type="save"}[10m]) > 0) >= 2`

* 説明：

    PDのリーダーが最近切り替わりました。

* 解決策：

    リーダーを再起動したり、手動でリーダーを転送したり、リーダーの優先順位を調整したりするなど、人為的な要因を除外してください。
    ネットワークとシステムの負荷状況を確認してください。
    環境要因で問題のあるPDインスタンスが回復できない場合は、オフラインにして置き換えてください。

#### `TiKV_space_used_more_than_80%`

* アラートルール：

    `sum(pd_cluster_status{type="storage_size"}) / sum(pd_cluster_status{type="storage_capacity"}) * 100 > 80`

* 説明：

    クラスタースペースの80%以上が占有されています。

* 解決策：

    容量を増やす必要があるかどうかを確認してください。
    ログ、スナップショット、コアダンプなど、多量のディスクスペースを占有しているファイルがないかを確認してください。

#### `PD_system_time_slow`

* アラートルール：

    `changes(pd_tso_events{type="system_time_slow"}[10m]) >= 1`

* 説明：

    システム時刻の巻き戻しが発生する可能性があります。

* 解決策：

    システム時刻が正しく設定されているかを確認してください。

#### `PD_no_store_for_making_replica`

* アラートルール：

    `increase(pd_checker_event_count{type="replica_checker", name="no_target_store"}[1m]) > 0`

* 説明：

    追加のレプリカに適切なストアがありません。

* 解決策：

    ストア内に十分なスペースがあるかを確認してください。
    設定されている場合は、ラベル構成に従って追加のレプリカ用のストアがあるかどうかを確認してください。

#### `PD_cluster_slow_tikv_nums`

* アラートルール：

    `sum(pd_cluster_status{type="store_slow_count"}) by (instance) > 0) and (sum(etcd_server_is_leader) by (instance) > 0`

* 説明：

    遅いTiKVノードがあります。 `raftstore.inspect-interval`はTiKV遅いノードの検出を制御します。詳細については[`raftstore.inspect-interval`](/tikv-configuration-file.md#inspect-interval)を参照してください。

* 解決策：

    [**TiKV-Details** > **PD**ダッシュボード](/grafana-tikv-dashboard.md#pd)を表示し、ストア遅延スコアメトリクスを見てください。値が80を超えるノードを特定し、遅いノードとして検出してください。
    [**TiKV-Details** > **Raft IO**ダッシュボード](/grafana-tikv-dashboard.md#raft-io)を表示し、遅延が増加しているかどうかを確認してください。遅延が高い場合、ディスクにボトルネックがある可能性があります。
    [`raftstore.inspect-interval`](/tikv-configuration-file.md#inspect-interval)構成項目を大きな値に設定してタイムアウト制限を増やしてください。
    アラートされたTiKVノードのパフォーマンス問題とチューニング方法の詳細な分析については、[パフォーマンス分析とチューニング](/performance-tuning-methods.md#storage-async-write-duration-store-duration-and-apply-duration)を参照してください。

## TiKVアラートルール

このセクションでは、TiKVコンポーネントのアラートルールが示されています。

### 緊急レベルのアラート

#### `TiKV_memory_used_too_fast`

* アラートルール：

    `process_resident_memory_bytes{job=~"tikv",instance=~".*"} - (process_resident_memory_bytes{job=~"tikv",instance=~".*"} offset 5m) > 5*1024*1024*1024`

* 説明：

    現在、TiKVのメモリに関する監視項目はありません。クラスター内のマシンのメモリ使用率をNode_exporterで監視できます。上記のルールは、5分以内にメモリ使用量が5 GBを超える場合（TiKV内でメモリが過剰に使用される場合）、アラートがトリガーされます。

* 解決策：

    `rocksdb.defaultcf`と`rocksdb.writecf`の`block-cache-size`の値を調整してください。

#### `TiKV_GC_can_not_work`

* アラートルール：
```
`sum(increase(tikv_gcworker_gc_tasks_vec{task="gc"}[1d])) < 1 and (sum(increase(tikv_gc_compaction_filter_perform[1d])) < 1 and sum(increase(tikv_engine_event_total{db="kv", cf="write", type="compaction"}[1d])) >= 1)`

* 説明：

    24時間以内にTiKVインスタンスでGCが正常に実行されなかった場合、GCが正常に動作していないことを示します。短期間でGCが実行されない場合は、あまり問題になりませんが、GCが停止すると、より多くのバージョンが保持され、クエリの遅延を引き起こす可能性があります。

* 解決方法：

    1. `SELECT VARIABLE_VALUE FROM mysql.tidb WHERE VARIABLE_NAME = "tikv_gc_leader_desc"` を実行して、GCリーダーに対応する `tidb-server` を特定します。
    2. `tidb-server` のログを表示し、tidb.log で gc_worker を grep します。
    3. GC worker がこの時間中にロックを解決していること（最後のログが "start resolve locks"）や範囲を削除していること（最後のログが "start delete {number} ranges"）を見つけた場合、GCプロセスは正常に実行されています。それ以外の場合は、PingCAPやコミュニティから[サポートを取得](/support.md)してください。

### クリティカルレベルのアラート

#### `TiKV_server_report_failure_msg_total`

* アラートルール：

    `sum(rate(tikv_server_report_failure_msg_total{type="unreachable"}[10m])) BY (store_id) > 10`

* 説明：

    リモートのTiKVに接続できないことを示します。

* 解決方法：

    1. ネットワークがクリアかどうかを確認します。
    2. リモートのTiKVがダウンしていないかを確認します。
    3. リモートのTiKVがダウンしていない場合は、プレッシャーが高すぎるかどうかを確認します。[`TiKV_channel_full_total`](#tikv_channel_full_total)の解決方法を参照してください。

#### `TiKV_channel_full_total`

* アラートルール：

    `sum(rate(tikv_channel_full_total[10m])) BY (type, instance) > 0`

* 説明：

    これは、Raftstoreスレッドが詰まっていて、TiKVに高いプレッシャーがかかっていることが原因であることが多い問題です。

* 解決方法：

    1. [**TiKV-Details** > **Raft Propose** ダッシュボード](/grafana-tikv-dashboard.md#raft-propose)を表示し、警告されたTiKVノードが他のTiKVノードよりもはるかに高いRaft提案を持っているかどうかを確認します。そうであれば、このTiKVノードにホットスポットがあることを示し、ホットスポットのスケジューリングが適切に機能しているかどうかを確認する必要があります。
    2. [**TiKV-Details** > **Raft IO** ダッシュボード](/grafana-tikv-dashboard.md#raft-io)を表示し、レイテンシが増加しているかどうかを確認します。レイテンシが高い場合、ディスクにボトルネックがある可能性があります。
    3. [**TiKV-Details** > **Raft process** ダッシュボード](/grafana-tikv-dashboard.md#raft-process)を表示し、`tick duration` が高いかどうかを確認します。高い場合は、[`raftstore.raft-base-tick-interval`](/tikv-configuration-file.md#raft-base-tick-interval)を `"2s"` に設定する必要があります。

#### `TiKV_write_stall`

* アラートルール：

    `delta(tikv_engine_write_stall[10m]) > 0`

* 説明：

    RocksDBへの書き込みプレッシャーが高すぎてスタールが発生しています。

* 解決方法：

    1. ディスクモニターを表示し、ディスクの問題をトラブルシューティングします。
    2. TiKV上で書き込みホットスポットが発生していないかを確認します。
    3. `[rocksdb]` および `[raftdb]` の構成で `max-sub-compactions` を大きな値に設定します。

...

    1. スケジューラーコマンドのデュレーションを表示し、どのコマンドが最も時間がかかるかを確認します。
```
    2. スケジューラー全モニターでスケジューラーのスキャン詳細を表示し、`total`と`process`が一致しているかどうかを確認します。大きく異なる場合、無効なスキャンが多い可能性があります。また、`over seek bound`があるかどうかも確認できます。あまり多い場合、GCがタイムリーに動作していないことを示します。

3. ストレージモニターで非同期スナップショット/書き込みの所要時間を表示し、Raft操作がタイムリーに実行されているかどうかを確認します。

#### `TiKV_thread_apply_worker_cpu_seconds`

* 警告ルール：

    `max(rate(tikv_thread_cpu_seconds_total{name=~"apply_.*"}[1m])) by (instance) > 0.9`

* 説明：

    Raftログ適用スレッドに大きな負荷がかかり、限界に近づいているか、あるいは超過している場合があります。これは通常、ライトの急増によって引き起こされます。

### 警告レベルのアラート

#### `TiKV_leader_drops`

* 警告ルール：

    `delta(tikv_pd_heartbeat_tick_total{type="leader"}[30s]) < -10`

* 説明：

    これは通常、スタックしたRaftstoreスレッドによって引き起こされます。

* 対処方法：

    1. [`TiKV_channel_full_total`](#tikv_channel_full_total)を参照してください。
    2. TiKVに低い圧力がかかっている場合は、PDのスケジューリングがあまりにも頻繁でないかどうかを検討してください。PDページのOperator作成パネルを表示し、実行されるPDスケジューリングのタイプと数を確認できます。

#### `TiKV_raft_process_ready_duration_secs`

* 警告ルール：

    `histogram_quantile(0.999, sum(rate(tikv_raftstore_raft_process_duration_secs_bucket{type='ready'}[1m])) by (le, instance, type)) > 2`

* 説明：

    Raft readyの処理時間を示します。この値が大きい場合、ログの追加タスクのスタックによるものです。

#### `TiKV_raft_process_tick_duration_secs`

* 警告ルール：

    `histogram_quantile(0.999, sum(rate(tikv_raftstore_raft_process_duration_secs_bucket{type='tick'}[1m])) by (le, instance, type)) > 2`

* 説明：

    Raft tickの処理時間を示します。この値が大きい場合、リージョンが多すぎるために引き起こされることがよくあります。

* 対処方法：

    1. `warn`や`error`といったより高いログレベルの使用を検討してください。
    2. `[raftstore]`構成の下に`raft-base-tick-interval = "2s"`を追加してください。

#### `TiKV_scheduler_context_total`

* 警告ルール：

    `abs(delta( tikv_scheduler_context_total[5m])) > 1000`

* 説明：

    スケジューラーによって実行されている書き込みコマンドの数を示します。この値が大きい場合、タスクがタイムリーに完了していないことを意味します。

* 対処方法：

    [`TiKV_scheduler_latch_wait_duration_seconds`](#tikv_scheduler_latch_wait_duration_seconds)を参照してください。

#### `TiKV_scheduler_command_duration_seconds`

* 警告ルール：

    `histogram_quantile(0.99, sum(rate(tikv_scheduler_command_duration_seconds_bucket[1m])) by (le, instance, type)) > 1`

* 説明：

    スケジューラーコマンドの実行にかかる時間を示します。

* 対処方法：

    [`TiKV_scheduler_latch_wait_duration_seconds`](#tikv_scheduler_latch_wait_duration_seconds)を参照してください。
マシンのメモリ使用量が80%を超えています。

* 解決策：

    * Grafana Node ExporterダッシュボードのホストのMemoryパネルを表示し、Used memoryが高すぎるかどうか、Available memoryが低すぎるかどうかを確認してください。
    * マシンにログインして、`free -m`コマンドを実行してメモリ使用量を確認します。`top`を実行して、過剰なメモリ使用量の異常なプロセスがないか確認できます。

### 警告レベルのアラート

#### `NODE_node_overload`

* アラートルール：

    `(node_load5 / count without (cpu, mode) (node_cpu_seconds_total{mode="system"})) > 1`

* 説明：

    マシンのCPU負荷が比較的高いです。

* 解決策：

    * Grafana Node ExporterダッシュボードでホストのCPU使用率と平均負荷を確認し、高すぎるかどうかを確認してください。
    * マシンにログインして、負荷平均とCPU使用率を確認するには`top`を実行し、過剰なCPU使用率の異常なプロセスがないか確認できます。

#### `NODE_cpu_used_more_than_80%`

* アラートルール：

    `avg(irate(node_cpu_seconds_total{mode="idle"}[5m])) by(instance) * 100 <= 20`

* 説明：

    マシンのCPU使用率が80%を超えています。

* 解決策：

    * Grafana Node ExporterダッシュボードでホストのCPU使用率と平均負荷を確認し、高すぎるかどうかを確認してください。
    * マシンにログインして`top`を実行し、負荷平均とCPU使用率を確認し、過剰なCPU使用率の異常なプロセスがないか確認できます。

#### `NODE_tcp_estab_num_more_than_50000`

* アラートルール：

    `node_netstat_Tcp_CurrEstab > 50000`

* 説明：

    マシン上で「確立」状態のTCPリンクが5万以上あります。

* 解決策：

    マシンにログインして`ss -s`を実行し、現在のシステムの「確立」状態のTCPリンクの数を確認してください。
    `netstat`を実行して、異常なリンクがあるかどうかを確認してください。

#### `NODE_disk_read_latency_more_than_32ms`

* アラートルール：

    `((rate(node_disk_read_time_seconds_total{device=~".+"}[5m]) / rate(node_disk_reads_completed_total{device=~".+"}[5m])) or (irate(node_disk_read_time_seconds_total{device=~".+"}[5m]) / irate(node_disk_reads_completed_total{device=~".+"}[5m])) ) * 1000 > 32`

* 説明：

    ディスクの読み取りレイテンシが32msを超えています。

* 解決策：

    Grafana Disk Performanceダッシュボードでディスクの状態を確認してください。
    Disk Latencyパネルでディスクの読み取りレイテンシを確認してください。
    Disk I/O UtilizationパネルでI/O使用率を確認してください。

#### `NODE_disk_write_latency_more_than_16ms`

* アラートルール：

    `((rate(node_disk_write_time_seconds_total{device=~".+"}[5m]) / rate(node_disk_writes_completed_total{device=~".+"}[5m])) or (irate(node_disk_write_time_seconds_total{device=~".+"}[5m]) / irate(node_disk_writes_completed_total{device=~".+"}[5m])))> 16`

* 説明：

    ディスクの書き込みレイテンシが16msを超えています。

* 解決策：

    Grafana Disk Performanceダッシュボードでディスクの状態を確認してください。
    Disk Latencyパネルでディスクの書き込みレイテンシを確認してください。
    Disk I/O UtilizationパネルでI/O使用率を確認してください。

## Blackbox_exporter TCP、ICMP、およびHTTPのアラートルール

このセクションでは、Blackbox_exporter TCP、ICMP、およびHTTPのアラートルールを示します。

### 緊急レベルのアラート

#### `TiDB_server_is_down`

* アラートルール：

    `probe_success{group="tidb"} == 0`

* 説明：

    TiDBサービスポートへのプローブに失敗しました。

* 解決策：

    TiDBサービスを提供するマシンがダウンしていないかどうかを確認してください。
    TiDBプロセスが存在するかどうかを確認してください。
    モニタリングマシンとTiDBマシンのネットワークが正常かどうかを確認してください。

#### `TiFlash_server_is_down`

* アラートルール：

    `probe_success{group="tiflash"} == 0`

* 説明：

    TiFlashサービスポートへのプローブに失敗しました。

* 解決策：

    TiFlashサービスを提供するマシンがダウンしていないかどうかを確認してください。
    TiFlashプロセスが存在するかどうかを確認してください。
    モニタリングマシンとTiFlashマシンのネットワークが正常かどうかを確認してください。

#### `Pump_server_is_down`

* アラートルール：

    `probe_success{group="pump"} == 0`

* 説明：

    Pumpサービスポートへのプローブに失敗しました。

* 解決策：

    Pumpサービスを提供するマシンがダウンしていないかどうかを確認してください。
    Pumpプロセスが存在するかどうかを確認してください。
    モニタリングマシンとPumpマシンのネットワークが正常かどうかを確認してください。

#### `Drainer_server_is_down`

* アラートルール：

    `probe_success{group="drainer"} == 0`

* 説明：

    Drainerサービスポートへのプローブに失敗しました。

* 解決策：

    Drainerサービスを提供するマシンがダウンしていないかどうかを確認してください。
    Drainerプロセスが存在するかどうかを確認してください。
    モニタリングマシンとDrainerマシンのネットワークが正常かどうかを確認してください。

#### `TiKV_server_is_down`

* アラートルール：

    `probe_success{group="tikv"} == 0`

* 説明：

    TiKVサービスポートへのプローブに失敗しました。

* 解決策：

    TiKVサービスを提供するマシンがダウンしていないかどうかを確認してください。
    TiKVプロセスが存在するかどうかを確認してください。
    モニタリングマシンとTiKVマシンのネットワークが正常かどうかを確認してください。

#### `PD_server_is_down`

* アラートルール：

    `probe_success{group="pd"} == 0`

* 説明：

    PDサービスポートへのプローブに失敗しました。

* 解決策：

    PDサービスを提供するマシンがダウンしていないかどうかを確認してください。
    PDプロセスが存在するかどうかを確認してください。
    モニタリングマシンとPDマシンのネットワークが正常かどうかを確認してください。

#### `Node_exporter_server_is_down`

* アラートルール：

    `probe_success{group="node_exporter"} == 0`

* 説明：

    Node_exporterサービスポートへのプローブに失敗しました。

* 解決策：

    Node_exporterサービスを提供するマシンがダウンしていないかどうかを確認してください。
    Node_exporterプロセスが存在するかどうかを確認してください。
    モニタリングマシンとNode_exporterマシンのネットワークが正常かどうかを確認してください。

#### `Blackbox_exporter_server_is_down`

* アラートルール：

    `probe_success{group="blackbox_exporter"} == 0`

* 説明：

    Blackbox_Exporterサービスポートへのプローブに失敗しました。

* 解決策：

    Blackbox_Exporterサービスを提供するマシンがダウンしていないかどうかを確認してください。
    Blackbox_Exporterプロセスが存在するかどうかを確認してください。
    モニタリングマシンとBlackbox_Exporterマシンのネットワークが正常かどうかを確認してください。

#### `Grafana_server_is_down`

* アラートルール：

    `probe_success{group="grafana"} == 0`

* 説明：

    Grafanaサービスポートへのプローブに失敗しました。

* 解決策：

    Grafanaサービスを提供するマシンがダウンしていないかどうかを確認してください。
    Grafanaプロセスが存在するかどうかを確認してください。
    モニタリングマシンとGrafanaマシンのネットワークが正常かどうかを確認してください。

#### `Pushgateway_server_is_down`

* アラートルール：

    `probe_success{group="pushgateway"} == 0`

* 説明：

    Pushgatewayサービスポートへのプローブに失敗しました。

* 解決策：

    Pushgatewayサービスを提供するマシンがダウンしていないかどうかを確認してください。
    Pushgatewayプロセスが存在するかどうかを確認してください。
    モニタリングマシンとPushgatewayマシンのネットワークが正常かどうかを確認してください。

#### `Kafka_exporter_is_down`

* アラートルール：

    `probe_success{group="kafka_exporter"} == 0`

* 説明：

    Kafka_Exporterサービスポートへのプローブに失敗しました。

* 解決策：

    Kafka_Exporterサービスを提供するマシンがダウンしていないかどうかを確認してください。
    Kafka_Exporterプロセスが存在するかどうかを確認してください。
    モニタリングマシンとKafka_Exporterマシンのネットワークが正常かどうかを確認してください。

#### `Pushgateway_metrics_interface`

* アラートルール：

    `probe_success{job="blackbox_exporter_http"} == 0`

* 説明：

    Pushgatewayサービスのhttpインタフェースへのプローブに失敗しました。

* 解決策：

    Pushgatewayサービスを提供するマシンがダウンしていないかどうかを確認してください。
    Pushgatewayプロセスが存在するかどうかを確認してください。
    モニタリングマシンとPushgatewayマシンのネットワークが正常かどうかを確認してください。

### 警告レベルのアラート

#### `BLACKER_ping_latency_more_than_1s`

* アラートルール：

    `max_over_time(probe_duration_seconds{job=~"blackbox_exporter.*_icmp"}[1m]) > 1`

* 説明：

    ピンのレイテンシが1秒を超えています。
```
* Solution:
  
  * Grafana Blackbox Exporter page で 2 つのノード間の ping の遅延を表示し、遅すぎるかどうかをチェックします。
  * Grafana Node Exporter ページの TCP パネルをチェックして、パケットロスがあるかどうかを確認します。
```