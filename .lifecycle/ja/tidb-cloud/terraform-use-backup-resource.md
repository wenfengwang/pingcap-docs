---
title: バックアップリソースの使用
summary: バックアップリソースを使用して TiDB Cloud クラスターのバックアップの作成方法を学ぶ

# バックアップリソースの使用

このドキュメントでは、`tidbcloud_backup` リソースを使用して TiDB Cloud クラスターのバックアップを作成する方法を学ぶことができます。

`tidbcloud_backup` リソースの機能には、以下が含まれます：

- TiDB Dedicated クラスターのバックアップを作成する。
- TiDB Dedicated クラスターのバックアップを削除する。

## 前提条件

- [TiDB Cloud Terraform プロバイダを取得](/tidb-cloud/terraform-get-tidbcloud-provider.md) します。
- バックアップとリストア機能は TiDB Serverless クラスターでは利用できません。バックアップリソースを使用するには、TiDB Dedicated クラスターを作成していることを確認してください。

## バックアップリソースを使用してバックアップを作成する

1. バックアップのためのディレクトリを作成し、そのディレクトリに移動します。

2. `backup.tf` ファイルを作成します。

    例：

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
    resource "tidbcloud_backup" "example_backup" {
      project_id  = "1372813089189561287"
      cluster_id  = "1379661944630234067"
      name        = "firstBackup"
      description = "create by terraform"
    }
    ```

    ファイル内のリソースの値（プロジェクト ID やクラスター ID など）を自分自身のもので置き換える必要があります。

    Terraform を使用してクラスターリソース（たとえば `example_cluster`）を維持している場合、実際のプロジェクト ID やクラスター ID を指定せずに、以下のようにバックアップリソースを構成することもできます。

    ```
    resource "tidbcloud_backup" "example_backup" {
      project_id  = tidbcloud_cluster.example_cluster.project_id
      cluster_id  = tidbcloud_cluster.example_cluster.id
      name        = "firstBackup"
      description = "create by terraform"
    }
    ```

3. `terraform apply` コマンドを実行します：

    ```
    $ terraform apply

    tidbcloud_cluster.example_cluster: Refreshing state... [id=1379661944630234067]

    Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
      + create

    Terraform will perform the following actions:

      # tidbcloud_backup.example_backup will be created
      + resource "tidbcloud_backup" "example_backup" {
          + cluster_id       = "1379661944630234067"
          + create_timestamp = (apply 後にわかる)
          + description      = "create by terraform"
          + id               = (apply 後にわかる)
          + name             = "firstBackup"
          + project_id       = "1372813089189561287"
          + size             = (apply 後にわかる)
          + status           = (apply 後にわかる)
          + type             = (apply 後にわかる)
        }

    Plan: 1 to add, 0 to change, 0 to destroy.

    これらのアクションを実行しますか？
      上記のアクションを Terraform が実行します。
      承認するには 'yes' のみが受け入れられます。

      値を入力してください：
    ```

4. バックアップを作成するには `yes` と入力します：

    ```
      Enter a value: yes

    tidbcloud_backup.example_backup: Creating...
    tidbcloud_backup.example_backup: Creation complete after 2s [id=1350048]

    Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

    ```

5. バックアップのステータスを確認するには `terraform state show tidbcloud_backup.${resource-name}` を使用します：

    ```
    $ terraform state show tidbcloud_backup.example_backup

    # tidbcloud_backup.example_backup:
    resource "tidbcloud_backup" "example_backup" {
        cluster_id       = "1379661944630234067"
        create_timestamp = "2022-08-26T07:56:10Z"
        description      = "create by terraform"
        id               = "1350048"
        name             = "firstBackup"
        project_id       = "1372813089189561287"
        size             = "0"
        status           = "PENDING"
        type             = "MANUAL"
    }
    ```

6. 数分待ち、`terraform refersh` を使用してステータスを更新します：

    ```
    $ terraform refresh
    tidbcloud_cluster.example_cluster: Refreshing state... [id=1379661944630234067]
    tidbcloud_backup.example_backup: Refreshing state... [id=1350048]
    $ terraform state show tidbcloud_backup.example_backup
    # tidbcloud_backup.example_backup:
    resource "tidbcloud_backup" "example_backup" {
        cluster_id       = "1379661944630234067"
        create_timestamp = "2022-08-26T07:56:10Z"
        description      = "create by terraform"
        id               = "1350048"
        name             = "firstBackup"
        project_id       = "1372813089189561287"
        size             = "198775"
        status           = "SUCCESS"
        type             = "MANUAL"
    }
    ```

ステータスが「SUCCESS」に変わると、クラスターのバックアップが作成されたことを示します。バックアップは作成後に更新できないことに注意してください。

これで、クラスターのためのバックアップが作成されました。バックアップを使用してクラスターを復元したい場合は、[リストアリソースを使用](/tidb-cloud/terraform-use-restore-resource.md) することができます。

## バックアップの更新

バックアップは更新できません。

## バックアップの削除

バックアップを削除するには、対応する `backup.tf` ファイルがあるバックアップディレクトリに移動し、`terraform destroy` コマンドを実行します。

```
$ terraform destroy

Plan: 0 to add, 0 to change, 1 to destroy.

本当にすべてのリソースを破棄しますか？
Terraform は上記のように管理されているすべてのインフラストラクチャを破壊します。
元に戻せる方法はありません。確認するには 'yes' のみが受け入れられます。

値を入力してください：yes
```

これで、`terraform show` コマンドを実行すると、リソースがクリアされているため何も表示されなくなります：

```
$ terraform show
```