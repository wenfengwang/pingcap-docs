---
title: ハイブリッドデプロイメントトポロジ
summary: TiDBクラスタのハイブリッドデプロイメントトポロジを学ぶ。
aliases: ['/docs/dev/hybrid-deployment-topology/']
---

# ハイブリッドデプロイメントトポロジ

このドキュメントでは、TiKVとTiDBのハイブリッドデプロイメントのトポロジと主要なパラメータについて説明します。

ハイブリッドデプロイメントは通常、次のシナリオで使用されます。

デプロイメントマシンに複数のCPUプロセッサと十分なメモリが搭載されています。物理マシンのリソースの利用率を向上させるために、複数のインスタンスを単一のマシンに展開できます。つまり、TiDBとTiKVのCPUリソースはNUMAノードバインディングを介して分離されます。PDとPrometheusは一緒に展開されますが、それらのデータディレクトリは別々のファイルシステムを使用する必要があります。

## トポロジ情報

| インスタンス | 数 | 物理マシンの構成 | IP | 設定 |
| :-- | :-- | :-- | :-- | :-- |
| TiDB | 6 | 32 VCore 64GB | 10.0.1.1<br/> 10.0.1.2<br/> 10.0.1.3 | CPUコアをバインドするためのNUMAの設定 |
| PD | 3 | 16 VCore 32 GB | 10.0.1.4<br/> 10.0.1.5<br/> 10.0.1.6 | `location_labels`パラメータを設定 |
| TiKV | 6 | 32 VCore 64GB | 10.0.1.7<br/> 10.0.1.8<br/> 10.0.1.9 | 1. インスタンスレベルのポートとstatus_portを別にする; <br/> 2. グローバルパラメータ`readpool`、`storage`および`raftstore`を構成する; <br/> 3. インスタンスレベルのホストのラベルを構成する; <br/> 4. CPUコアをバインドするためのNUMAの設定 |
| モニタリング＆Grafana | 1 | 4 VCore 8GB * 1 500GB (ssd)  | 10.0.1.10 | デフォルトの設定 |

### トポロジテンプレート

- [ハイブリッドデプロイメントのシンプルテンプレート](https://github.com/pingcap/docs/blob/master/config-templates/simple-multi-instance.yaml)
- [ハイブリッドデプロイメントの複雑なテンプレート](https://github.com/pingcap/docs/blob/master/config-templates/complex-multi-instance.yaml)

上述したTiDBクラスタのトポロジファイルの構成アイテムの詳細な説明については、[TiUPを使用してTiDBを展開するためのトポロジ構成ファイル](/tiup/tiup-cluster-topology-reference.md)を参照してください。

### 主要なパラメータ

このセクションでは、単一のマシンに複数のインスタンスを展開する際に必要となる主要なパラメータについて紹介します。これは主に、TiDBとTiKVの複数のインスタンスが単一のマシンに展開されるシナリオで使用されます。以下の計算方法に従って構成テンプレートに結果を記入する必要があります。

- TiKVの設定を最適化する

    - `readpool`をスレッドプールに自動適応させるように構成します。`readpool.unified.max-thread-count`パラメータを構成することで、`readpool.storage`と`readpool.coprocessor`を共通のスレッドプールで共有し、それぞれの自動適応スイッチを設定できます。

        - `readpool.storage`および`readpool.coprocessor`を有効にします:

            ```yaml
            readpool.storage.use-unified-pool: true
            readpool.coprocessor.use-unified-pool: true
            ```

        - 計算方法:

            ```
            readpool.unified.max-thread-count = cores * 0.8 / TiKVインスタンスの数
            ```

    - ストレージCF（すべてのRocksDBカラムファミリ）をメモリに自動適応させるように構成します。`storage.block-cache.capacity`パラメータを構成することで、CFはメモリ使用量を自動的にバランスさせます。

        - 計算方法:

            ```
            storage.block-cache.capacity = (MEM_TOTAL * 0.5 / TiKVインスタンスの数)
            ```

    - 単一の物理ディスクに複数のTiKVインスタンスが展開されている場合、TiKVの構成に`capacity`パラメータを追加します:

        ```
        raftstore.capacity = ディスクの総容量 / TiKVインスタンスの数
        ```

- ラベルスケジューリングの構成

    単一のマシンに複数のTiKVインスタンスが展開されているため、物理マシンがダウンすると、Raftグループはデフォルトの3レプリカのうち2つを失う可能性があり、クラスタの利用可能性が低下します。この問題を解決するために、ラベルを使用してPDのスマートスケジューリングを有効にし、同一マシン上の複数のTiKVインスタンスに3つ以上のレプリカを持つようにします。

    - TiKVの構成

        同一物理マシンのために同じホストレベルのラベル情報を構成します:

        ```yml
        config:
          server.labels:
            host: tikv1
        ```

    - PDの構成

        PDにリージョンを識別してスケジューリングするために、PDのラベルタイプを構成します:

        ```yml
        pd:
          replication.location-labels: ["host"]
        ```

- `numa_node`コアバインディング

    - インスタンスパラメータモジュールで、対応する`numa_node`パラメータを構成し、CPUコアの数を追加します。
    
    - コアをNUMAにバインドする前に、numactlツールがインストールされていることを確認し、物理マシンのCPU情報を確認します。その後、パラメータを構成します。

    - `numa_node`パラメータは`numactl --membind`の構成に対応します。

> **注意:**
>
> - 構成ファイルテンプレートを編集する際には、必要なパラメーター、IP、ポート、およびディレクトリを変更します。
> - 各コンポーネントはデフォルトでグローバル`<deploy_dir>/<components_name>-<port>`を、デフォルトの`deploy_dir`として使用します。たとえば、TiDBが`4001`ポートを指定した場合、その`deploy_dir`はデフォルトで`/tidb-deploy/tidb-4001`になります。したがって、マルチインスタンスのシナリオでは、デフォルトのポートを指定する必要がないため、ディレクトリを再度指定する必要はありません。
> - 構成ファイルで`deploy_dir`を相対パスとして指定する場合、クラスタはユーザーのホームディレクトリに展開されます。