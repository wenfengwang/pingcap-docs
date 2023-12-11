---
title: ProxySQL 統合ガイド
summary: TiDB Cloud と TiDB (セルフホスト) を ProxySQL と統合する方法を学びます。

# ProxySQL と TiDB を統合する

このドキュメントでは、ProxySQL の高レベルな紹介を提供し、ProxySQL を [開発環境](#development-environment) および [本番環境](#production-environment) で TiDB と統合する方法を説明し、[クエリルーティングの典型的なシナリオ](#typical-scenario) を通じて統合の主な利点を示します。

TiDB および ProxySQL について詳しく知りたい場合は、以下の有用なリンクを参照できます:

- [TiDB Cloud](https://docs.pingcap.com/tidbcloud)
- [TiDB 開発ガイド](/develop/dev-guide-overview.md)
- [ProxySQL ドキュメンテーション](https://proxysql.com/documentation/)

## ProxySQL とは?

[ProxySQL](https://proxysql.com/) は高性能なオープンソース SQL プロキシです。柔軟なアーキテクチャを持ち、さまざまなユースケースに理想的ないくつかの異なる方法で展開できます。例えば、ProxySQL は頻繁にアクセスされるデータをキャッシュすることでパフォーマンスを向上させるために使用できます。

ProxySQL は高速で効率的かつ使いやすいように設計されています。MySQL と完全に互換性があり、高品質な SQL プロキシから期待されるすべての機能をサポートしています。さらに、ProxySQL には幅広いアプリケーションに最適なユニークな機能が数多く用意されています。

## ProxySQL 統合の理由

- ProxySQL は多くのデータをロードするクエリを実行するアプリケーションであっても、Lambda のようなサーバーレス関数を使用してスケーラブルなアプリケーションを構築している場合など、非決定的でスパイクするワークロードの場合でも、[接続プール](https://proxysql.com/documentation/detailed-answers-on-faq/) や [頻繁に使用されるクエリのキャッシュ](https://proxysql.com/documentation/query-cache/) など ProxySQL の強力な機能を利用することで、アプリケーションは即座に利点を得ることができます。
- ProxySQL は [クエリルール](#query-rules) という簡単に設定できる機能を使用して、SQL インジェクションなどの SQL 脆弱性に対する追加のアプリケーションセキュリティ保護層として機能します。
- [ProxySQL](https://github.com/sysown/proxysql) と [TiDB](https://github.com/pingcap/tidb) がどちらもオープンソースプロジェクトであるため、ベンダー固有のロックインがないという利点を得ることができます。

## 配置アーキテクチャ

TiDB と ProxySQL を展開する最も明白な方法は、ProxySQL をアプリケーションレイヤーと TiDB の間に独立した中間層として追加することです。ただし、拡張性と障害耐性は保証されず、ネットワークホップによる追加の遅延も発生します。これらの問題を避けるために、代替の展開アーキテクチャは以下のように ProxySQL をサイドカーとして展開することです:

![proxysql-client-side-tidb-cloud](/media/develop/proxysql-client-side-tidb-cloud.png)

> **注意:**
>
> 上記のイラストは参考用です。実際の展開アーキテクチャに応じて適応する必要があります。

## 開発環境

このセクションでは、開発環境で TiDB を ProxySQL と統合する方法について説明します。ProxySQL 統合を開始するために、必要な [前提条件](#prerequisite) を整えた後、TiDB クラスターのタイプに応じて次のいずれかを選択できます。

- オプション1：[TiDB Cloud を ProxySQL と統合](#option-1-integrate-tidb-cloud-with-proxysql)
- オプション2：[TiDB (セルフホスト) を ProxySQL と統合](#option-2-integrate-tidb-self-hosted-with-proxysql)

### 前提条件

選択したオプションによって、次のパッケージが必要になる場合があります:

- [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
- [Docker](https://docs.docker.com/get-docker/)
- [Python 3](https://www.python.org/downloads/)
- [Docker Compose](https://docs.docker.com/compose/install/linux/)
- [MySQL クライアント](https://dev.mysql.com/doc/refman/8.0/en/mysql.html)

以下の手順に従ってインストール手順を実行できます:

<SimpleTab groupId="os">

<div label="macOS" value="macOS">

1. [Docker をダウンロード](https://docs.docker.com/get-docker/)して開始します (Docker Desktop には Docker Compose が含まれています)。
2. Python と `mysql-client` をインストールするには、次のコマンドを実行します:

    ```bash
    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
    brew install python mysql-client
    ```

</div>

<div label="CentOS" value="CentOS">

```bash
curl -fsSL https://get.docker.com | bash -s docker
yum install -y git python39 docker-ce docker-ce-cli containerd.io docker-compose-plugin mysql
systemctl start docker
```

</div>

<div label="Windows" value="Windows">

- Git をダウンロードしてインストールします。

    1. [Git Windows ダウンロード](https://git-scm.com/download/win)ページから **64-bit Git for Windows Setup** パッケージをダウンロードします。
    2. セットアップウィザードに従って Git パッケージをインストールします。デフォルトのインストール設定を使用するには、数回 **次へ** をクリックできます。

        ![proxysql-windows-git-install](/media/develop/proxysql-windows-git-install.png)

- MySQL Shell をダウンロードしてインストールします。

    1. [MySQL Community Server ダウンロード](https://dev.mysql.com/downloads/mysql/)ページから MySQL インストーラの ZIP ファイルをダウンロードします。
    2. ファイルを展開し、`bin` フォルダ内の `mysql.exe` を見つけます。`bin` フォルダのパスをシステム変数に追加し、Git Bash で `PATH` 変数に設定する必要があります:

        ```bash
        echo 'export PATH="(あなたの bin フォルダ)":$PATH' >>~/.bash_profile
        source ~/.bash_profile
        ```

        例:

        ```bash
        echo 'export PATH="/c/Program Files (x86)/mysql-8.0.31-winx64/bin":$PATH' >>~/.bash_profile
        source ~/.bash_profile
        ```

- Docker をダウンロードしてインストールします。

    1. [Docker ダウンロード](https://www.docker.com/products/docker-desktop/)ページから Docker Desktop インストーラをダウンロードします。
    2. インストーラをダブルクリックして実行します。インストールが完了すると、再起動を促されます。

        ![proxysql-windows-docker-install](/media/develop/proxysql-windows-docker-install.png)

- [Python Download](https://www.python.org/downloads/)ページから最新の Python 3 インストーラをダウンロードして実行します。

</div>

</SimpleTab>

### オプション1：TiDB Cloud を ProxySQL と統合

この統合では、[ProxySQL Docker イメージ](https://hub.docker.com/r/proxysql/proxysql) と TiDB サーバーレスクラスターを使用します。以下の手順では、ポート `16033` で ProxySQL を設定するため、このポートが利用可能であることを確認してください。

#### ステップ1. TiDB サーバーレスクラスターを作成する

1. [無料の TiDB サーバーレスクラスターを作成](https://docs.pingcap.com/tidbcloud/tidb-cloud-quickstart#step-1-create-a-tidb-cluster) してください。クラスターで設定したルートパスワードを覚えておいてください。
2. 後で使用するために、クラスターのホスト名、ポート、およびユーザー名を取得します。

    1. [クラスタ](https://tidbcloud.com/console/clusters) ページで、クラスタの名前をクリックしてクラスタの概要ページに移動します。
    2. クラスタの概要ページで **接続** ペインを見つけ、後で使用するために、`Endpoint`、`Port`、および `User` フィールドをコピーします。`Endpoint` はクラスタのホスト名です。

#### ステップ2. ProxySQL 構成ファイルを生成する

1. TiDB および ProxySQL の [統合例コードリポジトリ](https://github.com/pingcap-inc/tidb-proxysql-integration) をクローンします:

    <SimpleTab groupId="os">

    <div label="macOS" value="macOS">

    ```bash
    git clone https://github.com/pingcap-inc/tidb-proxysql-integration.git
    ```

    </div>

    <div label="CentOS" value="CentOS">

    ```bash
    git clone https://github.com/pingcap-inc/tidb-proxysql-integration.git
    ```

    </div>

    <div label="Windows (Git Bash)" value="Windows">

    ```bash
    git clone https://github.com/pingcap-inc/tidb-proxysql-integration.git
    ```

    </div>

    </SimpleTab>

2. `tidb-cloud-connect` フォルダに移動します:

    <SimpleTab groupId="os">

    <div label="macOS" value="macOS">

    ```bash
    cd tidb-proxysql-integration/example/tidb-cloud-connect
    ```

    </div>

    <div label="CentOS" value="CentOS">

    ```bash
    cd tidb-proxysql-integration/example/tidb-cloud-connect
    ```

    </div>

    <div label="Windows (Git Bash)" value="Windows">

    ```bash
    cd tidb-proxysql-integration/example/tidb-cloud-connect
    ```

    </div>

    </SimpleTab>

3. プロキシSQL構成ファイルを生成するには、`proxysql-config.py` を実行します：

    <SimpleTab groupId="os">
    
    <div label="macOS" value="macOS">

    ```bash
    python3 proxysql-config.py
    ```

    </div>

    <div label="CentOS" value="CentOS">

    ```bash
    python3 proxysql-config.py
    ```

    </div>

    <div label="Windows (Git Bash)" value="Windows">

    ```bash
    python proxysql-config.py
    ```

    </div>

    </SimpleTab>

    プロンプトが表示されたら、「Serverless Tier Host」のクラスタのエンドポイントを入力し、次にクラスタのユーザー名とパスワードを入力します。

    以下はサンプルの出力です。`tidb-cloud-connect` フォルダの下に3つの構成ファイルが生成されたことがわかります。

    ```
    [Begin] generating configuration files..
    tidb-cloud-connect.cnf generated successfully.
    proxysql-prepare.sql generated successfully.
    proxysql-connect.py generated successfully.
    [End] all files generated successfully and placed in the current folder.
    ```

#### ステップ3. ProxySQLを構成する

1. Docker を起動します。すでに Docker が起動している場合は、このステップをスキップしてください：

    <SimpleTab groupId="os">

    <div label="macOS" value="macOS">

    インストール済みの Docker のアイコンをダブルクリックして起動します。

    </div>

    <div label="CentOS" value="CentOS">

    ```bash
    systemctl start docker
    ```

    </div>

    <div label="Windows" value="Windows">

    インストール済みの Docker のアイコンをダブルクリックして起動します。

    </div>

    </SimpleTab>

2. ProxySQL イメージを取得し、ProxySQL コンテナをバックグラウンドで起動します：

    <SimpleTab groupId="os">

    <div label="macOS" value="macOS">

    ```bash
    docker compose up -d
    ```

    </div>

    <div label="CentOS" value="CentOS">

    ```bash
    docker compose up -d
    ```

    </div>

    <div label="Windows (Git Bash)" value="Windows">

    ```bash
    docker compose up -d
    ```

    </div>

    </SimpleTab>

3. 以下のコマンドを実行して、ProxySQL に統合します。これにより、「ProxySQL 管理インターフェース」内で `proxysql-prepare.sql` が実行されます：

    <SimpleTab groupId="os">

    <div label="macOS" value="macOS">

    ```bash
    docker compose exec proxysql sh -c "mysql -uadmin -padmin -h127.0.0.1 -P6032 < ./proxysql-prepare.sql"
    ```

    </div>

    <div label="CentOS" value="CentOS">

    ```bash
    docker compose exec proxysql sh -c "mysql -uadmin -padmin -h127.0.0.1 -P6032 < ./proxysql-prepare.sql"
    ```

    </div>

    <div label="Windows (Git Bash)" value="Windows">

    ```bash
    docker compose exec proxysql sh -c "mysql -uadmin -padmin -h127.0.0.1 -P6032 < ./proxysql-prepare.sql"
    ```

    </div>

    </SimpleTab>

    > **注意:**
    >
    > `proxysql-prepare.sql` スクリプトは、以下のことを行います：
    >
    > 1. クラスタのユーザー名とパスワードを使用してユーザーを追加します。
    > 2. ユーザーを監視アカウントに割り当てます。
    > 3. TiDB Serverless クラスタをホストのリストに追加します。
    > 4. ProxySQL と TiDB Serverless クラスタ間の安全な接続を有効にします。
    >
    > より良い理解のためには、`proxysql-prepare.sql` ファイルを確認することを強くお勧めします。ProxySQL の構成について詳しくは、[ProxySQL documentation](https://proxysql.com/documentation/proxysql-configuration/) を参照してください。

    以下はサンプルの出力です。クラスタのホスト名が出力されるため、ProxySQL と TiDB Serverless クラスタ間の接続が確立されていることがわかります。

    ```
    *************************** 1. row ***************************
        hostgroup_id: 0
            hostname: gateway01.us-west-2.prod.aws.tidbcloud.com
                port: 4000
            gtid_port: 0
                status: ONLINE
                weight: 1
            compression: 0
        max_connections: 1000
    max_replication_lag: 0
                use_ssl: 1
        max_latency_ms: 0
                comment:
    ```

#### ステップ4. ProxySQL を介して TiDB クラスタに接続する

1. TiDB クラスタに接続するには、`proxysql-connect.py` を実行します。このスクリプトは自動的に MySQL クライアントを起動し、[ステップ2](#ステップ2-ProxySQL構成ファイルの生成) で指定したユーザー名とパスワードを使用します。

    <SimpleTab groupId="os">

    <div label="macOS" value="macOS">

    ```bash
    python3 proxysql-connect.py
    ```

    </div>

    <div label="CentOS" value="CentOS">

    ```bash
    python3 proxysql-connect.py
    ```

    </div>

    <div label="Windows (Git Bash)" value="Windows">

    ```bash
    python proxysql-connect.py
    ```

    </div>

    </SimpleTab>

2. TiDB クラスタに接続した後、接続を検証するために次の SQL ステートメントを使用できます：

    ```sql
    SELECT VERSION();
    ```

    TiDB のバージョンが表示されれば、ProxySQL を介して TiDB Serverless クラスタに正常に接続されています。いつでも MySQL クライアントから退出するには、`quit` と入力して <kbd>enter</kbd> キーを押します。

    > **注意:**
    >
    > ***デバッグ用:*** クラスタに接続できない場合は、`tidb-cloud-connect.cnf`、`proxysql-prepare.sql`、`proxysql-connect.py` ファイルをチェックしてください。提供したサーバー情報が利用可能で正しいことを確認してください。

3. コンテナを停止し削除し、前のディレクトリに移動するには、次のコマンドを実行します：

    <SimpleTab groupId="os">

    <div label="macOS" value="macOS">

    ```bash
    docker compose down
    cd -
    ```

    </div>

    <div label="CentOS" value="CentOS">

    ```bash
    docker compose down
    cd -
    ```

    </div>

    <div label="Windows (Git Bash)" value="Windows">

    ```bash
    docker compose down
    cd -
    ```

    </div>

    </SimpleTab>

### オプション2: TiDB（セルフホスト型）を ProxySQL と統合する

この統合では、[TiDB](https://hub.docker.com/r/pingcap/tidb) と [ProxySQL](https://hub.docker.com/r/proxysql/proxysql) の Docker イメージを使用して環境をセットアップします。自己の興味に合わせて [TiDB（セルフホスト型）の他のインストール方法](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb) を試してみることをお勧めします。

以下の手順では、ProxySQL をポート `6033`、TiDB をポート `4000` で設定するため、これらのポートが使用可能であることを確認してから進めてください。

1. Docker を起動します。すでに Docker が起動している場合は、このステップをスキップしてください：

    <SimpleTab groupId="os">

    <div label="macOS" value="macOS">

    インストール済みの Docker のアイコンをダブルクリックして起動します。

    </div>

    <div label="CentOS" value="CentOS">

    ```bash
    systemctl start docker
    ```

    </div>

    <div label="Windows" value="Windows">

    インストール済みの Docker のアイコンをダブルクリックして起動します。

    </div>

    </SimpleTab>

2. [integration example code repository](https://github.com/pingcap-inc/tidb-proxysql-integration) から TiDB と ProxySQL の Docker イメージをクローンします：

    <SimpleTab groupId="os">

    <div label="macOS" value="macOS">

    ```bash
    git clone https://github.com/pingcap-inc/tidb-proxysql-integration.git
    ```

    </div>

    <div label="CentOS" value="CentOS">

    ```bash
    git clone https://github.com/pingcap-inc/tidb-proxysql-integration.git
    ```

    </div>

    <div label="Windows (Git Bash)" value="Windows">

    ```bash
    git clone https://github.com/pingcap-inc/tidb-proxysql-integration.git
    ```

    </div>

    </SimpleTab>

3. ProxySQL および TiDB の最新のイメージを取得します：

    <SimpleTab groupId="os">

    <div label="macOS" value="macOS">

    ```bash
    cd tidb-proxysql-integration && docker compose pull
    ```

    </div>

    <div label="CentOS" value="CentOS">

    ```bash
    cd tidb-proxysql-integration && docker compose pull
    ```

    </div>

    <div label="Windows (Git Bash)" value="Windows">

    ```bash
    cd tidb-proxysql-integration && docker compose pull
    ```

    </div>

    </SimpleTab>

4. 両方のTiDBとProxySQLをコンテナとして実行する統合環境を開始します：

    <SimpleTab groupId="os">

    <div label="macOS" value="macOS">

    ```bash
    docker compose up -d
    ```

    </div>

    <div label="CentOS" value="CentOS">

    ```bash
    docker compose up -d
    ```

    </div>

    <div label="Windows (Git Bash)" value="Windows">

    ```bash
    docker compose up -d
    ```

    </div>

    </SimpleTab>

    ProxySQLの`6033`ポートにログインするには、パスワードを空にした`root`ユーザーを使用できます。

5. ProxySQL経由でTiDBに接続します：

    <SimpleTab groupId="os">

    <div label="macOS" value="macOS">

    ```bash
    mysql -u root -h 127.0.0.1 -P 6033
    ```

    </div>

    <div label="CentOS" value="CentOS">

    ```bash
    mysql -u root -h 127.0.0.1 -P 6033
    ```

    </div>

    <div label="Windows (Git Bash)" value="Windows">

    ```bash
    mysql -u root -h 127.0.0.1 -P 6033
    ```

    </div>

    </SimpleTab>

6. TiDBクラスターに接続した後、以下のSQL文を使用して接続を検証できます：

    ```sql
    SELECT VERSION();
    ```

    TiDBバージョンが表示された場合、TiDBコンテナにProxySQL経由で正常に接続されています。

7. コンテナを停止および削除し、前のディレクトリに移動するには、次のコマンドを実行します：

    <SimpleTab groupId="os">

    <div label="macOS" value="macOS">

    ```bash
    docker compose down
    cd -
    ```

    </div>

    <div label="CentOS" value="CentOS">

    ```bash
    docker compose down
    cd -
    ```

    </div>

    <div label="Windows (Git Bash)" value="Windows">

    ```bash
    docker compose down
    cd -
    ```

    </div>

    </SimpleTab>

## 本番環境

本番環境では、[TiDB Cloud](https://en.pingcap.com/tidb-cloud/)を直接使用して、完全管理されたエクスペリエンスをお勧めします。

### 必要条件

MySQLクライアントをダウンロードしてインストールしてください。例えば、[MySQL Shell](https://dev.mysql.com/downloads/shell/)をご利用いただけます。

### CentOS上のTiDB CloudとProxySQLの統合

ProxySQLは多くの異なるプラットフォームにインストールできます。以下はCentOSを例にしています。

サポートされているプラットフォームの完全なリストと対応するバージョン要件については、[ProxySQLドキュメント](https://proxysql.com/documentation/installing-proxysql/)を参照してください。

#### ステップ1. TiDB専用クラスターを作成する

詳細な手順については、 [TiDBクラスターの作成](https://docs.pingcap.com/tidbcloud/create-tidb-cluster)を参照してください。

#### ステップ2. ProxySQLのインストール

1. ProxySQLをYUMリポジトリに追加します：

    ```bash
    cat > /etc/yum.repos.d/proxysql.repo << EOF
    [proxysql]
    name=ProxySQL YUMリポジトリ
    baseurl=https://repo.proxysql.com/ProxySQL/proxysql-2.4.x/centos/\$releasever
    gpgcheck=1
    gpgkey=https://repo.proxysql.com/ProxySQL/proxysql-2.4.x/repo_pub_key
    EOF
    ```

2. ProxySQLをインストールします：

    ```bash
    yum install -y proxysql
    ```

3. ProxySQLを起動します：

    ```bash
    systemctl start proxysql
    ```

ProxySQLのサポートされているプラットフォームおよびそのインストールについての詳細については、 [ProxySQL README](https://github.com/sysown/proxysql#installation)または [ProxySQLインストールドキュメント](https://proxysql.com/documentation/installing-proxysql/)を参照してください。

#### ステップ3. ProxySQLを構成する

ProxySQLをTiDBのプロキシとして使用するには、ProxySQLを構成する必要があります。これを行うには、[ProxySQL管理インターフェース内でSQLステートメントを実行](#option-1-configure-proxysql-using-the-admin-interface)するか、[構成ファイル](#option-2-configure-proxysql-using-a-configuration-file)を使用することができます。

> **注:**
>
> 以下のセクションでは、ProxySQLの必要な構成項目のみがリストされています。
> 詳細な構成のリストについては、[ProxySQLドキュメント](https://proxysql.com/documentation/proxysql-configuration/)を参照してください。

##### オプション1: 管理インターフェースを使用してProxySQLを構成する

1. 通常のProxySQL管理インターフェースを使用して、ProxySQLの内部を再構成します。これにより、デフォルトでポート`6032`で利用可能なMySQLコマンドラインクライアントを介して、ProxySQL管理プロンプトに移動します。

    ```bash
    mysql -u admin -padmin -h 127.0.0.1 -P6032 --prompt 'ProxySQL Admin> '
    ```

    上記の手順でProxySQL管理プロンプトに移動します。

2. 使用するTiDBクラスターを構成します。ここでは、例として1つのTiDB専用クラスターを追加します。`<tidb cloud dedicated cluster host>`および`<tidb cloud dedicated cluster port>`をご自身のTiDB Cloudのエンドポイントとポート（デフォルトポートは`4000`）に置き換える必要があります。

    ```sql
    INSERT INTO mysql_servers(hostgroup_id, hostname, port) 
    VALUES 
      (
        0,
        '<tidb cloud dedicated cluster host>', 
        <tidb cloud dedicated cluster port>
      );
    LOAD mysql servers TO runtime;
    SAVE mysql servers TO DISK;
    ```

##### オプション2: 構成ファイルを使用してProxySQLを構成する

このオプションは、ProxySQLを構成する別の方法として検討するべきです。詳細については、[構成ファイルを使用したProxySQLの構成](https://github.com/sysown/proxysql#configuring-proxysql-through-the-config-file)を参照してください。

1. 存在するSQLiteデータベース（内部で構成が保存されている場所）を削除します：

    ```bash
    rm /var/lib/proxysql/proxysql.db
    ```

    > **警告:**
    >
    > SQLiteデータベースファイルを削除すると、ProxySQL管理インターフェースを使用して行った構成変更が失われます。

2. `/etc/proxysql.cnf`の構成ファイルを必要に応じて変更します。例：

    ```
    mysql_servers:
    (
        {
            address="<tidb cloud dedicated cluster host>"
            port=<tidb cloud dedicated cluster port>
            hostgroup=0
            max_connections=2000
        }
    )

    mysql_users:
    (
        {
```yaml
            username = "<tidb cloud dedicated cluster username>"
            password = "<tidb cloud dedicated cluster password>"
            default_hostgroup = 0
            max_connections = 1000
            default_schema = "test"
            active = 1
            transaction_persistent = 1
        }
    )
    ```

    上記の例では、

    - `address` および `port`: あなたのTiDB Cloudクラスターのエンドポイントおよびポートを指定します。
    - `username` および `password`: あなたのTiDB Cloudクラスターのユーザー名およびパスワードを指定します。

3. ProxySQLを再起動します:

    ```bash
    systemctl restart proxysql
    ```

    再起動後、SQLiteデータベースが自動的に作成されます。

> **警告:**
>
> 本番環境ではデフォルトの資格情報を使用せず、proxysqlサービスを起動する前に `/etc/proxysql.cnf` ファイルの `admin_credentials` 変数を変更してください。


## 典型的なシナリオ

このセクションでは、クエリのルーティングを例に挙げ、ProxySQLをTiDBと統合することで活用できる利点の一部を示します。

### クエリルール

データベースは高いトラフィック、不良なコード、または悪意のあるスパムにより過負荷になることがあります。ProxySQLのクエリルールを使用すると、クエリの再ルーティング、書き換え、または拒否により、これらの問題に迅速かつ効果的に対応できます。

![proxysql-client-side-rules](/media/develop/proxysql-client-side-rules.png)

> **注意:**
>
> 次のステップでは、TiDBおよびProxySQLのコンテナイメージを使用してクエリルールを構成します。これらをまだプルしていない場合は、詳細な手順については [統合セクション](#option-2-integrate-tidb-self-hosted-with-proxysql) をご確認ください。

1. TiDBおよびProxySQLの[統合例コードリポジトリ](https://github.com/pingcap-inc/tidb-proxysql-integration)をクローンします。すでにクローンしている場合は、このステップをスキップしてください。

    <SimpleTab groupId="os">

    <div label="macOS" value="macOS">

    ```bash
    git clone https://github.com/pingcap-inc/tidb-proxysql-integration.git
    ```

    </div>

    <div label="CentOS" value="CentOS">

    ```bash
    git clone https://github.com/pingcap-inc/tidb-proxysql-integration.git
    ```

    </div>

    <div label="Windows (Git Bash)" value="Windows">

    ```bash
    git clone https://github.com/pingcap-inc/tidb-proxysql-integration.git
    ```

    </div>

    </SimpleTab>

2. ProxySQLルールの例ディレクトリに移動します:

    <SimpleTab groupId="os">

    <div label="macOS" value="macOS">

    ```bash
    cd tidb-proxysql-integration/example/proxy-rule-admin-interface
    ```

    </div>

    <div label="CentOS" value="CentOS">

    ```bash
    cd tidb-proxysql-integration/example/proxy-rule-admin-interface
    ```

    </div>

    <div label="Windows (Git Bash)" value="Windows">

    ```bash
    cd tidb-proxysql-integration/example/proxy-rule-admin-interface
    ```

    </div>

    </SimpleTab>

3. 以下のコマンドを実行して、2つのTiDBコンテナと1つのProxySQLコンテナを起動します:

    <SimpleTab groupId="os">

    <div label="macOS" value="macOS">

    ```bash
    docker compose up -d
    ```

    </div>

    <div label="CentOS" value="CentOS">

    ```bash
    docker compose up -d
    ```

    </div>

    <div label="Windows (Git Bash)" value="Windows">

    ```bash
    docker compose up -d
    ```

    </div>

    </SimpleTab>

    すべてが正常に行われた場合、以下のコンテナが起動されます:

    - 2つのTiDBクラスターのDockerコンテナ（ポート `4001`、`4002` で公開）
    - 1つのProxySQL Dockerコンテナ（ポート `6034` で公開）

4. 2つのTiDBコンテナで、`mysql`を使用して似たようなスキーマ定義のテーブルを作成し、異なるデータ (`'tidb-server01-port-4001'`、`'tidb-server02-port-4002'`) を挿入して、これらのコンテナを識別します。

    <SimpleTab groupId="os">

    <div label="macOS" value="macOS">

    ```bash
    mysql -u root -h 127.0.0.1 -P 4001 << EOF
    DROP TABLE IF EXISTS test.tidb_server;
    CREATE TABLE test.tidb_server (server_name VARCHAR(255));
    INSERT INTO test.tidb_server (server_name) VALUES ('tidb-server01-port-4001');
    EOF

    mysql -u root -h 127.0.0.1 -P 4002 << EOF
    DROP TABLE IF EXISTS test.tidb_server;
    CREATE TABLE test.tidb_server (server_name VARCHAR(255));
    INSERT INTO test.tidb_server (server_name) VALUES ('tidb-server02-port-4002');
    EOF
    ```

    </div>

    <div label="CentOS" value="CentOS">

    ```bash
    mysql -u root -h 127.0.0.1 -P 4001 << EOF
    DROP TABLE IF EXISTS test.tidb_server;
    CREATE TABLE test.tidb_server (server_name VARCHAR(255));
    INSERT INTO test.tidb_server (server_name) VALUES ('tidb-server01-port-4001');
    EOF

    mysql -u root -h 127.0.0.1 -P 4002 << EOF
    DROP TABLE IF EXISTS test.tidb_server;
    CREATE TABLE test.tidb_server (server_name VARCHAR(255));
    INSERT INTO test.tidb_server (server_name) VALUES ('tidb-server02-port-4002');
    EOF
    ```

    </div>

    <div label="Windows (Git Bash)" value="Windows">

    ```bash
    mysql -u root -h 127.0.0.1 -P 4001 << EOF
    DROP TABLE IF EXISTS test.tidb_server;
    CREATE TABLE test.tidb_server (server_name VARCHAR(255));
    INSERT INTO test.tidb_server (server_name) VALUES ('tidb-server01-port-4001');
    EOF

    mysql -u root -h 127.0.0.1 -P 4002 << EOF
    DROP TABLE IF EXISTS test.tidb_server;
    CREATE TABLE test.tidb_server (server_name VARCHAR(255));
    INSERT INTO test.tidb_server (server_name) VALUES ('tidb-server02-port-4002');
    EOF
    ```

    </div>

    </SimpleTab>

5. 以下のコマンドを実行して、ProxySQLを構成します。これにより、ProxySQL管理インターフェイス内の `proxysql-prepare.sql` を実行して、TiDBコンテナとProxySQLの間にプロキシ接続を確立します。

    <SimpleTab groupId="os">

    <div label="macOS" value="macOS">

    ```bash
    docker compose exec proxysql sh -c "mysql -uadmin -padmin -h127.0.0.1 -P6032 < ./proxysql-prepare.sql"
    ```

    </div>

    <div label="CentOS" value="CentOS">

    ```bash
    docker compose exec proxysql sh -c "mysql -uadmin -padmin -h127.0.0.1 -P6032 < ./proxysql-prepare.sql"
    ```

    </div>

    <div label="Windows (Git Bash)" value="Windows">

    ```bash
    docker compose exec proxysql sh -c "mysql -uadmin -padmin -h127.0.0.1 -P6032 < ./proxysql-prepare.sql"
    ```

    </div>

    </SimpleTab>

    > **注意:**
    >
    > `proxysql-prepare.sql` は以下のようなことを行います:
    >
    > - `hostgroup_id` を `0` および `1` としてProxySQLにTiDBクラスターを追加します。
    > - ユーザー `root` を空のパスワードで追加し、`default_hostgroup` を `0` に設定します。
    > - `^SELECT.*FOR UPDATE$` をルール `1` および `destination_hostgroup` を `0` として追加します。このルールに一致するSQLステートメントは、`hostgroup` を `0` としてTiDBクラスターに転送されます。
    > - `^SELECT` をルール `2` および `destination_hostgroup` を `1` として追加します。このルールに一致するSQLステートメントは、`hostgroup` を `1` としてTiDBクラスターに転送されます。
    >
    > より理解を深めるためには、`proxysql-prepare.sql` ファイルを確認することを強くお勧めします。ProxySQLの構成について詳しくは、[ProxySQL documentation](https://proxysql.com/documentation/proxysql-configuration/) を参照してください。
    
    次の情報は、ProxySQLのパターンがクエリルールに一致する方法に関する追加情報です:

    - ProxySQLは、`rule_id` の昇順にルールを1つずつ一致させようとします。
- `^` symbol matches the beginning of a SQL statement and `$` matches the end.

ProxySQLの正規表現およびパターンマッチングについての詳細は、[mysql-query_processor_regex](https://proxysql.com/documentation/global-variables/mysql-variables/#mysql-query_processor_regex)を参照してください。ProxySQLドキュメントの[mysql_query_rules](https://proxysql.com/documentation/main-runtime/#mysql_query_rules)でも、パラメータの完全なリストを確認できます。

6. 構成を確認し、クエリルールが機能しているかどうかを確認します。

    1. `root`ユーザーとしてProxySQL MySQLインターフェースにログインします：

        <SimpleTab groupId="os">

        <div label="macOS" value="macOS">

        ```bash
        mysql -u root -h 127.0.0.1 -P 6034
        ```

        </div>

        <div label="CentOS" value="CentOS">

        ```bash
        mysql -u root -h 127.0.0.1 -P 6034
        ```

        </div>

        <div label="Windows (Git Bash)" value="Windows">

        ```bash
        mysql -u root -h 127.0.0.1 -P 6034
        ```

        </div>

        </SimpleTab>

    2. 次のSQLステートメントを実行します：

        - `SELECT`ステートメントを実行します：

            ```sql
            SELECT * FROM test.tidb_server;
            ```

            このステートメントは`rule_id 2`に一致し、`hostgroup 1`のTiDBクラスタにステートメントが転送されます。

        - `SELECT ... FOR UPDATE`ステートメントを実行します：

            ```sql
            SELECT * FROM test.tidb_server FOR UPDATE;
            ```

            このステートメントは`rule_id 1`に一致し、`hostgroup 0`のTiDBクラスタにステートメントが転送されます。

        - トランザクションを開始します：

            ```sql
            BEGIN;
            INSERT INTO test.tidb_server (server_name) VALUES ('insert this and rollback later');
            SELECT * FROM test.tidb_server;
            ROLLBACK;
            ```

            このトランザクションでは、`BEGIN`ステートメントはいかなるルールにも一致しません。これはデフォルトのホストグループ（この例では`hostgroup 0`）を使用します。ProxySQLはデフォルトでユーザーのtransaction_persistentを有効にしているため、同じトランザクション内のすべてのステートメントを同じホストグループで実行します。したがって、`INSERT`および`SELECT * FROM test.tidb_server;`ステートメントもTiDBクラスタの`hostgroup 0`に転送されます。

        以下は出力の例です。同様の出力を取得した場合、ProxySQLでクエリルールが正常に構成されていることを確認できます。

        ```sql
        +-------------------------+
        | server_name             |
        +-------------------------+
        | tidb-server02-port-4002 |
        +-------------------------+
        +-------------------------+
        | server_name             |
        +-------------------------+
        | tidb-server01-port-4001 |
        +-------------------------+
        +--------------------------------+
        | server_name                    |
        +--------------------------------+
        | tidb-server01-port-4001        |
        | insert this and rollback later |
        +--------------------------------+
        ```

    3. MySQLクライアントからいつでも終了するには、`quit`と入力して<kbd>enter</kbd>を押します。

7. コンテナを停止および削除し、前のディレクトリに移動するには、次のコマンドを実行します：

    <SimpleTab groupId="os">

    <div label="macOS" value="macOS">

    ```bash
    docker compose down
    cd -
    ```

    </div>

    <div label="CentOS" value="CentOS">

    ```bash
    docker compose down
    cd -
    ```

    </div>

    <div label="Windows (Git Bash)" value="Windows">

    ```bash
    docker compose down
    cd -
    ```

    </div>

    </SimpleTab>