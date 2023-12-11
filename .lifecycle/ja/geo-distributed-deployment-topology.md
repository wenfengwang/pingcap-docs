---
title: ジオ分散展開のトポロジ
summary: TiDBのジオ分散展開のトポロジを学ぶ
aliases: ['/docs/dev/geo-distributed-deployment-topology/']
---

# ジオ分散展開のトポロジ

このドキュメントは、2つの都市に3つのデータセンター（DC）の典型的なアーキテクチャを例に取り、ジオ分散展開アーキテクチャと主要な構成を紹介します。この例で使用される都市は、上海（`sha`として参照）と北京（`bja`および`bjb`として参照）です。

## トポロジ情報

| インスタンス | 数 | 物理マシン構成 | BJ IP | SH IP | 構成 |
| :-- | :-- | :-- | :-- | :-- | :-- |
| TiDB | 5 | 16 VCore 32GB * 1 | 10.0.1.1 <br/> 10.0.1.2 <br/> 10.0.1.3 <br/> 10.0.1.4 | 10.0.1.5 | デフォルトポート <br/> グローバルディレクトリ構成 |
| PD | 5 | 4 VCore 8GB * 1 | 10.0.1.6 <br/> 10.0.1.7 <br/> 10.0.1.8 <br/> 10.0.1.9 | 10.0.1.10 | デフォルトポート <br/> グローバルディレクトリ構成 |
| TiKV | 5 | 16 VCore 32GB 4TB（nvme ssd）* 1 | 10.0.1.11 <br/> 10.0.1.12 <br/> 10.0.1.13 <br/> 10.0.1.14 | 10.0.1.15 | デフォルトポート <br/> グローバルディレクトリ構成 |
| Monitoring＆Grafana | 1 | 4 VCore 8GB * 1 500GB（ssd） | 10.0.1.16 | | デフォルトポート <br/> グローバルディレクトリ構成 |

### トポロジテンプレート

- [ジオ分散トポロジテンプレート](https://github.com/pingcap/docs/blob/master/config-templates/geo-redundancy-deployment.yaml)

上記のTiDBクラスタトポロジファイルの構成項目の詳細については、[TiUPを使用してTiDBを展開するためのトポロジ構成ファイル](/tiup/tiup-cluster-topology-reference.md)を参照してください。

### 主要なパラメータ

このセクションでは、TiDBのジオ分散展開の主要なパラメータ構成について説明します。

#### TiKVパラメータ

- gRPC圧縮形式（デフォルト：`none`）：

    ジオ分散ターゲットノード間のgRPCパッケージの転送速度を向上させるために、このパラメータを`gzip`に設定します。

    ```yaml
    server.grpc-compression-type: gzip
    ```

- ラベル構成：

    TiKVは異なるデータセンターに展開されているため、物理マシンがダウンした場合、Raft Groupはデフォルトの5つのレプリカのうち3つを失う可能性があり、クラスタの使用不可を引き起こします。この問題を解決するために、ラベルを構成して、PDのスマートスケジューリングを有効にし、Raft Groupが同じデータセンターの同じキャビネットのTiKVインスタンスに3つのレプリカを配置しないようにします。

- TiKV構成：

    同じ物理マシンのホストレベルのラベル情報を構成します。

    ```yaml
    config:
      server.labels:
        zone: bj
        dc: bja
        rack: rack1
        host: host2
    ```

- リモートTiKVノードが不要なRaft選挙を起動しないようにするために、リモートTiKVノードが選挙を起動するための最小および最大のticks数を増やす必要があります。これらの2つのパラメータはデフォルトで`0`に設定されています。

    ```yaml
    raftstore.raft-min-election-timeout-ticks: 1000
    raftstore.raft-max-election-timeout-ticks: 1020
    ```

#### PDパラメータ

- PDメタデータ情報はTiKVクラスタのトポロジを記録します。PDはRaft Groupレプリカを次の4つの次元でスケジュールします：

    ```yaml
    replication.location-labels: ["zone","dc","rack","host"]
    ```

- クラスタの高可用性を確保するために、Raft Groupレプリカの数を`5`に調整します：

    ```yaml
    replication.max-replicas: 5
    ```

- リモートTiKV Raftレプリカがリーダーとして選出されるのを禁止します：

    ```yaml
    label-property:
          reject-leader:
            - key: "dc"
              value: "sha"
    ```

   > **注意:**
   >
   > TiDB 5.2以降、`label-property`構成はデフォルトでサポートされていません。レプリカポリシーを設定するには、[配置ルール](/configure-placement-rules.md)を使用してください。

これらのラベルおよびRaft Groupレプリカの数に関する詳細情報については、[トポロジラベルでレプリカをスケジュール](/schedule-replicas-by-topology-labels.md)を参照してください。

> **注意:**
>
> - 構成ファイルで`tidb`ユーザを手動で作成する必要はありません。 TiUPクラスタコンポーネントが対象マシン上で`tidb`ユーザを自動的に作成します。ユーザをカスタマイズするか、ユーザを制御マシンと一貫させることができます。
> - 展開ディレクトリを相対パスとして構成する場合、クラスタはユーザのホームディレクトリに展開されます。