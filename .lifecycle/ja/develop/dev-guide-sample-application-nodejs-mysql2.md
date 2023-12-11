---
title: node-mysql2でTiDBに接続する
summary: node-mysql2を使用してTiDBに接続する方法を学びます。このチュートリアルではTiDBとnode-mysql2を使用して動作するNode.jsのサンプルコードスニペットが提供されます。
---

# node-mysql2でTiDBに接続する

TiDBはMySQL互換のデータベースであり、[node-mysql2](https://github.com/sidorares/node-mysql2)はNode.js向けの高速な[mysqljs/mysql](https://github.com/mysqljs/mysql)互換のMySQLドライバです。

このチュートリアルでは、TiDBとnode-mysql2を使用して次のタスクを実行する方法を学習できます。

- 環境をセットアップする。
- node-mysql2を使用してTiDBクラスタに接続する。
- アプリケーションをビルドおよび実行する。オプションで、基本的なCRUD操作の[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。

> **注意:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、およびTiDB Self-Hostedと動作します。

## 前提条件

このチュートリアルを完了するには、次のものが必要です:

- マシンにインストールされている[Node.js](https://nodejs.org/en) >= 16.x。
- マシンにインストールされている[Git](https://git-scm.com/downloads)。
- 実行中のTiDBクラスタ。

**TiDBクラスタがない場合は、以下のように作成できます:**

<CustomContent platform="tidb">

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDBクラウドクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[本番TiDBクラスタのデプロイ](/production-deployment-using-tiup.md)に従って、ローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って、独自のTiDBクラウドクラスタを作成します。
- [ローカルテストTiDBクラスタをデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[本番TiDBクラスタをデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従って、ローカルクラスタを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を示します。

### ステップ1: サンプルアプリのリポジトリをクローンする

ターミナルウィンドウで次のコマンドを実行して、サンプルコードリポジトリをクローンします:

```shell
git clone https://github.com/tidb-samples/tidb-nodejs-mysql2-quickstart.git
cd tidb-nodejs-mysql2-quickstart
```

### ステップ2: 依存関係をインストールする

次のコマンドを実行して、サンプルアプリケーションに必要なパッケージ（`mysql2`および`dotenv`を含む）をインストールします:

```shell
npm install
```

<details>
<summary><b>既存のプロジェクトに依存関係をインストールする</b></summary>

既存のプロジェクトの場合は、次のコマンドを実行してパッケージをインストールします:

```shell
npm install mysql2 dotenv --save
```

</details>

### ステップ3: 接続情報を構成する

TiDBのデプロイオプションに応じてTiDBクラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、対象のクラスタ名をクリックして概要ページに移動します。

2. 右上隅の**接続**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの構成が動作環境に一致することを確認します。

    - **エンドポイントタイプ**が`Public`に設定されていること。
    - **接続先**が`General`に設定されていること。
    - **オペレーティングシステム**がアプリケーションが実行されているオペレーティングシステムに一致していること。

4. パスワードをまだ設定していない場合は、「パスワードの作成」をクリックしてランダムなパスワードを生成します。

5. 次のコマンドを実行して`.env.example`をコピーし、`.env`に名前を変更します:

    ```shell
    cp .env.example .env
    ```

6. `.env`ファイルを編集し、次のように環境変数を設定し、接続ダイアログの接続パラメータで対応するプレースホルダー`{}`を置き換えます:

    ```dotenv
    TIDB_HOST={ホスト}
    TIDB_PORT=4000
    TIDB_USER={ユーザー}
    TIDB_PASSWORD={パスワード}
    TIDB_DATABASE=test
    TIDB_ENABLE_SSL=true
    ```

    > **注意**
    >
    > TiDB Serverlessを使用する場合、パブリックエンドポイントを使用する際には`TIDB_ENABLE_SSL`を介してTLS接続を有効にする必要があります。

7. `.env`ファイルを保存します。

</div>
<div label="TiDB Dedicated">

1. [**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、対象のクラスタ名をクリックして概要ページに移動します。

2. 右上隅の**接続**をクリックします。接続ダイアログが表示されます。

3. **どこからでもアクセスを許可**をクリックし、その後**TiDBクラスタCAをダウンロード**してCA証明書をダウンロードします。

    接続文字列の入手方法の詳細については、[TiDB Dedicated標準接続](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4. 次のコマンドを実行して`.env.example`をコピーし、`.env`に名前を変更します:

    ```shell
    cp .env.example .env
    ```

5. `.env`ファイルを編集し、次のように環境変数を設定し、接続ダイアログの接続パラメータで対応するプレースホルダー`{}`を置き換えます:

    ```dotenv
    TIDB_HOST={ホスト}
    TIDB_PORT=4000
    TIDB_USER={ユーザー}
    TIDB_PASSWORD={パスワード}
    TIDB_DATABASE=test
    TIDB_ENABLE_SSL=true
    TIDB_CA_PATH={ダウンロードされたSSL CAパス}
    ```

    > **注意**
    >
    > TiDB Dedicatedに公開エンドポイントを使用する場合、TLS接続を有効にすることを推奨します。
    >
    > TLS接続を有効にするには、`TIDB_ENABLE_SSL`を`true`に変更し、接続ダイアログからダウンロードしたCA証明書のファイルパスを`TIDB_CA_PATH`に指定します。

6. `.env`ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して`.env.example`をコピーし、`.env`に名前を変更します:

    ```shell
    cp .env.example .env
    ```

2. `.env`ファイルを編集し、次のように環境変数を設定し、接続ダイアログの接続パラメータで対応するプレースホルダー`{}`を置き換えます:

    ```dotenv
    TIDB_HOST={ホスト}
    TIDB_PORT=4000
    TIDB_USER=root
    TIDB_PASSWORD={パスワード}
    TIDB_DATABASE=test
    ```

   TiDBをローカルで実行している場合、デフォルトのホストアドレスは`127.0.0.1`であり、パスワードは空です。

3. `.env`ファイルを保存します。

</div>
</SimpleTab>

### ステップ4: コードを実行して結果を確認する

次のコマンドを実行してサンプルコードを実行します:

```shell
npm start
```

接続に成功すると、コンソールにTiDBクラスタのバージョンが出力されます:

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

以下のサンプルコードスニペットを参照して独自のアプリケーション開発を完了させることができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-nodejs-mysql2-quickstart](https://github.com/tidb-samples/tidb-nodejs-mysql2-quickstart)リポジトリをチェックしてください。

### 接続オプションを使用して接続する

次のコードは、環境変数で定義されたオプションを使用してTiDBに接続を確立します:

```javascript
// ステップ1. 'mysql'および'dotenv'パッケージをインポートします。
import { createConnection } from "mysql2/promise";
import dotenv from "dotenv";
import * as fs from "fs";

// ステップ2. .envファイルから環境変数を読み込みます。
dotenv.config();

async function main() {
   // ステップ3. TiDBクラスタに接続します。
   const options = {
      host: process.env.TIDB_HOST || '127.0.0.1',
      port: process.env.TIDB_PORT || 4000,
      user: process.env.TIDB_USER || 'root',
      password: process.env.TIDB_PASSWORD || '',
      database: process.env.TIDB_DATABASE || 'test',
      ssl: process.env.TIDB_ENABLE_SSL === 'true' ? {
         minVersion: 'TLSv1.2',
```javascript
async function main() {
  const options = {
    host: process.env.TIDB_HOST,
    port: process.env.TIDB_PORT,
    user: process.env.TIDB_USER,
    password: process.env.TIDB_PASSWORD,
    database: process.env.TIDB_DATABASE,
    ssl: process.env.TIDB_ENABLE_SSL ? {
      ca: process.env.TIDB_CA_PATH ? fs.readFileSync(process.env.TIDB_CA_PATH) : undefined
    } : null,
  }
  const conn = await createConnection(options);

  // Step 4. Perform some SQL operations...

  // Step 5. Close the connection.
  await conn.end();
}

void main();
```

> **Note**
>
> TiDB Serverlessを使用する場合は、パブリックエンドポイントを使用するときに`TIDB_ENABLE_SSL`経由でTLS接続を有効にする**必要があります**。ただし、Node.jsはデフォルトで組み込みの[Mozilla CA証明書](https://wiki.mozilla.org/CA/Included_Certificates)を使用するため、TiDB Serverlessに信頼されているため、`TIDB_CA_PATH`を介してSSL CA証明書を指定する必要は**ありません**。

### データの挿入

次のクエリは単一の`Player`レコードを作成し、`ResultSetHeader`オブジェクトを返します:

```javascript
const [rsh] = await conn.query('INSERT INTO players (coins, goods) VALUES (?, ?);', [100, 100]);
console.log(rsh.insertId);
```

詳細については、[データの挿入](/develop/dev-guide-insert-data.md)を参照してください。

### データのクエリ

次のクエリは、ID `1`の単一の`Player`レコードを返します:

```javascript
const [rows] = await conn.query('SELECT id, coins, goods FROM players WHERE id = ?;', [1]);
console.log(rows[0]);
```

詳細については、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データの更新

次のクエリは、ID `1`の`Player`に`50`コインと`50`商品を追加します:

```javascript
const [rsh] = await conn.query(
    'UPDATE players SET coins = coins + ?, goods = goods + ? WHERE id = ?;',
    [50, 50, 1]
);
console.log(rsh.affectedRows);
```

詳細については、[データの更新](/develop/dev-guide-update-data.md)を参照してください。

### データの削除

次のクエリは、ID `1`の`Player`レコードを削除します:

```javascript
const [rsh] = await conn.query('DELETE FROM players WHERE id = ?;', [1]);
console.log(rsh.affectedRows);
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md)を参照してください。

## 便利なメモ

- データベース接続を管理するために[connection pools](https://github.com/sidorares/node-mysql2#using-connection-pools)を使用すると、頻繁に接続を確立および破棄することによるパフォーマンスのオーバーヘッドを減らすことができます。
- SQLインジェクションを防ぐために、[prepared statements](https://github.com/sidorares/node-mysql2#using-prepared-statements)を使用することをお勧めします。
- 複雑なSQL文が多く含まれないシナリオでは、[Sequelize](https://sequelize.org/)、[TypeORM](https://typeorm.io/)、または[Prisma](https://www.prisma.io/)のようなORMフレームワークを使用することで開発効率を大幅に向上させることができます。
- データベースにおける大きな数値（`BIGINT`および`DECIMAL`列）を扱う場合は、`supportBigNumbers: true`オプションを有効にすることをお勧めします。
- ネットワークの問題によるソケットエラー`read ECONNRESET`を回避するために、`enableKeepAlive: true`オプションを有効にすることをお勧めします。（関連する問題：[sidorares/node-mysql2#683](https://github.com/sidorares/node-mysql2/issues/683)）

## 次のステップ

- [node-mysql2のドキュメント](https://github.com/sidorares/node-mysql2#readme)からnode-mysql2ドライバーの使用法についてさらに学習します。
- [開発者ガイド](/develop/dev-guide-overview.md)の章を通じて、TiDBアプリケーション開発のベストプラクティスを学びます。[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、[SQLのパフォーマンス最適化](/develop/dev-guide-optimize-sql-overview.md)など。
- [プロのTiDB開発者コース](https://www.pingcap.com/education/)を通じて学習し、試験に合格した後に[TiDBの認定資格](https://www.pingcap.com/education/certification/)を取得します。

## お問い合わせ

<CustomContent platform="tidb">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問をしたり、[サポートチケットを作成](/support.md)することができます。

</CustomContent>

<CustomContent platform="tidb-cloud">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問をしたり、[サポートチケットを作成](https://support.pingcap.com/)することができます。

</CustomContent>
```