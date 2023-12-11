---
title: サブクエリを使用したステートメントの説明
summary: TiDBのEXPLAINステートメントによって返される実行計画情報について学びます。

# サブクエリを使用したステートメントの説明

TiDBはサブクエリのパフォーマンスを向上させるために[いくつかの最適化](/subquery-optimization.md)を行います。このドキュメントでは、一般的なサブクエリに対するこれらの最適化のいくつかを説明し、`EXPLAIN`の出力の解釈方法について説明します。

このドキュメントの例は、次のサンプルデータに基づいています：

```sql
CREATE TABLE t1 (id BIGINT NOT NULL PRIMARY KEY auto_increment, pad1 BLOB, pad2 BLOB, pad3 BLOB, int_col INT NOT NULL DEFAULT 0);
CREATE TABLE t2 (id BIGINT NOT NULL PRIMARY KEY auto_increment, t1_id BIGINT NOT NULL, pad1 BLOB, pad2 BLOB, pad3 BLOB, INDEX(t1_id));
CREATE TABLE t3 (
 id INT NOT NULL PRIMARY KEY auto_increment,
 t1_id INT NOT NULL,
 UNIQUE (t1_id)
);

INSERT INTO t1 SELECT NULL, RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024), 0 FROM dual;
INSERT INTO t1 SELECT NULL, RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024), 0 FROM t1 a JOIN t1 b JOIN t1 c LIMIT 10000;
INSERT INTO t2 SELECT NULL, a.id, RANDOM_BYTES(1024), RANDOM_BYTES(1024), RANDOM_BYTES(1024) FROM t1 a JOIN t1 b JOIN t1 c LIMIT 10000;
INSERT INTO t3 SELECT NULL, id FROM t1 WHERE id < 1000;

SELECT SLEEP(1);
ANALYZE TABLE t1, t2, t3;
```

## 内部結合（一意でないサブクエリ）

次の例では、`IN`サブクエリはテーブル`t2`からIDのリストを検索します。意味的に正しいため、TiDBは`EXPLAIN`を使用して重複を削除し、`INNER JOIN`操作を実行する実行計画を見ることができます：

```sql
EXPLAIN SELECT * FROM t1 WHERE id IN (SELECT t1_id FROM t2);
```

```sql
+--------------------------------+----------+-----------+------------------------------+---------------------------------------------------------------------------------------------------------------------------+
| id                             | estRows  | task      | access object                | operator info                                                                                                             |
+--------------------------------+----------+-----------+------------------------------+---------------------------------------------------------------------------------------------------------------------------+
| IndexJoin_15                   | 21.11    | root      |                              | inner join, inner:TableReader_12, outer key:test.t2.t1_id, inner key:test.t1.id, equal cond:eq(test.t2.t1_id, test.t1.id) |
| ├─StreamAgg_44(Build)          | 21.11    | root      |                              | group by:test.t2.t1_id, funcs:firstrow(test.t2.t1_id)->test.t2.t1_id                                                      |
| │ └─IndexReader_45             | 21.11    | root      |                              | index:StreamAgg_34                                                                                                        |
| │   └─StreamAgg_34             | 21.11    | cop[tikv] |                              | group by:test.t2.t1_id,                                                                                                   |
| │     └─IndexFullScan_26       | 90000.00 | cop[tikv] | table:t2, index:t1_id(t1_id) | keep order:true                                                                                                           |
| └─TableReader_12(Probe)        | 21.11    | root      |                              | data:TableRangeScan_11                                                                                                    |
|   └─TableRangeScan_11          | 21.11    | cop[tikv] | table:t1                     | range: decided by [test.t2.t1_id], keep order:false                                                                       |
+--------------------------------+----------+-----------+------------------------------+---------------------------------------------------------------------------------------------------------------------------+
```

上記のクエリ結果から、TiDBがインデックス結合操作`IndexJoin_15`を使用してサブクエリを結合および変換することが分かります。実行計画では、実行プロセスは次のようになります：

1. TiKV側のインデックススキャンオペレーター`└─IndexFullScan_26`は、`t2.t1_id`列の値を読み取ります。
2. 一部の`└─StreamAgg_34`オペレーターのタスクは、TiKVの`t1_id`の値の重複を除去します。
3. 一部の`├─StreamAgg_44(Build)`オペレーターのタスクは、TiDBの`t1_id`の値の重複を除去します。重複除去は、集約関数`firstrow(test.t2.t1_id)`によって行われます。
4. 操作結果は、`t1`テーブルの主キーと結合されます。結合条件は`eq(test.t1.id, test.t2.t1_id)`です。

## 内部結合（一意のサブクエリ）

前の例では、テーブル`t1`に対して結合する前に、`t1_id`の値が一意であることを保証するために集計が必要でした。しかし、次の例では、`t3.t1_id`がすでに`UNIQUE`制約によって一意であるため：

```sql
EXPLAIN SELECT * FROM t1 WHERE id IN (SELECT t1_id FROM t3);
```

```sql
+-----------------------------+---------+-----------+------------------------------+---------------------------------------------------------------------------------------------------------------------------+
| id                          | estRows | task      | access object                | operator info                                                                                                             |
+-----------------------------+---------+-----------+------------------------------+---------------------------------------------------------------------------------------------------------------------------+
| IndexJoin_18                | 999.00  | root      |                              | inner join, inner:TableReader_15, outer key:test.t3.t1_id, inner key:test.t1.id, equal cond:eq(test.t3.t1_id, test.t1.id) |
+-----------------------------+---------+-----------+------------------------------+---------------------------------------------------------------------------------------------------------------------------+
```
| ├─IndexReader_41(Build)     | 999.00  | root      |                              | index:IndexFullScan_40                                                                                                    |
| │ └─IndexFullScan_40        | 999.00  | cop[tikv] | table:t3, index:t1_id(t1_id) | keep order:false                                                                                                          |
| └─TableReader_15(Probe)     | 999.00  | root      |                              | data:TableRangeScan_14                                                                                                    |
|   └─TableRangeScan_14       | 999.00  | cop[tikv] | table:t1                     | range: decided by [test.t3.t1_id], keep order:false                                                                       |
+-----------------------------+---------+-----------+------------------------------+---------------------------------------------------------------------------------------------------------------------------+

意味的には、`t3.t1_id` は一意であるため、`INNER JOIN` として直接実行できます。

## セミ結合 (相関副問い合わせ)

前の 2 つの例では、TiDB はサブクエリ内のデータを一意にして (`StreamAgg` を経由した) か、一意であると保証した後に (`Index Join` を使用して) `INNER JOIN` 操作を実行できます。

この例では、TiDB は異なる実行プランを選択します:

```sql
EXPLAIN SELECT * FROM t1 WHERE id IN (SELECT t1_id FROM t2 WHERE t1_id != t1.int_col);
```

```sql
+-----------------------------+----------+-----------+------------------------------+--------------------------------------------------------------------------------------------------------+
| id                          | estRows  | task      | access object                | operator info                                                                                          |
+-----------------------------+----------+-----------+------------------------------+--------------------------------------------------------------------------------------------------------+
| MergeJoin_9                 | 45446.40 | root      |                              | semi join, left key:test.t1.id, right key:test.t2.t1_id, other cond:ne(test.t2.t1_id, test.t1.int_col) |
| ├─IndexReader_24(Build)     | 90000.00 | root      |                              | index:IndexFullScan_23                                                                                 |
| │ └─IndexFullScan_23        | 90000.00 | cop[tikv] | table:t2, index:t1_id(t1_id) | keep order:true                                                                                        |
| └─TableReader_22(Probe)     | 56808.00 | root      |                              | data:Selection_21                                                                                      |
|   └─Selection_21            | 56808.00 | cop[tikv] |                              | ne(test.t1.id, test.t1.int_col)                                                                        |
|     └─TableFullScan_20      | 71010.00 | cop[tikv] | table:t1                     | keep order:true                                                                                        |
+-----------------------------+----------+-----------+------------------------------+--------------------------------------------------------------------------------------------------------+
```

上記の結果から、TiDB が `Semi Join` アルゴリズムを使用していることがわかります。セミ結合は内部結合と異なります: セミ結合では右側のキー (`t2.t1_id`) の最初の値のみが許可されるため、重複が結合演算の一部として排除されます。また、結合アルゴリズムはマージ結合であり、オペレータは両側のデータをソート順に読み取る効率的なジッパー結合のようなものです。

元の文は _相関副問い合わせ_ と見なされます。サブクエリがサブクエリ外に存在する列 (`t1.int_col`) を参照するためです。ただし、`EXPLAIN` の出力は [サブクエリの非相関化最適化](/correlated-subquery-optimization.md) を適用した後の実行プランを示しています。`t1_id != t1.int_col` の条件は `t1.id != t1.int_col` に書き換えられます。テーブル `t1` からデータを読み取っているため、TiDB は `└─Selection_21` でこの非相関化と書き換えを実行できます。そのため、この非相関化と書き換えにより、実行がはるかに効率的になります。

## アンチセミ結合 (`NOT IN` サブクエリ)

次の例では、クエリはセマンティック的に、サブクエリ内の `id` が存在しない場合に限り、テーブル `t3` からすべての行を返します:

```sql
EXPLAIN SELECT * FROM t3 WHERE t1_id NOT IN (SELECT id FROM t1 WHERE int_col < 100);
```

```sql
+-----------------------------+---------+-----------+---------------+-------------------------------------------------------------------------------------------------------------------------------+
| id                          | estRows | task      | access object | operator info                                                                                                                 |
+-----------------------------+---------+-----------+---------------+-------------------------------------------------------------------------------------------------------------------------------+
| IndexJoin_16                | 799.20  | root      |               | anti semi join, inner:TableReader_12, outer key:test.t3.t1_id, inner key:test.t1.id, equal cond:eq(test.t3.t1_id, test.t1.id) |
| ├─TableReader_28(Build)     | 999.00  | root      |               | data:TableFullScan_27                                                                                                         |
| │ └─TableFullScan_27        | 999.00  | cop[tikv] | table:t3      | keep order:false                                                                                                              |
| └─TableReader_12(Probe)     | 999.00  | root      |               | data:Selection_11                                                                                                             |
|   └─Selection_11            | 999.00  | cop[tikv] |               | lt(test.t1.int_col, 100)                                                                                                      |
|     └─TableRangeScan_10     | 999.00  | cop[tikv] | table:t1      | range: decided by [test.t3.t1_id], keep order:false                                                                           |
+-----------------------------+---------+-----------+---------------+-------------------------------------------------------------------------------------------------------------------------------+
```

このクエリはまずテーブル `t3` を読み込み、その後 `PRIMARY KEY` を基にテーブル `t1` にプローブします。結合タイプは _アンチセミ結合_ です。これは (`NOT IN` のため) 値の非存在に対して行われるため _アンチ_ であり、最初の行のみ一致すれば結合が拒否されるため _セミ結合_ です。
| └─HashAgg_17(Probe)          | 1.00    | root      |               | group by:test.s.a, test.s.b, funcs:firstrow(test.s.a)->test.s.a, funcs:firstrow(test.s.b)->test.s.b |
|   └─TableReader_24           | 1.00    | root      |               | data:Selection_23                                                                                   |
|     └─Selection_23           | 1.00    | cop[tikv] |               | not(isnull(test.s.a)), not(isnull(test.s.b))                                                        |
|       └─TableFullScan_22     | 1.00    | cop[tikv] | table:s       | keep order:false, stats:pseudo                                                                      |
+------------------------------+---------+-----------+---------------+-----------------------------------------------------------------------------------------------------+
8 rows in set (0.01 sec)
```

最初のクエリステートメント「IN」のサブクエリによって変換された左外部セミジョインは、`a`と`b`のカラムが`NULLABLE`であるため、nullを意識したものです。具体的には、まずデカルト積が計算され、その後に`IN`または`= ANY`で接続されたカラムが他の条件に通常の等価クエリとしてフィルタリングされます。

2番目のクエリステートメント「EXPLAIN SELECT * FROM t WHERE (a,b) IN (SELECT * FROM s);」では、`t`と`s`のテーブルの`a`と`b`の列が`NULLABLE`であるため、`IN`サブクエリはnullを意識したセミジョインに変換されるべきでした。しかし、TiDBはセミジョインをインナージョインと集約に変換して最適化しています。これは、`NULL`と`false`が非スカラー出力のため、`IN`サブクエリの中で等価です。プッシュダウンフィルタの`NULL`行は、`WHERE`句の否定的な意味をもたらします。そのため、これらの行は事前に無視できます。

> **注意:**
>
> `Exists`演算子もセミジョインに変換されますが、nullを意識しません。

## Null-aware anti semi join (`NOT IN`および`!= ALL`サブクエリ)

`NOT IN`または`!= ALL`集合演算子の値は3つの値（ `true`、 `false`、および`NULL`）です。これらの2つの演算子から変換されたジョインタイプに対して、TiDBはジョインキーの両側の`NULL`を認識し、特別な方法で処理する必要があります。

`NOT IN`および`!= ALL`演算子を含むサブクエリは、それぞれアンチセミジョインとアンチ左外部セミジョインに変換されます。[アンチセミジョイン](#アンチセミジョイン-not-inサブクエリ)の前述の例では、`test.t3.t1_id`と`test.t1.id`という両方のジョインキーの列は`NOT NULL`であるため、アンチセミジョインにはnullを意識した特別な処理は必要ありません（`NULL`は特別に処理されません）。

TiDB v6.3.0はnullを意識したアンチジョイン（NAAJ）を次のように最適化しています。

- Null意識的な等価条件（NA-EQ）を使用してハッシュジョインを構築

    セット演算子は等価条件を導入し、条件の両側のオペランドの`NULL`値に対して特別な処理が必要です。Nullを意識する等価条件はNA-EQと呼ばれます。以前のバージョンとは異なり、TiDB v6.3.0はNA-EQを以前のように処理せず、ジョイン後に他の条件に置き、その後でカルテシアン積をマッチングし、結果セットの正当性を判定します。

    TiDB v6.3.0以降、弱体化された等価条件であるNA-EQは、依然としてハッシュジョインを構築するために使用されます。これにより、トラバースする必要があるデータのマッチング量が減少し、マッチングプロセスが高速化されます。ビルドテーブルの`DISTINCT（）`値の合計パーセンテージがほぼ100％である場合、加速はより顕著です。

- `NULL`の特別なプロパティを使用してマッチング結果を高速化

    アンチセミジョインは、連言標準形（CNF）です。ジョインのどちらかの側の`NULL`は、決定的な結果につながります。この特性は、マッチングプロセス全体の返却を高速化するために使用できます。

以下は例です。

```sql
CREATE TABLE t(a INT, b INT);
CREATE TABLE s(a INT, b INT);
EXPLAIN SELECT (a, b) NOT IN (SELECT * FROM s) FROM t;
EXPLAIN SELECT * FROM t WHERE (a, b) NOT IN (SELECT * FROM s);
```

```sql
tidb> EXPLAIN SELECT (a, b) NOT IN (SELECT * FROM s) FROM t;
+-----------------------------+----------+-----------+---------------+---------------------------------------------------------------------------------------------+
| id                          | estRows  | task      | access object | operator info                                                                               |
+-----------------------------+----------+-----------+---------------+---------------------------------------------------------------------------------------------+
| HashJoin_8                  | 10000.00 | root      |               | Null-aware anti left outer semi join, equal:[eq(test.t.b, test.s.b) eq(test.t.a, test.s.a)] |
| ├─TableReader_12(Build)     | 10000.00 | root      |               | data:TableFullScan_11                                                                       |
| │ └─TableFullScan_11        | 10000.00 | cop[tikv] | table:s       | keep order:false, stats:pseudo                                                              |
| └─TableReader_10(Probe)     | 10000.00 | root      |               | data:TableFullScan_9                                                                        |
|   └─TableFullScan_9         | 10000.00 | cop[tikv] | table:t       | keep order:false, stats:pseudo                                                              |
+-----------------------------+----------+-----------+---------------+---------------------------------------------------------------------------------------------+
5 rows in set (0.00 sec)

tidb> EXPLAIN SELECT * FROM t WHERE (a, b) NOT IN (SELECT * FROM s);
+-----------------------------+----------+-----------+---------------+----------------------------------------------------------------------------------+
| id                          | estRows  | task      | access object | operator info                                                                    |
+-----------------------------+----------+-----------+---------------+----------------------------------------------------------------------------------+
| HashJoin_8                  | 8000.00  | root      |               | Null-aware anti semi join, equal:[eq(test.t.b, test.s.b) eq(test.t.a, test.s.a)] |
| ├─TableReader_12(Build)     | 10000.00 | root      |               | data:TableFullScan_11                                                            |
| │ └─TableFullScan_11        | 10000.00 | cop[tikv] | table:s       | keep order:false, stats:pseudo                                                   |
| └─TableReader_10(Probe)     | 10000.00 | root      |               | data:TableFullScan_9                                                             |
|   └─TableFullScan_9         | 10000.00 | cop[tikv] | table:t       | keep order:false, stats:pseudo                                                   |
+-----------------------------+----------+-----------+---------------+----------------------------------------------------------------------------------+
5 rows in set (0.00 sec)
```

初めのクエリステートメント `EXPLAIN SELECT (a, b) NOT IN (SELECT * FROM s) FROM t;` では、`t`と`s`のテーブルの`a`と`b`の列が`NULLABLE`であるため、`NOT IN`サブクエリによって変換された左外部セミジョインはヌルを意識しています。 ナールザーデフェレンスの違いと、NA-EQをハッシュジョイン条件として使用している点にあります。これにより、ジョインの計算が大幅にスピードアップします。

2番目のクエリステートメント `EXPLAIN SELECT * FROM t WHERE (a, b) NOT IN (SELECT * FROM s);` では、`t`と`s`のテーブルの`a`と`b`の列が`NULLABLE`であるため、`NOT IN`サブクエリによって変換されたアンチセミジョインはヌルを意識しています。 ナールザーデフェレンスの違いと、NA-EQをハッシュジョイン条件として使用している点にあります。これにより、ジョインの計算が大幅にスピードアップします。

現在、TiDBはアンチセミジョインとアンチ左外部セミジョインをヌルを意識しています。 ハッシュジョインタイプのみがサポートされ、そのビルドテーブルは右側のテーブルに固定されている必要があります。

> **注意:**
>
> `Not Exists`演算子もアンチセミジョインに変換されますが、nullを意識しません。

## 他のタイプのサブクエリを使用した説明ステートメント

+ [MPPモードでの説明ステートメント](/explain-mpp.md)
+ [インデックスを使用した説明ステートメント](/explain-indexes.md)
+ [結合を使用した説明ステートメント](/explain-joins.md)
+ [集約を使用した説明ステートメント](/explain-aggregation.md)
+ [ビューを使用した説明ステートメント](/explain-views.md)
+ [パーティションを使用した説明ステートメント](/explain-partitions.md)
+ [インデックスマージを使用した説明ステートメント](/explain-index-merge.md)