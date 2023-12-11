---
title: Next.jsでmysql2を使用してTiDBに接続する
summary: この記事では、Next.jsでTiDBとmysql2を使用してCRUDアプリケーションを構築する方法を説明し、簡単なコードスニペットの例を提供します。

# Next.jsでmysql2を使用してTiDBに接続する

TiDBはMySQL互換のデータベースであり、[mysql2](https://github.com/sidorares/node-mysql2)はNode.js向けの人気のあるオープンソースドライバです。

このチュートリアルでは、次のタスクを実行するためにNext.jsでTiDBとmysql2を使用する方法を学ぶことができます。

- 環境をセットアップします。
- mysql2を使用してTiDBクラスタに接続します。
- アプリケーションをビルドして実行します。オプションで、基本的なCRUD操作の[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。

> **注意**
>
> このチュートリアルは、TiDB ServerlessとTiDB Self-Hostedの両方で動作します。

## 前提条件

このチュートリアルを完了するには、次のものが必要です。

- [Node.js **18**](https://nodejs.org/en/download/) 以上
- [Git](https://git-scm.com/downloads)
- TiDBクラスタ

<CustomContent platform="tidb">

**TiDBクラスタをお持ちでない場合は、次のように作成できます:**

- (推奨) [TiDB Serverlessクラスタを作成](/develop/dev-guide-build-cluster-in-cloud.md)して、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタをデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)したり、[本番環境のTiDBクラスタをデプロイ](/production-deployment-using-tiup.md)したりして、ローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスタをお持ちでない場合は、次のように作成できます:**

- (推奨) [TiDB Serverlessクラスタを作成](/develop/dev-guide-build-cluster-in-cloud.md)して、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタをデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)したり、[本番環境のTiDBクラスタをデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)したりして、ローカルクラスタを作成します。

</CustomContent>

## TiDBに接続するためのサンプルアプリを実行する

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を示します。

> **注意**
>
> 完全なコードスニペットと実行手順については、[tidb-nextjs-vercel-quickstart](https://github.com/tidb-samples/tidb-nextjs-vercel-quickstart) GitHubリポジトリを参照してください。

### ステップ1: サンプルアプリのリポジトリをクローンする

次のコマンドをターミナルウィンドウで実行して、サンプルコードリポジトリをクローンします。

```bash
git clone git@github.com:tidb-samples/tidb-nextjs-vercel-quickstart.git
cd tidb-nextjs-vercel-quickstart
```

### ステップ2: 依存関係をインストールする

次のコマンドを実行して、サンプルアプリケーションに必要なパッケージ（mysql2を含む）をインストールします。

```bash
npm install
```

### ステップ3: 接続情報を構成する

TiDBの展開オプションに応じてTiDBクラスタに接続します。

<SimpleTab>

<div label="TiDB Serverless">

1. [**Clusters**ページ](https://tidbcloud.com/console/clusters)に移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの構成が操作環境と一致することを確認します。

    - **Endpoint Type**が`Public`に設定されていること
    - **Connect With**が`General`に設定されていること
    - **Operating System**が環境と一致していること

    > **注意**
    >
    > Node.jsアプリケーションでは、TLS（SSL）接続を確立する際、Node.jsはデフォルトで組み込みの[Mozilla CA証明書](https://wiki.mozilla.org/CA/Included_Certificates)を使用するため、SSL CA証明書を提供する必要はありません。

4. ランダムなパスワードを作成するには、**Create password**をクリックします。

    > **ヒント**
    >
    > 以前にパスワードを作成した場合、元のパスワードを使用するか、新しいパスワードを生成するには**Reset password**をクリックします。

5. 次のコマンドを実行して`.env.example`をコピーして`.env`にリネームします：

    ```bash
    # Linux
    cp .env.example .env
    ```

    ```powershell
    # Windows
    Copy-Item ".env.example" -Destination ".env"
    ```

6. 対応する接続文字列を`.env`ファイルにコピーして貼り付けます。次の例は結果です：

    ```bash
    TIDB_HOST='{gateway-region}.aws.tidbcloud.com'
    TIDB_PORT='4000'
    TIDB_USER='{prefix}.root'
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    ```

    `{}`内のプレースホルダを接続ダイアログで取得した値で置き換えます。

7. `.env`ファイルを保存します。

</div>

<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して`.env.example`をコピーして`.env`にリネームします：

    ```bash
    # Linux
    cp .env.example .env
    ```

    ```powershell
    # Windows
    Copy-Item ".env.example" -Destination ".env"
    ```

2. 対応する接続文字列を`.env`ファイルにコピーして貼り付けます。次の例は結果です：

    ```bash
    TIDB_HOST='{tidb_server_host}'
    TIDB_PORT='4000'
    TIDB_USER='root'
    TIDB_PASSWORD='{password}'
    TIDB_DB_NAME='test'
    ```

    `{}`内のプレースホルダを**Connect**ウィンドウで取得した値で置き換えます。TiDBをローカルで実行している場合、デフォルトのホストアドレスは`127.0.0.1`で、パスワードは空です。

3. `.env`ファイルを保存します。

</div>

</SimpleTab>

### ステップ4: コードを実行し結果を確認する

1. アプリケーションを起動します：

   ```bash
   npm run dev
   ```

2. ブラウザを開いて`http://localhost:3000`にアクセスします（実際のポート番号はターミナルで確認し、デフォルトは`3000`です）。

3. **RUN SQL**をクリックしてサンプルコードを実行します。

4. ターミナルで出力を確認します。出力が以下のようなものであれば、接続が成功しています：

   ```json
   {
     "results": [
       {
         "Hello World": "Hello World"
       }
     ]
   }
   ```

## サンプルコードスニペット

次のサンプルコードスニペットを参照して、独自のアプリケーション開発を完了させることができます。

完全なサンプルコードとその実行方法については、[tidb-nextjs-vercel-quickstart](https://github.com/tidb-samples/tidb-nextjs-vercel-quickstart)リポジトリをご覧ください。

### TiDBに接続する

次のコードは、環境変数で定義されたオプションを使用してTiDBに接続を確立します：

```javascript
// src/lib/tidb.js
import mysql from 'mysql2';

let pool = null;

export function connect() {
  return mysql.createPool({
    host: process.env.TIDB_HOST, // TiDBホスト、例: {gateway-region}.aws.tidbcloud.com
    port: process.env.TIDB_PORT || 4000, // TiDBポート、デフォルト: 4000
    user: process.env.TIDB_USER, // TiDBユーザー、例: {prefix}.root
    password: process.env.TIDB_PASSWORD, // TiDBユーザーのパスワード
    database: process.env.TIDB_DATABASE || 'test', // TiDBデータベース名、デフォルト: test
    ssl: {
      minVersion: 'TLSv1.2',
      rejectUnauthorized: true,
    },
    connectionLimit: 1, // connectionLimitを「1」に設定することで、サーバーレス関数環境でリソースの使用量を最適化し、コストを削減し、接続の安定性を確保し、透過的なスケーラビリティを実現します。
    maxIdle: 1, // max idle connections、デフォルト値は`connectionLimit`と同じです
    enableKeepAlive: true,
  });
}

export function getPool() {
  if (!pool) {
    pool = createPool();
  }
  return pool;
}
```

### データの挿入

次のクエリは、単一の`Player`レコードを作成し、`ResultSetHeader`オブジェクトを返します：

```javascript
const [rsh] = await pool.query('INSERT INTO players (coins, goods) VALUES (?, ?);', [100, 100]);
console.log(rsh.insertId);
```

詳細は [データの挿入](/develop/dev-guide-insert-data.md) を参照してください。

### データのクエリ

次のクエリは、IDが`1`の単一の`Player`レコードを返します：

```javascript
const [rows] = await pool.query('SELECT id, coins, goods FROM players WHERE id = ?;', [1]);
console.log(rows[0]);
```

詳細は [データの取得](/develop/dev-guide-get-data-from-single-table.md) を参照してください。

### データの更新

次のクエリは、ID `1` の `Player` に `50` のコインと `50` の商品を追加します：

```javascript
const [rsh] = await pool.query(
    'UPDATE players SET coins = coins + ?, goods = goods + ? WHERE id = ?;',
    [50, 50, 1]
);
console.log(rsh.affectedRows);
```

詳細については、[データの更新](/develop/dev-guide-update-data.md)を参照してください。

### データの削除

次のクエリは、ID `1` の `Player` レコードを削除します：

```javascript
const [rsh] = await pool.query('DELETE FROM players WHERE id = ?;', [1]);
console.log(rsh.affectedRows);
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md)を参照してください。

## 便利なメモ

- [接続プール](https://github.com/sidorares/node-mysql2#using-connection-pools)を使用してデータベース接続を管理すると、頻繁に接続を確立および破棄することによる性能のオーバーヘッドを軽減できます。
- SQLインジェクションを防ぐためには、[プリペアドステートメント](https://github.com/sidorares/node-mysql2#using-prepared-statements)を使用することが推奨されています。
- 複雑なSQLステートメントがあまり関与していないシナリオでは、[Sequelize](https://sequelize.org/)、[TypeORM](https://typeorm.io/)、または [Prisma](https://www.prisma.io/)などのORMフレームワークを使用すると、開発効率が大幅に向上します。

## 次のステップ

- ORMとNext.jsを使用して複雑なアプリケーションを構築する詳細については、[Bookshop Demo](https://github.com/pingcap/tidb-prisma-vercel-demo)を参照してください。
- [node-mysql2のドキュメント](https://github.com/sidorares/node-mysql2/tree/master/documentation/en)からnode-mysql2ドライバの使用法についてさらに学びます。
- [開発者ガイド](/develop/dev-guide-overview.md)の章を通じて、TiDBアプリケーション開発のベストプラクティスについて学んでください。[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルからのデータ取得](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、および[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
- [プロフェッショナルなTiDB開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格して[TiDBの認定資格](https://www.pingcap.com/education/certification/)を取得できます。

## ヘルプが必要ですか？

<CustomContent platform="tidb">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問したり、[サポートチケットを作成](/support.md)することができます。

</CustomContent>

<CustomContent platform="tidb-cloud">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問したり、[サポートチケットを作成](https://support.pingcap.com/)することができます。

</CustomContent>