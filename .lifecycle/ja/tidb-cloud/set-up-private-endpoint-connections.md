---
title: AWSを使用してプライベートエンドポイント経由でTiDB専用クラスタに接続する方法
summary: AWSを使用してTiDB Cloudクラスタにプライベートエンドポイント経由で接続する方法を学びます。

# AWSを使用してプライベートエンドポイント経由でTiDB専用クラスタに接続する

このドキュメントでは、AWSを使用してプライベートエンドポイント経由でTiDB専用クラスタに接続する方法について説明します。

> **ヒント：**
>
> TiDBサーバーレスクラスタにプライベートエンドポイント経由で接続する方法については、[プライベートエンドポイント経由でTiDBサーバーレスに接続](/tidb-cloud/set-up-private-endpoint-connections-serverless.md)を参照してください。
> Google Cloudを使用してTiDB専用クラスタにプライベートエンドポイント経由で接続する方法については、[Google Cloudでのプライベートサービス接続を使用したTiDB専用への接続](/tidb-cloud/set-up-private-endpoint-connections-on-google-cloud.md)を参照してください。

TiDB Cloudでは、AWS VPCにホストされたTiDB Cloudサービスへの高度にセキュアで片方向のアクセスを [AWS PrivateLink](https://aws.amazon.com/privatelink/?privatelink-blogs.sort-by=item.additionalFields.createdDate&privatelink-blogs.sort-order=desc) 経由でサポートしています。自分自身のVPC内にサービスがあるかのように、プライベートエンドポイントがVPCに公開され、許可を得てエンドポイント経由でTiDB Cloudサービスに接続できます。

AWS PrivateLinkにより、エンドポイント接続は安全かつプライベートであり、データを公共インターネットに公開しません。さらに、エンドポイント接続はCIDRオーバーラップをサポートし、ネットワークの管理が容易です。

プライベートエンドポイントのアーキテクチャは以下の通りです：

![プライベートエンドポイントアーキテクチャ](/media/tidb-cloud/aws-private-endpoint-arch.png)

プライベートエンドポイントとエンドポイントサービスの詳細定義については、次のAWSのドキュメントを参照してください：

- [AWS PrivateLinkについて](https://docs.aws.amazon.com/vpc/latest/privatelink/what-is-privatelink.html)
- [AWS PrivateLinkのコンセプト](https://docs.aws.amazon.com/vpc/latest/privatelink/concepts.html)

## 制限事項

- `組織所有者` および `プロジェクト所有者` のロールのみがプライベートエンドポイントを作成できます。
- 接続するプライベートエンドポイントおよびTiDBクラスタは同じリージョンに配置する必要があります。

ほとんどのシナリオでは、VPCピアリングよりもプライベートエンドポイント接続を使用することがお勧めされます。ただし、次のシナリオでは、VPCピアリングの代わりにプライベートエンドポイント接続を使用する必要があります：

- [TiCDC](https://docs.pingcap.com/tidb/stable/ticdc-overview) クラスタを使用して、ソースTiDBクラスタからターゲットTiDBクラスタにデータを複製し、高可用性を得るために、リージョン間でデータを複製します。現在、プライベートエンドポイントはクロスリージョン接続をサポートしていません。
- TiCDCクラスタを使用して、下流クラスタ (Amazon Aurora、MySQL、Kafkaなど) にデータを複製しますが、エンドポイントサービスを自分で維持できません。
- PDまたはTiKVノードに直接接続している場合。

## AWSでプライベートエンドポイントを設定する

プライベートエンドポイントを介してTiDB専用クラスタに接続するには、[前提条件](#前提条件) を完了し、以下の手順に従ってください：

1. [TiDBクラスタを選択する](#ステップ-1-choose-a-tidb-cluster)
2. [サービスエンドポイントのリージョンを確認する](#ステップ-2-check-the-service-endpoint-region)
3. [AWSインターフェイスエンドポイントを作成する](#ステップ-3-create-an-aws-interface-endpoint)
4. [エンドポイント接続を承認する](#ステップ-4-accept-the-endpoint-connection)
5. [プライベートDNSを有効にする](#ステップ-5-enable-private-dns)
6. [TiDBクラスタに接続する](#ステップ-6-connect-to-your-tidb-cluster)

複数のクラスタを持っている場合、AWS PrivateLinkを使用して接続する各クラスタごとにこれらの手順を繰り返す必要があります。

### 前提条件

1. [TiDB Cloudコンソール](https://tidbcloud.com) にログインします。
2. 左下隅の <MDSvgIcon name="icon-left-projects" /> をクリックし、複数のプロジェクトを持っている場合は対象のプロジェクトに切り替え、次に **プロジェクトの設定** をクリックします。
3. プロジェクトの **プロジェクト設定** ページで、左側のナビゲーションペインで **ネットワークアクセス** をクリックし、**プライベートエンドポイント** タブをクリックします。
4. 右上隅の **プライベートエンドポイントの作成** をクリックし、次に **AWSプライベートエンドポイント** を選択します。

### ステップ 1. TiDBクラスタを選択する

1. ドロップダウンリストをクリックして使用可能なTiDB専用クラスタを選択します。
2. **次へ** をクリックします。

### ステップ 2. サービスエンドポイントのリージョンを確認する

サービスエンドポイントのリージョンがデフォルトで選択されています。素早くチェックして **次へ** をクリックします。

> **注意：**
>
> デフォルトのリージョンは、クラスタの位置がある場所です。変更しないでください。クロスリージョンのプライベートエンドポイントは現在サポートされていません。

### ステップ 3. AWSインターフェイスエンドポイントを作成する

> **注意：**
>
> 2023年3月28日以降に作成された各TiDB専用クラスタに対応するエンドポイントサービスは、クラスタ作成後3〜4分後に自動的に作成されます。

`エンドポイントサービスの準備完了` メッセージが表示された場合、コンソールの下部でのコマンドから後で使用するエンドポイントサービス名をメモしてください。それ以外の場合は、3〜4分待ってTiDB Cloudがクラスタのエンドポイントサービスを作成するのを待ちます。

```bash
aws ec2 create-vpc-endpoint --vpc-id ${your_vpc_id} --region ${your_region} --service-name ${your_endpoint_service_name} --vpc-endpoint-type Interface --subnet-ids ${your_application_subnet_ids}
```

次に、AWSマネジメントコンソールを使用するか、AWS CLIを使用してAWSインターフェイスエンドポイントを作成します。

<SimpleTab>
<div label="AWSコンソールを使用する">

AWSマネジメントコンソールを使用してVPCインターフェイスエンドポイントを作成するには、次の手順を実行します：

1. [AWS Management Console](https://aws.amazon.com/console/) にサインインし、Amazon VPCコンソールを開きます：<https://console.aws.amazon.com/vpc/>。
2. ナビゲーションペインで **エンドポイント** をクリックし、右上隅の **エンドポイントの作成** をクリックします。

    **エンドポイントの作成** ページが表示されます。

    ![エンドポイントサービスの確認](/media/tidb-cloud/private-endpoint/create-endpoint-2.png)

3. **その他のエンドポイントサービス** を選択します。
4. TiDB Cloudコンソールで見つけたサービス名を入力します。
5. **サービスを確認** をクリックします。
6. ドロップダウンリストでVPCを選択します。
7. **サブネット** 領域で、TiDBクラスタが配置されている可用ゾーンを選択します。

    > **ヒント：**
    >
    > サービスが3つ以上の可用ゾーン (AZs) をまたがっている場合は、 **サブネット** 領域でAZを選択できないことがあります。これは、選択したリージョンにTiDBクラスタの他にもう1つの余分なAZがある場合に発生する問題です。このような場合は、[PingCAPテクニカルサポート](https://docs.pingcap.com/tidbcloud/tidb-cloud-support)に連絡してください。

8. **セキュリティグループ** 領域で適切なセキュリティグループを選択します。

    > **注意：**
    >
    > 選択したセキュリティグループが、EC2インスタンスからPort 4000またはカスタマー定義ポートへのインバウンドアクセスを許可していることを確認してください。

9. **エンドポイントの作成** をクリックします。

</div>
<div label="AWS CLIを使用する">

AWS CLIを使用してVPCインターフェイスエンドポイントを作成するには、次の手順を実行します：

1. プライベートエンドポイントの作成ページで **VPC ID** および **Subnet IDs** のフィールドを入力します。IDはAWSマネジメントコンソールから取得できます。
2. ページの下部にあるコマンドをコピーしてターミナルで実行します。次に **次へ** をクリックします。

> **ヒント：**
>
> - コマンドを実行する前に、AWS CLIをインストールして構成する必要があります。詳細については、[AWS CLIの構成基本](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html)を参照してください。
>
> - サービスが3つ以上の可用ゾーン (AZs) をまたがっている場合、サブネットのAZをサポートしていないというエラーメッセージが表示されます。これは、選択したリージョンにTiDBクラスタの他にもう1つの余分なAZがある場合に発生する問題です。このような場合は、[PingCAPテクニカルサポート](https://docs.pingcap.com/tidbcloud/tidb-cloud-support)に連絡できます。
>
> - TiDB Cloudがバックグラウンドでエンドポイントサービスを作成を完了するまで、コマンドをコピーすることはできません。

</div>
</SimpleTab>

### ステップ 4. エンドポイント接続を承認する

1. TiDB Cloudコンソールに戻ります。
2. **プライベートエンドポイントの作成** ページのボックスにVPCエンドポイントIDを入力します。
3. **次へ** をクリックします。

### ステップ 5. プライベートDNSを有効にする

AWSでプライベートDNSを有効にします。AWSマネジメントコンソールまたはAWS CLIを使用できます。

<SimpleTab>
<div label="AWSコンソールを使用する">

AWSマネジメントコンソールでプライベートDNSを有効にするには：

1. **VPC** > **エンドポイント** に移動します。
2. エンドポイントIDを右クリックし、**プライベートDNS名の変更** を選択します。
3. **このエンドポイント用に有効にする** チェックボックスを選択します。
4. **変更を保存** をクリックします。

    ![プライベートDNSの有効化](/media/tidb-cloud/private-endpoint/enable-private-dns.png)

</div>
<div label="AWS CLIを使用する">

AWS CLIを使用してプライベートDNSを有効にするには、コマンドをコピーしてAWS CLIで実行します。

```bash
```
aws ec2 modify-vpc-endpoint --vpc-endpoint-id ${your_vpc_endpoint_id} --private-dns-enabled
```

</div>
</SimpleTab>

TiDB Cloudコンソールで**作成**をクリックしてプライベートエンドポイントの作成を最終確定します。

その後、エンドポイントサービスに接続できます。

### ステップ6 TiDBクラスターに接続します

プライベートDNSを有効にしたら、TiDB Cloudコンソールに戻り、次の手順を実行します。

1. [**クラスター**](https://tidbcloud.com/console/clusters) ページで、 **...**を**アクション** 列でクリックします。
2. **接続** をクリックします。接続ダイアログボックスが表示されます。
3. **プライベートエンドポイント**タブを選択します。作成したばかりのプライベートエンドポイントが **ステップ1: プライベートエンドポイントの作成**の下に表示されます。
4. **ステップ2: あなたの接続を接続します** の下で、 **接続**をクリックし、好みの接続方法のタブをクリックして、接続文字列でクラスターに接続します。接続文字列内のプレースホルダー `<cluster_endpoint_name>:<port>` は自動的に実際の値に置換されます。

> **ヒント:**
>
> クラスターに接続できない場合、原因は、AWSのVPCエンドポイントのセキュリティグループが適切に設定されていない可能性があります。解決策については、[このFAQ](#troubleshooting) を参照してください。

### プライベートエンドポイントのステータス参照

プライベートエンドポイントの接続を使用すると、プライベートエンドポイントまたはプライベートエンドポイントサービスのステータスが [**プライベートエンドポイント** ページ](#prerequisites) に表示されます。

プライベートエンドポイントの可能なステータスは次のとおりです。

- **構成されていません**: エンドポイントサービスは作成されていますが、プライベートエンドポイントはまだ作成されていません。
- **保留中**: 処理を待っています。
- **アクティブ**: プライベートエンドポイントは使用可能です。このステータスのプライベートエンドポイントを編集することはできません。
- **削除中**: プライ ベートエンドポイントが削除されています。
- **失敗**: プライベートエンドポイントの作成に失敗しました。その行の **編集** をクリックして作成を再試行できます。

プライウベートエンドポイントサービスの可能なステータスは次のとおりです。

- **作成中**: エンドポイントサービスが作成中です。3から5分かかります。
- **アクティブ**: エンドポイントサービスが作成されました。プライベートエンドポイントが作成されているかどうかにかかわらずです。
- **削除中**: エンドポイントサービスまたはクラスターが削除中です。3から5分かかります。

## トラブルシューティング

### プライベートDNSを有効にした後にプライベートエンドポイントを介してTiDBクラスターに接続できません。どうしてですか？

AWS管理コンソールの **VPC** > **エンドポイント** に移動して、VPCエンドポイントを右クリックし、適切な **セキュリティグループを管理** を選択する必要があるかもしれません。VPC内の適切なセキュリティグループを選択し、Port 4000または顧客定義ポートへのEC2インスタンスからの受信アクセスを許可する必要があります。

![セキュリティグループの管理](/media/tidb-cloud/private-endpoint/manage-security-groups.png)

### プライベートDNSを有効にできません。`enableDnsSupport` および `enableDnsHostnames` VPC属性が有効化されていないというエラーが報告されています。

VPC設定でDNSホスト名とDNS解決がいずれも有効になっていることを確認してください。AWS Management ConsoleでVPCを作成する際、これらはデフォルトで無効になっています。