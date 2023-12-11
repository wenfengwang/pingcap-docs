---
title: peeweeを使用してTiDBに接続する
summary: peeweeを使用してTiDBに接続する方法を学びます。このチュートリアルでは、Pythonのサンプルコードスニペットを使用してpeeweeを使ってTiDBと連携する方法が示されています。
---

# peeweeを使用してTiDBに接続する

TiDBはMySQL互換のデータベースであり、[peewee](https://docs.peewee-orm.com/)はPython向けの人気のあるObject Relational Mapper (ORM) です。

このチュートリアルでは、TiDBとpeeweeを使用して次のタスクを実行する方法を学ぶことができます。

- 環境のセットアップ。
- peeweeを使用してTiDBクラスタに接続する。
- アプリケーションのビルドと実行。オプションで、基本的なCRUD操作のためのサンプルコードスニペットを見つけることができます。

> **注意:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、およびTiDB Self-Hostedクラスタと共に動作します。

## 前提条件

このチュートリアルを完了するには、次のものが必要です。

- [Python 3.8 以上](https://www.python.org/downloads/)
- [Git](https://git-scm.com/downloads)
- TiDBクラスタ

<CustomContent platform="tidb">

**TiDBクラスタを持っていない場合、次のように作成できます:**

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)の手順に従って、TiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)や[本番用TiDBクラスタのデプロイ](/production-deployment-using-tiup.md)の手順に従って、ローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスタを持っていない場合、次のように作成できます:**

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)の手順に従って、TiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)や[本番用TiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)の手順に従って、ローカルクラスタを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を示します。

### ステップ1: サンプルアプリのリポジトリをクローンする

次のコマンドをターミナルウィンドウで実行して、サンプルコードリポジトリをクローンします。

```shell
git clone https://github.com/tidb-samples/tidb-python-peewee-quickstart.git
cd tidb-python-peewee-quickstart
```

### ステップ2: 依存関係をインストールする

次のコマンドを実行して、サンプルアプリケーション用に必要なパッケージ（peeweeやPyMySQLを含む）をインストールします。

```shell
pip install -r requirements.txt
```

#### PyMySQLを使用する理由は？

peeweeは複数のデータベースと連携するORMライブラリであり、データベースの高レベルな抽象化を提供し、開発者がよりオブジェクト指向的な方法でSQLステートメントを書くのを支援します。ただし、peeweeにはデータベースドライバが含まれていません。データベースに接続するには、データベースドライバをインストールする必要があります。このサンプルアプリケーションでは、TiDBと互換性のある純粋なPython MySQLクライアントライブラリであるPyMySQLをデータベースドライバとして使用しています。詳細については、[peewee公式ドキュメント](https://docs.peewee-orm.com/en/latest/peewee/database.html?highlight=mysql#using-mysql)を参照してください。

### ステップ3: 接続情報を設定する

選択したTiDBの展開オプションに応じて、TiDBクラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの設定が環境に一致することを確認します。

    - **Endpoint Type**が`Public`に設定されていること
    - **Connect With**が`General`に設定されていること
    - **Operating System**が環境に一致すること

    > **ヒント:**
    >
    > プログラムがWindows Subsystem for Linux (WSL)で実行されている場合は、対応するLinuxディストリビューションに切り替えてください。

4. ランダムなパスワードを作成するには、**Create password**をクリックします。

    > **ヒント:**
    >
    > 以前にパスワードを作成した場合は、元のパスワードを使用するか、新しいパスワードを生成するために**Reset password**をクリックできます。

5. 次のコマンドを実行して`.env.example`をコピーし、`.env`にリネームします:

    ```shell
    cp .env.example .env
    ```

6. 対応する接続文字列を`.env`ファイルにコピーして貼り付けます。例を以下に示します：

    ```dotenv
    TIDB_HOST='{host}'  # 例: gateway01.ap-northeast-1.prod.aws.tidbcloud.com
    TIDB_PORT='4000'
    TIDB_USER='{user}'  # 例: xxxxxx.root
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    CA_PATH='{ssl_ca}'  # 例: /etc/ssl/certs/ca-certificates.crt (Debian / Ubuntu / Arch)
    ```

    接続ダイアログから取得した接続パラメータでプレースホルダ `{}` を置き換えてください。

7. `.env`ファイルを保存します。

</div>
<div label="TiDB Dedicated">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere**をクリックし、その後**Download TiDB cluster CA**をクリックしてCA証明書をダウンロードします。

    接続文字列の取得方法の詳細については、[TiDB Dedicated標準接続](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4. 次のコマンドを実行して`.env.example`をコピーし、`.env`にリネームします:

    ```shell
    cp .env.example .env
    ```

5. 対応する接続文字列を`.env`ファイルにコピーして貼り付けます。例を以下に示します：

    ```dotenv
    TIDB_HOST='{host}'  # 例: tidb.xxxx.clusters.tidb-cloud.com
    TIDB_PORT='4000'
    TIDB_USER='{user}'  # 例: root
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    CA_PATH='{your-downloaded-ca-path}'
    ```

    接続ダイアログから取得した接続パラメータでプレースホルダ `{}` を置き換えてください。また、前の手順でダウンロードした証明書のパスを`CA_PATH`に設定してください。

6. `.env`ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して`.env.example`をコピーし、`.env`にリネームします:

    ```shell
    cp .env.example .env
    ```

2. 対応する接続文字列を`.env`ファイルにコピーして貼り付けます。例を以下に示します：

    ```dotenv
    TIDB_HOST='{tidb_server_host}'
    TIDB_PORT='4000'
    TIDB_USER='root'
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    ```

    接続パラメータをプレースホルダ `{}` で置き換えてください。また、`CA_PATH`の行を削除してください。TiDBをローカルで実行している場合、デフォルトのホストアドレスは`127.0.0.1`であり、パスワードは空です。

3. `.env`ファイルを保存します。

</div>
</SimpleTab>

### ステップ4: コードを実行して結果を確認する

1. 次のコマンドを実行してサンプルコードを実行します：

    ```shell
    python peewee_example.py
    ```

2. 出力が期待どおりかどうかを確認するために[Expected-Output.txt](https://github.com/tidb-samples/tidb-python-peewee-quickstart/blob/main/Expected-Output.txt)を確認します。

## サンプルコードスニペット

次のサンプルコードスニペットを参照して、独自のアプリケーション開発を完了させることができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-python-peewee-quickstart](https://github.com/tidb-samples/tidb-python-peewee-quickstart)リポジトリを確認してください。

### TiDBに接続する

```python
from peewee import MySQLDatabase

def get_db_engine():
    config = Config()
    connect_params = {}
    if ${ca_path}:
        connect_params = {
            "ssl_verify_cert": True,
            "ssl_verify_identity": True,
            "ssl_ca": ${ca_path},
        }
    return MySQLDatabase(
        ${tidb_db_name},
        host=${tidb_host},
        port=${tidb_port},
        user=${tidb_user},
        password=${tidb_password},
        **connect_params,
    )
```
この機能を使用する場合は、 `${tidb_host}`, `${tidb_port}`, `${tidb_user}`, `${tidb_password}`, `${tidb_db_name}`, `${ca_path}` を、TiDBクラスターの実際の値に置き換える必要があります。

### テーブルを定義する

```python
from peewee import Model, CharField, IntegerField

db = get_db_engine()

class BaseModel(Model):
    class Meta:
        database = db

class Player(BaseModel):
    name = CharField(max_length=32, unique=True)
    coins = IntegerField(default=0)
    goods = IntegerField(default=0)

    class Meta:
        table_name = "players"
```

詳細については、[peewee documentation: Models and Fields](https://docs.peewee-orm.com/en/latest/peewee/models.html) を参照してください。

### データを挿入する

```python
# 1つのレコードを挿入
Player.create(name="test", coins=100, goods=100)

# 複数のレコードを挿入
Player.insert_many(
    [
        {"name": "test1", "coins": 100, "goods": 100},
        {"name": "test2", "coins": 100, "goods": 100},
    ]
).execute()
```

詳細については、[データの挿入](/develop/dev-guide-insert-data.md) を参照してください。

### データをクエリする

```python
# すべてのレコードをクエリ
players = Player.select()

# 単一のレコードをクエリ
player = Player.get(Player.name == "test")

# 複数のレコードをクエリ
players = Player.select().where(Player.coins == 100)
```

詳細については、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md) を参照してください。

### データを更新する

```python
# 単一のレコードを更新
player = Player.get(Player.name == "test")
player.coins = 200
player.save()

# 複数のレコードを更新
Player.update(coins=200).where(Player.coins == 100).execute()
```

詳細については、[データの更新](/develop/dev-guide-update-data.md) を参照してください。

### データを削除する

```python
# 単一のレコードを削除
player = Player.get(Player.name == "test")
player.delete_instance()

# 複数のレコードを削除
Player.delete().where(Player.coins == 100).execute()
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md) を参照してください。

## 次のステップ

- [peeweeのドキュメント](https://docs.peewee-orm.com/) からpeeweeのさらなる使用方法を学びます。
- [開発者ガイド](/develop/dev-guide-overview.md)の章を通じて、TiDBアプリケーション開発のベストプラクティスを学びます。[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルの読み取り](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)を参照してください。
- [TiDB開発者コース](https://www.pingcap.com/education/)を通じて学習し、試験に合格すると[TiDB認定](https://www.pingcap.com/education/certification/)を取得できます。

## ヘルプが必要ですか？

<CustomContent platform="tidb">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](/support.md)してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。

</CustomContent>