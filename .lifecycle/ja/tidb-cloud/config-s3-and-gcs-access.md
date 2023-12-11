---
title: Amazon S3アクセスとGCSアクセスの構成
summary: Amazon Simple Storage Service (Amazon S3) アクセスとGoogle Cloud Storage (GCS) アクセスの構成方法について学ぶ。
---

# Amazon S3アクセスとGCSアクセスの構成

ソースデータがAmazon S3またはGoogle Cloud Storage (GCS) バケットに格納されている場合、TiDB Cloudにデータをインポートまたは移行する前に、バケットへのクロスアカウントアクセスを構成する必要があります。このドキュメントでは、それを行う方法について説明します。

## Amazon S3アクセスの構成

TiDB CloudがAmazon S3バケット内のソースデータにアクセスできるようにするには、次のいずれかの方法でバケットへのアクセスを構成する必要があります。

- AWSアクセスキーの使用: IAMユーザーのアクセスキーを使用してAmazon S3バケットにアクセスします。
- Role ARNの使用: Role ARNを使用してAmazon S3バケットにアクセスします。

<SimpleTab>
<div label="Role ARN">

以下の手順でTiDB Cloudのためにバケットアクセスを構成し、Role ARNを取得します。

1. [TiDB Cloudコンソール](https://tidbcloud.com/)で、ターゲットTiDBクラスタのTiDB CloudアカウントIDと外部IDを取得します。

    1. プロジェクトの[**Clusters**](https://tidbcloud.com/console/clusters) ページに移動します。

        > **ヒント:**
        >
        > 複数のプロジェクトがある場合は、左下の<MDSvgIcon name="icon-left-projects" />をクリックして別のプロジェクトに切り替えることができます。

    2. ターゲットクラスタの名前をクリックして概要ページに移動し、左のナビゲーションペインで**Import** をクリックします。

    3. **Import** ページで、右上の**Import Data**をクリックし、**From S3**を選択します。

    4. **Import from S3** ページで、**required Role ARNを取得するためのガイド**をクリックして、TiDB CloudアカウントIDとTiDB Cloud External IDを取得します。これらのIDを後で使用するためにメモしておきます。

2. AWS Management Consoleで、Amazon S3バケットに対して管理ポリシーを作成します。

    1. AWS Management Consoleにサインインして、Amazon S3コンソールを<https://console.aws.amazon.com/s3/>で開きます。
    2. **Buckets** リストで、ソースデータのあるバケットの名前を選択し、**Copy ARN**をクリックしてS3バケットARN（例: `arn:aws:s3:::tidb-cloud-source-data`）を取得します。後で使用するためにバケットARNをメモしておきます。

        ![Copy bucket ARN](/media/tidb-cloud/copy-bucket-arn.png)

    3. IAMコンソールを<https://console.aws.amazon.com/iam/>で開き、左側のナビゲーションペインで**Policies**をクリックし、次に**Create Policy**をクリックします。

        ![Create a policy](/media/tidb-cloud/aws-create-policy.png)

    4. **Create policy** ページで、**JSON**タブをクリックします。
    5. 以下のアクセスポリシーテンプレートをコピーしてポリシーテキストフィールドに貼り付けます。

        ```
        {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Sid": "VisualEditor0",
                    "Effect": "Allow",
                    "Action": [
                        "s3:GetObject",
                        "s3:GetObjectVersion"
                    ],
                    "Resource": "<Your S3 bucket ARN>/<Directory of your source data>/*"
                },
                {
                    "Sid": "VisualEditor1",
                    "Effect": "Allow",
                    "Action": [
                        "s3:ListBucket",
                        "s3:GetBucketLocation"
                    ],
                    "Resource": "<Your S3 bucket ARN>"
                }
            ]
        }
        ```

        ポリシーテキストフィールドで、次の構成を自分自身の値に更新します。

        - `"Resource": "<Your S3 bucket ARN>/<Directory of the source data>/*"`

           例えば、ソースデータが`tidb-cloud-source-data`バケットのルートディレクトリに格納されている場合は、`"Resource": "arn:aws:s3:::tidb-cloud-source-data/*"`を使用します。ソースデータがバケットの`mydata`ディレクトリに格納されている場合は、`"Resource": "arn:aws:s3:::tidb-cloud-source-data/mydata/*"`を使用します。ディレクトリの末尾に`/*`が追加されていることを確認してください。これにより、TiDB Cloudがこのディレクトリ内のすべてのファイルにアクセスできます。

        - `"Resource": "<Your S3 bucket ARN>"`

           例: `"Resource": "arn:aws:s3:::tidb-cloud-source-data"`。

    6. **Next: Tags**をクリックし、ポリシーのタグ（オプション）を追加し、次に**Next:Review**をクリックします。

    7. ポリシー名を設定し、**Create policy**をクリックします。

3. AWS Management Consoleで、TiDB Cloud用のアクセスロールを作成し、ロールARNを取得します。

    1. IAMコンソールで<https://console.aws.amazon.com/iam/>に移動し、左側のナビゲーションペインで**Roles**をクリックし、**Create role**をクリックします。

        ![Create a role](/media/tidb-cloud/aws-create-role.png)

    2. ロールを作成するために、以下の情報を入力します:

        - **信頼されたエンティティタイプ**で、**AWSアカウント** を選択します。
        - **AWSアカウント**で、**別のAWSアカウント**を選択し、次にTiDB CloudアカウントIDを**Account ID**フィールドに貼り付けます。
        - **Options**で、**Require external ID (Best practice when a third party will assume this role)**をクリックし、次にTiDB Cloud External IDを**External ID**フィールドに貼り付けます。"Require external ID"なしでロールを作成した場合、プロジェクト内の1つのTiDBクラスタに構成が完了すると、そのプロジェクト内のすべてのTiDBクラスタが同じRole ARNを使用してAmazon S3バケットにアクセスできます。アカウントIDと外部IDでロールを作成した場合、対応するTiDBクラスタのみがバケットにアクセスできます。

    3. **Next**をクリックしてポリシーリストを開き、作成したポリシーを選択し、次に**Next**をクリックします。
    4. **Role details**で、ロールの名前を設定し、右下の**Create role**をクリックします。ロールが作成された後、ロールのリストが表示されます。
    5. ロールのリストで、作成したロールの名前をクリックして概要ページに移動し、次にロールARNをコピーします。

        ![Copy AWS role ARN](/media/tidb-cloud/aws-role-arn.png)

4. TiDB Cloudコンソールで、TiDB CloudアカウントIDと外部IDを取得した**Data Import**ページに移動し、ロールARNを**Role ARN**フィールドに貼り付けます。

</div>
<div label="Access Key">

AWSアカウントのルートユーザーではなくIAMユーザーを使用してアクセスキーを作成することを推奨します。

アクセスキーを構成する手順は次のとおりです:

1. 次のポリシーを持つIAMユーザーを作成します:

   - `AmazonS3ReadOnlyAccess`
   - [`CreateOwnAccessKeys`（必須）および`ManageOwnAccessKeys`（オプション）](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#access-keys_required-permissions)

   これらのポリシーは、ソースデータを格納するバケットにのみ適用されるようにすることが推奨されます。

   詳細については、[IAMユーザーの作成](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html#id_users_create_console)を参照してください。

2. AWSアカウントIDまたはアカウントのエイリアス、IAMユーザー名、およびパスワードを使用して、[IAMコンソール](https://console.aws.amazon.com/iam)にサインインします。

3. アクセスキーを作成します。詳細については、「[IAMユーザーのアクセスキーの作成](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey)」を参照してください。

> **注意:**
>
> TiDB Cloudはアクセスキーを保存しません。インポートが完了したら、[アクセスキーを削除](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey)することを推奨します。

</div>
</SimpleTab>

## GCSアクセスの構成

TiDB CloudがGCSバケット内のソースデータにアクセスできるようにするには、バケットのGCSアクセスを構成する必要があります。プロジェクト内の1つのTiDBクラスタに構成が完了すると、そのプロジェクト内のすべてのTiDBクラスタがGCSバケットにアクセスできます。

1. TiDB Cloudコンソールで、ターゲットTiDBクラスタのGoogle Cloud Service Account IDを取得します。

    1. プロジェクトの[**Clusters**](https://tidbcloud.com/console/clusters) ページに移動します。

        > **ヒント:**
        >
        > 複数のプロジェクトがある場合は、左下の<MDSvgIcon name="icon-left-projects" />をクリックして別のプロジェクトに切り替えることができます。

    2. ターゲットクラスタの名前をクリックして概要ページに移動し、左のナビゲーションペインで**Import** をクリックします。

    3. 右上の**Import Data**をクリックし、**Show Google Cloud Service Account ID**をクリックして、Service Account IDをコピーします。後で使用するためにService Account IDをメモしておきます。
```
2. In the Google Cloud console, create an IAM role for your GCS bucket.

    1. Sign in to the [Google Cloud console](https://console.cloud.google.com/).
    2. Go to the [Roles](https://console.cloud.google.com/iam-admin/roles) page, and then click **CREATE ROLE**.

        ![Create a role](/media/tidb-cloud/gcp-create-role.png)

    3. Enter a name, description, ID, and role launch stage for the role. The role name cannot be changed after the role is created.
    4. Click **ADD PERMISSIONS**.
    5. Add the following read-only permissions to the role, and then click **Add**.

        - storage.buckets.get
        - storage.objects.get
        - storage.objects.list

        You can copy a permission name to the **Enter property name or value** field as a filter query, and choose the name in the filter result. To add the three permissions, you can use **OR** between the permission names.

        ![Add permissions](/media/tidb-cloud/gcp-add-permissions.png)

3. Go to the [Bucket](https://console.cloud.google.com/storage/browser) page, and click the name of the GCS bucket you want TiDB Cloud to access.

4. On the **Bucket details** page, click the **PERMISSIONS** tab, and then click **GRANT ACCESS**.

    ![Grant Access to the bucket ](/media/tidb-cloud/gcp-bucket-permissions.png)

5. Fill in the following information to grant access to your bucket, and then click **SAVE**.

    - In the **New Principals** field, paste the Google Cloud Service Account ID of the target TiDB cluster.
    - In the **Select a role** drop-down list, type the name of the IAM role you just created, and then choose the name from the filter result.

    > **Note:**
    >
    > To remove the access to TiDB Cloud, you can simply remove the access that you have granted.

6. On the **Bucket details** page, click the **OBJECTS** tab.

    If you want to copy a file's gsutil URI, select the file, click **Open object overflow menu**, and then click **Copy gsutil URI**.

    ![Get bucket URI](/media/tidb-cloud/gcp-bucket-uri01.png)

    If you want to use a folder's gsutil URI, open the folder, and then click the copy button following the folder name to copy the folder name. After that, you need to add `gs://` to the beginning and `/` to the end of the name to get a correct URI of the folder.

    For example, if the folder name is `tidb-cloud-source-data`, you need to use `gs://tidb-cloud-source-data/` as the URI.

    ![Get bucket URI](/media/tidb-cloud/gcp-bucket-uri02.png)

7. In the TiDB Cloud console, go to the **Data Import** page where you get the Google Cloud Service Account ID, and then paste the GCS bucket gsutil URI to the **Bucket gsutil URI** field. For example, paste `gs://tidb-cloud-source-data/`.
```