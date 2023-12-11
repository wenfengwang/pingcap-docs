---
title: Next.jsを使用してData AppのOpenAPI仕様を使用する
summary: Data AppのOpenAPI仕様を使用してクライアントコードを生成し、Next.jsアプリケーションを開発する方法について学びます。

# Next.jsを使用してData AppのOpenAPI仕様を使用する

このドキュメントでは、[Data App](/tidb-cloud/tidb-cloud-glossary.md#data-app)のOpenAPI仕様を使用してクライアントコードを生成し、Next.jsアプリケーションを開発する方法について紹介します。

## 開始する前に

Next.jsでOpenAPI仕様を使用する前に、次のものを持っていることを確認してください。

- TiDBクラスター。詳細については、[TiDB Serverlessクラスターを作成](/tidb-cloud/create-tidb-cluster-serverless.md)または[TiDB Dedicatedクラスターを作成](/tidb-cloud/create-tidb-cluster.md)を参照してください。
- [Node.js](https://nodejs.org/en/download)
- [npm](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)
- [yarn](https://yarnpkg.com/getting-started/install)

このドキュメントでは、TiDB Serverlessクラスターを例にしています。

## ステップ1. データを準備する

まず、TiDBクラスターで`test.repository`というテーブルを作成し、いくつかのサンプルデータを挿入します。次の例は、PingCAPが開発したオープンソースプロジェクトをデモンストレーション用のデータとして挿入しています。

SQLステートメントを実行するには、[TiDB Cloudコンソール](https://tidbcloud.com)の[Chat2Query](/tidb-cloud/explore-data-with-chat2query.md)を使用できます。

```sql
-- データベースを選択する
USE test;

-- テーブルを作成する
CREATE TABLE repository (
        id int NOT NULL PRIMARY KEY AUTO_INCREMENT,
        name varchar(64) NOT NULL,
        url varchar(256) NOT NULL
);

-- テーブルにいくつかのサンプルデータを挿入する
INSERT INTO repository (name, url)
VALUES ('tidb', 'https://github.com/pingcap/tidb'),
        ('tikv', 'https://github.com/tikv/tikv'),
        ('pd', 'https://github.com/tikv/pd'),
        ('tiflash', 'https://github.com/pingcap/tiflash');
```

## ステップ2. Data Appを作成する

データが挿入されたら、[TiDB Cloudコンソール](https://tidbcloud.com)の[**Data Service**](https://tidbcloud.com/console/data-service)ページに移動して、TiDBクラスターにリンクするData Appを作成し、Data App用のAPIキーを作成し、次にData App内に`GET /repositories`エンドポイントを作成します。このエンドポイントの対応するSQLステートメントは、`test.repository`テーブルからすべての行を取得するものです。

```sql
SELECT * FROM test.repository;
```

詳細については、[データサービスのはじめ方](/tidb-cloud/data-service-get-started.md)を参照してください。

## ステップ3. クライアントコードを生成する

次の例では、Next.jsを使用してData AppのOpenAPI仕様を使用してクライアントコードを生成する方法を示します。

1. `hello-repos`というNext.jsプロジェクトを作成します。

    公式のテンプレートを使用してNext.jsプロジェクトを作成するには、次のコマンドを使用し、プロンプトされたときにすべてのデフォルトオプションを維持します。

    ```shell
    yarn create next-app hello-repos
    ```

    次のコマンドを使用して、新しく作成したプロジェクトのディレクトリに移動します。

    ```shell
    cd hello-repos
    ```

2. 依存関係をインストールします。

    このドキュメントでは、[OpenAPI Generator](https://github.com/OpenAPITools/openapi-generator)を使用して、OpenAPI仕様から自動的にAPIクライアントライブラリを生成します。

    開発依存関係としてOpenAPI Generatorをインストールするには、次のコマンドを実行します。

    ```shell
    yarn add @openapitools/openapi-generator-cli --dev
    ```

3. OpenAPI仕様をダウンロードし、`oas/doc.json`として保存します。

    1. TiDB Cloudの[**Data Service**](https://tidbcloud.com/console/data-service)ページで、左ペインでData App名をクリックしてApp設定を表示します。
    2. **API Specification**エリアで、**Download**をクリックし、JSON形式を選択し、プロンプトが表示された場合は**Authorize**をクリックします。
    3. ダウンロードしたファイルを`hello-repos`プロジェクトディレクトリに`oas/doc.json`として保存します。

    詳細については、[OpenAPI仕様のダウンロード](/tidb-cloud/data-service-manage-data-app.md#download-the-openapi-specification)を参照してください。

    `oas/doc.json`ファイルの構造は次のようになります。

    ```json
    {
      "openapi": "3.0.3",
      "components": {
        "schemas": {
          "getRepositoriesResponse": {
            "properties": {
              "data": {
                "properties": {
                  "columns": { ... },
                  "result": { ... },
                  "rows": {
                    "items": {
                      "properties": {
                        "id": {
                          "type": "string"
                        },
                        "name": {
                          "type": "string"
                        },
                        "url": {
                          "type": "string"
                        }
    ...
      "paths": {
        "/repositories": {
          "get": {
            "operationId": "getRepositories",
            "responses": {
              "200": {
                "content": {
                  "application/json": {
                    "schema": {
                      "$ref": "#/components/schemas/getRepositoriesResponse"
                    }
                  }
                },
                "description": "OK"
              },
    ...
    ```

4. クライアントコードを生成します。

    ```shell
    yarn run openapi-generator-cli generate -i oas/doc.json --generator-name typescript-fetch -o gen/api
    ```

    このコマンドは、`oas/doc.json`仕様を入力として使用してクライアントコードを生成し、クライアントコードを`gen/api`ディレクトリに出力します。

## ステップ4. Next.jsアプリケーションを開発する

生成されたクライアントコードを使用して、Next.jsアプリケーションを開発できます。

1. `hello-repos`プロジェクトディレクトリで、`.env.local`ファイルを作成し、次の変数を設定して、Data Appのパブリックキーとプライベートキーを設定します。

    ```
    TIDBCLOUD_DATA_SERVICE_PUBLIC_KEY=YOUR_PUBLIC_KEY
    TIDBCLOUD_DATA_SERVICE_PRIVATE_KEY=YOUR_PRIVATE_KEY
    ```

    Data AppのAPIキーを作成する方法については、[APIキーの作成](/tidb-cloud/data-service-api-key.md#create-an-api-key)を参照してください。

2. `hello-repos`プロジェクトディレクトリで、`app/page.tsx`のコンテンツを次のコードに置き換えて、`GET /repositories`エンドポイントからデータを取得し、それをレンダリングします。

    ```js
    import {DefaultApi, Configuration} from "../gen/api"

    export default async function Home() {
      const config = new Configuration({
        username: process.env.TIDBCLOUD_DATA_SERVICE_PUBLIC_KEY,
        password: process.env.TIDBCLOUD_DATA_SERVICE_PRIVATE_KEY,
      });
      const apiClient = new DefaultApi(config);
      const resp = await apiClient.getRepositories();
      return (
        <main className="flex min-h-screen flex-col items-center justify-between p-24">
          <ul className="font-mono text-2xl">
            {resp.data.rows.map((repo) => (
              <a href={repo.url}>
                <li key={repo.id}>{repo.name}</li>
              </a>
            ))}
          </ul>
        </main>
      )
    }
    ```

    > **注意:**
    >
    > Data Appのリンクされたクラスターが異なるリージョンにホストされている場合、ダウンロードしたOpenAPI仕様ファイルの`servers`セクションに複数のアイテムが表示されます。この場合、`config`オブジェクトでエンドポイントパスを次のように構成する必要があります。
    >
    >  ```js
    >  const config = new Configuration({
    >      username: process.env.TIDBCLOUD_DATA_SERVICE_PUBLIC_KEY,
    >      password: process.env.TIDBCLOUD_DATA_SERVICE_PRIVATE_KEY,
    >      basePath: "https://${YOUR_REGION}.data.dev.tidbcloud.com/api/v1beta/app/${YOUR_DATA_APP_ID}/endpoint"
    >    });
    >  ```
    >
    > `basePath`をData Appの実際のエンドポイントパスに置き換えてください。`${YOUR_REGION}`と`${YOUR_DATA_APP_ID}`を取得するには、エンドポイントの**プロパティ**パネルで**Endpoint URL**を確認してください。

## ステップ5. Next.jsアプリケーションをプレビューする

> **注意:**
>
> プレビュー前に、すべての必要な依存関係がインストールされて正しく構成されていることを確認してください。

ローカル開発サーバーでアプリケーションをプレビューするには、次のコマンドを実行します。

```shell
yarn dev
```

その後、ブラウザで<http://localhost:3000>を開いて、`test.repository`データベースからのデータがページに表示されることを確認できます。