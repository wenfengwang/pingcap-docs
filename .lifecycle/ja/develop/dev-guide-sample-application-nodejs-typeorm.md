---
title: TypeORMを使用してTiDBに接続する
summary: TypeORMを使用してTiDBに接続する方法を学びます。このチュートリアルでは、Node.jsのサンプルコードスニペットを使用して、TypeORMを使ってTiDBと連携する方法を示します。
---

# TypeORMを使用してTiDBに接続する

TiDBはMySQL互換のデータベースです。[TypeORM](https://github.com/TypeORM/TypeORM)はNode.js向けの人気のあるオープンソースORMフレームワークです。

このチュートリアルでは、TiDBとTypeORMを使用して次のタスクを実行する方法について学ぶことができます。

- 環境を設定する。
- TypeORMを使用してTiDBクラスタに接続する。
- アプリケーションをビルドして実行する。オプションで、基本的なCRUD操作用の[サンプルコードスニペット](#sample-code-snippets)を見つけることができます。

> **注意**
>
> このチュートリアルは、TiDB Serverless、TiDB Dedicated、およびTiDB Self-Hostedで動作します。

## 前提条件

このチュートリアルを完了するには、次のものが必要です。

- 自分のマシンにインストールされている[Node.js](https://nodejs.org/en) >= 16.x。
- 自分のマシンにインストールされている[Git](https://git-scm.com/downloads)。
- 実行中のTiDBクラスタ。

**TiDBクラスタを持っていない場合は、次のように作成できます:**

<CustomContent platform="tidb">

- (推奨) [TiDB Serverlessクラスタを作成](/develop/dev-guide-build-cluster-in-cloud.md) に従って、独自のTiDB Cloudクラスタを作成します。
- [ローカルでテスト用TiDBクラスタをデプロイ](/quick-start-with-tidb.md#deploy-a-local-test-cluster) するか、[本番用TiDBクラスタをデプロイ](/production-deployment-using-tiup.md) してローカルクラスタを作成します。

</CustomContent>
<CustomContent platform="tidb-cloud">

- (推奨) [TiDB Serverlessクラスタを作成](/develop/dev-guide-build-cluster-in-cloud.md) に従って、独自のTiDB Cloudクラスタを作成します。
- [ローカルでテスト用TiDBクラスタをデプロイ](https://docs.pingcap.com/tidb/stable/quick-start-with-tidb#deploy-a-local-test-cluster) するか、[本番用TiDBクラスタをデプロイ](https://docs.pingcap.com/tidb/stable/production-deployment-using-tiup) してローカルクラスタを作成します。

</CustomContent>

## サンプルアプリを実行してTiDBに接続する

このセクションでは、サンプルアプリケーションコードを実行してTiDBに接続する方法について説明します。

### ステップ1: サンプルアプリのリポジトリをクローンする

次のコマンドを端末ウィンドウで実行して、サンプルコードリポジトリをクローンします:

```shell
git clone https://github.com/tidb-samples/tidb-nodejs-typeorm-quickstart.git
cd tidb-nodejs-typeorm-quickstart
```

### ステップ2: 依存関係をインストールする

次のコマンドを実行して、サンプルアプリケーションに必要なパッケージ（`typeorm`および`mysql2`を含む）をインストールします:

```shell
npm install
```

<details>
<summary><b>既存のプロジェクトに依存関係をインストールする</b></summary>

既存のプロジェクトの場合、次のコマンドを実行してパッケージをインストールします:

- `typeorm`: Node.js向けのORMフレームワーク。
- `mysql2`: Node.js向けのMySQLドライバ。`mysql`ドライバを使用することもできます。
- `dotenv`: `.env`ファイルから環境変数を読み込みます。
- `typescript`: TypeScriptコードをJavaScriptにコンパイルします。
- `ts-node`: コンパイルせずにTypeScriptコードを直接実行します。
- `@types/node`: Node.js向けのTypeScript型定義を提供します。

```shell
npm install typeorm mysql2 dotenv --save
npm install @types/node ts-node typescript --save-dev
```

</details>

### ステップ3: 接続情報を設定する

TiDBのデプロイメントオプションに応じて、TiDBクラスタに接続します。

<SimpleTab>
<div label="TiDB Serverless">

1. [**クラスタ**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3. 接続ダイアログの設定が実行環境に一致することを確認します。

    - **エンドポイントタイプ** が `Public` に設定されていること。
    - **Connect With** が `General` に設定されていること。
    - **オペレーティングシステム** がアプリケーションの実行環境に一致すること。

4. パスワードがまだ設定されていない場合は、**Create password** をクリックしてランダムなパスワードを生成します。

5. 次のコマンドを実行して、`.env.example`をコピーして`.env`にリネームします:

    ```shell
    cp .env.example .env
    ```

6. `.env`ファイルを編集し、次のように環境変数を設定します。接続ダイアログの対応するプレースホルダ `{}` を接続パラメータで置き換えます:

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
    > TiDB Serverlessを使用する場合は、パブリックエンドポイントを使用する際に`TIDB_ENABLE_SSL`を介してTLS接続を有効にする必要があります。

7. `.env`ファイルを保存します。

</div>
<div label="TiDB Dedicated">

1. [**クラスタ**](https://tidbcloud.com/console/clusters) ページに移動し、ターゲットクラスタの名前をクリックして概要ページに移動します。

2. 右上隅の **Connect** をクリックします。接続ダイアログが表示されます。

3. **Anywhereからのアクセスを許可** をクリックし、次に **Download TiDB cluster CA** をクリックしてCA証明書をダウンロードします。

    接続文字列を取得する詳細な手順については、[TiDB Dedicated standard connection](https://docs.pingcap.com/tidbcloud/connect-via-standard-connection) を参照してください。

4. 次のコマンドを実行して、`.env.example`をコピーして`.env`にリネームします:

    ```shell
    cp .env.example .env
    ```

5. `.env`ファイルを編集し、次のように環境変数を設定します。接続ダイアログの対応するプレースホルダ `{}` を接続パラメータで置き換えます:

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
    > TiDB Dedicatedを使用する場合は、パブリックエンドポイントを使用する際に`TIDB_ENABLE_SSL`を介してTLS接続を有効にすることを **推奨** します。`TIDB_ENABLE_SSL=true`を設定する場合、接続ダイアログからダウンロードしたCA証明書のパスを`TIDB_CA_PATH=/path/to/ca.pem`で指定する必要があります。

6. `.env`ファイルを保存します。

</div>
<div label="TiDB Self-Hosted">

1. 次のコマンドを実行して、`.env.example`をコピーして`.env`にリネームします:

    ```shell
    cp .env.example .env
    ```

2. `.env`ファイルを編集し、次のように環境変数を設定します。TiDBクラスタの接続パラメータで対応するプレースホルダ `{}` を置き換えます:

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

### ステップ4: データベーススキーマを初期化する

次のコマンドを実行して、TypeORM CLIを起動し、`src/migrations`フォルダ内のマイグレーションファイルに書かれたSQLステートメントでデータベースを初期化します:

```shell
npm run migration:run
```

<details>
<summary><b>期待される実行結果</b></summary>

以下のSQLステートメントは、`players`テーブルと`profiles`テーブルを作成し、2つのテーブルを外部キーで関連付けます。

```sql
query: SELECT VERSION() AS `version`
query: SELECT * FROM `INFORMATION_SCHEMA`.`COLUMNS` WHERE `TABLE_SCHEMA` = 'test' AND `TABLE_NAME` = 'migrations'
query: CREATE TABLE `migrations` (`id` int NOT NULL AUTO_INCREMENT, `timestamp` bigint NOT NULL, `name` varchar(255) NOT NULL, PRIMARY KEY (`id`)) ENGINE=InnoDB
query: SELECT * FROM `test`.`migrations` `migrations` ORDER BY `id` DESC
0 migrations are already loaded in the database.
1 migrations were found in the source code.
1 migrations are new migrations must be executed.
query: START TRANSACTION
query: CREATE TABLE `profiles` (`player_id` int NOT NULL, `biography` text NOT NULL, PRIMARY KEY (`player_id`)) ENGINE=InnoDB
query: CREATE TABLE `players` (`id` int NOT NULL AUTO_INCREMENT, `name` varchar(50) NOT NULL, `coins` decimal NOT NULL, `goods` int NOT NULL, `created_at` datetime NOT NULL, `profilePlayerId` int NULL, UNIQUE INDEX `uk_players_on_name` (`name`), UNIQUE INDEX `REL_b9666644b90ccc5065993425ef` (`profilePlayerId`), PRIMARY KEY (`id`)) ENGINE=InnoDB
```
```
問い合わせ: ALTER TABLE `players` ADD CONSTRAINT `fk_profiles_on_player_id` FOREIGN KEY (`profilePlayerId`) REFERENCES `profiles`(`player_id`) ON DELETE NO ACTION ON UPDATE NO ACTION
問い合わせ: INSERT INTO `test`.`migrations`(`timestamp`, `name`) VALUES (?, ?) -- パラメータ: [1693814724825,"Init1693814724825"]
マイグレーション Init1693814724825 は正常に実行されました。
問い合わせ: COMMIT
```

マイグレーション ファイルは、`src/entities` フォルダで定義されたエンティティから生成されます。TypeORM でエンティティを定義する方法については、[TypeORM: エンティティ](https://typeorm.io/entities)を参照してください。

### ステップ 5: コードを実行し、結果を確認する

次のコマンドを実行して、サンプルコードを実行します。

```shell
npm start
```

**期待される実行結果:**

接続が成功すると、ターミナルに TiDB クラスタのバージョンが次のように出力されます。

```
🔌 TiDB クラスタに接続しました！ (TiDB バージョン: 8.0.11-TiDB-v7.4.0)
🆕 ID 2 の新しいプレイヤーが作成されました。
ℹ️ プレイヤー 2 の情報: Player { id: 2, coins: 100, goods: 100 }
🔢 50 コインと 50 グッズがプレイヤー 2 に追加され、現在、プレイヤー 2 は 100 コインと 150 グッズを持っています。
🚮 1 つのプレイヤーデータが削除されました。
```

## サンプルコードスニペット

次のサンプルコードスニペットを参照して、独自のアプリケーション開発を完了させることができます。

完全なサンプルコードとその実行方法については、[tidb-samples/tidb-nodejs-typeorm-quickstart](https://github.com/tidb-samples/tidb-nodejs-typeorm-quickstart) リポジトリを確認してください。

### 接続オプションを使用して接続する

次のコードは、環境変数で定義されたオプションを使用して TiDB に接続します。

```typescript
// src/dataSource.ts

// .env ファイルから環境変数を読み込む
require('dotenv').config();

export const AppDataSource = new DataSource({
  type: "mysql",
  host: process.env.TIDB_HOST || '127.0.0.1',
  port: process.env.TIDB_PORT ? Number(process.env.TIDB_PORT) : 4000,
  username: process.env.TIDB_USER || 'root',
  password: process.env.TIDB_PASSWORD || '',
  database: process.env.TIDB_DATABASE || 'test',
  ssl: process.env.TIDB_ENABLE_SSL === 'true' ? {
    minVersion: 'TLSv1.2',
    ca: process.env.TIDB_CA_PATH ? fs.readFileSync(process.env.TIDB_CA_PATH) : undefined
  } : null,
  synchronize: process.env.NODE_ENV === 'development',
  logging: false,
  entities: [Player, Profile],
  migrations: [__dirname + "/migrations/**/*{.ts,.js}"],
});
```

> **注意**
>
> TiDB Serverless を使用する場合、パブリックエンドポイントを使用する際には TLS 接続を有効にする必要があります。このサンプルコードでは、`.env` ファイルで環境変数 `TIDB_ENABLE_SSL` を `true` に設定してください。
>
> ただし、TiDB Serverless では `TIDB_CA_PATH` を介して SSL CA 証明書を指定する必要はありません。なぜなら、Node.js はデフォルトで組み込みの [Mozilla CA 証明書](https://wiki.mozilla.org/CA/Included_Certificates) を使用し、TiDB Serverless によって信頼されるからです。

### データを挿入する

次のクエリは、単一の `Player` レコードを作成し、TiDB で生成された `id` フィールドを含む、作成された `Player` オブジェクトを返します。

```typescript
const player = new Player('Alice', 100, 100);
await this.dataSource.manager.save(player);
```

詳細については、[データを挿入する](/develop/dev-guide-insert-data.md)を参照してください。

### データをクエリする

次のクエリは、ID 101 の単一の `Player` オブジェクトを返し、レコードが見つからない場合は `null` を返します。

```typescript
const player: Player | null = await this.dataSource.manager.findOneBy(Player, {
  id: id
});
```

詳細については、[データをクエリする](/develop/dev-guide-get-data-from-single-table.md)を参照してください。

### データを更新する

次のクエリは、ID 101 の `Player` に `50` グッズを追加します。

```typescript
const player = await this.dataSource.manager.findOneBy(Player, {
  id: 101
});
player.goods += 50;
await this.dataSource.manager.save(player);
```

詳細については、[データを更新する](/develop/dev-guide-update-data.md)を参照してください。

### データを削除する

次のクエリは、ID 101 の `Player` を削除します。

```typescript
await this.dataSource.manager.delete(Player, {
  id: 101
});
```

詳細については、[データを削除する](/develop/dev-guide-delete-data.md)を参照してください。

### 生の SQL クエリを実行する

次のクエリは、生の SQL ステートメント (`SELECT VERSION() AS tidb_version;`) を実行し、TiDB クラスタのバージョンを返します。

```typescript
const rows = await dataSource.query('SELECT VERSION() AS tidb_version;');
console.log(rows[0]['tidb_version']);
```

詳細については、[TypeORM: データソース API](https://typeorm.io/data-source-api)を参照してください。

## 便利な注意事項

### 外部キー制約

[外部キー制約](https://docs.pingcap.com/tidb/stable/foreign-key)（実験的）を使用すると、データベース側でチェックを追加することで、データの[参照整合性](https://en.wikipedia.org/wiki/Referential_integrity)を確保できます。ただし、これは大量のデータがあるシナリオでは重大なパフォーマンスの問題を引き起こす可能性があります。

エンティティ間のリレーションシップを構築する際に、`createForeignKeyConstraints` オプション（デフォルト値は `true`）を使用して、外部キー制約を作成するかどうかを制御できます。

```typescript
@Entity()
export class ActionLog {
    @PrimaryColumn()
    id: number

    @ManyToOne((type) => Person, {
        createForeignKeyConstraints: false,
    })
    person: Person
}
```

詳細については、[TypeORM FAQ](https://typeorm.io/relations-faq#avoid-foreign-key-constraint-creation)および[外部キー制約](https://docs.pingcap.com/tidbcloud/foreign-key#foreign-key-constraints)を参照してください。

## 次の手順

- [TypeORM のドキュメント](https://typeorm.io/)から TypeORM のさらなる使用方法を学ぶ。
- [開発者ガイド](/develop/dev-guide-overview.md)の章を通じて、TiDB アプリケーション開発のベストプラクティスを学ぶ。たとえば、[データを挿入する](/develop/dev-guide-insert-data.md)、[データを更新する](/develop/dev-guide-update-data.md)、[データを削除する](/develop/dev-guide-delete-data.md)、[データをクエリする](/develop/dev-guide-get-data-from-single-table.md)、[トランザクション](/develop/dev-guide-transaction-overview.md)、[SQL パフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)など。
- [TiDB 開発者コース](https://www.pingcap.com/education/)を通じて学び、試験に合格して[TiDB 認定資格](https://www.pingcap.com/education/certification/)を取得する。

## ヘルプが必要ですか？

<CustomContent platform="tidb">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](/support.md)してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

[Discord](https://discord.gg/DQZ2dy3cuc?utm_source=doc)で質問するか、[サポートチケットを作成](https://support.pingcap.com/)してください。

</CustomContent>
```