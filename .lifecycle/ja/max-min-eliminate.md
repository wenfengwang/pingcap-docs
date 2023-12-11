---
title: 最大値/最小値の除去
summary: 最大値/最小値関数の除去ルールを紹介します。

# 最大値/最小値の除去

SQLステートメントに`max`/`min`関数が含まれている場合、クエリオプティマイザは`max`/`min`集約関数を`max`/`min`最適化ルールを適用し、`max`/`min`集約関数をTopN演算子に変換しようとします。これにより、TiDBはインデックスを介してクエリを効率的に実行できます。

この最適化ルールは、`select`ステートメント内の`max`/`min`関数の数に応じて次の2つのタイプに分かれます。

- [1つの`max`/`min`関数を持つステートメント](#one-maxmin-function)
- [複数の`max`/`min`関数を持つステートメント](#multiple-maxmin-functions)

## 1つの`max`/`min`関数

SQLステートメントが次の条件を満たす場合、このルールが適用されます。

- ステートメントには`max`または`min`の集約関数が1つだけ含まれています。
- 集約関数には関連する`group by`句がありません。

例：

{{< copyable "sql" >}}

```sql
select max(a) from t
```

最適化ルールにより、ステートメントは次のように書き換えられます：

{{< copyable "sql" >}}

```sql
select max(a) from (select a from t where a is not null order by a desc limit 1) t
```

列`a`にインデックスがある場合、または列`a`がいくつかの複合インデックスのプレフィックスである場合、新しいSQLステートメントはデータの1行しかスキャンすることで最大値または最小値を見つけることができます。この最適化により、フルテーブルスキャンを回避できます。

例のステートメントの実行計画は次のとおりです：

```sql
mysql> explain select max(a) from t;
+------------------------------+---------+-----------+-------------------------+-------------------------------------+
| id                           | estRows | task      | access object           | operator info                       |
+------------------------------+---------+-----------+-------------------------+-------------------------------------+
| StreamAgg_13                 | 1.00    | root      |                         | funcs:max(test.t.a)->Column#4       |
| └─Limit_17                   | 1.00    | root      |                         | offset:0, count:1                   |
|   └─IndexReader_27           | 1.00    | root      |                         | index:Limit_26                      |
|     └─Limit_26               | 1.00    | cop[tikv] |                         | offset:0, count:1                   |
|       └─IndexFullScan_25     | 1.00    | cop[tikv] | table:t, index:idx_a(a) | keep order:true, desc, stats:pseudo |
+------------------------------+---------+-----------+-------------------------+-------------------------------------+
5 rows in set (0.00 sec)
```

## 複数の`max`/`min`関数

SQLステートメントが次の条件を満たす場合、このルールが適用されます。

- ステートメントに複数の`max`または`min`関数が含まれています。
- 集約関数に関連する`group by`句がありません。
- 各`max`/`min`関数の列には順序を保持するインデックスがあります。

例：

{{< copyable "sql" >}}

```sql
select max(a) - min(a) from t
```

最適化ルールはまず、列`a`に順序を保持するインデックスがあるかどうかをチェックします。もしインデックスがあれば、SQLステートメントは2つのサブクエリの直積として書き換えられます：

{{< copyable "sql" >}}

```sql
select max_a - min_a
from
    (select max(a) as max_a from t) t1,
    (select min(a) as min_a from t) t2
```

書き換えにより、最適化プログラムはそれぞれのサブクエリに対して1つの`max`/`min`関数を持つステートメントのルールを適用することができます。ステートメントは次のように書き換えられます：

{{< copyable "sql" >}}

```sql
select max_a - min_a
from
    (select max(a) as max_a from (select a from t where a is not null order by a desc limit 1) t) t1,
    (select min(a) as min_a from (select a from t where a is not null order by a asc limit 1) t) t2
```

同様に、列`a`に順序を保持するインデックスがある場合、最適化された実行はテーブル全体をスキャンする代わりにデータの2行だけをスキャンします。しかし、列`a`に順序を保持するインデックスがない場合は、このルールにより2つのテーブル全体のスキャンが発生しますが、そのステートメントを書き換えなければ1回のテーブル全体のスキャンだけが必要です。したがって、このような場合にはこのルールは適用されません。

最終的な実行計画は次のようになります：

```sql
mysql> explain select max(a)-min(a) from t;
+------------------------------------+---------+-----------+-------------------------+-------------------------------------+
| id                                 | estRows | task      | access object           | operator info                       |
+------------------------------------+---------+-----------+-------------------------+-------------------------------------+
| Projection_17                      | 1.00    | root      |                         | minus(Column#4, Column#5)->Column#6 |
| └─HashJoin_18                      | 1.00    | root      |                         | CARTESIAN inner join                |
|   ├─StreamAgg_45(Build)            | 1.00    | root      |                         | funcs:min(test.t.a)->Column#5       |
|   │ └─Limit_49                     | 1.00    | root      |                         | offset:0, count:1                   |
|   │   └─IndexReader_59             | 1.00    | root      |                         | index:Limit_58                      |
|   │     └─Limit_58                 | 1.00    | cop[tikv] |                         | offset:0, count:1                   |
|   │       └─IndexFullScan_57       | 1.00    | cop[tikv] | table:t, index:idx_a(a) | keep order:true, stats:pseudo       |
|   └─StreamAgg_24(Probe)            | 1.00    | root      |                         | funcs:max(test.t.a)->Column#4       |
|     └─Limit_28                     | 1.00    | root      |                         | offset:0, count:1                   |
|       └─IndexReader_38             | 1.00    | root      |                         | index:Limit_37                      |
|         └─Limit_37                 | 1.00    | cop[tikv] |                         | offset:0, count:1                   |
|           └─IndexFullScan_36       | 1.00    | cop[tikv] | table:t, index:idx_a(a) | keep order:true, desc, stats:pseudo |
+------------------------------------+---------+-----------+-------------------------+-------------------------------------+
12 rows in set (0.01 sec)
```