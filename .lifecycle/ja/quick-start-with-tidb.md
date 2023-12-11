---
title: TiDBデータベースプラットフォームのクイックスタートガイド
summary: TiDBプラットフォームの素早い始め方を学び、TiDBが適切な選択肢であるか確認します。
aliases: ['/docs/dev/quick-start-with-tidb/', '/docs/dev/test-deployment-using-docker/']
---

# TiDBデータベースプラットフォームのクイックスタートガイド

このガイドはTiDBの素早い始め方を提供します。本番環境以外では、次の方法のいずれかを使用してTiDBデータベースを展開できます。

- [ローカルテストクラスタを展開](#deploy-a-local-test-cluster)（macOSおよびLinux向け）
- [単一マシンで本番展開をシミュレート](#simulate-production-deployment-on-a-single-machine)（Linux向け）

さらに、[TiDB Playground](https://play.tidbcloud.com/?utm_source=docs&utm_medium=tidb_quick_start)でTiDBの機能を試すことができます。

> **注意:**
>
> このガイドで提供されている展開方法は**クイックスタート用**であり、**本番用ではありません**。
>
> - 自己ホスト型の本番クラスタを展開するには、[本番インストールガイド](/production-deployment-using-tiup.md)を参照してください。
> - KubernetesでTiDBを展開するには、[KubernetesでTiDBを始める](https://docs.pingcap.com/tidb-in-kubernetes/stable/get-started)を参照してください。
> - クラウドでTiDBを管理するには、[TiDB Cloudクイックスタート](https://docs.pingcap.com/tidbcloud/tidb-cloud-quickstart)を参照してください。

## ローカルテストクラスタを展開

- シナリオ：macOSまたはLinuxの単一サーバーを使用して簡単にTiDBクラスターをローカルに展開します。このようなクラスターを展開することで、TiDBの基本的なアーキテクチャやTiDB、TiKV、PD、およびモニタリングコンポーネントの操作などを学ぶことができます。

<SimpleTab>
<div label="macOS">

分散システムとして、基本的なTiDBテストクラスターは通常、2つのTiDBインスタンス、3つのTiKVインスタンス、3つのPDインスタンス、およびオプションのTiFlashインスタンスで構成されています。TiUP Playgroundを使用して、次の手順に従ってテストクラスターを素早く構築できます。

1. TiUPをダウンロードしてインストールします：

    {{< copyable "shell-regular" >}}

    ```shell
    curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
    ```

    以下のメッセージが表示された場合、TiUPが正常にインストールされています：

    ```log
    Successfully set mirror to https://tiup-mirrors.pingcap.com
    Detected shell: zsh
    Shell profile:  /Users/user/.zshrc
    /Users/user/.zshrc has been modified to add tiup to PATH
    open a new terminal or source /Users/user/.zshrc to use it
    Installed path: /Users/user/.tiup/bin/tiup
    ===============================================
    Have a try:     tiup playground
    ===============================================
    ```

    上記の出力にあるShellプロファイルパスに注意してください。次のステップでこのパスを使用する必要があります。

2. グローバル環境変数を宣言します：

    > **注意:**
    >
    > インストール後、TiUPは対応するShellプロファイルの絶対パスを表示します。パスに基づいて次の`source`コマンドで`${your_shell_profile}`を修正する必要があります。

    {{< copyable "shell-regular" >}}

    ```shell
    source ${your_shell_profile}
    ```

3. 現在のセッションでクラスターを開始します：

    - 最新バージョンのTiDBクラスターを1つのTiDBインスタンス、1つのTiKVインスタンス、1つのPDインスタンス、および1つのTiFlashインスタンスで開始するには、次のコマンドを実行します：

        {{< copyable "shell-regular" >}}

        ```shell
        tiup playground
        ```

    - TiDBのバージョンと各コンポーネントのインスタンス数を指定するには、次のようなコマンドを実行します：

        {{< copyable "shell-regular" >}}

        ```shell
        tiup playground v7.4.0 --db 2 --pd 3 --kv 3
        ```

        このコマンドは、v7.4.0などのバージョンのクラスターをローカルマシンにダウンロードして開始します。最新バージョンを表示するには、`tiup list tidb`を実行します。

        このコマンドにより、クラスターへのアクセス方法が返されます：

        ```log
        CLUSTER START SUCCESSFULLY, Enjoy it ^-^
        To connect TiDB: mysql --comments --host 127.0.0.1 --port 4001 -u root -p（パスワードなし）
        To connect TiDB: mysql --comments --host 127.0.0.1 --port 4000 -u root -p（パスワードなし）
        To view the dashboard: http://127.0.0.1:2379/dashboard
        PD client endpoints: [127.0.0.1:2379 127.0.0.1:2382 127.0.0.1:2384]
        To view Prometheus: http://127.0.0.1:9090
        To view Grafana: http://127.0.0.1:3000
        ```

        > **注意:**
        >
        > + v5.2.0以降、TiDBはApple M1チップを使用しているマシンで`tiup playground`を実行できます。
        > + この方法で操作するPlaygroundでは、テスト展開が完了するとTiUPが元のクラスターデータをクリーンアップします。コマンドを再実行すると新しいクラスターが取得できます。
        > + データをストレージに永続化する場合は、`tiup --tag <your-tag> playground ...`を実行します。詳細については、[TiUPリファレンス](/tiup/tiup-reference.md#-t---tag)ガイドを参照してください。

4. 新しいセッションを開いてTiDBにアクセスします：

    + TiUPクライアントを使用してTiDBに接続します。

        {{< copyable "shell-regular" >}}

        ```shell
        tiup client
        ```

    + または、MySQLクライアントを使用してTiDBに接続できます。

        {{< copyable "shell-regular" >}}

        ```shell
        mysql --host 127.0.0.1 --port 4000 -u root
        ```

5. TiDBのPrometheusダッシュボードには、<http://127.0.0.1:9090>からアクセスできます。

6. [TiDBダッシュボード](/dashboard/dashboard-intro.md)には、<http://127.0.0.1:2379/dashboard>からアクセスできます。デフォルトのユーザー名は`root`で、パスワードは空です。

7. TiDBのGrafanaダッシュボードには、<http://127.0.0.1:3000>からアクセスできます。デフォルトのユーザー名とパスワードは`admin`です。

8. （オプション）[TiFlashにデータをロード](/tiflash/tiflash-overview.md#use-tiflash)して分析します。

9. テスト展開後にクラスターをクリーンアップします：

    1. 上記のTiDBサービスを停止するには<kbd>Control+C</kbd>を押します。

    2. サービスが停止した後、次のコマンドを実行します：

        {{< copyable "shell-regular" >}}

        ```shell
        tiup clean --all
        ```

> **注意:**
>
> TiUP Playgroundはデフォルトで`127.0.0.1`をリッスンし、サービスはローカルからのみアクセスできます。サービスを外部からアクセス可能にする場合は、`--host`パラメータを使用してリッスンアドレスを指定してネットワークインタフェースカード（NIC）をバインドしてください。

</div>
<div label="Linux">

分散システムとして、基本的なTiDBテストクラスターは通常、2つのTiDBインスタンス、3つのTiKVインスタンス、3つのPDインスタンス、およびオプションのTiFlashインスタンスで構成されています。TiUP Playgroundを使用して、次の手順に従ってテストクラスターを素早く構築できます。

1. TiUPをダウンロードしてインストールします：

    {{< copyable "shell-regular" >}}

    ```shell
    curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
    ```

    以下のメッセージが表示された場合、TiUPが正常にインストールされています：

    ```log
    Successfully set mirror to https://tiup-mirrors.pingcap.com
    Detected shell: zsh
    Shell profile:  /Users/user/.zshrc
    /Users/user/.zshrc has been modified to add tiup to PATH
    open a new terminal or source /Users/user/.zshrc to use it
    Installed path: /Users/user/.tiup/bin/tiup
    ===============================================
    Have a try:     tiup playground
    ===============================================
    ```

    上記の出力にあるShellプロファイルパスに注意してください。次のステップでこのパスを使用する必要があります。

2. グローバル環境変数を宣言します：

    > **注意:**
    >
    > インストール後、TiUPは対応するShellプロファイルの絶対パスを表示します。パスに基づいて次の`source`コマンドで`${your_shell_profile}`を修正する必要があります。

    {{< copyable "shell-regular" >}}

    ```shell
    source ${your_shell_profile}
    ```

3. 現在のセッションでクラスタを開始します：

    - 最新バージョンのTiDBクラスタを1つのTiDBインスタンス、1つのTiKVインスタンス、1つのPDインスタンス、および1つのTiFlashインスタンスで起動するには、次のコマンドを実行します：

        {{< copyable "shell-regular" >}}

        ```shell
        tiup playground
        ```

    - TiDBのバージョンと各コンポーネントのインスタンス数を指定するには、次のようなコマンドを実行します：

        {{< copyable "shell-regular" >}}

        ```shell
        tiup playground v7.4.0 --db 2 --pd 3 --kv 3
        ```

        このコマンドは、v7.4.0などのバージョンのクラスタをローカルマシンにダウンロードし、それを起動します。最新バージョンを表示するには、`tiup list tidb`を実行します。

        このコマンドは、クラスタのアクセス方法を返します：

        ```log
        クラスタは正常に開始されました。 -^-
        TiDBに接続するには: mysql --host 127.0.0.1 --port 4000 -u root -p（パスワードなし）--comments
        ダッシュボードを表示するには: http://127.0.0.1:2379/dashboard
        PDクライアントエンドポイント：[127.0.0.1:2379]
        Prometheusを表示するには: http://127.0.0.1:9090
        Grafanaを表示するには: http://127.0.0.1:3000
        ```

        > **注意:**
        >
        > この方法で操作されるプレイグラウンドでは、テスト展開が完了すると、TiUPが元のクラスタデータをクリーンアップします。このコマンドを再実行すると新しいクラスタが取得できます。
        > データをストレージに永続化したい場合は、`tiup --tag <your-tag> playground ...`を実行します。詳細については、[TiUPリファレンス](/tiup/tiup-reference.md#-t---tag)ガイドを参照してください。

4. TiDBにアクセスするために新しいセッションを開始します：

    + TiUPクライアントを使用してTiDBに接続します。

        {{< copyable "shell-regular" >}}

        ```shell
        tiup client
        ```

    + または、MySQLクライアントを使用してTiDBに接続することもできます。

        {{< copyable "shell-regular" >}}

        ```shell
        mysql --host 127.0.0.1 --port 4000 -u root
        ```

5. TiDBのPrometheusダッシュボードには<http://127.0.0.1:9090>からアクセスします。

6. [TiDBダッシュボード](/dashboard/dashboard-intro.md)には<http://127.0.0.1:2379/dashboard>からアクセスします。デフォルトのユーザー名は `root` で、パスワードは空です。

7. TiDBのGrafanaダッシュボードには<http://127.0.0.1:3000>からアクセスします。デフォルトのユーザー名とパスワードは `admin` です。

8. (任意) [TiFlashにデータをロード](/tiflash/tiflash-overview.md#use-tiflash)して解析を行います。

9. テスト展開後にクラスタをクリーンアップします：

    1. <kbd>Control+C</kbd>を押してプロセスを停止します。

    2. サービスが停止した後、次のコマンドを実行します：

        {{< copyable "shell-regular" >}}

        ```shell
        tiup clean --all
        ```

> **注意:**
>
> プレイグラウンドではデフォルトで`127.0.0.1`でリッスンしており、サービスはローカルからのみアクセス可能です。サービスを外部からアクセス可能にする場合は、`--host`パラメータを使用してリッスンアドレスを指定して、ネットワークインターフェースカード（NIC）を外部からアクセス可能なIPアドレスにバインドします。
        port: 20162
       status_port: 20182
       config:
         server.labels: { host: "logic-host-3" }

    tiflash_servers:
     - host: 10.0.1.1

    monitoring_servers:
     - host: 10.0.1.1

    grafana_servers:
     - host: 10.0.1.1
    ```

    - `user: "tidb"`: `tidb`システムユーザー（デプロイ時に自動的に作成される）を使用して、クラスターの内部管理を行います。デフォルトではSSH経由でターゲットマシンにログインする際にポート22を使用します。
    - `replication.enable-placement-rules`: このPDパラメータは、TiFlashが正常に実行されることを保証するために設定されています。
    - `host`: ターゲットマシンのIPアドレス。

7. クラスターのデプロイコマンドを実行する：

    {{< copyable "shell-regular" >}}

    ```shell
    tiup cluster deploy <cluster-name> <version> ./topo.yaml --user root -p
    ```

    - `<cluster-name>`: クラスター名を設定します
    - `<version>`: TiDBクラスターのバージョンを設定します。例：`v7.4.0`。すべてのサポートされているTiDBバージョンは、`tiup list tidb`コマンドを実行することで確認できます
    - `-p`: ターゲットマシンに接続するために使用するパスワードを指定します。

        > **注意:**
        >
        > シークレットキーを使用する場合は、`-i`を通じてキーのパスを指定できます。`-i`と`-p`を同時に使用しないでください。

    "y"を入力し、`root`ユーザーのパスワードを入力してデプロイを完了します：

    ```log
    続行しますか？ [y/N]:  y
    SSHパスワードを入力：
    ```

8. クラスターを開始する：

    {{< copyable "shell-regular" >}}

    ```shell
    tiup cluster start <cluster-name>
    ```

9. クラスターにアクセスする：

    - MySQLクライアントをインストールします。すでにインストールされている場合は、このステップをスキップしてください。

        {{< copyable "shell-regular" >}}

        ```shell
        yum -y install mysql
        ```

    - TiDBにアクセスします。パスワードは空です：

        {{< copyable "shell-regular" >}}

        ```shell
        mysql -h 10.0.1.1 -P 4000 -u root
        ```

    - Grafana監視ダッシュボードにアクセス：<http://{grafana-ip}:3000>。デフォルトのユーザー名とパスワードはともに`admin`です。

    - [TiDBダッシュボード](/dashboard/dashboard-intro.md)にアクセス：<http://{pd-ip}:2379/dashboard>。デフォルトのユーザー名は`root`で、パスワードは空です。

    - 現在デプロイされているクラスターリストを表示するには：

        {{< copyable "shell-regular" >}}

        ```shell
        tiup cluster list
        ```

    - クラスタートポロジと状態を表示するには：

         {{< copyable "shell-regular" >}}

        ```shell
        tiup cluster display <cluster-name>
        ```

## 次は何をすればよいですか

ローカルテスト環境にTiDBクラスターをデプロイしたばかりの場合、次のステップがあります：

- [TiDBでの基本的なSQL操作](/basic-sql-operations.md)を参照して基本的なSQL操作について学びます。
- [TiDBへのデータ移行](/migration-overview.md)を参照してデータをTiDBに移行できます。

本番環境のためにTiDBクラスターをデプロイする準備が整った場合、次のステップがあります：

- [TiUPを使用したTiDBのデプロイ](/production-deployment-using-tiup.md)
- あるいは、[TiDB Operatorを使用したクラウド上のTiDBのデプロイ](https://docs.pingcap.com/tidb-in-kubernetes/stable)を参照して、TiDBをKubernetes上に展開できます。

TiFlashを使用した分析ソリューションをお探しの場合、次のステップがあります：

- [TiFlashの使用方法](/tiflash/tiflash-overview.md#use-tiflash)
- [TiFlash概要](/tiflash/tiflash-overview.md)