---
title: 2022年のTiDB Cloudリリースノート
summary: 2022年のTiDB Cloudのリリースノートについて学びます。
---

# 2022年のTiDB Cloudリリースノート

このページでは、[TiDB Cloud](https://www.pingcap.com/tidb-cloud/)の2022年のリリースノートをリストアップしています。

## 2022年12月28日

**一般変更**

- 現在、すべての[サーバーレスティア](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスタのデフォルトTiDBバージョンを[v6.3.0](https://docs.pingcap.com/tidb/v6.3/release-6.3.0)から[v6.4.0](https://docs.pingcap.com/tidb/v6.4/release-6.4.0)にアップグレードした後、特定の状況でコールドスタートが遅くなることがあります。そのため、すべてのサーバーレスティアクラスタのデフォルトTiDBバージョンをv6.4.0からv6.3.0にロールバックし、問題をできるだけ早く修正してから、後で再度アップグレードします。

## 2022年12月27日

**一般変更**

- [サーバーレスティア](/tidb-cloud/select-cluster-tier.md#tidb-serverless)のすべてのクラスタのデフォルトTiDBバージョンを[v6.3.0](https://docs.pingcap.com/tidb/v6.3/release-6.3.0)から[v6.4.0](https://docs.pingcap.com/tidb/v6.4/release-6.4.0)にアップグレードしました。

- 専用ティアクラスタのポイントインタイムリカバリ（PITR）が一般利用可能（GA）になりました。
  
  PITRは、任意の時点のデータを新しいクラスタに復元できます。PITR機能を使用するには、TiDBクラスタのバージョンが少なくともv6.4.0であること、TiKVノードのサイズが少なくとも8 vCPUおよび16 GiBであることを確認してください。

  PITR機能は、[TiDB Cloudコンソール](https://tidbcloud.com)の**バックアップ設定**で有効または無効にできます。

  詳細については、[TiDBクラスタデータのバックアップとリストア](/tidb-cloud/backup-and-restore.md)を参照してください。

- 複数のチェンジフィードの管理および既存のチェンジフィードの編集をサポートしました。

  - 異なるデータレプリケーションタスクを管理するために必要なチェンジフィードを必要な数だけ作成できるようになりました。現在、各クラスタには最大10のチェンジフィードを持つことができます。詳細については、[チェンジフィードの概要](/tidb-cloud/changefeed-overview.md)を参照してください。
  - 一時停止状態の既存のチェンジフィードの構成を編集できるようになりました。詳細については、[チェンジフィードの編集](/tidb-cloud/changefeed-overview.md#edit-a-changefeed)を参照してください。

- Amazon Aurora MySQL、Amazon Relational Database Service（RDS）MySQL、または自己ホスト型MySQL互換データベースからTiDB Cloudにデータを直接移行する機能が一般利用可能になりました。

  - 次の6つの地域でサービスを提供します。
      - AWSオレゴン（us-west-2）
      - AWS N.バージニア（us-east-1）
      - AWSムンバイ（ap-south-1）
      - AWSシンガポール（ap-southeast-1）
      - AWS東京（ap-northeast-1）
      - AWSフランクフルト（eu-central-1）
  - 複数の仕様をサポートします。必要なパフォーマンスに応じて適切な仕様を選択でき、最適なデータ移行体験を実現できます。

  TiDB Cloudへのデータ移行方法については、[ユーザードキュメント](/tidb-cloud/migrate-from-mysql-using-data-migration.md)を参照してください。料金の詳細については、[データ移行の課金](/tidb-cloud/tidb-cloud-billing-dm.md)を参照してください。

- ローカルのCSVファイルをTiDB Cloudにインポートする機能をサポートしました。

  タスク構成を完了するのに数回のクリックだけで、ローカルのCSVデータをTiDBクラスタに迅速にインポートできます。この方法を使用する場合、クラウドストレージバケットパスとRole ARNを提供する必要はありません。インポート全体のプロセスは迅速かつスムーズです。

  詳細については、[ローカルファイルをTiDB Cloudにインポート](/tidb-cloud/tidb-cloud-import-local-files.md)を参照してください。

## 2022年12月20日

**一般変更**

- [Datadog](/tidb-cloud/monitor-datadog-integration.md)ダッシュボードにプロジェクト情報をフィルタとして追加しました。

  フィルタの`project name`を使用して、必要なクラスタを迅速に検索できます。

## 2022年12月13日

**一般変更**

- サーバーレスティア向けにTiDB Cloud SQLエディタ（ベータ版）を導入しました。

  これは、サーバーレスティアのデータベースに対して直接SQLクエリを編集および実行できるWebベースのSQLエディタです。サーバーレスティアでは、Web SQL ShellがSQLエディタに取って代わられます。

- 専用ティアでの[Changefeeds](/tidb-cloud/changefeed-overview.md)を使用してデータをストリーミングする機能をサポートしました。

  - [MySQLへのデータ変更ログのストリーミング](/tidb-cloud/changefeed-sink-to-mysql.md)をサポートします。
  
    MySQL/AuroraからTiDBにデータを移行する場合、予期せぬデータ移行の問題を防ぐために、MySQLをスタンバイデータベースとして使用することがよくあります。その場合、TiDBからMySQLにデータをストリーミングすることができます。

  - [Apache Kafkaへのデータ変更ログのストリーミング](/tidb-cloud/changefeed-sink-to-apache-kafka.md)（ベータ版）をサポートします。

    TiDBデータをメッセージキューにストリーミングすることは、データ統合シナリオに非常に一般的な要件です。Kafkaを使用して他のデータ処理システム（Snowflakeなど）との統合を実現したり、ビジネスの消費をサポートしたりできます。

  詳細については、[Changefeedの概要](/tidb-cloud/changefeed-overview.md)を参照してください。

- 組織所有者は**組織設定**で組織名を編集できます。

**コンソール変更**

- [TiDB Cloudコンソール](https://tidbcloud.com)のナビゲーションレイアウトを最適化し、新しいナビゲーション体験をユーザーに提供しました。

  新しいレイアウトには次の変更が含まれます：

  - 画面使用効率を最大化するために左側のナビゲーションバーを導入しました。
  - よりフラットなナビゲーション階層を採用しました。

- サーバーレスティアユーザー向けに[**接続**](/tidb-cloud/connect-to-tidb-cluster-serverless.md)エクスペリエンスを最適化しました。

  開発者は今、数回のクリックでSQLエディタに接続したり、好きなツールで接続したりすることができ、コンテキストの切り替えが必要ありません。

## 2022年12月6日

**一般変更**

- 新しい[専用ティア](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスタのデフォルトTiDBバージョンを[v6.1.2](https://docs.pingcap.com/tidb/stable/release-6.1.2)から[v6.1.3](https://docs.pingcap.com/tidb/stable/release-6.1.3)にアップグレードしました。

## 2022年11月29日

**一般変更**

- AWS MarketplaceおよびGoogle Cloud Marketplaceからのユーザーエクスペリエンスを改善しました。

  TiDB Cloudの新規ユーザーであっても、TiDB Cloudアカウントをすでに持っているユーザーであっても、AWSやGCPの課金アカウントとリンクできるようになり、AWSまたはGCP Marketplaceのサブスクリプションを簡単に完了できるようになりました。

  リンク方法については、[AWS MarketplaceまたはGoogle Cloud Marketplaceからの請求](/tidb-cloud/tidb-cloud-billing.md#billing-from-aws-marketplace-or-google-cloud-marketplace)を参照してください。

## 2022年11月22日

**一般変更**

* Amazon Aurora MySQL、Amazon Relational Database Service（RDS）MySQL、または自己ホスト型MySQL互換データベースからTiDB Cloudにデータを直接移行する機能をサポートしました（ベータ版）。

  以前は、ビジネスを一時停止してデータをオフラインでインポートするか、サードパーティツールを使用してTiDB Cloudにデータを移行する必要がありましたが、**データ移行**機能を使用すれば、TiDB Cloudコンソールでの操作を実行するだけで、最小限のダウンタイムでデータを安全にTiDB Cloudに移行できます。

  さらに、データ移行は、既存のデータとデータソースからの進行中の変更の両方を移行するための完全および増分データ移行機能を提供します。

  現在、データ移行機能は**ベータ**の状態です。専用ティアクラスタでのみ使用でき、AWSオレゴン（us-west-2）およびAWSシンガポール（ap-southeast-1）地域でのみ利用可能です。組織ごとに1つの移行ジョブを無料で作成できます。組織ごとに複数の移行ジョブを作成するには、[チケットを作成](/tidb-cloud/tidb-cloud-support.md)する必要があります。

  詳細については、[データ移行を使用してMySQL互換データベースをTiDB Cloudに移行](/tidb-cloud/migrate-from-mysql-using-data-migration.md)を参照してください。

## 2022年11月15日

**一般変更**

* [専用ティア](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスタでのポイントインタイムリカバリ（PITR）をサポートしました（ベータ版）。

  PITRは、任意の時点のデータを新しいクラスタに復元できます。これを使用して、次のことができます：

  * ダイザスタリカバリでのRPOを削減します。
    * データ書き込みエラーを解決するには、エラーイベントよりも前の時間地点を復元します。
    * ビジネスの歴史データを監査します。

  PITR機能を使用するには、TiDBクラスターバージョンが少なくともv6.3.0であること、およびTiKVノードのサイズが少なくとも8 vCPUおよび16 GiBであることを確認してください。

  バックアップデータは、デフォルトでクラスターが作成されたリージョンと同じリージョンに保存されます。日本では、PITRが有効になっているGCPでホストされているTiDBクラスターの場合、バックアップデータを1つまたは2つのリージョン（東京と/または大阪）に保存することができます。別のリージョンからデータを復元すると、より高いデータ安全性が提供され、リージョンの障害に耐えることができます。

  詳細については、[TiDBクラスターデータのバックアップとリストア](/tidb-cloud/backup-and-restore.md)を参照してください。

  この機能はまだベータ版であり、リクエストに基づいてのみ利用できます。

    * TiDB Cloudコンソールの右下隅にある **ヘルプ** をクリックします。
    * ダイアログボックスで、**説明**欄に「PITRの申請」と記入し、**送信**をクリックします。

* データベース監査ログ機能がいまGAになりました。

    データベース監査ログを使用して、ログにユーザーアクセスの詳細（実行されたSQLステートメントなど）の履歴を記録し、データベース監査ログを定期的に分析することで、データベースのセキュリティを維持するのに役立ちます。

    詳細については、[データベース監査ログ](/tidb-cloud/tidb-cloud-auditing.md)を参照してください。

## 2022年11月8日

**一般的な変更**

* ユーザーフィードバックチャンネルを改善しました。

    今後は、TiDB Cloudコンソールの **サポート** > **フィードバックを送信** で、デモやクレジットのリクエストを行うことができます。TiDB Cloudについて詳しく知りたい場合に役立ちます。

    リクエストを受け取った後、可能な限り迅速にお手伝いするために、私たちはすぐに連絡いたします。

## 2022年10月28日

**一般的な変更**

* [サーバーレスティア](/tidb-cloud/select-cluster-tier.md#tidb-serverless) が [開発者ティア](/tidb-cloud/select-cluster-tier.md#tidb-serverless) からアップグレードされました。サーバーレスティアは、TiDBのフルマネージド、自動スケーリング展開であり、現在ベータ版で無料でご利用いただけます。

    * サーバーレスティアクラスターには、完全に機能するHTAP機能が含まれています。
    * サーバーレスティアはクラスターの作成時間が高速化され、瞬時のコールドスタート時間が提供されます。開発者ティアと比較して、作成時間は分から数秒に短縮されます。
    * デプロイトポロジについて心配する必要はありません。サーバーレスティアは、リクエストに応じて自動的に調整されます。
    * セキュリティのために、サーバーレスティアではクラスターへのTLS接続が要求されます。
    * 既存の開発者ティアクラスターは、数か月以内に自動的にサーバーレスティアに移行されます。クラスターの利用に影響を与えることはありませんし、ベータ版のサーバーレスティアクラスターの使用については、請求されません。

  [こちら](/tidb-cloud/tidb-cloud-quickstart.md)で始めましょう。

## 2022年10月25日

**一般的な変更**

- サポートされているシステム変数の一部を動的に変更および永続化する機能（ベータ版）をサポートしました。

    サポートされているシステム変数に新しい値を設定するために標準のSQLステートメントを使用できます。

    ```sql
    SET [GLOBAL|SESSION] <variable>
    ```

    例：

    ```sql
    SET GLOBAL tidb_committer_concurrency = 127;
    ```

    変数が `GLOBAL` レベルで設定されている場合、その変数はクラスターに適用され、永続化されます（再起動またはサーバーの再読み込み後も有効です）。 `SESSION` レベルの変数は永続化されず、現在のセッションでのみ有効です。

    **この機能はまだベータ版であり**、サポートされている変数は限られています。その他の [システム変数](/system-variables.md) を変更することは、副作用の不確実性のためお勧めしません。TiDB v6.1に基づいてすべてのサポートされている変数のリストについては、以下のリストを参照してください：

    - [`require_secure_transport`](/system-variables.md#require_secure_transport-new-in-v610)
    - [`tidb_committer_concurrency`](/system-variables.md#tidb_committer_concurrency-new-in-v610)
    - [`tidb_enable_batch_dml`](/system-variables.md#tidb_enable_batch_dml)
    - [`tidb_enable_prepared_plan_cache`](/system-variables.md#tidb_enable_prepared_plan_cache-new-in-v610)
    - [`tidb_max_tiflash_threads`](/system-variables.md#tidb_max_tiflash_threads-new-in-v610)
    - [`tidb_mem_oom_action`](/system-variables.md#tidb_mem_oom_action-new-in-v610)
    - [`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query)
    - [`tidb_prepared_plan_cache_size`](/system-variables.md#tidb_prepared_plan_cache_size-new-in-v610)
    - [`tidb_query_log_max_len`](/system-variables.md#tidb_query_log_max_len)

- 新しい [Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスターのデフォルトのTiDBバージョンを、[v6.1.1](https://docs.pingcap.com/tidb/stable/release-6.1.1) から [v6.1.2](https://docs.pingcap.com/tidb/stable/release-6.1.2) にアップグレードしました。

## 2022年10月19日

**統合変更**

* [Vercel Integration Marketplace](https://vercel.com/integrations#databases) に [TiDB Cloud Vercel Integration](https://vercel.com/integrations/tidb-cloud) を公開しました。

    [Vercel](https://vercel.com) は、フロントエンド開発者向けのプラットフォームであり、革新者がインスピレーションの瞬間に創造するために必要な速度と信頼性を提供します。TiDB Cloud Vercel Integrationを使用すると、Vercelプロジェクトを簡単にTiDB Cloudクラスターに接続することができます。詳細については、[TiDB CloudをVercelに統合](/tidb-cloud/integrate-tidbcloud-with-vercel.md) のドキュメントを参照してください。

* [Vercelテンプレートリスト](https://vercel.com/templates) に [TiDB Cloud Starter Template](https://vercel.com/templates/next.js/tidb-cloud-starter) を公開しました。

    このテンプレートを使用して、VercelとTiDB Cloudを試すのにスタートとして利用できます。このテンプレートを使用する前に、まずTiDB Cloudクラスターにデータを[インポート](https://github.com/pingcap/tidb-prisma-vercel-demo#2-import-table-structures-and-data)する必要があります。

## 2022年10月18日

**一般的な変更**

* Dedicated Tierクラスターにおいて、TiKVまたはTiFlashノードの最小ストレージサイズが500 GiBから200 GiBに変更されました。これにより、小規模なデータボリュームのワークロードを持つユーザーにとって、よりコスト効果の高いものになります。

    詳細については、[TiKVノードストレージ](/tidb-cloud/size-your-cluster.md#tikv-node-storage) および [TiFlashノードストレージ](/tidb-cloud/size-your-cluster.md#tiflash-node-storage) を参照してください。

* カスタマイズされたTiDB Cloudサブスクリプションのオンライン契約を導入し、コンプライアンス要件を満たすためのサポートを開始しました。

    TiDB Cloudコンソールの **請求** ページに [**契約** タブ](/tidb-cloud/tidb-cloud-billing.md#contract) が追加されました。契約に同意し、契約をオンラインで処理するためのメールを受け取った場合、**契約** タブに移動して契約を確認および承諾することができます。契約について詳しく知りたい場合は、[販売担当者にお問い合わせ](https://www.pingcap.com/contact-us/) してください。

**ドキュメント変更**

* [TiDB Cloud Terraform Provider](https://registry.terraform.io/providers/tidbcloud/tidbcloud) の [ドキュメント](/tidb-cloud/terraform-tidbcloud-provider-overview.md) を追加しました。

    TiDB Cloud Terraform Providerは、クラスターやバックアップ、およびリストアなどのTiDB Cloudリソースを管理するために、[Terraform](https://www.terraform.io/) を使用できるプラグインです。リソースのプロビジョニングおよびインフラストラクチャのワークフローを簡素化する方法をお探しの場合、[ドキュメント](/tidb-cloud/terraform-tidbcloud-provider-overview.md)に従ってTiDB Cloud Terraform Providerをお試しください。

## 2022年10月11日

**一般的な変更**

* 新しい [開発者ティア](/tidb-cloud/select-cluster-tier.md#tidb-serverless) クラスターのデフォルトのTiDBバージョンを、[v6.2.0](https://docs.pingcap.com/tidb/v6.2/release-6.2.0) から [v6.3.0](https://docs.pingcap.com/tidb/v6.3/release-6.3.0) にアップグレードしました。

**コンソール変更**

* [請求明細ページ](/tidb-cloud/tidb-cloud-billing.md#billing-details) で請求情報を最適化しました：

    * **サービスごとの概要** セクションで、ノードレベルの詳細な請求情報を提供します。
    * **使用の詳細** セクションを追加しました。使用の詳細をCSVファイルとしてダウンロードすることもできます。

## 2022年9月27日

**一般的な変更**

* 招待による複数の組織への参加をサポートしました。
    + TiDB Cloudコンソールでは、参加したすべての組織を表示し、それらの間を切り替えることができます。詳細については、[組織間を切り替える](/tidb-cloud/manage-user-access.md#switch-between-organizations)を参照してください。

* [スロークエリ](/tidb-cloud/tune-performance.md#slow-query) ページをSQL診断用に追加します。

スロークエリページでは、TiDBクラスターでのすべてのスロークエリを検索して表示し、それぞれのスロークエリのボトルネックを調査するために、[実行計画](https://docs.pingcap.com/tidbcloud/explain-overview)、SQL実行情報、およびその他の詳細を表示できます。

* アカウントのパスワードをリセットすると、TiDB Cloudは新しいパスワードの入力を直近の4つのパスワードと照合し、それらのいずれかを使用しないように注意を促します。直近に使用された4つのパスワードは許可されません。

詳細については、[パスワード認証](/tidb-cloud/tidb-cloud-password-authentication.md)を参照してください。

## 2022年9月20日

**一般的な変更**

* セルフサービスユーザー向けに[cost quota-based invoice](/tidb-cloud/tidb-cloud-billing.md#invoices)を導入します。

TiDB Cloudは、コストがクォータに達すると請求書を生成します。クォータを引き上げたり、月ごとに請求書を受け取るには、[弊社の営業部門](https://www.pingcap.com/contact-us/)に連絡してください。

* ストレージ操作料金をデータバックアップコストから免除します。最新の料金情報については、[TiDB Cloudの価格詳細](https://www.pingcap.com/tidb-cloud-pricing-details/)を参照してください。

**コンソールの変更**

* データインポート用の新しいWeb UIを提供します。新しいUIでは、ユーザーエクスペリエンスが向上し、データのインポートが効率化されます。

新しいUIを使用すると、インポート対象データのプレビュー、インポートプロセスの表示、およびすべてのインポートタスクの簡単な管理ができます。

**APIの変更**

* TiDB Cloud API（ベータ版）は、すべてのユーザーに利用可能です。

TiDB CloudコンソールでAPIキーを作成することで、APIを利用することができます。詳細については、[APIドキュメント](/tidb-cloud/api-overview.md)を参照してください。

## 2022年9月15日

**一般的な変更**

* TLS経由でTiDB Cloud [Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスターに接続するサポートを追加します。

Dedicated Tierクラスターでは、[Connect](/tidb-cloud/connect-via-standard-connection.md) ダイアログの **Standard Connection** タブが、TiDBクラスターCAのダウンロードリンクを提供し、TLS接続用の接続文字列とサンプルコードを提供します。サードパーティのMySQLクライアント、MyCLI、JDBC、Python、Go、Node.jsなど様々な接続方法を使用して、Dedicated Tierクラスターに[TLS接続を行う](/tidb-cloud/connect-via-standard-connection.md)ことができます。この機能により、アプリケーションからTiDBクラスターへのデータ転送のセキュリティが確保されます。

## 2022年9月14日

**コンソールの変更**

* [Clusters](https://tidbcloud.com/console/clusters) ページとクラスターの概要ページのUIを最適化して、ユーザーエクスペリエンスを向上させます。

新しいデザインでは、Dedicated Tierへのアップグレード、クラスターへの接続、およびデータインポートのエントランスが強調表示されています。

* [Developer Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless) クラスター用のPlaygroundを導入します。

PlaygroundにはGitHubイベントの事前ロードデータセットが含まれており、データのインポートやクライアントへの接続を行わずにクエリを即座に実行してTiDB Cloudを始めることができます。

## 2022年9月13日

**一般的な変更**

* [Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスター用の新しいGoogle Cloudリージョン `N. Virginia (us-east4)` をサポートします。

## 2022年9月9日

**一般的な変更**

* データドッグにDedicated Tierクラスターの[さらなるメトリクス](/tidb-cloud/monitor-datadog-integration.md#metrics-available-to-datadog)をサポートし、クラスターのパフォーマンスステータスをよりよく理解できるようにします。

TiDB Cloudを[Datadogと統合](/tidb-cloud/monitor-datadog-integration.md)している場合、これらのメトリクスをDatadogのダッシュボードで直接表示することができます。

## 2022年9月6日

**一般的な変更**

* 新しい[Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスターのデフォルトTiDBバージョンを[v6.1.0](https://docs.pingcap.com/tidb/stable/release-6.1.0)から[v6.1.1](https://docs.pingcap.com/tidb/stable/release-6.1.1)にアップグレードします。

**コンソールの変更**

* TiDB Cloudコンソールの右上隅のエントリから、これまでPoCの申請ができるようになります。

**APIの変更**

* [TiDB Cloud API](/tidb-cloud/api-overview.md)を介してTiKVまたはTiFlashノードのストレージを増やすサポートを追加します。APIエンドポイントの`storage_size_gib`フィールドを使用してスケーリングを行うことができます。

現在、TiDB Cloud APIはベータ版であり、リクエストによってのみ利用可能です。

詳細については、[専用Tierクラスターの変更](https://docs.pingcap.com/tidbcloud/api/v1beta#tag/Cluster/operation/UpdateCluster)を参照してください。

## 2022年8月30日

**一般的な変更**

* TiDB Cloud [Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスター向けに、AWS PrivateLinkを利用したエンドポイント接続をサポートします。

このエンドポイント接続は、セキュアでプライベートであり、データをインターネットに公開することはありません。さらに、エンドポイント接続はCIDRの重複をサポートし、ネットワーク管理が容易です。

詳細については、[プライベートエンドポイント接続の設定](/tidb-cloud/set-up-private-endpoint-connections.md)を参照してください。

**コンソールの変更**

* [Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスター用の **VPC Peering** タブと **Private Endpoint** タブにおいて、MySQL、MyCLI、JDBC、Python、Go、Node.jsの接続文字列を提供します。

これにより、Dedicated Tierクラスターに対して簡単に接続コードをアプリケーションにコピー&ペーストすることができます。

## 2022年8月24日

**一般的な変更**

* Dedicated Tierクラスターを一時停止または再開するサポートを追加します。

TiDB Cloudでは、クラスターを一時停止すると、ノードの計算コストが請求されなくなります。

## 2022年8月23日

**一般的な変更**

* 新しい[Developer Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless) クラスターのデフォルトTiDBバージョンを[v6.1.0](https://docs.pingcap.com/tidb/stable/release-6.1.0)から[v6.2.0](https://docs.pingcap.com/tidb/v6.2/release-6.2.0)にアップグレードします。

**APIの変更**

* ベータ版のTiDB Cloud APIを導入します。

このAPIを使用すると、TiDB Cloudリソース（クラスターなど）を自動的に効率的に管理することができます。詳細については、[TiDB Cloud API ドキュメント](https://docs.pingcap.com/tidbcloud/api/v1beta)を参照してください。

現在、TiDB Cloud APIはベータ版であり、リクエストによってのみ利用可能です。利用希望の際は次の手順に従ってAPIアクセスを申請してください。

- [TiDB Cloud console](https://tidbcloud.com/console/clusters)の右下の **ヘルプ** をクリックします。
- ダイアログにおいて、「TiDB Cloud APIを申請」と入力し、**送信** をクリックします。

## 2022年8月16日

* TiDBとTiKVの`2 vCPU、8 GiB（ベータ版）`ノードサイズを追加します。

    * 各`2 vCPU、8 GiB（ベータ版）` TiKVノードのストレージサイズは200 GiBから500 GiBの範囲内です。

    * 推奨の使用シナリオ：
        * 中小企業向けの低ワークロードの本番環境
        * PoCおよびステージング環境
        * 開発環境

* PoCユーザー向けに[クレジット](/tidb-cloud/tidb-cloud-billing.md#credits)（以前はトライアルポイントと呼ばれていました）を導入します。

組織のクレジット情報を**クレジット**タブの**Billing**ページで確認できるようになりました。クレジットはTiDB Cloudの料金支払いに使用できます。クレジットを取得するには[お問い合わせください](https://en.pingcap.com/apply-for-poc/)。

## 2022年8月9日

* [Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスター作成用にGCP地域 `Osaka` のサポートを追加します。

## 2022年8月2日

* TiDBとTiKVの`4 vCPU、16 GiB`ノードサイズが一般提供（GA）になりました。
* 各`4 vCPU、16 GiB`のTiKVノードに対して、ストレージサイズは200 GiBから2 TiBの間です。
* 推奨される使用シナリオ：

    * SMB向けの低ワークロードの本番環境
    * PoCおよびステージング環境
    * 開発環境

* [Dedicated Tierクラスタ](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)向けに**Diagnosis**タブに[監視ページ](/tidb-cloud/built-in-monitoring.md)を追加します。

    監視ページは、全体のパフォーマンス診断のためのシステムレベルのエントリを提供します。トップダウンのパフォーマンス分析手法に基づき、監視ページはデータベース時間の分解に基づいてTiDBのパフォーマンスメトリクスを整理し、これらのメトリクスを異なる色で表示します。これらの色をチェックすることで、全体のシステムのパフォーマンスボトルネックを一目で特定でき、パフォーマンス診断時間が大幅に短縮され、パフォーマンスの分析と診断が簡素化されます。

* **Data Import**ページに、CSVおよびParquetソースファイルに対する**カスタムパターン**の有効化または無効化の切り替えを追加します。

    **カスタムパターン**機能はデフォルトで無効になっています。特定のパターンに一致するCSVまたはParquetファイルを単一のターゲットテーブルにインポートする場合に有効にできます。

    詳細については、[CSVファイルのインポート](/tidb-cloud/import-csv-files.md)と[Apache Parquetファイルのインポート](/tidb-cloud/import-parquet-files.md)を参照してください。

* TiDB Cloudサポートプラン（ベーシック、スタンダード、エンタープライズ、プレミアム）を追加して、顧客の組織のさまざまなサポートニーズに対応します。詳細については、[TiDB Cloudサポート](/tidb-cloud/tidb-cloud-support.md)を参照してください。

* [クラスタ](https://tidbcloud.com/console/clusters)ページとクラスタ詳細ページのUIを最適化します：

    * **Clusters**ページに**Connect**および**Import data**ボタンを追加します。
    * クラスタ詳細ページで**Connect**および**Import data**ボタンを右上隅に移動します。

## 2022年7月28日

* **Security Quick Start**ダイアログに**どこからでもアクセスを許可**ボタンを追加し、クラスタが任意のIPアドレスからアクセス可能になります。詳細については、[クラスタセキュリティ設定の構成](/tidb-cloud/configure-security-settings.md)を参照してください。

## 2022年7月26日

* 新しい[Developer Tierクラスタ](/tidb-cloud/select-cluster-tier.md#tidb-serverless)に自動休止と再開のサポートを追加します。

    Developer Tierクラスタは、非アクティブ状態の7日後に削除されないため、1年間の無料トライアル終了までいつでも使用できます。非アクティブ状態の24時間後、Developer Tierクラスタは自動的に休止します。クラスタを再開するには、クラスタに新しい接続を送信するか、TiDB Cloudコンソールの**再開**ボタンをクリックします。クラスタは50秒以内に自動的に再開され、サービスに復帰します。

* [Developer Tierクラスタ](/tidb-cloud/select-cluster-tier.md#tidb-serverless)にユーザー名のプレフィックス制限を追加します。

    データベースのユーザー名を使用または設定する場合、ユーザー名にクラスタのプレフィックスを含める必要があります。詳細については、[ユーザー名プレフィックス](/tidb-cloud/select-cluster-tier.md#user-name-prefix)を参照してください。

* [Developer Tierクラスタ](/tidb-cloud/select-cluster-tier.md#tidb-serverless)向けにバックアップとリストア機能を無効にします。

    Developer Tierクラスタでは、バックアップとリストア機能（自動バックアップおよび手動バックアップの両方を含む）を無効にします。データのバックアップとして[Dumpling](https://docs.pingcap.com/tidb/stable/dumpling-overview)を使用することができます。

* [Developer Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスタのストレージサイズを500 MiBから1 GiBに増加します。
* ナビゲーション体験を向上するためにTiDB Cloudコンソールにパンくずリストを追加します。
* TiDB Cloudにデータをインポートする際に複数のフィルタールールを構成できるようにします。
* **プロジェクト設定**から**トラフィックフィルタ**ページを削除し、**Connect to TiDB**ダイアログから**デフォルトセットからのルールを追加**ボタンを削除します。

## 2022年7月19日

* [TiKVノードサイズ](/tidb-cloud/size-your-cluster.md#tikv-vcpu-and-ram)に新しいオプション`8 vCPU、32 GiB`を追加します。8 vCPU TiKVノードには、`8 vCPU、32 GiB`または`8 vCPU、64 GiB`のいずれかを選択できます。
* [**Connect to TiDB**](/tidb-cloud/connect-via-standard-connection.md)ダイアログで提供されるサンプルコードのシンタックスハイライトをサポートして、コードの可読性を向上します。サンプルコードで置き換える必要があるパラメータを簡単に特定できます。
* [**Data Import Task**](/tidb-cloud/import-sample-data.md)ページで、インポートタスクを確認した後にTiDB Cloudがソースデータにアクセスできるかどうかを自動的に検証するサポートを追加します。
* TiDB Cloudコンソールのテーマカラーを[TiDB Cloudコンソール](https://en.pingcap.com/)と一貫したものに変更します。

## 2022年7月12日

* [**Data Import Task**](/tidb-cloud/import-sample-data.md)ページに**検証**ボタンを追加して、Amazon S3でデータインポートを開始する前にデータアクセスの問題を検出できるようにします。
* [**Payment Method**](/tidb-cloud/tidb-cloud-billing.md#payment-method)タブに**Billing Profile**を追加します。**Billing Profile**に税金登録番号を提供することで、請求書から特定の税金が免除される場合があります。詳細については、[請求書プロファイル情報の編集](/tidb-cloud/tidb-cloud-billing.md#edit-billing-profile-information)を参照してください。

## 2022年7月5日

* 列指向ストレージ[TiFlash](/tiflash/tiflash-overview.md)が一般提供（GA）になりました。

    - TiFlashはTiDBを基本的にハイブリッドトランザクショナル/分析処理（HTAP）データベースにします。アプリケーションデータはまずTiKVに格納され、その後Raftコンセンサスアルゴリズムを介してTiFlashにレプリケートされます。これにより、行ストレージから列指向ストレージへのリアルタイムレプリケーションが行われます。
    - TiFlashレプリカを持つテーブルでは、TiDBオプティマイザはコスト推定に基づいてTiKVレプリカまたはTiFlashレプリカのどちらを使用するかを自動的に決定します。

    TiFlashがもたらすメリットを体験するには、[TiDB Cloud HTAPクイックスタートガイド](/tidb-cloud/tidb-cloud-htap-quickstart.md)を参照してください。

* [Dedicated Tierクラスタ](/tidb-cloud/scale-tidb-cluster.md#change-storage)のTiKVおよびTiFlashのストレージサイズを増やすサポートを追加します。
* ノード・サイズフィールドにメモリ情報を表示するサポートを追加します。

## 2022年6月28日

* [TiDB v5.4.1](https://docs.pingcap.com/tidb/stable/release-5.4.1)から[TiDB v6.1.0](https://docs.pingcap.com/tidb/stable/release-6.1.0)へTiDB Cloud Dedicated Tierのアップグレードを行います。

## 2022年6月23日

* [TiDB Cloud](/tidb-cloud/size-your-cluster.md#tikv-node-storage)のTiKVの最大[ストレージ容量](/tidb-cloud/size-your-cluster.md#tikv-node-storage)を増やすサポートを追加します。

    - 8 vCPUまたは16 vCPUのTiKV：最大4 TiBのストレージ容量をサポート。
    - 4 vCPUのTiKV：最大2 TiBのストレージ容量をサポート。

## 2022年6月21日

* [Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスタ作成時にGCPリージョン`Taiwan`のサポートを追加します。
* TiDB Cloudコンソールでユーザープロファイル（名前、姓、会社名、国、電話番号）を更新できるようにするサポートを追加します。
* MySQL、MyCLI、JDBC、Python、Go、Node.jsの接続文字列を[**Connect to TiDB**](/tidb-cloud/connect-via-standard-connection.md)ダイアログで提供するサポートを追加します。
* [無料トライアル](https://tidbcloud.com/free-trial) 登録ページを追加して、TiDB Cloud に素早くサインアップできるようにします。
* プラン選択ページから **概念の証明 (PoC) プラン** オプションを削除します。14日間の無償PoCトライアルを申し込む場合は、[PoC申込](https://en.pingcap.com/apply-for-poc/) ページに移動してください。詳細については、[TiDB Cloud での概念の証明 (PoC) の実行](/tidb-cloud/tidb-cloud-poc.md)をご覧ください。
* TiDB Cloud にサインアップしたユーザーに対し、90日ごとにパスワードをリセットするよう促すことで、システムのセキュリティを向上させます。詳細については、[パスワード認証](/tidb-cloud/tidb-cloud-password-authentication.md)をご覧ください。

## 2022年5月24日

* [Dedicated Tier](/tidb-cloud/select-cluster-tier.md) クラスタを[作成](/tidb-cloud/create-tidb-cluster.md)または[復元](/tidb-cloud/backup-and-restore.md#restore)する際に TiDB ポート番号をカスタマイズできるようにサポートします。

## 2022年5月19日

* [開発者 Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless) クラスタ作成時に AWS リージョン `Frankfurt` のサポートを追加します。

## 2022年5月18日

* GitHub アカウントでTiDB Cloudに[サインアップ](https://tidbcloud.com/signup)できるようにサポートします。

## 2022年5月13日

* Google アカウントでTiDB Cloudに[サインアップ](https://tidbcloud.com/signup)できるようにサポートします。

## 2022年5月1日

* [Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスタを[作成](/tidb-cloud/create-tidb-cluster.md)または[復元](/tidb-cloud/backup-and-restore.md#restore)する際に TiDB、TiKV、TiFlashのvCPUサイズを構成できるようにサポートします。
* クラスタ作成にAWSリージョン `Mumbai` のサポートを追加します。
* [TiDB Cloud の料金設定](/tidb-cloud/tidb-cloud-billing.md)のコンピューティング、ストレージ、データ転送のコストを更新しました。

## 2022年4月7日

* 開発者 Tier を[TiDB v6.0.0](https://docs.pingcap.com/tidb/v6.0/release-6.0.0-dmr)にアップグレードしました。

## 2022年3月31日

TiDB Cloud が一般提供開始となりました。[サインアップ](https://tidbcloud.com/signup)して以下のいずれかのオプションを選択できます:

* 無料で [開発者 Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless) を利用する。
* [14日間の無料PoCトライアル](https://en.pingcap.com/apply-for-poc/)を申し込む。
* [Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) で完全なアクセスを取得する。

## 2022年3月25日

新機能:

* [TiDB Cloud 組込みアラート機能](/tidb-cloud/monitor-built-in-alerting.md)をサポートします。

    TiDB Cloud 組込みアラート機能を使用すると、プロジェクト内の TiDB Cloud クラスタが TiDB Cloud 組込みアラート条件のいずれかをトリガーしたときに、メールで通知を受け取れます。

## 2022年3月15日

一般的な変更:

* 固定されたクラスタサイズのクラスタ Tier はもう存在しません。TiDB、TiKV、TiFlashの [クラスタサイズ](/tidb-cloud/size-your-cluster.md)を簡単にカスタマイズできます。
* TiFlashのない既存のクラスタに[TiFlash](/tiflash/tiflash-overview.md)ノードを追加することがサポートされます。
* 新しいクラスタを作成する際にストレージサイズ (500から2048 GiB) を指定できるようになりました。作成後にストレージサイズを変更することはできません。
* 新しいパブリックリージョン `eu-central-1`を導入しました。
* 8 vCPU TiFlash を非推奨にし、16 vCPU TiFlash を提供します。
* CPUとストレージの価格を分けて提示しました（どちらも30%のパブリックプレビュー割引が適用されます）。
* [料金情報](/tidb-cloud/tidb-cloud-billing.md)と[価格表](https://en.pingcap.com/tidb-cloud/#pricing)を更新しました。

新機能:

* [PrometheusとGrafanaの統合](/tidb-cloud/monitor-prometheus-and-grafana-integration.md)をサポートします。

    PrometheusとGrafanaの統合を使用すると、[Prometheus](https://prometheus.io/)サービスを構成してTiDB Cloudエンドポイントから重要なメトリクスを読み取り、[Grafana](https://grafana.com/)を使用してこれらのメトリクスを表示できます。

* 選択したリージョンに基づいてデフォルトのバックアップ時間を割り当てるサポートを追加します。

    詳細については、[TiDBクラスタデータのバックアップとリストア](/tidb-cloud/backup-and-restore.md)をご覧ください。

## 2022年3月4日

新機能:

* [Datadogの統合](/tidb-cloud/monitor-datadog-integration.md)をサポートします。

    Datadogの統合を使用すると、TiDB Cloudを構成してTiDBクラスタに関するメトリックデータを[Datadog](https://www.datadoghq.com/)に送信できます。その後、これらのメトリクスをDatadogのダッシュボードで直接表示できます。

## 2022年2月15日

一般的な変更:

* 開発者 Tier を[TiDB v5.4.0](https://docs.pingcap.com/tidb/stable/release-5.4.0)にアップグレードしました。

改善:

* [CSVファイル](/tidb-cloud/import-csv-files.md)または[Apache Parquetファイル](/tidb-cloud/import-parquet-files.md)をTiDB Cloudにインポートする際に、カスタムファイル名を使用することをサポートします。

## 2022年1月11日

一般的な変更:

* TiDB Operator を[v1.2.6](https://docs.pingcap.com/tidb-in-kubernetes/stable/release-1.2.6)にアップグレードしました。

改善:

* [**Connect**](/tidb-cloud/connect-via-standard-connection.md) ページのMySQLクライアントに推奨オプション `--connect-timeout 15` を追加しました。

バグ修正:

* ユーザーがパスワードにシングルクォートを含む場合、クラスタを作成できない問題を修正しました。
* 組織に所有者が1人しかいない場合でも、所有者を削除したり別の権限に変更したりする問題を修正しました。