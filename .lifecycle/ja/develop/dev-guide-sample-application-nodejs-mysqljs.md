---
title: mysql.jsを使用してTiDBに接続する
summary: mysql.jsを使用してTiDBに接続する方法を学びます。このチュートリアルでは、mysql.jsを使用してTiDBと連携するNode.jsのサンプルコードスニペットが提供されます。
---

# mysql.jsを使用してTiDBに接続する

TiDBはMySQL互換のデータベースであり、[mysql.js](https://github.com/mysqljs/mysql)ドライバはMySQLプロトコルを実装した純粋なNode.js JavaScriptクライアントです。

このチュートリアルでは、次のタスクを実行するためにTiDBとmysql.jsドライバを使用する方法を学ぶことができます。

- 環境のセットアップ
- mysql.jsドライバを使用してTiDBクラスタに接続
- アプリケーションの構築と実行。任意で、基本的なCRUD操作の[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。

> **注意:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、TiDB Self-Hostedと連携します。

## 前提条件

このチュートリアルを完了するには、次のものが必要です。

- マシンにインストールされた[Node.js](https://nodejs.org/en) >= 16.x
- マシンにインストールされた[Git](https://git-scm.com/downloads)
- 実行中のTiDBクラスタ

**TiDBクラスタを持っていない場合は、以下のように作成できます。**

<CustomContent platform="tidb">

- (推奨) [TiDB Serverlessクラスタを作成](/develop/dev-guide-build-cluster-in-cloud.md)して、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番TiDBクラスタのデプロイ](/production-deployment-using-tiup.md)を作成するために従ってください。

</CustomContent>
<CustomContent platform="tidb-cloud">

- (推奨) [TiDB Serverlessクラスタを作成](https://docs.pingcap.com/tidb/stable/dev-guide-build-cluster-in-cloud)して、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番TiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)を作成するために従ってください。

</CustomContent>

## TiDBに接続するためのサンプルアプリケーションを実行する

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を示します。

### ステップ1: サンプルアプリケーションリポジトリをクローンする

以下のコマンドをターミナルウィンドウで実行して、サンプルコードリポジトリをクローンします。

```shell
git clone https://github.com/tidb-samples/tidb-nodejs-mysqljs-quickstart.git
cd tidb-nodejs-mysqljs-quickstart
```

### ステップ2: 依存関係をインストールする

次のコマンドを実行して、サンプルアプリケーションに必要なパッケージ（`mysql`および`dotenv`を含む）をインストールします。

```shell
npm install
```

<details>
<summary><b>既存のプロジェクトに依存関係をインストールする</b></summary>

既存のプロジェクトの場合は、次のコマンドを実行してパッケージをインストールします。

```shell
npm install mysql dotenv --save
```

</details>

### ステップ3: 接続情報を構成する

TiDBのデプロイメントオプションに応じてTiDBクラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅にある**Connect**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの構成が動作環境に一致していることを確認します。

    - **Endpoint Type**が`Public`に設定されていること。
    - **Connect With**が`General`に設定されていること。
    - **Operating System**がアプリケーションを実行するオペレーティングシステムに一致していること。

4. まだパスワードを設定していない場合は、ランダムなパスワードを生成するために**Create password**をクリックします。

5. 次のコマンドを実行して`.env.example`をコピーし、`.env`にリネームします。

    ```shell
    cp .env.example .env
    ```

6. `.env`ファイルを編集し、次のように環境変数を設定し、接続ダイアログの対応するプレースホルダ `{}` を接続パラメータで置き換えます。

    ```dotenv
    TIDB_HOST={host}
    TIDB_PORT=4000
    TIDB_USER={user}
    TIDB_PASSWORD={password}
    TIDB_DATABASE=test
    TIDB_ENABLE_SSL=true
    ```

    > **注意**
    >
    > TiDB Serverlessの場合、`TIDB_ENABLE_SSL`を使用してTLS接続を有効にする必要があります。

7. `.env`ファイルを保存します。

</div>
<div label="TiDB Dedicated">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅にある**Connect**をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere**をクリックし、次に**Download TiDB cluster CA**をクリックしてCA証明書をダウンロードします。

    接続文字列の取得方法の詳細については、[TiDB Dedicated標準接続](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4. 次のコマンドを実行して`.env.example`をコピーし、`.env`にリネームします。

    ```shell
    cp .env.example .env
    ```

5. `.env`ファイルを編集し、次のように環境変数を設定し、接続ダイアログの対応するプレースホルダ `{}` を接続パラメータで置き換えます。

    ```dotenv
    TIDB_HOST={host}
    TIDB_PORT=4000
    TIDB_USER={user}
    TIDB_PASSWORD={password}
    TIDB_DATABASE=test
    TIDB_ENABLE_SSL=true
    TIDB_CA_PATH={downloaded_ssl_ca_path}
    ```

    > **注意**
    >
    > TiDB Dedicatedに接続する場合は、TLS接続を有効にすることが推奨されます。

6. `.env`ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して`.env.example`をコピーし、`.env`にリネームします。

    ```shell
    cp .env.example .env
    ```

2. `.env`ファイルを編集し、クラスタの接続パラメータに応じて次のようにプレースホルダ `{}` を置き換えます。

    ```dotenv
    TIDB_HOST={host}
    TIDB_PORT=4000
    TIDB_USER=root
    TIDB_PASSWORD={password}
    TIDB_DATABASE=test
    ```

    TiDBをローカルで実行している場合、デフォルトのホストアドレスは`127.0.0.1`であり、パスワードは空です。

3. `.env`ファイルを保存します。

</div>
</SimpleTab>

### ステップ4: コードを実行し、結果を確認する

以下のコマンドを実行してサンプルコードを実行します。

```shell
npm start
```

接続が成功すると、コンソールにTiDBクラスタのバージョンが表示されます。

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

以下のサンプルコードスニペットを参照して、独自のアプリケーション開発を完了させることができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-nodejs-mysqljs-quickstart](https://github.com/tidb-samples/tidb-nodejs-mysqljs-quickstart)リポジトリを確認してください。

### 接続オプションを使用して接続する

次のコードは、環境変数で定義されたオプションを使用してTiDBに接続します。

```javascript
// ステップ1. 'mysql'および'dotenv'パッケージをインポートします。
import { createConnection } from "mysql";
import dotenv from "dotenv";
import * as fs from "fs";

// ステップ2. .envファイルから環境変数をロードします。
dotenv.config();

// ステップ3. TiDBクラスタに接続します。
const options = {
    host: process.env.TIDB_HOST || '127.0.0.1',
    port: process.env.TIDB_PORT || 4000,
    user: process.env.TIDB_USER || 'root',
    password: process.env.TIDB_PASSWORD || '',
    database: process.env.TIDB_DATABASE || 'test',
    ssl: process.env.TIDB_ENABLE_SSL === 'true' ? {
        minVersion: 'TLSv1.2',
        ca: process.env.TIDB_CA_PATH ? fs.readFileSync(process.env.TIDB_CA_PATH) : undefined
    } : null,
}
const conn = createConnection(options);
```
//ステップ4. SQL操作を実行します...

//ステップ5. 接続を閉じます。
conn.end();
```

>**注意**
>
>TiDB Serverlessを使用する場合、公開エンドポイントを使用する際には`TIDB_ENABLE_SSL`を介したTLS接続を必ず有効にしてください。ただし、Node.jsはデフォルトで組み込みの[Mozilla CA証明書](https://wiki.mozilla.org/CA/Included_Certificates)を使用するため、TiDB Serverlessによって信頼されるようになるSSL CA証明書を`TIDB_CA_PATH`で指定する必要はありません。

###データの挿入

次のクエリは単一の`Player`レコードを作成し、新しく作成されたレコードのIDを返します:

```javascript
conn.query('INSERT INTO players (coins, goods) VALUES (?, ?);', [100, 100], (err, ok) => {
   if (err) {
       console.error(err);
   } else {
       console.log(ok.insertId);
   }
});
```

詳細については、[データの挿入](/develop/dev-guide-insert-data.md)を参照してください。

###データのクエリ

次のクエリはID`1`の単一の`Player`レコードを返します:

```javascript
conn.query('SELECT id, coins, goods FROM players WHERE id = ?;', [1], (err, rows) => {
   if (err) {
      console.error(err);
   } else {
      console.log(rows[0]);
   }
});
```

詳細については、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

###データの更新

次のクエリはID`1`の`Player`に`50`コインと`50`商品を追加します:

```javascript
conn.query(
   'UPDATE players SET coins = coins + ?, goods = goods + ? WHERE id = ?;',
   [50, 50, 1],
   (err, ok) => {
      if (err) {
         console.error(err);
      } else {
          console.log(ok.affectedRows);
      }
   }
);
```

詳細については、[データの更新](/develop/dev-guide-update-data.md)を参照してください。

###データの削除

次のクエリはID`1`の`Player`レコードを削除します:

```javascript
conn.query('DELETE FROM players WHERE id = ?;', [1], (err, ok) => {
    if (err) {
        reject(err);
    } else {
        resolve(ok.affectedRows);
    }
});
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md)を参照してください。

##便利なノート

- [接続プール](https://github.com/mysqljs/mysql#pooling-connections)を使用することで、頻繁に接続を確立および破棄することによるパフォーマンスのオーバーヘッドを削減できます。
- SQLインジェクション攻撃を防ぐために、SQLを実行する前に[クエリ値をエスケープ](https://github.com/mysqljs/mysql#escaping-query-values)することをお勧めします。

    >**注意**
    >
    >`mysqljs/mysql`パッケージはまだプリペアドステートメントをサポートしていません。クライアント側で値のエスケープのみを行います（関連する問題: [mysqljs/mysql#274](https://github.com/mysqljs/mysql/issues/274)）。
    >
    >SQLインジェクションを防止したり、バッチの挿入/更新の効率を改善するためにこの機能を使用したい場合は、代わりに[mysql2](https://github.com/sidorares/node-mysql2)パッケージを使用することをお勧めします。

- 複数の複雑なSQL文がないシナリオで開発効率を向上させるためにORMフレームワークを使用することをお勧めします。例: [Sequelize](https://sequelize.org/)、[TypeORM](https://typeorm.io/)、および[Prisma](/develop/dev-guide-sample-application-nodejs-prisma.md)。
- データベース内のビッグナンバー（`BIGINT`および`DECIMAL`列）を扱う際には、`supportBigNumbers: true`オプションを有効にすることをお勧めします。

##次のステップ

- [mysql.jsのドキュメント](https://github.com/mysqljs/mysql#readme)からmysql.jsドライバのさらなる使用法を学びます。
- [開発者ガイド](/develop/dev-guide-overview.md)の章を通じて、TiDBアプリケーション開発のベストプラクティスを学びます。[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)など。
- プロの[TiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格して[TiDB認定資格](https://www.pingcap.com/education/certification/)を取得します。

##サポートが必要ですか？

<CustomContent platform="tidb">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](/support.md)してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。

</CustomContent>