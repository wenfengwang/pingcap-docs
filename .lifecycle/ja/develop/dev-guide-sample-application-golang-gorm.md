---
title: GORMを使用してTiDBに接続する
summary: GORMを使用してTiDBに接続する方法を学びます。このチュートリアルでは、GORMを使用してTiDBと連携するためのGolangのサンプルコードスニペットが提供されます。
---

# GORMを使用してTiDBに接続

TiDBはMySQL互換のデータベースであり、[GORM](https://gorm.io/index.html)はGolang向けの人気のあるオープンソースのORMフレームワークです。GORMは、`AUTO_RANDOM`などのTiDBの機能に適応し、[TiDBをデフォルトのデータベースオプションとしてサポート](https://gorm.io/docs/connecting_to_the_database.html#TiDB)しています。

このチュートリアルでは、以下のタスクを達成するためにTiDBとGORMを使用する方法を学ぶことができます:

- 環境の設定。
- GORMを使用してTiDBクラスターに接続する。
- アプリケーションのビルドと実行。オプションとして、[サンプルコードスニペット](#sample-code-snippets)を使用して基本的なCRUD操作を行うこともできます。

> **注意:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、TiDB Self-Hostedと連携します。

## 前提条件

このチュートリアルを完了するには、以下が必要です:

- [Go](https://go.dev/) **1.20** 以上。
- [Git](https://git-scm.com/downloads)。
- TiDBクラスター。

<CustomContent platform="tidb">

**TiDBクラスターをお持ちでない場合は、以下のように作成できます:**

- (推奨) [TiDB Serverlessクラスターの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って独自のTiDB Cloudクラスターを作成します。
- [ローカルテストTiDBクラスターの展開](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番用TiDBクラスターの展開](/production-deployment-using-tiup.md)に従って、ローカルクラスターを作成します。

</CustomContent>

<CustomContent platform="tidb-cloud">

**TiDBクラスターをお持ちでない場合は、以下のように作成できます:**

- (推奨) [TiDB Serverlessクラスターの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って独自のTiDB Cloudクラスターを作成します。
- [ローカルテストTiDBクラスターの展開](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番用TiDBクラスターの展開](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスターを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を示します。

### ステップ 1: サンプルアプリのリポジトリをクローンする

ターミナルウィンドウで以下のコマンドを実行して、サンプルコードリポジトリをクローンします:

```shell
git clone https://github.com/tidb-samples/tidb-golang-gorm-quickstart.git
cd tidb-golang-gorm-quickstart
```

### ステップ 2: 接続情報を構成する

TiDBの展開オプションに応じてTiDBクラスターに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、対象のクラスターの名前をクリックして概要ページに移動します。

2. 右上隅にある**Connect**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの設定が、運用環境に一致することを確認します。

    - **Endpoint Type**が`Public`に設定されていること。
    - **Connect With**が`General`に設定されていること。
    - **Operating System**が環境に一致していること。

    > **ヒント:**
    >
    > プログラムがWindows Subsystem for Linux (WSL)で実行されている場合は、対応するLinuxディストリビューションに切り替えます。

4. ランダムなパスワードを作成するには、**Create password**をクリックします。

    > **ヒント:**
    >
    > 以前にパスワードを作成した場合は、元のパスワードを使用するか、新しいものを生成するために**Reset password**をクリックします。

5. 次のコマンドを実行して`.env.example`をコピーし、`.env`に名前を変更します:

    ```shell
    cp .env.example .env
    ```

6. 対応する接続文字列を`.env`ファイルにコピーして貼り付けます。例として、以下のようになります:

    ```dotenv
    TIDB_HOST='{host}'  # 例: gateway01.ap-northeast-1.prod.aws.tidbcloud.com
    TIDB_PORT='4000'
    TIDB_USER='{user}'  # 例: xxxxxx.root
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    USE_SSL='true'
    ```

    この接続ダイアログから取得した接続パラメータでプレースホルダー`{}`を置き換えてください。

    TiDB Serverlessでは安全な接続が必要です。そのため、`USE_SSL`の値を`true`に設定する必要があります。

7. `.env`ファイルを保存します。

</div>
<div label="TiDB Dedicated">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、対象のクラスターの名前をクリックして概要ページに移動します。

2. 右上隅にある**Connect**をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere**をクリックし、次に**Download TiDB cluster CA**をクリックしてCA証明書をダウンロードします。

    接続文字列の取得方法の詳細については、[TiDB Dedicated standard connection](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4. 次のコマンドを実行して`.env.example`をコピーし、`.env`に名前を変更します:

    ```shell
    cp .env.example .env
    ```

5. 対応する接続文字列を`.env`ファイルにコピーして貼り付けます。例として、以下のようになります:

    ```dotenv
    TIDB_HOST='{host}'  # 例: tidb.xxxx.clusters.tidb-cloud.com
    TIDB_PORT='4000'
    TIDB_USER='{user}'  # 例: root
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    USE_SSL='false'
    ```

    この接続ダイアログから取得した接続パラメータでプレースホルダー`{}`を置き換えてください。

6. `.env`ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して`.env.example`をコピーし、`.env`に名前を変更します:

    ```shell
    cp .env.example .env
    ```

2. 対応する接続文字列を`.env`ファイルにコピーして貼り付けます。例として、以下のようになります:

    ```dotenv
    TIDB_HOST='{host}'
    TIDB_PORT='4000'
    TIDB_USER='root'
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    USE_SSL='false'
    ```

    プレースホルダー`{}`を接続パラメータに置き換え、`USE_SSL`を`false`に設定してください。TiDBをローカルで実行している場合、デフォルトのホストアドレスは`127.0.0.1`で、パスワードは空です。

3. `.env`ファイルを保存します。

</div>
</SimpleTab>

### ステップ 3: コードを実行して結果を確認する

1. 次のコマンドを実行してサンプルコードを実行します:

    ```shell
    make
    ```

2. 出力が一致するかどうかは、[Expected-Output.txt](https://github.com/tidb-samples/tidb-golang-gorm-quickstart/blob/main/Expected-Output.txt)を確認してください。

## サンプルコードスニペット

以下のサンプルコードスニペットを参照して、独自のアプリケーション開発を完了させることができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-golang-gorm-quickstart](https://github.com/tidb-samples/tidb-golang-gorm-quickstart)リポジトリを確認してください。

### TiDBに接続

```golang
func createDB() *gorm.DB {
    dsn := fmt.Sprintf("%s:%s@tcp(%s:%s)/%s?charset=utf8mb4&tls=%s",
        ${tidb_user}, ${tidb_password}, ${tidb_host}, ${tidb_port}, ${tidb_db_name}, ${use_ssl})

    db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{
        Logger: logger.Default.LogMode(logger.Info),
    })
    if err != nil {
        panic(err)
    }

    return db
}
```

この関数を使用する際には、`${tidb_host}`、`${tidb_port}`、`${tidb_user}`、`${tidb_password}`、`${tidb_db_name}`をTiDBクラスターの実際の値で置き換える必要があります。TiDB Serverlessでは安全な接続が必要です。そのため、`${use_ssl}`の値を`true`に設定する必要があります。

### データの挿入

```golang
db.Create(&Player{ID: "id", Coins: 1, Goods: 1})
```

詳細については、[データの挿入](/develop/dev-guide-insert-data.md)を参照してください。

### データのクエリ

```golang
var queryPlayer Player
db.Find(&queryPlayer, "id = ?", "id")
```

詳細については、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データの更新

```golang
```json
db.Save(&Player{ID: "id", Coins: 100, Goods: 1})
```

詳細については、[データの更新](/develop/dev-guide-update-data.md) を参照してください。

### データの削除

```golang
db.Delete(&Player{ID: "id"})
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md) を参照してください。

## 次のステップ

- [GORM のドキュメント](https://gorm.io/docs/index.html) および [GORM ドキュメントの TiDB セクション](https://gorm.io/docs/connecting_to_the_database.html#TiDB) から GORM のより詳細な使用法を学びます。
- [開発者ガイド](/develop/dev-guide-overview.md) の章を通じて、TiDB アプリケーション開発のベストプラクティスを学びます。たとえば、[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルの読み取り](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQL パフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md) 。
- [プロの TiDB 開発者コース](https://www.pingcap.com/education/) を通じて学び、試験に合格した後に[TiDB 認定](https://www.pingcap.com/education/certification/)を取得します。

## 必要なヘルプ？

<CustomContent platform="tidb">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc) での質問、または [サポートチケットの作成](/support.md) をしてください。

</CustomContent>

<CustomContent platform="tidb-cloud">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc) での質問、または[サポートチケットの作成](https://support.pingcap.com/)をしてください。

</CustomContent>
```