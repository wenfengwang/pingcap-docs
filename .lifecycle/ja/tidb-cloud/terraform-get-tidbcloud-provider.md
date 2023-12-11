---
title: TiDB Cloud Terraform Providerの取得
summary: TiDB Cloud Terraform Providerの取得方法を学びます。

# TiDB Cloud Terraform Providerの取得

このドキュメントではTiDB Cloud Terraform Providerの取得方法について学びます。

## 前提条件

[TiDB Cloud Terraform Providerの概要](/tidb-cloud/terraform-tidbcloud-provider-overview.md#requirements)に記載されている要件を満たしていることを確認してください。

## ステップ1. Terraformのインストール

TiDB Cloud Terraform Providerは[Terraform Registry](https://registry.terraform.io/)にリリースされています。必要なのはTerraform (>=1.0)をインストールするだけです。

macOSの場合、以下の手順に従ってHomebrewでTerraformをインストールできます。

1. 必要なHomebrewパッケージがすべて含まれるリポジトリであるHashiCorp tapをインストールします。

    ```shell
    brew tap hashicorp/tap
    ```

2. `hashicorp/tap/terraform`を使用してTerraformをインストールします。

    ```shell
    brew install hashicorp/tap/terraform
    ```

他のオペレーティングシステムの場合は、[Terraformのドキュメント](https://learn.hashicorp.com/tutorials/terraform/install-cli)に記載された手順を参照してください。

## ステップ2. APIキーの作成

TiDB Cloud APIはHTTP Digest Authenticationを使用しています。これにより、プライベートキーがネットワーク上で送信されることを防ぎます。

現時点では、TiDB Cloud Terraform ProviderではAPIキーの管理がサポートされていません。そのため、[TiDB Cloudコンソール](https://tidbcloud.com/console/clusters)でAPIキーを作成する必要があります。

詳細な手順については、[TiDB Cloud APIのドキュメント](https://docs.pingcap.com/tidbcloud/api/v1beta#section/Authentication/API-Key-Management)を参照してください。

## ステップ3. TiDB Cloud Terraform Providerのダウンロード

1. `main.tf`ファイルを作成します:

   ```
   terraform {
     required_providers {
       tidbcloud = {
         source = "tidbcloud/tidbcloud"
         version = "~> 0.1.0"
       }
     }
     required_version = ">= 1.0.0"
   }
   ```

   - `source`属性は[Terraform Registry](https://registry.terraform.io/)からダウンロードするターゲットTerraformプロバイダを指定します。
   - `version`属性はオプションであり、Terraformプロバイダのバージョンを指定します。指定しない場合、デフォルトで最新のプロバイダバージョンが使用されます。
   - `required_version`はオプションであり、Terraformのバージョンを指定します。指定しない場合、デフォルトで最新のTerraformバージョンが使用されます。

2. `terraform init`コマンドを実行して、Terraform RegistryからTiDB Cloud Terraform Providerをダウンロードします。

   ```
   $ terraform init

   バックエンドを初期化中...

   プロバイダープラグインを初期化中...
   - 依存関係ロックファイルからtidbcloud/tidbcloudの以前のバージョンを再利用しています
   - 以前にインストールされたtidbcloud/tidbcloud v0.1.0を使用しています

   Terraformの初期化が成功しました！

   これでTerraformで作業を開始できます。"terraform plan"を実行して、インフラストラクチャに必要な変更を確認してみてください。すべてのTerraformコマンドが動作するはずです。

   もしTerraformのモジュールやバックエンド構成を設定したり変更したりした場合は、作業ディレクトリを再初期化するためにこのコマンドを再実行してください。これを忘れた場合は、他のコマンドがそれを検出し、必要に応じて再実行するように促すでしょう。
   ```

## ステップ4. APIキーを使用したTiDB Cloud Terraform Providerの構成

次のように`main.tf`ファイルを構成できます:

```
terraform {
  required_providers {
    tidbcloud = {
      source = "tidbcloud/tidbcloud"
    }
  }
}

provider "tidbcloud" {
  public_key = "your_public_key"
  private_key = "your_private_key"
}
```

`public_key`と`private_key`はAPIキーの公開キーと秘密キーです。環境変数を介してそれらを渡すこともできます:

```
export TIDBCLOUD_PUBLIC_KEY = ${public_key}
export TIDBCLOUD_PRIVATE_KEY = ${private_key}
```

これでTiDB Cloud Terraform Providerを使用できます。

## 次の手順

[クラスタリソース](/tidb-cloud/terraform-use-cluster-resource.md)を使用してクラスタを管理することで、始める準備が整いました。