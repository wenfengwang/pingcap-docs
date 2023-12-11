---
title: 3ノードハイブリッド展開のベストプラクティス
summary: 3ノードハイブリッド展開のベストプラクティスを学びます。
---

# 3ノードハイブリッド展開のベストプラクティス

TiDBクラスターにおいて、高いパフォーマンス要件はないがコスト管理が必要な場合、TiDB、TiKV、およびPDのコンポーネントを3台のマシンにハイブリッド展開することができます。

このドキュメントでは、3ノードハイブリッド展開の例と、その展開シナリオおよびパラメータの調整に関するベストプラクティスを提供します。

## 展開とテスト方法の前提条件

この例では、3つの物理マシンがデプロイに使用され、それぞれが16 CPUコアと32 GBのメモリを搭載しています。各マシン（ノード）には、TiDBのインスタンス1つ、TiKVのインスタンス1つ、およびPDのインスタンス1つがハイブリッド展開されています。

PDとTiKVは両方とも情報をディスクに格納するため、ディスクの読み書き遅延はPDとTiKVのサービスの遅延に直接影響します。PDとTiKVがディスクリソースを競合させてお互いに影響を与えないようにするために、PDとTiKVには異なるディスクを使用することを推奨します。

この例では、TiUP benchでTPC-C 5000 Warehouseデータを使用し、`terminals`パラメータを`128`（同時実行数）に設定して12時間のテストを行います。クラスターのパフォーマンス安定性に関連するメトリクスに注意を払います。

以下のイメージは、デフォルトのパラメータ設定で12時間以内にクラスターのQPSモニターを示しています。このイメージから、明らかなパフォーマンスのブレが見て取れます。

![デフォルトの構成でのQPS](/media/best-practices/three-nodes-default-config-qps.png)

パラメータの調整後、パフォーマンスが向上しました。

![変更された構成でのQPS](/media/best-practices/three-nodes-final-config-qps.png)

## パラメータの調整

上記のイメージにパフォーマンスブレが発生するのは、デフォルトのスレッドプール構成とバックグラウンドタスクへのリソース割り当てが、十分なリソースを持つマシン向けに設定されているためです。しかしながら、ハイブリッド展開のシナリオでは、リソースが複数のコンポーネント間で共有されるため、設定パラメータを使用してリソース消費を制限する必要があります。

このテストでの最終クラスター構成は以下の通りです。

{{< 編集可能 "" >}}

```yaml
tikv:
    readpool.unified.max-thread-count: 6
    server.grpc-concurrency: 2
    storage.scheduler-worker-pool-size: 2
    gc.max-write-bytes-per-sec: 300K
    rocksdb.max-background-jobs: 3
    rocksdb.max-sub-compactions: 1
    rocksdb.rate-bytes-per-sec: "200M"

  tidb:
    performance.max-procs: 8
```

以下のセクションでは、これらのパラメータの意味と調整方法について紹介します。

### TiKVスレッドプールサイズの設定

このセクションでは、フォアグラウンドアプリケーションのスレッドプールへのリソース割り当てに関連するパラメータの調整方法を提供します。これらのスレッドプールのサイズを縮小するとパフォーマンスが低下しますが、リソースが限られたハイブリッド展開のシナリオでは、パフォーマンスよりもクラスター全体の安定性を優先する必要があります。

実際の負荷テストを実施する場合、まずデフォルトの構成を使用して各スレッドプールの実際のリソース使用状況を観察します。その後、対応する構成項目を調整し、使用率が低いスレッドプールのサイズを縮小します。

#### `readpool.unified.max-thread-count`

このパラメータのデフォルト値はマシンスレッド数の80%です。ハイブリッド展開のシナリオでは、手動でこの値を計算して指定する必要があります。まずTiKVが利用する予想されるCPUスレッド数の80%に設定できます。

#### `server.grpc-concurrency`

このパラメータのデフォルト値は`4`です。既存の展開計画ではCPUリソースが制限されており、実際のリクエストが少ないため、監視パネルを観察し、このパラメータの値を低く設定し、使用率を80%以下に保ちます。

このテストでは、このパラメータの値を`2`に設定します。**gRPCポールCPU**パネルを観察すると、使用率がちょうど80%程度であることがわかります。

![gRPCポールCPU](/media/best-practices/three-nodes-grpc-pool-usage.png)

#### `storage.scheduler-worker-pool-size`

TiKVがマシンのCPUコア数が`16`以上であることを検出すると、このパラメータ値は`8`にデフォルトで設定されます。CPUコア数が`16`未満の場合、このパラメータ値は`4`にデフォルトで設定されます。このパラメータは、TiKVが複雑なトランザクションリクエストを単純なキー・バリューの読み取りまたは書き込みに変換するときに使用されますが、スケジューラスレッドプールは何も書込みを行いません。

理想的には、スケジューラスレッドプールの使用率を50%から75%の間に保ちます。gRPCスレッドプールと同様に、ハイブリッド展開時にはこの`storage.scheduler-worker-pool-size`パラメータがデフォルト値よりも大きくなり、リソース使用率が不足するため、このテストではこのパラメータの値を`2`に設定します。これは、**スケジューラワーカーCPU**パネルでの対応するメトリクスを観察して導かれたベストプラクティスに一致します。

![スケジューラワーカーCPU](/media/best-practices/three-nodes-scheduler-pool-usage.png)

### TiKVバックグラウンドタスクのリソース設定

フォアグラウンドタスクと同様に、TiKVは定期的にバックグラウンドタスクでデータを整理し、古いデータを削除します。デフォルト設定では、高トラフィックライトのシナリオに十分なリソースが割り当てられています。

しかし、ハイブリッド展開のシナリオでは、このデフォルト設定はベストプラクティスと一致しません。リソース使用を制限するために、以下のパラメータを調整する必要があります。

#### `rocksdb.max-background-jobs` および `rocksdb.max-sub-compactions`

RocksDBスレッドプールは、コンパクションおよびフラッシュジョブを実行するために使用されます。`rocksdb.max-background-jobs`のデフォルト値は`8`であり、必要なリソースを明らかに超過しています。したがって、この値を調整してリソース使用を制限する必要があります。

`rocksdb.max-sub-compactions`は、単一のコンパクションジョブに対して許可される同時サブタスクの数を示しますが、デフォルト値は`3`です。書き込みトラフィックが少ない場合、この値を下げることができます。

このテストでは、`rocksdb.max-background-jobs`の値を`3`、`rocksdb.max-sub-compactions`の値を`1`に設定します。TPC-C負荷の12時間テスト中に書き込みスタールが発生しません。これら2つのパラメータ値を実際の負荷に応じて最適化する際は、監視メトリクスに基づいて値を徐々に下げることができます:

* 書き込みスタールが発生する場合は、`rocksdb.max-background-jobs`の値を増やします。
* 書き込みスタールが続く場合は、`rocksdb.max-sub-compactions`の値を`2`または`3`に設定します。

#### `rocksdb.rate-bytes-per-sec`

このパラメータは、バックグラウンドコンパクションジョブのディスクトラフィックを制限するために使用されます。デフォルト構成では、このパラメータには制限がありません。バックグラウンドサービスのリソースを占有する状況を回避するためには、このパラメータ値をディスクのシーケンシャル読み込みおよび書き込み速度に応じて調整し、前景サービスに十分なディスク帯域を確保します。

RocksDBスレッドプールの最適化方法は、コンパクションスレッドプールの最適化方法と類似しています。調整した値が適切かどうかは、書き込みスタールが発生するかどうかによって判断できます。

#### `gc.max_write_bytes_per_sec`

TiDBはマルチバージョン並行制御（MVCC）モデルを使用するため、TiKVは定期的にバックグラウンドで古いバージョンのデータをクリーンアップします。利用可能なリソースが限られている場合、この操作によって定期的なパフォーマンスブレが発生します。`gc.max_write_bytes_per_sec`パラメータを使用して、この操作のリソース使用を制限することができます。

![GCの影響](/media/best-practices/three-nodes-gc-impact.png)

このパラメータ値を設定ファイルで設定する他に、tikv-ctlでこの値を動的に調整することもできます。

{{< 編集可能 "shell-regular" >}}

```shell
tiup ctl:v<CLUSTER_VERSION> tikv --host=${ip:port} modify-tikv-config -n gc.max_write_bytes_per_sec -v ${limit}
```

> **注意:**
>
> 頻繁な更新が発生するアプリケーションシナリオでは、GCトラフィックを制限するとMVCCバージョンが蓄積され、読み取り性能に影響を与える可能性があります。現在はパフォーマンスブレとパフォーマンスの低下のバランスを取るために、このパラメータ値を複数回調整する必要があります。

### TiDBパラメータの調整

通常、`tidb_hash_join_concurrency`や`tidb_index_lookup_join_concurrency`などのシステム変数を使用して、TiDBの実行演算子のパラメータを調整できます。

このテストでは、これらのパラメータは調整されていません。実際のアプリケーションの負荷テストでは、実行演算子が過度のCPUリソースを消費する場合、アプリケーションシナリオに応じて特定の演算子のリソース使用を制限することができます。詳細については、[TiDBシステム変数](/system-variables.md)を参照してください。

#### `performance.max-procs`

このパラメータは、Goプロセス全体が使用できるCPUコア数を制御するために使用されます。デフォルトでは、値は現在のマシンまたはcgroupsのCPUコア数に等しくなります。

Goが実行中の場合、一部のスレッドはGCなどのバックグラウンドタスクに使用されます。`performance.max-procs`パラメータの値を制限しないと、これらのバックグラウンドタスクが過度にCPUを消費する可能性があります。