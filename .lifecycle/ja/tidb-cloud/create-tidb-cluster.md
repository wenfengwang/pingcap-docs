---
title: TiDB専用クラスターの作成
summary: TiDB専用クラスターを作成する方法について学びます。

# TiDB専用クラスターの作成

このチュートリアルでは、TiDB専用クラスターのサインアップおよび作成方法について説明します。

> **ヒント:**
>
> TiDB Serverlessクラスターの作成方法については、[TiDB Serverlessクラスターの作成](/tidb-cloud/create-tidb-cluster-serverless.md)を参照してください。

## 開始する前に

TiDB Cloudアカウントをお持ちでない場合は、[こちら](https://tidbcloud.com/signup)をクリックしてアカウントを作成してください。

- TiDB Cloudでパスワードを管理するためにメールアドレスとパスワードでサインアップするか、Google、GitHub、またはMicrosoftアカウントでサインアップすることができます。
- AWS Marketplaceユーザーの場合は、AWS Marketplaceを介してサインアップすることもできます。詳細については、[AWS Marketplace](https://aws.amazon.com/marketplace)で「TiDB Cloud」を検索し、TiDB Cloudに登録し、その後画面の指示に従ってTiDB Cloudアカウントを設定してください。
- Google Cloud Marketplaceユーザーの場合は、Google Cloud Marketplaceを介してサインアップすることもできます。詳細については、[Google Cloud Marketplace](https://console.cloud.google.com/marketplace)で「TiDB Cloud」を検索し、TiDB Cloudに登録し、その後画面の指示に従ってTiDB Cloudアカウントを設定してください。

## (オプション) ステップ1. デフォルトプロジェクトを使用するか新しいプロジェクトを作成する

[TiDB Cloudコンソール](https://tidbcloud.com/)にログインすると、デフォルトの[プロジェクト](/tidb-cloud/tidb-cloud-glossary.md#project)があります。組織にプロジェクトが1つしかない場合は、クラスターはそのプロジェクトに作成されます。プロジェクトについての詳細については、[組織とプロジェクト](/tidb-cloud/manage-user-access.md#organizations-and-projects)を参照してください。

組織所有者である場合は、次のようにしてデフォルトのプロジェクトの名前を変更するか、必要に応じてクラスターのための新しいプロジェクトを作成することができます。

1. [TiDB Cloudコンソール](https://tidbcloud.com/)にログインし、左下隅の<MDSvgIcon name="icon-top-organization" />をクリックします。

2. **組織の設定**をクリックします。

    **プロジェクト**タブがデフォルトで表示されます。

3. 次のいずれかを行います。

    - デフォルトのプロジェクトの名前を変更する場合は、「アクション」列の**名前変更**をクリックします。
    - プロジェクトを作成する場合は、「新しいプロジェクトを作成」をクリックし、プロジェクトの名前を入力し、「確認」をクリックします。

4. クラスターページに戻るには、ウィンドウの左上隅にあるTiDB Cloudロゴをクリックします。

## ステップ2. TiDB専用クラスターを作成する

「組織所有者」または「プロジェクト所有者」の役割にある場合、次の手順でTiDB専用クラスターを作成できます。

1. プロジェクトの[**クラスター**](https://tidbcloud.com/console/clusters)ページに移動します。

    > **ヒント:**
    >
    > 複数のプロジェクトを持っている場合、「左下隅の<MDSvgIcon name="icon-left-projects" />」をクリックして別のプロジェクトに切り替えることができます。

2. **クラスターを作成**をクリックします。

3. **クラスターを作成**ページで、**専用**を選択し、以下のようにクラスター情報を構成します。

    1. クラウドプロバイダーとリージョンを選択します。

        > **注意:**
        >
        > - [AWS Marketplace](https://aws.amazon.com/marketplace)経由でTiDB Cloudにサインアップした場合、クラウドプロバイダーはAWSであり、TiDB Cloudで変更することはできません。
        > - [Google Cloud Marketplace](https://console.cloud.google.com/marketplace)経由でTiDB Cloudにサインアップした場合、クラウドプロバイダーはGoogle Cloudであり、TiDB Cloudで変更することはできません。

    2. TiDB、TiKV、およびTiFlash（オプション）の[クラスターサイズ](/tidb-cloud/size-your-cluster.md)をそれぞれ構成します。
    3. 必要に応じてデフォルトのクラスター名とポート番号を更新します。
    4. 現在のプロジェクトの最初のクラスターであり、このプロジェクトに対してCIDRが構成されていない場合、プロジェクトのCIDRを設定する必要があります。**プロジェクトCIDR**フィールドが表示されない場合は、すでにこのプロジェクトのCIDRが構成されていることを意味します。

        > **注意:**
        >
        > プロジェクトCIDRを設定する際は、アプリケーションが配置されているVPCのCIDRとの競合を避けてください。プロジェクトCIDRが設定されると、それを変更することはできません。

4. 右側のクラスターおよび請求情報を確認します。

5. 支払い方法が追加されていない場合は、左下隅の**クレジットカードを追加**をクリックします。

    > **注意:**
    >
    > [AWS Marketplace](https://aws.amazon.com/marketplace)または[Google Cloud Marketplace](https://console.cloud.google.com/marketplace)を介してTiDB Cloudにサインアップした場合、AWSアカウントまたはGoogle Cloudアカウントを介して支払いを行うことができますが、TiDB Cloudコンソールで支払い方法を追加したり、請求書をダウンロードしたりすることはできません。

6. **作成**をクリックします。

    TiDB Cloudクラスターはおよそ20〜30分で作成されます。

## ステップ3. セキュリティ設定を構成する

クラスターが作成された後は、以下の手順に従ってセキュリティ設定を構成します。

1. クラスター概要ページの右上隅にある**...**をクリックし、**セキュリティ設定**を選択します。

2. ルートパスワードを設定し、クラスターに接続するための許可されたIPアドレスを設定し、「適用」をクリックします。

## 次は何ですか

TiDB Cloudでクラスターが作成されたら、[TiDB専用クラスターへの接続](/tidb-cloud/connect-via-standard-connection-serverless.md)方法を提供されている方法で接続できます。