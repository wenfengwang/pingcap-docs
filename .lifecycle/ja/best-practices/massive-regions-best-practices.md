---
title: 大規模リージョンでのTiKVパフォーマンスチューニングのベストプラクティス
summary: 大量のリージョンを持つTiKVのパフォーマンスをチューニングする方法について学びます。
aliases: ['/docs/dev/best-practices/massive-regions-best-practices/','/docs/dev/reference/best-practices/massive-regions/']
---

# 大規模リージョンでのTiKVパフォーマンスチューニングのベストプラクティス

TiDBでは、データはリージョンに分割され、それぞれが特定のキー範囲のデータを格納しています。これらのリージョンは複数のTiKVインスタンスに分散されています。クラスタにデータが書き込まれると、数百万ものリージョンが作成される可能性があります。1つのTiKVインスタンスにあまりにも多くのリージョンがあると、クラスタに重い負荷をかけ、パフォーマンスに影響を与える可能性があります。

この文書では、Raftstore（TiKVのコアモジュール）のワークフローを紹介し、なぜ大量のリージョンがパフォーマンスに影響を与えるのか、TiKVパフォーマンスのチューニング方法を提供します。

## Raftstoreのワークフロー

TiKVインスタンスには複数のリージョンがあります。RaftstoreモジュールはRaftステートマシンを駆動してリージョンメッセージを処理します。これらのメッセージにはリージョン上の読み取りまたは書き込みリクエストの処理、Raftログの永続化やレプリケーション、Raftハートビートの処理が含まれます。ただし、リージョン数の増加はクラスタ全体のパフォーマンスに影響を与える可能性があります。これを理解するには、以下に示すRaftstoreのワークフローを学ぶ必要があります。

![Raftstoreのワークフロー](/media/best-practices/raft-process.png)

> **注記:**
>
> この図はRaftstoreのワークフローを表しており、実際のコード構造を表しているわけではありません。

上記の図から、TiDBサーバからのリクエストは、gRPCとストレージモジュールを経由して、KV（キー・バリュー）の読み取りおよび書き込みメッセージになり、対応するリージョンに送信されます。これらのメッセージは即座に処理されるのではなく、一時的に保存されます。Raftstoreは各リージョンが処理するメッセージがあるか定期的にポーリングします。リージョンが処理するメッセージがある場合、RaftstoreはそのリージョンのRaftステートマシンを駆動してこれらのメッセージを処理し、これらのメッセージの状態変化に応じた後続の操作を実行します。たとえば、書き込みリクエストが来た場合、Raftステートマシンはログをディスクに保存し、他のリージョンレプリカにログを送信します。ハートビート間隔が達成されたとき、Raftステートマシンは他のリージョンレプリカにハートビート情報を送信します。

## パフォーマンス問題

Raftstoreのワークフロー図から、各リージョンのメッセージが1つずつ処理されていることがわかります。リージョンが大量に存在すると、これらのリージョンのハートビートを処理するためにRaftstoreに時間がかかり、遅延を引き起こす可能性があります。その結果、一部の読み取りおよび書き込みリクエストが時間内に処理されないことがあります。読み取りおよび書き込みの圧力が高い場合、ロードされたRaftstoreのCPU使用率は容易にボトルネックになり、遅延を増大させ、パフォーマンスに影響を与える可能性があります。

一般的に、負荷のかかったRaftstoreのCPU使用率が85%以上になると、Raftstoreはビジー状態になり、ボトルネックになります。同時に、`提案待機時間` は数百ミリ秒になることがあります。

> **注記:**
>
> + 上記のRaftstoreのCPU使用率に関しては、Raftstoreは単一スレッドです。Raftstoreがマルチスレッドである場合、CPU使用率のしきい値（85%）を比例して増やすことができます。
> + ラフトストアスレッドにはI/O操作が存在するため、CPU使用率は100%になりません。

### パフォーマンスモニタリング

Grafanaの**TiKVダッシュボード**で次の監視メトリクスをチェックできます：

+ **Thread-CPU** パネルの中の `Raft store CPU`

    リファレンス値: `raftstore.store-pool-size * 85%`未満

    ![Raftstore CPUのチェック](/media/best-practices/raft-store-cpu.png)

+ **Raft Propose** パネルの中の `提案待機時間`

    `提案待機時間` は、リクエストがRaftstoreに送信されてからRaftstoreが実際にリクエストの処理を開始するまでの遅延を意味します。遅延が長いと、Raftstoreがビジーであるか、ログの追加処理に時間がかかり、時間内にリクエストを処理できなくなります。

    リファレンス値: クラスタサイズに応じて50〜100ミリ秒未満

    ![提案待機時間のチェック](/media/best-practices/propose-wait-duration.png)

## パフォーマンスチューニング方法

パフォーマンス問題の原因を特定した後、次の2つの観点から解決しようとしてください：

+ 1つのTiKVインスタンス上のリージョン数を減らす
+ 1つのリージョンのメッセージ数を減らす

### 方法1：Raftstore並列性を増やす

TiDB v3.0以降、Raftstoreはマルチスレッドモジュールにアップグレードされ、Raftstoreスレッドがボトルネックになる可能性が大幅に減少しました。

デフォルトで、`raftstore.store-pool-size` はTiKVで `2` に設定されています。Raftstoreでボトルネックが発生した場合は、この構成項目の値を状況に応じて適切に増やすことができます。ただし、不要なスレッド切り替えのオーバーヘッドを導入しないようにするために、この値をあまり高く設定しないことをお勧めします。

### 方法2：ハイバネートリージョンを有効にする

実際の状況では、読み取りおよび書き込みリクエストはすべてのリージョンに均等に分布されているのではなく、一部のリージョンに集中しています。そして、一時的にアイドル状態のリージョンに対してRaftリーダーとフォロワー間のメッセージ数を最小限に抑えることが可能です。これがハイバネートリージョンの機能です。この機能では、Raftstoreはアイドル状態のリージョンへの必要がない場合のリフレッシュメッセージを送信しません。そのため、これらのRaftステートマシンはハートビートメッセージを生成することがなくなり、Raftstoreのワークロードを大幅に減らすことができます。

ハイバネートリージョンはデフォルトで[TiKV master](https://github.com/tikv/tikv/tree/master)で有効になっています。この機能を必要に応じて構成することができます。詳細については、[ハイバネートリージョンの構成](/tikv-configuration-file.md)を参照してください。

### 方法3：`Region Merge` を有効にする

> **注記:**
>
> `Region Merge` はTiDB v3.0以降でデフォルトで有効になっています。

`Region Merge` を有効にすることで、リージョン数を減らすこともできます。`Region Split` とは逆に、スケジュールを使用して隣接する小さなリージョンを統合するプロセスのことです。データの削除や `Drop Table` や `Truncate Table` ステートメントの実行後、小さなリージョンまたは空のリージョンを統合してリソース消費を減らすことができます。

以下のパラメータを構成することで、 `Region Merge` を有効にできます：

{{< copyable "" >}}

```
config set max-merge-region-size 20
config set max-merge-region-keys 200000
config set merge-schedule-limit 8
```

詳細については、[Region Merge](https://tikv.org/docs/4.0/tasks/configure/region-merge/)および[PD構成ファイル](/pd-configuration-file.md#schedule)の次の3つの構成パラメータを参照してください：

- [`max-merge-region-size`](/pd-configuration-file.md#max-merge-region-size)
- [`max-merge-region-keys`](/pd-configuration-file.md#max-merge-region-keys)
- [`merge-schedule-limit`](/pd-configuration-file.md#merge-schedule-limit)

デフォルトの `Region Merge` パラメータの構成はかなり保守的です。[PDスケジューリングベストプラクティス](/best-practices/pd-scheduling-best-practices.md#region-merge-is-slow)で提供されている方法に基づいて `Region Merge` プロセスを高速化することができます。

### 方法4：TiKVインスタンスの数を増やす

I/OリソースとCPUリソースが十分な場合には、1つのマシンに複数のTiKVインスタンスを展開して1つのTiKVインスタンス上のリージョン数を減らすことができます。または、TiKVクラスタ内のマシン数を増やすこともできます。

### 方法5：`raft-base-tick-interval` を調整する

リージョン数を減らすだけでなく、単位時間あたりの各リージョンのメッセージ数を減らすことでRaftstoreにかかる負荷を軽減することもできます。たとえば、`raft-base-tick-interval` 構成項目の値を適切に増やすことができます：

{{< copyable "" >}}

```
[raftstore]
raft-base-tick-interval = "2s"
```

上記の構成では、 `raft-base-tick-interval` はRaftstoreが各リージョンのRaftステートマシンを駆動する時間間隔を表しており、すなわち、この時間間隔でRaftstoreはリージョンのRaftステートマシンにリフレッシュメッセージを送信します。この間隔の増加により、Raftstoreからのメッセージ数を効果的に減らすことができます。

注意すべきは、このリフレッシュメッセージの間隔は、選挙タイムアウトおよびハートビートの間隔も決定します。以下の例をご覧ください：

{{< copyable "" >}}

```
raft-election-timeout = raft-base-tick-interval * raft-election-timeout-ticks
raft-heartbeat-interval = raft-base-tick-interval * raft-heartbeat-ticks
```

リージョンのフォロワーが`raft-election-timeout` インターバル内でリーダーからハートビートを受信していない場合、これらのフォロワーはリーダーの障害が発生したと判断し、新たな選挙を開始します。`raft-heartbeat-interval` はリーダーがフォロワーにハートビートを送信する間隔です。したがって、 `raft-base-tick-interval` の値を増やすことで、Raftステートマシンが送信するネットワークメッセージ数を減らすだけでなく、Raftステートマシンがリーダー障害を検出するまでの時間も長くなります。

### 方法6：リージョンサイズの調整
```
The default size of a Region is 96 MiB, and you can reduce the number of Regions by setting Regions to a larger size. For more information, see [Tune Region Performance](/tune-region-performance.md).

> **Warning:**
>
> Currently, customized Region size is an experimental feature introduced in TiDB v6.1.0. It is not recommended that you use it in production environments. The risks are as follows:
>
> + Performance jitter might be caused.
> + The query performance, especially for queries that deal with a large range of data, might decrease.
> + The Region scheduling slows down.

## その他の問題と解決策

このセクションでは、その他の問題と解決策について説明します。

### PDリーダーの切り替えが遅い

PDは、PDがPDリーダーノードを切り替えた後にすばやくRegionのルーティングサービスを提供できるように、etcdにRegionメタ情報を永続化する必要があります。 Regionsの数が増えると、etcdのパフォーマンス問題が現れ、PDがリーダーを切り替える際にetcdからRegionメタ情報を取得するのが遅くなります。 数百万のRegionがあると、etcdからメタ情報を取得するのに10秒以上、あるいは数十秒かかることがあります。

この問題に対処するために、`use-region-storage`はTiDB v3.0以降、PDでデフォルトで有効になっています。この機能を有効にすると、PDはRegionメタ情報をローカルのLevelDBに保存し、他のメカニズムを介してPDノード間で情報を同期します。

### PDルーティング情報がタイムリーに更新されない

TiKVでは、pd-workerが定期的にRegionメタ情報をPDに報告します。 TiKVが再起動するかRegionリーダーを切り替えると、PDは統計を通じてRegionの`近似サイズ/キー数`を再計算する必要があります。したがって、大量のRegionがある場合、単一スレッドのpd-workerがボトルネックとなり、タスクが溜まり、タイムリーに処理されなくなる可能性があります。この状況では、PDは一定のRegionメタ情報をタイムリーに取得できず、したがってルーティング情報がタイムリーに更新されません。この問題は実際の読み書きに影響を与えませんが、不正確なPDスケジューリングを引き起こし、TiDBがRegionキャッシュを更新する際にいくつかのラウンドトリップを必要とする可能性があります。

**TiKV Grafana**パネルの**Task**の**Worker pending tasks**で、pd-workerのタスクが溜まっているかどうかを確認できます。一般的には、`pending tasks`を比較的低い値で保つ必要があります。

![Check pd-worker](/media/best-practices/pd-worker-metrics.png)

pd-workerは、[v3.0.5](/releases/release-3.0.5.md#tikv)以降、パフォーマンスが向上しました。同様の問題に遭遇した場合は、最新バージョンにアップグレードすることをお勧めします。

### Prometheusのメトリクスクエリーが遅い

大規模なクラスターでは、TiKVインスタンスの数が増えると、Prometheusはメトリクスをクエリーするプレッシャーが増大し、Grafanaがこれらのメトリクスを表示するのが遅くなります。この問題を緩和するために、v3.0以降ではメトリクスの事前計算が構成されています。 
```