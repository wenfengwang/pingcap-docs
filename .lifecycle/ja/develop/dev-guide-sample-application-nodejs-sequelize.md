---
title: Sequelizeを使用してTiDBに接続する
summary: Sequelizeを使用してTiDBに接続する方法について学びます。このチュートリアルでは、Sequelizeを使用してTiDBと連携するためのNode.jsサンプルコードスニペットが提供されます。
---

# Sequelizeを使用してTiDBに接続する

TiDBはMySQL互換のデータベースであり、[Sequelize](https://sequelize.org/)はNode.js向けの人気のあるORMフレームワークです。

このチュートリアルでは、TiDBとSequelizeを使用して次のタスクを実行する方法を学ぶことができます。

- 環境をセットアップする。
- Sequelizeを使用してTiDBクラスターに接続する。
- アプリケーションを構築し、実行する。オプションで、基本的なCRUD操作の[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。

> **注意**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、TiDB Self-Hostedと共に動作します。

## 前提条件

このチュートリアルを完了するには、次のものが必要です：

- [Node.js **18**](https://nodejs.org/en/download/) 以降。
- [Git](https://git-scm.com/downloads)。
- TiDBクラスター。

<CustomContent platform="tidb">

**TiDBクラスターがない場合は、以下のように作成することができます：**

- (推奨) [TiDB Serverlessクラスターを作成](/develop/dev-guide-build-cluster-in-cloud.md)して、独自のTiDB Cloudクラスターを作成します。
- [ローカルテストTiDBクラスターをデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster)するか、または[本番用TiDBクラスターをデプロイ](/production-deployment-using-tiup.md)する手順に従って、ローカルクラスターを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

**TiDBクラスターがない場合は、以下のように作成することができます：**

- (推奨) [TiDB Serverlessクラスターを作成](/develop/dev-guide-build-cluster-in-cloud.md)して、独自のTiDB Cloudクラスターを作成します。
- [ローカルテストTiDBクラスターをデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster)するか、または[本番用TiDBクラスターをデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup)する手順に従って、ローカルクラスターを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する

このセクションでは、サンプルアプリケーションコードを実行してTiDBに接続する手順を示します。

> **注意**
>
> 完全なコードスニペットと実行手順については、[tidb-samples/tidb-nodejs-sequelize-quickstart](https://github.com/tidb-samples/tidb-nodejs-sequelize-quickstart) GitHubリポジトリを参照してください。

### ステップ1：サンプルアプリリポジトリをクローンする

ターミナルウィンドウで次のコマンドを実行して、サンプルコードリポジトリをクローンします:

```bash
git clone git@github.com:tidb-samples/tidb-nodejs-sequelize-quickstart.git
cd tidb-nodejs-sequelize-quickstart
```

### ステップ2：依存関係をインストールする

次のコマンドを実行して、サンプルアプリに必要なパッケージ（`sequelize`を含む）をインストールします:

```bash
npm install
```

### ステップ3：接続情報を設定する

TiDBのデプロイオプションに応じてTiDBクラスターに接続します。

<SimpleTab>

<div label="TiDB Serverless">

1. [**クラスター**](https://tidbcloud.com/console/clusters)ページに移動し、対象クラスターの名前をクリックして概要ページに移動します。

2. 右上の**Connect**をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの構成が、操作環境に一致するようにするため、次のことを確認してください。

    - **エンドポイントのタイプ**が`Public`に設定されていること
    - **Connect With**が`General`に設定されていること
    - **Operating System**が環境に一致していること

    > **注**
    >
    > Node.jsアプリケーションでは、TLS（SSL）接続を確立する際、組み込みの[Mozilla CA証明書](https://wiki.mozilla.org/CA/Included_Certificates)がデフォルトで使用されるため、SSL CA証明書を提供する必要はありません。

4. ランダムなパスワードを作成するために**Create password**をクリックします。

    > **ヒント**
    >
    > 以前にパスワードを生成した場合は、元のパスワードを使用するか、新しいパスワードを生成するために**Reset password**をクリックできます。

5. 次のコマンドを実行して`.env.example`をコピーし、`.env`に名前を変更します:

    ```shell
    cp .env.example .env
    ```

6. `.env`ファイルを編集し、次のように環境変数を設定します。接続ダイアログの接続パラメーターに対応するプレースホルダー`{}`を置き換えます:

    ```dotenv
    TIDB_HOST='{ホスト}'
    TIDB_PORT='4000'
    TIDB_USER='{ユーザー}'
    TIDB_PASSWORD='{パスワード}'
    TIDB_DB_NAME='test'
    TIDB_ENABLE_SSL='true'
    ```

7. `.env`ファイルを保存します。

</div>

<div label="TiDB Dedicated">

1. [**クラスター**](https://tidbcloud.com/console/clusters)ページに移動し、対象クラスターの名前をクリックして概要ページに移動します。

2. 右上の**Connect**をクリックします。接続ダイアログが表示されます。

3. **任意の場所からアクセスを許可**をクリックし、その後**TiDBクラスターCAをダウンロード**をクリックしてCA証明書をダウンロードします。

    接続文字列の詳細については、[TiDB Dedicated standard connection](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection)を参照してください。

4. 次のコマンドを実行して`.env.example`をコピーし、`.env`に名前を変更します:

    ```shell
    cp .env.example .env
    ```

5. `.env`ファイルを編集し、次のように環境変数を設定します。接続ダイアログの接続パラメーターに対応するプレースホルダー`{}`を置き換えます:

    ```shell
    TIDB_HOST='{ホスト}'
    TIDB_PORT='4000'
    TIDB_USER='{ユーザー}'
    TIDB_PASSWORD='{パスワード}'
    TIDB_DB_NAME='test'
    TIDB_ENABLE_SSL='true'
    TIDB_CA_PATH='{パス/トゥ/CA}'
    ```

6. `.env`ファイルを保存します。

</div>

<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して`.env.example`をコピーし、`.env`に名前を変更します:

    ```shell
    cp .env.example .env
    ```

2. `.env`ファイルを編集し、次のように環境変数を設定します。接続ダイアログの接続パラメーターに対応するプレースホルダー`{}`を置き換えます:

    ```shell
    TIDB_HOST='{ホスト}'
    TIDB_PORT='4000'
    TIDB_USER='root'
    TIDB_PASSWORD='{パスワード}'
    TIDB_DB_NAME='test'
    ```

    TiDBをローカルで実行している場合、デフォルトのホストアドレスは`127.0.0.1`であり、パスワードは空です。

3. `.env`ファイルを保存します。

</div>

</SimpleTab>

### ステップ4：サンプルアプリを実行する

次のコマンドを実行して、サンプルコードを実行します:

```shell
npm start
```

<details>
<summary>**期待される出力（一部）:**</summary>

```shell
INFO (app/10117): Getting sequelize instance...
Executing (default): SELECT 1+1 AS result
Executing (default): DROP TABLE IF EXISTS `players`;
Executing (default): CREATE TABLE IF NOT EXISTS `players` (`id` INTEGER NOT NULL auto_increment  COMMENT 'The unique ID of the player.', `coins` INTEGER NOT NULL COMMENT 'The number of coins that the player had.', `goods` INTEGER NOT NULL COMMENT 'The number of goods that the player had.', `createdAt` DATETIME NOT NULL, `updatedAt` DATETIME NOT NULL, PRIMARY KEY (`id`)) ENGINE=InnoDB;
Executing (default): SHOW INDEX FROM `players`
Executing (default): INSERT INTO `players` (`id`,`coins`,`goods`,`createdAt`,`updatedAt`) VALUES (1,100,100,'2023-08-31 09:10:11','2023-08-31 09:10:11'),(2,200,200,'2023-08-31 09:10:11','2023-08-31 09:10:11'),(3,300,300,'2023-08-31 09:10:11','2023-08-31 09:10:11'),(4,400,400,'2023-08-31 09:10:11','2023-08-31 09:10:11'),(5,500,500,'2023-08-31 09:10:11','2023-08-31 09:10:11');
Executing (default): SELECT `id`, `coins`, `goods`, `createdAt`, `updatedAt` FROM `players` AS `players` WHERE `players`.`coins` > 300;
Executing (default): UPDATE `players` SET `coins`=?,`goods`=?,`updatedAt`=? WHERE `id` = ?
Executing (default): DELETE FROM `players` WHERE `id` = 6
```

</details>

## サンプルコードスニペット

次のサンプルコードスニペットを参照して、独自のアプリケーション開発を完了させることができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-nodejs-sequelize-quickstart](https://github.com/tidb-samples/tidb-nodejs-sequelize-quickstart) リポジトリを確認してください。

### TiDB への接続

環境変数で定義されたオプションを使用して、以下のコードは TiDB への接続を確立します:

```typescript
// src/lib/tidb.ts
import { Sequelize } from 'sequelize';

export function initSequelize() {
  return new Sequelize({
    dialect: 'mysql',
    host: process.env.TIDB_HOST || 'localhost',     // 例: {gateway-region}.aws.tidbcloud.com の TiDB ホスト
    port: Number(process.env.TIDB_PORT) || 4000,    // デフォルト: 4000 の TiDB ポート
    username: process.env.TIDB_USER || 'root',      // 例: {prefix}.root の TiDB ユーザー
    password: process.env.TIDB_PASSWORD || 'root',  // TiDB パスワード
    database: process.env.TIDB_DB_NAME || 'test',   // デフォルト: test の TiDB データベース名
    dialectOptions: {
      ssl:
        process.env?.TIDB_ENABLE_SSL === 'true'     // (オプション) SSL を有効にする
          ? {
              minVersion: 'TLSv1.2',
              rejectUnauthorized: true,
              ca: process.env.TIDB_CA_PATH          // (オプション) カスタム CA 証明書へのパス
                ? readFileSync(process.env.TIDB_CA_PATH)
                : undefined,
            }
          : null,
    },
}

export async function getSequelize() {
  if (!sequelize) {
    sequelize = initSequelize();
    try {
      await sequelize.authenticate();
      logger.info('接続に成功しました。');
    } catch (error) {
      logger.error('データベースに接続できません:');
      logger.error(error);
      throw error;
    }
  }
  return sequelize;
}
```

### データの挿入

以下のクエリは単一の `Players` レコードを作成し、`Players` オブジェクトを返します:

```typescript
logger.info('新しいプレイヤーを作成中...');
const newPlayer = await playersModel.create({
  id: 6,
  coins: 600,
  goods: 600,
});
logger.info('新しいプレイヤーを作成しました。');
logger.info(newPlayer.toJSON());
```

詳細については、[データの挿入](/develop/dev-guide-insert-data.md) を参照してください。

### データのクエリ

以下のクエリは、`coins` が `300` より大きい単一の `Players` レコードを返します:

```typescript
logger.info('coins > 300 のすべてのプレイヤーを読み込んでいます...');
const allPlayersWithCoinsGreaterThan300 = await playersModel.findAll({
  where: {
    coins: {
      [Op.gt]: 300,
    },
  },
});
logger.info('coins > 300 のすべてのプレイヤーを読み込みました。');
logger.info(allPlayersWithCoinsGreaterThan300.map((p) => p.toJSON()));
```

詳細については、[データのクエリ](/develop/dev-guide-get-data-from-single-table.md) を参照してください。

### データの更新

以下のクエリは、[データの挿入](#insert-data) セクションで作成された ID `6` の `Players` に `700` のコインと `700` の商品を設定します:

```typescript
logger.info('新しいプレイヤーを更新中...');
await newPlayer.update({ coins: 700, goods: 700 });
logger.info('新しいプレイヤーを更新しました。');
logger.info(newPlayer.toJSON());
```

詳細については、[データの更新](/develop/dev-guide-update-data.md) を参照してください。

### データの削除

以下のクエリは、[データの挿入](#insert-data) セクションで作成された ID `6` の `Player` レコードを削除します:

```typescript
logger.info('新しいプレイヤーを削除中...');
await newPlayer.destroy();
const deletedNewPlayer = await playersModel.findByPk(6);
logger.info('新しいプレイヤーを削除しました。');
logger.info(deletedNewPlayer?.toJSON());
```

詳細については、[データの削除](/develop/dev-guide-delete-data.md) を参照してください。

## 次の手順

- [Sequelize のドキュメンテーション](https://sequelize.org/) から ORM フレームワーク Sequelize ドライバーのより詳細な利用法を学びます。
- [開発者ガイド](/develop/dev-guide-overview.md) の章で TiDB アプリケーション開発のベストプラクティスを学びます: [データの挿入](/develop/dev-guide-insert-data.md), [データの更新](/develop/dev-guide-update-data.md), [データの削除](/develop/dev-guide-delete-data.md), [単一テーブルの読み取り](/develop/dev-guide-get-data-from-single-table.md), [トランザクション](/develop/dev-guide-transaction-overview.md), [SQL パフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)。
- [PingCAP の公式 TiDB 開発者コース](https://www.pingcap.com/education/) を通じて学び、試験に合格すると [TiDB 認定](https://www.pingcap.com/education/certification/) を取得できます。

## 助けが必要ですか？

<CustomContent platform="tidb">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc) で質問をしたり、[サポートチケットを作成](/support.md) したりできます。

</CustomContent>

<CustomContent platform="tidb-cloud">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc) で質問をしたり、[サポートチケットを作成](https://support.pingcap.com/) したりできます。

</CustomContent>