---
title: TiDB Cloud FAQs
summary: TiDB Cloudに関するよくある質問（FAQ）について学習します。

# TiDB Cloud FAQs

<!-- markdownlint-disable MD026 -->

このドキュメントは、TiDB Cloudに関するよくある質問をリストしています。

## 一般的なFAQ

### TiDB Cloudとは何ですか？

TiDB Cloudは、直感的なコンソールを介して制御できる完全に管理されたクラウドインスタンスで、TiDBクラスタの展開、管理、および維持がさらに簡素化されます。 Amazon Web ServicesまたはGoogle Cloud上で簡単に展開し、使命-criticalなアプリケーションを迅速に構築できます。

TiDB Cloudでは、簡単なクリック操作でTiDBクラスタを縮小または拡大できるため、コストをかけずにデータベースを必要な期間と必要な数だけプロビジョニングできます。

### TiDBとTiDB Cloudの関係は何ですか？

TiDBはオープンソースのデータベースであり、TiDB Self-Hostedを自社のデータセンター、自己管理型のクラウド環境、またはその両方で実行したい組織にとって最良の選択肢です。

TiDB CloudはTiDBの完全に管理されたクラウドデータベースサービスです。使命-criticalな本番環境のTiDBクラスタを管理できる使いやすいWebベースの管理コンソールが備わっています。

### TiDB CloudはMySQLと互換性がありますか？

現時点では、TiDB Cloudはトリガー、ストアドプロシージャ、ユーザー定義関数、外部キーを除く、MySQL 5.7構文の大部分をサポートしています。詳細については、[MySQLとの互換性](https://docs.pingcap.com/tidb/stable/mysql-compatibility)を参照してください。

### TiDB Cloudで使用できるプログラミング言語は何ですか？

MySQLクライアントまたはドライバーでサポートされている任意の言語を使用できます。

### TiDB Cloudを実行できる場所はどこですか？

TiDB Cloudは現在、Amazon Web ServicesおよびGoogle Cloudで利用できます。

### TiDB Cloudは異なるクラウドサービスプロバイダ間でのVPCピアリングをサポートしていますか？

いいえ。

### TiDB CloudではどのバージョンのTiDBがサポートされていますか？

- 2023年7月25日から、新しいTiDB DedicatedクラスタのデフォルトのTiDBバージョンはv7.1.1です。
- 2023年3月7日から、新しいTiDB ServerlessクラスタのデフォルトのTiDBバージョンはv6.6.0です。

詳細については、[TiDB Cloudリリースノート](/tidb-cloud/tidb-cloud-release-notes.md)を参照してください。

### TiDBまたはTiDB Cloudを本番で使用している企業はどのような企業ですか？

TiDBは、金融サービス、ゲーム、電子商取引など、さまざまな産業をカバーする1500以上のグローバル企業に信頼されています。当社のユーザーには、Square（米国）、Shopee（シンガポール）、および中国銀聯（中国）などが含まれます。詳細については、[事例研究](https://en.pingcap.com/customers/)をご覧ください。

### SLAはどのように見えますか？

TiDB Cloudは99.99%のSLAを提供しています。詳細については、[TiDB Cloudサービスのサービスレベル契約](https://en.pingcap.com/legal/service-level-agreement-for-tidb-cloud-services/)を参照してください。

### TiDB Cloudについてもっと詳しく学ぶ方法は？

TiDB Cloudについてもっと詳しく学ぶ最良の方法は、ステップバイステップのチュートリアルに従うことです。次のトピックをチェックして初めてください：

- [TiDB Cloud概要](/tidb-cloud/tidb-cloud-intro.md)
- [はじめに](/tidb-cloud/tidb-cloud-quickstart.md)
- [TiDB Serverlessクラスタを作成する](/tidb-cloud/create-tidb-cluster-serverless.md)

### クラスタを削除する際の`XXX's Org/default project/Cluster0`とは何を指しますか？

TiDB Cloudでは、クラスタは組織名、プロジェクト名、およびクラスタ名の組み合わせによって一意に識別されます。意図したクラスタを削除していることを確認するためには、そのクラスタの完全修飾名（`XXX's Org/default project/Cluster0`など）を提供する必要があります。

## アーキテクチャのFAQ

### TiDBクラスタには異なるコンポーネントがあります。TiDB、TiKV、およびTiFlashノードとは何ですか？

TiDBは、TiKVまたはTiFlashストアから戻されたクエリからデータを集約するSQLコンピューティングレイヤです。 TiDBは水平方向に拡張可能であり、TiDBノードの数を増やすと、クラスタが処理できる同時クエリの数も増加します。

TiKVは、OLTPデータを格納するためのトランザクションストアです。 TiKVのすべてのデータは複数のレプリカ（デフォルトで3つのレプリカ）で自動的に維持されるため、TiKVはネイティブの高可用性を持ち、自動フェイルオーバをサポートします。 TiKVは水平方向に拡張可能であり、トランザクションストアの数を増やすとOLTPスループットが増加します。

TiFlashは、トランザクションストア（TiKV）からデータをリアルタイムで複製し、リアルタイムのOLAPワークロードをサポートする分析ストレージです。 TiFlashは列ごとにデータを格納して分析処理を高速化します。 TiFlashも水平方向に拡張可能であり、TiFlashノードを増やすとOLAPストレージと計算能力が増加します。

PD（Placement Driver）は、TiDBクラスタ全体のメタデータを格納するための"脳"として機能し、リアルタイムでTiKVノードから報告されたデータ分布状態に基づいて特定のTiKVノードにデータスケジューリングコマンドを送信します。 TiDB Cloudでは、各クラスタのPDはPingCAPによって管理されるため、ユーザーは見ることも管理することもできません。

### TiKVノード間でデータをいかにしてレプリケートしますか？

TiKVは、キー値スペースをキーレンジに分割し、各キーレンジを"リージョン"として扱います。 TiKVでは、データはクラスタ内のすべてのノードに分散され、リージョンを基本単位として使用します。 PDは、リージョンをクラスタ内のすべてのノードにできるだけ均等にスプレッド（スケジュール）する責任を持っています。

TiDBは、Raftコンセンサスアルゴリズムを使用して、リージョンごとにデータをレプリケートします。異なるノードに保存されているリージョンの複数のレプリカがRaftグループを形成します。

各データ変更はRaftログとして記録されます。 Raftログのレプリケーションにより、データはRaftグループの複数のノードに安全かつ信頼性を持ってレプリケートされます。

## 高可用性のFAQ

### TiDB Cloudはいかにして高可用性を確保していますか？

TiDBは、Raftコンセンサスアルゴリズムを使用してデータを高可用性に保ち、Raftグループ内のストレージ全体に安全にレプリケートします。データは冗長にTiKVノード間にコピーされ、異なる可用ゾーンに配置されており、マシンまたはデータセンターの障害に対して保護されています。自動フェイルオーバにより、TiDBは常に動作していることを保証します。

サービスプロバイダとして、データセキュリティを真剣に考えています。 [Service Organization Control (SOC) 2 Type 1準拠](https://en.pingcap.com/press-release/pingcap-successfully-completes-soc-2-type-1-examination-for-tidb-cloud/)で必要とされる厳格な情報セキュリティポリシーと手順を整備しています。これにより、データが安全で利用可能であり機密情報であることを確保しています。

## 移行のFAQ

### 他のRDBMSからTiDB Cloudへの簡単な移行経路はありますか？

TiDBはMySQLと高い互換性があります。自己ホストのMySQLインスタンスまたはパブリッククラウドが提供するRDSサービスからTiDBに滑らかにデータを移行できます。詳細については、[データマイグレーションを使用したMySQL互換データベースからTiDB Cloudへのマイグレーション](/tidb-cloud/migrate-from-mysql-using-data-migration.md)を参照してください。

## バックアップと復元のFAQ

### TiDB Cloudは増分バックアップをサポートしていますか？

いいえ。クラスタのバックアップ保持期間内で任意の時点にデータを復元する必要がある場合、PITR（ポイントインタイムリカバリ）を使用できます。詳細については、[TiDB DedicatedクラスタでのPITRの使用](/tidb-cloud/backup-and-restore.md#automatic-backup)または[TiDB ServerlessクラスタでのPITRの使用](/tidb-cloud/backup-and-restore-serverless.md#restore)を参照してください。

## HTAPのFAQ

### TiDB CloudのHTAP機能をどのように利用すればよいですか？

従来、オンライントランザクション処理（OLTP）データベースとオンライン解析処理（OLAP）データベースの2種類のデータベースがありました。OLTPおよびOLAPリクエストは通常、異なるお互いに孤立したデータベースで処理されます。この伝統的なアーキテクチャでは、OLTPデータベースからOLAPのデータウェアハウスやデータレイクにデータを移行することは、長くエラーの多いプロセスでした。

Hybrid Transactional Analytical Processing（HTAP）データベースとして、TiDB Cloudは、データをOLTP（TiKV）ストアとOLAP（TiFlash）ストア間で確実に自動複製することで、システムアーキテクチャを単純化し、保守の複雑さを減らし、トランザクションデータのリアルタイム分析をサポートします。典型的なHTAPユースケースには、ユーザー個別化、AI推薦、不正検出、ビジネスインテリジェンス、リアルタイムレポートが含まれます。

さらにHTAPシナリオについては、[データプラットフォームを単純化するHTAPデータベースを構築する方法](https://pingcap.com/blog/how-we-build-an-htap-database-that-simplifies-your-data-platform)を参照してください。

### データを直接TiFlashにインポートすることはできますか？

いいえ。TiDB Cloudにデータをインポートすると、データはTiKVにインポートされます。インポートが完了した後、指定したテーブルをTiFlashにレプリケートするためにSQLステートメントを使用できます。その後、TiDBは指定したテーブルのレプリカをTiFlashに作成します。詳細については、[TiFlashレプリカを作成する](/tiflash/create-tiflash-replicas.md)を参照してください。

### TiFlashのデータをCSV形式でエクスポートすることはできますか？

いいえ。TiFlashデータはエクスポートできません。

## セキュリティのFAQ

### TiDB Cloudはセキュリティが確保されていますか？

```markdown
+ {R}
  + {R}
    + {R}
      + {R}
```
```markdown
+ {T}
  + {T}
    + {T}
      + {T}
```