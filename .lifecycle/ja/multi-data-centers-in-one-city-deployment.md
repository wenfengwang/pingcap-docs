---
title: リージョン内の複数の可用性ゾーンデプロイメント
summary: リージョン内の複数の可用性ゾーンへのデプロイメントソリューションについて学びます。
aliases: ['/docs/dev/how-to/deploy/geographic-redundancy/overview/','/docs/dev/geo-redundancy-deployment/','/tidb/dev/geo-redundancy-deployment']
---

# リージョン内の複数の可用性ゾーンデプロイメント

<!-- TiDB のローカライズに関する注意事項:

- 英語: distributed SQL を使用し、HTAP を強調してください
- 中国語: "NewSQL" のままで、ワンストップのリアルタイム HTAP ("一栈式实时 HTAP") を強調します
- 日本語: よく知られた "NewSQL" を使用してください

-->

TiDBは分散型SQLデータベースであり、従来の関係型データベースの優れた機能とNoSQLデータベースのスケーラビリティを組み合わせ、可用性が高く、可用性ゾーン(AZs)間で利用することができます。このドキュメントでは、リージョン内の複数のAZへのデプロイメントについて紹介します。

このドキュメントにおける「リージョン」とは地理的なエリアを指し、「リージョン」という大文字の「Region」とはTiKVでの基本的なデータストレージ単位を指します。「AZ」とはリージョン内の孤立した場所を指し、各リージョンには複数のAZがあります。このドキュメントで説明されているソリューションは、複数のデータセンターが同一の都市に存在するシナリオにも適用することができます。

## Raftプロトコル

Raftは分散一致アルゴリズムです。このアルゴリズムを使用することで、PDおよびTiKVの各コンポーネントがTiDBクラスタのデータの障害復旧を実現しています。これは以下のメカニズムを通じて実現されています。

- Raftメンバーの主な役割はログレプリケーションと状態マシンの実行です。Raftメンバー間ではログを複製することでデータの複製が実現されます。Raftメンバーは異なる条件で状態を変更し、リーダーを選出してサービスを提供します。
- Raftは過半数のプロトコルに従った投票システムです。Raftグループ内のメンバーが投票の過半数を獲得すると、そのメンバーのメンバーシップはリーダーに変わります。つまり、Raftグループ内にノードの過半数が残っている場合、リーダーを選出してサービスを提供することができます。

Raftの信頼性を活用するために、実際のデプロイメントシナリオでは以下の条件を満たす必要があります。

- サーバは少なくとも３つ使用し、一つのサーバが故障した場合でもサービスを提供できるようにします。
- ラックは少なくとも３つ使用し、一つのラックが故障した場合でもサービスを提供できるようにします。
- AZは少なくとも３つ使用し、一つのAZが故障した場合でもサービスを提供できるようにします。
- データの安全に問題が発生した場合、少なくとも３つのリージョンにTiDBをデプロイします。

元々のRaftプロトコルは偶数のレプリカに対しては十分なサポートを持っていません。クロスリージョンのネットワーク遅延の影響を考慮すると、同じリージョン内の３つのAZが高い可用性と耐障害性を持つRaftデプロイメントに最適なソリューションかもしれません。

## １リージョン内の３つのAZへのデプロイメント

TiDBクラスタは同じリージョン内の３つのAZにデプロイすることができます。このソリューションでは、３つのAZ間でのデータレプリケーションはクラスタ内でRaftプロトコルを使用して実現されます。これらの３つのAZは同時に読み書きサービスを提供することができ、一つのAZがダウンしてもデータの整合性に影響はありません。

### シンプルなアーキテクチャ

TiDB、TiKV、PDは３つのAZ間に分散配置され、可用性が最も高くなるようにデプロイされます。

![3-AZ デプロイメントアーキテクチャ](/media/deploy-3dc.png)

**利点:**

- すべてのレプリカは３つのAZ間に分散しており、高い可用性と災害復旧能力を持っています。
- 一つのAZがダウンしてもデータは失われません (RPO = 0)。
- 一つのAZがダウンした場合でも、他の２つのAZは自動的にリーダーの選出を開始し、一定の期間内にサービスを自動的に再開します（ほとんどの場合20秒以内）。詳細については、以下の図を参照してください:

![3-AZ デプロイメントの災害復旧](/media/deploy-3dc-dr.png)

**欠点:**

ネットワーク遅延によってパフォーマンスが影響を受けることがあります。

- 書き込みについては、すべてのデータを少なくとも２つのAZにレプリケートする必要があります。TiDBは２段階コミットを使用して書き込みを行うため、２つのAZ間のネットワークの遅延と同じくらいの待ち時間がかかります。
- リーダーが読み込みリクエストを送信するTiDBノードと同じAZにない場合、読み込みパフォーマンスもネットワークの遅延に影響を受けます。
- 各TiDBトランザクションはPDリーダーからTime Stamp Oracle（TSO）を取得する必要があります。つまり、TiDBとPDのリーダーが同じAZにない場合、トランザクションのパフォーマンスもネットワークの遅延に影響を受けます。書き込みリクエストごとにTSOを２回取得する必要があるためです。

### 最適化されたアーキテクチャ

３つのAZのうちすべてのAZがアプリケーションにサービスを提供する必要がない場合、リクエストを１つのAZにディスパッチし、TiKVリージョンリーダーとPDリーダーを同じAZに配置するようにスケジューリングポリシーを構成することができます。このようにすることで、AZ間のネットワーク遅延を考慮する必要がなくなります。このAZがダウンした場合、PDリーダーやTiKVリージョンリーダーは他の残存しているAZで自動的に選出され、リクエストをまだ生きているAZに切り替えるだけで済みます。

![読み込みパフォーマンス最適化された 3-AZ デプロイメント](/media/deploy-3dc-optimize.png)

**利点:**

クラスタの読み込みパフォーマンスとTSOの取得能力が改善されます。スケジューリングポリシーの構成テンプレートは以下のようになります:

```shell
-- 他のAZのすべてのリーダーをアプリケーションにサービスを提供しているAZに排除します。
config set label-property reject-leader LabelName labelValue

-- PDリーダーの移行と優先度の設定
member leader transfer pdName1
member leader_priority pdName1 5
member leader_priority pdName2 4
member leader_priority pdName3 3
```

      server.labels: { zone: "z1", az: "az1", rack: "r2", host: "32" }
  - host: 10.63.10.33
    config:
      server.labels: { zone: "z1", az: "az1", rack: "r2", host: "33" }

  - host: 10.63.10.34
    config:
      server.labels: { zone: "z2", az: "az2", rack: "r1", host: "34" }
  - host: 10.63.10.35
    config:
      server.labels: { zone: "z2", az: "az2", rack: "r1", host: "35" }
  - host: 10.63.10.36
    config:
      server.labels: { zone: "z2", az: "az2", rack: "r2", host: "36" }
  - host: 10.63.10.37
    config:
      server.labels: { zone: "z2", az: "az2", rack: "r2", host: "37" }

  - host: 10.63.10.38
    config:
      server.labels: { zone: "z3", az: "az3", rack: "r1", host: "38" }
  - host: 10.63.10.39
    config:
      server.labels: { zone: "z3", az: "az3", rack: "r1", host: "39" }
  - host: 10.63.10.40
    config:
      server.labels: { zone: "z3", az: "az3", rack: "r2", host: "40" }
  - host: 10.63.10.41
    config:
      server.labels: { zone: "z3", az: "az3", rack: "r2", host: "41" }

前述の例では、`zone`はレプリカの分離を制御する論理的な可用性ゾーンレイヤであり、この例のクラスタには3つのレプリカがあります。

将来的にAZをスケーリングアウトする可能性があるため、`az`、`rack`、`host`の3層ラベル構造は直接採用されません。`AZ2`、`AZ3`、`AZ4`をスケーリングアウトする場合、対応する可用性ゾーンでAZをスケーリングアウトし、対応するAZでラックをスケーリングアウトする必要があります。

この3層ラベル構造が直接採用される場合、AZをスケーリングアウトした後、新しいラベルを適用する必要があり、TiKVのデータを再バランスする必要があります。

### 高可用性と災害復旧分析

リージョン内の複数のAZ配置により、1つのAZが故障してもクラスタは手動での介入なしでサービスを自動的に回復できます。データの整合性も保証されます。スケジューリングポリシーはパフォーマンスを最適化するために使用されますが、障害発生時にはこれらのポリシーはパフォーマンスよりも可用性を優先します。