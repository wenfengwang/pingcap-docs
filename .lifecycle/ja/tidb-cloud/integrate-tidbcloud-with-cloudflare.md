---
title: TiDB CloudをCloudflare Workersと統合する
summary: Cloudflare Workersを使用してTiDB Cloudと統合する方法を学びます。

# TiDB CloudをCloudflare Workersと統合する

[Cloudflare Workers](https://workers.cloudflare.com/)は、特定のイベントに応答してコードを実行するプラットフォームです。これには、HTTPリクエストやデータベースへの変更などが含まれます。Cloudflare Workersは使いやすく、カスタムAPI、サーバーレス関数、マイクロサービスなど、さまざまなアプリケーションの構築に使用することができます。低レイテンシのパフォーマンスが必要なアプリケーションや迅速にスケーリングする必要があるアプリケーションに特に有用です。

ただし、Cloudflare WorkersはV8エンジン上で実行されており、直接のTCP接続を行うことができないため、Cloudflare WorkersからTiDB Cloudに接続することが難しいことがあります。

幸いなことに、Prismaには[データプロキシ](https://www.prisma.io/docs/data-platform/data-proxy)があります。これを使用すると、Cloudflare Workersを使用してTCP接続経由で転送されるデータを処理および操作することができます。

このドキュメントでは、TiDB CloudとPrismaデータプロキシを使用してCloudflare Workersを展開する手順を段階的に示します。

> **注意:**
>
> Cloudflare Workersにローカルに展開されたTiDBを接続したい場合は、Cloudflareトンネルをプロキシとして使用する[worker-tidb](https://github.com/shiyuhang0/worker-tidb)を試すことができます。ただし、worker-tidbは本番環境での使用は推奨されません。

## 開始する前に

この記事の手順を試す前に、以下の準備が必要です。

- TiDB CloudアカウントおよびTiDB Cloud上のTiDB Serverlessクラスター。詳細については、[TiDB Cloudクイックスタート](/tidb-cloud/tidb-cloud-quickstart.md#step-1-create-a-tidb-cluster)を参照してください。
- [Cloudflare Workersアカウント](https://dash.cloudflare.com/login)。
- [Prisma Data Platformアカウント](https://cloud.prisma.io/)。
- [GitHubアカウント](https://github.com/login)。
- Node.jsとnpmをインストールします。
- `npm install -D prisma typescript wrangler`を使用して依存関係をインストールします。

## 手順1: Wranglerをセットアップする

[Wrangler](https://developers.cloudflare.com/workers/wrangler/)は、公式のCloudflare Worker CLIです。これを使用して、Workersの生成、ビルド、プレビュー、および公開ができます。

1. Wranglerに認証するには、`wrangler login`を実行します：

    ```
    wrangler login
    ```

2. Wranglerを使用してワーカープロジェクトを作成します：

    ```
    wrangler init prisma-tidb-cloudflare
    ```

3. ターミナルで、プロジェクトに関連する一連の質問が表示されます。すべての質問に対してデフォルト値を選択します。

## 手順2: Prismaをセットアップする

1. プロジェクトディレクトリに移動します：

    ```
    cd prisma-tidb-cloudflare
    ```

2. `prisma init`コマンドを使用してPrismaをセットアップします：

    ```
    npx prisma init
    ```

    これにより、`prisma/schema.prisma`にPrismaスキーマが作成されます。

3. `prisma/schema.prisma`内に、TiDBのテーブルに応じたスキーマを追加します。TiDBに`table1`と`table2`があると仮定し、以下のスキーマを追加できます：

    ```
    generator client {
      provider = "prisma-client-js"
    }

    datasource db {
      provider = "mysql"
      url      = env("DATABASE_URL")
    }

    model table1 {
      id   Int                   @id @default(autoincrement())
      name String
    }

    model table2 {
      id   Int                   @id @default(autoincrement())
      name String
    }
    ```

    このデータモデルは、Workerからの受信リクエストを保存するために使用されます。

## 手順3: プロジェクトをGitHubにプッシュする

1. GitHubで`prisma-tidb-cloudflare`という名前のリポジトリを[作成](https://github.com/new)します。

2. リポジトリを作成した後、プロジェクトをGitHubにプッシュできます：

    ```
    git remote add origin https://github.com/<username>/prisma-tidb-cloudflare
    git add .
    git commit -m "initial commit"
    git push -u origin main
    ```

## 手順4: プロジェクトをPrisma Data Platformにインポートする

Cloudflare Workersでは、TCPサポートがないため、直接データベースにアクセスすることはできません。代わりに、前述のようにPrisma Data Proxyを使用できます。

1. [Prisma Data Platform](https://cloud.prisma.io/)にサインインし、**新しいプロジェクト**をクリックします。
2. **接続文字列**にこのパターン `mysql://USER:PASSWORD@HOST:PORT/DATABASE?sslaccept=strict` を入力します。接続情報は[TiDB Cloudコンソール](https://tidbcloud.com/console/clusters)にあります。
3. **Static IPs**は無効のままにしておきます。TiDB ServerlessはどのIPアドレスからでもアクセス可能です。
4. TiDB Cloudクラスターの場所に地理的に近いデータプロキシリージョンを選択します。その後、**プロジェクトを作成**をクリックします。

   ![プロジェクト設定を構成](/media/tidb-cloud/cloudflare/cloudflare-project.png)

5. リポジトリを入力し、**Get Started**ページで**Link Prisma schema**をクリックします。
6. **新しい接続文字列を作成**をクリックし、`prisma://`で始まる新しい接続文字列が取得できます。この接続文字列をコピーして保存します。

   ![新しい接続文字列を作成](/media/tidb-cloud/cloudflare/cloudflare-start.png)

7. **Skip and continue to Data Platform**をクリックして、データプラットフォームに移動します。

## 手順5: 環境でData Proxy接続文字列を設定する

1. Data Proxy接続文字列をローカル環境の `.env` ファイルに追加します：

    ```
    DATABASE_URL=prisma://aws-us-east-1.prisma-data.com/?api_key=•••••••••••••••••"
    ```

2. Data Proxy接続をCloudflare Workersにシークレットとして追加します：

    ```
    wrangler secret put DATABASE_URL
    ```

3. プロンプトに従って、Data Proxy接続文字列を入力します。

> **注意:**
>
> Cloudflare Workersダッシュボードを使用して、`DATABASE_URL`シークレットを編集することもできます。

## 手順6: Prisma Clientを生成する

[Data Proxy](https://www.prisma.io/docs/data-platform/data-proxy)を介して接続するPrisma Clientを生成します：

```
npx prisma generate --data-proxy
```

## 手順7: Cloudflare Worker関数を開発する

必要に応じて`src/index.ts`を変更する必要があります。

たとえば、URL変数で異なるテーブルをクエリしたい場合は、次のコードを使用できます：

```js
import { PrismaClient } from '@prisma/client/edge'
const prisma = new PrismaClient()

addEventListener('fetch', (event) => {
  event.respondWith(handleEvent(event))
})

async function handleEvent(event: FetchEvent): Promise<Response> {
  // URLパラメーターを取得
  const { request } = event
  const url = new URL(request.url);
  const table = url.searchParams.get('table');
  let limit = url.searchParams.get('limit');
  const limitNumber = limit? parseInt(limit): 100;

  // モデルを取得
  let model
  for (const [key, value] of Object.entries(prisma)) {
    if (typeof value == 'object' && key == table) {
      model = value
      break
    }
  }
  if(!model){
    return new Response("Table not defined")
  }

  // データを取得
  const result = await model.findMany({ take: limitNumber })
  return new Response(JSON.stringify({ result }))
}
```

## 手順8: Cloudflare Workersに公開する

Cloudflare Workersにデプロイする準備ができました。

プロジェクトディレクトリで、以下のコマンドを実行します：

```
npx wrangler publish
```

## 手順9: Cloudflare Workersを試す

1. [Cloudflareダッシュボード](https://dash.cloudflare.com)に移動して、ワーカーを検索します。概要ページでワーカーのURLを見つけることができます。

2. テーブル名を指定してURLにアクセスします：`https://{your-worker-url}/?table={table_name}`。対応するTiDBテーブルから結果が得られます。

## プロジェクトの更新

### サーバーレス関数の変更

サーバーレス関数を変更する場合は、`src/index.ts`を更新し、再度Cloudflare Workersに公開してください。

### 新しいテーブルの作成

新しいテーブルを作成し、それをクエリしたい場合は、以下の手順を実行します：

1. `prisma/schema.prisma`に新しいモデルを追加します。
2. 変更をリポジトリにプッシュします。

    ```
    git add prisma
    git commit -m "add new model"
    git push
    ```

3. Prisma Clientを再度生成します。

    ```
    npx prisma generate --data-proxy
    ```

4. Cloudflare Workerを再度公開します。

    ```
    npx wrangler publish
    ```