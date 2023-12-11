---
title: JetBrains DataGripを使用してTiDBに接続する
summary: JetBrains DataGripを使用してTiDBに接続する方法を学びます。このチュートリアルは、IntelliJ、PhpStorm、PyCharmなどの他のJetBrains IDEにも使用できるDatabase ToolsとSQLプラグインにも適用されます。
---

# JetBrains DataGripを使用してTiDBに接続する

TiDBはMySQL互換のデータベースです。[JetBrains DataGrip](https://www.jetbrains.com/help/datagrip/getting-started.html)は、データベースとSQLのための強力な統合開発環境（IDE）です。このチュートリアルでは、DataGripを使用してTiDBクラスタに接続する手順を説明します。

> **注記:**
>
> このチュートリアルはTiDB Serverless、TiDB Dedicated、TiDB Self-Hostedに対応しています。

DataGripは2つの方法で使用できます:

- [DataGrip IDE](https://www.jetbrains.com/datagrip/download)として独立したツールとして。
- IntelliJ、PhpStorm、PyCharmなどのJetBrains IDEに含まれる[Database ToolsとSQLプラグイン](https://www.jetbrains.com/help/idea/relational-databases.html)として。

このチュートリアルは主に独立したDataGrip IDEに焦点を当てています。JetBrains IDEに含まれるDatabase ToolsとSQLプラグインを使用したTiDBへの接続手順も類似しています。また、このドキュメントの手順は、任意のJetBrains IDEからTiDBに接続する際の参考としても利用できます。

## 前提条件

このチュートリアルを完了するには、以下が必要です:

- [DataGrip **2023.2.1** 以降](https://www.jetbrains.com/datagrip/download/)または非コミュニティエディションの[JetBrains](https://www.jetbrains.com/) IDE。
- TiDBクラスタ。

<CustomContent platform="tidb">

**TiDBクラスタをお持ちでない場合は、以下の手順で作成できます:**

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)を参照して、TiDB Cloudクラスタを作成します。
- ローカルクラスタを作成するには、[ローカルテストTiDBクラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番TiDBクラスタのデプロイ](/production-deployment-using-tiup.md)に従います。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスタをお持ちでない場合は、以下の手順で作成できます:**

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)を参照して、TiDB Cloudクラスタを作成します。
- ローカルクラスタを作成するには、[ローカルテストTiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番TiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従います。

</CustomContent>

## TiDBに接続する

選択したTiDBのデプロイオプションに応じてTiDBクラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**クラスタ**](https://tidbcloud.com/console/clusters) ページに移動し、対象のクラスタの名前をクリックしてその概要ページに移動します。

2. 右上隅の**接続**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログ内の構成が実行環境と一致することを確認します。

    - **エンドポイントタイプ**が `Public` に設定されていること
    - **Connect With** が `JDBC` に設定されていること
    - **オペレーティングシステム** が環境に一致していること

4. ランダムなパスワードを作成するには、**パスワードを作成** をクリックします。

    > **ヒント:**
    >
    > 以前にパスワードを作成している場合は、元のパスワードを使用するか、**パスワードをリセット** をクリックして新しいパスワードを生成できます。

5. DataGripを起動し、接続を管理するプロジェクトを作成します。

    ![DataGripでプロジェクトを作成](/media/develop/datagrip-create-project.jpg)

6. 新しく作成したプロジェクトで、**データベースエクスプローラ** パネルの左上隅の **+** をクリックし、 **データソース** > **その他** > **TiDB** を選択します。

    ![DataGripでデータソースを選択](/media/develop/datagrip-data-source-select.jpg)

7. TiDB Cloud接続ダイアログからJDBCストリングをコピーし、`<your_password>` を実際のパスワードに置き換えます。その後、**URL** フィールドに貼り付けると残りのパラメータが自動で入力されます。例を以下に示します:

    ![TiDB ServerlessのURLフィールドの構成](/media/develop/datagrip-url-paste.jpg)

    **ドライバファイルのダウンロードが不足しています** という警告が表示された場合は、**ダウンロード** をクリックしてドライバファイルを取得します。

8. 接続をTiDB Serverlessクラスタに検証するには、**接続をテスト** をクリックします。

    ![TiDB Serverlessクラスタへの接続をテスト](/media/develop/datagrip-test-connection.jpg)

9. 接続の構成を保存するには、**OK** をクリックします。

</div>
<div label="TiDB Dedicated">

1. [**クラスタ**](https://tidbcloud.com/console/clusters) ページに移動し、対象のクラスタの名前をクリックしてその概要ページに移動します。

2. 右上隅の**接続**をクリックします。接続ダイアログが表示されます。

3. **いつでもアクセスを許可** をクリックし、その後 **TiDBクラスタCAをダウンロード** をクリックしてCA証明書をダウンロードします。

    接続文字列の取得方法の詳細については、[TiDB Dedicated標準接続](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4. DataGripを起動し、接続を管理するプロジェクトを作成します。

    ![DataGripでプロジェクトを作成](/media/develop/datagrip-create-project.jpg)

5. 新しく作成したプロジェクトで、**データベースエクスプローラ** パネルの左上隅の **+** をクリックし、 **データソース** > **その他** > **TiDB** を選択します。

    ![DataGripでデータソースを選択](/media/develop/datagrip-data-source-select.jpg)

6. 適切な接続文字列を **DataGripのデータソースとドライバ** ウィンドウにコピーして貼り付けます。DataGripのフィールドとTiDB専用接続文字列の対応は次のとおりです:

    | DataGripフィールド | TiDB専用接続文字列 |
    | -------------- | ------------------------------- |
    | ホスト           | `{host}`                        |
    | ポート           | `{port}`                        |
    | ユーザ           | `{user}`                        |
    | パスワード       | `{password}`                    |

    以下に例を示します:

    ![TiDB Dedicatedの接続パラメータの構成](/media/develop/datagrip-dedicated-connect.jpg)

7. **SSH/SSL** タブをクリックし、**SSLを使用** チェックボックスを選択し、CA証明書のパスを **CAファイル** フィールドに入力します。

    ![TiDB DedicatedのCAの構成](/media/develop/datagrip-dedicated-ssl.jpg)

    **ドライバファイルのダウンロードが不足しています** という警告が表示された場合は、**ダウンロード** をクリックしてドライバファイルを取得します。

8. **詳細** タブをクリックし、**enabledTLSProtocols** パラメータを探し、その値を `TLSv1.2,TLSv1.3` に設定します。

    ![TiDB DedicatedのTLSの構成](/media/develop/datagrip-dedicated-advanced.jpg)

9. 接続をTiDB Dedicatedクラスタに検証するには、**接続をテスト** をクリックします。

    ![TiDB Dedicatedクラスタへの接続をテスト](/media/develop/datagrip-dedicated-test-connection.jpg)

10. 接続の構成を保存するには、**OK** をクリックします。

</div>
<div label="TiDB Self-Hosted">

1. DataGripを起動し、接続を管理するプロジェクトを作成します。

    ![DataGripでプロジェクトを作成](/media/develop/datagrip-create-project.jpg)

2. 新しく作成したプロジェクトで、**データベースエクスプローラ** パネルの左上隅の **+** をクリックし、 **データソース** > **その他** > **TiDB** を選択します。

    ![DataGripでデータソースを選択](/media/develop/datagrip-data-source-select.jpg)

3. 以下の接続パラメータを構成します:

    - **ホスト**: TiDB Self-HostedクラスタのIPアドレスまたはドメイン名
    - **ポート**: TiDB Self-Hostedクラスタのポート番号
    - **ユーザ**: TiDB Self-Hostedクラスタへの接続に使用するユーザー名
    - **パスワード**: ユーザーのパスワード

    以下に例を示します:

    ![TiDB Self-Hostedの接続パラメータの構成](/media/develop/datagrip-self-hosted-connect.jpg)

    **ドライバファイルのダウンロードが不足しています** という警告が表示された場合は、**ダウンロード** をクリックしてドライバファイルを取得します。

4. 接続をTiDB Self-Hostedクラスタに検証するには、**接続をテスト** をクリックします。

    ![TiDB Self-Hostedクラスタへの接続をテスト](/media/develop/datagrip-self-hosted-test-connection.jpg)

5. 接続の構成を保存するには、**OK** をクリックします。

</div>
</SimpleTab>

## 次のステップ

- [DataGripのドキュメント](https://www.jetbrains.com/help/datagrip/getting-started.html)からDataGripの詳細な使い方を学んでください。
- [開発者ガイド](/develop/dev-guide-overview.md) の章を通じて、TiDBアプリケーション開発のベストプラクティスを学びましょう。[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルの読み取り](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQL パフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md) などが含まれています。
- [TiDB 開発者コース](https://www.pingcap.com/education/)を受講して、試験に合格した後に[TiDB 認定](https://www.pingcap.com/education/certification/)を取得しましょう。

## ヘルプが必要ですか？

<CustomContent platform="tidb">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](/support.md)してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。

</CustomContent>