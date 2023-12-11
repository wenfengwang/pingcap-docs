---
title: パブリッククラウド上のTiDBのベストプラクティス
summary: パブリッククラウド上でTiDBを展開するためのベストプラクティスについて学びます。
---

# パブリッククラウド上のTiDBのベストプラクティス

パブリッククラウドインフラストラクチャは、TiDBを展開および管理するための人気のある選択肢となっています。ただし、パブリッククラウド上でTiDBを展開するには、パフォーマンスチューニング、コスト最適化、信頼性、およびスケーラビリティなど、いくつかの重要な要素を慎重に考慮する必要があります。

このドキュメントでは、パブリッククラウド上でTiDBを展開するためのさまざまな必須のベストプラクティスについて説明します。これには、Raft Engine用の専用ディスクの使用、KV RocksDBでのコンパクションI/Oフローの削減、クロスAZトラフィックのコスト最適化、Google Cloudのライブマイグレーションイベントの緩和、および大規模クラスタでのPDサーバーの最適化などが含まれます。これらのベストプラクティスに従うことで、パブリッククラウド上のTiDB展開のパフォーマンス、コスト効率、信頼性、およびスケーラビリティを最大限に引き出すことができます。

## Raft Engine用の専用ディスクの使用

TiKVにおける[Raft Engine](/glossary.md#raft-engine)は、従来のデータベースのWrite-Ahead Log（WAL）と同様に重要な役割を果たします。最適なパフォーマンスと安定性を実珅するためには、TiDBをパブリッククラウド上に展開する際に、Raft Engineに専用のディスクを割り当てることが重要です。次の`iostat`は、書き込みが多いワークロードを持つTiKVノードでのI/O特性を示しています。

```
Device            r/s     rkB/s       w/s     wkB/s      f/s  aqu-sz  %util
sdb           1649.00 209030.67   1293.33 304644.00    13.33    5.09  48.37
sdd           1033.00   4132.00   1141.33  31685.33   571.00    0.94 100.00
```

デバイス`sdb`はKV RocksDBに使用され、一方で`sdd`はRaft Engineログのリストアに使用されます。`sdd`は、デバイスの完了したフラッシュリクエストの数を表す`f/s`値が著しく高いことに注意してください。Raft Engineでは、バッチ内の書き込みが同期的にマークされたとき、バッチのリーダーは書き込んだ後に`fdatasync()`を呼び出し、バッファされたデータがストレージにフラッシュされることを保証します。Raft Engine用の専用ディスクを使用することで、TiKVはリクエストの平均キュー長を減らし、最適な安定した書き込みレイテンシを確実にします。

さまざまなクラウドプロバイダは、IOPSやMBPSなどの異なる性能特性を持つさまざまなディスクタイプを提供しています。そのため、ワークロードに基づいて適切なクラウドプロバイダ、ディスクタイプ、およびディスクサイズを選択することが重要です。

### パブリッククラウド上のRaft Engine用の適切なディスクの選択

このセクションでは、異なるパブリッククラウドのRaft Engine用の適切なディスクの選択に関するベストプラクティスについて説明します。パフォーマンス要件に応じて、2種類の推奨されるディスクタイプが利用可能です。

#### 中間範囲のディスク

以下は、異なるパブリッククラウドにおける中間範囲のディスクの推奨事項です。

- AWSでは、[gp3](https://aws.amazon.com/ebs/general-purpose/)が推奨されます。gp3ボリュームは、ボリュームサイズに関係なく、通常はRaft Engineに十分な3000 IOPSと125 MB/sのスループットを無料で提供します。

- Google Cloudでは、[pd-ssd](https://cloud.google.com/compute/docs/disks#disk-types/)が推奨されます。IOPSおよびMBPSは、割り当てられたディスクサイズによって異なります。パフォーマンス要件を満たすためには、Raft Engineに200 GBを割り当てることを推奨します。Raft Engine自体はこのような大きなスペースを必要としませんが、これにより最適なパフォーマンスが確保されます。

- Azureでは、[Premium SSD v2](https://learn.microsoft.com/en-us/azure/virtual-machines/disks-types#premium-ssd-v2)が推奨されます。AWSのgp3と同様に、Premium SSD v2は、ボリュームサイズに関係なく、通常はRaft Engineに十分な3000 IOPSと125 MB/sのスループットを無料で提供します。

#### 上位範囲のディスク

Raft Engineのさらに低レイテンシが必要な場合は、上位範囲のディスクを検討してください。以下は、異なるパブリッククラウドにおける上位範囲のディスクの推奨事項です。

- AWSでは、[io2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html)が推奨されます。ディスクサイズとIOPSは、特定の要件に応じてプロビジョニングできます。

- Google Cloudでは、[pd-extreme](https://cloud.google.com/compute/docs/disks#disk-types/)が推奨されます。ディスクサイズ、IOPS、およびMBPSはプロビジョニングできますが、64 CPUコア以上のインスタンスでのみ利用可能です。

- Azureでは、[ultra disk](https://learn.microsoft.com/en-us/azure/virtual-machines/disks-types#ultra-disks)が推奨されます。ディスクサイズ、IOPS、およびMBPSは特定の要件に応じてプロビジョニングできます。

### 例1：AWSでソーシャルネットワークワークロードを実行する

AWSは、20 GBの[gp3](https://aws.amazon.com/ebs/general-purpose/)ボリュームに3000 IOPSと125 MBPS/sを提供します。

AWSの20 GBの[d3](https://aws.amazon.com/ebs/general-purpose/) Raft Engineディスクを、書き込みが多いソーシャルネットワークアプリケーションワークロードに使用することで、次の改善が見られますが、見積もりコストはわずかに0.4%増加します:

- QPS（1秒あたりのクエリ数）が17.5%増加
- 挿入ステートメントの平均レイテンシが18.7%減少
- 挿入ステートメントのp99レイテンシが45.6%減少

| メトリクス | 共有Raft Engineディスク | 専用Raft Engineディスク | 差分（%） |
| ------------- | ------------- |------------- |------------- |
| QPS (K/s)| 8.0 | 9.4 | 17.5|
| 平均挿入レイテンシ (ms)| 11.3 | 9.2 | -18.7 |
| P99挿入レイテンシ (ms)| 29.4 | 16.0 | -45.6|

### 例2：AzureでTPC-C/Sysbenchワークロードを実行する

Azureでは、Raft Engine用の32GBの[ultra disk](https://learn.microsoft.com/en-us/azure/virtual-machines/disks-types#ultra-disks)を専用で使用することで、以下の改善が見られます:

- Sysbench `oltp_read_write`ワークロード: QPSが17.8%増加し、平均レイテンシが15.6%減少
- TPC-Cワークロード: QPSが27.6%増加し、平均レイテンシが23.1%減少

| メトリクス | ワークロード | 共有Raft Engineディスク | 専用Raft Engineディスク | 差分（%） |
| ------------- | ------------- | ------------- |------------- |------------- |
| QPS (K/s) | Sysbench `oltp_read_write` | 60.7 | 71.5 | 17.8|
| QPS (K/s) | TPC-C | 23.9 | 30.5 | 27.6|
| 平均レイテンシ (ms)| Sysbench `oltp_read_write` |  4.5 | 3.8 | -15.6 |
| 平均レイテンシ (ms)| TPC-C |  3.9 | 3.0 | -23.1 |

### 例3：TiKVマニフェストでGoogle Cloudに専用のpd-ssdディスクをアタッチする

次のTiKVの構成例は、[TiDB Operator](https://docs.pingcap.com/tidb-in-kubernetes/stable)によってデプロイされたGoogle Cloudのクラスタに、Raft Engineログをこの特定のディスクに保存するように構成された追加の512GBの[pd-ssd](https://cloud.google.com/compute/docs/disks#disk-types/)ディスクをアタッチする方法を示しています。

```
tikv:
    config: |
      [raft-engine]
        dir = "/var/lib/raft-pv-ssd/raft-engine"
        enable = true
        enable-log-recycle = true
    requests:
      storage: 4Ti
    storageClassName: pd-ssd
    storageVolumes:
    - mountPath: /var/lib/raft-pv-ssd
      name: raft-pv-ssd
      storageSize: 512Gi
```

## KV RocksDBのコンパクションI/Oフローを削減する

TiKVのストレージエンジンである[RocksDB](https://rocksdb.org/)は、ユーザーデータの格納に使用されます。クラウドのEBS上での割り当てられたIOスループットは通常、コストの観点から制限されているため、RocksDBでは高い書き込み増幅が発生し、ディスクスループットがワークロードのボトルネックとなる可能性があります。その結果、未処理のコンパクションバイトの総数が時間とともに増加し、フローコントロールをトリガーし、TiKVにはフォアグラウンドの書き込みフローに追いつくための十分なディスク帯域幅がないことを示します。

制限されたディスクスループットによるボトルネックを緩和するために、RocksDBの圧縮レベルを上げ、ディスクスループットを減らすことでパフォーマンスを向上させることができます。たとえば、以下の例を参考に、デフォルトのカラムファミリのすべての圧縮レベルを`zstd`に上げることができます。

```
[rocksdb.defaultcf]
compression-per-level = ["zstd", "zstd", "zstd", "zstd", "zstd", "zstd", "zstd"]
```

## クロスAZネットワークトラフィックのコストを最適化する

複数の可用性ゾーン（AZ）にわたってTiDBを展開すると、クロスAZデータ転送料金が増加する可能性があります。コストを最適化するためには、クロスAZネットワークトラフィックを削減することが重要です。

クロスAZのリードトラフィックを減らすためには、[Follower Read feature](/follower-read.md)を有効にすることができます。これにより、TiDBは同じ可用性ゾーン内のレプリカの選択を優先するようになります。この機能を有効にするには、[`tidb_replica_read`](/system-variables.md#tidb_replica_read-new-in-v40)変数を`closest-replicas`または`closest-adaptive`に設定します。

TiKVインスタンスでのクロスAZの書き込みトラフィックを減らすためには、データをネットワーク上で送信する前に圧縮するgRPC圧縮機能を有効にすることができます。次の構成例では、TiKVのgzip gRPC圧縮を有効にする方法が示されています。

```
server_configs:
  tikv:
    server.grpc-compression-type: gzip
```

TiFlash MPPタスクのデータシャッフルによるネットワークトラフィックを減らすためには、同じ可用性ゾーン（AZ）に複数のTiFlashインスタンスを展開することが推奨されています。v6.6.0からは、[compression exchange](/explain-mpp.md#mpp-version-and-exchange-data-compression)がデフォルトで有効になっており、MPPデータシャッフルによるネットワークトラフィックを減らします。

## Google Cloud上のライブマイグレーションメンテナンスイベントを軽減する

Google Cloudの[Live Migration feature](https://cloud.google.com/compute/docs/instances/live-migration-process)により、VMをダウンタイムを引き起こさずにホスト間でシームレスに移行させることができます。しかし、これらの移行イベントはまれではありますが、TiDBクラスタで実行されているVMを含むパフォーマンスに大きな影響を与える可能性があります。こうしたイベント中、影響を受けるVMではパフォーマンスが低下し、TiDBクラスタ内でクエリ処理時間が長くなる可能性があります。

Google Cloudによって開始されたライブマイグレーションイベントを検出し、これらのイベントによるパフォーマンスへの影響を軽減するために、TiDBはGoogleのメタデータ[example](https://github.com/GoogleCloudPlatform/python-docs-samples/blob/master/compute/metadata/main.py)に基づいた[watching script](https://github.com/PingCAP-QE/tidb-google-maintenance)を提供しています。このスクリプトをTiDB、TiKV、PDノードに展開してメンテナンスイベントを検出することができます。メンテナンスイベントが検出されると、次のように自動的に適切なアクションを実行して、中断を最小限に抑えクラスタの動作を最適化することができます。

- TiDB: TiDBノードをクラスタから除外し、TiDBポッドを削除します。これには、TiDBインスタンスのノードプールが自動スケールおよびTiDB専用に設定されていることが前提となります。ノードで実行されている他のポッドに中断が発生する可能性があり、封鎖されたノードは自動スケーラーによってリクレームされることが予想されます。
- TiKV: メンテナンス中に影響を受けるTiKVストアのリーダーを追い出します。
- PD: 現在のPDインスタンスがPDリーダーである場合、リーダーを解任します。

このwatching scriptは、Kubernetes環境でTiDBを管理する[TiDB Operator](https://docs.pingcap.com/tidb-in-kubernetes/dev/tidb-operator-overview)を使用して展開されたTiDBクラスタ専用に設計されていることに留意することが重要です。

メンテナンスイベント中に必要なアクションを実行してwatching scriptを活用することで、TiDBクラスタはGoogle Cloud上のライブマイグレーションイベントをよりスムーズに処理し、クエリ処理と応答時間への影響を最小限に抑えることができます。

## 高QPSを持つ大規模なTiDBクラスタに対するPDの調整

TiDBクラスタでは、重要なタスクを処理するために単一のアクティブなPlacement Driver（PD）サーバーが使用されます。ただし、単一のアクティブなPDサーバーに依存することは、TiDBクラスタのスケーラビリティを制限する可能性があります。

### PD制限の症状

次の図は、56個のCPUを備えた3つのPDサーバーから成る大規模なTiDBクラスタの症状を示しています。これらの図からは、クエリごとの秒数（QPS）が100万を超え、TSO（Timestamp Oracle）のリクエスト数が162,000を超えると、CPU利用率が約4,600%に達することが分かります。この高いCPU利用率は、PDリーダーが重い負荷を受けており、利用可能なCPUリソースが不足していることを示しています。

![pd-server-cpu](/media/performance/public-cloud-best-practice/baseline_cpu.png)
![pd-server-metrics](/media/performance/public-cloud-best-practice/baseline_metrics.png)

### PDパフォーマンスの調整

PDサーバーの高いCPU利用率の問題に対処するには、次の調整を行うことができます。

#### PD構成の調整

[`tso-update-physical-interval`](/pd-configuration-file.md#tso-update-physical-interval): このパラメータは、PDサーバーが物理的なTSOバッチを更新する間隔を制御します。この間隔を短縮することで、PDサーバーはTSOバッチをより頻繁に割り当てることができ、次回の割り当てまでの待ち時間を短縮することができます。

```
tso-update-physical-interval = "10ms" # デフォルト: 50ms
```

#### TiDBグローバル変数の調整

PDの構成に加えて、TSOクライアントバッチ待機機能を有効にすることで、TSOクライアントの動作をさらに最適化することができます。この機能を有効にするには、グローバル変数[`tidb_tso_client_batch_max_wait_time`](/system-variables.md#tidb_tso_client_batch_max_wait_time-new-in-v530)を非ゼロの値に設定できます。

```
set global tidb_tso_client_batch_max_wait_time = 2; # デフォルト: 0
```

#### TiKV構成の調整

リージョン数を減らし、システム上のハートビートオーバーヘッドを軽減するためには、TiKV構成でリージョンサイズを`96MB`から`256MB`に増やすことを推奨します。

```
[coprocessor]
  region-split-size = "256MB"
```

## 調整後

調整後、以下の効果が見られます。

- 秒ごとのTSOリクエストが64,800に減少します。
- CPU利用率は、約4,600%から約1,400%に大幅に低下します。
- `PDサーバーのTSO処理時間`のP999値は2msから0.5msに低下します。

これらの改善点から、調整の変更がPDサーバーのCPU利用率を成功裏に低減させながら安定したTSO処理パフォーマンスを維持したことがわかります。

![pd-server-cpu](/media/performance/public-cloud-best-practice/after_tuning_cpu.png)
![pd-server-metrics](/media/performance/public-cloud-best-practice/after_tuning_metrics.png)