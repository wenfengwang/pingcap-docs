---
title: TiDBスケジューリング
summary: TiDBクラスタでのPDスケジュールングコンポーネントを紹介します。
---

# TiDBスケジューリング

Placement Driver（[PD](https://github.com/tikv/pd)）は、TiDBクラスタでのマネージャーとして機能し、クラスタ内のリージョンをスケジュールします。この記事では、PDスケジューリングコンポーネントの設計と基本概念について紹介します。

## スケジューリングの状況

TiDBで使用される分散型のキーバリューストレージエンジンであるTiKVでは、データはリージョンとして組織化され、複数のストア上に複製されます。すべての複製では、リーダーが読み書きを担当し、フォロワーがリーダーからRaftログを複製する役割を担当します。

以下の状況について考えてみましょう：

* ストレージスペースを効率的に利用するために、同じリージョンの複数のレプリカはリージョンのサイズに応じて異なるノードに適切に配置する必要があります。
* 複数のデータセンタートポロジにおいて、1つのデータセンターの障害が全リージョンの1つのレプリカの障害を引き起こすだけで済むようにする必要があります。
* 新しいTiKVストアが追加された場合、データをそのストアに再バランスする必要があります。
* TiKVストアが障害した場合、PDは以下の点を考慮する必要があります：
    * 障害したストアの回復時間
        * 短い場合（例：サービスが再起動された場合）、スケジュールが必要かどうかを判断する
        * 長い場合（例：ディスクの故障とデータの喪失）、どのようにスケジューリングするかを考える
    * すべてのリージョンのレプリカ
        * いくつかのリージョンに対してレプリカ数が足りない場合、PDはそれらを補完する必要があります。
        * レプリカの数が予想以上に多い場合（例：障害したストアが回復後にクラスターに再参加した場合）、PDはそれらを削除する必要があります。
* 読み込み/書き込み操作はリーダーで行われ、それは個々のストア上だけでは分散されません。
* すべてのリージョンがホットではないため、すべてのTiKVストアの負荷を均等にする必要があります。
* リージョンがバランシングされる際、データの転送は多くのネットワーク/ディスクトラフィックとCPU時間を利用するため、オンラインサービスに影響を与える可能性があります。

これらの状況は同時に発生することがあり、解決が困難になります。また、システム全体が動的に変化しているため、スケジューラがクラスターに関するすべての情報を収集し、クラスターを調整する必要があります。そのため、PDがTiDBクラスターに導入されています。

## スケジューリングの要件

上記の状況は次の2つのタイプに分類することができます：

1. 分散型で高可用性のあるストレージシステムは、次の要件を満たす必要があります：

    * 正しい数のレプリカ。
    * レプリカは異なるマシンに、異なるトポロジに基づいて分散する必要があります。
    * クラスターはTiKVのピアの障害からの自動的な障害復旧ができるようにする必要があります。

2. より優れた分散システムは、次の最適化を持つ必要があります：

    * すべてのリージョンのリーダーがストア上で均等に分散する。
    * すべてのTiKVのピアのストレージ容量がバランスされる。
    * ホットスポットがバランスされる。
    * リージョンの負荷分散の速度は制限され、オンラインサービスが安定していることを確認する。
    * 運用者がピアを手動でオンライン/オフラインにすることができる。

最初の要件が満たされると、システムは障害に耐えることができます。2番目の要件が満たされると、リソースがより効率的に利用され、システムのスケーラビリティが向上します。

これらの目標を達成するためには、PDはまず、ピアの状態、Raftグループの情報、およびピアへのアクセス統計などの情報を収集する必要があります。そして、PDにいくつかの戦略を指定して、この情報と戦略からスケジューリングプランを作成できるようにします。最後に、PDはいくつかのオペレータをTiKVのピアに配布して、スケジューリングプランを完了させます。

## 基本的なスケジューリングオペレータ

すべてのスケジューリングプランには、次の3つの基本的なオペレータが含まれています：

* 新しいレプリカを追加する
* レプリカを削除する
* Raftグループ内のレプリカ間でリージョンのリーダーを転送する

これらはRaftコマンド`AddReplica`、`RemoveReplica`、および`TransferLeader`によって実装されます。

## 情報の収集

スケジューリングは情報の収集に基づいて行われます。要するに、PDスケジューリングコンポーネントはすべてのTiKVのピアとすべてのリージョンの状態を知る必要があります。TiKVのピアはPDに次の情報を報告します：

- 各TiKVのピアが報告する状態情報：

    各TiKVのピアは定期的にPDにハートビートを送信します。PDはストアが生存しているかどうかだけでなく、ハートビートメッセージの中で[`StoreState`](https://github.com/pingcap/kvproto/blob/master/proto/pdpb.proto#L473)を収集します。`StoreState`には次の情報が含まれます：

    * 総ディスクスペース
    * 利用可能なディスクスペース
    * リージョンの数
    * データの読み書き速度
    * スナップショットの送信/受信数（データはスナップショットを介してレプリカ間で複製される場合があります）
    * ストアが過負荷かどうか
    * ラベル（トポロジの認識に関する情報については[Perception of Topology](https://docs.pingcap.com/tidb/stable/schedule-replicas-by-topology-labels)を参照）

    PDコントロールを使用してTiKVストアのステータスを確認することができます。ステータスは「Up」、「Disconnect」、「Offline」、「Down」、「Tombstone」のいずれかです。以下にそれぞれのステータスの説明とその関係を示します。

    + **Up**: TiKVストアはサービス中です。
    + **Disconnect**: PDとTiKVストアの間のハートビートメッセージが20秒以上ロストしています。「max-store-down-time」で指定された時間を超える場合、ステータス「Disconnect」は「Down」に変更されます。
    + **Down**: PDとTiKVストアの間のハートビートメッセージが「max-store-down-time」（デフォルトでは30分）以上ロストしています。この状態では、TiKVストアは生き残るストア上の各リージョンのレプリカを補充し始めます。
    + **Offline**: TiKVストアはPD Controlで手動でオフラインに設定されます。これはストアがオフラインになるための中間的なステータスです。「Offline」の状態では、このストアはリロケーションの条件を満たす「Up」ストアにそのリージョンを移動します。`leader_count`と`region_count`（PD Controlで取得）がともに`0`を示す場合、ストアのステータスは「Offline」から「Tombstone」に変わります。「Offline」の状態では、ストアのサービスまたはストアが配置されている物理サーバーを無効にしないでください。ストアがオフラインになるプロセス中、リージョンをリロケーションする対象ストアがクラスターにない場合（例：クラスター内のレプリカを保持するためのストアが不十分な場合）、ストアは常に「Offline」の状態になります。
    + **Tombstone**: TiKVストアは完全にオフラインです。このステータスのTiKVを安全にクリーンアップするために、`remove-tombstone`インターフェースを使用することができます。

    ![TiKVストアのステータスの関係](/media/tikv-store-status-relationship.png)

- リージョンのリーダーが報告する情報：

    各リージョンのリーダーは定期的にPDにハートビートを送信し、[`RegionState`](https://github.com/pingcap/kvproto/blob/master/proto/pdpb.proto#L312)を報告します。`RegionState`には次の情報が含まれます：
    
    * リーダー自体の位置
    * 他のレプリカの位置
    * オフラインレプリカの数
    * データの読み書き速度

PDはこれら2種類のハートビートからクラスターの情報を収集し、それに基づいて決定を行います。

また、より詳細な判断をするために、PDは拡張されたインターフェースからさらに多くの情報を取得することができます。たとえば、ストアのハートビートが途切れている場合、PDはピアが一時的に退任したのか永遠に退任したのかを知ることができません。デフォルトでは、しばらく待ってから（30分）ハートビートがまだ受信されていない場合、ストアをオフラインと見なします。その後、PDはストア上のすべてのリージョンを他のストアにバランスします。

ただし、ストアは運用者によって手動でオフラインに設定される場合もあります。その場合、運用者はPDコントロールインターフェースを使用してPDに通知することができます。その後、PDはすぐにすべてのリージョンをバランスします。

## スケジューリングの戦略

情報を収集した後、PDはスケジューリングプランを作成するためのいくつかの戦略が必要です。

**戦略1：リージョンのレプリカの数が正しいことが必要です**

PDは、リージョンのレプリカ数がリージョンのリーダーからのハートビートによって正しくないことがわかるかもしれません。それが起こると、PDはレプリカの数を追加/削除することによって調整できます。レプリカ数が正しくない理由は次のとおりです：

* ストアの障害により、一部のリージョンのレプリカ数が予想よりも少ない場合
* ストアの障害後の回復により、一部のリージョンのレプリカ数が予想よりも多くなる場合
* [`max-replicas`](https://github.com/pingcap/pd/blob/v4.0.0-beta/conf/config.toml#L95)が変更される

**戦略2：リージョンのレプリカは異なる位置にある必要があります**

ここで言う「位置」は「マシン」とは異なります。一般的に、PDはリージョンのレプリカが同じピアに存在しないようにすることで、ピアの障害によって複数のレプリカが失われることを防ぐことができます。ただし、実際のプロダクション環境では、次のような要件があるかもしれません：

* 複数のTiKVピアが1つのマシン上に存在する。
* TiKVピアが複数のラック上にあり、ラックの障害が発生してもシステムが利用可能である。
* TiKVピアが複数のデータセンターにあり、データセンターの障害が発生してもシステムが利用可能である。

これらの要件のキーは、ピアが同じ「位置」を持つことができ、その「位置」が障害耐性の最小単位であることです。リージョンのレプリカは同じユニットには存在していてはなりません。そのため、TiKVのピアにラベルを設定し、PD上で[location-labels](https://github.com/pingcap/pd/blob/v4.0.0-beta/conf/config.toml#L100)を設定して、どのラベルが位置を示すのに使用されるかを指定します。

**戦略3：レプリカはストア間でバランスする必要があります**
```
The size limit of a Region replica is fixed, so keeping the replicas balanced between stores is helpful for data size balance.

**Strategy 4: Leaders need to be balanced between stores**

Read and write operations are performed on leaders according to the Raft protocol, so that PD needs to distribute leaders into the whole cluster instead of several peers.

**Strategy 5: Hot spots need to be balanced between stores**

PD can detect hot spots from store heartbeats and Region heartbeats, so that PD can distribute hot spots.

**Strategy 6: Storage size needs to be balanced between stores**

When started up, a TiKV store reports `capacity` of storage, which indicates the store's space limit. PD will consider this when scheduling.

**Strategy 7: Adjust scheduling speed to stabilize online services**

Scheduling utilizes CPU, memory, network and I/O traffic. Too much resource utilization will influence online services. Therefore, PD needs to limit the number of the concurrent scheduling tasks. By default this strategy is conservative, while it can be changed if quicker scheduling is required.

## Scheduling implementation

PD collects cluster information from store heartbeats and Region heartbeats, and then makes scheduling plans from the information and strategies. Scheduling plans are a sequence of basic operators. Every time PD receives a Region heartbeat from a Region leader, it checks whether there is a pending operator on the Region or not. If PD needs to dispatch a new operator to a Region, it puts the operator into heartbeat responses, and monitors the operator by checking follow-up Region heartbeats.

Note that here "operators" are only suggestions to the Region leader, which can be skipped by Regions. Leader of Regions can decide whether to skip a scheduling operator or not based on its current status.
```