---
title: AWS Lambdaファンクションでmysql2を使用してTiDBに接続する方法
summary: この記事では、TiDBおよびmysql2を使用してAWS LambdaファンクションでCRUDアプリケーションを構築する方法について説明し、シンプルなコードスニペットの例を提供します。
---

# AWS Lambdaファンクションでmysql2を使用してTiDBに接続する

TiDBはMySQL互換のデータベースであり、[AWS Lambdaファンクション](https://aws.amazon.com/lambda/)はコンピューティングサービスであり、[mysql2](https://github.com/sidorares/node-mysql2)はNode.js向けの人気のあるオープンソースドライバです。

このチュートリアルでは、TiDBとmysql2をAWS Lambdaファンクションで使用して以下のタスクを達成する方法を学ぶことができます。

- 環境のセットアップ。
- mysql2を使用してTiDBクラスタに接続する。
- アプリケーションのビルドおよび実行。オプションで、基本的なCRUD操作の[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。
- AWS Lambdaファンクションをデプロイする。

> **注意**
>
> このチュートリアルはTiDB ServerlessとTiDB Self-Hostedと互換です。

## 前提条件

このチュートリアルを完了するには、次のものが必要です：

- [Node.js **18**](https://nodejs.org/ja/download/) またはそれ以降。
- [Git](https://git-scm.com/downloads)。
- TiDBクラスタ。
- 管理者権限を持つ[AWSユーザ](https://docs.aws.amazon.com/ja_jp/IAM/latest/UserGuide/id_users.html)。
- [AWS CLI](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/cli-chap-install.html)
- [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html)

<CustomContent platform="tidb">

**TiDBクラスタをお持ちでない場合は、次の方法で作成できます：**

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md) に従って、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタをデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster) または [プロダクションTiDBクラスタをデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup) に従って、ローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスタをお持ちでない場合は、次の方法で作成できます：**

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md) に従って、独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタをデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster) または [プロダクションTiDBクラスタをデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup) に従って、ローカルクラスタを作成します。

</CustomContent>

AWSアカウントやユーザをお持ちでない場合は、[Lambdaのはじめに](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/getting-started.html)ガイドの手順に従って作成できます。

## サンプルアプリを実行してTiDBに接続する

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を示します。

> **注意**
>
> 完全なコードスニペットと実行手順については、[tidb-samples/tidb-aws-lambda-quickstart](https://github.com/tidb-samples/tidb-aws-lambda-quickstart) GitHubリポジトリを参照してください。

### ステップ1: サンプルアプリリポジトリをクローンする

ターミナルウィンドウで次のコマンドを実行して、サンプルコードリポジトリをクローンします:

```bash
git clone git@github.com:tidb-samples/tidb-aws-lambda-quickstart.git
cd tidb-aws-lambda-quickstart
```

### ステップ2: 依存関係をインストールする

次のコマンドを実行して、サンプルアプリケーションで必要なパッケージ（`mysql2`を含む）をインストールします:

```bash
npm install
```

### ステップ3: 接続情報を設定する

TiDBのデプロイオプションに応じてTiDBクラスタに接続します。

<SimpleTab>

<div label="TiDB Serverless">

1. [**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、対象のクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログボックスが表示されます。

3. 接続ダイアログの構成が使用している環境に一致することを確認します。

    - **Endpoint Type**は`Public`に設定されています
    - **Connect With**は`General`に設定されています
    - **Operating System**は環境に一致しています。

    > **注意**
    >
    > Node.jsアプリケーションでは、TLS（SSL）接続を確立する際にNode.jsがデフォルトで組み込みの[Mozilla CA証明書](https://wiki.mozilla.org/CA/Included_Certificates)を使用するため、SSL CA証明書を提供する必要はありません。

4. ランダムなパスワードを作成するために**Create password**をクリックします。

    > **ヒント**
    >
    > 以前にパスワードを生成した場合は、元のパスワードを使用するか、新しいパスワードを生成するために**Reset password**をクリックできます。

5. 対応する接続文字列を`env.json`にコピーして貼り付けます。以下は例です:

    ```json
    {
      "Parameters": {
        "TIDB_HOST": "{gateway-region}.aws.tidbcloud.com",
        "TIDB_PORT": "4000",
        "TIDB_USER": "{prefix}.root",
        "TIDB_PASSWORD": "{password}"
      }
    }
    ```

    `{}`内のプレースホルダを接続ダイアログで取得した値で置き換えます。

</div>

<div label="TiDB Self-Hosted">

対応する接続文字列を`env.json`にコピーして貼り付けます。以下は例です:

```json
{
  "Parameters": {
    "TIDB_HOST": "{tidb_server_host}",
    "TIDB_PORT": "4000",
    "TIDB_USER": "root",
    "TIDB_PASSWORD": "{password}"
  }
}
```

`{}`内のプレースホルダを**Connect**ウィンドウから取得した値で置き換えます。

</div>

</SimpleTab>

### ステップ4: コードを実行して結果を確認する

1. (前提条件) [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html)をインストールします。

2. バンドルをビルドします:

    ```bash
    npm run build
    ```

3. サンプルLambdaファンクションを呼び出します:

    ```bash
    sam local invoke --env-vars env.json -e events/event.json "tidbHelloWorldFunction"
    ```

4. ターミナルで出力を確認します。出力が以下と似ている場合、接続が成功しています:

    ```bash
    {"statusCode":200,"body":"{\"results\":[{\"Hello World\":\"Hello World\"}]}"}
    ```

接続が成功したことを確認したら、AWS Lambdaファンクションをデプロイするために[次のセクション](#deploy-the-aws-lambda-function)に続きます。

## AWS Lambdaファンクションをデプロイする

AWS Lambdaファンクションは[SAM CLIデプロイ](#sam-cli-deployment-recommended)または[Webコンソールデプロイ](#web-console-deployment)を使用してデプロイできます。

### SAM CLIデプロイ（推奨）

1. ([前提条件](#prerequisites)) [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html)をインストールします。

2. バンドルをビルドします:

    ```bash
    npm run build
    ```

3. [`template.yml`](https://github.com/tidb-samples/tidb-aws-lambda-quickstart/blob/main/template.yml)の環境変数を更新します:

    ```yaml
    Environment:
      Variables:
        TIDB_HOST: {tidb_server_host}
        TIDB_PORT: 4000
        TIDB_USER: {prefix}.root
        TIDB_PASSWORD: {password}
    ```

4. AWS環境変数を設定します（[短期認証](https://docs.aws.amazon.com/cli/latest/userguide/cli-authentication-short-term.html)を参照）:

    ```bash
    export AWS_ACCESS_KEY_ID={your_access_key_id}
    export AWS_SECRET_ACCESS_KEY={your_secret_access_key}
    export AWS_SESSION_TOKEN={your_session_token}
    ```

5. AWS Lambdaファンクションをデプロイします:

    ```bash
    sam deploy --guided

    # 例:

    # SAMデプロイの構成
    # ======================

    #        Configファイル[samconfig.toml]を検索中:  見つかりません

    #        'sam deploy'のためのデフォルト引数を設定中
    #        =========================================
    #        スタック名 [sam-app]: tidb-aws-lambda-quickstart
    #        AWSリージョン [us-east-1]:
    #        #リソースの変更を表示し、デプロイを開始するには'Y'を入力してください
    #        デプロイ前に変更を確認する [y/N]:
    #        #SAMは、テンプレート内のリソースに接続するためのロールを作成できるようにするために、許可が必要です
    #        SAM CLI IAMロールの作成を許可しますか [Y/n]:
    #        #操作が失敗したときに以前にプロビジョニングされたリソースの状態を保持します
    #        ロールバックを無効にする [y/N]:
    #        tidbHelloWorldFunctionには認証が定義されていないことがあります。 これは大丈夫ですか？ [y/N]: y
    #        tidbHelloWorldFunctionには認証が定義されていないことがあります。 これは大丈夫ですか？ [y/N]: y
```
    # tidbHelloWorldFunctionの認可が定義されていない場合、よろしいですか？[y/N]: y
    # tidbHelloWorldFunctionの認可が定義されていない場合、よろしいですか？[y/N]: y
    # 引数を設定ファイルに保存する[Y/n]:
    # SAM設定ファイル[samconfig.toml]:
    # SAM設定環境[default]:

    # デプロイに必要なリソースを検索中:
    # 必要なリソースを作成中...
    # 成功的に作成されました！
    ```

### Webコンソールデプロイ

1. バンドルをビルドする：

    ```bash
    npm run build

    # AWS Lambda用のバンドル
    # =====================
    # dist/index.zip
    ```

2. [AWS Lambdaコンソール](https://console.aws.amazon.com/lambda/home#/functions)を訪れます。

3. [Node.js Lambda関数の作成](https://docs.aws.amazon.com/lambda/latest/dg/lambda-nodejs.html)の手順に従ってください。

4. [Lambdaデプロイパッケージ](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-package.html#gettingstarted-package-zip)の手順に従い、`dist/index.zip`ファイルをアップロードします。

5. Lambda関数で[対応する接続文字列をコピーして構成します](https://docs.aws.amazon.com/lambda/latest/dg/configuration-envvars.html)。

    1. Lambdaコンソールの[関数](https://console.aws.amazon.com/lambda/home#/functions)ページで、**構成**タブを選択し、**環境変数**を選択します。
    2. **編集**を選択します。
    3. データベースへのアクセス資格情報を追加するには、以下の手順を実行します：
        - **環境変数を追加**を選択し、**キー**に`TIDB_HOST`、**値**にホスト名を入力します。
        - **環境変数を追加**を選択し、**キー**に`TIDB_PORT`、**値**にポート（デフォルトは4000）を入力します。
        - **環境変数を追加**を選択し、**キー**に`TIDB_USER`、**値**にユーザー名を入力します。
        - **環境変数を追加**を選択し、**キー**に`TIDB_PASSWORD`、**値**にデータベース作成時に選択したパスワードを入力します。
        - **保存**を選択します。

## サンプルコードスニペット

以下のサンプルコードスニペットを参照して、独自のアプリケーション開発を完了させることができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-aws-lambda-quickstart](https://github.com/tidb-samples/tidb-aws-lambda-quickstart)リポジトリをチェックしてください。

### TiDBへの接続

次のコードは、環境変数で定義されたオプションを使用してTiDBへの接続を確立します：

```typescript
// lib/tidb.ts
import mysql from 'mysql2';

let pool: mysql.Pool | null = null;

function connect() {
  return mysql.createPool({
    host: process.env.TIDB_HOST, // TiDBホスト、例：{gateway-region}.aws.tidbcloud.com
    port: process.env.TIDB_PORT ? Number(process.env.TIDB_PORT) : 4000, // TiDBポート、デフォルト：4000
    user: process.env.TIDB_USER, // TiDBユーザー、例：{prefix}.root
    password: process.env.TIDB_PASSWORD, // TiDBパスワード
    database: process.env.TIDB_DATABASE || 'test', // TiDBデータベース名、デフォルト：test
    ssl: {
      minVersion: 'TLSv1.2',
      rejectUnauthorized: true,
    },
    connectionLimit: 1, // サーバーレス環境でconnectionLimitを「1」に設定することで、リソース使用量を最適化し、コストを削減し、接続の安定性を確保し、シームレスなスケーラビリティを実現します。
    maxIdle: 1, // 最大アイドル接続数、デフォルト値は`connectionLimit`と同じです
    enableKeepAlive: true,
  });
}

export function getPool(): mysql.Pool {
  if (!pool) {
    pool = connect();
  }
  return pool;
}
```

### データの挿入

次のクエリは単一の`Player`レコードを作成し、`ResultSetHeader`オブジェクトを返します：

```typescript
const [rsh] = await pool.query('INSERT INTO players (coins, goods) VALUES (?, ?);', [100, 100]);
console.log(rsh.insertId);
```

詳細については、[データの挿入](/develop/dev-guide-insert-data.md)を参照してください。

### データのクエリ

次のクエリは、ID `1`による単一の`Player`レコードを返します：

```typescript
const [rows] = await pool.query('SELECT id, coins, goods FROM players WHERE id = ?;', [1]);
console.log(rows[0]);
```

詳細については、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データの更新

次のクエリは、ID `1`の`Player`に`50`のコインと`50`の商品を追加します：

```typescript
const [rsh] = await pool.query(
    'UPDATE players SET coins = coins + ?, goods = goods + ? WHERE id = ?;',
    [50, 50, 1]
);
console.log(rsh.affectedRows);
```

詳細については、[データの更新](/develop/dev-guide-update-data.md)を参照してください。

### データの削除

次のクエリは、ID `1`の`Player`レコードを削除します：

```typescript
const [rsh] = await pool.query('DELETE FROM players WHERE id = ?;', [1]);
console.log(rsh.affectedRows);
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md)を参照してください。

## 便利なノート

- [接続プール](https://github.com/sidorares/node-mysql2#using-connection-pools)を使用してデータベース接続を管理すると、頻繁な接続の確立と破棄による性能オーバーヘッドを減らすことができます。
- SQLインジェクションを防ぐために、[プリペアドステートメント](https://github.com/sidorares/node-mysql2#using-prepared-statements)を使用することを推奨します。
- 複雑なSQLステートメントが多くないシナリオでは、[Sequelize](https://sequelize.org/)、[TypeORM](https://typeorm.io/)、または[Prisma](https://www.prisma.io/)などのORMフレームワークを使用することで、開発効率を大幅に向上させることができます。
- アプリケーションのためにRESTful APIを構築する場合は、[AWS LambdaとAPI Gateway](https://docs.aws.amazon.com/lambda/latest/dg/services-apigateway.html)を使用することをお勧めします。
- TiDBサーバーレスとAWS Lambdaを使用した高性能アプリケーションの設計については、[このブログ](https://aws.amazon.com/blogs/apn/designing-high-performance-applications-using-serverless-tidb-cloud-and-aws-lambda/)を参照してください。

## 次のステップ

- 「TiDB AWS Lambda関数でのTiDBの使用方法」の詳細については、[TiDB-Lambda-integration/aws-lambda-bookstore Demo](https://github.com/pingcap/TiDB-Lambda-integration/blob/main/aws-lambda-bookstore/README.md)をご覧ください。また、AWS API Gatewayを使用してアプリケーションのためのRESTful APIを構築することもできます。
- [mysql2のドキュメント](https://github.com/sidorares/node-mysql2/tree/master/documentation/en)から`mysql2`のより詳細な使用方法について学ぶことができます。
- [LambdaのAWS開発者ガイド](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)からAWS Lambdaのより詳細な使用方法について学ぶことができます。
- [デベロッパーガイド](/develop/dev-guide-overview.md)の章を通じて、TiDBアプリケーション開発のベストプラクティスについて学ぶことができます。[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[単一テーブルからのデータ取得](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)などが含まれます。
- [TiDBデベロッパーコース](https://www.pingcap.com/education/)を通じてさらなる学習ができ、試験に合格した後に[TiDB認定](https://www.pingcap.com/education/certification/)を取得することができます。

## お手伝いが必要ですか？

<CustomContent platform="tidb">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問をするか、[サポートチケットを作成](/support.md)してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問をするか、[サポートチケットを作成](https://support.pingcap.com/)してください。

</CustomContent>