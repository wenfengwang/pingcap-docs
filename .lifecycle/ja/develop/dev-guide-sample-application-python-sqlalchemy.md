---
title: SQLAlchemyを使用してTiDBに接続する
summary: SQLAlchemyを使用してTiDBに接続する方法を学びます。このチュートリアルでは、Pythonのサンプルコードスニペットが含まれており、SQLAlchemyを使用してTiDBと連携する方法を説明します。
aliases: ['/tidb/dev/dev-guide-outdated-for-sqlalchemy']
---

# SQLAlchemyを使用してTiDBに接続する

TiDBはMySQL互換データベースであり、[SQLAlchemy](https://www.sqlalchemy.org/)は人気のあるPython SQLツールキットおよびオブジェクト関係マッピング（ORM）です。

このチュートリアルでは、TiDBとSQLAlchemyを使用して、次のタスクを達成する方法を学ぶことができます。

- 環境をセットアップします。
- SQLAlchemyを使用してTiDBクラスタに接続します。
- アプリケーションを構築し、実行します。オプションで、基本的なCRUD操作のためのサンプルコードスニペットが提供されています。

> **注意:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、およびTiDB Self-Hostedクラスタで動作します。

## 前提条件

このチュートリアルを完了するには、次のものが必要です。

- [Python 3.8 以上](https://www.python.org/downloads/)
- [Git](https://git-scm.com/downloads)
- TiDBクラスタ

<CustomContent platform="tidb">

**TiDBクラスタをお持ちでない場合は、次の手順に従って作成できます:**

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[プロダクションTiDBクラスタのデプロイ](/production-deployment-using-tiup.md)に従って、ローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスタをお持ちでない場合は、次の手順に従って作成できます:**

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[プロダクションTiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスタを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を示します。

### ステップ1: サンプルアプリケーションのリポジトリをクローンする

次のコマンドをターミナルウィンドウで実行して、サンプルコードリポジトリをクローンします:

```shell
git clone https://github.com/tidb-samples/tidb-python-sqlalchemy-quickstart.git
cd tidb-python-sqlalchemy-quickstart
```

### ステップ2: 依存関係をインストールする

次のコマンドを実行して、サンプルアプリケーションに必要なパッケージ（SQLAlchemyおよびPyMySQLを含む）をインストールします:

```shell
pip install -r requirements.txt
```

#### なぜPyMySQLを使用するのですか？

SQLAlchemyは複数のデータベースと連携するORMライブラリであり、開発者がよりオブジェクト指向的な方法でSQL文を書くのに役立つ、データベースの高レベルな抽象化を提供します。ただし、SQLAlchemyにはデータベースドライバが含まれていません。データベースに接続するには、データベースドライバをインストールする必要があります。このサンプルアプリケーションでは、データベースドライバとして、TiDBと互換性のある純粋なPython MySQLクライアントライブラリであるPyMySQLを使用しています。

[mysqlclient](https://github.com/PyMySQL/mysqlclient)や[mysql-connector-python](https://dev.mysql.com/doc/connector-python/en/)などの他のデータベースドライバを使用することもできますが、これらは純粋なPythonライブラリではなく、対応するC/C++コンパイラとMySQLクライアントが必要です。詳細については、[SQLAlchemy公式ドキュメント](https://docs.sqlalchemy.org/en/20/core/engines.html#mysql)を参照してください。

### ステップ3: 接続情報を構成する

TiDBのデプロイオプションに応じて、TiDBクラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の**接続**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの構成が操作環境と一致していることを確認します。

    - **エンドポイントタイプ**が `Public` に設定されていること
    - **接続先** が `General` に設定されていること
    - **オペレーティングシステム**が環境に一致していること

    > **ヒント:**
    >
    > プログラムがWindows Subsystem for Linux（WSL）で実行されている場合は、対応するLinuxディストリビューションに切り替えます。

4. `.env.example`をコピーして`.env`にリネームするために次のコマンドを実行します:

    ```shell
    cp .env.example .env
    ```

5. `.env`ファイルに対応する接続文字列をコピーして貼り付けます。次の例は、その結果です:

    ```dotenv
    TIDB_HOST='{host}'  # 例: gateway01.ap-northeast-1.prod.aws.tidbcloud.com
    TIDB_PORT='4000'
    TIDB_USER='{user}'  # 例: xxxxxx.root
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    CA_PATH='{ssl_ca}'  # 例: /etc/ssl/certs/ca-certificates.crt (Debian / Ubuntu / Arch)
    ```

    プレースホルダ `{}` を接続ダイアログから取得した接続パラメータで置き換えてください。

6. `.env`ファイルを保存します。

</div>
<div label="TiDB Dedicated">

1. [**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の**接続**をクリックします。接続ダイアログが表示されます。

3. **どこからでもアクセスを許可** をクリックし、**TiDBクラスタCAをダウンロード** をクリックしてCA証明書をダウンロードします。

    接続文字列の取得方法の詳細については、[TiDB Dedicated標準接続](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4. `.env.example`をコピーして`.env`にリネームするために次のコマンドを実行します:

    ```shell
    cp .env.example .env
    ```

5. `.env`ファイルに対忖する接続文字列をコピーして貼り付けます。次の例は、その結果です:

    ```dotenv
    TIDB_HOST='{host}'  # 例: tidb.xxxx.clusters.tidb-cloud.com
    TIDB_PORT='4000'
    TIDB_USER='{user}'  # 例: root
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    CA_PATH='{your-downloaded-ca-path}'
    ```

    接続ダイアログから取得した接続パラメータでプレースホルダ `{}` を置き換え、`CA_PATH` を前の手順でダウンロードした証明書パスに構成してください。

6. `.env`ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1. `.env.example`をコピーして`.env`にリネームするために次のコマンドを実行します:

    ```shell
    cp .env.example .env
    ```

2. `.env`ファイルに対囮する接続文字列をコピーして貼り付けます。次の例は、その結果です:

    ```dotenv
    TIDB_HOST='{tidb_server_host}'
    TIDB_PORT='4000'
    TIDB_USER='root'
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    ```

    プレースホルダ `{}` を接続パラメータで置き換えてください。また、`CA_PATH`行を削除してください。TiDBをローカルで実行している場合、デフォルトのホストアドレスは `127.0.0.1` であり、パスワードは空です。

3. `.env`ファイルを保存します。

</div>
</SimpleTab>

### ステップ4: コードを実行して結果を確認する

1. 次のコマンドを実行して、サンプルコードを実行します:

    ```shell
    python sqlalchemy_example.py
    ```

2. 出力が期待どおりかを確認するために、[Expected-Output.txt](https://github.com/tidb-samples/tidb-python-sqlalchemy-quickstart/blob/main/Expected-Output.txt)を確認します。

## サンプルコードスニペット

次のサンプルコードスニペットを参照して、独自のアプリケーション開発を完了させることができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-python-sqlalchemy-quickstart](https://github.com/tidb-samples/tidb-python-sqlalchemy-quickstart)リポジトリをご覧ください。

### TiDBに接続する

```python
from sqlalchemy import create_engine, URL
from sqlalchemy.orm import sessionmaker

def get_db_engine():
    connect_args = {}
    if ${ca_path}:
        connect_args = {
            "ssl_verify_cert": True,
            "ssl_verify_identity": True,
            "ssl_ca": ${ca_path},
        }
    return create_engine(
        URL.create(
            drivername="mysql+pymysql",
            username=${tidb_user},
```python
      パスワード=${tidb_password},
      ホスト=${tidb_host},
      ポート=${tidb_port},
      データベース=${tidb_db_name},
  ),
  connect_args=connect_args,
)

engine = get_db_engine()
Session = sessionmaker(bind=engine)
```

この関数を使用する際は、`${tidb_host}`、`${tidb_port}`、`${tidb_user}`、`${tidb_password}`、`${tidb_db_name}`、`${ca_path}`の実際の値に置き換える必要があります。

### テーブルを定義する

```python
from sqlalchemy import Column, Integer, String
from sqlalchemy.orm import declarative_base

Base = declarative_base()

class Player(Base):
    id = Column(Integer, primary_key=True)
    name = Column(String(32), unique=True)
    coins = Column(Integer)
    goods = Column(Integer)

    __tablename__ = "players"
```

詳細は、[SQLAlchemyドキュメント：デクラレーティブを使用したクラスのマッピング](https://docs.sqlalchemy.org/en/20/orm/declarative_mapping.html)を参照してください。

### データを挿入する

```python
with Session() as session:
    player = Player(name="test", coins=100, goods=100)
    session.add(player)
    session.commit()
```

詳細は、[データの挿入](/develop/dev-guide-insert-data.md)を参照してください。

### データをクエリする

```python
with Session() as session:
    player = session.query(Player).filter_by(name == "test").one()
    print(player)
```

詳細は、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データを更新する

```python
with Session() as session:
    player = session.query(Player).filter_by(name == "test").one()
    player.coins = 200
    session.commit()
```

詳細は、[データの更新](/develop/dev-guide-update-data.md)を参照してください。

### データを削除する

```python
with Session() as session:
    player = session.query(Player).filter_by(name == "test").one()
    session.delete(player)
    session.commit()
```

詳細は、[データの削除](/develop/dev-guide-delete-data.md)を参照してください。

## 次の手順

- [SQLAlchemyドキュメント](https://www.sqlalchemy.org/)からSQLAlchemyの詳細な使い方を学びます。
- [開発者ガイド](/develop/dev-guide-overview.md)の章を通じて、TiDBアプリケーション開発のベストプラクティスを学びます。[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルからのデータ取得](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)に関する章を学びます。
- プロフェッショナルな[TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格した後に[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？

<CustomContent platform="tidb">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問したり、[サポートチケットを作成](/support.md)したりしてください。

</CustomContent>

<CustomContent platform="tidb-cloud">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問したり、[サポートチケットを作成](https://support.pingcap.com/)したりしてください。

</CustomContent>
```