---
title: サブクエリ関連の最適化
summary: サブクエリに関連する最適化について理解する。
---

# サブクエリ関連の最適化

この記事では、主にサブクエリに関連する最適化について紹介します。

通常、サブクエリは以下の状況で現れます：

- `NOT IN (SELECT ... FROM ...)`
- `NOT EXISTS (SELECT ... FROM ...)`
- `IN (SELECT ... FROM ..)`
- `EXISTS (SELECT ... FROM ...)`
- `... >/>=/</<=/=/!= (SELECT ... FROM ...)`

サブクエリには、`SELECT * FROM t2 WHERE t.b=t2.b` のように、サブクエリ内でサブクエリ以外の列が含まれることがあります。このようなサブクエリは通常、「相関サブクエリ」と呼ばれ、外部から導入された列は「相関列」と呼ばれます。相関サブクエリに関する最適化については、「相関サブクエリの除去」を参照してください。この記事では、相関列を含まないサブクエリに焦点を当てます。

デフォルトでは、TiDB では、サブクエリに対して [TiDB 実行計画の理解](/jp/explain-overview.md) で言及されているように `semi join` を実行方法として使用します。特定の特殊なサブクエリに対しては、より優れたパフォーマンスを得るために、いくつかの論理的な書き換えを行います。

## `... < ALL (SELECT ... FROM ...)` または `... > ANY (SELECT ... FROM ...)`

この場合、`ALL` と `ANY` は `MAX` と `MIN` に置き換えることができます。テーブルが空の場合、`MAX(EXPR)` と `MIN(EXPR)` の結果は NULL になります。また、`EXPR` の結果に `NULL` が含まれている場合も同様です。`EXPR` の結果に `NULL` が含まれているかどうかは、式の最終結果に影響を与える可能性があります。そのため、次の形式で完全な書き換えが行われます：

- `t.id < all (select s.id from s)` は、`t.id < min(s.id) and if(sum(s.id is null) != 0, null, true)` に書き換えられます
- `t.id < any (select s.id from s)` は、`t.id < max(s.id) or if(sum(s.id is null) != 0, null, false)` に書き換えられます

## `... != ANY (SELECT ... FROM ...)`

この場合、サブクエリからのすべての値が一意であれば、クエリとそれらを比較するだけで十分です。サブクエリ内の異なる値の数が 1 より多い場合、不等号が存在する必要があります。そのため、このようなサブクエリは次のように書き換えることができます：

- `select * from t where t.id != any (select s.id from s)` は、`(select t.* from t, (select s.id, count(distinct s.id) as cnt_distinct from s) where (t.id != s.id or cnt_distinct > 1)` に書き換えられます

## `... = ALL (SELECT ... FROM ...)`

この場合、サブクエリ内の異なる値の数が 1 より多い場合、この式の結果は必ず false になります。そのため、このようなサブクエリは TiDB では、次の形式に書き換えられます：

- `select * from t where t.id = all (select s.id from s)` は、`(select t.* from t, (select s.id, count(distinct s.id) as cnt_distinct from s ) where (t.id = s.id and cnt_distinct <= 1)` に書き換えられます

## `... IN (SELECT ... FROM ...)`

この場合、`IN` のサブクエリは `SELECT ... FROM ... GROUP ...` に書き換えられ、その後通常の `JOIN` の形式に書き換えられます。

例えば、`select * from t1 where t1.a in (select t2.a from t2)` は、`select t1.* from t1, (select distinct(a) a from t2) t2 where t1.a = t2` の形に書き換えられます。ここでの `DISTINCT` 属性は、`t2.a` が `UNIQUE` 属性を持っている場合は自動的に取り除かれることができます。

{{< copyable "sql" >}}

```sql
explain select * from t1 where t1.a in (select t2.a from t2);
```

```sql
+------------------------------+---------+-----------+------------------------+----------------------------------------------------------------------------+
| id                           | estRows | task      | access object          | operator info                                                              |
+------------------------------+---------+-----------+------------------------+----------------------------------------------------------------------------+
| IndexJoin_12                 | 9990.00 | root      |                        | inner join, inner:TableReader_11, outer key:test.t2.a, inner key:test.t1.a |
| ├─HashAgg_21(Build)          | 7992.00 | root      |                        | group by:test.t2.a, funcs:firstrow(test.t2.a)->test.t2.a                   |
| │ └─IndexReader_28           | 9990.00 | root      |                        | index:IndexFullScan_27                                                     |
| │   └─IndexFullScan_27       | 9990.00 | cop[tikv] | table:t2, index:idx(a) | keep order:false, stats:pseudo                                             |
| └─TableReader_11(Probe)      | 7992.00 | root      |                        | data:TableRangeScan_10                                                     |
|   └─TableRangeScan_10        | 7992.00 | cop[tikv] | table:t1               | range: decided by [test.t2.a], keep order:false, stats:pseudo              |
+------------------------------+---------+-----------+------------------------+----------------------------------------------------------------------------+
```

この書き換えは、`IN` サブクエリが相対的に小さく、外部クエリが相対的に大きい場合に性能が向上します。なぜなら、書き換えなしで、t2 をドライビングテーブルとして使用する `index join` の使用が不可能だからです。ただし、欠点としては、書き換え中に集約を自動的に除去できない場合や、`t2` テーブルが比較的大きい場合、この書き換えがクエリのパフォーマンスに影響を与えます。現在、変数 [tidb_opt_insubq_to_join_and_agg](/jp/system-variables.md#tidb_opt_insubq_to_join_and_agg) を使用して、この最適化を制御します。この最適化が適切ではない場合、手動で無効にすることができます。

## `EXISTS` サブクエリと `... >/>=/</<=/=/!= (SELECT ... FROM ...)`

現在、このようなシナリオのサブクエリに関しては、サブクエリが相関サブクエリでない場合、TiDB は最適化段階で事前評価し、直接的に結果セットに置き換えます。以下の図に示すように、最適化段階で `EXISTS` サブクエリは事前に `TRUE` と評価されるため、最終的な実行結果には表示されません。

{{< copyable "sql" >}}

```sql
create table t1(a int);
create table t2(a int);
insert into t2 values(1);
explain select * from t1 where exists (select * from t2);
```

```sql
+------------------------+----------+-----------+---------------+--------------------------------+
| id                     | estRows  | task      | access object | operator info                  |
+------------------------+----------+-----------+---------------+--------------------------------+
| TableReader_12         | 10000.00 | root      |               | data:TableFullScan_11          |
| └─TableFullScan_11     | 10000.00 | cop[tikv] | table:t1      | keep order:false, stats:pseudo |
+------------------------+----------+-----------+---------------+--------------------------------+
```

前述の最適化では、オプティマイザーは自動的にステートメントの実行を最適化します。さらに、[`SEMI_JOIN_REWRITE`](/jp/optimizer-hints.md#semi_join_rewrite) ヒントを追加して、ステートメントをさらに書き換えることもできます。

このヒントを使用してクエリを書き換えない場合、実行計画でハッシュ結合が選択された場合、セミジョインクエリはサブクエリをハッシュテーブルを構築するだけに使用できます。この場合、サブクエリの結果が外部クエリよりも大きい場合、実行速度は期待よりも遅くなる可能性があります。

同様に、実行計画でインデックスジョインが選択された場合、セミジョインクエリは外部クエリをドライビングテーブルとしてのみ使用できます。この場合、サブクエリの結果が外部クエリよりも小さい場合、実行速度は期待よりも遅くなる可能性があります。

`SEMI_JOIN_REWRITE()` を使用してクエリを書き換えると、オプティマイザーは最適な実行計画を選択するために選択範囲を拡張できます。