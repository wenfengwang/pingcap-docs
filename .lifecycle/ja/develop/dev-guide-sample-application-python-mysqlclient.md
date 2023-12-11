---
title: mysqlclientを使用してTiDBに接続する
summary: mysqlclientを使用してTiDBに接続する方法を学びます。このチュートリアルでは、mysqlclientを使用したTiDBと連携するPythonサンプルコードスニペットを提供します。
---

# mysqlclientを使用してTiDBに接続する

TiDBはMySQL互換のデータベースであり、[mysqlclient](https://github.com/PyMySQL/mysqlclient)はPython向けの人気のあるオープンソースドライバです。

このチュートリアルでは、TiDBとmysqlclientを使用して次のタスクを実行する方法を学ぶことができます。

- 環境のセットアップ。
- mysqlclientを使用してTiDBクラスタに接続。
- アプリケーションのビルドと実行。オプションとして、基本的なCRUD操作のためのサンプルコードスニペットも利用できます。

> **注意:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、TiDB Self-Hostedと連携します。

## 前提条件

このチュートリアルを完了するには、次のものが必要です。

- [Python **3.10** 以上](https://www.python.org/downloads/)
- [Git](https://git-scm.com/downloads)
- TiDBクラスタ

<CustomContent platform="tidb">

**TiDBクラスタをお持ちでない場合、次の方法で作成できます:**

- (推奨) [TiDB Serverlessクラスタを作成](/develop/dev-guide-build-cluster-in-cloud.md) に従って、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタをデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster) または [プロダクションTiDBクラスタをデプロイ](/production-deployment-using-tiup.md) に従って、ローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスタをお持ちでない場合、次の方法で作成できます:**

- (推奨) [TiDB Serverlessクラスタを作成](/develop/dev-guide-build-cluster-in-cloud.md) に従って、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタをデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster) または [プロダクションTiDBクラスタをデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup) に従って、ローカルクラスタを作成します。

</CustomContent>

## サンプルアプリの実行とTiDBへの接続

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を示します。

### ステップ1: サンプルアプリのリポジトリをクローン

次のコマンドをターミナルウィンドウで実行して、サンプルコードリポジトリをクローンします:

```shell
git clone https://github.com/tidb-samples/tidb-python-mysqlclient-quickstart.git
cd tidb-python-mysqlclient-quickstart;
```

### ステップ2: 依存関係のインストール

次のコマンドを実行して、サンプルアプリケーションの必要なパッケージ（`mysqlclient`を含む）をインストールします:

```shell
pip install -r requirements.txt
```

インストールに問題がある場合は、[mysqlclientの公式ドキュメント](https://github.com/PyMySQL/mysqlclient#install)を参照してください。

### ステップ3: 接続情報の設定

TiDBデプロイオプションに応じてTiDBクラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの構成がオペレーティング環境に一致することを確認します。

    - **Endpoint Type** が `Public` に設定されていること
    - **Connect With** が `General` に設定されていること
    - オペレーティングシステムが環境と一致していること

    > **ヒント:**
    >
    > プログラムがWindows Subsystem for Linux (WSL)で実行されている場合、対応するLinuxディストリビューションに切り替えてください。

4. ランダムなパスワードを作成するには、 **Create password** をクリックします。

    > **ヒント:**
    >
    > 以前にパスワードを作成した場合は、元のパスワードを使用するか、新しいパスワードを生成するために **Reset password** をクリックできます。

5. 次のコマンドを実行して `.env.example` をコピーし、 `.env` に名前を変更します:

    ```shell
    cp .env.example .env
    ```

6. 対応する接続文字列を `.env` ファイルにコピーして貼り付けます。例を以下に示します:

    ```dotenv
    TIDB_HOST='{gateway-region}.aws.tidbcloud.com'
    TIDB_PORT='4000'
    TIDB_USER='{prefix}.root'
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    CA_PATH=''
    ```

    接続ダイアログから取得した接続パラメータでプレースホルダー `{}` を置き換えてください。

    TiDB Serverlessでは安全な接続が必要です。`mysqlclient`の`ssl_mode`のデフォルト値は `PREFERRED` なので、`CA_PATH` を手動で指定する必要はありません。空のままにしてください。ただし、特別な理由で`CA_PATH` を手動で指定する必要がある場合は、[TiDB ServerlessへのTLS接続](https://docs.pingcap.com/tidbcloud/secure-connections-to-serverless-clusters)を参照して、異なるオペレーティングシステム用の証明書パスを取得してください。

7. `.env` ファイルを保存します。

</div>
<div label="TiDB Dedicated">

1. [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere** をクリックし、次に **Download TiDB cluster CA** をクリックしてCA証明書をダウンロードします。

    接続文字列の取得方法の詳細については、[TiDB Dedicated標準接続](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4. 次のコマンドを実行して `.env.example` をコピーし、 `.env` に名前を変更します:

    ```shell
    cp .env.example .env
    ```

5. 対応する接続文字列を `.env` ファイルにコピーして貼り付けます。例を以下に示します:

    ```dotenv
    TIDB_HOST='{host}.clusters.tidb-cloud.com'
    TIDB_PORT='4000'
    TIDB_USER='{username}'
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    CA_PATH='{your-downloaded-ca-path}'
    ```

    接続ダイアログから取得した接続パラメータでプレースホルダー `{}` を置き換えてください。また、前のステップでダウンロードした証明書パスで `CA_PATH` を構成してください。

6. `.env` ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して `.env.example` をコピーし、 `.env` に名前を変更します:

    ```shell
    cp .env.example .env
    ```

2. 対応する接続文字列を `.env` ファイルにコピーして貼り付けます。例を以下に示します:

    ```dotenv
    TIDB_HOST='{tidb_server_host}'
    TIDB_PORT='4000'
    TIDB_USER='root'
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    ```

    接続パラメータでプレースホルダー `{}` を置き換えてください。また、`CA_PATH` 行を削除してください。TiDBをローカルで実行している場合、デフォルトのホストアドレスは `127.0.0.1` 、パスワードは空です。

3. `.env` ファイルを保存します。

</div>
</SimpleTab>

### ステップ4: コードを実行して結果を確認

1. 次のコマンドを実行してサンプルコードを実行します:

    ```shell
    python mysqlclient_example.py
    ```

2. 出力が期待どおりかどうかを確認するには、[Expected-Output.txt](https://github.com/tidb-samples/tidb-python-mysqlclient-quickstart/blob/main/Expected-Output.txt) を確認してください。

## サンプルコードスニペット

次のサンプルコードスニペットを参照して、独自のアプリケーション開発を完了させることができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-python-mysqlclient-quickstart](https://github.com/tidb-samples/tidb-python-mysqlclient-quickstart) リポジトリを確認してください。

### TiDBに接続

```python
def get_mysqlclient_connection(autocommit:bool=True) -> MySQLdb.Connection:
    db_conf = {
        "host": ${tidb_host},
        "port": ${tidb_port},
        "user": ${tidb_user},
        "password": ${tidb_password},
        "database": ${tidb_db_name},
        "autocommit": autocommit
    }

    if ${ca_path}:
        db_conf["ssl_mode"] = "VERIFY_IDENTITY"
        db_conf["ssl"] = {"ca": ${ca_path}}

    return MySQLdb.connect(**db_conf)
```

この関数を使用する際には、TiDBクラスタの実際の値で `${tidb_host}`、`${tidb_port}`、`${tidb_user}`、`${tidb_password}`、`${tidb_db_name}`、`${ca_path}` を置き換える必要があります。

### データの挿入

```python
with get_mysqlclient_connection(autocommit=True) as conn:
    with conn.cursor() as cur:
        player = ("1", 1, 1)
```
```python
cursor.execute("INSERT INTO players (id, coins, goods) VALUES (%s, %s, %s)", player)
```

詳細については、[データの挿入](/develop/dev-guide-insert-data.md)を参照してください。

### データのクエリ

```python
with get_mysqlclient_connection(autocommit=True) as conn:
    with conn.cursor() as cur:
        cur.execute("SELECT count(*) FROM players")
        print(cur.fetchone()[0])
```

詳細については、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データの更新

```python
with get_mysqlclient_connection(autocommit=True) as conn:
    with conn.cursor() as cur:
        player_id, amount, price="1", 10, 500
        cursor.execute(
            "UPDATE players SET goods = goods + %s, coins = coins + %s WHERE id = %s",
            (-amount, price, player_id),
        )
```

詳細については、[データの更新](/develop/dev-guide-update-data.md)を参照してください。

### データの削除

```python
with get_mysqlclient_connection(autocommit=True) as conn:
    with conn.cursor() as cur:
        player_id = "1"
        cursor.execute("DELETE FROM players WHERE id = %s", (player_id,))
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md)を参照してください。

## 便利なメモ

### ドライバーまたはORMフレームワークを使用する？

Pythonのドライバーはデータベースへの低レベルのアクセスを提供しますが、以下が必要です：

- データベース接続の手動の確立と解除。
- データベーストランザクションの手動の管理。
- データ行（`mysqlclient`内のタプルとして表される）をデータオブジェクトに手動でマッピングする。

複雑なSQL文を書く必要がない場合は、[ORM](https://ja.wikipedia.org/wiki/ORM)フレームワーク、例えば[SQLAlchemy](/develop/dev-guide-sample-application-python-sqlalchemy.md)や[Peewee](/develop/dev-guide-sample-application-python-peewee.md)、Django ORMを使用することをお勧めします。これにより、次のようなことができます：

- 接続とトランザクションの管理における[冗長なコード](https://ja.wikipedia.org/wiki/冗長なコード)を減らす。
- 一連のSQL文の代わりにデータオブジェクトでデータを操作する。

## 次のステップ

- [mysqlclientのドキュメント](https://mysqlclient.readthedocs.io/)で`mysqlclient`のより多くの使用法を学びます。
- [開発者ガイド](/develop/dev-guide-overview.md)の章でTiDBアプリケーション開発のベストプラクティスを学びます。例：[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルのデータ取得](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
- プロフェッショナルな[TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格して[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。

## 必要なヘルプか？
<CustomContent platform="tidb">
[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](/support.md)してください。
</CustomContent>
<CustomContent platform="tidb-cloud">
[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。
</CustomContent>
```