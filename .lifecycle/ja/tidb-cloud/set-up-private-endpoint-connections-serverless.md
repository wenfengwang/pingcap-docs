---
title: プライベートエンドポイントを介した TiDB サーバーレスへの接続
summary: TiDB Cloud クラスターへのプライベートエンドポイントを介した接続方法について学びます。

# プライベートエンドポイントを介した TiDB サーバーレスへの接続

このドキュメントでは、TiDB サーバーレスクラスターへのプライベートエンドポイントを介した接続方法について説明します。

> **ヒント:**
>
> AWS を使用して TiDB Dedicated クラスターにプライベートエンドポイントを介して接続する方法については、[AWS を使用した TiDB Dedicated へのプライベートエンドポイントを介した接続方法](/tidb-cloud/set-up-private-endpoint-connections.md) を参照してください。
> Google Cloud を使用して TiDB Dedicated クラスターにプライベートエンドポイントを介して接続する方法については、[Google Cloud を使用した TiDB Dedicated へのプライベートサービス接続方法](/tidb-cloud/set-up-private-endpoint-connections-on-google-cloud.md) を参照してください。

TiDB Cloud は、AWS VPC にホストされた TiDB Cloud サービスへの高度に安全で一方向のアクセスをサポートし、サービスが独自の VPC 内にあるかのように動作します。プライベートエンドポイントが VPC に公開され、許可を受けてエンドポイント経由で TiDB Cloud サービスに接続できます。

AWS PrivateLink によって動作するエンドポイント接続は安全でプライベートであり、データをインターネットに公開しません。また、エンドポイント接続は CIDR オーバーラップをサポートし、ネットワーク管理が容易です。

プライベートエンドポイントのアーキテクチャは次のとおりです:

![プライベートエンドポイントのアーキテクチャ](/media/tidb-cloud/aws-private-endpoint-arch.png)

プライベートエンドポイントおよびエンドポイントサービスの詳細な定義については、以下の AWS ドキュメントを参照してください:

- [AWS PrivateLink とは](https://docs.aws.amazon.com/vpc/latest/privatelink/what-is-privatelink.html)
- [AWS PrivateLink のコンセプト](https://docs.aws.amazon.com/vpc/latest/privatelink/concepts.html)

## 制限事項

- 現在、TiDB Cloud はエンドポイントサービスが AWS でホストされている場合にのみ TiDB サーバーレスへのプライベートエンドポイント接続をサポートしています。サービスが Google Cloud でホストされている場合、プライベートエンドポイントは適用されません。
- リージョン間のプライベートエンドポイント接続はサポートされていません。

## AWS でプライベートエンドポイントを設定する

TiDB サーバーレスクラスターへのプライベートエンドポイントを介して接続する手順は次のとおりです:

1. **TiDB クラスタを選択します** (#step-1-choose-a-tidb-cluster)
2. **AWS インターフェースエンドポイントを作成します** (#step-2-create-an-aws-interface-endpoint)
3. **TiDB クラスタに接続します** (#step-3-connect-to-your-tidb-cluster)

### ステップ 1. TiDB クラスタを選択する

1. [**クラスタ**](https://tidbcloud.com/console/clusters) ページで、ターゲットの TiDB サーバーレスクラスタ名をクリックして、その概要ページに移動します。
2. 右上隅の **Connect** をクリックします。接続ダイアログが表示されます。
3. **エンドポイント タイプ** のドロップダウンリストから **Private** を選択します。
4. **サービス名**、**可用性ゾーン ID**、および **リージョン ID** をメモしてください。

    > **注意:**
    >
    > 同じリージョンにあるすべての TiDB サーバーレスクラスタで共有できる1つの AWS リージョンにつき1つのプライベートエンドポイントを作成する必要があります。

### ステップ 2. AWS インターフェースエンドポイントを作成します

<SimpleTab>
<div label="AWS コンソールを使用">

AWS マネジメントコンソールを使用して VPC インターフェースエンドポイントを作成する手順は次のとおりです:

1. [AWS マネジメントコンソール](https://aws.amazon.com/console/) にサインインし、Amazon VPC コンソールを開きます: <https://console.aws.amazon.com/vpc/>.
2. ナビゲーションペインで **エンドポイント** をクリックし、右上隅の **Create Endpoint** をクリックします。

    **Create endpoint** ページが表示されます。

    ![エンドポイントサービスの検証](/media/tidb-cloud/private-endpoint/create-endpoint-2.png)

3. **他のエンドポイントサービス** を選択します。
4. **ステップ 1** で見つけたサービス名を入力します。
5. **サービスを検証** をクリックします。
6. ドロップダウンリストで VPC を選択します。**追加の設定** を展開し、**DNS 名の有効化** チェックボックスを選択します。
7. **サブネット** 領域で、TiDB クラスタが配置されている可用性ゾーンを選択し、サブネット ID を選択します。
8. **セキュリティグループ** 領域で適切なセキュリティグループを選択します。

    > **注意:**
    >
    > 選択したセキュリティグループがEC2 インスタンスからポート4000への受信アクセスを許可していることを確認してください。

9. **エンドポイントの作成** をクリックします。

</div>
<div label="AWS CLI を使用">

AWS CLI を使用して VPC インターフェースエンドポイントを作成する手順は次のとおりです:

1. **VPC ID** と **サブネット ID** を取得するには、AWS マネジメントコンソールに移動し、該当するセクションでそれらを探します。**ステップ 1** で見つけた **可用性ゾーン ID** を記入することを確認してください。
2. 下記のコマンドをコピーし、関連する引数を取得した情報で置き換え、ターミナルで実行します。

```bash
aws ec2 create-vpc-endpoint --vpc-id ${your_vpc_id} --region ${region_id} --service-name ${service_name} --vpc-endpoint-type Interface --subnet-ids ${your_subnet_id}
```

> **ヒント:**
>
> コマンドを実行する前に、AWS CLI をインストールし、構成を行う必要があります。詳細については、[AWS CLI の構成基本](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html) を参照してください。

</div>
</SimpleTab>

その後、プライベート DNS 名を使用してエンドポイントサービスに接続できます。

### ステップ 3: TiDB クラスタに接続します

インターフェースエンドポイントを作成した後、TiDB Cloud コンソールに戻り、次の手順を実行します:

1. [**クラスタ**](https://tidbcloud.com/console/clusters) ページで、ターゲットのクラスタ名をクリックして、その概要ページに移動します。
2. 右上隅の **Connect** をクリックします。接続ダイアログが表示されます。
3. **エンドポイント タイプ** のドロップダウンリストから **Private** を選択します。
4. **Connect With** ドロップダウンリストから希望の接続方法を選択します。対応する接続文字列がダイアログの下部に表示されます。
5. 接続文字列を使用してクラスタに接続します。

> **ヒント:**
>
> クラスタに接続できない場合、VPC エンドポイントのセキュリティグループが適切に設定されていない可能性があります。解決策については、[この FAQ](#troubleshooting) を参照してください。
>
> VPC エンドポイントを作成する際に、エラー `private-dns-enabled cannot be set because there is already a conflicting DNS domain for gatewayXX-privatelink.XX.prod.aws.tidbcloud.com in the VPC vpc-XXXXX` が発生した場合は、既にプライベートエンドポイントが作成されており、新しく作成する必要がないためです。

## トラブルシューティング

### プライベート DNS を有効にした後、TiDB クラスタにプライベートエンドポイントを介して接続できません。何が原因ですか?

AWS マネジメントコンソールで VPC エンドポイントのセキュリティグループを適切に設定する必要があるかもしれません。**VPC** > **エンドポイント** に移動し、VPC エンドポイントを右クリックし、適切な **セキュリティグループの管理** を選択します。VPC 内の適切なセキュリティグループが、EC2 インスタンスからポート4000またはユーザー定義ポートへの受信アクセスを許可していることを確認してください。

![セキュリティグループの管理](/media/tidb-cloud/private-endpoint/manage-security-groups.png)

### プライベート DNS を有効にできません。 VPC 属性の `enableDnsSupport` と `enableDnsHostnames` が無効であるというエラーが報告されています

VPC 設定で DNS ホスト名と DNS 解決の両方が有効になっていることを確認してください。これらは、AWS マネジメントコンソールで VPC を作成する際にデフォルトで無効になっています。