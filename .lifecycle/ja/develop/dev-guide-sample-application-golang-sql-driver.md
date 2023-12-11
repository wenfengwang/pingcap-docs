---
title: Go-MySQL-Driverを使用してTiDBに接続する
summary: Go-MySQL-Driverを使用してTiDBに接続する方法を学びます。このチュートリアルでは、Go-MySQL-Driverを使用してTiDBと連携するGolangサンプルコードの断片が示されます。
aliases: ['/tidb/dev/dev-guide-outdated-for-go-sql-driver-mysql','/tidb/dev/dev-guide-outdated-for-gorm','/tidb/dev/dev-guide-sample-application-golang']
---

# Go-MySQL-Driverを使用してTiDBに接続する

TiDBはMySQL互換のデータベースであり、[Go-MySQL-Driver](https://github.com/go-sql-driver/mysql)は[database/sql](https://pkg.go.dev/database/sql)インターフェースのためのMySQL実装です。

このチュートリアルでは、TiDBとGo-MySQL-Driverを使用して次のタスクを達成する方法を学ぶことができます：

- 環境をセットアップする。
- Go-MySQL-Driverを使用してTiDBクラスターに接続する。
- アプリケーションをビルドして実行する。オプションで、基本的なCRUD操作の[サンプルコード断片](#サンプルコード断片)を見つけることができます。

> **注意:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、およびTiDB Self-Hostedと共に動作します。

## 前提条件

このチュートリアルを完了するには、以下が必要です：

- [Go](https://go.dev/) **1.20** 以上。
- [Git](https://git-scm.com/downloads)。
- TiDBクラスター。

<CustomContent platform="tidb">

**TiDBクラスターがない場合、以下のように作成することができます：**

- (推奨) [TiDB Serverlessクラスターの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスターを作成します。
- [ローカルテストTiDBクラスターの展開](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番TiDBクラスターの展開](/production-deployment-using-tiup.md)に従って、ローカルクラスターを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスターがない場合、以下のように作成することができます：**

- (推奨) [TiDB Serverlessクラスターの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDB Cloudクラスターを作成します。
- [ローカルテストTiDBクラスターの展開](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番TiDBクラスターの展開](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスターを作成します。
</CustomContent>

## サンプルアプリを実行してTiDBに接続

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を示します。

### ステップ1：サンプルアプリリポジトリをクローンする

次のコマンドをターミナルウィンドウで実行して、サンプルコードリポジトリをクローンします：

```shell
git clone https://github.com/tidb-samples/tidb-golang-sql-driver-quickstart.git
cd tidb-golang-sql-driver-quickstart
```

### ステップ2：接続情報を構成する

TiDBのデプロイオプションに応じてTiDBクラスターに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**クラスター**](https://tidbcloud.com/console/clusters)ページに移動し、対象クラスターの名前をクリックして概要ページに移動します。

2. 右上隅の**接続**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの構成が実行環境に一致していることを確認します。

    - **エンドポイントタイプ**が `Public` に設定されていること
    - **接続方法**が `General` に設定されていること
    - **オペレーティングシステム**が環境に一致していること

    > **Tip:**
    >
    > プログラムがWindows Subsystem for Linux（WSL）で実行されている場合は、該当するLinuxディストリビューションに切り替えてください。

4. ランダムなパスワードを作成するために**パスワードを作成**をクリックします。

    > **Tip:**
    >
    > 以前にパスワードを作成している場合は、元のパスワードを使用するか、新しいパスワードを生成するために**パスワードをリセット**をクリックします。

5. 次のコマンドを実行して`.env.example`をコピーし、`.env`にリネームします：

    ```shell
    cp .env.example .env
    ```

6. `.env`ファイルに対応する接続文字列をコピーし、貼り付けます。例として以下のようになります：

    ```dotenv
    TIDB_HOST='{host}'  # 例: gateway01.ap-northeast-1.prod.aws.tidbcloud.com
    TIDB_PORT='4000'
    TIDB_USER='{user}'  # 例: xxxxxx.root
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    USE_SSL='true'
    ```

    プレースホルダー `{}` を接続ダイアログから取得した接続パラメータで置き換えてください。

    TiDB Serverlessでは安全な接続が必要なため、`USE_SSL`の値を`true`に設定する必要があります。

7. `.env`ファイルを保存します。

</div>
<div label="TiDB Dedicated">

1. [**クラスター**](https://tidbcloud.com/console/clusters)ページに移動し、対象クラスターの名前をクリックして概要ページに移動します。

2. 右上隅の**接続**をクリックします。接続ダイアログが表示されます。

3. **どこからでもアクセスを許可**をクリックし、次に**TiDBクラスターCAをダウンロード**をクリックしてCA証明書をダウンロードします。

    接続文字列の取得方法の詳細については、[TiDB Dedicated標準接続](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4. 次のコマンドを実行して`.env.example`をコピーし、`.env`にリネームします：

    ```shell
    cp .env.example .env
    ```

5. `.env`ファイルに対応する接続文字列をコピーし、貼り付けます。例として以下のようになります：

    ```dotenv
    TIDB_HOST='{host}'  # 例: tidb.xxxx.clusters.tidb-cloud.com
    TIDB_PORT='4000'
    TIDB_USER='{user}'  # 例: root
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    USE_SSL='false'
    ```

    プレースホルダー `{}` を接続ダイアログから取得した接続パラメータで置き換えてください。

6. `.env`ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して`.env.example`をコピーし、`.env`にリネームします：

    ```shell
    cp .env.example .env
    ```

2. `.env`ファイルに対応する接続文字列をコピーし、貼り付けます。例として以下のようになります：

    ```dotenv
    TIDB_HOST='{host}'
    TIDB_PORT='4000'
    TIDB_USER='root'
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    USE_SSL='false'
    ```

    プレースホルダー `{}` を接続パラメータに置き換え、`USE_SSL`を`false`に設定してください。TiDBをローカルで実行している場合、デフォルトのホストアドレスは`127.0.0.1`であり、パスワードは空です。

3. `.env`ファイルを保存します。

</div>
</SimpleTab>

### ステップ3：コードを実行し、結果を確認する

1. 次のコマンドを実行してサンプルコードを実行します：

    ```shell
    make
    ```

2. 出力が一致するかどうかを確認するために[Expected-Output.txt](https://github.com/tidb-samples/tidb-golang-sql-driver-quickstart/blob/main/Expected-Output.txt)を確認します。

## サンプルコード断片

以下のサンプルコード断片を参照して、独自のアプリケーション開発を完了させることができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-golang-sql-driver-quickstart](https://github.com/tidb-samples/tidb-golang-sql-driver-quickstart)リポジトリを確認してください。

### TiDBに接続

```golang
func openDB(driverName string, runnable func(db *sql.DB)) {
    dsn := fmt.Sprintf("%s:%s@tcp(%s:%s)/%s?charset=utf8mb4&tls=%s",
        ${tidb_user}, ${tidb_password}, ${tidb_host}, ${tidb_port}, ${tidb_db_name}, ${use_ssl})
    db, err := sql.Open(driverName, dsn)
    if err != nil {
        panic(err)
    }
    defer db.Close()

    runnable(db)
}
```

この関数を使用する際には、`${tidb_host}`、`${tidb_port}`、`${tidb_user}`、`${tidb_password}`、および`${tidb_db_name}`をTiDBクラスターの実際の値に置き換える必要があります。TiDB Serverlessでは安全な接続が必要なため、`${use_ssl}`の値を`true`に設定する必要があります。

### データの挿入

```golang
openDB("mysql", func(db *sql.DB) {
    insertSQL = "INSERT INTO player (id, coins, goods) VALUES (?, ?, ?)"
    _, err := db.Exec(insertSQL, "id", 1, 1)

    if err != nil {
        panic(err)
    }
})
```
より詳細な情報は、[データの挿入](/develop/dev-guide-insert-data.md)を参照してください。

### データのクエリ

```golang
openDB("mysql", func(db *sql.DB) {
    selectSQL = "SELECT id, coins, goods FROM player WHERE id = ?"
    rows, err := db.Query(selectSQL, "id")
    if err != nil {
        panic(err)
    }

    // この行は非常に重要です！
    defer rows.Close()

    id, coins, goods := "", 0, 0
    if rows.Next() {
        err = rows.Scan(&id, &coins, &goods)
        if err == nil {
            fmt.Printf("player id: %s, coins: %d, goods: %d\n", id, coins, goods)
        }
    }
})
```

より詳細な情報は、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データの更新

```golang
openDB("mysql", func(db *sql.DB) {
    updateSQL = "UPDATE player set goods = goods + ?, coins = coins + ? WHERE id = ?"
    _, err := db.Exec(updateSQL, 1, -1, "id")

    if err != nil {
        panic(err)
    }
})
```

より詳細な情報は、[データの更新](/develop/dev-guide-update-data.md)を参照してください。

### データの削除

```golang
openDB("mysql", func(db *sql.DB) {
    deleteSQL = "DELETE FROM player WHERE id=?"
    _, err := db.Exec(deleteSQL, "id")

    if err != nil {
        panic(err)
    }
})
```

より詳細な情報は、[データの削除](/develop/dev-guide-delete-data.md)を参照してください。

## 有用なノート

### ドライバーまたはORMフレームワークを使用しますか？

Golangのドライバーはデータベースへの低レベルなアクセスを提供しますが、開発者は次の作業が必要です：

- 手動でデータベース接続を確立し、解放する。
- データベーストランザクションを手動で管理する。
- データ行をデータオブジェクトに手動でマッピングする。

複雑なSQL文を書く必要がない限り、[ORM](https://en.wikipedia.org/w/index.php?title=Object-relational_mapping)フレームワーク（たとえば[GORM](/develop/dev-guide-sample-application-golang-gorm.md)）を使用することをお勧めします。これにより、次のことができます：

- 接続やトランザクションの管理における[ボイラープレートコード](https://en.wikipedia.org/wiki/Boilerplate_code)を削減します。
- 複数のSQL文の代わりにデータオブジェクトでデータを操作します。

## 次のステップ

- [Go-MySQL-Driverのドキュメント](https://github.com/go-sql-driver/mysql/blob/master/README.md)からGo-MySQL-Driverのさらなる使用法を学びます。
- [デベロッパーガイド](/develop/dev-guide-overview.md)の章を通じて、TiDBアプリケーション開発のベストプラクティスを学びます。例：[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルの読み取り](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
- [TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格して[TiDB認定資格](https://www.pingcap.com/education/certification/)を取得します。

## ヘルプが必要ですか？

<CustomContent platform="tidb">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](/support.md)してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。

</CustomContent>