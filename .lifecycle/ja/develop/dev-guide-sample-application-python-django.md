---
title: Djangoを使用してTiDBに接続する
summary: Djangoを使用してTiDBに接続する方法を学びます。このチュートリアルでは、Pythonのサンプルコードスニペットを使用してTiDBをDjangoと連携させる方法が説明されています。
aliases: ['/tidb/dev/dev-guide-outdated-for-django']
---

# Djangoを使用してTiDBに接続

TiDBはMySQL互換のデータベースであり、[Django](https://www.djangoproject.com)はPython向けの人気のあるWebフレームワークであり、強力なオブジェクトリレーショナルマッピング（ORM）ライブラリを含んでいます。

このチュートリアルでは、TiDBとDjangoを使用して次のタスクを実行する方法を学ぶことができます。

- 環境をセットアップする。
- Djangoを使用してTiDBクラスタに接続する。
- アプリケーションを構築し実行する。オプションで、基本的なCRUD操作のためのサンプルコードスニペットを見つけることができます。

> **注意:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、TiDB Self-Hostedクラスタと共に動作します。

## 前提条件

このチュートリアルを完了するには、以下が必要です:

- [Python 3.8 以上](https://www.python.org/downloads/)
- [Git](https://git-scm.com/downloads)
- TiDBクラスタ

<CustomContent platform="tidb">

**TiDBクラスタをお持ちでない場合は、以下の手順で作成できます:**

- (推奨) [TiDB Serverlessクラスタを作成](/develop/dev-guide-build-cluster-in-cloud.md)し、独自のTiDB Cloudクラスタを作成します。
- [ローカルテスト用TiDBクラスタをデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または [プロダクション向けTiDBクラスタをデプロイ](/production-deployment-using-tiup.md) してローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスタをお持ちでない場合は、以下の手順で作成できます:**

- (推奨) [TiDB Serverlessクラスタを作成](/develop/dev-guide-build-cluster-in-cloud.md)し、独自のTiDB Cloudクラスタを作成します。
- [ローカルテスト用TiDBクラスタをデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster) または [プロダクション向けTiDBクラスタをデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup) してローカルクラスタを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を示します。

### ステップ1: サンプルアプリのリポジトリをクローン

以下のコマンドを端末ウィンドウで実行して、サンプルコードのリポジトリをクローンします:

```shell
git clone https://github.com/tidb-samples/tidb-python-django-quickstart.git
cd tidb-python-django-quickstart
```

### ステップ2: 依存関係をインストール

以下のコマンドを実行して、サンプルアプリケーションに必要なパッケージ（Django、django-tidb、mysqlclientを含む）をインストールします:

```shell
pip install -r requirements.txt
```

mysqlclientのインストールに関する問題がある場合は、[mysqlclient公式ドキュメント](https://github.com/PyMySQL/mysqlclient#install)を参照してください。

#### `django-tidb`とは？

`django-tidb`は、TiDBとDjangoの互換性の問題を解決するためのDjango向けのTiDB方言です。

`django-tidb`をインストールするには、Djangoバージョンに合ったバージョンを選択してください。たとえば、`django==4.2.*`を使用している場合、`django-tidb==4.2.*`をインストールしてください。マイナーバージョンは同じである必要はありません。最新のマイナーバージョンを使用することを推奨します。

詳細については、[django-tidbリポジトリ](https://github.com/pingcap/django-tidb)を参照してください。

### ステップ3: 接続情報を構成する

TiDBデプロイメントオプションに応じてTiDBクラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、対象のクラスタ名をクリックして概要ページに移動します。

2. 右上隅にある**Connect**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの構成が使用環境に一致することを確認してください。

    - **Endpoint Type**が `Public` に設定されていること
    - **Connect With**が `General` に設定されていること
    - **Operating System**が使用環境に一致していること

    > **ヒント:**
    >
    > プログラムがWindows Subsystem for Linux (WSL)で実行されている場合は、対応するLinuxディストリビューションに切り替えてください。

4. **Create password**をクリックしてランダムなパスワードを作成します。

    > **ヒント:**
    >
    > 以前にパスワードを作成した場合は、元のパスワードを使用するか、新しいパスワードを生成するために **Reset password**をクリックできます。

5. 以下のコマンドを実行して `.env.example` をコピーし、`.env` として名前を変更します:

    ```shell
    cp .env.example .env
    ```

6. 対応する接続文字列を`.env`ファイルにコピーして貼り付けます。例を以下に示します:

    ```dotenv
    TIDB_HOST='{host}'  # 例: gateway01.ap-northeast-1.prod.aws.tidbcloud.com
    TIDB_PORT='4000'
    TIDB_USER='{user}'  # 例: xxxxxx.root
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    CA_PATH='{ssl_ca}'  # 例: /etc/ssl/certs/ca-certificates.crt (Debian / Ubuntu / Arch)
    ```

    接続ダイアログから取得した接続パラメータでプレースホルダ `{}` を置き換えてください。

    TiDB Serverlessは安全な接続が必要です。`mysqlclient`の`ssl_mode`はデフォルトで`PREFERRED`に設定されているため、手動で`CA_PATH`を指定する必要はありません。空のままにしてください。ただし、特別な理由がある場合は、さまざまな操作システムに対する証明書パスを取得するために [TLS connections to TiDB Serverless](https://docs.pingcap.com/tidbcloud/secure-connections-to-serverless-clusters) を参照することができます。

7. `.env`ファイルを保存します。

</div>
<div label="TiDB Dedicated">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、対象のクラスタ名をクリックして概要ページに移動します。

2. 右上隅にある**Connect**をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere**をクリックして、その後 **Download TiDB cluster CA** をクリックしてCA証明書をダウンロードします。

    接続文字列の取得方法の詳細については、[TiDB Dedicated standard connection](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4. 以下のコマンドを実行して `.env.example` をコピーし、`.env` として名前を変更します:

    ```shell
    cp .env.example .env
    ```

5. 対応する接続文字列を`.env`ファイルにコピーして貼り付けます。例を以下に示します:

    ```dotenv
    TIDB_HOST='{host}'  # 例: tidb.xxxx.clusters.tidb-cloud.com
    TIDB_PORT='4000'
    TIDB_USER='{user}'  # 例: root
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    CA_PATH='{your-downloaded-ca-path}'
    ```

    接続ダイアログから取得した接続パラメータでプレースホルダ `{}` を置き換えてください。`CA_PATH`には前の手順でダウンロードした証明書パスを構成してください。

6. `.env`ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1. 以下のコマンドを実行して `.env.example` をコピーし、`.env` として名前を変更します:

    ```shell
    cp .env.example .env
    ```

2. 対応する接続文字列を`.env`ファイルにコピーして貼り付けます。例を以下に示します:

    ```dotenv
    TIDB_HOST='{tidb_server_host}'
    TIDB_PORT='4000'
    TIDB_USER='root'
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    ```

    プレースホルダ `{}` を置き換えてください。TiDBをローカルで実行している場合、デフォルトのホストアドレスは `127.0.0.1`、パスワードは空です。

3. `.env`ファイルを保存します。

</div>
</SimpleTab>

### ステップ4: データベースの初期化

プロジェクトのルートディレクトリで、以下のコマンドを実行してデータベースを初期化します:

```shell
python manage.py migrate
```

### ステップ5: サンプルアプリケーションを実行

1. 開発モードでアプリケーションを実行します:

    ```shell
    python manage.py runserver
    ```

    アプリケーションはデフォルトでポート `8000` で実行されます。別のポートを使用する場合は、コマンドにポート番号を追加できます。以下は例です:

    ```shell
    python manage.py runserver 8080
    ```

2. アプリケーションにアクセスするには、ブラウザを開き、 `http://localhost:8000/` に移動します。サンプルアプリケーションで以下の操作ができます:

    - 新しいプレイヤーを作成します。
    - 複数のプレイヤーを一括作成します。
    - すべてのプレイヤーを表示します。
    - プレイヤーを更新します。
    - プレイヤーを削除します。
    - 2人のプレイヤー間で商品を取引します。

## コードスニペットのサンプル

以下のサンプルコードスニペットを参照して、独自のアプリケーション開発を完成させることができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-python-django-quickstart](https://github.com/tidb-samples/tidb-python-django-quickstart) リポジトリを参照してください。

### TiDBへの接続

ファイル `sample_project/settings.py` に、以下の構成を追加します：

```python
DATABASES = {
    "default": {
        "ENGINE": "django_tidb",
        "HOST": ${tidb_host},
        "PORT": ${tidb_port},
        "USER": ${tidb_user},
        "PASSWORD": ${tidb_password},
        "NAME": ${tidb_db_name},
        "OPTIONS": {
            "charset": "utf8mb4",
        },
    }
}

TIDB_CA_PATH = ${ca_path}
if TIDB_CA_PATH:
    DATABASES["default"]["OPTIONS"]["ssl_mode"] = "VERIFY_IDENTITY"
    DATABASES["default"]["OPTIONS"]["ssl"] = {
        "ca": TIDB_CA_PATH,
    }
```

`${tidb_host}`, `${tidb_port}`, `${tidb_user}`, `${tidb_password}`, `${tidb_db_name}`, および `${ca_path}` を、TiDBクラスターの実際の値に置き換える必要があります。

### データモデルの定義

```python
from django.db import models

class Player(models.Model):
    name = models.CharField(max_length=32, blank=False, null=False)
    coins = models.IntegerField(default=100)
    goods = models.IntegerField(default=1)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

詳細については、[Django モデル](https://docs.djangoproject.com/en/dev/topics/db/models/)を参照してください。

### データの挿入

```python
# オブジェクトの単純な挿入
player = Player.objects.create(name="player1", coins=100, goods=1)

# 複数のオブジェクトの一括挿入
Player.objects.bulk_create([
    Player(name="player1", coins=100, goods=1),
    Player(name="player2", coins=200, goods=2),
    Player(name="player3", coins=300, goods=3),
])
```

詳細については、[データの挿入](/develop/dev-guide-insert-data.md)を参照してください。

### データのクエリ

```python
# オブジェクトの取得
player = Player.objects.get(name="player1")

# 複数のオブジェクトの取得
filtered_players = Player.objects.filter(name="player1")

# すべてのオブジェクトの取得
all_players = Player.objects.all()
```

詳細については、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データの更新

```python
# オブジェクトの更新
player = Player.objects.get(name="player1")
player.coins = 200
player.save()

# 複数のオブジェクトの更新
Player.objects.filter(coins=100).update(coins=200)
```

詳細については、[データの更新](/develop/dev-guide-update-data.md)を参照してください。

### データの削除

```python
# オブジェクトの削除
player = Player.objects.get(name="player1")
player.delete()

# 複数のオブジェクトの削除
Player.objects.filter(coins=100).delete()
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md)を参照してください。

## 次のステップ

- [Django のドキュメント](https://www.djangoproject.com/)から Django のさらなる利用法を学んでください。
- [開発者ガイド](/develop/dev-guide-overview.md)の章を通じて、TiDB アプリケーション開発のベストプラクティスを学んでください。たとえば、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[シングルテーブルの読み取り](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および [SQL パフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
- プロフェッショナルな[TiDB 開発者コース](https://www.pingcap.com/education/)を学び、試験に合格して[TiDB 認定](https://www.pingcap.com/education/certification/)を取得してください。

## ヘルプが必要ですか？

<CustomContent platform="tidb">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](/support.md)してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。

</CustomContent>