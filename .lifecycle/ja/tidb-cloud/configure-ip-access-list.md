---
title: IPアクセスリストを構成する
summary: TiDB Dedicatedクラスタにアクセスが許可されるIPアドレスを構成する方法を学びます。

# IPアクセスリストの構成
TiDB Cloudの各TiDB Dedicatedクラスタについて、クラスタへのインターネットトラフィックをフィルタリングするIPアクセスリストを構成できます。これはファイアウォールアクセス制御リストと同様に機能します。構成後、IPアクセスリストに含まれるIPアドレスを持つクライアントとアプリケーションのみがTiDB Dedicatedクラスタに接続できます。

> **注記:**
>
> IPアクセスリストの構成は[TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスタにのみ適用可能です。

TiDB Dedicatedクラスタでは、次のいずれかの方法でIPアクセスリストを構成できます。

- [標準接続でのIPアクセスリストの構成](#configure-an-ip-access-list-in-standard-connection)

- [セキュリティ設定でのIPアクセスリストの構成](#configure-an-ip-access-list-in-security-settings)

## 標準接続でのIPアクセスリストの構成
TiDB Dedicatedクラスタの標準接続でIPアクセスリストを構成するには、次の手順を実行してください：

1. [TiDB Cloudコンソール](https://tidbcloud.com/)にアクセスし、プロジェクトの[**Clusters**](https://tidbcloud.com/console/clusters)ページに移動します。
2. TiDB Dedicatedクラスタの行で**...**をクリックし、**Connect**を選択します。ダイアログが表示されます。
3. ダイアログで、****Standard Connection**タブにある**Step 1: Create traffic filter**を見つけ、IPアクセスリストを構成します。

    - クラスタのIPアクセスリストが設定されていない場合、**Add My Current IP Address**をクリックして現在のIPアドレスをIPアクセスリストに追加し、必要に応じて**Add Item**をクリックして他のIPアドレスを追加します。その後、**Update Filter**をクリックして構成を保存します。

        > **注記:**
        >
        > 各TiDB Dedicatedクラスタには、最大7つのIPアドレスをIPアクセスリストに追加できます。さらにIPアドレスを追加するための割り当てを申請するには、[TiDB Cloudサポート](/tidb-cloud/tidb-cloud-support.md)に連絡してください。

    - クラスタのIPアクセスリストが設定されている場合、**Edit**をクリックしてIPアドレスを追加、編集、または削除し、その後**Update Filter**をクリックして構成を保存します。

    - クラスタへの任意のIPアドレスのアクセスを許可するには（推奨されません）、**Allow Access From Anywhere**をクリックし、その後**Update Filter**をクリックします。セキュリティ上のベストプラクティスに従うと、クラスタへの任意のIPアドレスのアクセスを許可することは推奨されません。これにより、クラスタが完全にインターネットに公開され、非常に危険にさらされる可能性があります。

## セキュリティ設定でのIPアクセスリストの構成
TiDB Dedicatedクラスタのセキュリティ設定でIPアクセスリストを構成するには、次の手順を実行してください：

1. [TiDB Cloudコンソール](https://tidbcloud.com/)にアクセスし、プロジェクトの[**Clusters**](https://tidbcloud.com/console/clusters)ページに移動します。
2. TiDB Dedicatedクラスタの行で**...**をクリックし、**Security Settings**を選択します。セキュリティ設定のダイアログが表示されます。
3. ダイアログで、以下のようにIPアクセスリストを構成します：

    - 現在のIPアドレスをIPアクセスリストに追加する場合、**Add My Current IP Address**をクリックします。

    - IPアクセスリストにIPアドレスを追加する場合、IPアドレスと説明を入力し、**Add to IP List**をクリックします。

        > **注記:**
        >
        > 各TiDB Dedicatedクラスタには、最大7つのIPアドレスをIPアクセスリストに追加できます。さらにIPアドレスを追加するための割り当てを申請するには、[TiDB Cloudサポート](/tidb-cloud/tidb-cloud-support.md)に連絡してください。

    - クラスタへの任意のIPアドレスのアクセスを許可するには（推奨されません）、**Allow Access From Anywhere**をクリックします。セキュリティ上のベストプラクティスに従うと、クラスタへの任意のIPアドレスのアクセスを許可することは推奨されません。これにより、クラスタが完全にインターネットに公開され、非常に危険にさらされる可能性があります。

    - アクセスリストからIPアドレスを削除するには、IPアドレスの行で**Remove**をクリックします。

4. 構成を保存するために**Apply**をクリックします。