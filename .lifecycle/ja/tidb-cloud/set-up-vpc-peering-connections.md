---
title: VPCピアリングを介してTiDB専用に接続する
summary: TiDB専用をVPCピアリングを介して接続する方法を学びます。

# VPCピアリングを介してTiDB専用に接続する

> **注意：**
>
> VPCピアリング接続はTiDB専用クラスターのみに利用可能です。[TiDBサーバーレスクラスター](/tidb-cloud/select-cluster-tier.md#tidb-serverless)にVPCピアリングを使用して接続できません。

TiDBクラウドへのアプリケーションのVPCピアリング経由での接続を行うには、TiDBクラウドとの[VPCピアリング](/tidb-cloud/tidb-cloud-glossary.md#vpc-peering)を設定する必要があります。この文書では、AWSでのVPCピアリング接続の設定方法について、[AWS](#set-up-vpc-peering-on-aws)および[Google Cloud](#set-up-vpc-peering-on-google-cloud)でのVPCピアリング接続の設定と、VPCピアリングを介したTiDBクラウドへの接続について説明します。

VPCピアリング接続とは、2つのVPC間のネットワーク接続であり、プライベートIPアドレスを使用してそれらの間のトラフィックをルーティングできるようにします。どちらのVPCのインスタンスも、同じネットワーク内にあるかのように通信できます。

現在、TiDBクラウドは同じリージョンの同じプロジェクトに対してVPCピアリングをサポートしています。同じプロジェクト内の同じリージョンに作成されたTiDBクラスターは同じVPCに作成されます。したがって、プロジェクトのリージョンでVPCピアリングが設定されていると、このプロジェクトの同じリージョンに作成されたすべてのTiDBクラスターはVPC内で接続されます。VPCピアリングの設定方法はクラウドプロバイダーによって異なります。

> **ヒント：**
>
> アプリケーションをTiDBクラウドに接続するには、[TiDBクラウドとのプライベートエンドポイント接続](/tidb-cloud/set-up-private-endpoint-connections.md)も設定できます。これはセキュアでプライベートであり、データをインターネットに公開しません。VPCピアリング接続よりもプライベートエンドポイントを使用することが推奨されます。

## 前提条件：プロジェクトCIDRの設定

プロジェクトCIDR（Classless Inter-Domain Routing）とは、プロジェクトにおけるネットワークピアリングに使用されるCIDRブロックです。

特定のリージョンにVPCピアリングリクエストを追加する前に、プロジェクトのクラウドプロバイダ（AWSまたはGoogle Cloud）に対してプロジェクトCIDRを設定し、アプリケーションのVPCに対するピアリングリンクを確立する必要があります。

プロジェクトCIDRは、プロジェクトの最初のTiDB専用クラスターを作成する際に設定できます。クラスターの作成前にプロジェクトCIDRを設定したい場合は、以下の手順を実行してください：

1. [TiDBクラウドコンソール](https://tidbcloud.com)にログインします。
2. 左下隅の<MDSvgIcon name="icon-left-projects" />をクリックし、複数のプロジェクトを持っている場合はターゲットプロジェクトに切り替えてから**Project Settings**をクリックします。
3. プロジェクトの**Project Settings**ページで、左のナビゲーションペインで**Network Access**をクリックし、次に**Project CIDR**タブをクリックします。
4. クラウドプロバイダに応じて、**Project CIDR**フィールドに以下のネットワークアドレスのいずれかを指定し、**Confirm**をクリックします。

    > **注意：**
    >
    > アプリケーションが存在するVPCのCIDRとの競合を避けるために、このフィールドには異なるプロジェクトCIDRを設定する必要があります。

    - 10.250.0.0/16
    - 10.250.0.0/17
    - 10.250.128.0/17
    - 172.30.0.0/16
    - 172.30.0.0/17
    - 172.30.128.0/17

    ![Project-CIDR4](/media/tidb-cloud/Project-CIDR4.png)

5. クラウドプロバイダと特定のリージョンのCIDRを表示します。

    リージョンCIDRはデフォルトで非アクティブです。リージョンCIDRを有効にするには、ターゲットリージョンでクラスターを作成する必要があります。リージョンCIDRがアクティブになると、そのリージョンに対しVPCピアリングを作成できます。

    ![Project-CIDR2](/media/tidb-cloud/Project-CIDR2.png)

## AWSでのVPCピアリングの設定

このセクションでは、AWSでのVPCピアリング接続の設定方法について説明します。Google Cloudについては、[Google CloudでのVPCピアリングの設定](#set-up-vpc-peering-on-google-cloud)を参照してください。

### Step 1. VPCピアリングリクエストの追加

1. [TiDBクラウドコンソール](https://tidbcloud.com)にログインします。
2. 左下隅の<MDSvgIcon name="icon-left-projects" />をクリックし、複数のプロジェクトを持っている場合はターゲットプロジェクトに切り替えてから**Project Settings**をクリックします。
3. プロジェクトの**Project Settings**ページで、左のナビゲーションペインで**Network Access**をクリックし、次に**VPC Peering**タブをクリックします。

    **VPC Peering**構成がデフォルトで表示されます。

4. **Add**をクリックし、AWSアイコンを選択し、既存のAWS VPCの必要な情報を入力してください：

    - リージョン
    - AWSアカウントID
    - VPC ID
    - VPC CIDR

    これらの情報はVPCダッシュボードのVPCの詳細から取得できます。

    ![VPCピアリング](/media/tidb-cloud/vpc-peering/vpc-peering-creating-infos.png)

5. **Initialize**をクリックします。**Approve VPC Peerings**ダイアログが表示されます。

### Step 2. VPCピアリングの承認と構成

VPCピアリング接続を承認および構成するために、以下のいずれかのオプションを使用できます：

- [Option 1：AWS CLIを使用する](#option-1-use-aws-cli)
- [Option 2：AWSダッシュボードを使用する](#option-2-use-the-aws-dashboard)

#### Option 1：AWS CLIを使用する

1. AWS Command Line Interface（AWS CLI）をインストールします。

    {{< copyable "shell-regular" >}}

    ```bash
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    unzip awscliv2.zip
    sudo ./aws/install
    ```

2. AWS CLIをアカウント情報に応じて構成します。AWS CLIに必要な情報を取得するには、「[AWS CLI configuration basics](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html)」を参照してください。

    {{< copyable "shell-regular" >}}

    ```bash
    aws configure
    ```

3. 次の変数の値をアカウント情報で置き換えます。

    {{< copyable "shell-regular" >}}

    ```bash
    # 関連する変数を設定します。
    pcx_tidb_to_app_id="<TiDBピアリングID>"
    app_region="<アプリケーションのリージョン>"
    app_vpc_id="<あなたのVPC ID>"
    tidbcloud_project_cidr="<TiDB CloudプロジェクトVPC CIDR>"
    ```

    例：

    ```
    # 関連する変数を設定します
    pcx_tidb_to_app_id="pcx-069f41efddcff66c8"
    app_region="us-west-2"
    app_vpc_id="vpc-0039fb90bb5cf8698"
    tidbcloud_project_cidr="10.250.0.0/16"
    ```

4. 次のコマンドを実行します。

    {{< copyable "shell-regular" >}}

    ```bash
    # VPCピアリング接続リクエストを承認します。
    aws ec2 accept-vpc-peering-connection --vpc-peering-connection-id "$pcx_tidb_to_app_id"
    ```

    {{< copyable "shell-regular" >}}

    ```bash
    # ルートテーブルのルールを作成します。
    aws ec2 describe-route-tables --region "$app_region" --filters Name=vpc-id,Values="$app_vpc_id" --query 'RouteTables[*].RouteTableId' --output text | tr "\t" "\n" | while read row
    do
        app_route_table_id="$row"
        aws ec2 create-route --route-table-id "$app_route_table_id" --destination-cidr-block "$tidbcloud_project_cidr" --vpc-peering-connection-id "$pcx_tidb_to_app_id"
    done
    ```

    > **注意：**
    >
    > ルートテーブルのルールが正常に作成されても、「An error occurred (MissingParameter) when calling the CreateRoute operation: The request must contain the parameter routeTableId」というエラーメッセージが表示されることがあります。この場合は作成したルールを確認し、エラーを無視してください。

    {{< copyable "shell-regular" >}}

    ```bash
    # VPC属性を変更してDNSホスト名とDNSサポートを有効にします。
    aws ec2 modify-vpc-attribute --vpc-id "$app_vpc_id" --enable-dns-hostnames
    aws ec2 modify-vpc-attribute --vpc-id "$app_vpc_id" --enable-dns-support
    ```

構成を完了すると、VPCピアリングが作成されます。結果を確認するために[TiDBクラスターに接続](#connect-to-the-tidb-cluster)できます。

#### Option 2：AWSダッシュボードを使用する
AWSのダッシュボードを使用して、VPCピアリング接続を構成することもできます。

1. AWSコンソールでピア接続要求を承諾することを確認してください。

    1. AWSコンソールにサインインし、上部メニューバーで **サービス** をクリックします。検索ボックスに `VPC` と入力し、VPCサービスページに移動します。

        ![AWSダッシュボード](/media/tidb-cloud/vpc-peering/aws-vpc-guide-1.jpg)

    2. 左側のナビゲーションバーから、**Peering Connections（ピアリング接続）** ページを開きます。**Create Peering Connection（ピアリング接続の作成）** タブで、ピアリング接続が **Pending Acceptance（保留中の承認）** の状態にあります。

    3. リクエスターの所有者が TiDB Cloud (`380838443567`) であることを確認してください。ピアリング接続を右クリックし、**Accept Request（リクエストを受諾）** を選択して、**Accept VPC peering connection request（VPCピアリング接続リクエストの受諾）** ダイアログでリクエストを承諾します。

        ![AWS VPCピアリングリクエスト](/media/tidb-cloud/vpc-peering/aws-vpc-guide-3.png)

2. 各VPCサブネットのルートテーブルにTiDB Cloud VPCにルートを追加します。

    1. 左側のナビゲーションバーから **Route Tables（ルートテーブル）** ページを開きます。

    2. アプリケーションVPCに属するすべてのルートテーブルを検索します。

        ![VPCに関連するすべてのルートテーブルを検索](/media/tidb-cloud/vpc-peering/aws-vpc-guide-4.png)

    3. 各ルートテーブルを右クリックし、**Edit routes（ルートを編集）** を選択します。編集ページで、Project CIDR（TiDB Cloudコンソールの **VPC Peering（VPCピアリング）** 構成ページを確認）への宛先を持つルートを追加し、**Target（ターゲット）** 列にピアリング接続のIDを入力します。

        ![すべてのルートテーブルを編集](/media/tidb-cloud/vpc-peering/aws-vpc-guide-5.png)

3. VPCのプライベートDNSホストゾーンサポートを有効にしてください。

    1. 左側のナビゲーションバーから **Your VPCs（あなたのVPC）** ページを開きます。

    2. アプリケーションVPCを選択します。

    3. 選択したVPCを右クリックします。設定のドロップダウンリストが表示されます。

    4. ドロップダウンリストから、**Edit DNS hostnames（DNSホスト名を編集）** をクリックします。DNSホスト名を有効にし、**Save（保存）** をクリックします。

    5. ドロップダウンリストから、**Edit DNS resolution（DNS解決を編集）** をクリックします。DNS解決を有効にし、**Save（保存）** をクリックします。

これでVPCピアリング接続の設定が完了しました。次に、[VPCピアリング経由でTiDBクラスタに接続します](#connect-to-the-tidb-cluster)。