---
title: RailsフレームワークとActiveRecord ORMを使用してTiDBに接続する
summary: Railsフレームワークを使用してTiDBに接続する方法について学びます。このチュートリアルでは、TiDBとRailsフレームワーク、ActiveRecord ORMを使用して動作するRubyサンプルコードスニペットが提供されます。
---

# RailsフレームワークとActiveRecord ORMを使用してTiDBに接続

TiDBは、MySQL互換のデータベースです。[Rails](https://github.com/rails/rails)は、Rubyで書かれた人気のあるWebアプリケーションフレームワークであり、[ActiveRecord ORM](https://github.com/rails/rails/tree/main/activerecord)は、Railsのオブジェクト関係マッピングです。

このチュートリアルでは、TiDBとRailsを使用して次のタスクを実行する方法を学ぶことができます:

- 環境をセットアップする。
- Railsを使用してTiDBクラスタに接続する。
- アプリケーションをビルドして実行する。オプションで、ActiveRecord ORMを使用して基本的なCRUD操作の [サンプルコードスニペット](#sample-code-snippets) を見つけることができます。

> **注意:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、TiDB Self-Hostedと連携します。

## 前提条件

このチュートリアルを完了するには、次のものが必要です:

- マシンにインストールされた [Ruby](https://www.ruby-lang.org/en/) >= 3.0
- マシンにインストールされた [Bundler](https://bundler.io/)
- マシンにインストールされた [Git](https://git-scm.com/downloads)
- 実行中のTiDBクラスタ

**TiDBクラスタを持っていない場合は、以下のように作成できます:**

<CustomContent platform="tidb">

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md) に従ってTiDB Cloudクラスタを作成します。
- [ローカルテスト TiDBクラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster) もしくは [プロダクション TiDBクラスタのデプロイ](/production-deployment-using-tiup.md) に従ってローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md) に従ってTiDB Cloudクラスタを作成します。
- [ローカルテスト TiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster) もしくは [プロダクション TiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup) に従ってローカルクラスタを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する

このセクションでは、サンプルアプリケーションコードを実行してTiDBに接続する方法を示します。

### ステップ 1: サンプルアプリのリポジトリをクローンする

次のコマンドをターミナルウィンドウで実行して、サンプルコードのリポジトリをクローンします:

```shell
git clone https://github.com/tidb-samples/tidb-ruby-rails-quickstart.git
cd tidb-ruby-rails-quickstart
```

### ステップ 2: 依存関係をインストールする

次のコマンドを実行して、サンプルアプリのために `mysql2` と `dotenv` を含む必要なパッケージをインストールします:

```shell
bundle install
```

<details>
<summary><b>既存のプロジェクトの依存関係をインストールする</b></summary>

既存のプロジェクトについては、次のコマンドを実行してパッケージをインストールします:

```shell
bundle add mysql2 dotenv
```

</details>

### ステップ 3: 接続情報を設定する

選択したTiDBデプロイオプションに応じてTiDBクラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックしてその概要ページに移動します。

2. 右上隅にある **Connect** をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログで、 **Connect With** ドロップダウンリストから `Rails` を選択し、 **Endpoint Type** を `Public` のままデフォルト設定のままにします。

4. パスワードをまだ設定していない場合は、 **Create password** をクリックしてランダムなパスワードを生成します。

5. 次のコマンドを実行して `.env.example` をコピーして `.env` にリネームします:

    ```shell
    cp .env.example .env
    ```

6. `.env` ファイルを編集し、`DATABASE_URL` 環境変数を次のように設定し、接続ダイアログから接続文字列を変数値としてコピーします。

    ```dotenv
    DATABASE_URL=mysql2://<user>:<password>@<host>:<port>/<database_name>?ssl_mode=verify_identity
    ```

   > **注意**
   >
   > TiDB Serverlessを使用する場合は、パブリックエンドポイントを使用する場合に `ssl_mode=verify_identity` クエリパラメータでTLS接続を有効にする必要があります。

7. `.env` ファイルを保存します。

</div>
<div label="TiDB Dedicated">

1. [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックしてその概要ページに移動します。

2. 右上隅にある **Connect** をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere** をクリックしてから **Download TiDB cluster CA** をクリックしてCA証明書をダウンロードします。

   接続文字列の取得方法の詳細については、[TiDB Dedicated標準接続](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection) を参照してください。

4. 次のコマンドを実行して `.env.example` をコピーして `.env` にリネームします:

    ```shell
    cp .env.example .env
    ```

5. `.env` ファイルを編集し、`DATABASE_URL` 環境変数を次のように設定し、接続ダイアログから接続文字列を変数値としてコピーし、`sslca` クエリパラメータを接続ダイアログからダウンロードしたCA証明書のファイルパスに設定します:

    ```dotenv
    DATABASE_URL=mysql2://<user>:<password>@<host>:<port>/<database>?ssl_mode=verify_identity&sslca=/path/to/ca.pem
    ```

   > **注意**
   >
   > TiDB Dedicatedにパブリックエンドポイントを使用して接続する場合は、TLS接続を有効にすることを推奨します。
   >
   > TLS接続を有効にするには、`ssl_mode` クエリパラメータの値を `verify_identity` に変更し、 `sslca` の値を接続ダイアログからダウンロードしたCA証明書のファイルパスに変更してください。

6. `.env` ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して `.env.example` をコピーして `.env` にリネームします:

    ```shell
    cp .env.example .env
    ```

2. `.env` ファイルを編集し、`DATABASE_URL` 環境変数を次のように設定し、`<user>`、`<password>`、`<host>`、`<port>`、`<database>` を自分のTiDB接続情報に置き換えます:

    ```dotenv
    DATABASE_URL=mysql2://<user>:<password>@<host>:<port>/<database>
    ```

   TiDBをローカルで実行している場合、デフォルトのホストアドレスは `127.0.0.1` です。パスワードは空です。

3. `.env` ファイルを保存します。

</div>
</SimpleTab>

### ステップ 4: コードを実行して結果を確認する

1. データベースとテーブルを作成します:

    ```shell
    bundle exec rails db:create
    bundle exec rails db:migrate
    ```

2. サンプルデータをシードします:

    ```shell
    bundle exec rails db:seed
    ```

3. 次のコマンドを実行してサンプルコードを実行します:

    ```shell
    bundle exec rails runner ./quickstart.rb
    ```

接続が成功した場合、コンソールにはTiDBクラスタのバージョンが次のように表示されます:

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

以下のサンプルコードスニペットを参照して、独自のアプリケーション開発を完成させることができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-ruby-rails-quickstart](https://github.com/tidb-samples/tidb-ruby-rails-quickstart) リポジトリを確認してください。

### 接続オプションを使用してTiDBに接続する

`config/database.yml` の次のコードは、環境変数で定義されたオプションを使用してTiDBに接続します:

```yml
default: &default
  adapter: mysql2
  encoding: utf8mb4
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  url: <%= ENV["DATABASE_URL"] %>

development:
  <<: *default

test:
  <<: *default
  database: quickstart_test

production:
  <<: *default
```

> **注意**
>
> For TiDB Serverless, TLS connection **MUST** be enabled via setting the `ssl_mode` query parameter to `verify_identity` in `DATABASE_URL` when using public endpoint, but you **don't** have to specify an SSL CA certificate via `DATABASE_URL`, because mysql2 gem will search for existing CA certificates in a particular order until a file is discovered.

### データの挿入

次のクエリは、2つのフィールドを持つ単一のPlayerを作成し、作成された `Player` オブジェクトを返します。

```ruby
new_player = Player.create!(coins: 100, goods: 100)
```

詳細については、[データの挿入](/develop/dev-guide-insert-data.md) を参照してください。

### データのクエリ

次のクエリは、特定のIDのプレイヤーのレコードを返します。

```ruby
player = Player.find_by(id: new_player.id)
```

詳細については、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md) を参照してください。

### データの更新

次のクエリは、`Player` オブジェクトを更新します。

```ruby
player.update(coins: 50, goods: 50)
```

詳細については、[データの更新](/develop/dev-guide-update-data.md) を参照してください。

### データの削除

次のクエリは、`Player` オブジェクトを削除します。

```ruby
player.destroy
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md) を参照してください。

## ベストプラクティス

デフォルトでは、TiDBに接続するためにActiveRecord ORMで使用されるmysql2 gemは、ファイルが見つかるまで特定の順序で既存のCA証明書を検索します。

1. /etc/ssl/certs/ca-certificates.crt # Debian / Ubuntu / Gentoo / Arch / Slackware
2. /etc/pki/tls/certs/ca-bundle.crt # RedHat / Fedora / CentOS / Mageia / Vercel / Netlify
3. /etc/ssl/ca-bundle.pem # OpenSUSE
4. /etc/ssl/cert.pem # MacOS / Alpine (docker container)

CA証明書のパスを手動で指定することも可能ですが、このアプローチは異なるマシンや環境が異なる場所にCA証明書を格納しているため、マルチ環境の展開シナリオでは大きな不便を引き起こす可能性があります。そのため、異なる環境での柔軟性とデプロイの容易さのために、`sslca` を `nil` に設定することが推奨されます。

## 次のステップ

- [ActiveRecord のドキュメント](https://guides.rubyonrails.org/active_record_basics.html) からActiveRecord ORMのさらなる使用法を学びます。
- [開発者ガイド](/develop/dev-guide-overview.md) の章によるTiDBアプリケーション開発のベストプラクティスを学びます。たとえば、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
- プロの[TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格した後に[TiDB認定](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？

[Discord](https://discord.gg/vYU9h56kAX) チャンネルで質問してください。