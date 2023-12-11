---
title: VercelとTiDB Cloudの統合
summary: TiDB CloudクラスタをVercelプロジェクトに接続する方法について学びます。

<!-- markdownlint-disable MD029 -->

# VercelとTiDB Cloudの統合

[Vercel](https://vercel.com/)は、フロントエンド開発者向けのプラットフォームで、革新を生み出す際に必要な速度と信頼性を提供します。

TiDB CloudをVercelと統合することで、MySQL互換のリレーショナルモデルを使用して新しいフロントエンドアプリケーションを迅速に構築し、回復力、拡張性、および最高レベルのデータプライバシーおよびセキュリティを備えたプラットフォームでアプリを自信を持って拡大できます。

このガイドでは、TiDB CloudクラスタをVercelプロジェクトに接続する手順を、次の方法のいずれかを使用して説明します。

* [TiDB Cloud Vercel統合を介した接続](#connect-via-the-tidb-cloud-vercel-integration)
* [環境変数の手動設定を介した接続](#connect-via-manually-setting-environment-variables)

上記のいずれの方法でも、TiDB Cloudは、次のオプションをプログラムによるデータベースへの接続に提供します。

- 直接接続：MySQLの標準接続システムを使用してTiDB CloudクラスタをVercelプロジェクトに直接接続します。
- [データアプリ](/tidb-cloud/data-service-manage-data-app.md)：HTTPエンドポイントのコレクションを介してTiDB Cloudクラスタのデータにアクセスします。

## 前提条件

接続前に、次の前提条件を満たしていることを確認してください。

### VercelアカウントとVercelプロジェクト

Vercelのアカウントとプロジェクトを持っていることを想定しています。持っていない場合は、次のVercelのドキュメントを参照して作成してください。

* [新しい個人アカウントの作成](https://vercel.com/docs/teams-and-accounts#creating-a-personal-account)または[新しいチームの作成](https://vercel.com/docs/teams-and-accounts/create-or-join-a-team#creating-a-team)。
* Vercelでの[プロジェクトの作成](https://vercel.com/docs/concepts/projects/overview#creating-a-project)、またはデプロイするアプリケーションがない場合は、[TiDB Cloudスターターテンプレート](https://vercel.com/templates/next.js/tidb-cloud-starter)を使用して試してみてください。

1つのVercelプロジェクトは1つのTiDB Cloudクラスタにのみ接続できます。統合を変更するには、現在のクラスタを最初に切断し、次に新しいクラスタに接続する必要があります。

### TiDB CloudアカウントとTiDBクラスタ

TiDB Cloudのアカウントとクラスタを持っていることを想定しています。持っていない場合は、次の手順に従って作成してください。

- [TiDB Serverlessクラスタの作成](/tidb-cloud/create-tidb-cluster-serverless.md)

    > **注意:**
    >
    > TiDB Cloud Vercel統合では、TiDB Serverlessクラスタの作成がサポートされています。統合プロセス中に後で作成することもできます。

- [TiDB Dedicatedクラスタの作成](/tidb-cloud/create-tidb-cluster.md)

    > **注意:**
    >
    > TiDB Dedicatedクラスタの場合、クラスタのトラフィックフィルタが接続のためにすべてのIPアドレス（`0.0.0.0/0`に設定）を許可していることを確認してください。なぜなら、Vercelのデプロイは[動的IPアドレス](https://vercel.com/guides/how-to-allowlist-deployment-ip-address)を使用しているからです。TiDB Cloud Vercel統合を使用する場合、TiDB Cloudは自動的にクラスタに`0.0.0.0/0`のトラフィックフィルタを追加します。

[VercelをTiDB Cloud Vercel Integrationを介して統合する](#connect-via-the-tidb-cloud-vercel-integration)には、TiDB Cloudの組織の`Organization Owner`役割またはTiDB Cloudの`Project Owner`役割であることが想定されます。詳細については、[ユーザーの役割](/tidb-cloud/manage-user-access.md#user-roles)を参照してください。

1つのTiDB Cloudクラスタは複数のVercelプロジェクトに接続できます。

### Data Appおよびエンドポイント

[TiDB Cloudクラスタに[Data App](/tidb-cloud/data-service-manage-data-app.md)を介して接続したい場合は、あらかじめTiDB CloudにターゲットのData Appおよびエンドポイントを持っていることが想定されます。持っていない場合は、次の手順に従って作成してください。

1. [TiDB Cloudコンソール](https://tidbcloud.com)にアクセスし、プロジェクトの[**Data Service**](https://tidbcloud.com/console/data-service)ページに移動します。
2. プロジェクト用に[データアプリ](/tidb-cloud/data-service-manage-data-app.md#create-a-data-app)を作成します。
3. [Data Appを](/tidb-cloud/data-service-manage-data-app.md#manage-linked-data-sources)ターゲットのTiDB Cloudクラスタにリンクします。
4. [エンドポイントを管理](/tidb-cloud/data-service-manage-endpoint.md)し、SQLステートメントを実行するためにそれらをカスタマイズできるようにします。

1つのVercelプロジェクトは1つのTiDB Cloud Data Appにのみ接続できます。VercelプロジェクトのData Appを変更するには、まず現在のAppを切断し、次に新しいAppに接続する必要があります。

## TiDB Cloud Vercel Integrationを介した接続

TiDB Cloud Vercel Integrationを介して接続するには、[Vercelの統合マーケットプレイス](https://vercel.com/integrations)から[TiDB Cloud統合](https://vercel.com/integrations/tidb-cloud)ページに移動します。この方法を使用すると、接続するクラスタを選択したり、TiDB CloudがVercelプロジェクト用のすべての必要な環境変数を自動生成したりすることができます。

詳細な手順は以下の通りです：

<SimpleTab>
<div label="直接接続">

1. [TiDB Cloud Vercel integration](https://vercel.com/integrations/tidb-cloud)ページの右上にある**Add Integration**をクリックします。**Add TiDB Cloud**ダイアログが表示されます。
2. ドロップダウンリストで統合のスコープを選択し、**Continue**をクリックします。
3. インテグレーションを追加するVercelプロジェクトを選択し、**Continue**をクリックします。
4. インテグレーションのための必要な権限を確認し、**Add Integration**をクリックします。その後、TiDB Cloudコンソールのインテグレーションページに移動します。
5. インテグレーションページで、次の手順を実行します：

    1. 対象のVercelプロジェクトを選択し、**Next**をクリックします。
    2. ターゲットのTiDB Cloud組織とプロジェクトを選択します。
    3. 接続タイプとして**Cluster**を選択します。
    4. ターゲットのTiDB Cloudクラスタを選択します。**Cluster**のドロップダウンリストが空である場合や新しいTiDB Serverlessクラスタを選択したい場合は、リスト内の**+ Create Cluster**をクリックして作成します。
    5. Vercelプロジェクトで使用しているフレームワークを選択します。対象のフレームワークがリストされていない場合は、**General**を選択します。異なるフレームワークによって異なる環境変数が決定されます。
    6. **Add Integration and Return to Vercel**をクリックします。

![Vercel Integration Page](/media/tidb-cloud/vercel/integration-link-cluster-page.png)

6. Vercelダッシュボードに戻り、Vercelプロジェクトに移動し、**Settings** > **Environment Variables**をクリックし、ターゲットのTiDBクラスタの環境変数が自動的に追加されているかどうかを確認します。

    次の変数が追加されている場合、統合が完了しています。

    **General**

    ```shell
    TIDB_HOST
    TIDB_PORT
    TIDB_USER
    TIDB_PASSWORD
    ```

    TiDB Dedicatedクラスタの場合、ルートCAがこの変数に設定されています。

    ```
    TIDB_SSL_CA
    ```

    **Prisma**

    ```
    DATABASE_URL
    ```

</div>

<div label="Data App">

1. [TiDB Cloud Vercel integration](https://vercel.com/integrations/tidb-cloud)ページの右上にある**Add Integration**をクリックします。**Add TiDB Cloud**ダイアログが表示されます。
2. ドロップダウンリストで統合のスコープを選択し、**Continue**をクリックします。
3. インテグレーションを追加するVercelプロジェクトを選択し、**Continue**をクリックします。
4. インテグレーションのための必要な権限を確認し、**Add Integration**をクリックします。その後、TiDB Cloudコンソールのインテグレーションページに移動します。
5. インテグレーションページで、次の手順を実行します：

    1. 対象のVercelプロジェクトを選択し、**Next**をクリックします。
    2. ターゲットのTiDB Cloud組織とプロジェクトを選択します。
    3. 接続タイプとして**Data App**を選択します。
    4. ターゲットのTiDB Data Appを選択します。
    6. **Add Integration and Return to Vercel**をクリックします。

![Vercel Integration Page](/media/tidb-cloud/vercel/integration-link-data-app-page.png)

6. Vercelダッシュボードに戻り、Vercelプロジェクトに移動し、**Settings** > **Environment Variables**をクリックし、ターゲットのData Appの環境変数が自動的に追加されているかどうかを確認します。

    次の変数が追加されている場合、統合が完了しています。

    ```shell
    DATA_APP_BASE_URL
    DATA_APP_PUBLIC_KEY
    DATA_APP_PRIVATE_KEY
    ```

</div>
</SimpleTab>

## 環境変数の手動設定を介した接続

<SimpleTab>
<div label="直接接続">

1. TiDBクラスタの接続情報を取得します。
    クラスターの接続情報は、クラスターの接続ダイアログから取得できます。ダイアログを開くには、プロジェクトの [**クラスター**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスターの名前をクリックして概要ページに移動し、右上隅の **Connect** をクリックします。

> **注意:**
> 
> TiDB Dedicated クラスターの場合は、この手順で **Allow Access from Anywhere** トラフィックフィルターが設定されていることを確認してください。

2. Vercel ダッシュボードに移動し、Vercel プロジェクト > **Settings** > **Environment Variables** へ進み、その後、[接続情報に応じて各環境変数の値を宣言](https://vercel.com/docs/concepts/projects/environment-variables#declare-an-environment-variable)してください。

    ![Vercel Environment Variables](/media/tidb-cloud/vercel/integration-vercel-environment-variables.png)

ここでは Prisma アプリケーションを例として使用します。以下は、TiDB Serverless クラスターの Prisma スキーマファイルでのデータソース設定です。

```
datasource db {
    provider = "mysql"
    url      = env("DATABASE_URL")
}
```

Vercel では、環境変数を次のように宣言できます。

- **Key** = `DATABASE_URL`
- **Value** = `mysql://<User>:<Password>@<Endpoint>:<Port>/<Database>?sslaccept=strict`

`<User>`, `<Password>`, `<Endpoint>`, `<Port>`, `<Database>` の情報は、TiDB Cloud コンソールで入手できます。

</div>
<div label="Data App">

1. [データ APP の管理](/tidb-cloud/data-service-manage-data-app.md)と[エンドポイントの管理](/tidb-cloud/data-service-manage-endpoint.md)の手順に従って、データ APP とそのエンドポイントを作成していない場合は、手順に従ってください。

2. Vercel ダッシュボードに移動し、Vercel プロジェクト > **Settings** > **Environment Variables** と進み、[接続情報に応じて各環境変数の値を宣言](https://vercel.com/docs/concepts/projects/environment-variables#declare-an-environment-variable)してください。

    ![Vercel Environment Variables](/media/tidb-cloud/vercel/integration-vercel-environment-variables.png)

    Vercel では、環境変数を次のように宣言できます。

    - **Key** = `DATA_APP_BASE_URL`
    - **Value** = `<DATA_APP_BASE_URL>`
    - **Key** = `DATA_APP_PUBLIC_KEY`
    - **Value** = `<DATA_APP_PUBLIC_KEY>`
    - **Key** = `DATA_APP_PRIVATE_KEY`
    - **Value** = `<DATA_APP_PRIVATE_KEY>`

    `<DATA_APP_BASE_URL>`, `<DATA_APP_PUBLIC_KEY>`, `<DATA_APP_PRIVATE_KEY>` の情報は、TiDB Cloud コンソールの [Data Service](https://tidbcloud.com/console/data-service) ページから入手できます。

</div>
</SimpleTab>

## 接続の構成

[TiDB Cloud Vercel integration](https://vercel.com/integrations/tidb-cloud)をインストールしている場合は、連携内で接続を追加または削除できます。

1. Vercel ダッシュボードで **Integrations** をクリックします。
2. TiDB Cloud エントリで **Manage** をクリックします。
3. **Configure** をクリックします。
4. 接続を追加または削除するには、**Add Link** または **Remove** をクリックします。

    ![Vercel Integration Configuration Page](/media/tidb-cloud/vercel/integration-vercel-configuration-page.png)

    接続を削除すると、連携のワークフローで設定された環境変数が Vercel プロジェクトから削除されます。トラフィックフィルターや TiDB Cloud クラスターのデータには影響がありません。