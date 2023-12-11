---
title: TiDB Self-Hosted から TiDB Cloud への移行
summary: TiDB Self-Hosted から TiDB Cloud へデータを移行する方法について学びます。
---

# TiDB Self-Hosted から TiDB Cloud への移行

このドキュメントでは、Dumpling と TiCDC を使用して TiDB Self-Hosted クラスタから TiDB Cloud (AWS) にデータを移行する方法について説明します。

全体的な手順は次のとおりです:

1. 環境を構築し、ツールを準備します。
2. フルデータを移行します。手順は次のとおりです:

   1. Dumpling を使用して TiDB Self-Hosted から Amazon S3 にデータをエクスポートします。
   2. Amazon S3 から TiDB Cloud にデータをインポートします。

3. TiCDC を使用して増分データをレプリケーションします。
4. 移行されたデータを検証します。

## 前提条件

S3 バケットと TiDB Cloud クラスタを同じリージョンに配置することをお勧めします。リージョン間の移行では、データ変換に追加のコストが発生する可能性があります。

移行の前に、次の準備を行う必要があります:

- 管理者アクセス権を持つ [AWS アカウント](https://docs.aws.amazon.com/ja_jp/IAM/latest/UserGuide/getting-set-up.html)
- [AWS S3 バケット](https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/userguide/create-bucket.html)
- [TiDB Cloud アカウント](/tidb-cloud/tidb-cloud-quickstart.md) および、AWS でホストされている対象の TiDB Cloud クラスタに対して少なくとも [`プロジェクトデータアクセスの読み取りと書き込み`](/tidb-cloud/manage-user-access.md#user-roles) アクセスを持つ

## ツールの準備

次のツールを準備する必要があります:

- Dumpling: データエクスポートツール
- TiCDC: データレプリケーションツール

### Dumpling

[Dumpling](https://docs.pingcap.com/tidb/dev/dumpling-overview) は、TiDB または MySQL からデータを SQL または CSV ファイルにエクスポートするツールです。Dumpling を使用して、TiDB Self-Hosted からフルデータをエクスポートできます。

Dumpling を展開する前に、次の点に注意してください:

- Dumpling を TiDB Cloud クラスタと同じ VPC 内の新しい EC2 インスタンスに展開することをお勧めします。
- 推奨される EC2 インスタンスタイプは **c6g.4xlarge** (16 vCPU および 32 GiB メモリ) です。必要に応じて他の EC2 インスタンスタイプを選択できます。Amazon Machine Image (AMI) は Amazon Linux、Ubuntu、または Red Hat を使用できます。

TiUP を使用して Dumpling を展開するか、インストールパッケージを使用して Dumpling を展開できます。

#### TiUP を使用して Dumpling を展開する

TiUP を使用して Dumpling を展開します:

```bash
## TiUP を展開する
curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
source /root/.bash_profile
## Dumpling を展開し、最新バージョンに更新する
tiup install dumpling
tiup update --self && tiup update dumpling
```

#### インストールパッケージを使用して Dumpling を展開する

インストールパッケージを使用して Dumpling を展開するには:

1. [ツールキットパッケージ](https://docs.pingcap.com/tidb/stable/download-ecosystem-tools) をダウンロードします。

2. 対象のマシンに展開します。TiUP を使用して Dumpling を取得し、`tiup dumpling ...` を実行して Dumpling を実行できます。詳細については、[Dumpling の概要](https://docs.pingcap.com/tidb/stable/dumpling-overview#dumpling-introduction) を参照してください。

#### Dumpling の権限を設定する

上流データベースからデータをエクスポートするために、次の権限が必要です:

- SELECT
- RELOAD
- LOCK TABLES
- REPLICATION CLIENT
- PROCESS

### TiCDC を展開する

TiDB クラスタから TiDB Cloud に増分データをレプリケーションするために、[TiCDC を展開](https://docs.pingcap.com/tidb/dev/deploy-ticdc)する必要があります。

1. 現在の TiDB バージョンが TiCDC をサポートしているかどうかを確認します。TiDB v4.0.8.rc.1 およびそれ以降のバージョンは TiCDC をサポートしています。TiDB クラスタで `select tidb_version();` を実行して TiDB バージョンを確認できます。アップグレードが必要な場合は、「TiUP を使用した TiDB のアップグレード」 (https://docs.pingcap.com/tidb/dev/deploy-ticdc#upgrade-ticdc-using-tiup) を参照してください。

2. TiCDC コンポーネントを TiDB クラスタに追加します。TiCDC を追加するには、`scale-out.yml` ファイルを編集して、TiCDC を追加します。

    ```yaml
    cdc_servers:
    - host: 10.0.1.3
      gc-ttl: 86400
      data_dir: /tidb-data/cdc-8300
    - host: 10.0.1.4
      gc-ttl: 86400
      data_dir: /tidb-data/cdc-8300
    ```

3. TiCDC コンポーネントを追加し、状態を確認します。

    ```shell
    tiup cluster scale-out <cluster-name> scale-out.yml
    tiup cluster display <cluster-name>
    ```

## フルデータの移行

TiDB Self-Hosted クラスタから TiDB Cloud にデータを移行するには、次の手順でフルデータの移行を行います:

1. TiDB Self-Hosted クラスタから Amazon S3 へデータを移行します。
2. Amazon S3 から TiDB Cloud にデータを移行します。

### TiDB Self-Hosted クラスタから Amazon S3 へデータを移行する

TiDB Self-Hosted クラスタから Amazon S3 にデータを移行するには、Dumpling を使用します。

TiDB クラスタがローカル IDC にある場合や、Dumpling サーバと Amazon S3 の間のネットワークが接続されていない場合は、まずファイルをローカルストレージにエクスポートし、後でそれらを Amazon S3 にアップロードすることができます。

#### ステップ 1. 上流の TiDB Self-Hosted クラスタの GC メカニズムを一時的に無効にする

増分移行中に新しく書き込まれたデータが失われないようにするために、移行を開始する前に上流クラスタのガベージコレクション (GC) メカニズムを無効にする必要があります。

次のコマンドを実行して設定が成功したことを確認します。

```sql
SET GLOBAL tidb_gc_enable = FALSE;
```

以下は、無効になっていることを示す例の出力です。

```sql
SELECT @@global.tidb_gc_enable;
+-------------------------+
| @@global.tidb_gc_enable |
+-------------------------+
|                       0 |
+-------------------------+
1 row in set (0.01 sec)
```

#### ステップ 2. Dumpling が Amazon S3 バケットへのアクセス許可を設定する

AWS コンソールでアクセスキーを作成します。詳細については、「アクセスキーの作成」(https://docs.aws.amazon.com/ja_jp/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey) を参照してください。

1. AWS アカウントの ID またはアカウントエイリアス、IAM ユーザー名、およびパスワードを使用して、[IAM コンソール](https://console.aws.amazon.com/iam/home#/security_credentials) にサインインします。

2. 右上のナビゲーションバーでユーザー名を選択し、[マイセキュリティ資格情報] をクリックします。

3. アクセスキーを作成するには、[アクセスキーの作成] をクリックします。その後、**.csv ファイルをダウンロード** を選択して、アクセスキー ID とシークレットアクセスキーを CSV ファイルに保存します。ファイルを安全な場所に保存してください。このダイアログボックスを閉じると、シークレットアクセスキーに再度アクセスすることはできません。CSV ファイルをダウンロードした後、[閉じる] を選択します。アクセスキーを作成すると、キーペアはデフォルトでアクティブになり、そのペアをすぐに使用できます。

    ![アクセスキーの作成](/media/tidb-cloud/op-to-cloud-create-access-key01.png)

    ![CSV ファイルをダウンロード](/media/tidb-cloud/op-to-cloud-create-access-key02.png)

#### ステップ 3. Dumpling を使用して上流の TiDB クラスタから Amazon S3 にデータをエクスポートする

Dumpling を使用して上流の TiDB クラスタから Amazon S3 にデータをエクスポートするには、次の手順を実行します:

1. Dumpling の環境変数を設定します。

    ```shell
    export AWS_ACCESS_KEY_ID=${AccessKey}
    export AWS_SECRET_ACCESS_KEY=${SecretKey}
    ```

2. AWS コンソールから S3 バケットの URI とリージョン情報を取得します。詳細については、[バケットの作成](https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/userguide/create-bucket-create.html) を参照してください。

    次のスクリーンショットは、S3 バケットの URI 情報を取得する方法を示しています:

    ![S3 URI の取得](/media/tidb-cloud/op-to-cloud-copy-s3-uri.png)

    次のスクリーンショットは、リージョン情報を取得する方法を示しています:

    ![リージョン情報の取得](/media/tidb-cloud/op-to-cloud-copy-region-info.png)

3. Dumpling を実行して、データを Amazon S3 バケットにエクスポートします。

    ```ymal
    dumpling \
    -u root \
    -P 4000 \
    -h 127.0.0.1 \
    -r 20000 \
```
--filetype {sql|csv}  \
-F 256MiB  \
-t 8 \
-o "${S3のURI}" \
--s3.region "${s3.region}"
```
`-t`オプションはエクスポートのスレッド数を指定します。スレッド数を増やすと、Dumplingとエクスポート速度の並行性が向上し、データベースのメモリ消費量も増加します。したがって、このパラメータにあまり大きな数値を設定しないでください。

詳細は、[Dumpling](https://docs.pingcap.com/tidb/stable/dumpling-overview#export-to-sql-files)を参照してください。

4. エクスポートデータを確認します。通常、エクスポートされるデータには次のものが含まれます:

- `metadata`: このファイルにはエクスポートの開始時間と、マスターバイナリログの場所が含まれています。
- `{schema}-schema-create.sql`: スキーマを作成するためのSQLファイル
- `{schema}.{table}-schema.sql`: テーブルを作成するためのSQLファイル
- `{schema}.{table}.{0001}.{sql|csv}`: データファイル
- `*-schema-view.sql`, `*-schema-trigger.sql`, `*-schema-post.sql`: その他のエクスポートされたSQLファイル

### Amazon S3からTiDB Cloudへのデータ移行

TiDBセルフホストクラスタからAmazon S3にデータをエクスポートした後、そのデータをTiDB Cloudに移行する必要があります。

1. TiDB Cloudコンソールで、クラスタのアカウントIDと外部IDを取得します。詳細は、[ステップ2. Amazon S3へのアクセスの構成](/tidb-cloud/tidb-cloud-auditing.md#step-2-configure-amazon-s3-access)を参照してください。

次のスクリーンショットは、アカウントIDと外部IDを取得する方法を示しています:

![アカウントIDと外部IDを取得](/media/tidb-cloud/op-to-cloud-get-role-arn.png)

2. Amazon S3のアクセス権限を構成します。通常、次の読み取り専用の権限が必要です:

- s3:GetObject
- s3:GetObjectVersion
- s3:ListBucket
- s3:GetBucketLocation

S3バケットがサーバーサイド暗号化SSE-KMSを使用している場合、KMS権限も追加する必要があります。

- kms:Decrypt

3. アクセスポリシーを構成します。[AWSコンソール > IAM > アクセス管理 > ポリシー](https://console.aws.amazon.com/iamv2/home#/policies)に移動し、アクセスポリシーがTiDB Cloud用に既に存在するかどうかを確認するために、地域を切り替えます。存在しない場合は、このドキュメントに従ってポリシーを作成してください: [JSONタブでポリシーを作成する](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create-console.html)。

次は、jsonポリシーの例のテンプレートです。

```json
## jsonポリシーテンプレートを作成
##<Your customized directory>: インポート対象のデータファイルが存在するS3バケットのフォルダへのパスを入力してください。
##<Your S3 bucket ARN>: S3バケットのARNを入力してください。S3バケット概要ページで、ARNを取得するための「ARNのコピー」ボタンをクリックできます。
##<Your AWS KMS ARN>: S3バケットKMSキーのARNを入力してください。これは、S3バケット > プロパティ > デフォルトの暗号化 > AWS KMSキーのARNから取得できます。詳細については、https://docs.aws.amazon.com/AmazonS3/latest/userguide/viewing-bucket-key-settings.html を参照してください。

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:GetObjectVersion"
            ],
            "Resource": "arn:aws:s3:::<Your customized directory>"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket",
                "s3:GetBucketLocation"
            ],
            "Resource": "<Your S3 bucket ARN>"
        }
        // サーバーサイド暗号化SSE-KMSを有効にしている場合、次の権限も追加する必要があります。
        {
            "Effect": "Allow",
            "Action": [
                "kms:Decrypt"
            ],
            "Resource": "<Your AWS KMS ARN>"
        }
        ,
        {
            "Effect": "Allow",
            "Action": "kms:Decrypt",
            "Resource": "<Your AWS KMS ARN>"
        }
    ]
}
```

4. ロールを構成します。[IAMロールの作成(コンソール)](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-user.html)を参照してください。アカウントID欄には、ステップ1でメモしたTiDB CloudアカウントIDとTiDB Cloud外部IDを入力してください。

5. ロールのARNを取得します。[AWSコンソール > IAM > アクセス管理 > ロール](https://console.aws.amazon.com/iamv2/home#/roles)に移動し、地域を切り替えます。作成したロールをクリックし、ARNをメモしてください。これは、TiDB Cloudにデータをインポートする際に使用します。

6. TiDB Cloudにデータをインポートします。[Amazon S3またはGCSからTiDB CloudにCSVファイルをインポートする](/tidb-cloud/import-csv-files.md)を参照してください。

## 増分データをレプリケートする

増分データをレプリケートするには、次の手順を実行します:

1. 増分データ移行の開始時間を取得します。たとえば、フルデータ移行のメタデータファイルからこれを取得できます。

    ![メタデータ中の開始時間](/media/tidb-cloud/start_ts_in_metadata.png)

2. TiCDCがTiDB Cloudに接続する許可を付与します。[TiDB Cloudコンソール](https://tidbcloud.com/console/clusters)で、クラスタを見つけて、**概要** > **接続** > **標準接続** > **トラフィックフィルタの作成**に移動します。**編集** > **アイテムの追加**をクリックします。**IPアドレス**のフィールドにTiCDCコンポーネントのパブリックIPアドレスを入力し、**フィルタの更新**をクリックして保存します。これでTiCDCがTiDB Cloudにアクセスできるようになります。

    ![フィルタの更新](/media/tidb-cloud/edit_traffic_filter_rules.png)

3. 下流のTiDB Cloudクラスタの接続情報を取得します。[TiDB Cloudコンソール](https://tidbcloud.com/console/clusters)で、**概要** > **接続** > **標準接続** > **SQLクライアントを使用して接続**に移動します。接続情報から、クラスタのホストIPアドレスとポートを取得できます。詳細については、[標準接続を使用して接続する](/tidb-cloud/connect-via-standard-connection.md)を参照してください。

4. 増分レプリケーションタスクを作成および実行します。上流クラスタで、次のコマンドを実行します:

    ```shell
    tiup cdc cli changefeed create \
    --pd=http://172.16.6.122:2379  \
    --sink-uri="tidb://root:123456@172.16.6.125:4000"  \
    --changefeed-id="upstream-to-downstream"  \
    --start-ts="431434047157698561"
    ```

    - `--pd`: 上流クラスタのPDアドレス。フォーマットは次のとおりです: `[upstream_pd_ip]:[pd_port]`
    - `--sink-uri`: レプリケーションタスクの下流アドレス。`--sink-uri`の構成は、次のフォーマットに従ってください。現在、スキームが`mysql`、`tidb`、`kafka`、`s3`、`local`をサポートしています。

        ```shell
        [scheme]://[userinfo@][host]:[port][/path]?[query_parameters]
        ```

    - `--changefeed-id`: レプリケーションタスクのID。フォーマットは^[a-zA-Z0-9]+(\-[a-zA-Z0-9]+)*$正規表現と一致する必要があります。このIDが指定されていない場合、TiCDCは自動的にUUID（バージョン4フォーマット）をIDとして生成します。
    - `--start-ts`: changefeedの開始TSOを指定します。このTSOから、TiCDCクラスタはデータを取得し始めます。デフォルト値は現在時刻です。

    詳細については、[TiCDC ChangefeedsのCLIおよび構成パラメータ](https://docs.pingcap.com/tidb/dev/ticdc-changefeed-config)を参照してください。

5. 上流クラスタで再びGCメカニズムを有効にします。増分レプリケーションでエラーや遅延がない場合は、クラスタのガベージコレクションを再開するためにGCメカニズムを有効にします。

    次のコマンドを実行して、設定が機能しているかどうかを確認します。

    ```sql
    SET GLOBAL tidb_gc_enable = TRUE;
    ```

    以下は、`1`がGCが無効であることを示す例です。

    ```sql
    SELECT @@global.tidb_gc_enable;
    +-------------------------+
    | @@global.tidb_gc_enable |
    +-------------------------+
    |                       1 |
    +-------------------------+
    1 row in set (0.01 sec)
    ```

6. 増分レプリケーションタスクを確認します。

    - 出力に "Create changefeed successfully!" というメッセージが表示された場合、レプリケーションタスクが正常に作成されています。
    - 状態が`normal`である場合、レプリケーションタスクは正常です。

        ```shell
        tiup cdc cli changefeed list --pd=http://172.16.6.122:2379
        ```

        ![フィルタの更新](/media/tidb-cloud/normal_status_in_replication_task.png)
```
- レプリケーションを確認します。アップストリームクラスターに新しいレコードを書き込み、そのレコードがダウンストリームのTiDB Cloudクラスターにレプリケートされているかどうかを確認してください。