---
title: Amazon AppFlow と TiDB を統合する
summary: Amazon AppFlow と TiDB をステップバイステップで統合する方法を紹介します。

# Amazon AppFlow と TiDB を統合する

[Amazon AppFlow](https://aws.amazon.com/appflow/) は、SaaS（ソフトウェア・アズ・ア・サービス）アプリケーションをAWSサービスに接続し、データの安全な転送を可能にするフルマネージドAPI統合サービスです。Amazon AppFlowを使用すると、Salesforce、Amazon S3、LinkedIn、およびGitHubなどの様々なタイプのデータプロバイダからTiDBへのデータのインポートおよびエクスポートができます。詳細については、AWSのドキュメンテーションの[対応するソースおよび宛先アプリケーション](https://docs.aws.amazon.com/appflow/latest/userguide/app-specific.html)をご覧ください。

このドキュメントでは、TiDB を Amazon AppFlow と統合する方法を説明し、TiDB Serverless クラスタを統合する手順を例に挙げます。

TiDB クラスタを持っていない場合は、[TiDB Serverless](https://tidbcloud.com/console/clusters)クラスタを作成できます。これは無料でおよそ30秒で作成できます。

## 前提条件

- [Git](https://git-scm.com/)
- [JDK](https://openjdk.org/install/) 11以上
- [Maven](https://maven.apache.org/install.html) 3.8以上
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) バージョン2
- [AWS Serverless Application Model Command Line Interface (AWS SAM CLI)](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html) 1.58.0以上
- AWS [Identity and Access Management (IAM) ユーザー](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html) が以下の要件を満たしていること：

    - ユーザーは[アクセスキー](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html)を使用してAWSにアクセスできる。
    - ユーザーには以下の権限がある：

        - `AWSCertificateManagerFullAccess`: [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/)を読み書きするために使用される。
        - `AWSCloudFormationFullAccess`: SAM CLIは[Amazon S3](https://aws.amazon.com/s3/?nc2=h_ql_prod_fs_s3)を使ってAWSリソースを宣言するためにAWS CloudFormationを使用する。
        - `AmazonS3FullAccess`: AWS CloudFormationは[AWS S3](https://aws.amazon.com/s3/?nc2=h_ql_prod_fs_s3)を公開するために使用する。
        - `AWSLambda_FullAccess`: 現在、[AWS Lambda](https://aws.amazon.com/lambda/?nc2=h_ql_prod_fs_lbd)がAmazon AppFlowの新しいコネクタを実装する唯一の方法である。
        - `IAMFullAccess`: SAM CLIはコネクタのために`ConnectorFunctionRole`を作成する必要がある。

- [SalesForce](https://developer.salesforce.com) アカウント。

## ステップ1. TiDB コネクタを登録する

### コードをクローンする

TiDB と Amazon AppFlow の[統合例のコードリポジトリ](https://github.com/pingcap-inc/tidb-appflow-integration)をクローンします。

```bash
git clone https://github.com/pingcap-inc/tidb-appflow-integration
```

### Lambda をビルドおよびアップロードする

1. パッケージをビルドする：

    ```bash
    cd tidb-appflow-integration
    mvn clean package
    ```

2. (任意) AWSにアクセスするためのアクセスキーIDとシークレットアクセスキーを構成する（まだであれば）。

    ```bash
    aws configure
    ```

3. JARパッケージをLambdaとしてアップロードする：

    ```bash
    sam deploy --guided
    ```

    > **注意：**
    >
    > - `--guided`オプションはプロンプトを使用して展開を案内します。入力内容は構成ファイル（デフォルトでは`samconfig.toml`）に保存されます。
    > - `stack_name` は展開するAWS Lambdaの名前を指定します。
    > - この案内に従ってAWS Lambdaを展開するためには、TiDB Serverless のクラウドプロバイダとしてAWSを使用します。ソースまたは宛先としてAmazon S3を使用する場合は、AWS Lambdaの`region`をAmazon S3のリージョンと同じに設定する必要があります。
    > - すでに`sam deploy --guided`を実行したことがある場合は、単に代わりに`sam deploy`を実行するだけで、SAM CLIは構成ファイル`samconfig.toml`を使用してインタラクションを簡素化します。

    以下のような出力が表示された場合、このLambdaは正常に展開されています。

    ```
    Successfully created/updated stack - <stack_name> in <region>
    ```

4. [AWS Lambda コンソール](https://console.aws.amazon.com/lambda/home)に移動し、アップロードしたLambdaが表示されていることを確認します。ウィンドウの右上隅で正しいリージョンを選択する必要があります。

    ![lambda dashboard](/media/develop/aws-appflow-step-lambda-dashboard.png)

### Lambdaを使用してコネクタを登録する

1. [AWS Management Console](https://console.aws.amazon.com)で [Amazon AppFlow > Connectors](https://console.aws.amazon.com/appflow/home#/gallery)に移動し、**新しいコネクタを登録** をクリックします。

    ![register connector](/media/develop/aws-appflow-step-register-connector.png)

2. **新しいコネクタを登録** ダイアログで、アップロードしたLambda関数を選択し、コネクタ名を指定してコネクタラベルを指定します。

    ![register connector dialog](/media/develop/aws-appflow-step-register-connector-dialog.png)

3. **登録** をクリックします。これにより、TiDBコネクタが正常に登録されます。

## ステップ2. フローを作成する

[Amazon AppFlow > Flows](https://console.aws.amazon.com/appflow/home#/list)に移動し、**フローを作成** をクリックします。

![create flow](/media/develop/aws-appflow-step-create-flow.png)

### フロー名を設定する

フロー名を入力し、**次へ** をクリックします。

![name flow](/media/develop/aws-appflow-step-name-flow.png)

### ソースおよび宛先テーブルを設定する

**ソースの詳細** および **宛先の詳細** を選択します。TiDBコネクタはどちらも使用できます。

1. ソース名を選択します。このドキュメントでは、例として**Salesforce**を使用します。

    ![salesforce source](/media/develop/aws-appflow-step-salesforce-source.png)

    Salesforceに登録した後、Salesforceはプラットフォームにいくつかのサンプルデータを追加します。次の手順では、**アカウント** オブジェクトをソースオブジェクトとして使用します。

    ![salesforce data](/media/develop/aws-appflow-step-salesforce-data.png)

2. **接続** をクリックします。

    1. **Salesforceに接続** ダイアログで、この接続の名前を指定し、**続行** をクリックします。

        ![connect to salesforce](/media/develop/aws-appflow-step-connect-to-salesforce.png)

    2. AWSがSalesforceデータを読み込めるように**許可** をクリックします。

        ![allow salesforce](/media/develop/aws-appflow-step-allow-salesforce.png)

    > **注意：**
    >
    > 会社がすでにSalesforceのプロフェッショナルエディションを使用している場合、REST APIはデフォルトで有効になっていないことがあります。REST APIを使用するためには新しい開発者エディションを登録する必要があります。詳細については、[Salesforce フォーラムトピック](https://developer.salesforce.com/forums/?id=906F0000000D9Y2IAK)を参照してください。

3. **宛先の詳細** エリアで、宛先として **TiDB-Connector** を選択します。**接続** ボタンが表示されます。

    ![tidb dest](/media/develop/aws-appflow-step-tidb-dest.png)

4. **接続** をクリックする前に、Salesforce **アカウント** オブジェクト用にTiDBに`sf_account` テーブルを作成する必要があります。このテーブルのスキーマは、[Amazon AppFlowのチュートリアル](https://docs.aws.amazon.com/appflow/latest/userguide/flow-tutorial-set-up-source.html)のサンプルデータとは異なります。

    ```sql
    CREATE TABLE `sf_account` (
        `id` varchar(255) NOT NULL,
        `name` varchar(150) NOT NULL DEFAULT '',
        `type` varchar(150) NOT NULL DEFAULT '',
        `billing_state` varchar(255) NOT NULL DEFAULT '',
        `rating` varchar(255) NOT NULL DEFAULT '',
        `industry` varchar(255) NOT NULL DEFAULT '',
        PRIMARY KEY (`id`)
    );
    ```

5. `sf_account` テーブルが作成されたら、**接続** をクリックします。接続ダイアログが表示されます。
6. **TiDB-Connectorに接続する** ダイアログで、TiDBクラスターの接続プロパティを入力します。TiDB Serverless クラスターを使用している場合は、TiDBコネクタがTLS接続を使用するように**TLS** オプションを `Yes` に設定する必要があります。その後、**接続** をクリックします。

    ![tidb connection message](/media/develop/aws-appflow-step-tidb-connection-message.png)

7. 今、指定したデータベースのすべてのテーブルを取得できます。ドロップダウンリストから **sf_account** テーブルを選択します。

    ![database](/media/develop/aws-appflow-step-database.png)

    以下のスクリーンショットは、Salesforce **アカウント** オブジェクトからTiDBの `sf_account` テーブルにデータを転送するための設定を示しています。

    ![complete flow](/media/develop/aws-appflow-step-complete-flow.png)

8. **エラー処理**エリアで、**現在のフロー実行を停止**を選択します。**フロートリガー**エリアで、**必要に応じて実行**トリガータイプを選択します。これはつまり、フローを手動で実行する必要があることを意味します。次に、**次へ**をクリックします。

    ![step1を完了](/media/develop/aws-appflow-step-complete-step1.png)

### マッピングルールを設定

Salesforceの**アカウント**オブジェクトのフィールドをTiDBの`sf_account`テーブルにマップし、次に**次へ**をクリックします。

- `sf_account`テーブルはTiDBで新しく作成され、空です。

    ```sql
    test> SELECT * FROM sf_account;
    +----+------+------+---------------+--------+----------+
    | id | name | type | billing_state | rating | industry |
    +----+------+------+---------------+--------+----------+
    +----+------+------+---------------+--------+----------+
    ```

- マッピングルールを設定するには、左側のソースフィールド名を選択し、右側の宛先フィールド名を選択します。次に**フィールドのマッピング**をクリックし、ルールを設定します。

    ![マッピングルールの追加](/media/develop/aws-appflow-step-add-mapping-rule.png)

- このドキュメントには、以下のマッピングルール（ソースフィールド名->宛先フィールド名）が必要です:

    - アカウントID -> id
    - アカウント名 -> name
    - アカウントタイプ -> type
    - 請求州/都道府県 -> billing_state
    - アカウント評価 -> rating
    - 業界 -> industry

    ![ルールのマッピング](/media/develop/aws-appflow-step-mapping-a-rule.png)

    ![全てのマッピングルールを表示](/media/develop/aws-appflow-step-show-all-mapping-rules.png)

### (オプション) フィルターを設定

データフィールドにいくつかのフィルターを追加したい場合は、ここで設定できます。それ以外の場合は、この手順をスキップして**次へ**をクリックします。

![フィルター](/media/develop/aws-appflow-step-filters.png)

### 確認してフローを作成

作成するフローの情報を確認します。すべて正常に見える場合は、**フローを作成**をクリックします。

![レビュー](/media/develop/aws-appflow-step-review.png)

## ステップ3. フローを実行

新しく作成されたフローのページで、右上隅にある**フローを実行**をクリックします。

![フローを実行](/media/develop/aws-appflow-step-run-flow.png)

以下のスクリーンショットは、フローが正常に実行された例を示しています:

![実行成功](/media/develop/aws-appflow-step-run-success.png)

`sf_account`テーブルをクエリし、Salesforceの**アカウント**オブジェクトからのレコードが書き込まれたことが確認できます:

```sql
test> SELECT * FROM sf_account;
+--------------------+-------------------------------------+--------------------+---------------+--------+----------------+
| id                 | name                                | type               | billing_state | rating | industry       |
+--------------------+-------------------------------------+--------------------+---------------+--------+----------------+
| 001Do000003EDTlIAO | エンタイトルメント向けのサンプルアカウント | null               | null          | null   | null           |
| 001Do000003EDTZIA4 | エッジ通信                        | Customer - Direct  | TX            | Hot    | 電子機器        |
| 001Do000003EDTaIAO | バーリントン・テキスタイルズ・コーポレーション・オブ・アメリカ | Customer - Direct  | NC            | Warm   | 衣料品          |
| 001Do000003EDTbIAO | ピラミッド・コンストラクション社       | Customer - Channel | null          | null   | 建設            |
| 001Do000003EDTcIAO | ディッケンソン社                    | Customer - Channel | KS            | null   | コンサルティング |
| 001Do000003EDTdIAO | グランドホテル&リゾーツ社            | Customer - Direct  | IL            | Warm   | ホスピタリティ   |
| 001Do000003EDTeIAO | ユナイテッドオイル&ガス社            | Customer - Direct  | NY            | Hot    | エネルギー      |
| 001Do000003EDTfIAO | エクスプレス・ロジスティクス＆トランスポート社 | Customer - Channel | OR            | Cold   | 交通            |
| 001Do000003EDTgIAO | アリゾナ大学                        | Customer - Direct  | AZ            | Warm   | 教育            |
| 001Do000003EDThIAO | ユナイテッドオイル&ガス, UK         | Customer - Direct  | UK            | null   | エネルギー      |
| 001Do000003EDTiIAO | ユナイテッドオイル&ガス, シンガポール | Customer - Direct  | シンガポール   | null   | エネルギー      |
| 001Do000003EDTjIAO | ジーンポイント                      | Customer - Channel | CA            | Cold   | バイオテクノロジー |
| 001Do000003EDTkIAO | sForce                             | null               | CA            | null   | null           |
+--------------------+-------------------------------------+--------------------+---------------+--------+----------------+
```

## 注意事項

- 何か問題が発生した場合は、AWS Management Consoleの[CloudWatch](https://console.aws.amazon.com/cloudwatch/home)ページに移動してログを取得できます。
- このドキュメントの手順は、[Amazon AppFlow Custom Connector SDKを使用したカスタムコネクターの構築](https://aws.amazon.com/blogs/compute/building-custom-connectors-using-the-amazon-appflow-custom-connector-sdk/)に基づいています。
- [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)は**本番環境ではありません**。
- 過度な長さを防ぐため、このドキュメントの例では`Insert`戦略のみを示していますが、`Update`および`Upsert`戦略もテストされており使用可能です。