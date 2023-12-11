---
title: バックアップストレージ
summary: TiDBのバックアップとリストアで使用されるストレージURI形式の説明
aliases: ['/docs/dev/br/backup-and-restore-storages/','/tidb/dev/backup-storage-S3/','/tidb/dev/backup-storage-azblob/','/tidb/dev/backup-storage-gcs/','/tidb/dev/external-storage/']
---

# バックアップストレージ

TiDBは、バックアップデータをAmazon S3、Google Cloud Storage（GCS）、Azure Blob Storage、およびNFSに保存することができます。具体的には、`br`コマンドの`--storage`または`-s`パラメータでバックアップストレージのURIを指定できます。このドキュメントでは、異なる外部ストレージサービスの[URI形式](#uri-format)と[認証](#authentication)、[サーバーサイド暗号化](#server-side-encryption)を紹介します。

## TiKVへの認証情報の送信

| CLIパラメータ | 説明 | デフォルト値
|:----------|:-------|:-------|
| `--send-credentials-to-tikv` | BRで取得した認証情報をTiKVに送信するかどうかを制御します。 | `true`|

デフォルトで、BRはAmazon S3、GCS、またはAzure Blob Storageをストレージシステムとして使用する際に、各TiKVノードに認証情報を送信します。この動作は構成を簡略化し、`--send-credentials-to-tikv`（または短縮形の`-c`）パラメータで制御されます。

これはクラウド環境には適用されません。IAMロールの認証情報を使用する場合、各ノードにはそれぞれのロールと権限があります。この場合、認証情報の送信を無効にするために、`--send-credentials-to-tikv=false`（または短縮形の`-c=0`）を設定する必要があります。

```bash
./br backup full -c=0 -u pd-service:2379 --storage 's3://bucket-name/prefix'
```

もし、[`BACKUP`](/sql-statements/sql-statement-backup.md)と[`RESTORE`](/sql-statements/sql-statement-restore.md)ステートメントを使用してデータをバックアップまたはリストアする場合、`SEND_CREDENTIALS_TO_TIKV = FALSE`オプションを追加することができます。

```sql
BACKUP DATABASE * TO 's3://bucket-name/prefix' SEND_CREDENTIALS_TO_TIKV = FALSE;
```

## URI形式

### URI形式の説明

このセクションでは、ストレージサービスのURI形式について説明します。

```shell
[scheme]://[host]/[path]?[parameters]
```

<SimpleTab groupId="storage">
<div label="Amazon S3" value="amazon">

- `scheme`: `s3`
- `host`: `バケット名`
- `parameters`:

    - `access-key`: アクセスキーを指定します。
    - `secret-access-key`: シークレットアクセスキーを指定します。
    - `session-token`: 一時セッショントークンを指定します。BRはまだこのパラメータをサポートしていません。
    - `use-accelerate-endpoint`: Amazon S3のアクセラレートエンドポイントを使用するかどうかを指定します（デフォルトは`false`）。
    - `endpoint`: S3互換サービスのカスタムエンドポイントのURLを指定します（例：`<https://s3.example.com/>`）。
    - `force-path-style`: 仮想ホスト形式のアクセスではなくパス形式のアクセスを使用するよう指定します（デフォルトは`true`）。
    - `storage-class`: アップロードされたオブジェクトのストレージクラスを指定します（例：`STANDARD`または`STANDARD_IA`）。
    - `sse`: アップロードされたオブジェクトを暗号化するために使用されるサーバーサイド暗号化アルゴリズムを指定します（値の選択肢：``、`AES256`、または `aws:kms`）。
    - `sse-kms-key-id`: `sse`が`aws:kms`に設定されている場合、KMS IDを指定します。
    - `acl`: アップロードされたオブジェクトのキャニッドACLを指定します（例：`private`または`authenticated-read`）。
    - `role-arn`: 指定したIAMロールを使用してサードパーティからAmazon S3データにアクセスする必要がある場合は、IAMロールの対応する[Amazon Resource Name（ARN）](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html)を指定できます。詳細についてはAWSのドキュメントを参照してください。
    - `external-id`: サードパーティからAmazon S3データにアクセスする場合、IAMロールを仮定するために正しい[外部ID](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-user_externalid.html)を指定する必要がある場合があります。このような場合は、この`external-id` URLクエリパラメータを使用して外部IDを指定し、IAMロールを仮定し、対応するAmazon S3データにアクセスできるようにします。

</div>
<div label="GCS" value="gcs">

- `scheme`: `gcs`または`gs`
- `host`: `バケット名`
- `parameters`:

    - `credentials-file`: マイグレーションツールノード上の認証情報JSONファイルへのパスを指定します。
    - `storage-class`: アップロードされたオブジェクトのストレージクラスを指定します（例：`STANDARD`または`COLDLINE`）。
    - `predefined-acl`: アップロードされたオブジェクトの事前定義ACLを指定します（例：`private`または`project-private`）。

</div>
<div label="Azure Blob Storage" value="azure">

- `scheme`: `azure`または`azblob`
- `host`: `コンテナ名`
- `parameters`:

    - `account-name`: ストレージのアカウント名を指定します。
    - `account-key`: アクセスキーを指定します。
    - `sas-token`: 共有アクセス署名（SAS）トークンを指定します。
    - `access-tier`: アップロードされたオブジェクトのアクセス層を指定します。例：`Hot`、`Cool`、または`Archive`。デフォルト値はストレージアカウントのデフォルトのアクセス層です。
    - `encryption-scope`: サーバーサイド暗号化の[暗号化スコープ](https://learn.microsoft.com/en-us/azure/storage/blobs/encryption-scope-manage?tabs=powershell#upload-a-blob-with-an-encryption-scope)を指定します。
    - `encryption-key`: サーバーサイド暗号化に使用される[暗号化キー](https://learn.microsoft.com/en-us/azure/storage/blobs/encryption-customer-provided-keys)を指定します。

</div>
</SimpleTab>

### URIの例

このセクションでは、`host`パラメータとして`external`を使用したいくつかのURIの例を提供します（前述のバケット名またはコンテナ名を使用）。

<SimpleTab groupId="storage">
<div label="Amazon S3" value="amazon">

**Amazon S3にスナップショットデータをバックアップ**

```shell
./br backup full -u "${PD_IP}:2379" \
--storage "s3://external/backup-20220915?access-key=${access-key}&secret-access-key=${secret-access-key}"
```

**Amazon S3からスナップショットデータをリストア**

```shell
./br restore full -u "${PD_IP}:2379" \
--storage "s3://external/backup-20220915?access-key=${access-key}&secret-access-key=${secret-access-key}"
```

</div>
<div label="GCS" value="gcs">

**GCSにスナップショットデータをバックアップ**

```shell
./br backup full --pd "${PD_IP}:2379" \
--storage "gcs://external/backup-20220915?credentials-file=${credentials-file-path}"
```

**GCSからスナップショットデータをリストア**

```shell
./br restore full --pd "${PD_IP}:2379" \
--storage "gcs://external/backup-20220915?credentials-file=${credentials-file-path}"
```

</div>
<div label="Azure Blob Storage" value="azure">

**Azure Blob Storageにスナップショットデータをバックアップ**

```shell
./br backup full -u "${PD_IP}:2379" \
--storage "azure://external/backup-20220915?account-name=${account-name}&account-key=${account-key}"
```

**Azure Blob Storageでスナップショットバックアップデータから`test`データベースをリストア**

```shell
./br restore db --db test -u "${PD_IP}:2379" \
--storage "azure://external/backup-20220915account-name=${account-name}&account-key=${account-key}"
```

</div>
</SimpleTab>

## 認証

クラウドストレージシステムにバックアップデータを保存する場合、特定のクラウドサービスプロバイダに応じて認証パラメータを構成する必要があります。このセクションでは、Amazon S3、GCS、Azure Blob Storageで使用される認証方法と、対応するストレージサービスへのアクセスに使用されるアカウントの構成方法について説明します。

<SimpleTab groupId="storage">
<div label="Amazon S3" value="amazon">

バックアップディレクトリへのアクセス権を設定する前に、以下の特権を構成してください。

- バックアップ中にバックアップディレクトリへのアクセス権限がTiKVおよびバックアップ＆リストア（BR）に必要な最小権限： `s3:ListBucket`、`s3:PutObject`、および`s3:AbortMultipartUpload`
- リストア中にバックアップディレクトリへのアクセス権限がTiKVおよびBRに必要な最小権限： `s3:ListBucket`、`s3:GetObject`、および`s3:PutObject`。 BRは、バックアップディレクトリの`./checkpoints`サブディレクトリにチェックポイント情報を書き込みます。バックアップデータのログを復元する際、BRは復元されたクラスタのテーブルIDマッピング関係をバックアップディレクトリの`./pitr_id_maps`サブディレクトリに書き込みます。

バックアップディレクトリをまだ作成していない場合は、[バケットの作成](https://docs.aws.amazon.com/AmazonS3/latest/userguide/create-bucket-overview.html)を参照して、指定されたリージョンでS3バケットを作成してください。必要に応じて、[フォルダの作成](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-folders.html)を参照して、バケット内にフォルダを作成することもできます。

S3へのアクセスを次のいずれかの方法で構成することをお勧めします：

- 方法1：アクセスキーを指定する

    URIでアクセスキーとシークレットアクセスキーを指定すると、指定されたアクセスキーとシークレットアクセスキーを使用して認証が行われます。 URIでキーを指定する以外に、以下の方法もサポートされています：

    - BRは環境変数`$AWS_ACCESS_KEY_ID`と`$AWS_SECRET_ACCESS_KEY`を読み取ります。
    - BRは環境変数`$AWS_ACCESS_KEY`と`$AWS_SECRET_KEY`を読み取ります。
    - BRは環境変数`$AWS_SHARED_CREDENTIALS_FILE`で指定されたパスの共有資格情報ファイルを読み取ります。
    - BRは`~/.aws/credentials`パスの共有資格情報ファイルを読み取ります。

- 方法2：IAMロールに基づくアクセス

    TiKVおよびBRノードが実行されているEC2インスタンスにS3にアクセスできるIAMロールを関連付けます。関連付けの後、BRは追加設定なしでS3のバックアップディレクトリに直接アクセスできます。

    ```shell
    br backup full --pd "${PD_IP}:2379" \
    --storage "s3://${host}/${path}"
    ```

</div>
<div label="GCS" value="gcs">

アクセスに使用するアカウントを構成するには、アクセスキーを指定します。`credentials-file`パラメータを指定すると、指定された`credentials-file`を使用して認証が行われます。 URIでキーを指定する以外に、以下の方法もサポートされています：

- BRは環境変数`$GOOGLE_APPLICATION_CREDENTIALS`で指定されたパスのファイルを読み取ります
- BRは`~/.config/gcloud/application_default_credentials.json`ファイルを読み取ります。
- クラスタがGCEまたはGAEで実行されている場合、BRはメタデータサーバから認証情報を取得します。

</div>
<div label="Azure Blob Storage" value="azure">

- 方法1：共有アクセス署名を指定する

    URIで`account-name`と`sas-token`を指定すると、指定されたアカウント名と共有アクセス署名（SAS）トークンを使用して認証が行われます。 SASトークンには`&`文字が含まれています。 URIに追加する前に、`%26`にエンコードする必要があります。また、`sas-token`全体をパーセントエンコーディングを使用して直接エンコードすることもできます。

- 方法2：アクセスキーを指定する

    URIで`account-name`と`account-key`を指定すると、指定されたアカウント名とアカウントキーを使用して認証が行われます。 URIでキーを指定する方法以外に、BRは環境変数`$AZURE_STORAGE_KEY`からもキーを読み取ることができます。

- 方法3：バックアップおよびリストアのためにAzure ADを使用する

    BRが実行されているノードで環境変数`$AZURE_CLIENT_ID`、`$AZURE_TENANT_ID`、および`$AZURE_CLIENT_SECRET`を構成します。

    - クラスタがTiUPを使用して起動される場合、TiKVはsystemdサービスを使用します。以下の例は、TiKVの前述の3つの環境変数を構成する方法を示しています：

        > **注意:**
        >
        > この方法を使用する場合、ステップ3でTiKVを再起動する必要があります。クラスタが再起動できない場合は、バックアップおよびリストアに **方法1：アクセスキーを指定する** を使用してください。

        1. このノード上のTiKVポートが`24000`であり、つまりsystemdサービスの名前が`tilev-24000`であると仮定します：

            ```shell
            systemctl edit tikv-24000
            ```

        2. TiKV構成ファイルを編集して3つの環境変数を構成します：

            ```
            [Service]
            Environment="AZURE_CLIENT_ID=aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa"
            Environment="AZURE_TENANT_ID=aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa"
            Environment="AZURE_CLIENT_SECRET=aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
            ```

        3. 構成の再読み込みとTiKVの再起動：

            ```shell
            systemctl daemon-reload
            systemctl restart tikv-24000
            ```

    - コマンドラインで開始されたTiKVとBRのためにAzure AD情報を構成する場合、次のコマンドを実行して環境変数`$AZURE_CLIENT_ID`、`$AZURE_TENANT_ID`、および`$AZURE_CLIENT_SECRET`が構成されているかどうかを確認するだけです：

        ```shell
        echo $AZURE_CLIENT_ID
        echo $AZURE_TENANT_ID
        echo $AZURE_CLIENT_SECRET
        ```

    - BRを使用してデータをAzure Blob Storageにバックアップする：

        ```shell
        ./br backup full -u "${PD_IP}:2379" \
        --storage "azure://external/backup-20220915?account-name=${account-name}"
        ```

</div>
## サーバーサイドの暗号化

### Amazon S3のサーバーサイド暗号化

BRはAmazon S3にデータをバックアップする際のサーバーサイド暗号化をサポートしています。また、BRを使用してS3サーバーサイド暗号化に作成したAWS KMSキーを使用することもできます。 詳細については、[BR S3サーバーサイド暗号化](/encryption-at-rest.md#br-s3-server-side-encryption)を参照してください。

### Azure Blob Storageのサーバーサイド暗号化

BRは、Azure Blob Storageにデータをバックアップする際のAzureサーバーサイド暗号化のスコープを指定したり、暗号化キーを提供したりすることをサポートしています。この機能を使用すると、同じストレージアカウントの異なるバックアップデータに対してセキュリティ境界を確立できます。 詳細については、[BR Azure Blob Storageサーバーサイド暗号化](/encryption-at-rest.md#br-azure-blob-storage-server-side-encryption)を参照してください。

## ストレージサービスがサポートするその他の機能

BR v6.3.0はAWSの[S3 Object Lock](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html)をサポートしています。この機能を有効にすると、バックアップデータが改ざんされたり削除されたりすることを防ぐことができます。