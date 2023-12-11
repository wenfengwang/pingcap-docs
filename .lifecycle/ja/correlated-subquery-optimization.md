---
title: 相関サブクエリの非相関化
summary: 相関サブクエリの非相関化方法を理解する
---

# 相関サブクエリの非相関化

[サブクエリ関連の最適化](/subquery-optimization.md)では、TiDBが相関する列がない場合にサブクエリをどのように処理するかについて説明しています。相関サブクエリの非相関化は複雑なため、この記事ではいくつかのシンプルなシナリオと最適化ルールが適用される範囲について紹介します。

## はじめに

以下のようなクエリを例に取り上げます: `select * from t1 where t1.a < (select sum(t2.a) from t2 where t2.b = t1.b)`。このサブクエリ `t1.a < (select sum(t2.a) from t2 where t2.b = t1.b)` は、クエリ条件の中で相関列を参照していますが、この条件はたまたま同等の条件です。したがって、このクエリは `select t1.* from t1, (select b, sum(a) sum_a from t2 group by b) t2 where t1.b = t2.b and t1.a < t2.sum_a;` のように書き換えることができます。これにより、相関サブクエリが `JOIN` に書き換えられます。

TiDBがこの書き換えを行う理由は、相関サブクエリが実行されるたびにその外部クエリの結果に束縛されるためです。上記の例では、`t1.a` に1000万の値がある場合、このサブクエリは `t2.b=t1.b` の条件が `t1.a` の値によって異なるため、1000万回繰り返されます。何らかの方法で相関が解除されると、このサブクエリは1回だけ実行されます。

## 制限

この書き換えの欠点は、相関が解除されない場合、オプティマイザが相関する列のインデックスを利用できることです。つまり、このサブクエリは何度も繰り返されるかもしれませんが、インデックスを使用してデータをフィルタリングすることができます。書き換えルールを使用した後は、相関列の位置が通常変更されます。サブクエリはたった1回だけ実行されますが、非相関化しない場合よりも単一の実行時間が長くなることがあります。

したがって、外部値が少ない場合は、非相関化を実行しないでください。これにより実行パフォーマンスが向上することがあります。この場合、[`NO_DECORRELATE`](/optimizer-hints.md#no_decorrelate) オプティマイザヒントを使用するか、[最適化ルールと式のプッシュダウンのブロックリスト](/blocklist-control-plan.md)で "サブクエリの非相関化" 最適化ルールを無効にすることで、この最適化を無効にできます。ほとんどの場合、非相関化を無効にするためにオプティマイザヒントを使用し、[SQLプラン管理](/sql-plan-management.md)と併用することをお勧めします。

## 例

{{< copyable "sql" >}}

```sql
create table t1(a int, b int);
create table t2(a int, b int, index idx(b));
explain select * from t1 where t1.a < (select sum(t2.a) from t2 where t2.b = t1.b);
```

```sql
+----------------------------------+----------+-----------+---------------+-----------------------------------------------------------------------------------------+
| id                               | estRows  | task      | access object | operator info                                                                           |
+----------------------------------+----------+-----------+---------------+-----------------------------------------------------------------------------------------+
| HashJoin_11                      | 9990.00  | root      |               | inner join, equal:[eq(test.t1.b, test.t2.b)], other cond:lt(cast(test.t1.a), Column#7)  |
| ├─HashAgg_23(Build)              | 7992.00  | root      |               | group by:test.t2.b, funcs:sum(Column#8)->Column#7, funcs:firstrow(test.t2.b)->test.t2.b |
| │ └─TableReader_24               | 7992.00  | root      |               | data:HashAgg_16                                                                         |
| │   └─HashAgg_16                 | 7992.00  | cop[tikv] |               | group by:test.t2.b, funcs:sum(test.t2.a)->Column#8                                      |
| │     └─Selection_22             | 9990.00  | cop[tikv] |               | not(isnull(test.t2.b))                                                                  |
| │       └─TableFullScan_21       | 10000.00 | cop[tikv] | table:t2      | keep order:false, stats:pseudo                                                          |
| └─TableReader_15(Probe)          | 9990.00  | root      |               | data:Selection_14                                                                       |
|   └─Selection_14                 | 9990.00  | cop[tikv] |               | not(isnull(test.t1.b))                                                                  |
|     └─TableFullScan_13           | 10000.00 | cop[tikv] | table:t1      | keep order:false, stats:pseudo                                                          |
+----------------------------------+----------+-----------+---------------+-----------------------------------------------------------------------------------------+
```

上記は、最適化が効果を発揮する例です。 `HashJoin_11` は通常の `inner join` です。

次に、`NO_DECORRELATE` のオプティマイザヒントを使用して、オプティマイザに対してサブクエリの非相関化を行わないよう指示できます:

{{< copyable "sql" >}}

```sql
explain select * from t1 where t1.a < (select /*+ NO_DECORRELATE() */ sum(t2.a) from t2 where t2.b = t1.b);
```

```sql
+------------------------------------------+-----------+-----------+------------------------+--------------------------------------------------------------------------------------+
| id                                       | estRows   | task      | access object          | operator info                                                                        |
+------------------------------------------+-----------+-----------|------------------------+--------------------------------------------------------------------------------------+
| Projection_10                            | 10000.00  | root      | test.t1.a, test.t1.b   |
| └─Apply_12                               | 10000.00  | root      |                        |
|   ├─TableReader_14(Build)                | 10000.00  | root      | table:t1               |
|   │ └─TableFullScan_13                   | 10000.00  | cop[tikv] | keep order:false, stats:pseudo                                                   |
|   └─MaxOneRow_15(Probe)                  | 10000.00  | root      |                        |
|     └─StreamAgg_20                       | 10000.00  | root      | funcs:sum(Column#14)->Column#7                                                     |
|       └─Projection_45                    | 100000.00 | root      | cast(test.t2.a, decimal(10,0) BINARY)->Column#14                                   |
|         └─IndexLookUp_44                 | 100000.00 | root      |                        |
|           ├─IndexRangeScan_42(Build)     | 100000.00 | cop[tikv] | table:t2, index:idx(b) | range: decided by [eq(test.t2.b, test.t1.b)], keep order:false, stats:pseudo         |
|           └─TableRowIDScan_43(Probe)     | 100000.00 | cop[tikv] | table:t2               | keep order:false, stats:pseudo                                                      |
+------------------------------------------+-----------+-----------+------------------------+--------------------------------------------------------------------------------------+
```

非相関化ルールを無効にすることでも同じ効果が得られます:

{{< copyable "sql" >}}

```sql
insert into mysql.opt_rule_blacklist values("decorrelate");
admin reload opt_rule_blacklist;
explain select * from t1 where t1.a < (select sum(t2.a) from t2 where t2.b = t1.b);
```

```sql
+------------------------------------------+-----------+-----------+------------------------+--------------------------------------------------------------------------------------+
| id                                       | estRows   | task      | access object          | operator info                                                                        |
+------------------------------------------+-----------+-----------|------------------------+--------------------------------------------------------------------------------------+
| Projection_10                            | 10000.00  | root      | test.t1.a, test.t1.b   |
| └─Apply_12                               | 10000.00  | root      |                        |
|   ├─TableReader_14(Build)                | 10000.00  | root      | table:t1               |
|   │ └─TableFullScan_13                   | 10000.00  | cop[tikv] | keep order:false, stats:pseudo                                                   |
|   └─MaxOneRow_15(Probe)                  | 10000.00  | root      |                        |
|     └─StreamAgg_20                       | 10000.00  | root      | funcs:sum(Column#14)->Column#7                                                     |
|       └─Projection_45                    | 100000.00 | root      | cast(test.t2.a, decimal(10,0) BINARY)->Column#14                                   |
|         └─IndexLookUp_44                 | 100000.00 | root      |                        |
```
|           ├─IndexRangeScan_42(Build)     | 100000.00 | cop[tikv] | table:t2, index:idx(b) | range: decided by [eq(test.t2.b, test.t1.b)], keep order:false, stats:pseudo         |
|           └─TableRowIDScan_43(Probe)     | 100000.00 | cop[tikv] | table:t2               | keep order:false, stats:pseudo                                                       |
+------------------------------------------+-----------+-----------+------------------------+--------------------------------------------------------------------------------------+
```

サブクエリのデコレーションルールを無効にした後、`IndexRangeScan_42(Build)`の`operator info`に`range: decided by [eq(test.t2.b, test.t1.b)]`が表示されます。これは、相関サブクエリのデコレーションが行われず、TiDBがインデックス範囲クエリを使用していることを意味します。