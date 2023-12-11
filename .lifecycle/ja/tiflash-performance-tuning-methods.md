---
title: TiFlashのパフォーマンス分析とチューニング方法
summary: パフォーマンス概要ダッシュボード上のTiFlashメトリクスを紹介し、TiFlashのワークロードをよりよく理解・監視できます。
---

# TiFlashのパフォーマンス分析とチューニング方法

このドキュメントでは、TiFlashリソースの利用状況と主要なパフォーマンスメトリクスについて紹介します。[パフォーマンス概要](/grafana-performance-overview-dashboard.md#tiflash) ダッシュボードの[TiFlashパネル](/grafana-performance-overview-dashboard.md#tiflash)を通じて、TiFlashクラスタのパフォーマンスを監視し評価できます。

## TiFlashクラスタのリソース利用状況

以下の3つのメトリクスを使用して、TiFlashクラスタのリソース利用状況を素早く把握できます：

- CPU: TiFlashインスタンスごとのCPU利用率。
- メモリ: TiFlashインスタンスごとのメモリ使用量。
- IO利用率: TiFlashインスタンスごとのIO利用率。

例: [CH-benCHmarkワークロード](/benchmark/benchmark-tidb-using-ch.md) 中のリソース利用状況

このTiFlashクラスタは2つのノードで構成されており、それぞれのノードには16コアと48GBのメモリが搭載されています。CH-benCHmarkワークロード中、CPU利用率は最大で1500%、メモリ使用量は最大で20GB、IO利用率は最大で91%に達することがあります。これらのメトリクスは、TiFlashノードのリソースが飽和状態に近づいていることを示しています。

![CH-TiFlash-MPP](/media/performance/tiflash/tiflash-resource-usage.png)  

## TiFlashパフォーマンスの主要メトリクス

### スループットメトリクス

以下のメトリクスを使用して、TiFlashのスループットを取得できます：

- MPPクエリ数: 各TiFlashインスタンスのMPPクエリ数の瞬時値であり、TiFlashインスタンスごとに処理する必要のあるMPPクエリの現在の数を反映します（処理中とスケジュール待ちを含む）。
- リクエストQPS: すべてのTiFlashインスタンスで受信したコプロセッサリクエストの数。
    - `run_mpp_task`、`dispatch_mpp_task`、`mpp_establish_conn` がMPPリクエスト。
    - `batch`: バッチリクエスト数。
    - `cop`: コプロセッサを介して直接送信されるコプロセッサリクエスト数。
    - `cop_execution`: 現在実行中のコプロセッサリクエスト数。
    - `remote_read`、`remote_read_constructed`、`remote_read_sent` はリモート読み込みに関連するメトリクスです。リモート読み込みの増加は通常、システムに問題があることを示します。
- Executor QPS: すべてのTiFlashインスタンスで受信した各種類のdag演算子の数。`table_scan`はテーブルスキャン演算子、`selection`は選択演算子、`aggregation`は集約演算子、`top_n`はTopN演算子、`limit`はリミット演算子、`join`は結合演算子、「exchange_sender」はデータ送信演算子、「exchange_receiver」はデータ受信演算子です。

### レイテンシメトリクス

以下のメトリクスを使用して、TiFlashのレイテンシを取得できます：

- リクエスト処理概要: 各TiFlashインスタンスにおいて、全リクエストタイプの総処理時間のスタックチャートを提供します。

    - リクエストのタイプが `run_mpp_task`、`dispatch_mpp_task`、または `mpp_establish_conn`の場合、これはSQLステートメントの実行がTiFlashに部分的にまたは完全にプッシュダウンされたことを示し、通常は結合およびデータ配布操作が関わります。これはTiFlashで最も一般的なリクエストタイプです。
    - リクエストのタイプが `cop` の場合、このリクエストに関連するステートメントがTiFlashに完全にプッシュダウンされていないことを示します。通常、TiDBはTiFlashに対してデータアクセスとフィルタリングのためのテーブルフルスキャン演算子をプッシュダウンします。スタックチャートで `cop` が最も一般的なリクエストタイプになった場合、それが妥当かどうかを確認する必要があります。

        - SQLステートメントがクエリされるデータの量が多い場合、オプティマイザはコストモデルに従ってTiFlashのフルテーブルスキャンがよりコスト効果が高いと推定する場合があります。
        - クエリされるテーブルのスキーマに適切なインデックスがない場合、クエリをTiFlashに対してフルテーブルスキャンすることしかできません。この場合、適切なインデックスを作成し、TiKVを介してデータにアクセスすることが効率的です。

- リクエスト処理時間: 各TiFlashインスタンスでの、すべてのMPPおよびコプロセッサリクエストタイプの総処理時間。これには平均レイテンシとp99レイテンシが含まれます。
- リクエスト処理時間のハンドル: `cop`および`batch cop`のリクエストの実行開始から完了までの時間を、待機時間を除いたもの。このメトリクスは `cop` および `batch cop` のタイプのリクエストにのみ適用され、平均およびP99レイテンシが含まれます。

例1: TiFlash MPPリクエストの処理時間概要

以下のダイアグラムのワークロードでは、`run_mpp_task`と`mpp_establish_conn`リクエストが総処理時間の大半を占めており、ほとんどのリクエストが完全にTiFlashにプッシュダウンされて実行されるMPPタスクであることを示しています。

`cop`リクエストの処理時間は比較的短いため、一部のリクエストがコプロセッサを介してTiFlashにプッシュダウンされ、データアクセスとフィルタリングが行われていることを示しています。

![CH-TiFlash-MPP](/media/performance/tiflash/ch-2tiflash-op.png)

例2: TiFlash `cop`リクエストが総処理時間の大半を占める

以下のダイアグラムのワークロードでは、`cop`リクエストが総処理時間の大半を占めています。この場合、これらの`cop`リクエストが生成される理由を確認するために、SQL実行プランをチェックできます。

![Cop](/media/performance/tiflash/tiflash_request_duration_by_type.png)

### Raft関連メトリクス

以下のメトリクスを使用して、TiFlashのRaftレプリケーション状況を取得できます：

- Raftウェイトインデックスの処理時間: すべてのTiFlashインスタンスにおける、ローカルリージョンのインデックスが `read_index` 以上になるまで待機する時間。これは `wait_index` 操作のレイテンシを表します。このメトリクスが高すぎる場合、TiKVからTiFlashへのデータレプリケーションに大きな遅延が発生していることを示します。可能な理由としては以下が考えられます：

    - TiKVリソースが過負荷になっている。
    - TiFlashリソースが過負荷になっており、特にIOリソースが過負荷になっている。
    - TiKVとTiFlashの間にネットワークボトルネックがある。

- Raftバッチリードインデックスの処理時間: すべてのTiFlashインスタンスにおける、`read_index` のレイテンシ。このメトリクスが高すぎる場合、TiFlashとTiKVの相互作用が遅いことを示します。可能な理由としては以下が考えられます：

    - TiFlashリソースが過負荷になっている。
    - TiKVリソースが過負荷になっている。
    - TiFlashとTiKVの間にネットワークボトルネックがある。

### IOスループットメトリクス

以下のメトリクスを使用して、TiFlashのIOスループットを取得できます：

- インスタンスごとの書き込みスループット: 各TiFlashインスタンスによって書き込まれたデータのスループット。RaftデータログとRaftスナップショットを適用するスループットが含まれます。
- 書き込みフロー: すべてのTiFlashインスタンスによるディスク書き込みのトラフィック。

    - ファイルディスクリプタ: TiFlashが使用するDeltaTreeストレージエンジンの安定層。
    - ページ: TiFlashが使用するDeltaTreeストレージエンジンのDeltaプレーンの層。

- 読み込みフロー: すべてのTiFlashインスタンスによるディスク読み込み操作のトラフィック。

    - ファイルディスクリプタ: TiFlashが使用するDeltaTreeストレージエンジンの安定層。
    - ページ: TiFlashが使用するDeltaTreeストレージエンジンのDeltaプレーンの層。

全体のTiFlashクラスタの書き込み増幅要因は、`(読み込みフロー + 書き込みフロー) ÷ インスタンスごとの総書き込みスループット`の数式を使用して計算できます。

例1: セルフホスティング環境における[CH-benCHmarkワークロード](/benchmark/benchmark-tidb-using-ch.md) のRaftおよびIOメトリクス

以下のダイアグラムに示されるように、このTiFlashクラスタの `Raftウェイトインデックスの処理時間` および `Raftバッチリードインデックスのp99パーセンタイル` は比較的高く、それぞれ3.24秒と753ミリ秒です。これは、このクラスタのTiFlashワークロードが高く、データレプリケーションに遅延が発生していることを示しています。

このクラスタには2つのTiFlashノードがあります。TiKVからTiFlashへの増分データレプリケーション速度はおおよそ28MB/sです。安定層（ファイルディスクリプタ）の最大書き込みスループットは939MB/sであり、最大読み込みスループットは1.1 GiB/sです。一方、Delta層（ページ）の最大書き込みスループットは74MB/sであり、最大読み込みスループットは111MB/sです。この環境では、TiFlashは強力なIOスループット能力を持つ専用のNVMEディスクを使用しています。

![CH-2TiFlash-OP](/media/performance/tiflash/ch-2tiflash-raft-io-flow.png)

例2: 公共クラウド展開環境における[CH-benCHmarkワークロード](/benchmark/benchmark-tidb-using-ch.md) のRaftおよびIOメトリクス
```
      + As shown in the following diagram, the 99th percentile of `Raft Wait Index Duration` is up to 438 milliseconds, and 99th percentile of the `Raft Batch Read Index Duration` is up to 125 milliseconds. This cluster has only one TiFlash node. TiKV replicates about 5 MB of incremental data to TiFlash per second. The maximum write traffic of the stable layer (File Descriptor) is 78 MB/s and the maximum read traffic is 221 MB/s. In the meantime, the maximum write traffic of the Delta layer (Page) is 8 MB/s and the maximum read traffic is 18 MB/s. In this environment, TiFlash uses an AWS EBS cloud disk, which has relatively weak IO throughput.

![CH-TiFlash-MPP](/media/performance/tiflash/ch-1tiflash-raft-io-flow-cloud.png)
```