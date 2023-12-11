---
title: External Storage ServicesのURI形式
summary: Amazon S3、GCS、Azure Blob Storageなどの外部ストレージサービスのURI形式について学びます。

## External Storage ServicesのURI形式

このドキュメントでは、Amazon S3、GCS、Azure Blob Storageなどの外部ストレージサービスのURI形式について説明します。

URIの基本的な形式は以下の通りです：

```shell
[scheme]://[host]/[path]?[parameters]
```

## Amazon S3 URI形式

- `scheme`: `s3`
- `host`: `バケット名`
- `parameters`:

    - `access-key`: アクセスキーを指定します。
    - `secret-access-key`: シークレットアクセスキーを指定します。
    - `session-token`: 一時セッショントークンを指定します。BRはこのパラメータをまだサポートしていません。
    - `use-accelerate-endpoint`: Amazon S3でアクセラレートエンドポイントを使用するかどうかを指定します（デフォルトは`false`）。
    - `endpoint`: S3互換サービスのカスタムエンドポイントのURLを指定します（例: `<https://s3.example.com/>`）。
    - `force-path-style`: 仮想ホストスタイルに代わりパススタイルアクセスを使用します（デフォルトは`true`）。
    - `storage-class`: アップロードオブジェクトのストレージクラスを指定します（例: `STANDARD`または`STANDARD_IA`）。
    - `sse`: アップロードされたオブジェクトを暗号化するために使用されるサーバーサイド暗号化アルゴリズムを指定します（値のオプション: ``, `AES256`、または`aws:kms`）。
    - `sse-kms-key-id`: `sse`が`aws:kms`に設定されている場合、KMS IDを指定します。
    - `acl`: アップロードされたオブジェクトのキャニスタイルACLを指定します（例: `private`または`authenticated-read`）。
    - `role-arn`: [IAMロール](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html)を使用してサードパーティからAmazon S3データにアクセスする必要がある場合、IAMロールの対応する[Amazonリソース名（ARN）](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html)を`role-arn` URLクエリパラメータで指定できます。例: `arn:aws:iam::888888888888:role/my-role`。IAMロールを使用してサードパーティからAmazon S3データにアクセスする場合の詳細については、[AWSドキュメント](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_common-scenarios_third-party.html)を参照してください。
    - `external-id`: サードパーティからAmazon S3データにアクセスする際、IAMロールを想定するために[外部ID](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-user_externalid.html)を指定する必要がある場合、この`external-id` URLクエリパラメータを使用して外部IDを指定し、IAMロールを想定できるようにします。外部IDは、IAMロールARNとともにサードパーティによって提供される任意の文字列です。IAMロールを想定する際に外部IDを指定することは任意です。これはつまり、サードパーティがIAMロールの外部IDを要求しない場合、このパラメータを提供せずにIAMロールを想定し、対応するAmazon S3データにアクセスできるということです。

以下はTiDB LightningおよびBRのAmazon S3 URIの例です。この例では、特定のファイルパス`testfolder`を指定する必要があります。

```shell
s3://external/testfolder?access-key=${access-key}&secret-access-key=${secret-access-key}
```

以下は[`IMPORT INTO`](/sql-statements/sql-statement-import-into.md)のAmazon S3 URIの例です。この例では、特定のファイル名`test.csv`を指定する必要があります。

```shell
s3://external/test.csv?access-key=${access-key}&secret-access-key=${secret-access-key}
```

## GCS URI形式

- `scheme`: `gcs`または`gs`
- `host`: `バケット名`
- `parameters`:

    - `credentials-file`: マイグレーションツールノード上の認証情報JSONファイルへのパスを指定します。
    - `storage-class`: アップロードオブジェクトのストレージクラスを指定します（例: `STANDARD`または`COLDLINE`）。
    - `predefined-acl`: アップロードオブジェクトの事前定義ACLを指定します（例: `private`または`project-private`）。

以下はTiDB LightningおよびBRのGCS URIの例です。この例では、特定のファイルパス`testfolder`を指定する必要があります。

```shell
gcs://external/testfolder?credentials-file=${credentials-file-path}
```

以下は[`IMPORT INTO`](/sql-statements/sql-statement-import-into.md)のGCS URIの例です。この例では、特定のファイル名`test.csv`を指定する必要があります。

```shell
gcs://external/test.csv?credentials-file=${credentials-file-path}
```

## Azure Blob Storage URI形式

- `scheme`: `azure`または`azblob`
- `host`: `コンテナ名`
- `parameters`:

    - `account-name`: ストレージのアカウント名を指定します。
    - `account-key`: アクセスキーを指定します。
    - `sas-token`: 共有アクセス署名（SAS）トークンを指定します。
    - `access-tier`: アップロードオブジェクトのアクセス層を指定します。例: `Hot`、`Cool`、または`Archive`。デフォルト値はストレージアカウントのデフォルトアクセス層です。
    - `encryption-scope`: [サーバーサイド暗号化の暗号化スコープ](https://learn.microsoft.com/en-us/azure/storage/blobs/encryption-scope-manage?tabs=powershell#upload-a-blob-with-an-encryption-scope)を指定します。
    - `encryption-key`: [サーバーサイド暗号化の暗号化キー](https://learn.microsoft.com/en-us/azure/storage/blobs/encryption-customer-provided-keys)を指定します。これにはAES256暗号化アルゴリズムが使用されます。

以下はTiDB LightningおよびBRのAzure Blob Storage URIの例です。この例では、特定のファイルパス`testfolder`を指定する必要があります。

```shell
azure://external/testfolder?account-name=${account-name}&account-key=${account-key}
```

以下は[`IMPORT INTO`](/sql-statements/sql-statement-import-into.md)のAzure Blob Storage URIの例です。この例では、特定のファイル名`test.csv`を指定する必要があります。

```shell
azure://external/test.csv?account-name=${account-name}&account-key=${account-key}
```