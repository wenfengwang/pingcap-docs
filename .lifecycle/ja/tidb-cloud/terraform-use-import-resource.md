---
title: インポートリソースの使用
summary: インポートリソースを使用してインポートタスクを管理する方法について学びます。

# インポートリソースの使用

このドキュメントでは、`tidbcloud_import`リソースを使用してTiDB Cloudクラスターにデータをインポートする方法について学ぶことができます。

`tidbcloud_import`リソースの機能は、以下を含みます:

- TiDB ServerlessおよびTiDB Dedicatedクラスター向けのインポートタスクの作成
- ローカルディスクやAmazon S3バケットからのデータのインポート
- 進行中のインポートタスクのキャンセル

## 必要条件

- [TiDB Cloud Terraformプロバイダーの取得](/tidb-cloud/terraform-get-tidbcloud-provider.md)。
- [TiDB Serverlessクラスターの作成](/tidb-cloud/create-tidb-cluster-serverless.md)または[TiDB Dedicatedクラスターの作成](/tidb-cloud/create-tidb-cluster.md)。

## インポートタスクの作成と実行

インポートリソースを使用して、ローカルインポートタスクまたはAmazon S3インポートタスクのいずれかを管理できます。

### ローカルインポートタスクの作成と実行

> **注意:**
>
> ローカルファイルのインポートはTiDB Serverlessクラスターのみでサポートされており、TiDB Dedicatedクラスターではサポートされていません。

1. インポート用のCSVファイルを作成します。例:

   ```
   id;name;age
   1;Alice;20
   2;Bob;30
   ```

2. `import`ディレクトリを作成し、その中に`main.tf`を作成します。例:

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

    resource "tidbcloud_import" "example_local" {
      project_id  = "your_project_id"
      cluster_id  = "your_cluster_id"
      type        = "LOCAL"
      data_format = "CSV"
      csv_format = {
        separator = ";"
      }
      target_table = {
        database = "test"
        table  = "import_test"
      }
      file_name = "your_csv_path"
    }
    ```

   ファイル内のリソース値（プロジェクトID、クラスターID、CSVパスなど）を自分自身のものに置き換えます。また、[構成ページ](https://registry.terraform.io/providers/tidbcloud/tidbcloud/latest/docs/resources/import#nested-schema-for-csv_format)で`csv_format`の詳細を見つけることができます。

3. `terraform apply`コマンドを実行してインポートタスクを作成し、作成およびインポートの確認のために`yes`と入力します:

   ```
   $ terraform apply
   ...
   Plan: 1 to add, 0 to change, 0 to destroy.

   Do you want to perform these actions?
   Terraform will perform the actions described above.
   Only 'yes' will be accepted to approve.

   Enter a value: yes

   tidbcloud_import.example_local: Creating...
   tidbcloud_import.example_local: Creation complete after 6s [id=781074]
   ```

4. インポートタスクの状態を確認するには、`terraform state show tidbcloud_import.${resource-name}`を使用します:

   ```
   $ terraform state show tidbcloud_import.example_local
   # tidbcloud_import.example_local:
   resource "tidbcloud_import" "example_local" {
       all_completed_tables          = [
           {
               message    = ""
               result     = "SUCCESS"
               table_name = "`test`.`import_test`"
           },
       ]
       cluster_id                    = "1379661944641274168"
       completed_percent             = 100
       completed_tables              = 1
       created_at                    = "2023-02-06T05:39:46.000Z"
       csv_format                    = {
           separator = ";"
       }
       data_format                   = "CSV"
       elapsed_time_seconds          = 48
       file_name                     = "./t.csv"
       id                            = "781074"
       new_file_name                 = "2023-02-06T05:39:42Z-t.csv"
       pending_tables                = 0
       post_import_completed_percent = 100
       processed_source_data_size    = "31"
       project_id                    = "1372813089191151295"
       status                        = "IMPORTING"
       target_table                  = {
           database = "test"
           table  = "import_test"
       }
       total_files                   = 0
       total_size                    = "31"
       total_tables_count            = 1
       type                          = "LOCAL"
   }
   ```

5. 数分後に状態を更新するために、`terraform refresh`を使用します:

   ```
   $ terraform refresh && terraform state show tidbcloud_import.example_local
   tidbcloud_import.example_local: Refreshing state... [id=781074]
   # tidbcloud_import.example_local:
   resource "tidbcloud_import" "example_local" {
       all_completed_tables          = [
           {
               message    = ""
               result     = "SUCCESS"
               table_name = "`test`.`import_test`"
           },
       ]
       cluster_id                    = "1379661944641274168"
       completed_percent             = 100
       completed_tables              = 1
       created_at                    = "2023-02-06T05:39:46.000Z"
       csv_format                    = {
           separator = ";"
       }
       data_format                   = "CSV"
       elapsed_time_seconds          = 49
       file_name                     = "./t.csv"
       id                            = "781074"
       new_file_name                 = "2023-02-06T05:39:42Z-t.csv"
       pending_tables                = 0
       post_import_completed_percent = 100
       processed_source_data_size    = "31"
       project_id                    = "1372813089191151295"
       status                        = "COMPLETED"
       target_table                  = {
           database = "test"
           table  = "import_test"
       }
       total_files                   = 0
       total_size                    = "31"
       total_tables_count            = 1
       type                          = "LOCAL"
   }
   ```

   状態が`COMPLETED`になった時、インポートタスクが完了していることを示します。

6. MySQL CLIを使用してインポートされたデータを確認します:

   ```
   mysql> SELECT * FROM test.import_test;
   +------+-------+------+
   | id   | name  | age  |
   +------+-------+------+
   |    1 | Alice |   20 |
   |    2 | Bob   |   30 |
   +------+-------+------+
   2 rows in set (0.24 sec)
   ```

### Amazon S3インポートタスクの作成と実行

> **注意:**
>
> Amazon S3バケット内のファイルにTiDB Cloudがアクセスできるようにするには、まず[Amazon S3アクセスの設定](/tidb-cloud/config-s3-and-gcs-access.md#configure-amazon-s3-access)を行う必要があります。

1. `import`ディレクトリを作成し、その中に`main.tf`を作成します。例:

   ```
   terraform {
     required_providers {
       tidbcloud = {
         source = "tidbcloud/tidbcloud"
       }
     }
   }

   provider "tidbcloud" {
     public_key  = "your_public_key"
     private_key = "your_private_key"
   }

   resource "tidbcloud_import" "example_s3_csv" {
     project_id   = "your_project_id"
     cluster_id   = "your_cluster_id"
     type         = "S3"
     data_format  = "CSV"
     aws_role_arn = "your_arn"
     source_url   = "your_url"
   }

   resource "tidbcloud_import" "example_s3_parquet" {
     project_id   = "your_project_id"
     cluster_id   = "your_cluster_id"
     type         = "S3"
     data_format  = "Parquet"
     aws_role_arn = "your_arn"
     source_url   = "your_url"
   }
   ```

2. `terraform apply`コマンドを実行してインポートタスクを作成し、作成およびインポートの確認のために`yes`と入力します:

   ```
   $ terraform apply
   ...
   Plan: 2 to add, 0 to change, 0 to destroy.

   Do you want to perform these actions?
   Terraform will perform the actions described above.
   Only 'yes' will be accepted to approve.

   Enter a value: yes

   tidbcloud_import.example_s3_csv: Creating...
   tidbcloud_import.example_s3_csv: Creation complete after 3s [id=781075]
   tidbcloud_import.example_s3_parquet: Creating...
   tidbcloud_import.example_s3_parquet: Creation complete after 4s [id=781076]
   ```

3. インポートタスクの状態を更新し、確認するには`terraform refresh`および`terraform state show tidbcloud_import.${resource-name}`を使用します。

## インポートタスクの更新

インポートタスクは更新できません。

## インポートタスクの削除

Terraformでは、インポートタスクの削除は対応するインポートリソースのキャンセルを意味します。

`COMPLETED`のインポートタスクはキャンセルすることはできません。それ以外の場合、以下の例のように`Delete Error`が発生します:

```
$ terraform destroy
...
```
      + {R}
      + {R}
    + {R}
  + {R}
```

```
      + {T}
      + {T}
    + {T}
  + {T}
```