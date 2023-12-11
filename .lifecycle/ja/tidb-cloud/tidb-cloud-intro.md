---
title: TiDB クラウド イントロダクション
summary: TiDB クラウドとそのアーキテクチャについて学ぶ
category: intro
---

# TiDB クラウド イントロダクション

[TiDB Cloud](https://www.pingcap.com/tidb-cloud/) は、クラウド上に、オープンソースのハイブリッドトランザクショナルおよびアナリティカルプロセッシング（HTAP）データベースである[TiDB](https://docs.pingcap.com/tidb/stable/overview)をもたらす完全に管理されたデータベースサービス（DBaaS）です。 TiDB クラウドは、データベースの複雑さではなく、アプリケーションに集中できるように、データベースのデプロイメントや管理を簡単に行う方法を提供しています。Google Cloud や Amazon Web Services（AWS）上でミッションクリティカルなアプリケーションを迅速に構築できるように、TiDB Cloud クラスタを作成できます。

![TiDB Cloud 概要](/media/tidb-cloud/tidb-cloud-overview.png)

## TiDB クラウドの利点

TiDB クラウドを使用すると、ほとんど訓練を受けることなく、インフラストラクチャの管理やクラスターのデプロイメントなどの複雑なタスクを簡単に処理できます。

- 開発者やデータベース管理者（DBA）は、多くのオンライントラフィックを簡単に処理し、複数のデータセット上の大量のデータを迅速に分析できます。

- あらゆる規模の企業は、前払いなしでビジネスの成長に適応するために、TiDB クラウドを簡単に展開し管理することができます。

次のビデオをご覧いただき、TiDB クラウドについて詳しく学びましょう：

<iframe width="600" height="450" src="https://www.youtube.com/embed/skCV9BEmjbo?enablejsapi=1" title="Why TiDB Cloud?" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

TiDB クラウドでは、次の主な機能を利用できます。

- **高速かつカスタマイズされたスケーリング**

    重要なワークロードのために数百ノードに弾力的かつ透過的にスケールし、ACID トランザクションを維持します。シャーディングの必要はありません。また、パフォーマンスノードとストレージノードをビジネスニーズに応じて別々にスケーリングできます。

- **MySQL 互換性**

    TiDB の MySQL 互換性により、既存の MySQL インスタンスからコードを書き換える必要なく、アプリケーションの生産性を向上し、市場投入までの時間を短縮できます。

- **高可用性と信頼性**

    設計上自然な高可用性。複数のアベイラビリティーゾーン間でデータをレプリケーションし、デイリーバックアップや自動フェイルオーバーにより、ハードウェアの障害、ネットワークの分断、またはデータセンターの喪失に関係なく、ビジネスの継続性を確保します。

- **リアルタイムアナリティクス**

    組み込みのアナリティクスエンジンで現在のデータに一貫した分析クエリ結果を取得できます。TiDB クラウドは、ミッションクリティカルなアプリケーションを妨げることなく、現在のデータで一貫した分析クエリを実行します。

- **エンタープライズグレードのセキュリティ**

    専用ネットワークおよびマシンでデータを安全に保護し、データの飛行中および保存中の暗号化をサポートします。TiDB クラウドは、SOC 2 タイプ 2、ISO 27001:2013、ISO 27701 に認証されており、GDPR に完全に準拠しています。

- **完全管理型サービス**

    クリック数回で TiDB クラスタをデプロイし、スケーリングし、監視し、管理できます。使いやすい Web ベースの管理プラットフォームを介して操作できます。

- **マルチクラウドサポート**

    クラウドベンダーロックインをせずに柔軟性を維持できます。TiDB クラウドは現在、AWS および Google Cloud で利用できます。

- **シンプルな価格設定プラン**

    透明で前払いのない価格設定で、使用した分だけ支払います。隠れた料金は一切ありません。

- **ワールドクラスのサポート**

    サポートポータル、<a href="mailto:tidbcloud-support@pingcap.com">メール</a>、チャット、ビデオ会議を通じてワールドクラスのサポートを受けることができます。

## デプロイメントオプション

TiDB クラウドには、次の 2 つのデプロイメントオプションが用意されています。

- [TiDB Serverless](https://www.pingcap.com/tidb-serverless)

    TiDB Serverless は、完全管理型のマルチテナント TiDB オファリングです。インスタントで、オートスケーリング可能な MySQL 互換のデータベースを提供し、使用制限を超えた後は消費ベースの請求を提供します。

- [TiDB Dedicated](https://www.pingcap.com/tidb-dedicated)

    TiDB Dedicated は、クロスゾーンの高可用性、水平スケーリング、および[HTAP](https://en.wikipedia.org/wiki/Hybrid_transactional/analytical_processing)の利点を備えた本番用です。

TiDB Serverless と TiDB Dedicated の機能比較については、[TiDB: An advanced, open source, distributed SQL database](https://www.pingcap.com/get-started-tidb) を参照してください。

## アーキテクチャ

![TiDB クラウドアーキテクチャ](/media/tidb-cloud/tidb-cloud-architecture.png)

- TiDB VPC（仮想プライベートクラウド）

    各 TiDB クラウドクラスタには、TiDB ノードと TiDB オペレーターノード、ログノードなど、すべての補助ノードが独立した VPC に展開されます。

- TiDB クラウドセントラルサービス

    請求、アラート、メタストレージ、ダッシュボード UI などのセントラルサービスは独立して展開されています。インターネット経由でダッシュボード UI にアクセスし、TiDB クラスタを操作できます。

- あなたの VPC

    プライベートエンドポイント接続または VPC ピアリング接続を介して TiDB クラスタに接続できます。詳細については、[プライベートエンドポイント接続の設定](/tidb-cloud/set-up-private-endpoint-connections.md)、[VPC ピアリング接続の設定](/tidb-cloud/set-up-vpc-peering-connections.md) を参照してください。