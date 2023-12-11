---
title: Visual Studio Codeを使用してTiDBに接続する
summary: Visual Studio CodeまたはGitHub Codespacesを使用してTiDBに接続する方法を学びます。

# Visual Studio Codeを使用してTiDBに接続する

TiDBはMySQL互換のデータベースであり、[Visual Studio Code (VS Code)](https://code.visualstudio.com/)は軽量でありながらパワフルなソースコードエディターです。このチュートリアルでは、TiDBを[公式ドライバー](https://marketplace.visualstudio.com/items?itemName=mtxr.sqltools-driver-mysql)としてサポートする[SQLTools](https://marketplace.visualstudio.com/items?itemName=mtxr.sqltools)拡張機能を使用します。

このチュートリアルでは、Visual Studio Codeを使用してTiDBクラスターに接続する方法を学ぶことができます。

> **注意：**
>
> - このチュートリアルは、TiDB Serverless、TiDB Dedicated、およびTiDB Self-Hostedと互換性があります。
> - このチュートリアルは、[GitHub Codespaces](https://github.com/features/codespaces)、[Visual Studio Code Dev Containers](https://code.visualstudio.com/docs/devcontainers/containers)、および[Visual Studio Code WSL](https://code.visualstudio.com/docs/remote/wsl)などのVisual Studio Codeリモート開発環境とも互換性があります。

## 前提条件

このチュートリアルを完了するには、以下が必要です。

- [Visual Studio Code](https://code.visualstudio.com/#alt-downloads) **1.72.0**以降のバージョン
- Visual Studio Code用の[SQLTools MySQL/MariaDB/TiDB](https://marketplace.visualstudio.com/items?itemName=mtxr.sqltools-driver-mysql)拡張機能。インストールする方法は次のいずれかを使用できます：

    - このリンクをクリックして、VS Codeを起動して拡張機能を直接インストールします。
    - [VS Code Marketplace](https://marketplace.visualstudio.com/items?itemName=mtxr.sqltools-driver-mysql)に移動して、**インストール**をクリックします。

- TiDBクラスター

<CustomContent platform="tidb">

**TiDBクラスターをまだお持ちでない場合は、次のようにして作成できます：**

- （推奨）[TiDB Serverlessクラスターの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、TiDB Cloudクラスターを作成します。
- [ローカルテストTiDBクラスターのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[プロダクションTiDBクラスターのデプロイ](/production-deployment-using-tiup.md)に従って、ローカルクラスターを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスターをまだお持ちでない場合は、次のようにして作成できます：**

- （推奨）[TiDB Serverlessクラスターの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、TiDB Cloudクラスターを作成します。
- [ローカルテストTiDBクラスターのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[プロダクションTiDBクラスターのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスターを作成します。

</CustomContent>

## TiDBに接続する

選択したTiDBのデプロイオプションに応じてTiDBクラスターに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、対象クラスターの名前をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの構成が操作環境と一致していることを確認します。

    - **Endpoint Type**が`Public`に設定されていること。
    - **Connect With**が`General`に設定されていること。
    - **Operating System**が環境に一致していること。

    > **ヒント：**
    >
    > VS Codeがリモート開発環境で実行されている場合は、リストからリモートオペレーティングシステムを選択します。たとえば、Windows Subsystem for Linux（WSL）を使用している場合は、対応するLinuxディストリビューションに切り替えます。これはGitHub Codespacesを使用している場合は必要ありません。

4. ランダムなパスワードを作成するには、**Create password**をクリックします。

    > **ヒント：**
    >
    > 以前にパスワードを作成した場合は、元のパスワードを使用するか、新しいパスワードを生成するために**Reset password**をクリックできます。

5. VS Codeを起動し、ナビゲーションペインで**SQLTools**拡張機能を選択します。**CONNECTIONS**セクションの下で、**Add New Connection**をクリックし、データベースドライバーとして**TiDB**を選択します。

    ![VS Code SQLTools：新しい接続の追加](/media/develop/vsc-sqltools-add-new-connection.jpg)

6. 設定ペインで、次の接続パラメーターを構成します：

    - **Connection name**：この接続に意味のある名前を付けます。
    - **Connection group**：（オプション）この接続のグループに意味のある名前を付けます。同じグループ名の接続はグループ化されます。
    - **Connect using**：**Server and Port**を選択します。
    - **Server Address**：TiDB Cloud接続ダイアログから`host`パラメーターを入力します。
    - **Port**：TiDB Cloud接続ダイアログから`port`パラメーターを入力します。
    - **Database**：接続したいデータベースを入力します。
    - **Username**：TiDB Cloud接続ダイアログから`user`パラメーターを入力します。
    - **Password mode**：**SQLTools Driver Credentials**を選択します。
    - **MySQL driver specific options**エリアで、次のパラメーターを構成します：

        - **Authentication Protocol**：**default**を選択します。
        - **SSL**：**Enabled**を選択します。TiDB Serverlessでは安全な接続が必要です。**SSL Options (node.TLSSocket)**エリアで、TiDB Cloud接続ダイアログから`ssl_ca`パラメーターを**Certificate Authority (CA) Certificate File**フィールドに設定します。

            > **注意：**
            >
            > WindowsまたはGitHub Codespacesで実行している場合、**SSL**を空白のままにしておくことができます。デフォルトで、SQLToolsはMozillaによって整備された信頼できるCAを信頼します。詳細については、[TiDB Serverlessルート証明書管理](https://docs.pingcap.com/tidbcloud/secure-connections-to-serverless-clusters#root-certificate-management)を参照してください。

    ![VS Code SQLTools：TiDB Serverlessの接続設定の構成](/media/develop/vsc-sqltools-connection-config-serverless.jpg)

7. **TEST CONNECTION**をクリックして、TiDB Serverlessクラスターへの接続を検証します。

    1. ポップアップウィンドウで**Allow**をクリックします。
    2. **SQLTools Driver Credentials**ダイアログで、ステップ4で作成したパスワードを入力します。

        ![VS Code SQLTools：TiDB Serverlessに接続するためのパスワードの入力](/media/develop/vsc-sqltools-password.jpg)

8. 接続テストが成功すると、**Successfully connected!**メッセージが表示されます。接続構成を保存するには、**SAVE CONNECTION**をクリックします。

</div>
<div label="TiDB Dedicated">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、対象クラスターの名前をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere**をクリックします。

    接続文字列の取得方法の詳細については、[TiDB Dedicated standard connection](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4. VS Codeを起動し、ナビゲーションペインで**SQLTools**拡張機能を選択します。**CONNECTIONS**セクションの下で**Add New Connection**をクリックし、データベースドライバーとして**TiDB**を選択します。

    ![VS Code SQLTools：新しい接続の追加](/media/develop/vsc-sqltools-add-new-connection.jpg)

5. 設定ペインで、次の接続パラメーターを構成します：

    - **Connection name**：この接続に意味のある名前を付けます。
    - **Connection group**：（オプション）この接続のグループに意味のある名前を付けます。同じグループ名の接続はグループ化されます。
    - **Connect using**：**Server and Port**を選択します。
    - **Server Address**：TiDB Cloud接続ダイアログから`host`パラメーターを入力します。
    - **Port**：TiDB Cloud接続ダイアログから`port`パラメーターを入力します。
    - **Database**：接続したいデータベースを入力します。
    - **Username**：TiDB Cloud接続ダイアログから`user`パラメーターを入力します。
    - **Password mode**：**SQLTools Driver Credentials**を選択します。
    - **MySQL driver specific options**エリアで、次のパラメーターを構成します：

        - **Authentication Protocol**：**default**を選択します。
        - **SSL**：**Disabled**を選択します。

    ![VS Code SQLTools：TiDB Dedicatedの接続設定の構成](/media/develop/vsc-sqltools-connection-config-dedicated.jpg)

6. **TEST CONNECTION**をクリックして、TiDB Dedicatedクラスターへの接続を検証します。

    1. ポップアップウィンドウで**Allow**をクリックします。
    2. TiDB Dedicatedクラスターのパスワードを入力します。

    ![VS Code SQLTools：TiDB Dedicatedに接続するためのパスワードの入力](/media/develop/vsc-sqltools-password.jpg)

7. 接続テストが成功すると、**Successfully connected!**メッセージが表示されます。接続構成を保存するには、**SAVE CONNECTION**をクリックします。

</div>
```markdown
<div label="TiDB Self-Hosted">

1. Launch VS Code and select the **SQLTools** extension on the navigation pane. Under the **CONNECTIONS** section, click **Add New Connection** and select **TiDB** as the database driver.

    ![VS Code SQLTools: add new connection](/media/develop/vsc-sqltools-add-new-connection.jpg)

2. In the setting pane, configure the following connection parameters:

    - **Connection name**: give this connection a meaningful name.
    - **Connection group**: (optional) give this group of connections a meaningful name. Connections with the same group name will be grouped together.
    - **Connect using**: select **Server and Port**.
    - **Server Address**: enter the IP address or domain name of your TiDB Self-Hosted cluster.
    - **Port**: enter the port number of your TiDB Self-Hosted cluster.
    - **Database**: enter the database that you want to connect to.
    - **Username**: enter the username to use to connect to your TiDB Self-Hosted cluster.
    - **Password mode**:
        - If the password is empty, select **Use empty password**.
        - Otherwise, select **SQLTools Driver Credentials**.
    - In the **MySQL driver specific options** area, configure the following parameters:
        - **Authentication Protocol**: select **default**.
        - **SSL**: select **Disabled**.

    ![VS Code SQLTools: configure connection settings for TiDB Self-Hosted](/media/develop/vsc-sqltools-connection-config-self-hosted.jpg)

3. Click **TEST CONNECTION** to validate the connection to the TiDB Self-Hosted cluster.

    If the password is not empty, click **Allow** in the pop-up window, and then enter the password of the TiDB Self-Hosted cluster.

    ![VS Code SQLTools: enter password to connect to TiDB Self-Hosted](/media/develop/vsc-sqltools-password.jpg)

4. If the connection test is successful, you can see the **Successfully connected!** message. Click **SAVE CONNECTION** to save the connection configuration.
</div>
</SimpleTab>

## 次のステップ

- Visual Studio Codeの使用方法については、[Visual Studio Codeのドキュメント](https://code.visualstudio.com/docs)を参照してください。
- VS Code SQLTools拡張機能の使用方法については、[ドキュメント](https://marketplace.visualstudio.com/items?itemName=mtxr.sqltools)および[GitHubリポジトリ](https://github.com/mtxr/vscode-sqltools)を参照してください。
- [開発者ガイド](/develop/dev-guide-overview.md)の章を参照して、TiDBアプリケーション開発のベストプラクティスについて学んでください。具体的には、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルからのデータ取得](/develop/dev-guide-get-data-from-single-t
able.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)などです。
- [TiDB 開発者コース](https://www.pingcap.com/education/)を受講し、試験に合格して[TiDB認定](https://www.pingcap.com/education/certification/)を取得することで、専門のTiDB開発者になることができます。

## お困りですか？

<CustomContent platform="tidb">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](/support.md)してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケット](https://support.pingcap.com/)を作成してください。

</CustomContent>
```