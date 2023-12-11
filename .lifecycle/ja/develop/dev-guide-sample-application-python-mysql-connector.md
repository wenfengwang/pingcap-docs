---
title: MySQL Connector/Pythonを使用してTiDBに接続する
summary: MySQL Connector/Pythonを使用してTiDBに接続方法を学びます。このチュートリアルでは、TiDBとMySQL Connector/Pythonを使用して、次のタスクを実行するPythonのサンプルコードスニペットを提供します。
aliases: ['/tidb/dev/dev-guide-sample-application-python', '/tidb/dev/dev-guide-outdated-for-python-mysql-connector']
---

# MySQL Connector/Pythonを使用してTiDBに接続する

TiDBはMySQL互換のデータベースであり、[MySQL Connector/Python](https://dev.mysql.com/doc/connector-python/en/)はPythonの公式MySQLドライバです。

このチュートリアルでは、TiDBとMySQL Connector/Pythonを使用して次のタスクを実行する方法を学ぶことができます。

- 環境のセットアップ
- MySQL Connector/Pythonを使用してTiDBクラスタに接続する
- アプリケーションのビルドおよび実行。オプションで、基本的なCRUD操作のためのサンプルコードスニペットを見つけることができます。

> **注意:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、TiDB Self-Hostedクラスタと動作します。

## 前提条件

このチュートリアルを完了するには、次のものが必要です。

- [Python 3.8以上](https://www.python.org/downloads/)
- [Git](https://git-scm.com/downloads)
- TiDBクラスタ

<CustomContent platform="tidb">

**TiDBクラスタをお持ちでない場合は、次のように作成できます:**

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、TiDB Cloudクラスタを作成します。
- [ローカルテスト用TiDBクラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番用TiDBクラスタのデプロイ](/production-deployment-using-tiup.md)に従って、ローカルクラスタを作成します。

</CustomContent>

<CustomContent platform="tidb-cloud">

**TiDBクラスタをお持ちでない場合は、次のように作成できます:**

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、TiDB Cloudクラスタを作成します。
- [ローカルテスト用TiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番用TiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスタを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する

このセクションでは、サンプルアプリケーションコードを実行してTiDBに接続する方法を示します。

### ステップ1: サンプルアプリリポジトリをクローンする

ターミナルウィンドウで以下のコマンドを実行して、サンプルコードリポジトリをクローンします。

```shell
git clone https://github.com/tidb-samples/tidb-python-mysqlconnector-quickstart.git
cd tidb-python-mysqlconnector-quickstart
```

### ステップ2: 依存関係をインストールする

次のコマンドを実行して、サンプルアプリに必要なパッケージ（mysql-connector-pythonを含む）をインストールします。

```shell
pip install -r requirements.txt
```

### ステップ3: 接続情報を設定する

TiDBデプロイメントオプションに応じてTiDBクラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの設定が、ご利用の環境と一致していることを確認します。

    - **Endpoint Type**が`Public`に設定されていること
    - **Connect With**が`General`に設定されていること
    - **Operating System**が環境に一致していること

    > **ヒント:**
    >
    > プログラムがWindows Subsystem for Linux（WSL）で実行されている場合は、対応するLinuxディストリビューションに切り替えます。

4. ランダムなパスワードを作成するには、**Create password**をクリックします。

    > **ヒント:**
    >
    > 以前にパスワードを作成した場合は、元のパスワードを使用するか、新しいパスワードを生成するには**Reset password**をクリックします。

5. 次のコマンドを実行して、`.env.example`をコピーして`.env`にリネームします:

    ```shell
    cp .env.example .env
    ```

6. 対応する接続文字列を`.env`ファイルにコピーして貼り付けます。次の例は、結果です:

    ```dotenv
    TIDB_HOST='{host}'  # 例: gateway01.ap-northeast-1.prod.aws.tidbcloud.com
    TIDB_PORT='4000'
    TIDB_USER='{user}'  # 例: xxxxxx.root
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    CA_PATH='{ssl_ca}'  # 例: /etc/ssl/certs/ca-certificates.crt (Debian / Ubuntu / Arch)
    ```

    プレースホルダ `{}` を接続ダイアログから取得した接続パラメータに置き換えてください。

7. `.env`ファイルを保存します。

</div>
<div label="TiDB Dedicated">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere**をクリックしてから**Download TiDB cluster CA**をクリックして、CA証明書をダウンロードします。

    接続文字列の取得方法の詳細については、[TiDB Dedicated standard connection](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4. 次のコマンドを実行して、`.env.example`をコピーして`.env`にリネームします:

    ```shell
    cp .env.example .env
    ```

5. 対応する接続文字列を`.env`ファイルにコピーして貼り付けます。次の例は、結果です:

    ```dotenv
    TIDB_HOST='{host}'  # 例: tidb.xxxx.clusters.tidb-cloud.com
    TIDB_PORT='4000'
    TIDB_USER='{user}'  # 例: root
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    CA_PATH='{your-downloaded-ca-path}'
    ```

    プレースホルダ `{}` を接続ダイアログから取得した接続パラメータに置き換えてください。また、`CA_PATH`を前の手順でダウンロードした証明書パスに設定してください。

6. `.env`ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して、`.env.example`をコピーして`.env`にリネームします:

    ```shell
    cp .env.example .env
    ```

2. 対応する接続文字列を`.env`ファイルにコピーして貼り付けます。次の例は、結果です:

    ```dotenv
    TIDB_HOST='{tidb_server_host}'
    TIDB_PORT='4000'
    TIDB_USER='root'
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    ```

    プレースホルダ `{}` を接続パラメータに置き換えてください。また、 `CA_PATH`行を削除してください。TiDBをローカルで実行している場合、デフォルトのホストアドレスは `127.0.0.1` であり、パスワードは空です。

3. `.env`ファイルを保存します。

</div>
</SimpleTab>

### ステップ4: コードを実行して結果を確認する

1. 次のコマンドを実行して、サンプルコードを実行します:

    ```shell
    python mysql_connector_example.py
    ```

2. 出力が一致するかどうかは、[Expected-Output.txt](https://github.com/tidb-samples/tidb-python-mysqlconnector-quickstart/blob/main/Expected-Output.txt)を確認します。

## サンプルコードスニペット

次のサンプルコードスニペットを参照して、独自のアプリケーション開発を完了させることができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-python-mysqlconnector-quickstart](https://github.com/tidb-samples/tidb-python-mysqlconnector-quickstart)リポジトリを参照してください。

### TiDBに接続する

```python
def get_connection(autocommit: bool = True) -> MySQLConnection:
    config = Config()
    db_conf = {
        "host": ${tidb_host},
        "port": ${tidb_port},
        "user": ${tidb_user},
        "password": ${tidb_password},
        "database": ${tidb_db_name},
        "autocommit": autocommit,
        "use_pure": True,
    }

    if ${ca_path}:
        db_conf["ssl_verify_cert"] = True
        db_conf["ssl_verify_identity"] = True
        db_conf["ssl_ca"] = ${ca_path}
    return mysql.connector.connect(**db_conf)
```

この関数を使用する場合は、TiDBクラスタの実際の値で`${tidb_host}`、 `${tidb_port}`、 `${tidb_user}`、`${tidb_password}`、`${tidb_db_name}`、`${ca_path}`を置き換える必要があります。

### データの挿入

```python
with get_connection(autocommit=True) as conn:
    with conn.cursor() as cur:
        player = ("1", 1, 1)
```python
cursor.execute("INSERT INTO players (id, coins, goods) VALUES (%s, %s, %s)", player)
```

詳細については、[データの挿入](/develop/dev-guide-insert-data.md)を参照してください。

### データのクエリ

```python
with get_connection(autocommit=True) as conn:
    with conn.cursor() as cur:
        cur.execute("SELECT count(*) FROM players")
        print(cur.fetchone()[0])
```

詳細については、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データの更新

```python
with get_connection(autocommit=True) as conn:
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
with get_connection(autocommit=True) as conn:
    with conn.cursor() as cur:
        player_id = "1"
        cursor.execute("DELETE FROM players WHERE id = %s", (player_id,))
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md)を参照してください。

## 便利なノート

### ドライバーまたはORMフレームワークを使用しますか？

Pythonドライバーはデータベースへの低レベルなアクセスを提供しますが、開発者は次のことを行う必要があります。

- 手動でデータベース接続を確立し、解除する。
- 手動でデータベーストランザクションを管理する。
- データ行（`mysql-connector-python`のタプルまたは辞書で表される）をデータオブジェクトに手動でマップする。

複雑なSQL文を記述する必要がない場合は、[ORM](https://en.wikipedia.org/w/index.php?title=Object-relational_mapping)フレームワーク（たとえば[SQLAlchemy](/develop/dev-guide-sample-application-python-sqlalchemy.md)、[Peewee](/develop/dev-guide-sample-application-python-peewee.md)、Django ORMなど）を使用することをお勧めします。これにより、次のことができます。

- 接続とトランザクションの管理に対する[ボイラープレートコード](https://en.wikipedia.org/wiki/Boilerplate_code)を削減する。
- 一連のSQL文の代わりにデータオブジェクトでデータを操作する。

## 次のステップ

- [MySQL Connector/Pythonのドキュメント](https://dev.mysql.com/doc/connector-python/en/)でmysql-connector-pythonのさらなる使用法を学ぶ。
- [開発者ガイド](/develop/dev-guide-overview.md)の章でTiDBアプリケーション開発のベストプラクティスを学ぶ。例：[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルの読み取り](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
- [TiDB開発者コース](https://www.pingcap.com/education/)を通じて学習し、試験に合格した後に[TiDB認定資格](https://www.pingcap.com/education/certification/)を取得する。

## 必要な場合はヘルプが必要ですか？

<CustomContent platform="tidb">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問したり、[サポートチケットを作成](/support.md)することができます。

</CustomContent>

<CustomContent platform="tidb-cloud">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問したり、[サポートチケットを作成](https://support.pingcap.com/)することができます。

</CustomContent>
```