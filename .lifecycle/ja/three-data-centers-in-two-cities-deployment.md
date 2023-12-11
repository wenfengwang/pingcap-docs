---
title: 2つのリージョンに3つの可用ゾーンを展開
summary: 2つのリージョンに3つの可用ゾーンを展開するためのデプロイメントソリューションについて学ぶ
aliases: ['/docs/dev/three-data-centers-in-two-cities-deployment/']
---

# 2つのリージョンに3つの可用ゾーンを展開

この文書では、2つのリージョンに3つの可用ゾーン（AZ）を展開するアーキテクチャと構成を紹介します。

この文書における「リージョン」という用語は地理的なエリアを指し、「Region」という大文字の単語はTiKVにおけるデータストレージの基本単位を指します。「AZ」はリージョン内の隔離された場所を指し、各リージョンには複数のAZがあります。この文書で説明するソリューションは、複数のデータセンターが同じ都市にあるシナリオにも適用されます。

## 概要

2つのリージョンに3つのAZを持つアーキテクチャは、高い可用性と災害耐性を提供する展開ソリューションであり、本番データAZ、同じリージョンの災害復旧AZ、別のリージョンの災害復旧AZを提供します。このモードでは、2つのリージョンにある3つのAZが相互接続されています。1つのAZが障害を起こしたり災害に見舞われたりしても、他のAZは引き続き適切に動作し、主要アプリケーションまたはすべてのアプリケーションを引き継ぐことができます。1つのリージョン内に複数のAZがある展開と比較して、このソリューションはリージョン間の高可用性を持ち、リージョンレベルの自然災害に耐えることができます。

分散データベースTiDBは、Raftアルゴリズムを使用して3つのAZと2つのリージョンのアーキテクチャをネイティブにサポートし、データの一貫性と高可用性をデータベースクラスタ内で保証します。同じリージョン内のAZ間のネットワーク遅延が比較的低いため、アプリケーショントラフィックは同じリージョン内の2つのAZにディスパッチでき、TiKVリージョンリーダーとPDリーダーの分配を制御することで、これらの2つのAZでトラフィック負荷を共有できます。

## 展開アーキテクチャ

このセクションでは、SeattleとSan Franciscoの例を取り上げ、TiDBの分散データベースの3つのAZと2つのリージョンへの展開モードを説明します。

この例では、2つのAZ（AZ1とAZ2）がSeattleにあり、もう1つのAZ（AZ3）がSan Franciscoにあります。AZ1とAZ2の間のネットワーク遅延は3ミリ秒以下です。SeattleのAZ1およびAZ2との間のAZ3のネットワーク遅延は約20ミリ秒です（ISP専用ネットワークを使用）。

クラスタの展開アーキテクチャは次のようになります。

- TiDBクラスタはSeattleのAZ1、SeattleのAZ2、およびSan FranciscoのAZ3に展開されます。
- クラスタには5つのレプリカがあり、そのうち2つがAZ1に、2つがAZ2に、1つがAZ3に展開されています。TiKVコンポーネントでは、各ラックにラベルが付いており、各ラックにはレプリカがあります。
- Raftプロトコルが採用され、データの一貫性と高可用性がユーザーに透過的に提供されます。

![2リージョンに3つのAZを展開するアーキテクチャ](/media/three-data-centers-in-two-cities-deployment-01.png)

このアーキテクチャは高い可用性を持ちます。Regionリーダーの分配は同じリージョン（Seattle）内にある2つのAZ（AZ1とAZ2）に制限されています。Regionリーダーの分配が制限されていない3つのAZのソリューションと比較して、このアーキテクチャには次の利点と欠点があります。

- **利点**

    - Regionリーダーは遅延の少ない同じリージョン内のAZにあり、書き込みが速くなります。
    - 2つのAZは同時にサービスを提供できるため、リソース使用率が高くなります。
    - 1つのAZが障害を起こしてもサービスは利用可能であり、データの安全性が確保されます。

- **欠点**

    - データの一貫性がRaftアルゴリズムによって達成されるため、同じリージョン内の2つのAZが同時に障害を起こした場合、別のリージョンの災害復旧AZ（San Francisco）には生存しているレプリカが1つだけとなり、ほとんどのレプリカが生存するというRaftアルゴリズムの要件を満たすことができません。その結果、クラスタは一時的に利用できなくなります。メンテナンススタッフは、1つの生存しているレプリカとレプリケートされていない一部のホットデータを復旧する必要があります。ただし、このケースはまれな発生です。
    - ISP専用ネットワークを使用しているため、このアーキテクチャのネットワークインフラストラクチャには高いコストがかかります。
    - 2つのリージョンに3つのAZを構成するために5つのレプリカが設定されており、データ冗長性が増加し、より高いストレージコストが発生します。

### 展開の詳細

2つのリージョンに3つのAZ（SeattleとSan Francisco）を展開する構成計画は次のようになります：

![2リージョンに3つのAZを展開する構成](/media/three-data-centers-in-two-cities-deployment-02.png)

上記の図から、SeattleにはAZ1とAZ2の2つのAZがあります。AZ1にはrac1、rac2、rac3の3つのラックがあります。AZ2にはrac4とrac5の2つのラックがあります。San FranciscoのAZ3にはrac6のラックがあります。

AZ1のrac1には、1つのサーバーがTiDBおよびPDサービスを展開し、他の2つのサーバーはTiKVサービスを展開しています。各TiKVサーバーには2つのTiKVインスタンス（tikv-server）が展開されています。これはrac2、rac4、rac5、rac6についても同様です。

TiDBサーバー、制御マシン、および監視サーバーはrac3にあります。TiDBサーバーは定期的なメンテナンスおよびバックアップ用に展開されています。Prometheus、Grafana、およびリストアツールは制御マシンと監視マシンに展開されています。

別のバックアップサーバーを追加してDrainerを展開し、バイナリログデータを指定された場所に出力することで、増分バックアップを実珸することができます。

## 構成

### 例

次の `tiup topology.yaml` のyamlファイルを参照してください：

```yaml
# # Global variables are applied to all deployments and used as the default value of
# # the deployments if a specific deployment value is missing.
global:
  user: "tidb"
  ssh_port: 22
  deploy_dir: "/data/tidb_cluster/tidb-deploy"
  data_dir: "/data/tidb_cluster/tidb-data"

server_configs:
  tikv:
    server.grpc-compression-type: gzip
  pd:
    replication.location-labels: ["az","replication zone","rack","host"]

pd_servers:
  - host: 10.63.10.10
    name: "pd-10"
  - host: 10.63.10.11
    name: "pd-11"
  - host: 10.63.10.12
    name: "pd-12"
  - host: 10.63.10.13
    name: "pd-13"
  - host: 10.63.10.14
    name: "pd-14"

tidb_servers:
  - host: 10.63.10.10
  - host: 10.63.10.11
  - host: 10.63.10.12
  - host: 10.63.10.13
  - host: 10.63.10.14

tikv_servers:
  - host: 10.63.10.30
    config:
      server.labels: { az: "1", replication zone: "1", rack: "1", host: "30" }
  - host: 10.63.10.31
    config:
      server.labels: { az: "1", replication zone: "2", rack: "2", host: "31" }
  - host: 10.63.10.32
    config:
      server.labels: { az: "2", replication zone: "3", rack: "3", host: "32" }
  - host: 10.63.10.33
    config:
      server.labels: { az: "2", replication zone: "4", rack: "4", host: "33" }
  - host: 10.63.10.34
    config:
      server.labels: { az: "3", replication zone: "5", rack: "5", host: "34" }
      raftstore.raft-min-election-timeout-ticks: 1000
      raftstore.raft-max-election-timeout-ticks: 1200

monitoring_servers:
  - host: 10.63.10.60

grafana_servers:
  - host: 10.63.10.60

alertmanager_servers:
  - host: 10.63.10.60
```

### ラベルデザイン

2つのリージョンに3つのAZを展開する際のラベルデザインでは、可用性と災害復旧を考慮する必要があります。デプロイメントの物理的構造に基づいて、ラック、レプリケーションゾーン、ラック、およびホストを基に4つのレベル（`az`, `replication zone`, `rack`, `host`）を定義することが推奨されます。
![ラベルの論理的な定義](/media/three-data-centers-in-two-cities-deployment-03.png)

PDの設定では、TiKVのラベルのレベルの情報を追加します：

```yaml
server_configs:
  pd:
    replication.location-labels: ["az","replication zone","rack","host"]
```

`tikv_servers`の構成は、TiKVの実際の物理的展開位置のラベル情報に基づいており、これによりPDがグローバルな管理とスケジューリングを行う際に便利になります。

```yaml
tikv_servers:
  - host: 10.63.10.30
    config:
      server.labels: { az: "1", replication zone: "1", rack: "1", host: "30" }
  - host: 10.63.10.31
    config:
      server.labels: { az: "1", replication zone: "2", rack: "2", host: "31" }
  - host: 10.63.10.32
    config:
      server.labels: { az: "2", replication zone: "3", rack: "3", host: "32" }
  - host: 10.63.10.33
    config:
      server.labels: { az: "2", replication zone: "4", rack: "4", host: "33" }
  - host: 10.63.10.34
    config:
      server.labels: { az: "3", replication zone: "5", rack: "5", host: "34" }
```

### パラメータ構成の最適化

2つの地域に3つのAZを展開する際、パフォーマンスを最適化するためには、通常のパラメータだけでなくコンポーネントのパラメータも調整する必要があります。

- TiKVでgRPCメッセージ圧縮を有効にする。クラスタのデータがネットワーク上で転送されるため、gRPCメッセージ圧縮を有効にしてネットワークトラフィックを低減させることができます。

    ```yaml
    server.grpc-compression-type: gzip
    ```
- San Franciscoの別の地域のTiKVノードのネットワーク構成を最適化する。San FranciscoのAZ3向けに以下のTiKVパラメータを変更し、このTiKVノードのレプリカがRaft選挙に参加しないようにします。

    ```yaml
    raftstore.raft-min-election-timeout-ticks: 1000
    raftstore.raft-max-election-timeout-ticks: 1200
    ```

- スケジューリングを設定する。クラスタを有効にした後、`tiup ctl:v<CLUSTER_VERSION> pd`ツールを使用してスケジューリングポリシーを変更します。TiKV Raftのレプリカ数を変更します。この例では、レプリカの数を5に構成します。

    ```bash
    config set max-replicas 5
    ```

- RaftリーダーをAZ3にスケジューリングしないようにする。Raftリーダーを別の地域(AZ3)にスケジューリングすると、SeattleのAZ1/AZ2とSan FranciscoのAZ3間で不要なネットワークオーバーヘッドが発生します。ネットワーク帯域幅とレイテンシもTiDBクラスタのパフォーマンスに影響します。

    ```bash
    config set label-property reject-leader dc 3
    ```
   > **注記:**
   > 
   > TiDB v5.2以降、`label-property`構成はデフォルトでサポートされていません。レプリカポリシーを設定するには、[配置ルール](/configure-placement-rules.md)を使用してください。

- PDの優先度を構成する。PDリーダーが別の地域(AZ3)にある状況を避けるために、地元のPD(Seattle)の優先度を上げ、別の地域(San Francisco)のPDの優先度を下げることができます。数値が大きいほど優先度が高くなります。

    ```bash
    member leader_priority PD-10 5
    member leader_priority PD-11 5
    member leader_priority PD-12 5
    member leader_priority PD-13 5
    member leader_priority PD-14 1
    ```