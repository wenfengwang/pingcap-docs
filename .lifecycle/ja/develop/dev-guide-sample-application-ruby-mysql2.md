---
title: mysql2を使用してTiDBに接続する
summary: Rubyのmysql2を使用してTiDBに接続する方法を学びます。このチュートリアルでは、mysql2 gemを使用してTiDBで動作するRubyのサンプルコードスニペットが提供されます。
---

# mysql2を使用してTiDBに接続する

TiDBはMySQL互換のデータベースであり、[mysql2](https://github.com/brianmario/mysql2)はRuby向けの最も人気のあるMySQLドライバの1つです。

このチュートリアルでは、TiDBとmysql2を使用して次のタスクを達成する方法を学ぶことができます。

- 環境をセットアップする。
- mysql2を使用してTiDBクラスタに接続する。
- アプリケーションをビルドして実行する。オプションで、基本的なCRUD操作の[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。

> **注意:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、TiDB Self-Hostedと共に動作します。

## 必要なもの

このチュートリアルを完了するには、次のものが必要です。

- マシンにインストールされた[Ruby](https://www.ruby-lang.org/en/) >= 3.0
- マシンにインストールされた[Bundler](https://bundler.io/)
- マシンにインストールされた[Git](https://git-scm.com/downloads)
- 動作中のTiDBクラスタ

**TiDBクラスタをまだ持っていない場合は、次のようにして作成できます:**

<CustomContent platform="tidb">

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)を参照して、独自のTiDB Cloudクラスタを作成してください。
- [ローカルでのテストTiDBクラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番TiDBクラスタのデプロイ](/production-deployment-using-tiup.md)を作成するには、こちらを参照してください。

</CustomContent>
<CustomContent platform="tidb-cloud">

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)を参照して、独自のTiDB Cloudクラスタを作成してください。
- [ローカルでのテストTiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番TiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)を作成するには、こちらを参照してください。

</CustomContent>

## サンプルアプリを実行してTiDBに接続

このセクションでは、サンプルアプリケーションコードを実行してTiDBに接続する方法を示します。

### ステップ1: サンプルアプリケーションリポジトリをクローンする

次のコマンドをターミナルウィンドウで実行して、サンプルコードリポジトリをクローンします。

```shell
git clone https://github.com/tidb-samples/tidb-ruby-mysql2-quickstart.git
cd tidb-ruby-mysql2-quickstart
```

### ステップ2: 依存関係をインストールする

次のコマンドを実行して、サンプルアプリケーションに必要なパッケージ（`mysql2`および`dotenv`を含む）をインストールします。

```shell
bundle install
```

<details>
<summary><b>既存のプロジェクトの依存関係をインストールする</b></summary>

既存のプロジェクトの場合、次のコマンドを実行してパッケージをインストールします。

```shell
bundle add mysql2 dotenv
```

</details>

### ステップ3: 接続情報を構成する

TiDBデプロイメントオプションに応じてTiDBクラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックしてその概要ページに移動します。

2. 右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの構成が動作環境と一致することを確認します。

   - **Endpoint Type** が `Public` に設定されていること。
   - **Connect With** が `General` に設定されていること。
   - **Operating System** がアプリケーションを実行するオペレーティングシステムと一致していること。

4. まだパスワードを設定していない場合は、 **Create password** をクリックしてランダムなパスワードを生成します。

5. 次のコマンドを実行して、`.env.example`をコピーして`.env`にリネームします:

    ```shell
    cp .env.example .env
    ```

6. `.env`ファイルを編集し、次のように環境変数を設定し、接続ダイアログから取得した接続パラメータを対応するプレースホルダー`<>`で置き換えます:

    ```dotenv
    DATABASE_HOST=<host>
    DATABASE_PORT=4000
    DATABASE_USER=<user>
    DATABASE_PASSWORD=<password>
    DATABASE_NAME=test
    DATABASE_ENABLE_SSL=true
    ```

   > **注意**
   >
   > TiDB Serverlessでは、パブリックエンドポイントを使用する場合は、`DATABASE_ENABLE_SSL`を介してTLS接続を有効にする必要があります。

7. `.env`ファイルを保存します。

</div>
<div label="TiDB Dedicated">

1. [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックしてその概要ページに移動します。

2. 右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere** をクリックし、**Download TiDB cluster CA** をクリックしてCA証明書をダウンロードします。

   パブリックエンドポイントを使用してTiDB Dedicatedクラスタに接続する場合の接続文字列の詳細については、[TiDB Dedicated standard connection](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection) を参照してください。

4. 次のコマンドを実行して、`.env.example`をコピーして`.env`にリネームします:

    ```shell
    cp .env.example .env
    ```

5. `.env`ファイルを編集し、次のように環境変数を設定し、接続ダイアログから取得した接続パラメータを対応するプレースホルダー`<>`で置き換えます:

    ```dotenv
    DATABASE_HOST=<host>
    DATABASE_PORT=4000
    DATABASE_USER=<user>
    DATABASE_PASSWORD=<password>
    DATABASE_NAME=test
    DATABASE_ENABLE_SSL=true
    DATABASE_SSL_CA=<downloaded_ssl_ca_path>
    ```

   > **注意**
   >
   > TiDB Dedicatedクラスタにパブリックエンドポイントを使用する場合は、TLS接続を有効にすることをお勧めします。

6. `.env`ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して、`.env.example`をコピーして`.env`にリネームします:

    ```shell
    cp .env.example .env
    ```

2. `.env`ファイルを編集し、次のように環境変数を設定し、TiDBの接続情報を対応するプレースホルダー`<>`で置き換えます:

    ```dotenv
    DATABASE_HOST=<host>
    DATABASE_PORT=4000
    DATABASE_USER=<user>
    DATABASE_PASSWORD=<password>
    DATABASE_NAME=test
    ```

   ローカルでTiDBを実行している場合、デフォルトのホストアドレスは `127.0.0.1` であり、パスワードは空です。

3.  `.env`ファイルを保存します。

</div>
</SimpleTab>

### ステップ4: コードを実行して結果を確認する

次のコマンドを実行して、サンプルコードを実行します:

```shell
ruby app.rb
```

接続が成功すると、コンソールに次のようにTiDBクラスタのバージョンが出力されます:

```
🔌 Connected to TiDB cluster! (TiDB version: 8.0.11-TiDB-v7.4.0)
⏳ Loading sample game data...
✅ Loaded sample game data.

🆕 Created a new player with ID 12.
ℹ️ Got Player 12: Player { id: 12, coins: 100, goods: 100 }
🔢 Added 50 coins and 50 goods to player 12, updated 1 row.
🚮 Deleted 1 player data.
```

## サンプルコードスニペット

次のサンプルコードスニペットを参照して、独自のアプリケーション開発を完了させることができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-ruby-mysql2-quickstart](https://github.com/tidb-samples/tidb-ruby-mysql2-quickstart) リポジトリをご確認ください。

### 接続オプションを使用してTiDBに接続

次のコードは環境変数で定義されたオプションを使用してTiDBに接続します:

```ruby
require 'dotenv/load'
require 'mysql2'
Dotenv.load # .envファイルから環境変数を読み込む

options = {
  host: ENV['DATABASE_HOST'] || '127.0.0.1',
  port: ENV['DATABASE_PORT'] || 4000,
  username: ENV['DATABASE_USER'] || 'root',
  password: ENV['DATABASE_PASSWORD'] || '',
  database: ENV['DATABASE_NAME'] || 'test'
}
options.merge(ssl_mode: :verify_identity) unless ENV['DATABASE_ENABLE_SSL'] == 'false'
options.merge(sslca: ENV['DATABASE_SSL_CA']) if ENV['DATABASE_SSL_CA']
client = Mysql2::Client.new(options)
```

> **注意**
>
> TiDB Serverlessを使用する場合は、パブリックエンドポイントを使用する場合は、`DATABASE_ENABLE_SSL`を介してTLS接続を有効にする必要がありますが、`DATABASE_SSL_CA`を介してSSL CA証明書を指定する必要はありません。なぜなら、mysql2 gemは特定の順番で既存のCA証明書を検索し、ファイルが見つかるまでに検索を行うからです。

### データの挿入

以下のクエリは、2つのフィールドを持つ単一のプレイヤーを作成し、`last_insert_id`を返します：

```ruby
def create_player(client, coins, goods)
  result = client.query(
    "INSERT INTO players (coins, goods) VALUES (#{coins}, #{goods});"
  )
  client.last_id
end
```

詳細については、[データの挿入](/develop/dev-guide-insert-data.md)を参照してください。

### データのクエリ

以下のクエリは、特定のIDのプレイヤーのレコードを返します：

```ruby
def get_player_by_id(client, id)
  result = client.query(
    "SELECT id, coins, goods FROM players WHERE id = #{id};"
  )
  result.first
end
```

詳細については、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データの更新

以下のクエリは、特定のIDのプレイヤーのレコードを更新します：

```ruby
def update_player(client, player_id, inc_coins, inc_goods)
  result = client.query(
    "UPDATE players SET coins = coins + #{inc_coins}, goods = goods + #{inc_goods} WHERE id = #{player_id};"
  )
  client.affected_rows
end
```

詳細については、[データの更新](/develop/dev-guide-update-data.md)を参照してください。

### データの削除

以下のクエリは、特定のプレイヤーのレコードを削除します：

```ruby
def delete_player_by_id(client, id)
  result = client.query(
    "DELETE FROM players WHERE id = #{id};"
  )
  client.affected_rows
end
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md)を参照してください。

## ベストプラクティス

デフォルトでは、mysql2 gem はファイルが見つかるまで特定の順序で既存のCA証明書を検索します。

1. Debian、Ubuntu、Gentoo、Arch、またはSlackware の場合は `/etc/ssl/certs/ca-certificates.crt`。
2. RedHat、Fedora、CentOS、Mageia、Vercel、またはNetlify の場合は `/etc/pki/tls/certs/ca-bundle.crt`。
3. OpenSUSE の場合は `/etc/ssl/ca-bundle.pem`。
4. macOS または Alpine（dockerコンテナ）の場合は `/etc/ssl/cert.pem`。

CA証明書のパスを手動で指定することも可能ですが、異なるマシンや環境が異なる場所にCA証明書を保存しているため、複数の環境での展開においてかなりの不便が生じる可能性があります。そのため、`sslca`を`nil`に設定して柔軟性と異なる環境での展開の容易さを推奨します。

## 次のステップ

- [mysql2のドキュメント](https://github.com/brianmario/mysql2#readme)でmysql2ドライバのさらなる使用法を学ぶ。
- [デベロッパーガイド](/develop/dev-guide-overview.md)の章で、TiDBアプリケーション開発のベストプラクティスを学ぶ。たとえば、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
- プロの[TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格して[TiDB認定](https://www.pingcap.com/education/certification/)を取得する。

## お困りですか？

[Discord](https://discord.gg/vYU9h56kAX)チャンネルで質問してください。