---
title: PyMySQLを使用してTiDBに接続する
summary: PyMySQLを使用してTiDBに接続する方法について学びます。このチュートリアルでは、PyMySQLを使用してTiDBと連携するためのPythonサンプルコードスニペットが提供されます。
---

# PyMySQLを使用してTiDBに接続

TiDBはMySQL互換のデータベースであり、[PyMySQL](https://github.com/PyMySQL/PyMySQL)はPython用の人気のあるオープンソースドライバです。

このチュートリアルでは、TiDBとPyMySQLを使用して次のタスクを行う方法について学べます:

- 環境をセットアップする。
- PyMySQLを使用してTiDBクラスタに接続する。
- アプリケーションをビルドして実行する。オプションで、基本的なCRUD操作のためのサンプルコードスニペットを見つけることができます。

> **注意:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、およびTiDB Self-Hostedクラスタと共に動作します。

## 前提条件

このチュートリアルを完了するためには、次のものが必要です:

- [Python 3.8 以上](https://www.python.org/downloads/)
- [Git](https://git-scm.com/downloads)
- TiDBクラスタ

<CustomContent platform="tidb">

**TiDBクラスタをお持ちでない場合は、次のように作成できます:**

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)を参照して、独自のTiDBクラウドクラスタを作成します。
- [ローカルテストTiDBクラスタの展開](/quick-start-with-tidb.md#deploy-a-local-test-cluster)、または[プロダクションTiDBクラスタの展開](/production-deployment-using-tiup.md)を参照して、ローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスタをお持ちでない場合は、次のように作成できます:**

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)を参照して、独自のTiDBクラウドクラスタを作成します。
- [ローカルテストTiDBクラスタの展開](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster) または [プロダクションTiDBクラスタの展開](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)を参照して、ローカルクラスタを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を示します。

### ステップ1: サンプルアプリのリポジトリをクローンする

次のコマンドをターミナルウィンドウで実行して、サンプルコードリポジトリをクローンします:

```shell
git clone https://github.com/tidb-samples/tidb-python-pymysql-quickstart.git
cd tidb-python-pymysql-quickstart
```

### ステップ2: 依存関係をインストールする

次のコマンドを実行して、サンプルアプリケーションに必要なパッケージ（PyMySQLを含む）をインストールします:

```shell
pip install -r requirements.txt
```

### ステップ3: 接続情報を構成する

TiDB展開オプションに応じて、TiDBクラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**クラスタ**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの構成がオペレーティング環境に一致することを確認します。

    - **Endpoint Type** は `Public` に設定されていること
    - **Connect With** は `General` に設定されていること
    - **Operating System** が環境に一致していること

    > **TIP:**
    >
    > プログラムをWindows Subsystem for Linux（WSL）で実行している場合は、対応するLinuxディストリビューションに切り替えます。

4. **Create password** をクリックして、ランダムなパスワードを作成します。

    > **TIP:**
    >
    > 以前にパスワードを作成した場合は、元のパスワードを使用するか、新しいパスワードを生成するために **Reset password** をクリックできます。

5. 次のコマンドを実行して `.env.example` をコピーして `.env` にリネームします:

    ```shell
    cp .env.example .env
    ```

6. 対応する接続文字列を `.env` ファイルにコピーして貼り付けます。例は次のようになります:

    ```dotenv
    TIDB_HOST='{host}'  # 例: gateway01.ap-northeast-1.prod.aws.tidbcloud.com
    TIDB_PORT='4000'
    TIDB_USER='{user}'  # 例: xxxxxx.root
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    CA_PATH='{ssl_ca}'  # 例: /etc/ssl/certs/ca-certificates.crt (Debian / Ubuntu / Arch)
    ```

    プレースホルダ `{}` を接続ダイアログから取得した接続パラメータで置き換えてください。

7. `.env` ファイルを保存します。

</div>
<div label="TiDB Dedicated">

1. [**クラスタ**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere** をクリックし、次に **Download TiDB cluster CA** をクリックしてCA証明書をダウンロードします。

    接続文字列の取得方法の詳細については、[TiDB Dedicatedの標準接続](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4. 次のコマンドを実行して `.env.example` をコピーして `.env` にリネームします:

    ```shell
    cp .env.example .env
    ```

5. 対応する接続文字列を `.env` ファイルにコピーして貼り付けます。例は次のようになります:

    ```dotenv
    TIDB_HOST='{host}'  # 例: tidb.xxxx.clusters.tidb-cloud.com
    TIDB_PORT='4000'
    TIDB_USER='{user}'  # 例: root
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    CA_PATH='{your-downloaded-ca-path}'
    ```

    プレースホルダ `{}` を接続ダイアログから取得した接続パラメータで置き換えてください。また、前の手順でダウンロードした証明書のパスで `CA_PATH` を構成してください。

6. `.env` ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して `.env.example` をコピーして `.env` にリネームします:

    ```shell
    cp .env.example .env
    ```

2. 対応する接続文字列を `.env` ファイルにコピーして貼り付けます。例は次のようになります:

    ```dotenv
    TIDB_HOST='{tidb_server_host}'
    TIDB_PORT='4000'
    TIDB_USER='root'
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    ```

    プレースホルダ `{}` を接続パラメータで置き換えてください。また、`CA_PATH` 行は削除してください。TiDBをローカルで実行している場合、デフォルトのホストアドレスは `127.0.0.1` であり、パスワードは空です。

3. `.env` ファイルを保存します。

</div>
</SimpleTab>

### ステップ4: コードを実行して結果を確認する

1. 次のコマンドを実行してサンプルコードを実行します:

    ```shell
    python pymysql_example.py
    ```

2. 出力が一致しているかどうかを確認するために [Expected-Output.txt](https://github.com/tidb-samples/tidb-python-pymysql-quickstart/blob/main/Expected-Output.txt) を確認します。

## サンプルコードスニペット

次のサンプルコードスニペットを参照して、独自のアプリケーション開発を完了させることができます。

詳細なサンプルコードとその実行方法については、[tidb-samples/tidb-python-pymysql-quickstart](https://github.com/tidb-samples/tidb-python-pymysql-quickstart) リポジトリをチェックしてください。

### TiDBに接続

```python
from pymysql import Connection
from pymysql.cursors import DictCursor


def get_connection(autocommit: bool = True) -> Connection:
    config = Config()
    db_conf = {
        "host": ${tidb_host},
        "port": ${tidb_port},
        "user": ${tidb_user},
        "password": ${tidb_password},
        "database": ${tidb_db_name},
        "autocommit": autocommit,
        "cursorclass": DictCursor,
    }

    if ${ca_path}:
        db_conf["ssl_verify_cert"] = True
        db_conf["ssl_verify_identity"] = True
        db_conf["ssl_ca"] = ${ca_path}

    return pymysql.connect(**db_conf)
```

この関数を使用する際には、TiDBクラスタの実際の値で `${tidb_host}`、`${tidb_port}`、`${tidb_user}`、`${tidb_password}`、`${tidb_db_name}`、`${ca_path}` を置き換える必要があります。

### データの挿入

```python
with get_connection(autocommit=True) as conn:
    with conn.cursor() as cur:
        player = ("1", 1, 1)
        cursor.execute("INSERT INTO players (id, coins, goods) VALUES (%s, %s, %s)", player)
```

詳細については、[データの挿入](/develop/dev-guide-insert-data.md)を参照してください。

### データの照会

```python
with get_connection(autocommit=True) as conn:
    with conn.cursor() as cur:
        cur.execute("SELECT count(*) FROM players")
        print(cursor.fetchone()["count(*)"])
```

詳細については、[データの照会](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

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

Pythonドライバーはデータベースへの低レベルなアクセスを提供しますが、開発者には次のことが求められます：

- 手動でデータベース接続を確立および解除すること。
- データベーストランザクションを手動で管理すること。
- データ行をデータオブジェクトに手動でマッピングすること（`pymysql`のタプルまたは辞書として表される）。

複雑なSQL文を書く必要がない限り、[SQLAlchemy](/develop/dev-guide-sample-application-python-sqlalchemy.md)、[Peewee](/develop/dev-guide-sample-application-python-peewee.md)、Django ORMなどの[ORM](https://en.wikipedia.org/w/index.php?title=Object-relational_mapping)フレームワークを開発に使用することが推奨されています。これにより次のことが可能となります：

- 接続とトランザクションの管理における[ぼいらぷれーとコード](https://en.wikipedia.org/wiki/Boilerplate_code)を削減すること。
- 数多くのSQL文の代わりにデータオブジェクトでデータを操作すること。

## 次のステップ

- [PyMySQLのドキュメント](https://pymysql.readthedocs.io)からPyMySQLのさらなる使用法を学びます。
- [開発者ガイド](/develop/dev-guide-overview.md)の章でTiDBアプリケーション開発のベストプラクティスを学びます。たとえば、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルの読み取り](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
- 専門の[TiDB開発者コース](https://www.pingcap.com/education/)を受講し、試験に合格して[TiDB認定資格](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？

<CustomContent platform="tidb">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](/support.md)してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。

</CustomContent>