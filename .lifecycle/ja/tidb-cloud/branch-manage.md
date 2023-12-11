---
title: TiDBサーバーレスのブランチを管理する
summary: TiDBサーバーレスのブランチを管理する方法について学ぶ。

# TiDBサーバーレスのブランチを管理する

この文書では[TiDB Cloudコンソール](https://tidbcloud.com)を使用してTiDBサーバーレスのブランチを管理する方法について説明します。TiDB Cloud CLIを使用して管理する方法については、[`ticloud branch`](/tidb-cloud/ticloud-branch-create.md)を参照してください。

## 必要なアクセス権

- [ブランチを作成](#create-a-branch)したり、[ブランチに接続](#connect-to-a-branch)したりするには、組織の`所有者`ロールまたは対象プロジェクトの`プロジェクト所有者`ロールに属している必要があります。
- プロジェクト内のクラスタのブランチを[表示](#create-a-branch)するには、そのプロジェクトに属している必要があります。

アクセス権に関する詳細については、[ユーザーの役割](/tidb-cloud/manage-user-access.md#user-roles)を参照してください。

## ブランチを作成する

> **注意:**
>
> 2023年7月5日以降に作成されたTiDBサーバーレスクラスタに対してのみブランチを作成できます。詳細な制限事項については、[制限事項とクォータ](/tidb-cloud/branch-overview.md#limitations-and-quotas)を参照してください。

ブランチを作成するには、以下の手順を実行します：

1. [TiDB Cloudコンソール](https://tidbcloud.com/)にアクセスし、プロジェクトの[**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、対象のTiDBサーバーレスクラスタの名前をクリックして、概要ページに移動します。
2. 左側のナビゲーションペインで**Branches**をクリックします。
3. 右上隅の**Create Branch**をクリックします。
4. ブランチ名を入力し、**Create**をクリックします。

クラスタのデータサイズに応じて、ブランチの作成が数分で完了します。

## ブランチを表示する

クラスタのブランチを表示するには、以下の手順を実行します：

1. [TiDB Cloudコンソール](https://tidbcloud.com/)にアクセスし、プロジェクトの[**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、対象のTiDBサーバーレスクラスタの名前をクリックして、概要ページに移動します。
2. 左側のナビゲーションペインで**Branches**をクリックします。

    クラスタのブランチリストが右側のペインに表示されます。

## ブランチに接続する

ブランチに接続するには、以下の手順を実行します：

1. [TiDB Cloudコンソール](https://tidbcloud.com/)にアクセスし、プロジェクトの[**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、対象のTiDBサーバーレスクラスタの名前をクリックして、概要ページに移動します。
2. 左側のナビゲーションペインで**Branches**をクリックします。
3. 接続する対象のブランチの行で、**Action**列の**...**をクリックします。
4. ドロップダウンリストで**Connect**をクリックします。接続情報のダイアログが表示されます。
5. ルートパスワードを作成またはリセットするために、**Create password**または**Reset password**をクリックします。
6. 接続情報を使用してブランチに接続します。

> **注意:**
>
> 現在、ブランチでは[プライベートエンドポイント](/tidb-cloud/set-up-private-endpoint-connections-serverless.md)はサポートされていません。

## ブランチを削除する

ブランチを削除するには、以下の手順を実行します：

1. [TiDB Cloudコンソール](https://tidbcloud.com/)にアクセスし、プロジェクトの[**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、対象のTiDBサーバーレスクラスタの名前をクリックして、概要ページに移動します。
2. 左側のナビゲーションペインで**Branches**をクリックします。
3. 削除する対象のブランチの行で、**Action**列の**...**をクリックします。
4. ドロップダウンリストで**Delete**をクリックします。
5. 削除を確認します。

## 次の手順

- [TiDBサーバーレスのブランチをGitHub CI/CDパイプラインに統合する](/tidb-cloud/branch-github-integration.md)