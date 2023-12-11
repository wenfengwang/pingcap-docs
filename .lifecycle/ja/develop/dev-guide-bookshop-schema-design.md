---
title: ブックショップのサンプルアプリケーション
---

# ブックショップのサンプルアプリケーション

ブックショップは、さまざまなカテゴリの本を購入し、読んだ本に評価を付けることができる仮想オンライン書店アプリケーションです。

アプリケーション開発ガイド上の閲覧をスムーズにするために、ブックショップアプリケーションの[テーブル構造](#description-of-the-tables)とデータに基づくSQLステートメントの例を提供します。この文書では、テーブル構造のインポート方法とデータの定義に焦点を当てています。

## テーブル構造とデータのインポート

<CustomContent platform="tidb">

ブックショップのテーブル構造とデータは、[TiUPを介して](#method-1-via-tiup-demo)または[TiDB Cloudのインポート機能を介して](#method-2-via-tidb-cloud-import)インポートできます。

</CustomContent>

<CustomContent platform="tidb-cloud">

TiDB Cloudでは、[Method 1: Via `tiup demo`](#method-1-via-tiup-demo)をスキップして、TiDB Cloudのインポート機能を使用して[ブックショップのテーブル構造をインポート](#method-2-via-tidb-cloud-import)できます。

</CustomContent>

### 方法1：`tiup demo`を使用

<CustomContent platform="tidb">

TiUPを使用してTiDBクラスタを展開した場合、またはTiDBサーバに接続できる場合は、次のコマンドを実行することでブックショップアプリケーションのサンプルデータを迅速に生成およびインポートできます。

</CustomContent>

<CustomContent platform="tidb-cloud">

TiUPを使用してTiDBクラスタを展開した場合、またはTiDBサーバに接続できる場合は、次のコマンドを実行することでブックショップアプリケーションのサンプルデータを迅速に生成およびインポートできます。

</CustomContent>

```shell
tiup demo bookshop prepare
```

デフォルトでは、このコマンドにより、アプリケーションはアドレス`127.0.0.1`のポート`4000`に接続できるようになり、パスワードなしで`root`ユーザーとしてログインでき、データベース名`bookshop`に[テーブル構造](#description-of-the-tables)が作成されます。

#### 接続情報の設定

以下の表には接続パラメータがリストされています。デフォルトの設定を変更して環境に合わせることができます。

| パラメータ       | 省略形 | デフォルト値 | 説明                   |
| -------------- | ---- | ----------- | -------------------- |
| `--password`   | `-p` | なし        | データベースユーザーのパスワード |
| `--host`       | `-H` | `127.0.0.1` | データベースアドレス      |
| `--port`       | `-P` | `4000`      | データベースポート      |
| `--db`         | `-D` | `bookshop`  | データベース名          |
| `--user`       | `-U` | `root`      | データベースユーザー      |

例えば、TiDB Cloud上のデータベースに接続する場合、次のように接続情報を指定できます。

```shell
tiup demo bookshop prepare -U <username> -H <endpoint> -P 4000 -p <password>
```

#### データの容量を設定

各データベーステーブルで生成されたデータの容量を指定するには、以下のパラメータを設定します。

| パラメータ       | デフォルト値 | 説明                                   |
| ----------- | -------- | ------------------------------------ |
| `--users`   | `10000`  | `users`テーブルに生成されるデータの行数            |
| `--authors` | `20000`  | `authors`テーブルに生成されるデータの行数          |
| `--books`   | `20000`  | `books`テーブルに生成されるデータの行数            |
| `--orders`  | `300000` | `orders`テーブルに生成されるデータの行数          |
| `--ratings` | `300000` | `ratings`テーブルに生成されるデータの行数         |

例えば、次のコマンドを実行すると、次のように生成されます。

- `--users`パラメータを通じてユーザ情報の20万行
- `--books`パラメータを通じて本情報の50万行
- `--authors`パラメータを通じて著者情報の10万行
- `--ratings`パラメータを通じて評価情報の100万行
- `--orders`パラメータを通じて注文情報の100万行

```shell
tiup demo bookshop prepare --users=200000 --books=500000 --authors=100000 --ratings=1000000 --orders=1000000 --drop-tables
```

`--drop-tables`パラメータを使用すると、元のテーブル構造を削除できます。より詳細なパラメータの説明については、`tiup demo bookshop --help`コマンドを実行してください。

### 方法2：TiDB Cloudのインポートを使用

TiDB Cloudのクラスタ詳細ページで、**Import**エリアの**Import Data**をクリックして**Data Import**ページに移動します。このページでは、AWS S3からTiDB Cloudにブックショップのサンプルデータをインポートするための以下の手順を実行します。

1. **Data Format**で**SQL File**を選択します。
2. 次の**Bucket URI**および**Role ARN**を対応する入力欄にコピーします。

    **Bucket URI**:

    ```
    s3://developer.pingcap.com/bookshop/
    ```

   **Role ARN**:

    ```
    arn:aws:iam::494090988690:role/s3-tidb-cloud-developer-access
    ```

3. 次へをクリックして**File and filter**ステップに進み、インポートするファイルの情報を確認します。

4. 再度次へをクリックして**Preview**ステップに進み、インポートするデータのプレビューを確認します。

    この例では、事前に次のデータが生成されています。

    - ユーザ情報20万行
    - 本情報50万行
    - 著者情報10万行
    - 評価情報100万行
    - 注文情報100万行

5. **Start Import**をクリックしてインポートプロセスを開始し、TiDB Cloudがインポートを完了するのを待ちます。

TiDB Cloudにデータをインポートまたは移行する方法の詳細については、[TiDB Cloud Migration Overview](https://docs.pingcap.com/tidbcloud/tidb-cloud-migration-overview)を参照してください。

### データインポートのステータスの表示

インポートが完了すると、次のSQLステートメントを実行して、各テーブルのデータ容量情報を表示できます。

```sql
SELECT
    CONCAT(table_schema,'.',table_name) AS 'テーブル名',
    table_rows AS '行数',
    CONCAT(ROUND(data_length/(1024*1024*1024),4),'G') AS 'データサイズ',
    CONCAT(ROUND(index_length/(1024*1024*1024),4),'G') AS 'インデックスサイズ',
    CONCAT(ROUND((data_length+index_length)/(1024*1024*1024),4),'G') AS '合計'
FROM
    information_schema.TABLES
WHERE table_schema LIKE 'bookshop';
```

結果は次のようになります。

```
+-----------------------+----------------+-----------+------------+---------+
| テーブル名               | 行数             | データサイズ | インデックスサイズ | 合計      |
+-----------------------+----------------+-----------+------------+---------+
| bookshop.orders       |        1000000 | 0.0373G   | 0.0075G    | 0.0447G |
| bookshop.book_authors |        1000000 | 0.0149G   | 0.0149G    | 0.0298G |
| bookshop.ratings      |        4000000 | 0.1192G   | 0.1192G    | 0.2384G |
| bookshop.authors      |         100000 | 0.0043G   | 0.0000G    | 0.0043G |
| bookshop.users        |         195348 | 0.0048G   | 0.0021G    | 0.0069G |
| bookshop.books        |        1000000 | 0.0546G   | 0.0000G    | 0.0546G |
+-----------------------+----------------+-----------+------------+---------+
6 行が返されました (0.03 秒)
```

## テーブルの説明

このセクションでは、ブックショップアプリケーションのデータベーステーブルについて詳細に説明します。

### `books`テーブル

このテーブルは本の基本情報を保管しています。

| フィールド名       | タイプ         | 説明                 |
|------------------|---------------|----------------------|
| id               | bigint(20)    | 本のユニークID         |
| title            | varchar(100)  | 本のタイトル           |
| type             | enum          | 本の種類（例: 雑誌、アニメーション、教材）   |
| stock            | bigint(20)    | 在庫                  |
| price            | decimal(15,2) | 価格                  |
| published_at     | datetime      | 公開日                |

### `authors`テーブル

このテーブルは著者の基本情報を保管しています。

| フィールド名 | タイプ          | 説明                           |
|--------------|---------------|-------------------------------|
| id           | bigint(20)   | 著者のユニークID             |
| name         | varchar(100) | 著者の氏名                      |
| gender       | tinyint(1)   | 生物学的性別（0：女性、1：男性、NULL：不明） |
| birth_year   | smallint(6)  | 誕生年                        |
| death_year   | smallint(6)  | 死亡年                        |

### `users` テーブル

このテーブルは、書店のユーザー情報を保存します。

| フィールド名 | タイプ            | 説明       |
|--------------|-----------------|----------|
| id           | bigint(20)      | ユーザーのユニークID |
| balance      | decimal(15,2)   | 残高        |
| nickname     | varchar(100)    | ニックネーム |

### `ratings` テーブル

このテーブルは、ユーザーによる本の評価の記録を保存します。

| フィールド名 | タイプ      | 説明                        |
|--------------|-----------|----------------------------|
| book_id      | bigint    | 本のユニークID（[books](#books-table)にリンク） |
| user_id      | bigint    | ユーザーのユニーク識別子（[users](#users-table)にリンク） |
| score        | tinyint   | ユーザーの評価（1〜5）            |
| rated_at     | datetime  | 評価時刻                    |

### `book_authors` テーブル

著者は複数の本を執筆する場合があり、1つの本には複数の著者が関与する場合があります。このテーブルは、本と著者の対応関係を保存します。

| フィールド名 | タイプ        | 説明                           |
|--------------|-------------|-------------------------------|
| book_id      | bigint(20)  | 本のユニークID（[books](#books-table)にリンク）  |
| author_id    | bigint(20)  | 著者のユニークID（[authors](#authors-table)にリンク） |

### `orders` テーブル

このテーブルは、ユーザーの購入情報を保存します。

| フィールド名 | タイプ        | 説明                                                     |
|--------------|-------------|---------------------------------------------------------|
| id           | bigint(20)  | 注文のユニークID                                       |
| book_id      | bigint(20)  | 本のユニークID（[books](#books-table)にリンク）          |
| user_id      | bigint(20)  | ユーザーのユニーク識別子（[users](#users-table)と関連付け） |
| quantity     | tinyint(4)  | 購入数量                                               |
| ordered_at   | datetime    | 購入時刻                                               |

## データベース初期化スクリプト `dbinit.sql`

Bookshopアプリケーションでデータベーステーブル構造を手動で作成する場合は、次のSQLステートメントを実行してください。

```sql
CREATE DATABASE IF NOT EXISTS `bookshop`;

DROP TABLE IF EXISTS `bookshop`.`books`;
CREATE TABLE `bookshop`.`books` (
  `id` bigint(20) AUTO_RANDOM NOT NULL,
  `title` varchar(100) NOT NULL,
  `type` enum('Magazine', 'Novel', 'Life', 'Arts', 'Comics', 'Education & Reference', 'Humanities & Social Sciences', 'Science & Technology', 'Kids', 'Sports') NOT NULL,
  `published_at` datetime NOT NULL,
  `stock` int(11) DEFAULT '0',
  `price` decimal(15,2) DEFAULT '0.0',
  PRIMARY KEY (`id`) CLUSTERED
) DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;

DROP TABLE IF EXISTS `bookshop`.`authors`;
CREATE TABLE `bookshop`.`authors` (
  `id` bigint(20) AUTO_RANDOM NOT NULL,
  `name` varchar(100) NOT NULL,
  `gender` tinyint(1) DEFAULT NULL,
  `birth_year` smallint(6) DEFAULT NULL,
  `death_year` smallint(6) DEFAULT NULL,
  PRIMARY KEY (`id`) CLUSTERED
) DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;

DROP TABLE IF EXISTS `bookshop`.`book_authors`;
CREATE TABLE `bookshop`.`book_authors` (
  `book_id` bigint(20) NOT NULL,
  `author_id` bigint(20) NOT NULL,
  PRIMARY KEY (`book_id`,`author_id`) CLUSTERED
) DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;

DROP TABLE IF EXISTS `bookshop`.`ratings`;
CREATE TABLE `bookshop`.`ratings` (
  `book_id` bigint NOT NULL,
  `user_id` bigint NOT NULL,
  `score` tinyint NOT NULL,
  `rated_at` datetime NOT NULL DEFAULT NOW() ON UPDATE NOW(),
  PRIMARY KEY (`book_id`,`user_id`) CLUSTERED,
  UNIQUE KEY `uniq_book_user_idx` (`book_id`,`user_id`)
) DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;
ALTER TABLE `bookshop`.`ratings` SET TIFLASH REPLICA 1;

DROP TABLE IF EXISTS `bookshop`.`users`;
CREATE TABLE `bookshop`.`users` (
  `id` bigint AUTO_RANDOM NOT NULL,
  `balance` decimal(15,2) DEFAULT '0.0',
  `nickname` varchar(100) UNIQUE NOT NULL,
  PRIMARY KEY (`id`)
) DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;

DROP TABLE IF EXISTS `bookshop`.`orders`;
CREATE TABLE `bookshop`.`orders` (
  `id` bigint(20) AUTO_RANDOM NOT NULL,
  `book_id` bigint(20) NOT NULL,
  `user_id` bigint(20) NOT NULL,
  `quality` tinyint(4) NOT NULL,
  `ordered_at` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`) CLUSTERED,
  KEY `orders_book_id_idx` (`book_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin
```