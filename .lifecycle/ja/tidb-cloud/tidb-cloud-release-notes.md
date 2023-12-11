---
title: 2023年のTiDB Cloudリリースノート
summary: 2023年のTiDB Cloudのリリースノートについて学びます。
aliases: ['/tidbcloud/supported-tidb-versions','/tidbcloud/release-notes']
---

# 2023年のTiDB Cloudリリースノート

このページでは、[TiDB Cloud](https://www.pingcap.com/tidb-cloud/)の2023年のリリースノートがリストされています。

## 2023年8月23日

**一般的な変更**

- [TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスターでGoogle Cloudの[Private Service Connect](https://cloud.google.com/vpc/docs/private-service-connect)をサポートします。

    これにより、Google CloudにホストされているTiDB Dedicatedクラスターにプライベートエンドポイントを作成し、安全な接続を確立できるようになります。

    主な利点:

    - 分かりやすい操作: 数ステップでプライベートエンドポイントを作成できます。
    - 強化されたセキュリティ: データを保護するために安全な接続が確立されます。
    - 改善されたパフォーマンス: 低遅延かつ高帯域幅の接続が提供されます。

  詳細については、[Google Cloudでプライベートエンドポイントを介した接続](/tidb-cloud/set-up-private-endpoint-connections-on-google-cloud.md)をご覧ください。

- [TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスターから[Google Cloud Storage (GCS)](https://cloud.google.com/storage)へデータをストリーム配信するchangefeedのサポートを追加しました。

    これにより、自分のアカウントのバケットを使用し、厳密に適合した権限を提供して、TiDB CloudからGCSにデータを複製し、その後データの変更を任意に分析できるようになります。

    詳細については、[Cloud Storageへのストリーミング](/tidb-cloud/changefeed-sink-to-cloud-storage.md)をご覧ください。

## 2023年8月15日

**一般的な変更**

- `GET`リクエストのページネーションをサポートするために[Data Service (beta)](https://tidbcloud.com/console/data-service)をサポートします。

    `GET`リクエストの場合、**Advanced Properties**にある**Pagination**を有効化し、エンドポイントを呼び出す際にクエリパラメータとして`page`および`page_size`を指定することで、結果をページ分割して取得できます。たとえば、ページあたり10項目の2番目のページを取得するには、次のコマンドを使用できます。

    ```bash
    curl --digest --user '<Public Key>:<Private Key>' \
      --request GET 'https://<region>.data.tidbcloud.com/api/v1beta/app/<App ID>/endpoint/<Endpoint Path>?page=2&page_size=10'
    ```

    この機能は、最後のクエリが `SELECT` ステートメントである `GET` リクエストにのみ利用可能です。

    詳細については、[エンドポイントの呼び出し](/tidb-cloud/data-service-manage-endpoint.md#call-an-endpoint)をご覧ください。

- `GET`リクエストのエンドポイントのレスポンスを特定の時間（TTL）までキャッシュする機能を[Data Service (beta)](https://tidbcloud.com/console/data-service)でサポートします。

    この機能により、データベースの負荷を軽減し、エンドポイントのレイテンシを最適化できます。

    `GET`リクエストメソッドを使用するエンドポイントの場合は、**Advanced Properties**で**Cache Response**を有効にし、キャッシュのTTL期間を構成できます。

    詳細については、[Advanced properties](/tidb-cloud/data-service-manage-endpoint.md#advanced-properties)をご覧ください。

- 2023年8月15日以降にAWSにホストされた[TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスターでロードバランシングの改善を無効にします。

    これにより、AWSにホストされたTiDBノードのスケールアウト時に既存の接続を新しいTiDBノードに自動的に移行する機能が無効になり、また、AWSにホストされたTiDBノードのスケールイン時に既存の接続を利用可能なTiDBノードに自動的に移行する機能が無効になります。

  この変更により、ハイブリッド展開のリソース競合を回避し、この改善が有効な既存のクラスターには影響しません。新しいクラスターでこのロードバランシングの改善を有効にしたい場合は、[TiDB Cloud サポート](/tidb-cloud/tidb-cloud-support.md)に連絡してください。

## 2023年8月8日

**一般的な変更**

- [Data Service (beta)](https://tidbcloud.com/console/data-service)でBasic認証をサポートします。

    リクエストで公開キーをユーザー名とし、プライベートキーをパスワードとして提供できます。Digest認証と比較して、Basic認証はシンプルであり、Data Serviceエンドポイントの呼び出し時により直感的に使用できます。

    詳細については、[エンドポイントの呼び出し](/tidb-cloud/data-service-manage-endpoint.md#call-an-endpoint)をご覧ください。

## 2023年8月1日

**一般的な変更**

- TiDB Cloudの[Data Service](https://tidbcloud.com/console/data-service)でData AppsのためのOpenAPI Specificationをサポートします。

    TiDB Cloud Data Serviceでは、各Data Appの自動生成されたOpenAPIドキュメントを提供します。このドキュメントでは、エンドポイント、パラメータ、応答を表示し、エンドポイントを試すことができます。

    Data Appとその展開されたエンドポイントのためのOpenAPI Specification（OAS）をYAMLまたはJSON形式でダウンロードできます。OASにより、標準化されたAPIドキュメント、簡素化された統合、容易なコード生成が可能となり、より迅速な開発と改善されたコラボレーションが実現されます。

    詳細については、[OpenAPI Specificationの使用](/tidb-cloud/data-service-manage-data-app.md#use-the-openapi-specification)と[Next.jsでのOpenAPI Specificationの使用](/tidb-cloud/data-service-oas-with-nextjs.md)をご覧ください。

- [Postman](https://www.postman.com/)でData Appを実行できるようにサポートしました。

    Postmanの統合により、Data Appのエンドポイントをワークスペースにインポートし、PostmanのWebアプリおよびデスクトップアプリの両方をサポートして、強化されたコラボレーションとシームレスなAPIテストを利用できるようになります。

    詳細については、[PostmanでData Appを実行](/tidb-cloud/data-service-postman-integration.md)をご覧ください。

- [TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスターの新しいデフォルトTiDBバージョンを[v6.5.3](https://docs.pingcap.com/tidb/v6.5/release-6.5.3)から[v7.1.1](https://docs.pingcap.com/tidb/v7.1/release-7.1.1)にアップグレードしました。

**コンソールの変更**

- [TiDB Cloudのコンソール](https://tidbcloud.com/)の下部の<MDSvgIcon name="icon-top-organization" />に「サポート」の入口を追加し、左下隅の **?** アイコンのメニューを改善して、PingCAPサポートへのアクセスを簡素化しました。

  詳細については、[TiDB Cloudサポート](/tidb-cloud/tidb-cloud-support.md)をご覧ください。

## 2023年7月26日

**一般的な変更**

- TiDB Cloud [Data Service](https://tidbcloud.com/console/data-service)でパワフルな機能である自動エンドポイント生成を追加しました。

    開発者は、最小限のクリックと構成でHTTPエンドポイントを簡単に作成できるようになりました。繰り返しのテンプレートコードを排除し、エンドポイントの作成を簡素化・加速化し、潜在的なエラーを削減します。

    この機能の使用方法については、[自動エンドポイントの生成](/tidb-cloud/data-service-manage-endpoint.md#generate-an-endpoint-automatically)をご覧ください。

- TiDB Cloud [Data Service](https://tidbcloud.com/console/data-service)のエンドポイントで`PUT`および`DELETE`のリクエストメソッドをサポートします。

    - `PUT`メソッドを使用してデータを更新または変更できます。これは`UPDATE`ステートメントに類似します。
    - `DELETE`メソッドを使用してデータを削除できます。これは`DELETE`ステートメントに類似します。

  詳細については、[プロパティの構成](/tidb-cloud/data-service-manage-endpoint.md#configure-properties)をご覧ください。

- TiDB Cloud [Data Service](https://tidbcloud.com/console/data-service)で`POST`、`PUT`、`DELETE`のリクエストメソッドの**バッチ操作**をサポートします。

    エンドポイントの**バッチ操作**が有効になっている場合、単一のリクエストで複数の行に対して操作を行うことができます。たとえば、単一の`POST`リクエストで複数のデータ行を挿入することができます。

    詳細については、[Advanced properties](/tidb-cloud/data-service-manage-endpoint.md#advanced-properties)をご覧ください。

## 2023年7月25日

**一般的な変更**

- 新しい[TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスターのデフォルトのTiDBバージョンを[v6.5.3](https://docs.pingcap.com/tidb/v6.5/release-6.5.3)から[v7.1.1](https://docs.pingcap.com/tidb/v7.1/release-7.1.1)にアップグレードしました。

**コンソールの変更**

- TiDB Cloudユーザー向けにPingCAPサポートへのアクセスを最適化するために、サポートエントリを簡略化しました。改善点は以下の通りです:

    - <MDSvgIcon name="icon-top-organization" />の下部に **Support** の入口を追加しました。
    - 左下隅の[TiDB Cloudコンソール](https://tidbcloud.com/)の **?** アイコンのメニューを直感的にしました。

  詳細については、[TiDB Cloudサポート](/tidb-cloud/tidb-cloud-support.md)をご覧ください。

## 2023年7月18日

**一般的な変更**

- 組織レベルおよびプロジェクトレベルでの役割ベースのアクセス制御を洗練化し、ユーザーに最小限の権限を付与することで、セキュリティ、コンプライアンス、および生産性向上を図ることができるようにしました。
- 組織の役割には、`Organization Owner`、`Organization Billing Admin`、`Organization Console Audit Admin`、および`Organization Member` が含まれます。
   - プロジェクトの役割には、`Project Owner`、`Project Data Access Read-Write`、および`Project Data Access Read-Only`が含まれます。
   - プロジェクトでクラスタを管理するため（クラスタの作成、変更、削除など）、 `Organization Owner`または`Project Owner`の役割である必要があります。

  異なる役割のアクセス許可の詳細については、「[ユーザーの役割](/tidb-cloud/manage-user-access.md#user-roles)」を参照してください。

- AWSでホストされる[TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスタ向けに、Customer-Managed Encryption Key (CMEK) 機能（ベータ版）をサポートしています。

    TiDB Cloudコンソールから、AWS KMSに基づくCMEKを作成して、EBSおよびS3に保存されたデータを暗号化できます。これにより、顧客データは顧客が管理するキーで暗号化されるため、セキュリティが向上します。

    この機能には、まだ制限があり、リクエストによる利用が可能です。この機能を申請するには、「[TiDB Cloud Support](/tidb-cloud/tidb-cloud-support.md)」に連絡してください。

- TiDB CloudのImport機能を最適化し、データのインポート体験を向上させることを目的として、以下の改善が行われました。:

    - TiDB Serverless向けの統合されたImportエントリー：データのインポートのエントリーを統合して、ローカルファイルとAmazon S3からのファイルインポートをシームレスに切り替えることができます。
    - 合理化された構成：Amazon S3からのデータのインポートは、1つの手順のみで済むようになり、時間と労力を節約します。
    - 拡張されたCSV構成：CSV構成設定は、ファイルタイプオプションの下に配置されるようになり、必要なパラメーターを素早く構成できるようになります。
    - 強化されたターゲットテーブル選択：チェックボックスをクリックすることで、データのインポート先の対象テーブルを選択できるようになります。この改善により、複雑な式を記述する必要がなくなり、ターゲットテーブルの選択が簡素化されます。
    - 洗練された表示情報：インポートプロセス中に表示される情報が不正確な問題を解決しました。また、不完全なデータ表示を防止し、誤解を避けるためにPreview機能を削除しました。
    - ソースファイルのマッピングの改善：ソースファイルの名前を変更して特定の命名要件を満たすようにする課題に対処するため、ソースファイルとターゲットテーブルのマッピング関係を定義することができます。

## 2023年7月11日

**一般的な変更**

- [TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless)が一般提供されました。

- ユーザーサポート向けのOpenAIパワードチャットボットであるTiDB Bot（ベータ版）を導入しました。このボットはマルチ言語対応、24時間365日のリアルタイム対応、統合されたドキュメントアクセスを提供します。

    TiDB Botは以下の利点を提供します。

    - 持続的なサポート：強化されたサポート体験のために常に利用可能で、質問に回答します。
    - 効率の向上：自動応答によりレイテンシが低減し、全体的な運用が向上します。
    - シームレスなドキュメントアクセス：簡単に情報を取得し、迅速な問題解決のために、TiDB Cloudのドキュメントに直接アクセスできます。

  TiDB Botを使用するには、[TiDB Cloudコンソール](https://tidbcloud.com)の右下隅にある **?** をクリックし、 **Ask TiDB Bot**を選択してチャットを開始してください。

- [TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスタのブランチ機能（ベータ版）をサポートしています。

    TiDB Cloudでは、TiDB Serverlessクラスタのブランチを作成できます。クラスタのブランチは、オリジナルクラスタとは異なるデータの分岐コピーを含む独立したインスタンスです。これにより、影響を気にすることなく自由に接続して実験できる、隔離された環境が提供されます。

    2023年7月5日以降に作成されたTiDB Serverlessクラスタについては、[TiDB Cloudコンソール](/tidb-cloud/branch-manage.md)または[TiDB Cloud CLI](/tidb-cloud/ticloud-branch-create.md)を使用してブランチを作成できます。
    
    アプリケーション開発にGitHubを使用している場合、GitHubのCI/CDパイプラインにTiDB Serverlessのブランチを統合することで、プロダクションデータベースに影響を与えることなく、プルリクエストの自動テストを実行できます。詳細については、「GitHubとTiDB Serverless Branching(ベータ版)の統合」をご参照ください。

- [TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスタに対する週次バックアップをサポートしています。詳細については、「[TiDB Dedicatedデータのバックアップとリストア](/tidb-cloud/backup-and-restore.md#automatic-backup)」をご参照ください。

## 2023年7月4日

**一般的な変更**

- [TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスタ向けに、ポイントインタイムリカバリ（PITR）（ベータ版）をサポートしています。

    現在、過去90日間の任意の時点までTiDB Serverlessクラスタを復元できます。この機能は、TiDB Serverlessクラスタのデータ復旧能力を向上させます。例えば、データ書き込みエラーが発生し、データを以前の状態に復元したい場合などに、PITRを使用できます。

    詳細については、「[TiDB Serverlessデータのバックアップとリストア](/tidb-cloud/backup-and-restore-serverless.md#restore)」をご参照ください。

**コンソールの変更**

- [TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスタの概要ページの **今月の利用状況**パネルを強化し、現在のリソース利用状況をより明確に表示するようにしました。

- 次の変更により、ナビゲーション体験を向上しました。

    - 右上隅の<MDSvgIcon name="icon-top-organization" /> **Organization**と<MDSvgIcon name="icon-top-account-settings" /> **Account**を左のナビゲーションバーに統合しました。
    - 左のナビゲーションバーの<svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke-width="1.5" xmlns="http://www.w3.org/2000/svg"><path d="M12 14.5H7.5C6.10444 14.5 5.40665 14.5 4.83886 14.6722C3.56045 15.06 2.56004 16.0605 2.17224 17.3389C2 17.9067 2 18.6044 2 20M14.5 6.5C14.5 8.98528 12.4853 11 10 11C7.51472 11 5.5 8.98528 5.5 6.5C5.5 4.01472 7.51472 2 10 2C12.4853 2 14.5 4.01472 14.5 6.5ZM22 16.516C22 18.7478 19.6576 20.3711 18.8054 20.8878C18.7085 20.9465 18.6601 20.9759 18.5917 20.9911C18.5387 21.003 18.4613 21.003 18.4083 20.9911C18.3399 20.9759 18.2915 20.9465 18.1946 20.8878C17.3424 20.3711 15 18.7478 15 16.516V14.3415C15 13.978 15 13.7962 15.0572 13.6399C15.1077 13.5019 15.1899 13.3788 15.2965 13.2811C15.4172 13.1706 15.5809 13.1068 15.9084 12.9791L18.2542 12C18.3452 11.9646 18.4374 11.8 18.4374 11.8H18.5626C18.5626 11.8 18.6548 11.9646 18.7458 12L21.0916 12.9791C21.4191 13.1068 21.5828 13.1706 21.7035 13.2811C21.8101 13.3788 21.8923 13.5019 21.9428 13.6399C22 13.7962 22 13.978 22 14.3415V16.516Z" stroke="currentColor" stroke-width="inherit" stroke-linecap="round" stroke-linejoin="round"></path></svg> **Admin**を左のナビゲーションバーの<MDSvgIcon name="icon-left-projects" /> **Project**に統合し、左上隅の☰ホバーメニューを削除しました。これにより、<MDSvgIcon name="icon-left-projects" />をクリックすると、プロジェクト間を切り替えてプロジェクトの設定を変更できるようになります。
    - ナビゲーションバーの **?** アイコンのメニューに、ドキュメント、インタラクティブチュートリアル、自習トレーニング、サポートエントリなど、TiDB Cloud のヘルプとサポート情報を統合しました。

- TiDB Cloudコンソールは、快適で目にやさしい体験を提供するダークモードに対応しています。左のナビゲーションバーの下部から、ライトモードとダークモードを切り替えることができます。

## 2023年6月27日
## 2023年6月20日

**一般的な変更**

- [TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless) クラスタを新規作成する際の事前に構築されたサンプルデータセットを削除しました。

## 2023年6月13日

**一般的な変更**

- 新しい [TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスタのデフォルト TiDB バージョンを [v6.5.2](https://docs.pingcap.com/tidb/v6.5/release-6.5.2) から [v6.5.3](https://docs.pingcap.com/tidb/v6.5/release-6.5.3) にアップグレードしました。

## 2023年6月6日

**一般的な変更**

- Amazon S3 へのデータストリームのための changefeeds の使用をサポートしました。

Amazon S3 へのデータストリームにより、TiDB Cloud と Amazon S3 のシームレスな統合が可能になりました。これにより、[TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスタから Amazon S3 へのリアルタイムデータのキャプチャとレプリケーションが可能となり、下流のアプリケーションや分析が最新のデータにアクセスできるようになります。

詳細については、[クラウドストレージにシンク](/tidb-cloud/changefeed-sink-to-cloud-storage.md) を参照してください。

- 16 vCPU TiKV の最大ノードストレージを [TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスタ用に 4 TiB から 6 TiB に増加しました。

この強化により、TiDB Dedicated クラスタのデータストレージ容量が増加し、ワークロードのスケーリング効率が向上し、拡大するデータ要件に対応します。

詳細については、[クラスタのサイズを決定](/tidb-cloud/size-your-cluster.md) を参照してください。

- [TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless) クラスタの [モニタリングメトリクスの保持期間](/tidb-cloud/built-in-monitoring.md#metrics-retention-policy) を 3日間から7日間に延長しました。

メトリクスの保持期間を延長することで、より多くの過去のデータにアクセスできるようになります。これにより、より良い意思決定と迅速なトラブルシューティングのために、クラスタのトレンドやパターンを特定できるようになります。

**コンソールの変更**

- [TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスタの [**Key Visualizer**](/tidb-cloud/tune-performance.md#key-visualizer) ページのための新しいナティブウェブインフラストラクチャをリリースしました。

新しいインフラストラクチャでは、**Key Visualizer** ページを簡単にナビゲートし、より直感的かつ効率的な方法で必要な情報にアクセスできます。新しいインフラストラクチャにより、UXに関する多くの問題が解決され、SQLの診断プロセスがユーザーフレンドリーになります。

## 2023年6月5日

**一般的な変更**

- [Data App](/tidb-cloud/tidb-cloud-glossary.md#data-app) を GitHub に接続する機能をサポートしました。

[Data App を GitHub に接続する](/tidb-cloud/data-service-manage-github-connection.md) ことで、Data App のすべての構成をGitHub上のコードファイルとして管理できるようになりました。これにより、TiDB Cloud Data Service をシステムアーキテクチャやDevOpsプロセスにシームレスに統合できます。

この機能により、以下のタスクを簡単に実行できるようになり、Data App のCI/CD体験が向上します：

- GitHub を使用して Data App の変更を自動的にデプロイする。
- GitHub 上で Data App の変更のCI/CDパイプラインをバージョン管理する。
- 接続された GitHub リポジトリから切断する。
- デプロイ前にエンドポイントの変更を確認する。
- デプロイの履歴を表示し、必要なアクションを取ることができる。
- コミットを再デプロイして、以前のデプロイに戻る。

詳細については、[GitHub を使用して Data App を自動的にデプロイ](/tidb-cloud/data-service-manage-github-connection.md) を参照してください。

## 2023年6月2日

**一般的な変更**

- 弊社のシンプルでわかりやすい製品名を更新しました。

    - "TiDB Cloud Serverless Tier" は "TiDB Serverless" となりました。
    - "TiDB Cloud Dedicated Tier" は "TiDB Dedicated" となりました。
    - "TiDB On-Premises" は "TiDB Self-Hosted" となりました。

    これらのリフレッシュされた名前で同じ優れたパフォーマンスをお楽しみください。お客様の体験を第一に考えています。

## 2023年5月30日

**一般的な変更**

- TiDB Cloud のデータ移行機能における増分データの移行のサポートを強化しました。

指定したバイナリログの位置またはグローバルトランザクション識別子（GTID）を使用して、指定された位置以降に生成された増分データのみを TiDB Cloud にレプリケートできるようになりました。この強化により、特定の要件に合わせて必要なデータを選択してレプリケートできるため、柔軟性が向上します。

詳細については、[MySQL 互換データベースからの増分データの TiDB Cloud へのデータ移行](/tidb-cloud/migrate-incremental-data-from-mysql-using-data-migration.md) を参照してください。

- [**Events**](/tidb-cloud/tidb-cloud-events.md) ページに新しいイベントタイプ（`ImportData`）を追加しました。

- TiDB Cloud コンソールから **Playground** を削除しました。

    最適化された体験を提供する新しいスタンドアロンの Playground にご期待ください。

## 2023年5月23日

**一般的な変更**

- CSV ファイルを TiDB にアップロードする際、英数字だけでなく中国語や日本語などの文字を使って列名を定義できるようにしました。ただし、特殊文字についてはアンダースコア（`_`）のみがサポートされます。

詳細については、[ローカルファイルを TiDB Cloud にインポート](/tidb-cloud/tidb-cloud-import-local-files.md) を参照してください。

## 2023年5月16日

**コンソールの変更**

- Dedicated および Serverless ティアの機能カテゴリによって整理された左側ナビゲーションエントリを導入しました。

新しいナビゲーションにより、機能エントリをより簡単かつ直感的に見つけることができるようになります。新しいナビゲーションを表示するには、クラスタの概要ページにアクセスします。

- Dedicated ティアクラスタの **Diagnosis** ページの以下の2つのタブに向けた新しいナティブウェブインフラストラクチャをリリースしました。

    - [Slow Query](/tidb-cloud/tune-performance.md#slow-query)
    - [SQL Statement](/tidb-cloud/tune-performance.md#statement-analysis)

新しいインフラストラクチャにより、これら2つのタブを簡単にナビゲートして必要な情報にアクセスできます。新しいインフラストラクチャはユーザーエクスペリエンスを向上し、SQL診断プロセスをユーザーフレンドリーにします。

## 2023年5月9日

**一般的な変更**

- 2023年4月26日以降に作成された GCP ホストされたクラスタのノードサイズを変更する機能をサポートしました。

この機能により、需要増加に備えてパフォーマンスの高いノードにアップグレードしたり、コスト削減のためにパフォーマンスの低いノードにダウングレードしたりすることができます。この柔軟性により、クラスタの容量をワークロードに合わせて調整し、コストを最適化することができます。

詳細な手順については、[ノードサイズの変更](/tidb-cloud/scale-tidb-cluster.md#change-vcpu-and-ram) を参照してください。
- 圧縮ファイルのインポートをサポートします。 `.gzip`、`.gz`、`.zstd`、`.zst`、および`.snappy`形式のCSVとSQLファイルをインポートできます。この機能により、データのインポートが効率的かつコスト効果が高くなり、データ転送コストを削減できます。

    詳細については、[Amazon S3またはGCSからTiDB CloudにCSVファイルをインポート](/tidb-cloud/import-csv-files.md)および[サンプルデータのインポート](/tidb-cloud/import-sample-data.md)を参照してください。

- TiDB Cloud [サーバーレスティア](/tidb-cloud/select-cluster-tier.md#tidb-serverless) クラスターの新しいネットワークアクセス管理オプションとして、AWS PrivateLinkパワードエンドポイント接続をサポートします。

    プライベートエンドポイント接続はデータをインターネットに公開しません。また、エンドポイント接続ではCIDRオーバーラップをサポートし、ネットワーク管理が容易です。

    詳細については、[プライベートエンドポイント接続の設定](/tidb-cloud/set-up-private-endpoint-connections.md)を参照してください。

**コンソール変更**

- [**イベント**](/tidb-cloud/tidb-cloud-events.md) ページに新しいイベントタイプを追加して、[専用ティア](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスターのバックアップ、復元、およびチェンジフィードアクションを記録します。

    記録できるイベントの完全なリストについては、[記録されるイベント](/tidb-cloud/tidb-cloud-events.md#logged-events)を参照してください。

- [**SQL診断**](/tidb-cloud/tune-performance.md) ページに **SQLステートメント** タブを導入して、[サーバーレスティア](/tidb-cloud/select-cluster-tier.md#tidb-serverless) クラスター向けに次の情報を提供します。

    - TiDBデータベースによって実行されたすべてのSQLステートメントの包括的な概要を提供し、遅いクエリを簡単に特定および診断できます。
    - クエリ時間、実行計画、データベースサーバーの応答など、各SQLステートメントの詳細な情報を提供し、データベースのパフォーマンスを最適化できます。
    - 大量のデータを整理、フィルタリング、検索することが容易なユーザーフレンドリーなインターフェースを提供し、最も重要なクエリに焦点を合わせることができます。

  詳細については、[ステートメントの解析](/tidb-cloud/tune-performance.md#statement-analysis)を参照してください。

## 2023年5月6日

**一般的な変更**

- TiDB [サーバーレスティア](/tidb-cloud/select-cluster-tier.md#tidb-serverless) クラスターがあるリージョンで [データサービスエンドポイント](/tidb-cloud/tidb-cloud-glossary.md#endpoint) に直接アクセスできるようにサポートします。

    新しく作成されたサーバーレスティアクラスターでは、エンドポイントURLにクラスターリージョン情報が含まれます。 `<region>.data.tidbcloud.com` のリージョン別ドメインを要求することで、TiDBクラスターがあるリージョンのエンドポイントに直接アクセスできます。

    また、リージョンを指定せずにグローバルドメイン `data.tidbcloud.com` を要求することもできます。この方法では、TiDB Cloudはリクエストを内部的に対象のリージョンにリダイレクトしますが、これには通常追加のレイテンシが発生します。この方法を選択する場合は、curlコマンドでエンドポイントを呼び出す際に `--location-trusted` オプションを追加する必要があります。

    詳細については、[エンドポイントの呼び出し](/tidb-cloud/data-service-manage-endpoint.md#call-an-endpoint)を参照してください。

## 2023年4月25日

**一般的な変更**

- 組織内の最初の5つの [サーバーレスティア](/tidb-cloud/select-cluster-tier.md#tidb-serverless) クラスターについて、それぞれ以下のような無料利用枠を提供します。

    - 行ストレージ：5 GiB
    - [リクエストユニット（RUs）](/tidb-cloud/tidb-cloud-glossary.md#request-unit)：1か月あたり5000万RUs

 2023年5月31日までサーバーレスティアクラスターは無料ですが、無料枠を超えた利用については課金されます。

    簡単に[クラスターの利用状況を監視したり、利用枠を増やしたり](/tidb-cloud/manage-serverless-spend-limit.md#manage-spending-limit-for-tidb-serverless-clusters)できます。クラスターの無料枠に達した場合、このクラスターの読み込みおよび書き込み操作は、枠を増やすか新しい月の開始時に利用状況がリセットされるまで抑制されます。

    異なるリソースのRU消費（読み込み、書き込み、SQL CPU、ネットワーク出力など）、価格の詳細、および抑制情報については、[TiDB Cloud サーバーレスティアの価格詳細](https://www.pingcap.com/tidb-cloud-serverless-pricing-details)を参照してください。

- TiDB Cloud [サーバーレスティア](/tidb-cloud/select-cluster-tier.md#tidb-serverless) クラスターのバックアップと復元をサポートします。

    詳細については、[TiDBクラスターデータのバックアップと復元](/tidb-cloud/backup-and-restore-serverless.md)を参照してください。

- 新しい [専用ティア](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスターのデフォルトのTiDBバージョンを、2023年4月25日以降に作成されたものについて、[v6.5.1](https://docs.pingcap.com/tidb/v6.5/release-6.5.1) から[v6.5.2](https://docs.pingcap.com/tidb/v6.5/release-6.5.2) にアップグレードします。

- [専用ティア](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスターの計画されたメンテナンスアクティビティを簡単にスケジュールして管理するためのメンテナンスウィンドウ機能を提供します。

    メンテナンスウィンドウは、オペレーティングシステムの更新、セキュリティパッチ、インフラストラクチャのアップグレードなどの計画されたメンテナンスアクティビティが自動で実行される指定された時間枠です。これにより、TiDB Cloudサービスの信頼性、セキュリティ、およびパフォーマンスが確保されます。

    メンテナンスウィンドウ中に一時的な接続の中断やQPSの変動が発生する場合がありますが、クラスターは利用可能であり、SQL操作、既存のデータインポート、バックアップ、リストア、マイグレーション、およびレプリケーションタスクは通常通り実行されます。メンテナンス中の[許可および不許可の操作の一覧](/tidb-cloud/configure-maintenance-window.md#allowed-and-disallowed-operations-during-a-maintenance-window)を参照してください。

    また、メンテナンスの頻度を最小限に抑えます。メンテナンスウィンドウが計画されている場合、デフォルトの開始時刻はターゲット週の水曜日03:00（TiDB Cloud組織のタイムゾーンに基づく）です。潜在的な中断を避けるためには、メンテナンススケジュールを把握し、操作を計画することが重要です。

    - TiDB Cloudでは、メンテナンスウィンドウごとにメンテナンスタスクの前、開始時、および後に3つの電子メール通知を送信します。
    - メンテナンスの影響を最小限に抑えるために、 **メンテナンス** ページでメンテナンスの開始時刻を自分の好みに変更したり、メンテナンスアクティビティを延期したりできます。

  詳細については、[メンテナンスウィンドウの構成](/tidb-cloud/configure-maintenance-window.md)を参照してください。

- AWSでホストされ、2023年4月25日以降に作成された[専用ティア](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスターのTiDBノードのスケーリング時に、TiDBの負荷分散を改善し、接続のドロップを削減する機能を提供します。

    - TiDBノードをスケールアウトする際に既存の接続を新しいTiDBノードに自動的にマイグレーションする機能をサポートします。
    - TiDBノードをスケールインする際に既存の接続を利用可能なTiDBノードに自動的にマイグレーションする機能をサポートします。

  この機能は現在、AWSでホストされるすべての専用ティアクラスターで提供されています。

**コンソール変更**

- [専用ティア](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスターの [モニタリング](/tidb-cloud/built-in-monitoring.md#view-the-metrics-page) ページの新しいネイティブウェブインフラストラクチャをリリースします。

    新しいインフラストラクチャにより、[モニタリング](/tidb-cloud/built-in-monitoring.md#view-the-metrics-page) ページを簡単にナビゲートし、より直感的かつ効率的に必要な情報にアクセスできるようになります。新しいインフラストラクチャには多くのUXの問題が解決され、モニタリングプロセスがユーザーフレンドリーになります。

## 2023年4月18日

**一般的な変更**

- [専用ティア](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスターのデータ移行ジョブ仕様のスケーリングアップまたはダウンをサポートします。

    この機能により、仕様をスケーリングアップすることで移行パフォーマンスを向上させるか、スケーリングダウンすることでコストを削減できます。

    詳細については、[データ移行を使用してMySQL互換データベースをTiDB Cloudに移行](/tidb-cloud/migrate-from-mysql-using-data-migration.md#scale-a-migration-job-specification)を参照してください。

**コンソール変更**

- [クラスター作成](https://tidbcloud.com/console/clusters/create-cluster) の体験をユーザーフレンドリーにするためにUIを刷新し、クラスターの作成と設定を簡単な手順で行えるようにします。

    この新しいデザインは、シンプルさに焦点を当て、視覚的な混乱を減らし、明確な手順を提供します。クラスター作成ページで **作成** をクリックした後、クラスターの作成が完了するのを待たずにクラスター概要ページに移動します。

    詳細については、[クラスターの作成](/tidb-cloud/create-tidb-cluster.md)を参照してください。

- 所有組織および請求管理者のための割引情報を表示する **割引** タブを **請求** ページに導入します。
    詳細については、[Discounts](/tidb-cloud/tidb-cloud-billing.md#discounts) を参照してください。

## 2023年4月11日

**一般的な変更**

- TiDB の負荷分散を改善し、TiDB ノードをスケールアウトする際の接続ドロップを軽減します。これにより、[Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスター（AWS でホストされている）の TiDB ノードをスケールアウトする際に既存の接続を新しい TiDB ノードに自動的に移行するようにサポートします。また、TiDB ノードをスケールインする際に既存の接続を利用可能な TiDB ノードに自動的に移行するようにサポートします。

  現時点では、この機能は AWS の `Oregon (us-west-2)` リージョンでホストされている Dedicated Tier クラスターにのみ提供されています。

- [Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスター向けに [New Relic](https://newrelic.com/) 連携をサポートします。

    New Relic 連携により、TiDB クラスターのメトリクスデータを [New Relic](https://newrelic.com/) に送信するように TiDB Cloud を構成できます。その後、[New Relic](https://newrelic.com/) でアプリケーションパフォーマンスと TiDB データベースのパフォーマンスをモニタリングして分析できます。この機能により、潜在的な問題を迅速に特定しトラブルシューティングするための時間短縮に役立ちます。

    連携手順や利用可能なメトリクスについては、[Integrate TiDB Cloud with New Relic](/tidb-cloud/monitor-new-relic-integration.md) を参照してください。

- Dedicated Tier クラスターの Prometheus 連携に [changefeed](/tidb-cloud/changefeed-overview.md) メトリクスを追加します。

    - `tidbcloud_changefeed_latency`
    - `tidbcloud_changefeed_replica_rows`

    [Prometheus との統合](/tidb-cloud/monitor-prometheus-and-grafana-integration.md)を行っている場合、これらのメトリクスを使用して changefeed のパフォーマンスと状態をリアルタイムでモニタリングすることができます。さらに、Prometheus を使用してメトリクスを監視するためのアラートを簡単に作成できます。

**コンソールの変更**

- [Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスターの [Monitoring](/tidb-cloud/built-in-monitoring.md#view-the-metrics-page) ページを更新し、[node-level resource metrics](/tidb-cloud/built-in-monitoring.md#server) を使用するようにします。

    ノードレベルのリソースメトリクスを使用することで、購入したサービスの実際の利用状況をより正確に把握できます。

    これらのメトリクスにアクセスするには、クラスターの [Monitoring](/tidb-cloud/built-in-monitoring.md#view-the-metrics-page) ページに移動し、**Metrics** タブの下の **Server** カテゴリを確認してください。

- [Billing](/tidb-cloud/tidb-cloud-billing.md#billing-details) ページを最適化し、**Summary by Project** と **Summary by Service** の課金項目を再構成します。これにより、課金情報をより明確に表示できるようになります。

## 2023年4月4日

**一般的な変更**

- [TiDB Cloud の組込みアラート](/tidb-cloud/monitor-built-in-alerting.md#tidb-cloud-built-in-alert-conditions) から以下の2つのアラートを削除し、誤検知を防ぎます。これは、クラスターの1つのノードが一時的にオフラインになったり、メモリ不足 (OOM) の問題が発生しても、クラスター全体の健全性に大きな影響を与えないためです。

    - クラスター内の少なくとも1つの TiDB ノードでメモリが不足しています。
    - クラスターノードの1つ以上がオフラインです。

**コンソールの変更**

- [Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスター向けの [Alerts](/tidb-cloud/monitor-built-in-alerting.md) ページを導入し、各 Dedicated Tier クラスターのアクティブおよびクローズドアラートを一覧表示します。 

    **Alerts** ページには以下の機能が提供されます。

    - 分かりやすく使いやすいユーザーインターフェース。このページでクラスターのアラートを表示できるため、アラート通知メールの購読をしていなくても問題ありません。
    - 重要度、状態などの属性に基づいてアラートを素早く見つけたり並べ替えたりするための高度なフィルターオプション。過去7日間の履歴データを表示できるため、アラート履歴の追跡を容易にします。
    - **Edit Rule** 機能。アラートルール設定をカスタマイズしてクラスター固有のニーズに合わせることができます。

    詳細については、[TiDB Cloud built-in alerts](/tidb-cloud/monitor-built-in-alerting.md) を参照してください。

- TiDB Cloud 関連のヘルプ情報とアクションをまとめたページを追加します。

    これにより、[TiDB Cloud のヘルプ情報](/tidb-cloud/tidb-cloud-support.md) をまとめて参照し、[TiDB Cloud コンソール](https://tidbcloud.com/) の右下の **?** をクリックすることでサポートに連絡できるようになります。

- [Getting Started](https://tidbcloud.com/console/getting-started) ページを追加して、TiDB Cloud の学習を支援します。

    **Getting Started** ページでは、インタラクティブなチュートリアル、必須ガイド、有用なリンクが提供されます。インタラクティブなチュートリアルに従って、事前に構築された業界固有のデータセット（Steam Game Dataset および S&P 500 Dataset）を使用して、TiDB Cloud の機能と HTAP 機能を簡単に探索できます。

    **Getting Started** ページにアクセスするには、[TiDB Cloud コンソール](https://tidbcloud.com/) の左側のナビゲーションバーで <svg width="20" height="20" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M12 14.9998L9 11.9998M12 14.9998C13.3968 14.4685 14.7369 13.7985 16 12.9998M12 14.9998V19.9998C12 19.9998 15.03 19.4498 16 17.9998C17.08 16.3798 16 12.9998 16 12.9998M9 11.9998C9.53214 10.6192 10.2022 9.29582 11 8.04976C12.1652 6.18675 13.7876 4.65281 15.713 3.59385C17.6384 2.53489 19.8027 1.98613 22 1.99976C22 4.71976 21.22 9.49976 16 12.9998M9 11.9998H4C4 11.9998 4.55 8.96976 6 7.99976C7.62 6.91976 11 7.99976 11 7.99976M4.5 16.4998C3 17.7598 2.5 21.4998 2.5 21.4998C2.5 21.4998 6.24 20.9998 7.5 19.4998C8.21 18.6598 8.2 17.3698 7.41 16.5898C7.02131 16.2188 6.50929 16.0044 5.97223 15.9878C5.43516 15.9712 4.91088 16.1535 4.5 16.4998Z" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"></path></svg> **Getting Started** をクリックします。このページでは、**Query Sample Dataset** をクリックしてインタラクティブなチュートリアルを開いたり、TiDB Cloud を探索するための他のリンクをクリックしたりすることができます。また、右下の **?** をクリックして **Interactive Tutorials** をクリックすることもできます。

## 2023年3月29日

**一般的な変更**

- [Data Service (beta)](/tidb-cloud/data-service-overview.md) は、Data App に対してより細かなアクセス制御をサポートします。

    Data App の詳細ページで、クラスターを Data App にリンクさせ、各 API キーの役割を指定できるようになりました。役割は、API キーがリンクされたクラスターに対してデータを読み取るか書き込むかを制御し、「ReadOnly」または「ReadAndWrite」に設定できます。この機能により、Data Apps のためにクラスターレベルと権限レベルのアクセス制御を提供し、ビジネスのニーズに応じてアクセス範囲を柔軟に制御できるようになります。

    詳細については、[Manage linked clusters](/tidb-cloud/data-service-manage-data-app.md#manage-linked-data-sources) および [Manage API keys](/tidb-cloud/data-service-api-key.md) を参照してください。

## 2023年3月28日

**一般的な変更**

- [changefeed](/tidb-cloud/changefeed-overview.md) に 2 RCUs、4 RCUs、および 8 RCUs の仕様を追加し、[changefeed を作成](/tidb-cloud/changefeed-overview.md#create-a-changefeed) する際に希望する仕様を選択できるようにサポートします。 

    これらの新しい仕様を使用することで、16 RCUs が以前必要だったシナリオと比較して、データレプリケーションコストを最大 87.5% 削減することができます。

- 2023年3月28日以降に作成された [changefeed](/tidb-cloud/changefeed-overview.md) の仕様をスケールアップまたはスケールダウンすることをサポートします。
    リプリケーションの性能を向上させるには、より高い仕様を選択するか、リプリケーションコストを削減するためにより低い仕様を選択することができます。

詳細については、[Changefeedのスケーリング](/tidb-cloud/changefeed-overview.md#scale-a-changefeed)を参照してください。

- [AWS](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)の[Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスタから同じプロジェクトおよび同じリージョン内の[Serverless Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスタにリアルタイムで増分データをレプリケーションするサポート。

詳細については、[TiDB Cloudへのシンク](/tidb-cloud/changefeed-sink-to-tidb-cloud.md)を参照してください。

- [Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスタの[データ移行](/tidb-cloud/migrate-from-mysql-using-data-migration.md)機能のために、新しいGCPリージョンをサポート: `Singapore (asia-southeast1)` および `Oregon (us-west1)`。

これらの新しいリージョンにより、データをTiDB Cloudに移行するためのオプションが増えます。上流データがこれらのリージョン内または近辺に格納されている場合、GCPからTiDB Cloudへのデータ移行をより高速かつ信頼性の高いものにすることができます。

詳細については、[データ移行を使用したMySQL互換データベースのTiDB Cloudへの移行](/tidb-cloud/migrate-from-mysql-using-data-migration.md)を参照してください。

**コンソールの変更**

- [Serverless Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスタの[遅いクエリ](/tidb-cloud/tune-performance.md#slow-query)ページに新しいネイティブWebインフラストラクチャをリリースしました。

この新しいインフラストラクチャにより、[遅いクエリ](/tidb-cloud/tune-performance.md#slow-query)ページを簡単にナビゲートし、直感的かつ効率的な方法で必要な情報にアクセスすることができます。新しいインフラストラクチャにより、UXに関する多くの問題が解決され、SQL診断プロセスがユーザーフレンドリーになりました。

## 2023年3月21日

**一般的な変更**

- [Serverless Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスタ用に[Data Service (beta)](https://tidbcloud.com/console/data-service)を導入しました。これにより、カスタムAPIエンドポイントを使用してHTTPSリクエストでデータにアクセスすることができるようになります。

Data Serviceを使用すると、HTTPSに対応している任意のアプリケーションやサービスとTiDB Cloudをシームレスに統合することができます。以下は一般的な使用シナリオのいくつかです:

- モバイルアプリケーションやWebアプリケーションからTiDBクラスタのデータに直接アクセスする。
- サーバーレスのエッジ関数を使用してエンドポイントを呼び出し、データベース接続プーリングによる拡張性の問題を回避する。
- Data Serviceをデータソースとして使用してデータ可視化プロジェクトをTiDB Cloudに統合する。
- MySQLインターフェースがサポートしていない環境からデータベースに接続する。

さらに、TiDB CloudではChat2Query APIの提供も行っており、これはRESTfulインターフェースを介してAIを使用してSQLステートメントを生成および実行することができます。

Data Serviceにアクセスするには、左側のナビゲーションペインの[**Data Service**](https://tidbcloud.com/console/data-service)ページに移動します。詳細については、次のドキュメントを参照してください:

- [Data Serviceの概要](/tidb-cloud/data-service-overview.md)
- [Data Serviceのはじめ方](/tidb-cloud/data-service-get-started.md)
- [Chat2Query APIのはじめ方](/tidb-cloud/use-chat2query-api.md)

- [AWS](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)で作成された2022年12月31日以降の[Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスタのTiDB、TiKV、TiFlashノードのサイズを縮小する機能をサポート。

TiDB Cloudコンソールを介してノードサイズを縮小したり、TiDB Cloud API (beta)を介してノードサイズを変更したりすることができます。

- [Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスタの[データ移行](/tidb-cloud/migrate-from-mysql-using-data-migration.md)機能のために、新しいGCPリージョンをサポート: `Tokyo (asia-northeast1)`。

この機能を使用することで、Google Cloud Platform（GCP）上のMySQL互換データベースからTiDBクラスタにデータを容易かつ効率的に移行することができます。

詳細については、[データ移行を使用したMySQL互換データベースのTiDB Cloudへの移行](/tidb-cloud/migrate-from-mysql-using-data-migration.md)を参照してください。

**コンソールの変更**

- [Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスタに**Events**ページを追加し、クラスタに関する主な変更の記録を提供しています。

このページでは、過去7日間のイベント履歴を表示し、クラスタの一時停止やクラスタサイズの変更などの重要な詳細を追跡することができます。例えば、クラスタが一時停止されたときやクラスタサイズを変更したユーザーなどのイベントを表示することができます。

詳細については、[TiDB Cloudクラスタのイベント](/tidb-cloud/tidb-cloud-events.md)を参照してください。

- [Serverless Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスタの**Monitoring**ページに**Database Status**タブを追加し、以下のデータベースレベルのメトリクスを表示します:

    - DBごとのQPS
    - DBごとの平均クエリ時間
    - 失敗したクエリ数（DBごと）

これらのメトリクスを使用することで、個々のデータベースのパフォーマンスをモニタリングし、データに基づいた意思決定を行い、アプリケーションのパフォーマンスを向上させるためのアクションを取ることができます。

詳細については、[Serverless Tierクラスタの監視メトリクス](/tidb-cloud/built-in-monitoring.md)を参照してください。

## 2023年3月14日

**一般的な変更**

- 新しい[Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスタのデフォルトのTiDBバージョンを[v6.5.0](https://docs.pingcap.com/tidb/v6.5/release-6.5.0)から[v6.5.1](https://docs.pingcap.com/tidb/v6.5/release-6.5.1)にアップグレードしました。

- [Serverless Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスタにローカルCSVファイルをアップロードする際にTiDB Cloudによって作成されるターゲットテーブルの列名を変更する機能をサポートするようになりました。

[Serverless Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスタにローカルCSVファイルをヘッダー行付きでインポートする際、ヘッダー行の列名がTiDB Cloudの列命名規則に従っていない場合、該当する列名の横に警告アイコンが表示されます。警告を解消するためには、アイコンにカーソルを移動してメッセージに従って既存の列名を編集するか新しい列名を入力してください。

列の命名規則の詳細については、[ローカルファイルのインポート](/tidb-cloud/tidb-cloud-import-local-files.md#import-local-files)を参照してください。

## 2023年3月7日

**一般的な変更**

- [Serverless Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスタのデフォルトのTiDBバージョンを[v6.4.0](https://docs.pingcap.com/tidb/v6.4/release-6.4.0)から[v6.6.0](https://docs.pingcap.com/tidb/v6.6/release-6.6.0)にアップグレードしました。

## 2023年2月28日

**一般的な変更**

- [Serverless Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスタ用に[SQL Diagnosis](/tidb-cloud/tune-performance.md)機能を追加しました。

SQL Diagnosisを使用すると、SQL関連のランタイムステータスについて深い洞察を得ることができ、SQLパフォーマンスチューニングをより効率的に行うことができます。ただし、現在Serverless TierのSQL Diagnosis機能は遅いクエリデータのみを提供しています。

SQL Diagnosisを使用するには、Serverless Tierクラスタページの左側のナビゲーションバーで **SQL Diagnosis** をクリックしてください。

**コンソールの変更**

- 左側のナビゲーションを最適化しました。

これにより、以下のようなページ間を効率的に移動することができます:

- 左上隅にマウスを置いてクラスタやプロジェクト間を素早く切り替えることができます。
- **Clusters**ページと**Admin**ページを切り替えることができます。

**APIの変更**

- データのインポートに関するTiDB Cloud APIエンドポイントをいくつかリリースしました：

    - すべてのインポートタスクをリストする
    - インポートタスクを取得する
    - インポートタスクを作成する
    - インポートタスクを更新する
    - インポートタスクのためにローカルファイルをアップロードする
    - インポートタスクを開始する前にデータをプレビューする
    - インポートタスクのロール情報を取得する

詳細については、[APIドキュメント](https://docs.pingcap.com/tidbcloud/api/v1beta#tag/Import)を参照してください。

## 2023年2月22日

**一般的な変更**

- [TiDB Cloudコンソール](https://tidbcloud.com/)での組織内のメンバーによる様々なアクティビティを追跡するための[コンソール監査ログ](/tidb-cloud/tidb-cloud-console-auditing.md)機能をサポートするようになりました。
コンソール監査ログ機能は、所有者または監査管理者ロールのユーザーにのみ表示され、デフォルトでは無効になっています。有効にするには、[TiDB Cloudコンソール](https://tidbcloud.com/)の右上隅にある<MDSvgIcon name="icon-top-organization" /> **組織** > **コンソール監査ログ**をクリックしてください。

コンソール監査ログを分析することで、組織内で実行される不審な操作を特定し、組織のリソースとデータのセキュリティを向上させることができます。

詳細については、[コンソール監査ログ](/tidb-cloud/tidb-cloud-console-auditing.md)を参照してください。

**CLIの変更**

- [TiDB Cloud CLI](/tidb-cloud/cli-reference.md)の新しいコマンド[`ticloud cluster connect-info`](/tidb-cloud/ticloud-cluster-connect-info.md)を追加しました。

    `ticloud cluster connect-info`は、クラスターの接続文字列を取得するコマンドです。このコマンドを使用するには、[`ticloud`](/tidb-cloud/ticloud-update.md)をv0.3.2またはそれ以降のバージョンにアップデートしてください。

## 2023年2月21日

**一般的な変更**

- TiDB Cloudにデータをインポートする際、AWSのIAMユーザーのアクセスキーを使用するサポートを追加しました。

    この方法はRole ARNを使用する方法よりも簡単です。詳細については、[Amazon S3アクセスの構成](/tidb-cloud/config-s3-and-gcs-access.md#configure-amazon-s3-access)を参照してください。

- [モニタリングメトリクスの保持期間](/tidb-cloud/built-in-monitoring.md#metrics-retention-policy)を2日から次の期間に延長しました：

    - Dedicated Tierクラスターの場合、過去7日間のメトリクスデータを表示できます。
    - Serverless Tierクラスターの場合、過去3日間のメトリクスデータを表示できます。

  メトリクスの保持期間を延長することで、より多くの過去のデータにアクセスできるようになります。これによって、よりよい意思決定と迅速なトラブルシューティングのために、クラスターのトレンドとパターンを特定することができます。

**コンソールの変更**

- [Serverless Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスターのモニタリングページに新しいネイティブWebインフラストラクチャをリリースしました。

    新しいインフラストラクチャにより、モニタリングページを簡単にナビゲートし、より直感的で効率的な方法で必要な情報にアクセスできます。新しいインフラストラクチャはUXに関する多くの問題を解決し、モニタリングプロセスをはるかにユーザーフレンドリーにします。

## 2023年2月17日

**CLIの変更**

- [TiDB Cloud CLI](/tidb-cloud/cli-reference.md)に新しいコマンド[`ticloud connect`](/tidb-cloud/ticloud-connect.md)を追加しました。

    `ticloud connect`は、SQLクライアントをインストールせずにローカルマシンからTiDB Cloudクラスターに接続するコマンドです。TiDB Cloudクラスターに接続した後は、TiDB Cloud CLIでSQLステートメントを実行できます。

## 2023年2月14日

**一般的な変更**

- TiDB [Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスターのTiKVおよびTiFlashノードの数を減らすことをサポートしました。

    ノード数を減らす方法については、[TiDB Cloudコンソール](/tidb-cloud/scale-tidb-cluster.md#change-node-number)または[TiDB Cloud API（ベータ）](https://docs.pingcap.com/tidbcloud/api/v1beta#tag/Cluster/operation/UpdateCluster)を参照してください。

**コンソールの変更**

- [Serverless Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスター用の**モニタリング**ページを導入しました。

    **モニタリング**ページでは、秒ごとに実行されたSQLステートメントの数、クエリの平均所要時間、失敗したクエリの数など、さまざまなメトリクスとデータが提供され、Serverless Tierクラスター内のSQLステートメントの全体的なパフォーマンスをよりよく理解できます。

    詳細については、[TiDB Cloud組込みモニタリング](/tidb-cloud/built-in-monitoring.md)を参照してください。

## 2023年2月2日

**CLIの変更**

- TiDB Cloud CLIクライアント[`ticloud`](/tidb-cloud/cli-reference.md)を導入しました。

    `ticloud`を使用すると、わずか数行のコマンドでターミナルや他の自動ワークフローからTiDB Cloudリソースを簡単に管理できます。特にGitHub Actions向けに、[`setup-tidbcloud-cli`](https://github.com/marketplace/actions/set-up-tidbcloud-cli)を提供しており、簡単に`ticloud`をセットアップできます。

    詳細については、[TiDB Cloud CLIクイックスタート](/tidb-cloud/get-started-with-cli.md)と[TiDB Cloud CLIリファレンス](/tidb-cloud/cli-reference.md)を参照してください。

## 2023年1月18日

**一般的な変更**

* マイクロソフトアカウントで[TiDB Cloudにサインアップ](https://tidbcloud.com/free-trial)することをサポートしました。

## 2023年1月17日

**一般的な変更**

- 新しい[Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスターのデフォルトのTiDBバージョンを[v6.1.3](https://docs.pingcap.com/tidb/stable/release-6.1.3)から[v6.5.0](https://docs.pingcap.com/tidb/stable/release-6.5.0)にアップグレードしました。

- 新規登録ユーザーに対して、無料の[Serverless Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスターを自動的に作成するようにしました。これにより、TiDB Cloudでのデータ探索の旅を迅速に開始できます。

- [Dedicated Tier](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスター用に新しいAWSリージョン`Seoul (ap-northeast-2)`をサポートしました。

    このリージョンでは、以下の機能が有効になります：

    - [データマイグレーションを使用してMySQL互換データベースをTiDB Cloudに移行](/tidb-cloud/migrate-from-mysql-using-data-migration.md)
    - [changefeedを使用してTiDB Cloudから他のデータサービスにデータをストリーム配信](/tidb-cloud/changefeed-overview.md)
    - TiDBクラスターデータのバックアップとリストア](/tidb-cloud/backup-and-restore.md)

## 2023年1月10日

**一般的な変更**

- [Serverless Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスター向けに、ローカルのCSVファイルからのデータインポート機能を最適化し、ユーザーエクスペリエンスを向上させました。

    - CSVファイルをアップロードする際、アップロードエリアにドラッグアンドドロップするだけでよくなりました。
    - インポートタスクを作成する際、対象のデータベースやテーブルが存在しない場合、名前を入力してTiDB Cloudに自動作成させることができます。対象のテーブルが作成されるために、主キーを指定したり、複数のフィールドを選択して複合主キーを形成したりできます。
    - インポートが完了した後、タスクリスト内で[AIパワードのChat2Query](/tidb-cloud/explore-data-with-chat2query.md)を使用してデータを探索することができます。

  詳細については、[ローカルファイルをTiDB Cloudにインポート](/tidb-cloud/tidb-cloud-import-local-files.md)を参照してください。

**コンソールの変更**

- 各クラスターに**サポートを取得**オプションを追加し、特定のクラスターに対するサポートの要求プロセスを簡素化しました。

    クラスターのサポートをリクエストする方法は次のいずれかです：

    - プロジェクトの[**クラスター**](https://tidbcloud.com/console/clusters)ページで、クラスターの行で**...**をクリックして**サポートを取得**を選択します。
    - クラスターの概要ページで、右上隅の**...**をクリックして**サポートを取得**を選択します。

## 2023年1月5日

**コンソールの変更**

- [Serverless Tier](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスター向けにSQLエディター（ベータ）をChat2Query（ベータ）に名称変更し、AIを使用したSQLクエリの生成をサポートしました。

  Chat2Queryでは、AIにSQLクエリを自動生成させるか、手動でSQLクエリを書いて、ターミナルなしでデータベースに対してSQLクエリを実行することができます。

  Chat2Queryにアクセスするには、[**クラスター**](https://tidbcloud.com/console/clusters)ページに移動し、クラスター名をクリックし、左側のナビゲーションペインで**Chat2Query**をクリックします。

## 2023年1月4日

**一般的な変更**

- AWS上でホストされ、2022年12月31日以降に作成されたTiDB DedicatedクラスターのTiDB、TiKV、およびTiFlashノードのサイズを増やすことをサポートしました。

    ノードサイズを増やす方法については、[TiDB Cloudコンソール](/tidb-cloud/scale-tidb-cluster.md#change-vcpu-and-ram)または[TiDB Cloud API（ベータ）](https://docs.pingcap.com/tidbcloud/api/v1beta#tag/Cluster/operation/UpdateCluster)を参照してください。

- [**モニタリング**](/tidb-cloud/built-in-monitoring.md)ページ上のメトリクス保持期間を2日に延長しました。

    現在、過去2日間のメトリクスデータにアクセスできるため、クラスターのパフォーマンスやトレンドをより柔軟に把握できます。
この改善は追加費用なしで利用でき、クラスターの**診断**タブからアクセスできます。これにより、パフォーマンスの問題を特定してトラブルシューティングし、クラスター全体の健全性を効果的に監視できます。

- Prometheus インテグレーションのための Grafana ダッシュボード JSON のカスタマイズをサポート。

    [TiDB Cloud を Prometheus と統合](/tidb-cloud/monitor-prometheus-and-grafana-integration.md) した場合、事前に作成された Grafana ダッシュボードをインポートし、TiDB Cloud クラスターを監視するためのダッシュボードをカスタマイズできるようになります。この機能により、TiDB Cloud クラスターを簡単かつ迅速に監視でき、パフォーマンスの問題を迅速に特定できます。

    詳細は[メトリクスを視覚化するための Grafana GUI ダッシュボードの使用](/tidb-cloud/monitor-prometheus-and-grafana-integration.md#step-3-use-grafana-gui-dashboards-to-visualize-the-metrics)をご覧ください。

- [サーバーレスティア](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスターのデフォルトの TiDB バージョンを全て[v6.3.0](https://docs.pingcap.com/tidb/v6.3/release-6.3.0) から [v6.4.0](https://docs.pingcap.com/tidb/v6.4/release-6.4.0)にアップグレードします。また、v6.4.0にアップグレードした後のサーバーレスティアクラスターの起動遅延問題が解決されました。

**コンソールの変更**

- [**クラスター**](https://tidbcloud.com/console/clusters) ページとクラスターの概要ページの表示をシンプルにしました。

    - [**クラスター**](https://tidbcloud.com/console/clusters) ページでクラスター名をクリックして、クラスターの概要ページに移動してクラスターの操作を開始できます。
    - クラスターの概要ページから**接続**および**インポート**パネルを削除しました。上部右隅の**接続**をクリックして接続情報を取得し、左側のナビゲーションペインで**インポート**をクリックしてデータをインポートできます。