---
title: 2次インデックスの作成
summary: 2次インデックスの作成の手順、ルール、および例について学びます。

# 2次インデックスの作成

このドキュメントでは、SQLとさまざまなプログラミング言語を使用して2次インデックスを作成する手順とそのルールについて説明します。このドキュメントでは、[Bookshop](/develop/dev-guide-bookshop-schema-design.md)アプリケーションを例に取り、2次インデックスの作成手順を説明します。

## 開始する前に

2次インデックスを作成する前に、次の作業を行ってください。

- [TiDB Serverless Clusterを構築](/develop/dev-guide-build-cluster-in-cloud.md)します。
- [スキーマ設計の概要](/develop/dev-guide-schema-design-overview.md)をお読みください。
- [データベースを作成](/develop/dev-guide-create-database.md)します。
- [テーブルを作成](/develop/dev-guide-create-table.md)します。

## 2次インデックスとは

2次インデックスは、TiDBクラスター内の論理オブジェクトです。TiDBはクエリのパフォーマンスを向上させるために使用するデータソートの一種として簡単に見なすことができます。TiDBでは、2次インデックスの作成はオンライン操作であり、テーブル上のデータの読み取りおよび書き込み操作をブロックしません。各インデックスに対して、TiDBはテーブル内の各行の参照を作成し、データそのものではなく選択した列で参照をソートします。

<CustomContent platform="tidb">

2次インデックスに関する詳細については、[2次インデックス](/best-practices/tidb-best-practices.md#secondary-index)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

2次インデックスに関する詳細については、[2次インデックス](https://docs.pingcap.com/tidb/stable/tidb-best-practices#secondary-index)を参照してください。

</CustomContent>

TiDBでは、[既存のテーブルに2次インデックスを追加](#既存のテーブルに2次インデックスを追加)するか、[新しいテーブルを作成する際に2次インデックスを作成](#新しいテーブルを作成する際に2次インデックスを作成)することができます。

## 既存のテーブルに2次インデックスを追加

既存のテーブルに2次インデックスを追加するには、次のように[CREATE INDEX](/sql-statements/sql-statement-create-index.md)ステートメントを使用します。

```sql
CREATE INDEX {index_name} ON {table_name} ({column_names});
```

パラメータの説明：

- `{index_name}`: 2次インデックスの名前。
- `{table_name}`: テーブル名。
- `{column_names}`: インデックス化する列の名前。セミコロンとカンマで区切ります。

## 新しいテーブルを作成する際に2次インデックスを作成

テーブル作成時に同時に2次インデックスを作成するには、[CREATE TABLE](/sql-statements/sql-statement-create-table.md)ステートメントの末尾に`KEY`キーワードを含む節を追加します。

```sql
KEY `{index_name}` (`{column_names}`)
```

パラメータの説明：

- `{index_name}`: 2次インデックスの名前。
- `{column_names}`: インデックス化する列の名前。セミコロンとカンマで区切ります。

## 2次インデックスの作成ルール

インデックスの最適な設定方法については、[インデックスのベストプラクティス](/develop/dev-guide-index-best-practice.md)をご覧ください。

## 例

`bookshop`アプリケーションで**特定の年に出版されたすべての書籍を検索**するようにしたいとします。

`books`テーブルのフィールドは次の通りです。

| フィールド名    | タイプ          | フィールドの説明                |
|--------------|---------------|-----------------------------|
| id           | bigint(20)    | 書籍のユニークID               |
| title        | varchar(100)  | 書名                          |
| type         | enum          | 書籍の種類（例: 雑誌、アニメーション、教材） |
| stock        | bigint(20)    | ストック数                      |
| price        | decimal(15,2) | 価格                           |
| published_at | datetime      | 出版日                         |

`books`テーブルは次のSQLステートメントを使用して作成されます。

```sql
CREATE TABLE `bookshop`.`books` (
  `id` bigint(20) AUTO_RANDOM NOT NULL,
  `title` varchar(100) NOT NULL,
  `type` enum('Magazine', 'Novel', 'Life', 'Arts', 'Comics', 'Education & Reference', 'Humanities & Social Sciences', 'Science & Technology', 'Kids', 'Sports') NOT NULL,
  `published_at` datetime NOT NULL,
  `stock` int(11) DEFAULT '0',
  `price` decimal(15,2) DEFAULT '0.0',
  PRIMARY KEY (`id`) CLUSTERED
) DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;
```

**特定の年に出版されたすべての書籍を検索**するために、次のようにSQLステートメントを記述する必要があります。ここでは2022年を例に挙げます。

```sql
SELECT * FROM `bookshop`.`books` WHERE `published_at` >= '2022-01-01 00:00:00' AND `published_at` < '2023-01-01 00:00:00';
```

このSQLステートメントの実行計画を確認するために、[`EXPLAIN`](/sql-statements/sql-statement-explain.md)ステートメントを使用することができます。

```sql
EXPLAIN SELECT * FROM `bookshop`.`books` WHERE `published_at` >= '2022-01-01 00:00:00' AND `published_at` < '2023-01-01 00:00:00';
```

次のは実行計画の例の出力です。

```
+-------------------------+----------+-----------+---------------+--------------------------------------------------------------------------------------------------------------------------+
| id                      | estRows  | task      | access object | operator info                                                                                                            |
+-------------------------+----------+-----------+---------------+--------------------------------------------------------------------------------------------------------------------------+
| TableReader_7           | 346.32   | root      |               | data:Selection_6                                                                                                         |
| └─Selection_6           | 346.32   | cop[tikv] |               | ge(bookshop.books.published_at, 2022-01-01 00:00:00.000000), lt(bookshop.books.published_at, 2023-01-01 00:00:00.000000) |
|   └─TableFullScan_5     | 20000.00 | cop[tikv] | table:books   | keep order:false                                                                                                         |
+-------------------------+----------+-----------+---------------+--------------------------------------------------------------------------------------------------------------------------+
3 rows in set (0.61 sec)
```

出力には、`TableFullScan`という単語が`id`列に表示されており、これはこのクエリでTiDBが`books`テーブルを完全にスキャンする準備ができていることを意味します。しかし、大量のデータの場合、フルテーブルスキャンは非常に遅く、致命的な影響を与える可能性があります。

このような影響を避けるために、`books`テーブルの`published_at`列にインデックスを追加することができます。

```sql
CREATE INDEX `idx_book_published_at` ON `bookshop`.`books` (`bookshop`.`books`.`published_at`);
```

インデックスを追加した後、実行計画を再度確認するために`EXPLAIN`ステートメントを実行します。

次のは実行計画の例の出力です。

```
+-------------------------------+---------+-----------+--------------------------------------------------------+-------------------------------------------------------------------+
| id                            | estRows | task      | access object                                          | operator info                                                     |
+-------------------------------+---------+-----------+--------------------------------------------------------+-------------------------------------------------------------------+
| IndexLookUp_10                | 146.01  | root      |                                                        |                                                                   |
| ├─IndexRangeScan_8(Build)     | 146.01  | cop[tikv] | table:books, index:idx_book_published_at(published_at) | range:[2022-01-01 00:00:00,2023-01-01 00:00:00), keep order:false |
| └─TableRowIDScan_9(Probe)     | 146.01  | cop[tikv] | table:books                                            | keep order:false                                                  |
+-------------------------------+---------+-----------+--------------------------------------------------------+-------------------------------------------------------------------+
3 rows in set (0.18 sec)
```

出力には、**IndexRangeScan**が表示されており、**TableFullScan**の代わりにインデックスを使用してクエリを実行する準備ができていることを意味します。

実行計画に表示される**TableFullScan**や**IndexRangeScan**などの単語は、TiDBでの[オペレータ](/explain-overview.md#operator-overview)です。実行計画やオペレータに関する詳細については、[TiDBクエリ実行計画の概要](/explain-overview.md)を参照してください。

<CustomContent platform="tidb">

実行計画は常に同じオペレータを返しません。これは、TiDBが**コストベースの最適化（CBO）**アプローチを使用しており、実行計画がルールとデータの分散に依存するためです。TiDB SQLのパフォーマンスに関する詳細については、[SQLチューニングの概要](/sql-tuning-overview.md)をご覧ください。

</CustomContent>

<CustomContent platform="tidb-cloud">

実行計画は常に同じオペレータを返しません。これは、TiDBが**コストベースの最適化（CBO）**アプローチを使用しており、実行計画がルールとデータの分散に依存するためです。TiDB SQLのパフォーマンスに関する詳細については、[SQLチューニングの概要](/tidb-cloud/tidb-cloud-sql-tuning-overview.md)をご覧ください。

</CustomContent>

> **注意:**
```sql
テーブルでのインデックスのクエリをするには、[SHOW INDEXES](/sql-statements/sql-statement-show-indexes.md) ステートメントを使用できます：

```sql
SHOW INDEXES FROM `bookshop`.`books`;
```

以下は出力の例です：

```
+-------+------------+-----------------------+--------------+--------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+-----------+
| Table | Non_unique | Key_name              | Seq_in_index | Column_name  | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment | Visible | Expression | Clustered |
+-------+------------+-----------------------+--------------+--------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+-----------+
| books |          0 | PRIMARY               |            1 | id           | A         |           0 |     NULL | NULL   |      | BTREE      |         |               | YES     | NULL       | YES       |
| books |          1 | idx_book_published_at |            1 | published_at | A         |           0 |     NULL | NULL   |      | BTREE      |         |               | YES     | NULL       | NO        |
+-------+------------+-----------------------+--------------+--------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+-----------+
2 行が選択されました (1.63 秒)
```

## 次のステップ

データベースを作成し、テーブルにセカンダリインデックスを追加した後、アプリケーションにデータの[書き込み](/develop/dev-guide-insert-data.md)および[読み取り](/develop/dev-guide-get-data-from-single-table.md)機能を追加できます。
```