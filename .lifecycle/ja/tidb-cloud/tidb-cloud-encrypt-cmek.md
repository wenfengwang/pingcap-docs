---
title: Customer-Managed Encryption Keysを使用したデータの暗号化
summary: TiDB CloudでCustomer-Managed Encryption Key（CMEK）を使用する方法について学びます。

# Customer-Managed Encryption Keysを使用したデータの暗号化

Customer-Managed Encryption Key（CMEK）を使用すると、TiDB Dedicatedクラスター内の静的データを完全に管理できる暗号キーを利用してセキュリティを確保できます。このキーはCMEKキーと呼ばれます。

CMEKがプロジェクトで有効になると、そのプロジェクト内で作成されるすべてのクラスターは、そのCMEKキーを使用して静的データを暗号化します。さらに、これらのクラスターによって生成されるバックアップデータも同じキーを使用して暗号化されます。CMEKが有効でない場合、TiDB Cloudはクラスター内のすべてのデータを安全に保管するためにエスクローキーを使用します。

> **注意:**
>
> 現在、この機能はリクエストでのみ利用可能です。この機能を試す必要がある場合は、[サポート](/tidb-cloud/tidb-cloud-support.md)にお問い合わせください。

## 制限事項

- 現時点では、TiDB CloudはAWS KMSを使用してのみCMEKをサポートしています。
- CMEKを使用するには、プロジェクトの作成時にCMEKを有効にし、クラスターを作成する前にCMEK関連の設定を完了する必要があります。既存のプロジェクトに対してCMEKを有効にすることはできません。
- 現時点では、CMEKが有効なプロジェクトでのみ、AWSホスト上で[TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスターを作成できます。Google Cloud上のTiDB Dedicatedクラスターや[TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスターはサポートされていません。
- 特定のプロジェクトでは、1つのAWSリージョンに対してのみCMEKを有効にできます。一度設定すると、同じプロジェクト内で他のリージョンにクラスターを作成することはできません。

## CMEKの有効化

アカウントが所有するKMSを使用してデータを暗号化したい場合は、次の手順を実行してください。

### ステップ1. CMEKを有効にするプロジェクトの作成

組織の「所有者」の役割を持っている場合は、TiDB CloudコンソールまたはAPIを使用して、CMEKを有効にするプロジェクトを作成できます。

<SimpleTab groupId="method">
<div label="コンソールを使用" value="console">

CMEKを有効にするプロジェクトを作成するには、次の手順を実行してください。

1. TiDB Cloudコンソールの左下にある<MDSvgIcon name="icon-top-organization" />をクリックします。
2. **組織設定**をクリックします。
3. **組織設定**ページで、**新規プロジェクトの作成**をクリックしてプロジェクト作成ダイアログを開きます。
4. プロジェクト名を入力します。
5. プロジェクトでCMEK機能を有効にするように選択します。
6. **確認**をクリックしてプロジェクトの作成を完了します。

</div>
<div label="APIを使用" value="api">

TiDB Cloud APIを使用してこの手順を完了するには、[CMEKを有効にするプロジェクトの作成](https://docs.pingcap.com/tidbcloud/api/v1beta#tag/Project/operation/CreateProject)エンドポイントを使用します。`aws_cmek_enabled`フィールドが`true`に設定されていることを確認してください。

現時点では、TiDB Cloud APIはベータ版です。詳細については、[TiDB Cloud APIドキュメント](https://docs.pingcap.com/tidbcloud/api/v1beta)を参照してください。

</div>
</SimpleTab>

### ステップ2. プロジェクトのCMEK設定を完了する

TiDB CloudコンソールまたはAPIを使用して、プロジェクトのCMEK設定を完了できます。

> **注意:**
>
> キーのポリシーが要件を満たし、不十分な権限やアカウントの問題などのエラーがないことを確認してください。これらのエラーにより、クラスターが誤ってこのキーを使用して作成される場合があります。

<SimpleTab groupId="method">
<div label="コンソールを使用" value="console">

プロジェクトのCMEK設定を完了するには、次の手順を実行してください。

1. TiDB Cloudコンソールの左下にある<MDSvgIcon name="icon-left-projects" />をクリックして、複数のプロジェクトを持っている場合は対象のプロジェクトに切り替え、**プロジェクト設定**をクリックします。
2. **暗号化アクセス**をクリックして、プロジェクトの暗号化管理ページに入ります。
3. **暗号化キーの作成**をクリックして、キー作成ページに入ります。
4. キープロバイダーはAWS KMSのみをサポートしています。暗号化キーを使用できるリージョンを選択できます。
5. `ROLE-TRUST-POLICY.JSON`としてJSONファイルをコピーして保存します。このファイルは信頼関係を記述しています。
6. この信頼関係をAWS KMSのキーポリシーに追加します。詳細については、[AWS KMSのキーポリシー](https://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html)を参照してください。
7. TiDB Cloudコンソールで、キー作成ページの一番下までスクロールし、AWS KMSから取得した**KMSキーARN**を入力します。
8. キーを作成するために**作成**をクリックします。

</div>
<div label="APIを使用" value="api">

1. AWS KMSでキーポリシーを構成し、次の情報をキーポリシーに追加します。

    ```json
    {
        "Version": "2012-10-17",
        "Id": "cmek-policy",
        "Statement": [
            // EBS関連ポリシー
            {
                "Sid": "EBSを介してEBSを使用できるすべてのプリンシパルにアクセスを許可",
                "Effect": "Allow",
                "Principal": {
                    "AWS": "*"
                },
                "Action": [
                    "kms:Encrypt",
                    "kms:Decrypt",
                    "kms:ReEncrypt*",
                    "kms:GenerateDataKey*",
                    "kms:CreateGrant",
                    "kms:DescribeKey"
                ],
                "Resource": "*",
                "Condition": {
                    "StringEquals": {
                        "kms:CallerAccount": "<pingcap-account>",
                        "kms:ViaService": "ec2.<region>.amazonaws.com"
                    }
                }
            },
            // S3関連ポリシー
            {
                "Sid": "TiDBクラウドロールがS3に暗号化されたバックアップを保存するためにKMSを使用できるようにする",
                "Effect": "Allow",
                "Principal": {
                    "AWS": "arn:aws:iam::<pingcap-account>:root"
                },
                "Action": [
                    "kms:Decrypt",
                    "kms:GenerateDataKey"
                ],
                "Resource": "*"
            },
            ... // ユーザー独自のKMSへの管理アクセス
        ]
    }
    ```

    - `<pingcap-account>`はクラスターが実行されているアカウントです。アカウントが不明な場合は、[TiDB Cloudサポート](/tidb-cloud/tidb-cloud-support.md)にお問い合わせください。
    - `<region>`はクラスターを作成する地域です。たとえば、`us-west-2`のようなリージョンです。リージョンを指定したくない場合は、`<region>`をワイルドカード`*`に置き換え、`StringLike`ブロックに配置してください。
    - 前述のブロックのEBS関連ポリシーについては、[AWSのドキュメント](https://docs.aws.amazon.com/kms/latest/developerguide/conditions-kms.html#conditions-kms-caller-account)を参照してください。
    - 前述のブロックのS3関連ポリシーについては、[AWSブログ](https://repost.aws/knowledge-center/s3-bucket-access-default-encryption)を参照してください。

2. TiDB Cloud APIの[Configure AWS CMEK](https://docs.pingcap.com/tidbcloud/api/v1beta#tag/Cluster/operation/CreateAwsCmek)エンドポイントを呼び出します。

    現時点では、TiDB Cloud APIはベータ版です。詳細については、[TiDB Cloud APIドキュメント](https://docs.pingcap.com/tidbcloud/api/v1beta)を参照してください。

</div>
</SimpleTab>

> **注意:**
>
> この機能は将来的にさらに強化され、追加の権限が必要になる可能性があります。そのため、このポリシー要件は変更される可能性があります。

### ステップ3. クラスターの作成

[ステップ1](#ステップ1-プロジェクトの作成)で作成したプロジェクトの下で、AWSにホストされるTiDB Dedicatedクラスターを作成します。詳細な手順については、[このドキュメント](/tidb-cloud/create-tidb-cluster.md)を参照してください。クラスターが配置されるリージョンが[ステップ2](/tidb-cloud/tidb-cloud-encrypt-cmek.md#step-2-complete-the-cmek-configuration-of-the-project)と同じであることを確認してください。

> **注意:**
>
> CMEKが有効になると、クラスターのノードで使用されるEBSボリュームとクラスターバックアップに使用されるS3は、CMEKを使用して暗号化されます。

## CMEKのローテーション

AWS KMSで[自動CMEKローテーション](http://docs.aws.amazon.com/kms/latest/developerguide/rotate-keys.html)を構成できます。このローテーションが有効になっている場合、TiDB Cloudのプロジェクト設定の**暗号化アクセス**を更新する必要がありません（CMEK IDを含む）。

## CMEKの取り消しと復元

TiDB CloudからCMEKへのアクセスを一時的に取り消す必要がある場合は、次の手順に従ってください。

1. AWS KMSコンソールで対応する権限を取り消し、KMSキーポリシーを更新します。
2. TiDB Cloudコンソールで、プロジェクト内のすべてのクラスターを一時停止します。

> **注意:**
>
```
> After you revoke CMEK on AWS KMS, your running clusters are not affected. However, when you pause a cluster and then restore the cluster, the cluster will not be able to restore normally because it cannot access CMEK.

After revoking TiDB Cloud's access to CMEK, if you need to restore the access, follow these steps:

1. On the AWS KMS console, restore the CMEK access policy.
2. On the TiDB Cloud console, restore all clusters in the project.
```

```
> AWS KMSでCMEKの取り消しを行った後、実行中のクラスターに影響はありません。ただし、クラスターを一時停止し、その後クラスターを復元すると、CMEKにアクセスできないため、クラスターを通常に復元することができません。

TiDB CloudのCMEKへのアクセスを取り消した後、アクセスを復元する必要がある場合は、以下の手順に従ってください：

1. AWS KMSコンソールでCMEKアクセスポリシーを復元します。
2. TiDB Cloudコンソールでプロジェクト内のすべてのクラスターを復元します。
```