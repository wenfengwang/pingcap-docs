---
title: TiDB Serverlessクラスターの構築
summary: TiDB Cloud上でTiDB Serverlessクラスターを構築し、それに接続する方法について学びます。

<!-- markdownlint-disable MD029 -->

# TiDB Serverlessクラスターの構築

<CustomContent platform="tidb">

このドキュメントでは、TiDBの使用を素早く始める手順について説明します。[TiDB Cloud](https://en.pingcap.com/tidb-cloud)を使用して、TiDB Serverlessクラスターを作成し、それに接続し、サンプルアプリケーションを実行します。

ローカルマシンでTiDBを実行する必要がある場合は、[ローカルでTiDBを起動する](/quick-start-with-tidb.md) をご覧ください。

</CustomContent>

<CustomContent platform="tidb-cloud">

このドキュメントでは、TiDB Cloudを使用した素早く始める手順について説明します。TiDBクラスターを作成し、それに接続し、サンプルアプリケーションを実行します。

</CustomContent>

## ステップ1. TiDB Serverlessクラスターの作成

1. TiDB Cloudアカウントをお持ちでない場合は、[こちら](https://tidbcloud.com/free-trial) をクリックしてアカウントを登録してください。

2. [こちら](https://tidbcloud.com/) をクリックして、TiDB Cloudアカウントにログインします。

3. [**クラスター**](https://tidbcloud.com/console/clusters) ページで、**クラスターの作成** をクリックします。

4. **クラスターの作成** ページでは、**Serverless** がデフォルトで選択されています。必要に応じてデフォルトのクラスター名を更新し、その後、クラスターを作成するリージョンを選択します。

5. TiDB Serverlessクラスターを作成するために **作成** をクリックします。

    TiDB Cloudクラスターは約30秒で作成されます。

6. TiDB Cloudクラスターが作成された後、クラスター名をクリックしてクラスター概要ページに移動し、その後、右上隅にある **接続** をクリックします。接続ダイアログボックスが表示されます。

7. ダイアログボックスで、希望の接続方法とオペレーティングシステムを選択して対応する接続文字列を取得します。このドキュメントでは、MySQLクライアントを例に取り上げます。

8. ランダムなパスワードを生成するために **パスワードの作成** をクリックします。生成されたパスワードは再表示されないため、パスワードを安全な場所に保存してください。rootパスワードを設定しない場合、クラスターに接続することはできません。

<CustomContent platform="tidb">

> **Note:**
>
> [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) クラスターの場合、クラスターへの接続時にはユーザーネームにクラスターのプレフィックスを含め、名前を引用符で囲む必要があります。詳細は[ユーザー名のプレフィックス](https://docs.pingcap.com/tidbcloud/select-cluster-tier#user-name-prefix) をご覧ください。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **Note:**
>
> [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) クラスターの場合、クラスターへの接続時にはユーザーネームにクラスターのプレフィックスを含め、名前を引用符で囲む必要があります。詳細は[/tidb-cloud/select-cluster-tier.md#user-name-prefix](/tidb-cloud/select-cluster-tier.md#user-name-prefix) をご覧ください。

</CustomContent>

## ステップ2. クラスターへの接続

1. MySQLクライアントがインストールされていない場合は、オペレーティングシステムを選択し、以下の手順に従ってインストールしてください。

<SimpleTab>

<div label="macOS">

macOSの場合、[Homebrew](https://brew.sh/index) をインストールしていない場合は、以下のコマンドを実行してMySQLクライアントをインストールします。

```shell
brew install mysql-client
```

出力は次のようになります:

```
mysql-client is keg-only, which means it was not symlinked into /opt/homebrew,
because it conflicts with mysql (which contains client libraries).

If you need to have mysql-client first in your PATH, run:
  echo 'export PATH="/opt/homebrew/opt/mysql-client/bin:$PATH"' >> ~/.zshrc

For compilers to find mysql-client you may need to set:
  export LDFLAGS="-L/opt/homebrew/opt/mysql-client/lib"
  export CPPFLAGS="-I/opt/homebrew/opt/mysql-client/include"
```

MySQLクライアントをPATHに追加するには、上記の出力に次のコマンドを見つけて実行します（出力が文書と一貫していない場合は、代わりに出力に対応するコマンドを使用してください）:

```shell
echo 'export PATH="/opt/homebrew/opt/mysql-client/bin:$PATH"' >> ~/.zshrc
```

次に、`source` コマンドでグローバル環境変数を宣言し、MySQLクライアントが正常にインストールされていることを確認します:

```shell
source ~/.zshrc
mysql --version
```

期待される出力の例:

```
mysql  Ver 8.0.28 for macos12.0 on arm64 (Homebrew)
```

</div>

<div label="Linux">

Linuxの場合、CentOS 7を例に取り上げます:

```shell
yum install mysql
```

次に、MySQLクライアントが正常にインストールされていることを確認します:

```shell
mysql --version
```

期待される出力の例:

```
mysql  Ver 15.1 Distrib 5.5.68-MariaDB, for Linux (x86_64) using readline 5.1
```

</div>

</SimpleTab>

2. [ステップ1](#step-1-create-a-tidb-serverless-cluster) で取得した接続文字列を実行します。

    {{< copyable "shell-regular" >}}

    ```shell
    mysql --connect-timeout 15 -u '<prefix>.root' -h <host> -P 4000 -D test --ssl-mode=VERIFY_IDENTITY --ssl-ca=/etc/ssl/cert.pem -p
    ```

<CustomContent platform="tidb">

> **Note:**
>
> - TiDB Serverlessクラスターに接続する際には、[TLS接続を使用する必要があります](https://docs.pingcap.com/tidbcloud/secure-connections-to-serverless-clusters)。
> - TiDB Serverlessクラスターに接続する際に問題が発生した場合は、[TiDB Serverlessクラスターへの安全な接続](https://docs.pingcap.com/tidbcloud/secure-connections-to-serverless-clusters) を詳細をご覧ください。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **Note:**
>
> - TiDB Serverlessクラスターに接続する際には、[TLS接続を使用する必要があります](/tidb-cloud/secure-connections-to-serverless-clusters.md)。
> - TiDB Serverlessクラスターに接続する際に問題が発生した場合は、[TiDB Serverlessクラスターへの安全な接続](/tidb-cloud/secure-connections-to-serverless-clusters.md) を詳細をご覧ください。

</CustomContent>

3. パスワードを入力してサインインします。

## ステップ3. SQLステートメントの実行

TiDB Cloud上で最初のSQLステートメントを実行してみましょう。

```sql
SELECT 'Hello TiDB Cloud!';
```

期待される出力:

```sql
+-------------------+
| Hello TiDB Cloud! |
+-------------------+
| Hello TiDB Cloud! |
+-------------------+
```

実際の出力が期待される出力と類似している場合、おめでとうございます。TiDB Cloud上でのSQLステートメントの実行に成功しました。