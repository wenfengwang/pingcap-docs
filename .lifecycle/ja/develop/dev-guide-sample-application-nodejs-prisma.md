---
title: Prismaを使用してTiDBに接続する
summary: Prismaを使用してTiDBに接続する方法を学びます。このチュートリアルでは、Node.jsサンプルコードスニペットを使用してTiDBをPrismaと連携する方法を説明します。
---

# Prismaを使用してTiDBに接続する

TiDBはMySQL互換のデータベースであり、[Prisma](https://github.com/prisma/prisma)はNode.js向けの人気のあるオープンソースのORMフレームワークです。

このチュートリアルでは、TiDBとPrismaを使用して以下のタスクを達成する方法を学ぶことができます。

- 環境をセットアップする。
- Prismaを使用してTiDBクラスタに接続する。
- アプリケーションをビルドして実行する。オプションで、基本的なCRUD操作の[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。

> **注意:**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、TiDB Self-Hostedと連動します。

## 前提条件

このチュートリアルを完了するには、次のものが必要です。

- マシンにインストールされている[Node.js](https://nodejs.org/ja) >= 16.x
- マシンにインストールされている[Git](https://git-scm.com/downloads)
- 実行中のTiDBクラスタ

**TiDBクラスタをお持ちでない場合、次のように作成できます:**

<CustomContent platform="tidb">

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)または[プロダクションTiDBクラスタのデプロイ](/production-deployment-using-tiup.md)に従ってローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

- (推奨) [TiDB Serverlessクラスタの作成](/develop/dev-guide-build-cluster-in-cloud.md)に従って独自のTiDB Cloudクラスタを作成します。
- [ローカルテストTiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)または[プロダクションTiDBクラスタのデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)に従ってローカルクラスタを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する

このセクションでは、サンプルアプリケーションコードを実行し、TiDBに接続する方法を示します。

### ステップ1: サンプルアプリケーションリポジトリをクローンする

ターミナルウィンドウで次のコマンドを実行して、サンプルコードリポジトリをクローンします：

```shell
git clone https://github.com/tidb-samples/tidb-nodejs-prisma-quickstart.git
cd tidb-nodejs-prisma-quickstart
```

### ステップ2: 依存関係をインストールする

次のコマンドを実行して、サンプルアプリケーションに必要なパッケージ（`prisma`を含む）をインストールします：

```shell
npm install
```

<details>
<summary><b>既存のプロジェクトに依存関係をインストールする</b></summary>

既存のプロジェクトの場合、次のコマンドを実行してパッケージをインストールします：

```shell
npm install prisma typescript ts-node @types/node --save-dev
```

</details>

### ステップ3: 接続パラメーターを指定する

TiDBの展開オプションに応じてTiDBクラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、対象のクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの構成が実行環境に一致することを確認します。

    - **エンドポイントタイプ**が`Public`に設定されていること。
    - **Connect With**が`General`に設定されていること。
    - **オペレーティングシステム**がアプリケーションを実行するオペレーティングシステムに一致すること。

4. パスワードをまだ設定していない場合は、ランダムなパスワードを生成するために**Create password**をクリックします。

5. 次のコマンドを実行して`.env.example`をコピーし、`.env`に名前を変更します：

    ```shell
    cp .env.example .env
    ```

6. `.env`ファイルを編集し、環境変数`DATABASE_URL`を次のように設定します。接続ダイアログの対応するプレースホルダー`{}`を接続パラメーターで置き換えます：

    ```dotenv
    DATABASE_URL=mysql://{user}:{password}@{host}:4000/test?sslaccept=strict
    ```

    > **注意**
    >
    > TiDB Serverlessでは、パブリックエンドポイントを使用する場合は`sslaccept=strict`を設定してTLS接続を有効にする必要があります。

7. `.env`ファイルを保存します。
8. `prisma/schema.prisma`で、`mysql`を接続プロバイダとして設定し、`env("DATABASE_URL")`を接続URLとして設定します：

    ```prisma
    datasource db {
      provider = "mysql"
      url      = env("DATABASE_URL")
    }
    ```

</div>
<div label="TiDB Dedicated">

1. [**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動し、対象のクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の**Connect**をクリックします。接続ダイアログが表示されます。

3. **Allow Access from Anywhere**をクリックし、**Download TiDB cluster CA**をクリックしてCA証明書をダウンロードします。

    接続文字列の取得方法の詳細については、「[TiDB Dedicatedの標準接続](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)」を参照してください。

4. 次のコマンドを実行して`.env.example`をコピーし、`.env`に名前を変更します：

    ```shell
    cp .env.example .env
    ```

5. `.env`ファイルを編集し、環境変数`DATABASE_URL`を次のように設定します。接続ダイアログの対応するプレースホルダー`{}`を接続パラメーターで置き換えます：

    ```dotenv
    DATABASE_URL=mysql://{user}:{password}@{host}:4000/test?sslaccept=strict&sslcert={downloaded_ssl_ca_path}
    ```

    > **注意**
    >
    > TiDB Serverlessを使用する場合、パブリックエンドポイントでTLS接続を有効にすることを**推奨**します。`sslaccept=strict`を設定してTLS接続を有効にすると、`sslcert=/path/to/ca.pem`を通じて接続ダイアログからダウンロードされたCA証明書のファイルパスを指定する必要があります。

6. `.env`ファイルを保存します。
7. `prisma/schema.prisma`で、`mysql`を接続プロバイダとして設定し、`env("DATABASE_URL")`を接続URLとして設定します：

    ```prisma
    datasource db {
      provider = "mysql"
      url      = env("DATABASE_URL")
    }
    ```

</div>
<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して`.env.example`をコピーし、`.env`に名前を変更します：

    ```shell
    cp .env.example .env
    ```

2. `.env`ファイルを編集し、環境変数`DATABASE_URL`を次のように設定します。TiDBクラスタの接続パラメーターの対応するプレースホルダー`{}`を置き換えます：

    ```dotenv
    DATABASE_URL=mysql://{user}:{password}@{host}:4000/test
    ```

   もしTiDBをローカルで実行している場合は、デフォルトのホストアドレスは`127.0.0.1`であり、パスワードは空です。

3. `.env`ファイルを保存します。

4. `prisma/schema.prisma`で、`mysql`を接続プロバイダとして設定し、`env("DATABASE_URL")`を接続URLとして設定します：

    ```prisma
    datasource db {
      provider = "mysql"
      url      = env("DATABASE_URL")
    }
    ```

</div>
</SimpleTab>

### ステップ4: データベーススキーマを初期化する

次のコマンドを実行して[Prisma Migrate](https://www.prisma.io/docs/concepts/components/prisma-migrate)を呼び出し、`prisma/prisma.schema`で定義されたデータモデルを使用してデータベースを初期化します。

```shell
npx prisma migrate dev
```

**`prisma.schema`で定義されたデータモデル:**

```prisma
// `players`テーブルを表すPlayerモデルを定義します。
model Player {
  id        Int      @id @default(autoincrement())
  name      String   @unique(map: "uk_player_on_name") @db.VarChar(50)
  coins     Decimal  @default(0)
  goods     Int      @default(0)
  createdAt DateTime @default(now()) @map("created_at")
  profile   Profile?

  @@map("players")
}

// `profiles`テーブルを表すProfileモデルを定義します。
model Profile {
  playerId  Int    @id @map("player_id")
  biography String @db.Text

  // 外部キーを使用して`Player`と`Profile`モデルの1:1の関連を定義します。
  player    Player @relation(fields: [playerId], references: [id], onDelete: Cascade, map: "fk_profile_on_player_id")

  @@map("profiles")
}
```

Prismaでデータモデルを定義する方法については、「[Data model](https://www.prisma.io/docs/concepts/components/prisma-schema/data-model)」のドキュメントを参照してください。

**実行結果の期待値:**

```
Your database is now in sync with your schema.
```
✔ 54  (プリズマ クライアント (5.1.1 | ライブラリ) を ./node_modules/@prisma/client に生成するのに 54ms かかりました。)

このコマンドは、`prisma/prisma.schema` を基にした TiDB データベースへのアクセスを可能にする [Prisma クライアント](https://www.prisma.io/docs/concepts/components/prisma-client) も生成します。

### ステップ 5: コードを実行する

以下のコマンドを実行して、サンプルコードを実行します。

```shell
npm start
```

**サンプルコードのメインロジック:**

```typescript
// ステップ 1. 自動生成された `@prisma/client` パッケージをインポートします。
import {Player, PrismaClient} from '@prisma/client';

async function main(): Promise<void> {
  // ステップ 2. 新しい `PrismaClient` インスタンスを作成します。
  const prisma = new PrismaClient();
  try {

    // ステップ 3. Prisma クライアントでいくつかの CRUD 操作を実行します...

  } finally {
    // ステップ 4. Prisma クライアントを切断します。
    await prisma.$disconnect();
  }
}

void main();
```

**期待される実行結果:**

接続に成功すると、ターミナルには次のように TiDB クラスタのバージョンが表示されます:

```
🔌 TiDB クラスタに接続しました！ (TiDB version: 5.7.25-TiDB-v6.6.0-serverless)
🆕 ID 1 の新しいプレイヤーが作成されました。
ℹ️ プレイヤー 1: Player { id: 1, coins: 100, goods: 100 } を取得しました。
🔢 プレイヤー 1 に 50 コインと 50 貨物を追加しました。現在プレイヤー 1 は 150 コインと 150 貨物を持っています。
🚮 プレイヤー 1 が削除されました。
```

## サンプルコードスニペット

自分自身のアプリケーション開発を完成させるために、以下のサンプルコードスニペットを参照できます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-nodejs-prisma-quickstart](https://github.com/tidb-samples/tidb-nodejs-prisma-quickstart) リポジトリを確認してください。

### データの挿入

以下のクエリは、単一の `Player` レコードを作成し、TiDB によって生成された `id` フィールドを含む作成された `Player` オブジェクトを返します:

```javascript
const player: Player = await prisma.player.create({
   data: {
      name: 'Alice',
      coins: 100,
      goods: 200,
      createdAt: new Date(),
   }
});
```

詳細はこちらを参照してください: [データの挿入](/develop/dev-guide-insert-data.md)。

### データのクエリ

以下のクエリは、ID `101` の単一の `Player` オブジェクトを返します。見つからない場合は `null` を返します:

```javascript
const player: Player | null = prisma.player.findUnique({
   where: {
      id: 101,
   }
});
```

詳細はこちらを参照してください: [データのクエリ](/develop/dev-guide-get-data-from-single-table.md)。

### データの更新

以下のクエリは、ID `101` の `Player` に `50` コインと `50` 貨物を追加します:

```javascript
await prisma.player.update({
   where: {
      id: 101,
   },
   data: {
      coins: {
         increment: 50,
      },
      goods: {
         increment: 50,
      },
   }
});
```

詳細はこちらを参照してください: [データの更新](/develop/dev-guide-update-data.md)。

### データの削除

以下のクエリは、ID `101` の `Player` を削除します:

```javascript
await prisma.player.delete({
   where: {
      id: 101,
   }
});
```

詳細はこちらを参照してください: [データの削除](/develop/dev-guide-delete-data.md)。

## 便利なノート

### 外部キー制約 vs Prisma 関連モード

[参照整合性](https://en.wikipedia.org/wiki/Referential_integrity?useskin=vector) をチェックするために、外部キー制約または Prisma 関連モードを使用できます:

- [外部キー](https://docs.pingcap.com/tidb/stable/foreign-key) は、関連データのクロステーブル参照およびデータ整合性を維持する外部キー制約を可能にする TiDB v6.6.0 からサポートされる実験的な機能です。

    > **警告:**
    >
    > **外部キーは、小規模および中規模のデータシナリオに適しています。** 大規模なデータ量で外部キーを使用すると、重大なパフォーマンスの問題が発生する可能性があり、システムに予測できない影響を与える可能性があります。外部キーを使用する予定がある場合は、まず徹底的な検証を行い、慎重に使用してください。

- [Prisma 関連モード](https://www.prisma.io/docs/concepts/components/prisma-schema/relations/relation-mode) は Prisma Client サイドでの参照整合性のエミュレーションです。ただし、参照整合性を維持するために追加のデータベースクエリが必要となるため、パフォーマンスに影響を与える可能性があることに注意してください。

## 次のステップ

- [Prisma のドキュメント](https://www.prisma.io/docs) から ORM フレームワーク Prisma ドライバのより詳細な使用方法を学びます。
- [開発者ガイド](/develop/dev-guide-overview.md) の章で、TiDB アプリケーション開発のベストプラクティスを学びます。[データの挿入](/develop/dev-guide-insert-data.md)、[データの更新](/develop/dev-guide-update-data.md)、[データの削除](/develop/dev-guide-delete-data.md)、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、[SQL パフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md) などが含まれます。
- プロフェッショナルな [TiDB 開発者コース](https://www.pingcap.com/education/) を通じて学び、試験に合格して [TiDB 認定](https://www.pingcap.com/education/certification/) を取得します。

## お問い合わせ

<CustomContent platform="tidb">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc) で質問するか、[サポートチケットを作成](/support.md)してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc) で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。

</CustomContent>