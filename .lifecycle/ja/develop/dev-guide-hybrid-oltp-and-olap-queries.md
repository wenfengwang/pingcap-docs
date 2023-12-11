---
title: HTAPクエリ
summary: TiDBでのHTAPクエリを紹介します。

# HTAPクエリ

HTAPはHybrid Transactional and Analytical Processingの略です。従来、データベースはしばしばトランザクショナルまたは解析シナリオ向けに設計されているため、データプラットフォームはしばしばトランザクション処理と解析処理に分割され、データは解析クエリに迅速に応答するためにトランザクションデータベースから解析データベースにレプリケーションされる必要があります。TiDBデータベースはトランザクションおよび解析タスクの両方を実行できるため、データプラットフォームの構築を大幅に簡略化し、ユーザーがより新しいデータを分析に使用できるようにしています。

TiDBはOnline Transactional Processing（OLTP）用の行ベースのストレージエンジンであるTiKVと、Online Analytical Processing（OLAP）用の列ベースのストレージエンジンであるTiFlashを使用しています。行ベースのストレージエンジンと列ベースのストレージエンジンはHTAP向けに共存しています。両方のストレージエンジンはデータを自動的にレプリケートし、強い整合性を維持します。行ベースのストレージエンジンはOLTPのパフォーマンスを最適化し、列ベースのストレージエンジンはOLAPのパフォーマンスを最適化します。

[テーブルの作成](/develop/dev-guide-create-table.md#use-htap-capabilities)セクションでは、TiDBのHTAP機能を有効にする方法が紹介されています。次に、HTAPを使用してデータをより速く分析する方法について説明します。

## データ準備

開始する前に、`tiup demo`コマンドを使用してさらにサンプルデータをインポートできます（/develop/dev-guide-bookshop-schema-design.md#method-1-via-tiup-demo 参照）。例:

```shell
tiup demo bookshop prepare --users=200000 --books=500000 --authors=100000 --ratings=1000000 --orders=1000000 --host 127.0.0.1 --port 4000 --drop-tables
```

または、TiDB CloudのImport機能を使用して事前に準備されたサンプルデータをインポートできます。

## ウィンドウ関数

データベースを使用する際、データを格納するだけでなく、（例：書籍の順序付けや評価などの）アプリケーション機能を提供することもあります。また、データを解析してさらなる操作や決定を行う必要がある場合もあります。

単一のテーブルからデータをクエリ（Select）するドキュメント（/develop/dev-guide-get-data-from-single-table.md）は、データを全体として分析するために集約クエリを使用する方法について紹介しています。より複雑なシナリオでは、複数の集計クエリの結果を単一のクエリに集約することができます。特定の書籍の注文金額の過去のトレンドを知りたい場合には、各月のすべての注文データを`sum`してから、その`sum`の結果を集約することで過去のトレンドを取得できます。

そのような分析を容易にするために、TiDB v3.0以降では、TiDBはウィンドウ関数をサポートしています。この関数はデータの各行に対して複数の行にわたるデータにアクセスする機能を提供します。通常の集計クエリとは異なり、ウィンドウ関数は結果セットを単一の行にマージせずに行を集約します。

集約関数と同様に、ウィンドウ関数を使用する際には特定の構文に従う必要があります:

```sql
SELECT
    window_function() OVER ([partition_clause] [order_clause] [frame_clause]) AS alias
FROM
    table_name
```

### `ORDER BY`句

集約ウィンドウ関数`sum()`を使用すると、特定の書籍の注文金額の過去のトレンドを分析できます。例えば:

```sql
WITH orders_group_by_month AS (
  SELECT DATE_FORMAT(ordered_at, '%Y-%c') AS month, COUNT(*) AS orders
  FROM orders
  WHERE book_id = 3461722937
  GROUP BY 1
)
SELECT
month,
SUM(orders) OVER(ORDER BY month ASC) as acc
FROM orders_group_by_month
ORDER BY month ASC;
```

`sum()`関数は`ORDER BY`句で指定された順序でデータを蓄積します。結果は次のようになります:

```
+---------+-------+
| month   | acc   |
+---------+-------+
| 2011-5  |     1 |
| 2011-8  |     2 |
| 2012-1  |     3 |
| 2012-2  |     4 |
| 2013-1  |     5 |
| 2013-3  |     6 |
| 2015-11 |     7 |
| 2015-4  |     8 |
| 2015-8  |     9 |
| 2017-11 |    10 |
| 2017-5  |    11 |
| 2019-5  |    13 |
| 2020-2  |    14 |
+---------+-------+
13 rows in set (0.01 sec)
```

上記のデータを、水平軸を時間として、垂直軸を累積注文金額とする折れ線グラフで可視化すると、書籍の過去の注文トレンドを勾配の変化によって簡単に把握できます。

### `PARTITION BY`句

異なる種類の書籍の過去の注文トレンドを分析し、複数のシリーズで同じ折れ線グラフに可視化したいとします。

`PARTITION BY`句を使用して書籍をタイプごとにグループ化し、各タイプの過去の注文を個別にカウントできます。

```sql
WITH orders_group_by_month AS (
    SELECT
        b.type AS book_type,
        DATE_FORMAT(ordered_at, '%Y-%c') AS month,
        COUNT(*) AS orders
    FROM orders o
    LEFT JOIN books b ON o.book_id = b.id
    WHERE b.type IS NOT NULL
    GROUP BY book_type, month
), acc AS (
    SELECT
        book_type,
        month,
        SUM(orders) OVER(PARTITION BY book_type ORDER BY book_type, month ASC) as acc
    FROM orders_group_by_month
    ORDER BY book_type, month ASC
)
SELECT * FROM acc;
```

結果は次のとおりです:

```
+------------------------------+---------+------+
| book_type                    | month   | acc  |
+------------------------------+---------+------+
| Magazine                     | 2011-10 |    1 |
| Magazine                     | 2011-8  |    2 |
| Magazine                     | 2012-5  |    3 |
| Magazine                     | 2013-1  |    4 |
| Magazine                     | 2013-6  |    5 |
...
| Novel                        | 2011-3  |   13 |
| Novel                        | 2011-4  |   14 |
| Novel                        | 2011-6  |   15 |
| Novel                        | 2011-8  |   17 |
| Novel                        | 2012-1  |   18 |
| Novel                        | 2012-2  |   20 |
...
| Sports                       | 2021-4  |   49 |
| Sports                       | 2021-7  |   50 |
| Sports                       | 2022-4  |   51 |
+------------------------------+---------+------+
1500 rows in set (1.70 sec)
```

### 集約されていないウィンドウ関数

TiDBでは、さらに多くの分析ステートメントに使用するための集約されていない[ウィンドウ関数](/functions-and-operators/window-functions.md)も提供されています。

例えば、[ページネーションクエリ](/develop/dev-guide-paginate-results.md)ドキュメントは、`row_number()`関数を使用して効率的なページネーションバッチ処理を実現する方法について紹介しています。

## ハイブリッドワークロード

ハイブリッド負荷シナリオでTiDBを使用してリアルタイムオンライン解析処理を行う場合、データへのTiDBのエントリーポイントを提供するだけで済みます。TiDBは特定のビジネスに基づいて異なる処理エンジンを自動的に選択します。

### TiFlashレプリカの作成

TiDBはデフォルトで行ベースのストレージエンジンであるTiKVを使用しています。列ベースのストレージエンジンであるTiFlashを使用するには、[HTAP機能を有効にする](/develop/dev-guide-create-table.md#use-htap-capabilities)を参照してください。TiFlashを介してデータをクエリする前に、`books`および`orders`テーブルのためにTiFlashレプリカを作成する必要があります:

```sql
ALTER TABLE books SET TIFLASH REPLICA 1;
ALTER TABLE orders SET TIFLASH REPLICA 1;
```

以下のステートメントを使用してTiFlashレプリカの進捗状況を確認できます:

```sql
SELECT * FROM information_schema.tiflash_replica WHERE TABLE_SCHEMA = 'bookshop' and TABLE_NAME = 'books';
SELECT * FROM information_schema.tiflash_replica WHERE TABLE_SCHEMA = 'bookshop' and TABLE_NAME = 'orders';
```

`PROGRESS`列が1の場合、進捗が100%完了しており、`AVAILABLE`列が1の場合、レプリカが現在利用可能であることを示します。

```
+--------------+------------+----------+---------------+-----------------+-----------+----------+
| TABLE_SCHEMA | TABLE_NAME | TABLE_ID | REPLICA_COUNT | LOCATION_LABELS | AVAILABLE | PROGRESS |
+--------------+------------+----------+---------------+-----------------+-----------+----------+
| bookshop     | books      |      143 |             1 |                 |         1 |        1 |
+--------------+------------+----------+---------------+-----------------+-----------+----------+
1 row in set (0.07 sec)
+--------------+------------+----------+---------------+-----------------+-----------+----------+
```sql
      + {T}
    + COIN {T}
+--------------+------------+----------+---------------+-----------------+-----------+----------+
1 ﾚｳﾗ in set (0.07 sec)
```

  replicasを追加した後、上記のwindow関数[`PARTITION BY` clause](#partition-by-clause)の実行計画を確認するために`EXPLAIN`ステートメントを使用できます。実行計画に`cop[tiflash]`が表示されると、TiFlashエンジンが動作し始めたことを意味します。

次に、[`PARTITION BY` clause](#partition-by-clause)のサンプルSQLステートメントを再度実行します。結果は次のようになります：

```
+------------------------------+---------+------+
| book_type                    | month   | acc  |
+------------------------------+---------+------+
| Magazine                     | 2011-10 |    1 |
| Magazine                     | 2011-8  |    2 |
| Magazine                     | 2012-5  |    3 |
| Magazine                     | 2013-1  |    4 |
| Magazine                     | 2013-6  |    5 |
...
| Novel                        | 2011-3  |   13 |
| Novel                        | 2011-4  |   14 |
| Novel                        | 2011-6  |   15 |
| Novel                        | 2011-8  |   17 |
| Novel                        | 2012-1  |   18 |
| Novel                        | 2012-2  |   20 |
...
| Sports                       | 2021-4  |   49 |
| Sports                       | 2021-7  |   50 |
| Sports                       | 2022-4  |   51 |
+------------------------------+---------+------+
1500 ﾚｳﾗ in set (0.79 sec)
```

  2つの実行結果を比較することで、TiFlashを使用するとクエリのスピードが大幅に向上することが分かります（データのボリュームが大きいほど改善がより顕著です）。これは、window関数は通常、いくつかの列のフルテーブルスキャンに依存するためであり、列指向のTiFlashは、行指向のTiKVよりもこのタイプの解析タスクを処理するのに適しているからです。TiKVの場合、プライマリキーまたはインデックスを使用してクエリ対象の行数を減らすと、クエリが高速になり、TiFlashと比較してリソースを消費することも少なくなります。

### クエリエンジンを指定する

TiDBは、コストの見積もりに基づいて自動的にTiFlashレプリカの使用を選択するためにコストベースのオプティマイザ（CBO）を使用します。ただし、クエリがトランザクショナルか解析的かの確実な場合は、[オプティマイザヒント](/optimizer-hints.md)を使用して使用するクエリエンジンを指定できます。

クエリで使用するエンジンを指定するには、次のステートメントのように`/*+ read_from_storage(engine_name[table_name]) */`ヒントを使用できます。

> **ノート:**
>
> - テーブルにエイリアスがある場合は、ヒントでテーブル名の代わりにエイリアスを使用します。そうしないと、ヒントは機能しません。
> - `read_from_storage` ヒントは[共通テーブル式](/develop/dev-guide-use-common-table-expression.md)には機能しません。

```sql
WITH orders_group_by_month AS (
    SELECT
        /*+ read_from_storage(tikv[o]) */
        b.type AS book_type,
        DATE_FORMAT(ordered_at, '%Y-%c') AS month,
        COUNT(*) AS orders
    FROM orders o
    LEFT JOIN books b ON o.book_id = b.id
    WHERE b.type IS NOT NULL
    GROUP BY book_type, month
), acc AS (
    SELECT
        book_type,
        month,
        SUM(orders) OVER(PARTITION BY book_type ORDER BY book_type, month ASC) as acc
    FROM orders_group_by_month mo
    ORDER BY book_type, month ASC
)
SELECT * FROM acc;
```

上記のSQLステートメントの実行計画を確認するために`EXPLAIN`ステートメントを使用できます。もし、タスク列に`cop[tiflash]`や`cop[tikv]`が同時に表示された場合、TiFlashとTiKVが共にこのクエリを完了するためにスケジュールされていることを意味します。なお、TiFlashとTiKVのストレージエンジンは通常異なるTiDBノードを使用するため、これら2つのクエリの種類は互いに影響を受けません。

TiDBがTiFlashを使用するタイミングの詳細については、「TiDBでTiFlashのレプリカを読んでみる」を参照してください。[/tiflash/use-tidb-to-read-tiflash.md](/tiflash/use-tidb-to-read-tiflash.md)

## もっと詳しく

<CustomContent platform="tidb">

- [HTAPのクイックスタート](/quick-start-with-htap.md)
- [HTAPの探索](/explore-htap.md)

</CustomContent>

<CustomContent platform="tidb-cloud">

- [TiDB Cloud HTAPクイックスタート](/tidb-cloud/tidb-cloud-htap-quickstart.md)

</CustomContent>

- [ウィンドウ関数](/functions-and-operators/window-functions.md)
- [TiFlashの使用](/tiflash/tiflash-overview.md#use-tiflash)
```