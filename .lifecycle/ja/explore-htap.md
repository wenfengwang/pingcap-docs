---
title: HTAPの探索
summary: TiDB HTAPの機能を探索して使用する方法について学びます。

# HTAPの探索

このガイドでは、TiDB Hybrid Transactional and Analytical Processing（HTAP）の機能を探索して使用する方法について説明します。

> **注意：**
>
> TiDB HTAPを初めて使用する場合は、[HTAPのクイックスタート](/quick-start-with-htap.md)を参照してください。

## ユースケース

TiDB HTAPは急速に増加する大量のデータを処理し、DevOpsのコストを削減し、自己ホスト型またはクラウド環境に簡単に展開できます。これにより、データ資産の価値をリアルタイムで提供します。

次のようなHTAPの典型的なユースケースがあります：

- ハイブリッドワークロード

    TiDBをリアルタイムのオンライン解析処理（OLAP）に使用するハイブリッド負荷シナリオでは、データに対するTiDBのエントリポイントのみを提供すれば良く、TiDBは具体的なビジネスに基づいて異なる処理エンジンを自動で選択します。

- リアルタイムストリーム処理

    TiDBをリアルタイムストリーム処理シナリオで使用する場合、TiDBは定期的に流れ込むすべてのデータをリアルタイムでクエリできるようにします。同時に、TiDBは高い並行データワークロードとビジネスインテリジェンス（BI）クエリも処理できます。

- データハブ

    TiDBをデータハブとして使用すると、TiDBはアプリケーションとデータウェアハウスのデータをシームレスに接続し、特定のビジネスニーズを満たすことができます。

TiDB HTAPのユースケースに関する詳細は、[PingCAPウェブサイトのHTAPに関するブログ](https://en.pingcap.com/blog/?tag=htap)を参照してください。

TiDBの全体的なパフォーマンスを向上させるには、次の技術シナリオでHTAPを使用することをお勧めします：

- 解析処理のパフォーマンスを向上させる

    アプリケーションに複雑な解析クエリ（集計および結合操作など）が関与し、これらのクエリが大量のデータ（1,000万行以上）で実行される場合、行ベースのストレージエンジンである[TiKV](/tikv-overview.md)は、これらのクエリで効果的にインデックスを使用できず、インデックスの選択性が低い場合、パフォーマンス要件を満たすことができません。

- ハイブリッドワークロードの分離

    高並行オンライントランザクション処理（OLTP）ワークロードを処理する一方で、システムは一部のOLAPワークロードも処理する場合、全体的なシステムの安定性を確保するために、OLAPクエリがOLTPのパフォーマンスに影響を与えないようにすることを期待します。

- ETLテクノロジースタックの簡素化

    処理するデータが中規模（100 TB未満）、データ処理とスケジューリングプロセスが比較的単純であり、同時実行数が高くない（10未満）場合、システムの技術スタックを簡素化したいと考えるかもしれません。OLTP、ETL、およびOLAPシステムで使用される複数の異なるテクノロジースタックを1つのデータベースで置き換えることで、トランザクションシステムと解析システムの要件を満たすことができます。これにより、技術的複雑さと保守人員の必要性が低減します。

- 強力な一貫性のある解析

    リアルタイムで強力な一貫性のデータ解析と計算を達成し、解析結果をトランザクションデータと完全に一致させる場合、データの遅延と不整合問題を回避する必要があります。

## アーキテクチャ

TiDBでは、オンライントランザクション処理（OLTP）向けの行ベースストレージエンジンである[TiKV](/tikv-overview.md)と、オンライン解析処理（OLAP）向けの列ベースストレージエンジンである[TiFlash](/tiflash/tiflash-overview.md)が共存し、データを自動的にレプリケートして強力な一貫性を維持します。

アーキテクチャの詳細については、[TiDB HTAPのアーキテクチャ](/tiflash/tiflash-overview.md#architecture)を参照してください。

## 環境の準備

TiDB HTAPの機能を探索する前に、データのボリュームに応じてTiDBと対応するストレージエンジンを展開する必要があります。データボリュームが大きい場合（たとえば、100 TB）、主要なソリューションとしてTiFlash Massively Parallel Processing（MPP）を使用し、補助的なソリューションとしてTiSparkを使用することをお勧めします。

- TiFlash

    - TiFlashノードのないTiDBクラスターを展開している場合、現在のTiDBクラスターにTiFlashノードを追加します。詳細については、[TiFlashクラスターのスケーリングアウト](/scale-tidb-using-tiup.md#scale-out-a-tiflash-cluster)を参照してください。
    - TiDBクラスターを展開していない場合、[TiUPを使用してTiDBクラスターを展開](/production-deployment-using-tiup.md)してください。最小限のTiDBトポロジに基づいて、[TiFlashのトポロジ](/tiflash-deployment-topology.md)も展開する必要があります。
    - TiFlashノードの数を選択する際は、次のシナリオを考慮してください：

        - 小規模な解析処理とアドホッククエリにOLTPが必要な場合は、1つまたは複数のTiFlashノードを展開します。これにより、解析クエリのスピードを劇的に向上させることができます。
        - OLTPスループットがTiFlashノードのI/O使用率に著しい圧力を与えない場合、各TiFlashノードは計算により多くのリソースを使用し、したがってTiFlashクラスターはほぼ線形のスケーラビリティを持つことができます。TiFlashノードの数は、期待されるパフォーマンスと応答時間に基づいて調整する必要があります。
        - OLTPスループットが比較的高い場合（たとえば、書き込みまたは更新スループットが1時間あたり1,000万行を超える場合）、ネットワークと物理ディスクの書き込み容量が限られているため、TiKVとTiFlash間のI/Oがボトルネックとなり、読み書きのホットスポットが生じやすくなります。この場合、TiFlashノードの数と解析処理の計算量との関係は複雑で非線形であるため、システムの実際の状況に基づいてTiFlashノードの数を調整する必要があります。

- TiSpark

    - データをSparkで解析する必要がある場合は、TiSparkを展開します。プロセスの詳細については、[TiSparkユーザーガイド](/tispark-overview.md)を参照してください。

## データの準備

TiFlashを展開した後、TiKVはデータをTiFlashに自動的にレプリケートしません。手動でTiFlashにレプリケートする必要があります。その後、TiDBは対応するTiFlashのレプリカを作成します。

- TiDBクラスターにデータがない場合、まずデータをTiDBに移行します。詳細については、[データ移行](/migration-overview.md)を参照してください。
- TiDBクラスターに上流からレプリケーションされたデータがすでにある場合、TiFlashを展開した後、データレプリケーションは自動的に開始しません。手動でTiFlashにレプリケーションするテーブルを指定する必要があります。詳細については、[TiFlashの使用](/tiflash/tiflash-overview.md#use-tiflash)を参照してください。

## データ処理

TiDBを使用すると、クエリや書き込みリクエストのためのSQLステートメントを簡単に入力できます。TiFlashレプリカを持つテーブルでは、TiDBはフロントエンドオプティマイザを使用して自動的に最適な実行計画を選択します。

> **注意：**
>
> TiFlashのMPPモードはデフォルトで有効になっています。SQLステートメントを実行するとき、TiDBはオプティマイザを通じて自動的にMPPモードで実行するかどうかを判断します。
>
> - TiFlashのMPPモードを無効にするには、[tidb_allow_mpp](/system-variables.md#tidb_allow_mpp-new-in-v50)システム変数の値を `OFF` に設定します。
> - クエリの実行にTiFlashのMPPモードを強制的に有効にするには、[tidb_allow_mpp](/system-variables.md#tidb_allow_mpp-new-in-v50)および[tidb_enforce_mpp](/system-variables.md#tidb_enforce_mpp-new-in-v51)の値を `ON` に設定します。
> - 特定のクエリを実行するためにTiDBがMPPモードを選択したかどうかを確認するには、[MPPモードの説明ステートメント](/explain-mpp.md#explain-statements-in-the-mpp-mode)を参照してください。`EXPLAIN`ステートメントの出力に`ExchangeSender`および`ExchangeReceiver`オペレータが含まれている場合、MPPモードが使用されています。

## パフォーマンスモニタリング

TiDBを使用すると、TiDBクラスターの状態やパフォーマンスメトリクスを次のいずれかの方法でモニタリングできます：

- [TiDBダッシュボード](/dashboard/dashboard-intro.md)：TiDBクラスターの全体的な実行状況を表示し、読み書きトラフィックの分布とトレンドを分析し、遅いクエリの詳細な実行情報を学ぶことができます。
- [モニタリングシステム（Prometheus & Grafana）](/grafana-overview-dashboard.md)：PD、TiDB、TiKV、TiFlash、TiCDC、およびNode_exporterなど、TiDBクラスター関連のコンポーネントのモニタリングパラメータを表示できます。

TiDBクラスターおよびTiFlashクラスターのアラートルールについては、[TiDBクラスターのアラートルール](/alert-rules.md)および[TiFlashクラスターのアラートルール](/tiflash/tiflash-alert-rules.md)を参照してください。

## トラブルシューティング

TiDBの使用中に問題が発生した場合は、次のドキュメントを参照してください：

- [遅いクエリの分析](/analyze-slow-queries.md)
- [コストのかかるクエリの特定](/identify-expensive-queries.md)
- [ホットスポットのトラブルシューティング](/troubleshoot-hot-spot-issues.md)
- [TiDBクラスターのトラブルシューティングガイド](/troubleshoot-tidb-cluster.md)
- [TiFlashクラスターのトラブルシューティング](/tiflash/troubleshoot-tiflash.md)

```
## 次は

- TiFlashのバージョン、重要なログ、システムテーブルを確認するには、[TiFlashクラスタを維持する](/tiflash/maintain-tiflash.md)を参照してください。
- 特定のTiFlashノードを削除するには、[TiFlashクラスタをスケールアウトする](/scale-tidb-using-tiup.md#scale-out-a-tiflash-cluster)を参照してください。
```
