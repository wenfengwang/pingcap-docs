---
title: Google Cloud プライベートサービス接続を使用して TiDB Dedicated クラスターに接続する
summary: TiDB Cloud クラスターに Google Cloud プライベートサービス接続を使用して接続する方法について学びます。

# Google Cloud プライベートサービス接続を使用して TiDB Dedicated クラスターに接続する

このドキュメントでは、Google Cloud プライベートサービス接続を使用して TiDB Dedicated クラスターに接続する方法について説明します。Google Cloud プライベートサービス接続は、Google Cloud が提供するプライベートエンドポイントサービスです。

> **ヒント:**
>
> - AWS でプライベートエンドポイントを使用して TiDB Dedicated クラスターに接続する方法については、[AWS を使用して TiDB Dedicated にプライベートエンドポイントで接続する](/tidb-cloud/set-up-private-endpoint-connections.md) を参照してください。
> - プライベートエンドポイントを使用して TiDB Serverless クラスターに接続する方法については、[プライベートエンドポイントを使用して TiDB Serverless に接続する](/tidb-cloud/set-up-private-endpoint-connections-serverless.md) を参照してください。

TiDB Cloud は、[プライベートサービス接続](https://cloud.google.com/vpc/docs/private-service-connect) を介して Google Cloud VPC にホストされている TiDB Cloud サービスへの非常に安全で片方向のアクセスをサポートしています。エンドポイントを作成して、TiDB Cloud サービスに接続できます。

Google Cloud プライベートサービス接続によって提供されるエンドポイント接続は、セキュアでプライベートであり、データをインターネットに公開しません。また、CIDR のオーバーラップをサポートし、ネットワーク管理が容易です。

プライベートエンドポイントのアーキテクチャは以下の通りです：

![Private Service Connect architecture](/media/tidb-cloud/google-cloud-psc-endpoint-overview.png)

プライベートエンドポイントおよびエンドポイントサービスの詳細な定義については、以下の Google Cloud ドキュメントを参照してください：

- [プライベートサービス接続](https://cloud.google.com/vpc/docs/private-service-connect)
- [エンドポイントを介して公開サービスにアクセス](https://cloud.google.com/vpc/docs/configure-private-service-connect-services)

## 制限事項

- この機能は2023年4月13日以降に作成された TiDB Dedicated クラスターに適用されます。古いクラスターの場合、[TiDB Cloud サポート](/tidb-cloud/tidb-cloud-support.md)にお問い合わせください。
- `Organization Owner` および `Project Owner` ロールのみが Google Cloud プライベートサービス接続エンドポイントを作成できます。
- 各 TiDB クラスターは最大 10 個のエンドポイントからの接続を処理できます。
- 各 Google Cloud プロジェクトには、TiDB クラスターへの接続を行う最大 10 個のエンドポイントを持つことができます。
- エンドポイントサービスが設定されたプロジェクト内で最大 8 個の Google Cloud にホストされている TiDB Dedicated クラスターを作成できます。
- 接続を行うプライベートエンドポイントと接続される TiDB クラスターは同じリージョンに配置する必要があります。
- イーグレスファイアウォールルールは、エンドポイントの内部 IP アドレスへのトラフィックを許可する必要があります。[暗黙の許可イーグレスファイアウォールルール](https://cloud.google.com/firewall/docs/firewalls#default_firewall_rules) は、任意の宛先 IP アドレスへのエグレスを許可します。
- VPC ネットワークにイーグレス拒否ファイアウォールルールを作成している場合、または暗黙の許可されたエグレスの動作を変更する階層ファイアウォールポリシーを作成している場合、エンドポイントへのアクセスに影響を与える可能性があります。この場合、エンドポイントの内部宛先IP アドレスにトラフィックを許可する特定のエグレス許可ファイアウォールルールまたはポリシーを作成する必要があります。

ほとんどのシナリオでは、VPC ピアリングよりもプライベートエンドポイント接続を使用することを推奨します。ただし、次のシナリオでは、VPC ピアリングの代わりにプライベートエンドポイント接続を使用する必要があります：

- [TiCDC](https://docs.pingcap.com/tidb/stable/ticdc-overview) クラスターを使用して、ソース TiDB クラスターからターゲット TiDB クラスターにデータを別のリージョンにレプリケートして、高可用性を実現している場合。現時点では、プライベートエンドポイントはクロスリージョン接続をサポートしていません。
- TiCDC クラスターを使用して、下流クラスター（Amazon Aurora、MySQL、Kafka など）にデータをレプリケートするが、下流のエンドポイントサービスを自分で維持することができない場合。

## Google Cloud プライベートサービス接続を使用してプライベートエンドポイントを設定する

プライベートエンドポイントを使用して TiDB Dedicated クラスターにプライベートエンドポイント経由で接続するためには、[前提条件](#前提条件)を完了し、以下の手順に従います：

1. [TiDB クラスターを選択](#ステップ-1-tidb-クラスターを選択)
2. [エンドポイントの作成情報を提供](#ステップ-2-エンドポイントの作成情報を提供)
3. [エンドポイントへのアクセスを受け入れる](#ステップ-3-エンドポイントへのアクセスを受け入れる)
4. [TiDB クラスターに接続する](#ステップ-4-tidb-クラスターに接続する)

複数のクラスターを持っている場合、Google Cloud プライベートサービス接続を使用して接続する各クラスターについてこれらの手順を繰り返す必要があります。

### 前提条件

エンドポイントを作成する前に：

- Google Cloud プロジェクトで以下の API を[有効化](https://console.cloud.google.com/apis/library/compute.googleapis.com)します：
    - [Compute Engine API](https://cloud.google.com/compute/docs/reference/rest/v1)
    - [Service Directory API](https://cloud.google.com/service-directory/docs/reference/rest)
    - [Cloud DNS API](https://cloud.google.com/dns/docs/reference/v1)

- 以下の [IAM ロール](https://cloud.google.com/iam/docs/understanding-roles) を用意し、エンドポイントの作成に必要な権限を付与します。

    - タスク：
        - エンドポイントを作成
        - エンドポイントのために[DNS エントリ](https://cloud.google.com/vpc/docs/configure-private-service-connect-services#dns-endpoint)を自動的にまたは手動で構成する
    - 必要な IAM ロール：
        - [Compute Network Admin](https://cloud.google.com/iam/docs/understanding-roles#compute.networkAdmin) (roles/compute.networkAdmin)
        - [Service Directory Editor](https://cloud.google.com/iam/docs/understanding-roles#servicedirectory.editor) (roles/servicedirectory.editor)

以下の手順に従って**Google Cloud プライベートエンドポイント**ページに移動します：

1. [TiDB Cloud コンソール](https://tidbcloud.com)にログインします。
2. 左下の <MDSvgIcon name="icon-left-projects" /> をクリックして、複数のプロジェクトを持っている場合は対象のプロジェクトに切り替え、**プロジェクトの設定**をクリックします。
3. プロジェクトの**プロジェクト設定**ページで、左側のナビゲーションペインで **ネットワーク アクセス** をクリックし、**プライベート エンドポイント**タブをクリックします。
4. 右上の**プライベートエンドポイントの作成**をクリックし、**Google Cloud プライベートエンドポイント**を選択します。

### ステップ 1. TiDB クラスターを選択

ドロップダウンリストをクリックし、利用可能な TiDB Dedicated クラスターを選択します。

以下のいずれかのステータスを持つクラスターを選択できます：

- **利用可能**
- **リストア中**
- **変更中**
- **インポート中**

### ステップ 2. エンドポイントの作成情報を提供

1. プライベートエンドポイントの作成コマンドを生成するために、以下の情報を提供します：
   - **Google Cloud プロジェクト ID**：Google Cloud アカウントに関連付けられたプロジェクト ID。[Google Cloud の**ダッシュボード**ページ](https://console.cloud.google.com/home/dashboard)で ID を見つけることができます。
   - **Google Cloud VPC 名**：指定したプロジェクト内の VPC の名前。[Google Cloud の **VPC ネットワーク**ページ](https://console.cloud.google.com/networking/networks/list)で確認できます。
   - **Google Cloud サブネット名**：指定された VPC 内のサブネットの名前。**VPC ネットワークの詳細**ページで確認できます。
   - **プライベートサービス接続エンドポイント名**：作成されるプライベートエンドポイントの一意の名前を入力します。
2. 情報を入力した後、**コマンドを生成**をクリックします。
3. コマンドをコピーします。
4. [Google Cloud Shell](https://console.cloud.google.com/home/dashboard)に移動して、コマンドを実行します。

### ステップ 3. エンドポイントへのアクセスを受け入れる

Google Cloud Shell でコマンドを正常に実行した後、TiDB Cloud コンソールに戻り、**エンドポイント アクセスを受け入れる**をクリックします。

エラー `エンドポイントからの接続要求を受け取っていません` と表示された場合は、コマンドを正しくコピーして Google Cloud Shell で正常に実行されていることを確認してください。

### ステップ 4. TiDB クラスターに接続する

エンドポイント接続を受け入れた後は、TiDB クラスターに接続するために以下の手順を実行します：

1. [**クラスター**](https://tidbcloud.com/console/clusters)ページで、 **...** を **アクション** 列でクリックします。
2. **Connect** をクリックします。接続ダイアログボックスが表示されます。
3. **プライベート エンドポイント**タブを選択します。作成したプライベートエンドポイントが表示されます。TiDB クラスターに接続するためのコマンドをコピーします。

### 　プライベートエンドポイントのステータス参照

プライベートエンドポイント接続を使用すると、プライベートエンドポイントやプライベートエンドポイントサービスのステータスが [**プライベートエンドポイント**ページ](#前提条件)に表示されます。

プライベート エンドポイントの可能なステータスは次のように説明されています：

- **保留中**：処理を待っています。
- **アクティブ**：プライベートエンドポイントが使用可能です。このステータスのプライベートエンドポイントを編集することはできません。
- **削除中**：プライベートエンドポイントが削除されています。
- **失敗**：プライベートエンドポイントの作成に失敗しました。その行の**編集**をクリックして作成を再試行できます。

プライベートエンドポイント サービスの可能なステータスは次のとおりです：

- **作成中**：3 から 5 分かかるエンドポイント サービスが作成中です。
- **アクティブ**：エンドポイント サービスが作成されました。プライベートエンドポイントが作成されているかどうかにかかわらずです。

## トラブルシューティング

### TiDB Cloud はエンドポイント サービスを作成できません。どうすればよいですか？

```
The endpoint service is created automatically after you open the **Create Google Cloud Private Endpoint** page and choose the TiDB cluster. If it shows as failed or remains in the **Creating** state for a long time, submit a [support ticket](/tidb-cloud/tidb-cloud-support.md) for assistance.

### Fail to create an endpoint in Google Cloud. What should I do?

To troubleshoot the issue, you need to review the error message returned by Google Cloud Shell after you execute the private endpoint creation command. If it is a permission-related error, you must grant the necessary permissions before retrying.

### I cancelled some actions. What should I do to handle cancellation before accepting endpoint access?

Unsaved drafts of cancelled actions will not be retained or displayed. You need to repeat each step when creating a new private endpoint in the TiDB Cloud console next time.

If you have already executed the command to create a private endpoint in Google Cloud Shell, you need to manually [delete the corresponding endpoint](https://cloud.google.com/vpc/docs/configure-private-service-connect-services#delete-endpoint) in the Google Cloud console.

### Why can't I see the endpoints generated by directly copying the service attachment in the TiDB Cloud console?

In the TiDB Cloud console, you can only view endpoints that are created through the command generated on the **Create Google Cloud Private Endpoint** page.

However, endpoints generated by directly copying the service attachment (that is, not created through the command generated in the TiDB Cloud console) are not displayed in the TiDB Cloud console.
```