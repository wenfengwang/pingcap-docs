---
title: リストアリソースの使用
summary: リストアリソースの使用方法を学ぶ。
---

# リストアリソースの使用

このドキュメントでは、`tidbcloud_restore`リソースを使ってリストアタスクを管理する方法について学ぶことができます。

`tidbcloud_restore`リソースの機能には以下のようなものが含まれます：

- バックアップとおりにTiDB専用クラスターのリストアタスクを作成する。

## 前提条件

- [TiDB Cloud Terraformプロバイダーの取得](/tidb-cloud/terraform-get-tidbcloud-provider.md)。
- バックアップとリストア機能は、TiDB Serverlessクラスターでは利用できません。リストアリソースを使用する場合、TiDB専用クラスターが作成されていることを確認してください。

## リストアタスクを作成する

クラスターのバックアップを作成した後、`tidbcloud_restore`リソースを使用してリストアタスクを作成することにより、クラスターをリストアできます。

> **注意:**
>
> データは、より小さいノードサイズから同じ、あるいはより大きいノードサイズへのみリストアできます。

1. リストア用のディレクトリを作成し、そこに入る。

2. `restore.tf`ファイルを作成する。

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
    resource "tidbcloud_restore" "example_restore" {
      project_id = tidbcloud_cluster.example_cluster.project_id
      backup_id  = tidbcloud_backup.example_backup.id
      name       = "restoreCluster"
      config = {
        root_password = "Your_root_password1."
        port          = 4000
        components = {
          tidb = {
            node_size : "8C16G"
            node_quantity : 2
          }
          tikv = {
            node_size : "8C32G"
            storage_size_gib : 500
            node_quantity : 6
          }
          tiflash = {
            node_size : "8C64G"
            storage_size_gib : 500
            node_quantity : 2
          }
        }
      }
    }
    ```

3. `terraform apply`コマンドを実行し、確認のために`yes`と入力する：

    ```
    $ terraform apply
    tidbcloud_cluster.example_cluster: Refreshing state... [id=1379661944630234067]
    tidbcloud_backup.example_backup: Refreshing state... [id=1350048]

    Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
      + create

    Terraform will perform the following actions:

      # tidbcloud_restore.example_restore will be created
      + resource "tidbcloud_restore" "example_restore" {
          + backup_id        = "1350048"
          + cluster          = {
              + id     = (known after apply)
              + name   = (known after apply)
              + status = (known after apply)
            }
          + cluster_id       = (known after apply)
          + config           = {
              + components    = {
                  + tidb    = {
                      + node_quantity = 2
                      + node_size     = "8C16G"
                    }
                  + tiflash = {
                      + node_quantity    = 2
                      + node_size        = "8C64G"
                      + storage_size_gib = 500
                    }
                  + tikv    = {
                      + node_quantity    = 6
                      + node_size        = "8C32G"
                      + storage_size_gib = 500
                    }
                }
              + port          = 4000
              + root_password = "Your_root_password1."
            }
          + create_timestamp = (known after apply)
          + error_message    = (known after apply)
          + id               = (known after apply)
          + name             = "restoreCluster"
          + project_id       = "1372813089189561287"
          + status           = (known after apply)
        }

    Plan: 1 to add, 0 to change, 0 to destroy.

    Do you want to perform these actions?
      Terraform will perform the actions described above.
      Only 'yes' will be accepted to approve.

      Enter a value: yes

    tidbcloud_restore.example_restore: Creating...
    tidbcloud_restore.example_restore: Creation complete after 1s [id=780114]

    Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
    ```

4. `terraform state show tidbcloud_restore.${resource-name}`コマンドを使用して、リストアタスクのステータスを確認する：

    ```
    $ terraform state show tidbcloud_restore.example_restore

    # tidbcloud_restore.example_restore:
    resource "tidbcloud_restore" "example_restore" {
        backup_id        = "1350048"
        cluster          = {
            id     = "1379661944630264072"
            name   = "restoreCluster"
            status = "INITIALIZING"
        }
        cluster_id       = "1379661944630234067"
        config           = {
            components    = {
                tidb    = {
                    node_quantity = 2
                    node_size     = "8C16G"
                }
                tiflash = {
                    node_quantity    = 2
                    node_size        = "8C64G"
                    storage_size_gib = 500
                }
                tikv    = {
                    node_quantity    = 6
                    node_size        = "8C32G"
                    storage_size_gib = 500
                }
            }
            port          = 4000
            root_password = "Your_root_password1."
        }
        create_timestamp = "2022-08-26T08:16:33Z"
        id               = "780114"
        name             = "restoreCluster"
        project_id       = "1372813089189561287"
        status           = "PENDING"
    }
    ```

    リストアタスクのステータスが`PENDING`であり、クラスターのステータスが`INITIALIZING`であることがわかります。

5. 数分待ってから、`terraform refersh`を使ってステータスを更新します。

6. クラスターのステータスが`AVAILABLE`に変わった後、リストアタスクは`RUNNING`になり、最終的には`SUCCESS`になります。

リストアされたクラスターはTerraformによって管理されていないことに注意してください。[インポートする](/tidb-cloud/terraform-use-cluster-resource.md#import-a-cluster)ことでリストアされたクラスターを管理できます。

## リストアタスクの更新

リストアタスクは更新できません。

## リストアタスクの削除

リストアタスクは削除できません。