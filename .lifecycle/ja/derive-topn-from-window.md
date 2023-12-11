---
title: ウィンドウ関数からTopNまたはLimitを導出する
summary: ウィンドウ関数からTopNまたはLimitを導出する最適化ルールとそのルールを有効にする方法を紹介します。

# ウィンドウ関数からTopNまたはLimitを導出する

[ウィンドウ関数](/functions-and-operators/window-functions.md) は一般的なSQL関数の一種です。`ROW_NUMBER()`や`RANK()`などのウィンドウ関数を使用して行番号を付与する際に、ウィンドウ関数の結果を評価した後に結果をフィルタリングすることが一般的です。例えば:

```sql
SELECT * FROM (SELECT ROW_NUMBER() OVER (ORDER BY a) AS rownumber FROM t) dt WHERE rownumber <= 3
```

通常のSQL実行プロセスでは、TiDBはまずテーブル`t`のすべてのデータをソートし、次に各行の `ROW_NUMBER()` 結果を計算し、最後に `rownumber <= 3` でフィルタリングします。

v7.0.0以降、TiDBはウィンドウ関数からTopNまたはLimit演算子を導出することをサポートしています。この最適化ルールにより、TiDBは元のSQLを次のような同等の形式に書き換えることができます:

```sql
WITH t_topN AS (SELECT a FROM t1 ORDER BY a LIMIT 3) SELECT * FROM (SELECT ROW_NUMBER() OVER (ORDER BY a) AS rownumber FROM t_topN) dt WHERE rownumber <= 3
```

書き換えた後、TiDBはウィンドウ関数とその後のフィルタ条件からTopN演算子を導出することができます。元のSQLのソート演算子（`ORDER BY`）と比較して、TopN演算子ははるかに高い実行効率を有しています。さらに、TiKVとTiFlashの両方がTopN演算子をプッシュダウンすることをサポートしており、書き換えたSQLのパフォーマンスをさらに向上させることができます。

ウィンドウ関数からTopNまたはLimitを導出する機能はデフォルトで無効になっています。この機能を有効にするには、セッション変数 [tidb_opt_derive_topn](/system-variables.md#tidb_opt_derive_topn-new-in-v700) を `ON` に設定することができます。

この機能を有効にした後は、次のどちらかの操作を行うことで無効にすることができます:

* セッション変数 [tidb_opt_derive_topn](/system-variables.md#tidb_opt_derive_topn-new-in-v700) を `OFF` に設定する。
* [最適化ルールおよび式のプッシュダウンのブロックリスト](/blocklist-control-plan.md) で説明されている手順に従う。

## 制限事項

* `ROW_NUMBER()` ウィンドウ関数のみがSQLの書き換えにサポートされています。
* TiDBは `ROW_NUMBER()` の結果をフィルタリングし、フィルタ条件が `<` または `<=` の場合にのみSQLを書き換えることができます。

## 使用例

以下の例では、最適化ルールの使用方法を実演します。

### PARTITION BYのないウィンドウ関数

#### 例1: ORDER BYのないウィンドウ関数

```sql
CREATE TABLE t(id int, value int);
SET tidb_opt_derive_topn=on;
EXPLAIN SELECT * FROM (SELECT ROW_NUMBER() OVER () AS rownumber FROM t) dt WHERE rownumber <= 3;
```

結果は次の通りです:

```
+----------------------------------+---------+-----------+---------------+-----------------------------------------------------------------------+
| id                               | estRows | task      | access object | operator info                                                         |
+----------------------------------+---------+-----------+---------------+-----------------------------------------------------------------------+
| Projection_9                     | 2.40    | root      |               | Column#5                                                              |
| └─Selection_10                   | 2.40    | root      |               | le(Column#5, 3)                                                       |
|   └─Window_11                    | 3.00    | root      |               | row_number()->Column#5 over(rows between current row and current row) |
|     └─Limit_15                   | 3.00    | root      |               | offset:0, count:3                                                     |
|       └─TableReader_26           | 3.00    | root      |               | data:Limit_25                                                         |
|         └─Limit_25               | 3.00    | cop[tikv] |               | offset:0, count:3                                                     |
|           └─TableFullScan_24     | 3.00    | cop[tikv] | table:t       | keep order:false, stats:pseudo                                        |
+----------------------------------+---------+-----------+---------------+-----------------------------------------------------------------------+
```

このクエリでは、オプティマイザはウィンドウ関数からLimit演算子を導出し、TiKVにプッシュダウンしています。

#### 例2: ORDER BYのあるウィンドウ関数

```sql
CREATE TABLE t(id int, value int);
SET tidb_opt_derive_topn=on;
EXPLAIN SELECT * FROM (SELECT ROW_NUMBER() OVER (ORDER BY value) AS rownumber FROM t) dt WHERE rownumber <= 3;
```

結果は次の通りです:

```
+----------------------------------+----------+-----------+---------------+---------------------------------------------------------------------------------------------+
| id                               | estRows  | task      | access object | operator info                                                                               |
+----------------------------------+----------+-----------+---------------+---------------------------------------------------------------------------------------------+
| Projection_10                    | 2.40     | root      |               | Column#5                                                                                    |
| └─Selection_11                   | 2.40     | root      |               | le(Column#5, 3)                                                                             |
|   └─Window_12                    | 3.00     | root      |               | row_number()->Column#5 over(order by test.t.value rows between current row and current row) |
|     └─TopN_13                    | 3.00     | root      |               | test.t.value, offset:0, count:3                                                             |
|       └─TableReader_25           | 3.00     | root      |               | data:TopN_24                                                                                |
|         └─TopN_24                | 3.00     | cop[tikv] |               | test.t.value, offset:0, count:3                                                             |
|           └─TableFullScan_23     | 10000.00 | cop[tikv] | table:t       | keep order:false, stats:pseudo                                                              |
+----------------------------------+----------+-----------+---------------+---------------------------------------------------------------------------------------------+
```

このクエリでは、オプティマイザはウィンドウ関数からTopN演算子を導出し、TiKVにプッシュダウンしています。

### PARTITION BYのあるウィンドウ関数

> **注記:**
>
> `PARTITION BY` を含むウィンドウ関数の場合、最適化ルールは、パーティション列が主キーの接頭辞であり、かつ主キーがクラスター化インデックスである場合にのみ有効です。

#### 例3: ORDER BYのないウィンドウ関数

```sql
CREATE TABLE t(id1 int, id2 int, value1 int, value2 int, primary key(id1,id2) clustered);
SET tidb_opt_derive_topn=on;
EXPLAIN SELECT * FROM (SELECT ROW_NUMBER() OVER (PARTITION BY id1) AS rownumber FROM t) dt WHERE rownumber <= 3;
```

結果は次の通りです:

```
+------------------------------------+---------+-----------+---------------+-----------------------------------------------------------------------------------------------+
| id                                 | estRows | task      | access object | operator info                                                                                 |
+------------------------------------+---------+-----------+---------------+-----------------------------------------------------------------------------------------------+
| Projection_10                      | 2.40    | root      |               | Column#6                                                                                      |
| └─Selection_11                     | 2.40    | root      |               | le(Column#6, 3)                                                                               |
|   └─Shuffle_26                     | 3.00    | root      |               | execution info: concurrency:2, data sources:[TableReader_24]                                  |
|     └─Window_12                    | 3.00    | root      |               | row_number()->Column#6 over(partition by test.t.id1 rows between current row and current row) |
|       └─Sort_25                    | 3.00    | root      |               | test.t.id1                                                                                    |
|         └─TableReader_24           | 3.00    | root      |               | data:Limit_23                                                                                 |
|           └─Limit_23               | 3.00    | cop[tikv] |               | partition by test.t.id1, offset:0, count:3                                                    |
|             └─TableFullScan_22     | 3.00    | cop[tikv] | table:t       | keep order:false, stats:pseudo                                                                |
+------------------------------------+---------+-----------+---------------+-----------------------------------------------------------------------------------------------+
```

このクエリでは、オプティマイザはウィンドウ関数からLimit演算子を導出し、TiKVにプッシュダウンしています。ここで言うLimitは実際にはパーティションLimitであり、つまり、同じ `id1` の値を持つデータそれぞれにLimitが適用されます。

#### 例4: ORDER BYのあるウィンドウ関数

```sql
CREATE TABLE t(id1 int, id2 int, value1 int, value2 int, primary key(id1,id2) clustered);
SET tidb_opt_derive_topn=on;
EXPLAIN SELECT * FROM (SELECT ROW_NUMBER() OVER (PARTITION BY id1 ORDER BY value1) AS rownumber FROM t) dt WHERE rownumber <= 3;
```

結果は次の通りです:

```
+------------------------------------+----------+-----------+---------------+----------------------------------------------------------------------------------------------------------------------+
| id                                 | estRows  | task      | access object | operator info                                                                                                        |
+------------------------------------+----------+-----------+---------------+----------------------------------------------------------------------------------------------------------------------+
| Projection_10                      | 2.40     | root      |               | Column#6                                                                                                             |
```
| └─Selection_11                     | 2.40     | root      |               | le(Column#6, 3)                                                                                                      |
|   └─Shuffle_23                     | 3.00     | root      |               | execution info: concurrency:3, data sources:[TableReader_21]                                                         |
|     └─Window_12                    | 3.00     | root      |               | row_number()->Column#6 over(partition by test.t.id1 order by test.t.value1 rows between current row and current row) |
|       └─Sort_22                    | 3.00     | root      |               | test.t.id1, test.t.value1                                                                                            |
|         └─TableReader_21           | 3.00     | root      |               | data:TopN_19                                                                                                         |
|           └─TopN_19                | 3.00     | cop[tikv] |               | partition by test.t.id1 order by test.t.value1, offset:0, count:3                                                    |
|             └─TableFullScan_18     | 10000.00 | cop[tikv] | table:t       | keep order:false, stats:pseudo                                                                                       |
+------------------------------------+----------+-----------+---------------+----------------------------------------------------------------------------------------------------------------------+

このクエリでは、最適化プログラムがWindow関数からTopN演算子を導出し、TiKVにプッシュダウンします。このTopNは実際はパーティションTopNであり、つまりTopNは`id1`値が同じデータの各グループに適用されることを意味します。

#### 例5: PARTITION BY列が主キーの接頭辞ではない

```sql
CREATE TABLE t(id1 int, id2 int, value1 int, value2 int, primary key(id1,id2) clustered);
SET tidb_opt_derive_topn=on;
EXPLAIN SELECT * FROM (SELECT ROW_NUMBER() OVER (PARTITION BY value1) AS rownumber FROM t) dt WHERE rownumber <= 3;
```

結果は以下のようになります:

```
+----------------------------------+----------+-----------+---------------+--------------------------------------------------------------------------------------------------+
| id                               | estRows  | task      | access object | operator info                                                                                    |
+----------------------------------+----------+-----------+---------------+--------------------------------------------------------------------------------------------------+
| Projection_9                     | 8000.00  | root      |               | Column#6                                                                                         |
| └─Selection_10                   | 8000.00  | root      |               | le(Column#6, 3)                                                                                  |
|   └─Shuffle_15                   | 10000.00 | root      |               | execution info: concurrency:5, data sources:[TableReader_13]                                     |
|     └─Window_11                  | 10000.00 | root      |               | row_number()->Column#6 over(partition by test.t.value1 rows between current row and current row) |
|       └─Sort_14                  | 10000.00 | root      |               | test.t.value1                                                                                    |
|         └─TableReader_13         | 10000.00 | root      |               | data:TableFullScan_12                                                                            |
|           └─TableFullScan_12     | 10000.00 | cop[tikv] | table:t       | keep order:false, stats:pseudo                                                                   |
+----------------------------------+----------+-----------+---------------+--------------------------------------------------------------------------------------------------+

このクエリでは、`PARTITION BY`列が主キーの接頭辞ではないため、SQLが書き換えられていません。

#### 例6: PARTITION BY列が主キーの接頭辞であるが、クラスター化インデックスではない

```sql
CREATE TABLE t(id1 int, id2 int, value1 int, value2 int, primary key(id1,id2) nonclustered);
SET tidb_opt_derive_topn=on;
EXPLAIN SELECT * FROM (SELECT ROW_NUMBER() OVER (PARTITION BY id1) AS rownumber FROM t use index()) dt WHERE rownumber <= 3;
```

結果は以下のようになります:

```
+----------------------------------+----------+-----------+---------------+-----------------------------------------------------------------------------------------------+
| id                               | estRows  | task      | access object | operator info                                                                                 |
+----------------------------------+----------+-----------+---------------+-----------------------------------------------------------------------------------------------+
| Projection_9                     | 8000.00  | root      |               | Column#7                                                                                      |
| └─Selection_10                   | 8000.00  | root      |               | le(Column#7, 3)                                                                               |
|   └─Shuffle_15                   | 10000.00 | root      |               | execution info: concurrency:5, data sources:[TableReader_13]                                  |
|     └─Window_11                  | 10000.00 | root      |               | row_number()->Column#7 over(partition by test.t.id1 rows between current row and current row) |
|       └─Sort_14                  | 10000.00 | root      |               | test.t.id1                                                                                    |
|         └─TableReader_13         | 10000.00 | root      |               | data:TableFullScan_12                                                                         |
|           └─TableFullScan_12     | 10000.00 | cop[tikv] | table:t       | keep order:false, stats:pseudo                                                                |
+----------------------------------+----------+-----------+---------------+-----------------------------------------------------------------------------------------------+

このクエリでは、`PARTITION BY`列が主キーの接頭辞であるが、SQLが書き換えられていません。これは主キーがクラスター化インデックスでないためです。