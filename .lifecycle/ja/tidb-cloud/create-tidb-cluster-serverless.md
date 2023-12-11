---
title: TiDBサーバーレスクラスターの作成
summary: TiDBサーバレスクラスターの作成方法について学びます。

# TiDBサーバーレスクラスターの作成

この文書では、[TiDB Cloudコンソール](https://tidbcloud.com/)でTiDBサーバーレスクラスターを作成する方法について説明します。

> **ヒント:**
>
> TiDB専用クラスターの作成方法については、[TiDB専用クラスターの作成](/tidb-cloud/create-tidb-cluster.md)を参照してください。

## 開始する前に

TiDB Cloudアカウントをお持ちでない場合は、[こちら](https://tidbcloud.com/signup)をクリックしてアカウントを作成してください。

- TiDB Cloudを使用してパスワードを管理できるように、メールアドレスとパスワードでサインアップするか、Google、GitHub、またはMicrosoftアカウントでサインアップしてください。
- AWS Marketplaceユーザーは、AWS Marketplaceを介してもサインアップできます。そのためには、[AWS Marketplace](https://aws.amazon.com/marketplace)で`TiDB Cloud`を検索し、TiDB Cloudにサブスクライブし、その後画面の指示に従ってTiDB Cloudアカウントを設定してください。
- Google Cloud Marketplaceユーザーは、Google Cloud Marketplaceを介してもサインアップできます。そのためには、[Google Cloud Marketplace](https://console.cloud.google.com/marketplace)で`TiDB Cloud`を検索し、TiDB Cloudにサブスクライブし、その後画面の指示に従ってTiDB Cloudアカウントを設定してください。

## 手順

`Organization Owner`または`Project Owner`の役割であれば、次のようにTiDBサーバーレスクラスターを作成できます。

1. [TiDB Cloudコンソール](https://tidbcloud.com/)にログインし、[**Clusters**](https://tidbcloud.com/console/clusters)ページに移動します。

2. **クラスターの作成**をクリックします。

3. **クラスターの作成**ページでは、デフォルトで**Serverless**が選択されています。

4. TiDBサーバーレスのクラウドプロバイダーはAWSです。クラスターをホストするAWSリージョンを選択できます。

5. (オプション) [無料クォータ](/tidb-cloud/select-cluster-tier.md#usage-quota)よりも多くのストレージとコンピューティングリソースを使用する予定であれば、支出上限を変更してください。支出上限を編集した後、支払方法が登録されていない場合はクレジットカードを追加する必要があります。

    > **注意:**
    >
    > TiDB Cloudの各組織では、デフォルトで最大5つのTiDBサーバーレスクラスターを作成できます。さらにTiDBサーバーレスクラスターを作成するには、クレジットカードを追加し、使用量の[支出上限](/tidb-cloud/tidb-cloud-glossary.md#spending-limit)を設定する必要があります。

6. 必要に応じてデフォルトのクラスター名を更新し、**作成**をクリックします。

    クラスターの作成プロセスが開始され、TiDB Cloudクラスターは約30秒で作成されます。

## 次の手順

クラスターが作成されたら、[パブリックエンドポイント経由でTiDBサーバーレスに接続](/tidb-cloud/connect-via-standard-connection-serverless.md)し、クラスターのためのパスワードを作成してください。

> **注意:**
>
> パスワードを設定しない場合、クラスターに接続することはできません。