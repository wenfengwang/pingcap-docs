---
title: クラスターリソースの使用
summary: TiDB Cloudクラスターの作成と変更にクラスターリソースを使用する方法を学びます。

# クラスターリソースの使用

このドキュメントでは、`tidbcloud_cluster`リソースを使用してTiDB Cloudクラスターを管理する方法を学ぶことができます。

さらに、`tidbcloud_projects`および`tidbcloud_cluster_specs`データソースを使用して必要な情報を取得する方法も学ぶことができます。

`tidbcloud_cluster`リソースの機能には、次のものがあります。

- TiDB ServerlessおよびTiDB Dedicatedクラスターの作成。
- TiDB Dedicatedクラスターの変更。
- TiDB ServerlessおよびTiDB Dedicatedクラスターの削除。

## 必要条件

- [TiDB Cloud Terraformプロバイダーの取得](/tidb-cloud/terraform-get-tidbcloud-provider.md)。

## `tidbcloud_projects`データソースを使用してプロジェクトIDを取得

各TiDBクラスターはプロジェクトに属しています。TiDBクラスターを作成する前に、作成するクラスターが属するプロジェクトのIDを取得する必要があります。

利用可能なすべてのプロジェクトの情報を表示するには、次のように`tidbcloud_projects`データソースを使用できます。

1. [TiDB Cloud Terraformプロバイダーの取得](/tidb-cloud/terraform-get-tidbcloud-provider.md) で作成される`main.tf`ファイルに、次のように`data`および`output`ブロックを追加します。

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

   data "tidbcloud_projects" "example_project" {
     page      = 1
     page_size = 10
   }

   output "projects" {
     value = data.tidbcloud_projects.example_project.items
   }
   ```

   - `data`ブロックを使用して、TiDB Cloudのデータソースの種類およびデータソース名を含むデータソースを定義します。

      - プロジェクトデータソースを使用する場合は、データソースの種類を`tidbcloud_projects`に設定します。
      - データソース名は、必要に応じて定義できます。例えば、「example_project」という名前を定義できます。
      - `tidbcloud_projects`データソースでは、`page`および`page_size`属性を使用して、チェックするプロジェクトの最大数を制限できます。

   - `output`ブロックを使用して、出力に表示するデータソース情報を定義し、他のTerraform構成で使用するための情報を公開します。

      `output`ブロックは、プログラミング言語の返り値と同様に機能します。詳細については、[Terraformのドキュメント](https://www.terraform.io/language/values/outputs)を参照してください。

   リソースとデータソースの利用可能なすべての構成については、[構成ドキュメント](https://registry.terraform.io/providers/tidbcloud/tidbcloud/latest/docs)を参照してください。

2. 構成を適用するには、`terraform apply`コマンドを実行します。確認プロンプトで`yes`を入力する必要があります。

   プロンプトをスキップするには、`terraform apply --auto-approve`を使用してください。

   ```
   $ terraform apply --auto-approve

   Outputs:
     + projects = [
         + {
             + cluster_count    = 0
             + create_timestamp = "1649154426"
             + id               = "1372813089191121286"
             + name             = "test1"
             + org_id           = "1372813089189921287"
             + user_count       = 1
           },
         + {
             + cluster_count    = 1
             + create_timestamp = "1640602740"
             + id               = "1372813089189561287"
             + name             = "default project"
             + org_id           = "1372813089189921287"
             + user_count       = 1
           },
       ]

   このプランを適用して、これらの新しい出力値をTerraformステートに保存し、実際のインフラストラクチャを変更しないようにすることができます。

   Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

   Outputs:

   projects = tolist([
     {
       "cluster_count" = 0
       "create_timestamp" = "1649154426"
       "id" = "1372813089191121286"
       "name" = "test1"
       "org_id" = "1372813089189921287"
       "user_count" = 1
     },
     {
       "cluster_count" = 1
       "create_timestamp" = "1640602740"
       "id" = "1372813089189561287"
       "name" = "default project"
       "org_id" = "1372813089189921287"
       "user_count" = 1
     },
   ])
   ```

これで、出力から利用可能なすべてのプロジェクトを取得できます。必要なプロジェクトのIDをコピーしてください。

## `tidbcloud_cluster_specs`データソースを使用してクラスター仕様情報を取得

クラスターを作成する前に、利用可能な構成値（サポートされているクラウドプロバイダー、リージョン、ノードサイズなど）を含むクラスター仕様情報を取得する必要があります。

クラスター仕様情報を取得するには、次のように`tidbcloud_cluster_specs`データソースを使用できます。

1. `main.tf`ファイルを次のように編集します。

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

    data "tidbcloud_cluster_specs" "example_cluster_spec" {
    }

    output "cluster_spec" {
      value = data.tidbcloud_cluster_specs.example_cluster_spec.items
    }
    ```

2. `terraform apply --auto-approve`コマンドを実行して、クラスター仕様情報を取得します。

    参照のために、例の一部を取得するには、以下の行をクリックしてください。

    <details>
      <summary>クラスター仕様</summary>

    ```
    {
        "cloud_provider" = "AWS"
        "cluster_type" = "DEDICATED"
        "region" = "eu-central-1"
        "tidb" = tolist([
          {
            "node_quantity_range" = {
              "min" = 1
              "step" = 1
            }
            "node_size" = "2C8G"
          },
          {
            "node_quantity_range" = {
              "min" = 1
              "step" = 1
            }
            "node_size" = "4C16G"
          },
          {
            "node_quantity_range" = {
              "min" = 1
              "step" = 1
            }
            "node_size" = "8C16G"
          },
          {
            "node_quantity_range" = {
              "min" = 1
              "step" = 1
            }
            "node_size" = "16C32G"
          },
        ])
        "tiflash" = tolist([
          {
            "node_quantity_range" = {
              "min" = 0
              "step" = 1
            }
            "node_size" = "8C64G"
            "storage_size_gib_range" = {
              "max" = 2048
              "min" = 500
            }
          },
          {
            "node_quantity_range" = {
              "min" = 0
              "step" = 1
            }
            "node_size" = "16C128G"
            "storage_size_gib_range" = {
              "max" = 2048
              "min" = 500
            }
          },
        ])
        "tikv" = tolist([
          {
            "node_quantity_range" = {
              "min" = 3
              "step" = 3
            }
            "node_size" = "2C8G"
            "storage_size_gib_range" = {
              "max" = 500
              "min" = 200
            }
          },
          {
            "node_quantity_range" = {
              "min" = 3
              "step" = 3
            }
            "node_size" = "4C16G"
            "storage_size_gib_range" = {
              "max" = 2048
              "min" = 200
            }
          },
          {
            "node_quantity_range" = {
              "min" = 3
              "step" = 3
            }
            "node_size" = "8C32G"
            "storage_size_gib_range" = {
              "max" = 4096
              "min" = 500
            }
          },
          {
            "node_quantity_range" = {
              "min" = 3
              "step" = 3
            }
            "node_size" = "8C64G"
            "storage_size_gib_range" = {
              "max" = 4096
              "min" = 500
            }
          },
          {
            "node_quantity_range" = {
              "min" = 3
              "step" = 3
            }
            "node_size" = "16C64G"
            "storage_size_gib_range" = {
```
"最大" = 4096
              "最小" = 500
            }
          },
        ])
      }
    ```

    </details>

- `cloud_provider` は TiDB クラスタをホストできるクラウド プロバイダです。
- `region` は `cloud_provider` のリージョンです。
- `node_quantity_range` はノード数の最小値とステップを示しています。
- `node_size` はノードのサイズです。
- `storage_size_gib_range` はノードに設定できるストレージの最小値および最大値を示しています。

## クラスタ リソースを使用してクラスタを作成する

> **ノート:**
>
> 開始する前に、TiDB Cloud コンソールでプロジェクト CIDR を設定していることを確認してください。詳細については、[プロジェクト CIDR を設定する](/tidb-cloud/set-up-vpc-peering-connections.md#prerequisite-set-a-project-cidr) を参照してください。

`tidbcloud_cluster` リソースを使用してクラスタを作成できます。

次の例は、TiDB デディケーテッド クラスタを作成する方法を示しています。

1. クラスタ用にディレクトリを作成し、そのディレクトリに移動します。

2. `cluster.tf` ファイルを作成します:

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

    resource "tidbcloud_cluster" "example_cluster" {
      project_id     = "1372813089189561287"
      name           = "firstCluster"
      cluster_type   = "DEDICATED"
      cloud_provider = "AWS"
      region         = "eu-central-1"
      config = {
        root_password = "Your_root_password1."
        port = 4000
        components = {
          tidb = {
            node_size : "8C16G"
            node_quantity : 1
          }
          tikv = {
            node_size : "8C32G"
            storage_size_gib : 500,
            node_quantity : 3
          }
        }
      }
    }
    ```

    `resource` ブロックを使用して TiDB Cloud のリソースを定義します。これにはリソースタイプ、リソース名、リソースの詳細が含まれます。

    - クラスタ リソースを使用するには、リソースタイプを `tidbcloud_cluster` に設定します。
    - リソース名については、自分の必要に応じて定義できます。例: `example_cluster` など。
    - リソースの詳細は、プロジェクト ID とクラスタの仕様情報に応じて構成できます。

3. `terraform apply` コマンドを実行します。リソースを適用する際には `terraform apply --auto-approve` を使用しないことをお勧めします。

    ```shell
    $ terraform apply

    Terraform は以下のアクションを実行します:

      # tidbcloud_cluster.example_cluster が作成されます
      + resource "tidbcloud_cluster" "example_cluster" {
          + cloud_provider = "AWS"
          + cluster_type   = "DEDICATED"
          + config         = {
              + components     = {
                  + tidb = {
                      + node_quantity = 1
                      + node_size     = "8C16G"
                    }
                  + tikv = {
                      + node_quantity    = 3
                      + node_size        = "8C32G"
                      + storage_size_gib = 500
                    }
                }
              + ip_access_list = [
                  + {
                      + cidr        = "0.0.0.0/0"
                      + description = "all"
                    },
                ]
              + port           = 4000
              + root_password  = "Your_root_password1."
            }
          + id             = (known after apply)
          + name           = "firstCluster"
          + project_id     = "1372813089189561287"
          + region         = "eu-central-1"
          + status         = (known after apply)
        }

    プラン: 1 作成, 0 変更, 0 削除します。

    これらのアクションを実行しますか？
      Terraform は上記のアクションを実行します。
      承認は 'yes' のみが受け入れられます。

      値を入力:
    ```

   上記の結果のように、Terraform は実行計画を生成し、Terraform が実行するアクションを記述します:

   - 構成と状態の違いを確認できます。
   - この `apply` の結果を確認できます。新しいリソースを追加し、リソースの変更や削除は行われません。
   - `apply` 後に値を取得することがわかります。

4. プランに問題がない場合は、`yes` と入力して続行します:

    ```
    これらのアクションを実行しますか？
      Terraform は上記のアクションを実行します。
      承認は 'yes' のみが受け入れられます。

      値を入力: yes

    tidbcloud_cluster.example_cluster: 作成中...
    tidbcloud_cluster.example_cluster: 1s 後に作成が完了しました [id=1379661944630234067]

    適用完了! リソース: 1 追加, 0 変更, 0 削除。
    ```

5. `terraform show` または `terraform state show tidbcloud_cluster.${resource-name}` コマンドを使用してリソースの状態を調査します。前者はすべてのリソースとデータソースの状態を表示します。

    ```shell
    $ terraform state show tidbcloud_cluster.example_cluster

    # tidbcloud_cluster.example_cluster:
    resource "tidbcloud_cluster" "example_cluster" {
        cloud_provider = "AWS"
        cluster_type   = "DEDICATED"
        config         = {
            components     = {
                tidb = {
                    node_quantity = 1
                    node_size     = "8C16G"
                }
                tikv = {
                    node_quantity    = 3
                    node_size        = "8C32G"
                    storage_size_gib = 500
                }
            }
            ip_access_list = [
                # (1 unchanged element hidden)
            ]
            port           = 4000
            root_password  = "Your_root_password1."
        }
        id             = "1379661944630234067"
        name           = "firstCluster"
        project_id     = "1372813089189561287"
        region         = "eu-central-1"
        status         = "CREATING"
    }
    ```

    クラスタのステータスは `CREATING` です。この場合、一般に少なくとも 10 分かかることが多いため、`AVAILABLE` に変更されるまで待機する必要があります。

6. 最新のステータスを確認したい場合は、`terraform refresh` コマンドを実行して状態を更新し、その後 `terraform state show tidbcloud_cluster.${resource-name}` コマンドを実行して状態を表示します。

    ```
    $ terraform refresh

    tidbcloud_cluster.example_cluster: 状態を更新中... [id=1379661944630234067]

    $ terraform state show tidbcloud_cluster.example_cluste

    # tidbcloud_cluster.example_cluster:
    resource "tidbcloud_cluster" "example_cluster" {
        cloud_provider = "AWS"
        cluster_type   = "DEDICATED"
        config         = {
            components     = {
                tidb = {
                    node_quantity = 1
                    node_size     = "8C16G"
                }
                tikv = {
                    node_quantity    = 3
                    node_size        = "8C32G"
                    storage_size_gib = 500
                }
            }
            ip_access_list = [
                # (1 unchanged element hidden)
            ]
            port           = 4000
            root_password  = "Your_root_password1."
        }
        id             = "1379661944630234067"
        name           = "firstCluster"
        project_id     = "1372813089189561287"
        region         = "eu-central-1"
        status         = "AVAILABLE"
    }
    ```

ステータスが `AVAILABLE` の場合、TiDB クラスタが作成されて使用可能であることを示します。

## TiDB デディケーテッド クラスタを変更する

TiDB デディケーテッド クラスタでは、次のように、Terraform を使用してクラスタ リソースを管理できます:

- クラスタに TiFlash コンポーネントを追加します。
- クラスタをスケーリングします。
- クラスタを一時停止または再開します。

### TiFlash コンポーネントを追加する

1. [クラスタを作成する](#クラスタリソースを使用してクラスタを作成する) ときに使用する `cluster.tf` ファイルに、`components` フィールドに `tiflash` の構成を追加します。

    例:

    ```
        components = {
          tidb = {
            node_size : "8C16G"
            node_quantity : 1
          }
          tikv = {
            node_size : "8C32G"
            storage_size_gib : 500
            node_quantity : 3
          }
          tiflash = {
            node_size : "8C64G"
            storage_size_gib : 500
            node_quantity : 1
          }
        }
    ```

2. `terraform apply` コマンドを実行します:

    ```
    $ terraform apply

    tidbcloud_cluster.example_cluster: 状態を更新中... [id=1379661944630234067]

    Terraform は選択したプロバイダを使用して、次の実行計画を生成しました。リソースアクションは次のシンボルで示されます:
      ~ in-place で更新

    Terraform は以下のアクションを実行します:
```
```
      # tidbcloud_cluster.example_cluster will be updated in-place
      ~ resource "tidbcloud_cluster" "example_cluster" {
          ~ config         = {
              ~ components     = {
                  + tiflash = {
                      + node_quantity    = 1
                      + node_size        = "8C64G"
                      + storage_size_gib = 500
                    }
                    # (2 unchanged attributes hidden)
                }
                # (3 unchanged attributes hidden)
            }
            id             = "1379661944630234067"
            name           = "firstCluster"
          ~ status         = "AVAILABLE" -> (known after apply)
            # (4 unchanged attributes hidden)
        }

    Plan: 0 to add, 1 to change, 0 to destroy.

    Do you want to perform these actions?
      Terraform will perform the actions described above.
      Only 'yes' will be accepted to approve.

      Enter a value:

    ```

    上記の実行計画によると、TiFlashが追加され、１つのリソースが変更されます。

3. プランに問題がない場合は、続行するには `yes` と入力してください：

    ```
      Enter a value: yes

    tidbcloud_cluster.example_cluster: 修正中... [id=1379661944630234067]
    tidbcloud_cluster.example_cluster: 修正が完了しました after 2s [id=1379661944630234067]

    Apply 完了! リソース 0 追加済み、1 変更済み、0 削除済み。
    ```

4. ステータスを確認するには `terraform state show tidbcloud_cluster.${resource-name}` を使用してください：

    ```
    $ terraform state show tidbcloud_cluster.example_cluster

    # tidbcloud_cluster.example_cluster:
    resource "tidbcloud_cluster" "example_cluster" {
        cloud_provider = "AWS"
        cluster_type   = "DEDICATED"
        config         = {
            components     = {
                tidb    = {
                    node_quantity = 1
                    node_size     = "8C16G"
                }
                tiflash = {
                    node_quantity    = 1
                    node_size        = "8C64G"
                    storage_size_gib = 500
                }
                tikv    = {
                    node_quantity    = 3
                    node_size        = "8C32G"
                    storage_size_gib = 500
                }
            }
            ip_access_list = [
                # (1 unchanged element hidden)
            ]
            port           = 4000
            root_password  = "Your_root_password1."
        }
        id             = "1379661944630234067"
        name           = "firstCluster"
        project_id     = "1372813089189561287"
        region         = "eu-central-1"
        status         = "MODIFYING"
    }
    ```

    `MODIFYING` ステータスは、現在クラスタが変更中であることを示しています。しばらくお待ちください。ステータスは `AVAILABLE` に変わります。

### TiDB クラスタのスケーリング

TiDB クラスタのステータスが `AVAILABLE` の場合にスケールすることができます。

1. [クラスタを作成](#create-a-cluster-using-the-cluster-resource) する際に使用される `cluster.tf` ファイルにて、`components` の構成を編集します。

    たとえば、TiDB に1つのノードを追加し、TiKV に3つのノードを追加（TiKV ノードの数はそのステップが3であるため、そのステップが3である必要があります。[クラスタの仕様からこの情報を取得](#get-cluster-specification-information-using-the-tidbcloud_cluster_specs-data-source)することができます。）し、TiFlash に1つのノードを追加するには、次のように構成を編集します：

   ```
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
   ```

2. `terraform apply` コマンドを実行し、確認のために `yes` と入力してください：

   ```
   $ terraform apply

   tidbcloud_cluster.example_cluster: 状態を更新中... [id=1379661944630234067]

   Terraform は選択したプロバイダーを使用して、次の実行計画を生成しました。リソースアクションは次のシンボルで示されています：
     ~ in-place で更新

   Terraform は以下のアクションを実行します:

     # tidbcloud_cluster.example_cluster will be updated in-place
     ~ resource "tidbcloud_cluster" "example_cluster" {
         ~ config         = {
             ~ components     = {
                 ~ tidb    = {
                     ~ node_quantity = 1 -> 2
                       # (1 unchanged attribute hidden)
                   }
                 ~ tiflash = {
                     ~ node_quantity    = 1 -> 2
                       # (2 unchanged attributes hidden)
                   }
                 ~ tikv    = {
                     ~ node_quantity    = 3 -> 6
                       # (2 unchanged attributes hidden)
                   }
               }
               # (3 unchanged attributes hidden)
           }
           id             = "1379661944630234067"
           name           = "firstCluster"
         ~ status         = "AVAILABLE" -> (known after apply)
           # (4 unchanged attributes hidden)
       }

   Plan: 0 to add, 1 to change, 0 to destroy.

   これらのアクションを実行しますか？
     Terraform は上記のように記載されたアクションを実行します。
     承認するには 'yes' のみが受け入れられます。

     Enter a value: yes

   tidbcloud_cluster.example_cluster: 修正中... [id=1379661944630234067]
   tidbcloud_cluster.example_cluster: 修正が完了しました after 2s [id=1379661944630234067]

   Apply 完了! リソース 0 追加済み、1 変更済み、0 削除済み。
   ```

ステータスが `MODIFYING` から `AVAILABLE` に変わるのを待ってください。

### クラスタの一時停止または再開

クラスタのステータスが `AVAILABLE` の場合は、クラスタを一時停止することができます。また、ステータスが `PAUSED` の場合は、クラスタを再開することができます。

- `paused = true` を設定してクラスタを一時停止します。
- `paused = false` を設定してクラスタを再開します。

1. [クラスタを作成](#create-a-cluster-using-the-cluster-resource) する際に使用される `cluster.tf` ファイルに、`config` の構成に `pause = true` を追加します：

   ```
   config = {
       paused = true
       root_password = "Your_root_password1."
       port          = 4000
       ...
     }
   ```

2. `terraform apply` コマンドを実行し、確認後に `yes` と入力してください：

   ```
   $ terraform apply

   tidbcloud_cluster.example_cluster: 状態を更新中... [id=1379661944630234067]

   Terraform は選択したプロバイダーを使用して、次の実行計画を生成しました。リソースアクションは次のシンボルで示されています：
     ~ in-place で更新

   Terraform は以下のアクションを実行します:

     # tidbcloud_cluster.example_cluster will be updated in-place
     ~ resource "tidbcloud_cluster" "example_cluster" {
         ~ config         = {
             + paused         = true
               # (4 unchanged attributes hidden)
           }
           id             = "1379661944630234067"
           name           = "firstCluster"
         ~ status         = "AVAILABLE" -> (known after apply)
           # (4 unchanged attributes hidden)
       }

   Plan: 0 to add, 1 to change, 0 to destroy.

   これらのアクションを実行しますか？
     Terraform は上記のように記載されたアクションを実行します。
     承認するには 'yes' のみが受け入れられます。

     Enter a value: yes

   tidbcloud_cluster.example_cluster: 修正中... [id=1379661944630234067]
   tidbcloud_cluster.example_cluster: 修正が完了しました after 2s [id=1379661944630234067]

   Apply 完了! リソース 0 追加済み、1 変更済み、0 削除済み。
   ```

3. ステータスを確認するには `terraform state show tidbcloud_cluster.${resource-name}` コマンドを使用してください：

   ```
   $ terraform state show tidbcloud_cluster.example_cluster

   # tidbcloud_cluster.example_cluster:
   resource "tidbcloud_cluster" "example_cluster" {
       cloud_provider = "AWS"
       cluster_type   = "DEDICATED"
       config         = {
           components     = {
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
           ip_access_list = [
               # (1 unchanged element hidden)
           ]
           paused         = true
           port           = 4000
           root_password  = "Your_root_password1."
       }
       id             = "1379661944630234067"
       name           = "firstCluster"
       project_id     = "1372813089189561287"
       region         = "eu-central-1"
       status         = "PAUSED"
   }
   ```
```
4. クラスタを再開する必要がある場合は、`paused = false` に設定してください：

   ```
   config = {
       paused = false
       root_password = "Your_root_password1."
       port          = 4000
       ...
     }
   ```

5. `terraform apply` コマンドを実行し、確認には `yes` と入力してください。ステータスを確認するために `terraform state show tidbcloud_cluster.${resource-name}` コマンドを使用すると、ステータスが `RESUMING` に変わっていることがわかります：

   ```
   # tidbcloud_cluster.example_cluster:
   resource "tidbcloud_cluster" "example_cluster" {
       cloud_provider = "AWS"
       cluster_type   = "DEDICATED"
       config         = {
           components     = {
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
           ip_access_list = [
               # (1 unchanged element hidden)
           ]
           paused         = false
           port           = 4000
           root_password  = "Your_root_password1."
       }
       id             = "1379661944630234067"
       name           = "firstCluster"
       project_id     = "1372813089189561287"
       region         = "eu-central-1"
       status         = "RESUMING"
   }
   ```

6. しばらく待ってから、`terraform refersh` コマンドを使用してステータスを更新します。最終的にステータスは `AVAILABLE` に変更されます。

これで、Terraformを使用してTiDB専用クラスタを作成し、管理しました。次に、[backup resource](/tidb-cloud/terraform-use-backup-resource.md)を使用してクラスタのバックアップを作成することができます。

## クラスタのインポート

Terraformで管理されていないTiDBクラスタについては、単にインポートすることでTerraformを使用して管理することができます。

たとえば、Terraformで作成されていないクラスタをインポートしたり、[restore resource](/tidb-cloud/terraform-use-restore-resource.md#create-a-restore-task)で作成されたクラスタをインポートしたりすることができます。

1. 次のように `import_cluster.tf` ファイルを作成します：

    ```
    terraform {
     required_providers {
       tidbcloud = {
         source = "tidbcloud/tidbcloud"
       }
     }
   }
    resource "tidbcloud_cluster" "import_cluster" {}
    ```

2. `terraform import tidbcloud_cluster.import_cluster projectId,clusterId` コマンドを使用してクラスタをインポートします：

   例：

    ```
    $ terraform import tidbcloud_cluster.import_cluster 1372813089189561287,1379661944630264072

    tidbcloud_cluster.import_cluster: Importing from ID "1372813089189561287,1379661944630264072"...
    tidbcloud_cluster.import_cluster: Import prepared!
      Prepared tidbcloud_cluster for import
    tidbcloud_cluster.import_cluster: Refreshing state... [id=1379661944630264072]

    Import successful!

    The resources that were imported are shown above. These resources are now in your Terraform state and will henceforth be managed by Terraform.
    ```

3. `terraform state show tidbcloud_cluster.import_cluster` コマンドを実行してクラスタのステータスを確認します：

    ```
    $ terraform state show tidbcloud_cluster.import_cluster

    # tidbcloud_cluster.import_cluster:
    resource "tidbcloud_cluster" "import_cluster" {
        cloud_provider = "AWS"
        cluster_type   = "DEDICATED"
        config         = {
            components = {
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
            port       = 4000
        }
        id             = "1379661944630264072"
        name           = "restoreCluster"
        project_id     = "1372813089189561287"
        region         = "eu-central-1"
        status         = "AVAILABLE"
    }
    ```

4. Terraformを使用してクラスタを管理するために、前の手順の出力を構成ファイルにコピーすることができます。ただし、`id` と `status` の行を削除する必要があります。なぜなら、これらはTerraformによって制御されるからです：

    ```
    resource "tidbcloud_cluster" "import_cluster" {
          cloud_provider = "AWS"
          cluster_type   = "DEDICATED"
          config         = {
              components = {
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
              port       = 4000
          }
          name           = "restoreCluster"
          project_id     = "1372813089189561287"
          region         = "eu-central-1"
    }
    ```

5. 構成ファイルをフォーマットするために `terraform fmt` を使用することができます：

    ```
    $ terraform fmt
    ```

6. 構成とステートの一貫性を確保するために、`terraform plan` や `terraform apply` を実行できます。`No changes` と表示された場合、インポートは成功しています。

    ```
    $ terraform apply

    tidbcloud_cluster.import_cluster: Refreshing state... [id=1379661944630264072]

    No changes. Your infrastructure matches the configuration.

    Terraform has compared your real infrastructure against your configuration and found no differences, so no changes are needed.

    Apply complete! Resources: 0 added, 0 changed, 0 destroyed.
    ```

これで、Terraformを使用してクラスタを管理することができます。

## クラスタの削除

クラスタを削除するには、対応する `cluster.tf` ファイルがあるクラスタディレクトリに移動し、その後、`terraform destroy` コマンドを実行してクラスタリソースを破棄します：

```
$ terraform destroy

Plan: 0 to add, 0 to change, 1 to destroy.

Do you really want to destroy all resources?
Terraform will destroy all your managed infrastructure, as shown above.
There is no undo. Only 'yes' will be accepted to confirm.

Enter a value: yes
```

これで、`terraform show` コマンドを実行すると、リソースがクリアされたため、何も表示されなくなります：

```
$ terraform show
```