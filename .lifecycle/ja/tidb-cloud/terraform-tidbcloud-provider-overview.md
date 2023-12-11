---
title: Terraformの統合概要
summary: Terraformを使用してTiDB Cloudリソースを作成、管理、および更新します。

# Terraformの統合概要

[Terraform](https://www.terraform.io/)は、クラウドとセルフホストリソースの両方を人間が読める構成ファイルで定義し、バージョンを管理、再利用、共有できるインフラストラクチャ・コードのツールです。

[TiDB Cloud Terraform Provider](https://registry.terraform.io/providers/tidbcloud/tidbcloud)は、クラスタ、バックアップ、およびリストアなどのTiDB CloudリソースをTerraformで管理できるプラグインです。

リソースのプロビジョニングやインフラストラクチャ・ワークフローを自動化する簡単な方法をお探しの場合は、以下の機能を提供するTiDB Cloud Terraform Providerをお試しください。

- プロジェクト情報の取得
- サポートされるクラウドプロバイダ、リージョン、およびノードサイズなどのクラスタ仕様情報の取得
- クラスタの作成、スケーリング、中断、再開などのTiDBクラスタの管理
- クラスタのバックアップの作成と削除
- クラスタのリストアタスクの作成

## 要件

- [TiDB Cloudアカウント](https://tidbcloud.com/free-trial)
- [Terraformのバージョン](https://www.terraform.io/downloads.html) >= 1.0
- [Goのバージョン](https://golang.org/doc/install) >= 1.18（[TiDB Cloud Terraform Provider](https://github.com/tidbcloud/terraform-provider-tidbcloud)をローカルで構築する場合のみ必要）

## サポートされているリソースとデータソース

[リソース](https://www.terraform.io/language/resources)と[データソース](https://www.terraform.io/language/data-sources)は、Terraform言語の2つの重要な要素です。

TiDB Cloudは、次のリソースおよびデータソースをサポートしています。

- リソース

    - `tidbcloud_cluster`
    - `tidbcloud_backup`
    - `tidbcloud_restore`
    - `tidbcloud_import`

- データソース

    - `tidbcloud_projects`
    - `tidbcloud_cluster_specs`
    - `tidbcloud_clusters`
    - `tidbcloud_restores`
    - `tidbcloud_backups`

リソースとデータソースの利用可能な構成については、[構成ドキュメント](https://registry.terraform.io/providers/tidbcloud/tidbcloud/latest/docs)を参照してください。

## 次のステップ

- [Terraformの詳細を学ぶ](https://www.terraform.io/docs)
- [TiDB Cloud Terraform Providerを入手する](/tidb-cloud/terraform-get-tidbcloud-provider.md)
- [クラスタリソースの使用方法](/tidb-cloud/terraform-use-cluster-resource.md)
- [バックアップリソースの使用方法](/tidb-cloud/terraform-use-backup-resource.md)
- [リストアリソースの使用方法](/tidb-cloud/terraform-use-restore-resource.md)