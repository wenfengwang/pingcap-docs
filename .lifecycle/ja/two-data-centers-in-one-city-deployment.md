---
title: リージョン内の2つのアベイラビリティゾーンの展開
summary: 1つのリージョンでの2つのアベイラビリティゾーンの展開ソリューションについて学ぶ
aliases: ['/tidb/dev/synchronous-replication']
---

# リージョン内の2つのアベイラビリティゾーンの展開

このドキュメントでは、1つのリージョンでの2つのアベイラビリティゾーン（AZ）の展開モードについて、アーキテクチャ、構成、この展開モードの有効化方法、およびこのモードでのレプリカの使用方法を紹介します。

このドキュメントにおける "リージョン" とは、地理的な地域を指し、大文字で表される "Region" は TiKV におけるデータストレージの基本単位を指します。 "AZ" とは、リージョン内の孤立した場所を指し、各リージョンには複数の AZ があります。このドキュメントで記載されているソリューションは、1つの都市内に複数のデータセンターがある場合にも適用されます。

## はじめに

通常、TiDB は高い可用性と災害復旧能力を確保するために、マルチ-AZ 展開ソリューションを採用しています。マルチ-AZ 展開ソリューションには、1つのリージョン内の複数の AZ および2つのリージョン内の複数の AZ など、さまざまな展開モードが含まれています。このドキュメントでは、1つのリージョン内の2つの AZ の展開モードを紹介します。このモードで展開すると、より低コストで高可用性と災害復旧要件を満たすことができます。この展開ソリューションでは、Data Replication Auto Synchronous モード、または DR Auto-Sync モードが採用されています。

1つのリージョン内の2つの AZ を利用する展開モードでは、2つの AZ は50キロメートル未満離れています。通常、同じリージョン内にあるか、隣接する2つのリージョンに位置しています。2つの AZ 間のネットワーク遅延は1.5ミリ秒未満であり、帯域幅は10 Gbps以上です。

## 展開アーキテクチャ

このセクションでは、東および西それぞれに位置する2つのアベイラビリティゾーン AZ1 と AZ2 があるリージョンを例に取り、クラスター展開のアーキテクチャを以下のように説明します。

- クラスターは6つのレプリカを持っており、AZ1 には3つの Voter レプリカがあり、AZ2 には2つの Voter レプリカと1つの Learner レプリカがあります。また、TiKV コンポーネントでは、各ラックに適切なラベルが付与されています。
- データの一貫性と高可用性を確保するために Raft プロトコルが採用されており、ユーザーには透過的です。

![2-AZ-in-1-region アーキテクチャ](/media/two-dc-replication-1.png)

この展開ソリューションでは、クラスターのレプリケーション状態を制御および識別するために3つのステータスを定義し、TiKV のレプリケーションモードを制限します。クラスターのレプリケーションモードは、自動的にかつ適応的に、3つのステータス間で切り替えることができます。詳細については [ステータス切り替え](#status-switch) セクションを参照してください。

- **sync**: 同期レプリケーションモード。このモードでは、災害復旧 AZ において少なくとも1つのレプリカが主要な AZ と同期します。Raft アルゴリズムは、各ログがラベルに基づいて DR にレプリケートされることを保証します。
- **async**: 非同期レプリケーションモード。このモードでは、災害復旧 AZ が主要な AZ と完全に同期されていません。Raft アルゴリズムは多数決プロトコルに従ってログをレプリケートします。
- **sync-recover**: 同期回復モード。このモードでは、災害復旧 AZ が主要な AZ と完全に同期されていません。Raft は徐々にラベルレプリケーションモードに切り替わり、その後ラベル情報を PD に報告します。

## 構成

### 例

以下は、1つのリージョンでの2つのアベイラビリティゾーン展開モードの典型的なトポロジ構成の `tiup topology.yaml` の例です:

```yaml
# グローバル変数はすべての展開に適用され、展開のデフォルト値として使用されます。
global:
  user: "tidb"
  ssh_port: 22
  deploy_dir: "/data/tidb_cluster/tidb-deploy"
  data_dir: "/data/tidb_cluster/tidb-data"
server_configs:
  pd:
    replication.location-labels: ["az","rack","host"]
pd_servers:
  - host: 10.63.10.10
    name: "pd-10"
  - host: 10.63.10.11
    name: "pd-11"
  - host: 10.63.10.12
    name: "pd-12"
tidb_servers:
  - host: 10.63.10.10
  - host: 10.63.10.11
  - host: 10.63.10.12
tikv_servers:
  - host: 10.63.10.30
    config:
      server.labels: { az: "east", rack: "east-1", host: "30" }
  - host: 10.63.10.31
    config:
      server.labels: { az: "east", rack: "east-2", host: "31" }
  - host: 10.63.10.32
    config:
      server.labels: { az: "east", rack: "east-3", host: "32" }
  - host: 10.63.10.33
    config:
      server.labels: { az: "west", rack: "west-1", host: "33" }
  - host: 10.63.10.34
    config:
      server.labels: { az: "west", rack: "west-2", host: "34" }
  - host: 10.63.10.35
    config:
      server.labels: { az: "west", rack: "west-3", host: "35" }
monitoring_servers:
  - host: 10.63.10.60
grafana_servers:
  - host: 10.63.10.60
alertmanager_servers:
  - host: 10.63.10.60
```

### 配置ルール

プランしたトポロジに基づいてクラスターを展開するために、[配置ルール](/configure-placement-rules.md) を使用してクラスターのレプリカの配置を決定する必要があります。主要な AZ に2つの Voter レプリカがあり、災害復旧 AZ に1つの Voter レプリカと1つの Learner レプリカがある展開の例を取ると、以下のように Placement Rules を設定できます:

```
cat rule.json
[
  {
    "group_id": "pd",
    "group_index": 0,
    "group_override": false,
    "rules": [
      {
        "group_id": "pd",
        "id": "az-east",
        "start_key": "",
        "end_key": "",
        "role": "voter",
        "count": 3,
        "label_constraints": [
          {
            "key": "az",
            "op": "in",
            "values": [
              "east"
            ]
          }
        ],
        "location_labels": [
          "az",
          "rack",
          "host"
        ]
      },
      {
        "group_id": "pd",
        "id": "az-west-1",
        "start_key": "",
        "end_key": "",
        "role": "follower",
        "count": 2,
        "label_constraints": [
          {
            "key": "az",
            "op": "in",
            "values": [
              "west"
            ]
          }
        ],
        "location_labels": [
          "az",
          "rack",
          "host"
        ]
      },
      {
        "group_id": "pd",
        "id": "az-west-2",
        "start_key": "",
        "end_key": "",
        "role": "learner",
        "count": 1,
        "label_constraints": [
          {
            "key": "az",
            "op": "in",
            "values": [
              "west"
            ]
          }
        ],
        "location_labels": [
          "az",
          "rack",
          "host"
        ]
      }
    ]
  }
]
```

`rule.json` の構成を使用するには、既存の構成を `default.json` ファイルにバックアップし、既存の構成を `rule.json` で上書きします:

{{< copyable "shell-regular" >}}

```bash
pd-ctl config placement-rules rule-bundle load --out="default.json"
pd-ctl config placement-rules rule-bundle save --in="rule.json"
```

前の構成に戻す必要がある場合は、バックアップファイル `default.json` を復元するか、以下の JSON ファイルを手動で作成し、既存の構成をこの JSON ファイルで上書きしてください:

```
cat default.json
[
  {
    "group_id": "pd",
    "group_index": 0,
    "group_override": false,
    "rules": [
      {
        "group_id": "pd",
        "id": "default",
        "start_key": "",
        "end_key": "",
        "role": "voter",
        "count": 5
      }
    ]
  }
]
```

### DR Auto-Sync モードの有効化

レプリケーションモードは PD によって制御されます。PD の構成ファイルでレプリケーションモードを設定するには、以下の方法のいずれかを使用できます:

- 方法 1: PD の構成ファイルを構成し、その後クラスターを展開します。

    {{< copyable "" >}}

    ```toml
    [replication-mode]
    replication-mode = "dr-auto-sync"
    [replication-mode.dr-auto-sync]
    label-key = "az"
    primary = "east"
    dr = "west"
    primary-replicas = 3
    dr-replicas = 2
    wait-store-timeout = "1m"
    ```

- 方法 2: クラスタを展開済みの場合は、PD-ctlコマンドを使用してPDの構成を変更します。

    {{< copyable "" >}}

    ```shell
    config set replication-mode dr-auto-sync
    config set replication-mode dr-auto-sync label-key az
    config set replication-mode dr-auto-sync primary east
    config set replication-mode dr-auto-sync dr west
    config set replication-mode dr-auto-sync primary-replicas 3
    config set replication-mode dr-auto-sync dr-replicas 2
    ```

構成項目の説明:

+ `replication-mode` は有効にするレプリケーションモードです。前述の例では、`dr-auto-sync` に設定されています。デフォルトでは、過半数プロトコルが使用されます。
+ `label-key` は異なるAZを区別するために使用され、配置ルールに一致する必要があります。この例では、プライマリAZは "east" であり、災害復旧AZは "west" です。
+ `primary-replicas` はプライマリAZの有権者レプリカの数です。
+ `dr-replicas` は災害復旧AZの有権者レプリカの数です。
+ `wait-store-timeout` はネットワーク隔離や障害が発生した場合に非同期レプリケーションモードに切り替えるまでの待機時間です。ネットワーク障害の時間が待機時間を超えると、非同期レプリケーションモードが有効になります。デフォルトの待機時間は60秒です。

クラスタの現在のレプリケーション状態を確認するには、次のAPIを使用します:

{{< copyable "shell-regular" >}}

```bash
curl http://pd_ip:pd_port/pd/api/v1/replication_mode/status
```

{{< copyable "shell-regular" >}}

```bash
{
  "mode": "dr-auto-sync",
  "dr-auto-sync": {
    "label-key": "az",
    "state": "sync"
  }
}
```

#### ステータス切り替え

クラスタのレプリケーションモードは、自動的に適応的に3つのステータス間を切り替えることができます:

- クラスタが正常な場合、同期レプリケーションモードが有効になり、災害復旧AZのデータ整合性を最大化します。
- 2つのAZ間のネットワーク接続が失敗した場合や災害復旧AZがダウンした場合、事前に設定された保護間隔後、クラスタは非同期レプリケーションモードに切り替えてアプリケーションの可用性を確保します。
- ネットワークが再接続された場合や災害復旧AZが復旧した場合、TiKVノードはクラスタに再参加し、データを徐々にレプリケートします。最終的に、クラスタは同期レプリケーションモードに切り替わります。

ステータス切り替えの詳細は次の通りです:

1. **初期化**: 初期化段階では、クラスタは同期レプリケーションモードです。PDはTiKVにステータス情報を送信し、すべてのTiKVノードは厳密に同期レプリケーションモードに従って動作します。

2. **同期から非同期への切り替え**: PDは定期的にTiKVのハートビート情報をチェックし、TiKVノードが故障したかどうかを判断します。故障したノードの数がプライマリAZ (`primary-replicas`) と災害復旧AZ (`dr-replicas`) のレプリカ数を超える場合、同期レプリケーションモードではデータレプリケーションを適切に行えなくなり、ステータスを切り替える必要があります。故障または切断時間が `wait-store-timeout` で設定された時間を超えると、PDはクラスタのステータスを非同期モードに切り替えます。その後、PDはすべてのTiKVノードに非同期のステータスを送信し、TiKVのレプリケーションモードは2つのAZ間のレプリケーションから標準のRaft過半数へ切り替わります。

3. **非同期から同期への切り替え**: PDは定期的にTiKVのハートビート情報をチェックし、TiKVノードが再接続されたかどうかを判断します。故障したノードの数がプライマリAZ (`primary-replicas`) と災害復旧AZ (`dr-replicas`) のレプリカ数を下回る場合、同期レプリケーションモードを再度有効にできる可能性があります。PDはまずクラスタのステータスを sync-recover に切り替え、ステータス情報をすべてのTiKVノードに送信します。すべてのTiKVのリージョンは徐々に2つのAZ間の同期レプリケーションモードに切り替わり、その後PDにハートビート情報を報告します。PDはTiKVリージョンのステータスを記録し、復旧の進捗を計算します。すべてのTiKVリージョンが切り替えを完了すると、PDはレプリケーションモードを同期に切り替えます。

### 災害復旧

このセクションでは、1つのリージョンに2つのAZの災害復旧ソリューションを紹介します。

同期レプリケーションモードのクラスタに災害が発生した場合、 `RPO = 0` でデータ復旧を行うことができます:

- プライマリAZが失敗し、ほとんどのVoterレプリカが失われた場合、しかし完全なデータが災害復旧AZに存在する場合、失われたデータを災害復旧AZから復旧することができます。この時、専門のツールでの手動介入が必要です。PingCAPやコミュニティから[サポートを受ける](/support.md)ことができます。
- 災害復旧AZが失敗し、いくつかのVoterレプリカが失われた場合、クラスタは自動的に非同期レプリケーションモードに切り替わります。

同期レプリケーションモードでないクラスタに災害が発生し、 `RPO = 0` でデータ復旧を行うことができない場合:

- ほとんどのVoterレプリカが失われた場合、専門のツールでの手動介入が必要です。PingCAPやコミュニティから[サポートを受ける](/support.md)ことができます。