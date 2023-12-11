```yaml
---
title: SQL パフォーマンスチューニング
summary: TiDBのSQLパフォーマンスチューニングスキームと解析方法を紹介します。
---

# SQL パフォーマンスチューニング

このドキュメントでは、遅いSQLステートメントの一般的な原因とSQLパフォーマンスのチューニング技術について紹介します。

## 開始する前に

[`tiup demo` import](/develop/dev-guide-bookshop-schema-design.md#method-1-via-tiup-demo) を使用してデータを準備できます。

```shell
tiup demo bookshop prepare --host 127.0.0.1 --port 4000 --books 1000000
```

または、TiDB CloudのImport機能を使用して事前に準備されたサンプルデータをインポートできます。

## 問題: フルテーブルスキャン

遅いSQLクエリの最も一般的な原因は、`SELECT` ステートメントがフルテーブルスキャンを実行したり、誤ったインデックスを使用したりすることです。

TiDBは、主キーではないまたはセカンダリインデックスに存在しない列を基準として大きなテーブルから少数の行を取得する場合、通常パフォーマンスが低くなります。

```sql
SELECT * FROM books WHERE title = 'Marian Yost';
```

```sql
+------------+-------------+-----------------------+---------------------+-------+--------+
| id         | title       | type                  | published_at        | stock | price  |
+------------+-------------+-----------------------+---------------------+-------+--------+
| 65670536   | Marian Yost | Arts                  | 1950-04-09 06:28:58 | 542   | 435.01 |
| 1164070689 | Marian Yost | Education & Reference | 1916-05-27 12:15:35 | 216   | 328.18 |
| 1414277591 | Marian Yost | Arts                  | 1932-06-15 09:18:14 | 303   | 496.52 |
| 2305318593 | Marian Yost | Arts                  | 2000-08-15 19:40:58 | 398   | 402.90 |
| 2638226326 | Marian Yost | Sports                | 1952-04-02 12:40:37 | 191   | 174.64 |
+------------+-------------+-----------------------+---------------------+-------+--------+
5 rows in set
Time: 0.582s
```

このクエリが遅い理由を理解するために、`EXPLAIN` を使用して実行計画を確認できます。

```sql
EXPLAIN SELECT * FROM books WHERE title = 'Marian Yost';
```

```sql
+---------------------+------------+-----------+---------------+-----------------------------------------+
| id                  | estRows    | task      | access object | operator info                           |
+---------------------+------------+-----------+---------------+-----------------------------------------+
| TableReader_7       | 1.27       | root      |               | data:Selection_6                        |
| └─Selection_6       | 1.27       | cop[tikv] |               | eq(bookshop.books.title, "Marian Yost") |
|   └─TableFullScan_5 | 1000000.00 | cop[tikv] | table:books   | keep order:false                        |
+---------------------+------------+-----------+---------------+-----------------------------------------+
```

実行計画の`TableFullScan_5`からは、TiDBが`books`テーブルでフルテーブルスキャンを実行し、各行が`title`条件を満たすかどうかをチェックしていることがわかります。`TableFullScan_5`の`estRows`値は`1000000.00`で、このフルテーブルスキャンが`1000000.00`行のデータを読むとオプティマイザが推定していることを意味します。

`EXPLAIN`の使用方法の詳細については、[`EXPLAIN`の概要](/explain-walkthrough.md)を参照してください。

### 解決策: セカンダリインデックスを使用

上記のクエリを高速化するために、`books.title`列にセカンダリインデックスを追加します。

```sql
CREATE INDEX title_idx ON books (title);
```

クエリの実行ははるかに速くなります。

```sql
SELECT * FROM books WHERE title = 'Marian Yost';
```

```sql
+------------+-------------+-----------------------+---------------------+-------+--------+
| id         | title       | type                  | published_at        | stock | price  |
+------------+-------------+-----------------------+---------------------+-------+--------+
| 1164070689 | Marian Yost | Education & Reference | 1916-05-27 12:15:35 | 216   | 328.18 |
| 1414277591 | Marian Yost | Arts                  | 1932-06-15 09:18:14 | 303   | 496.52 |
| 2305318593 | Marian Yost | Arts                  | 2000-08-15 19:40:58 | 398   | 402.90 |
| 2638226326 | Marian Yost | Sports                | 1952-04-02 12:40:37 | 191   | 174.64 |
| 65670536   | Marian Yost | Arts                  | 1950-04-09 06:28:58 | 542   | 435.01 |
+------------+-------------+-----------------------+---------------------+-------+--------+
5 rows in set
Time: 0.007s
```

パフォーマンスが向上した理由を理解するために、新しい実行計画を確認するために`EXPLAIN`を使用します。

```sql
EXPLAIN SELECT * FROM books WHERE title = 'Marian Yost';
```

```sql
+---------------------------+---------+-----------+-------------------------------------+-------------------------------------------------------+
| id                        | estRows | task      | access object                       | operator info                                         |
+---------------------------+---------+-----------+-------------------------------------+-------------------------------------------------------+
| IndexLookUp_10            | 1.27    | root      |                                     |                                                       |
| ├─IndexRangeScan_8(Build) | 1.27    | cop[tikv] | table:books, index:title_idx(title) | range:["Marian Yost","Marian Yost"], keep order:false |
| └─TableRowIDScan_9(Probe) | 1.27    | cop[tikv] | table:books                         | keep order:false                                      |
+---------------------------+---------+-----------+-------------------------------------+-------------------------------------------------------+
```

実行計画の`IndexLookup_10`からは、TiDBが`title_idx`インデックスを使用してデータをクエリしていることがわかります。`estRows`値が`1.27`であり、オプティマイザが推定しているスキャンされる推定行数は、フルテーブルスキャンの`1000000.00`行のデータよりもはるかに少ないです。

`IndexLookup_10`実行計画は、まず`IndexRangeScan_8`オペレータを使用して`title_idx`インデックスを介して条件を満たすインデックスデータを読み取り、その後`TableRowIDScan_9`オペレータを使用してインデックスデータに格納されている行IDに従って対応する行をクエリすることです。

TiDB実行計画の詳細については、[TiDBクエリ実行プランの概要](/explain-overview.md)を参照してください。

### 解決策: カバリングインデックスを使用

インデックスがカバリングインデックスである場合、SQLステートメントでクエリされるすべての列を含んでおり、インデックスデータのスキャンがクエリに十分です。

たとえば、次のクエリでは、`title`に基づいて対応する`price`のみをクエリする必要があります。

```sql
SELECT title, price FROM books WHERE title = 'Marian Yost';
```

```sql
+-------------+--------+
| title       | price  |
+-------------+--------+
| Marian Yost | 435.01 |
| Marian Yost | 328.18 |
| Marian Yost | 496.52 |
| Marian Yost | 402.90 |
| Marian Yost | 174.64 |
+-------------+--------+
5 rows in set
Time: 0.007s
```

`title_idx`インデックスには`title`列のデータしか含まれていないため、TiDBはまずインデックスデータをスキャンし、次にテーブルから`price`列をクエリする必要があります。

```sql
EXPLAIN SELECT title, price FROM books WHERE title = 'Marian Yost';
```

```sql
+---------------------------+---------+-----------+-------------------------------------+-------------------------------------------------------+
| id                        | estRows | task      | access object                       | operator info                                         |
+---------------------------+---------+-----------+-------------------------------------+-------------------------------------------------------+
| IndexLookUp_10            | 1.27    | root      |                                     |                                                       |
| ├─IndexRangeScan_8(Build) | 1.27    | cop[tikv] | table:books, index:title_idx(title) | range:["Marian Yost","Marian Yost"], keep order:false |
| └─TableRowIDScan_9(Probe) | 1.27    | cop[tikv] | table:books                         | keep order:false                                      |
+---------------------------+---------+-----------+-------------------------------------+-------------------------------------------------------+
```

パフォーマンスを最適化するには、`title_idx`インデックスを削除し、新しいカバリングインデックス`title_price_idx`を作成します。

```sql
ALTER TABLE books DROP INDEX title_idx;
```

```sql
CREATE INDEX title_price_idx ON books (title, price);
```
```
```sql
'price'データは'title_price_idx'インデックスに格納されているため、次のクエリはインデックスデータのスキャンのみが必要です：

```sql
EXPLAIN SELECT title, price FROM books WHERE title = 'Marian Yost';
```

```sql
--------------------+---------+-----------+--------------------------------------------------+-------------------------------------------------------+
| id                 | estRows | task      | access object                                    | operator info                                         |
+--------------------+---------+-----------+--------------------------------------------------+-------------------------------------------------------+
| IndexReader_6      | 1.27    | root      |                                                  | index:IndexRangeScan_5                                |
| └─IndexRangeScan_5 | 1.27    | cop[tikv] | table:books, index:title_price_idx(title, price) | range:["Marian Yost","Marian Yost"], keep order:false |
+--------------------+---------+-----------+--------------------------------------------------+-------------------------------------------------------+
```

このクエリは次のように高速に実行されます：

```sql
SELECT title, price FROM books WHERE title = 'Marian Yost';
```

```sql
+-------------+--------+
| title       | price  |
+-------------+--------+
| Marian Yost | 174.64 |
| Marian Yost | 328.18 |
| Marian Yost | 402.90 |
| Marian Yost | 435.01 |
| Marian Yost | 496.52 |
+-------------+--------+
5 rows in set
Time: 0.004s
```

以降の例で`books`テーブルを使用するため、`title_price_idx`インデックスを削除します：

```sql
ALTER TABLE books DROP INDEX title_price_idx;
```

### ソリューション：プライマリインデックスを使用

クエリがプライマリキーを使用してデータをフィルタリングする場合、クエリの実行速度が速くなります。たとえば、`books`テーブルのプライマリキーは`id`カラムなので、`id`カラムを使用してデータをクエリできます：

```sql
SELECT * FROM books WHERE id = 896;
```

```sql
+-----+----------------+----------------------+---------------------+-------+--------+
| id  | title          | type                 | published_at        | stock | price  |
+-----+----------------+----------------------+---------------------+-------+--------+
| 896 | Kathryne Doyle | Science & Technology | 1969-03-18 01:34:15 | 468   | 281.32 |
+-----+----------------+----------------------+---------------------+-------+--------+
1 row in set
Time: 0.004s
```

実行計画を確認するには`EXPLAIN`を使用します：

```sql
EXPLAIN SELECT * FROM books WHERE id = 896;
```

```sql
+-------------+---------+------+---------------+---------------+
| id          | estRows | task | access object | operator info |
+-------------+---------+------+---------------+---------------+
| Point_Get_1 | 1.00    | root | table:books   | handle:896    |
+-------------+---------+------+---------------+---------------+
```

`Point_Get`は非常に速い実行計画です。

## 適切な結合タイプを使用する

[JOIN Execution Plan](/explain-joins.md)を参照してください。

### 関連情報

* [EXPLAIN Walkthrough](/explain-walkthrough.md)
* [Explain Statements That Use Indexes](/explain-indexes.md)
```