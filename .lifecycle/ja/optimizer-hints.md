---
title: オプティマイザーヒント
summary: クエリ実行プランに影響を与えるためにオプティマイザーヒントを使用します
aliases: ['/docs/dev/optimizer-hints/','/docs/dev/reference/performance/optimizer-hints/']
---

# オプティマイザーヒント

TiDBはMySQL 5.7で導入されたコメントのような構文に基づくオプティマイザーヒントをサポートしています。たとえば、一般的な構文の1つは `/*+ HINT_NAME([t1_name [, t2_name] ...]) */` です。オプティマイザーヒントは、TiDBオプティマイザーがより効率の良いクエリプランを選択しない場合に推奨されています。

ヒントが効果を持たない状況に遭遇した場合は、[ヒントが効果を持たない場合の一般的な問題のトラブルシューティング](#troubleshoot-common-issues-that-hints-do-not-take-effect)を参照してください。

## 構文

オプティマイザーヒントは、`SELECT`、 `UPDATE`、または `DELETE`キーワードの後にある`/*+ ... */`コメント内に指定され、大文字と小文字は区別されません。オプティマイザーヒントは現在は `INSERT` 文ではサポートされていません。

複数のヒントを指定する場合は、カンマで区切ります。たとえば、以下のクエリは3つの異なるヒントを使用しています:

```sql
SELECT /*+ USE_INDEX(t1, idx1), HASH_AGG(), HASH_JOIN(t1) */ count(*) FROM t t1, t t2 WHERE t1.a = t2.b;
```

オプティマイザーヒントがクエリ実行プランに与える影響は [`EXPLAIN`](/sql-statements/sql-statement-explain.md) と [`EXPLAIN ANALYZE`](/sql-statements/sql-statement-explain-analyze.md) の出力で確認できます。

不正確または不完全なヒントは、文エラーにはなりません。これはヒントがクエリ実行に対してヒント（提案）的な意味を持つことを意図しているためです。同様に、TiDBはヒントが適用されない場合に警告を最大1つまで返します。

> **注記:**
>
> 指定されたキーワードの後にコメントが続かない場合、一般的なMySQLコメントと見なされます。コメントは効果を持たず、警告も報告されません。

`INL_JOIN(t1_name [, tl_name ...])`ヒントは、指定したテーブルに対してインデックス入れ子ループ結合アルゴリズムを使用するようにオプティマイザに指示します。このアルゴリズムは、特定のシナリオではシステムリソースを少なく消費し、短い処理時間で済むことがありますが、他のシナリオでは逆の結果を生むかもしれません。外側のテーブルが`WHERE`条件でフィルタリングされた後の結果セットが10,000行未満の場合、このヒントの使用を推奨します。例:

{{< copyable "sql" >}}

```sql
select /*+ INL_JOIN(t1, t2) */ * from t1, t2 where t1.id = t2.id;
```

`INL_JOIN()`内のパラメータは、クエリプランを作成するときの内部テーブルの候補です。例えば、`INL_JOIN(t1)`は、TiDBがクエリプランを作成する際に`t1`のみを内部テーブルとして使用することを意味します。候補テーブルにエイリアスがある場合、`INL_JOIN()`のパラメータとしてエイリアスを使用する必要があります。エイリアスがない場合は、テーブルの元の名前をパラメータとして使用します。たとえば、`select /*+ INL_JOIN(t1) */ * from t t1, t t2 where t1.a = t2.b;`クエリでは、`t`テーブルのエイリアス`t1`または`t2`を使用する必要があります。

> **注意:**
>
> `TIDB_INLJ`は、TiDB 3.0.xおよびそれ以前のバージョンで`INL_JOIN`のエイリアスです。これらのバージョンのいずれかを使用している場合、このヒントのために`TIDB_INLJ(t1_name [, tl_name ...])`構文を適用する必要があります。TiDBの後のバージョンでは、`TIDB_INLJ`と`INL_JOIN`の両方の名前がヒントとして有効ですが、`INL_JOIN`の使用が推奨されています。

### NO_INDEX_JOIN(t1_name [, tl_name ...])

`NO_INDEX_JOIN(t1_name [, tl_name ...])`ヒントは、指定したテーブルに対してインデックス入れ子ループ結合アルゴリズムを使用しないようにオプティマイザに指示します。例:

```sql
SELECT /*+ NO_INDEX_JOIN(t1, t2) */ * FROM t1, t2 WHERE t1.id = t2.id;
```

### INL_HASH_JOIN

`INL_HASH_JOIN(t1_name [, tl_name])`ヒントは、オプティマイザに対してインデックス入れ子ハッシュ結合アルゴリズムを使用するように指示します。このアルゴリズムの使用条件は、インデックス入れ子ループ結合アルゴリズムの使用条件と同じです。2つのアルゴリズムの違いは、`INL_JOIN`が結合された内部テーブルにハッシュテーブルを作成するのに対し、`INL_HASH_JOIN`は結合された外部テーブルに対してハッシュテーブルを作成する点です。`INL_HASH_JOIN`はメモリ使用量に固定の制限がありますが、`INL_JOIN`のメモリ使用量は内部テーブルで一致した行数に依存します。

### NO_INDEX_HASH_JOIN(t1_name [, tl_name ...])

`NO_INDEX_HASH_JOIN(t1_name [, tl_name ...])`ヒントは、指定したテーブルに対してインデックス入れ子ハッシュ結合アルゴリズムを使用しないようにオプティマイザに指示します。

### INL_MERGE_JOIN(t1_name [, tl_name])

`INL_MERGE_JOIN(t1_name [, tl_name])`ヒントは、オプティマイザに対してインデックス入れ子マージ結合アルゴリズムを使用するように指示します。このアルゴリズムの使用条件は、インデックス入れ子ループ結合アルゴリズムの使用条件と同じです。

### NO_INDEX_MERGE_JOIN(t1_name [, tl_name ...])

`NO_INDEX_MERGE_JOIN(t1_name [, tl_name ...])`ヒントは、指定したテーブルに対してインデックス入れ子マージ結合アルゴリズムを使用しないようにオプティマイザに指示します。

### HASH_JOIN(t1_name [, tl_name ...])

`HASH_JOIN(t1_name [, tl_name ...])`ヒントは、指定したテーブルに対してハッシュ結合アルゴリズムを使用するようにオプティマイザに指示します。このアルゴリズムは、クエリを複数のスレッドで並行実行することを可能にし、これによりより高速な処理速度が実現できますが、より多くのメモリを消費します。例:

{{< copyable "sql" >}}

```sql
select /*+ HASH_JOIN(t1, t2) */ * from t1, t2 where t1.id = t2.id;
```

> **注意:**
>
> `TIDB_HJ`は、TiDB 3.0.xおよびそれ以前のバージョンで`HASH_JOIN`のエイリアスです。これらのバージョンのいずれかを使用している場合、このヒントのために`TIDB_HJ(t1_name [, tl_name ...])`構文を適用する必要があります。TiDBの後のバージョンでは、`TIDB_HJ`と`HASH_JOIN`の両方の名前がヒントとして有効ですが、`HASH_JOIN`の使用が推奨されています。

### NO_HASH_JOIN(t1_name [, tl_name ...])

`NO_HASH_JOIN(t1_name [, tl_name ...])`ヒントは、指定したテーブルに対してハッシュ結合アルゴリズムを使用しないようにオプティマイザに指示します。例:

```sql
SELECT /*+ NO_HASH_JOIN(t1, t2) */ * FROM t1, t2 WHERE t1.id = t2.id;
```

### HASH_JOIN_BUILD(t1_name [, tl_name ...])

`HASH_JOIN_BUILD(t1_name [, tl_name ...])`ヒントは、指定したテーブルでハッシュ結合アルゴリズムを使用し、これらのテーブルがビルド側として機能するようにオプティマイザに指示します。このようにして、特定のテーブルを使用してハッシュテーブルを構築することができます。例:

```sql
SELECT /*+ HASH_JOIN_BUILD(t1) */ * FROM t1, t2 WHERE t1.id = t2.id;
```

### HASH_JOIN_PROBE(t1_name [, tl_name ...])

`HASH_JOIN_PROBE(t1_name [, tl_name ...])`ヒントは、指定したテーブルでハッシュ結合アルゴリズムを使用し、これらのテーブルがプローブ側として機能するようにオプティマイザに指示します。このようにして、特定のテーブルをプローブ側としてハッシュ結合アルゴリズムを実行することができます。例:

```sql
SELECT /*+ HASH_JOIN_PROBE(t2) */ * FROM t1, t2 WHERE t1.id = t2.id;
```

### SEMI_JOIN_REWRITE()

`SEMI_JOIN_REWRITE()`ヒントは、セミ結合クエリを通常の結合クエリに書き換えるようにオプティマイザに指示します。現在、このヒントは`EXISTS`サブクエリにのみ適用されます。

このヒントを使用してクエリを書き換えない場合、実行計画でハッシュ結合が選択された場合、セミ結合クエリはサブクエリを使用してハッシュテーブルを構築するだけです。この場合、サブクエリの結果が外側クエリの結果よりも大きい場合、実行速度は期待よりも遅くなる可能性があります。

同様に、実行計画でインデックス結合が選択された場合、セミ結合クエリは外側クエリを駆動テーブルとして使用するだけです。この場合、サブクエリの結果が外側クエリの結果よりも小さい場合、実行速度は期待よりも遅くなる可能性があります。

`SEMI_JOIN_REWRITE()`を使用してクエリを書き換えると、オプティマイザは選択範囲を拡張してより良い実行計画を選択できます。

{{< copyable "sql" >}}

```sql
-- `SEMI_JOIN_REWRITE()`を使用してクエリを書き換えない場合。
EXPLAIN SELECT * FROM t WHERE EXISTS (SELECT 1 FROM t1 WHERE t1.a = t.a);
```

```sql
+-----------------------------+---------+-----------+------------------------+---------------------------------------------------+
| id                          | estRows | task      | access object          | operator info                                     |
+-----------------------------+---------+-----------+------------------------+---------------------------------------------------+
| MergeJoin_9                 | 7992.00 | root      |                        | semi join, left key:test.t.a, right key:test.t1.a |
| ├─IndexReader_25(Build)     | 9990.00 | root      |                        | index:IndexFullScan_24                            |
| │ └─IndexFullScan_24        | 9990.00 | cop[tikv] | table:t1, index:idx(a) | keep order:true, stats:pseudo                     |
| └─IndexReader_23(Probe)     | 9990.00 | root      |                        | index:IndexFullScan_22                            |
|   └─IndexFullScan_22        | 9990.00 | cop[tikv] | table:t, index:idx(a)  | keep order:true, stats:pseudo                     |
+-----------------------------+---------+-----------+------------------------+---------------------------------------------------+
```

{{< copyable "sql" >}}

```sql
-- `SEMI_JOIN_REWRITE()`を使用してクエリを書き換える場合。
EXPLAIN SELECT * FROM t WHERE EXISTS (SELECT /*+ SEMI_JOIN_REWRITE() */ 1 FROM t1 WHERE t1.a = t.a);
```

```sql
+------------------------------+---------+-----------+------------------------+---------------------------------------------------------------------------------------------------------------+
| id                           | estRows | task      | access object          | operator info                                                                                                 |
+------------------------------+---------+-----------+------------------------+---------------------------------------------------------------------------------------------------------------+
| IndexJoin_16                 | 1.25    | root      |                        | inner join, inner:IndexReader_15, outer key:test.t1.a, inner key:test.t.a, equal cond:eq(test.t1.a, test.t.a) |
| ├─StreamAgg_39(Build)        | 1.00    | root      |                        | group by:test.t1.a, funcs:firstrow(test.t1.a)->test.t1.a                                                      |
```
| │ └─IndexReader_34           | 1.00    | root      |                        | index:IndexFullScan_33                                                                                        |
| │   └─IndexFullScan_33       | 1.00    | cop[tikv] | table:t1, index:idx(a) | keep order:true                                                                                               |
| └─IndexReader_15(Probe)      | 1.25    | root      |                        | index:Selection_14                                                                                            |
|   └─Selection_14             | 1.25    | cop[tikv] |                        | not(isnull(test.t.a))                                                                                         |
|     └─IndexRangeScan_13      | 1.25    | cop[tikv] | table:t, index:idx(a)  | range: decided by [eq(test.t.a, test.t1.a)], keep order:false, stats:pseudo                                   |
+------------------------------+---------+-----------+------------------------+---------------------------------------------------------------------------------------------------------------+
From the preceding execution plan, you can see that the optimizer does not perform decorrelation. The execution plan still contains the Apply operator. The filter condition (`t2.b = t1.b`) with the correlated column is still the filter condition when accessing the `t2` table.

### HASH_AGG()

`HASH_AGG（）`ヒントは、指定されたクエリブロック内のすべての集計関数にハッシュ集約アルゴリズムを使用するようにオプティマイザに指示します。このアルゴリズムにより、クエリを複数のスレッドで並行して実行し、処理速度を高めることができますが、より多くのメモリを消費します。例：

{{< copyable "sql" >}}

```sql
select /*+ HASH_AGG() */ count(*) from t1, t2 where t1.a > 10 group by t1.id;
```

### STREAM_AGG()

`STREAM_AGG()`ヒントは、指定されたクエリブロック内のすべての集計関数にストリーム集約アルゴリズムを使用するようにオプティマイザに指示します。一般的に、このアルゴリズムはメモリを少なく消費しますが、処理時間が長くなります。非常に大きなデータ量やシステムメモリが不十分な場合は、このヒントを使用することをお勧めします。例：

{{< copyable "sql" >}}

```sql
select /*+ STREAM_AGG() */ count(*) from t1, t2 where t1.a > 10 group by t1.id;
```

### MPP_1PHASE_AGG()

`MPP_1PHASE_AGG()`は、指定されたクエリブロック内のすべての集計関数にワンフェーズ集約アルゴリズムを使用するようにオプティマイザに指示します。このヒントはMPPモードでのみ効果があります。例：

```sql
SELECT /*+ MPP_1PHASE_AGG() */ COUNT(*) FROM t1, t2 WHERE t1.a > 10 GROUP BY t1.id;
```

> **注意:**
>
> このヒントを使用する前に、現在の TiDB クラスタがクエリで TiFlash MPP モードをサポートしていることを確認してください。詳細は、[Use TiFlash MPP Mode](/tiflash/use-tiflash-mpp-mode.md)を参照してください。

### MPP_2PHASE_AGG()

`MPP_2PHASE_AGG()`は、指定されたクエリブロック内のすべての集計関数に2フェーズ集約アルゴリズムを使用するようにオプティマイザに指示します。このヒントはMPPモードでのみ有効です。例：

```sql
SELECT /*+ MPP_2PHASE_AGG() */ COUNT(*) FROM t1, t2 WHERE t1.a > 10 GROUP BY t1.id;
```

> **注意:**
>
> このヒントを使用する前に、現在の TiDB クラスタがクエリで TiFlash MPP モードをサポートしていることを確認してください。詳細は、[Use TiFlash MPP Mode](/tiflash/use-tiflash-mpp-mode.md)を参照してください。

### USE_INDEX(t1_name, idx1_name [, idx2_name ...])

`USE_INDEX(t1_name, idx1_name [, idx2_name ...])`ヒントは、指定された` t1_name` テーブルに対して指定されたインデックス（複数可）のみを使用するようオプティマイザに指示します。たとえば、次のヒントを適用することは、`select * from t t1 use index(idx1, idx2);` ステートメントを実行することと同じ効果を持ちます。

{{< copyable "sql" >}}

```sql
SELECT /*+ USE_INDEX(t1, idx1, idx2) */ * FROM t1;
```

> **注意:**
>
> このヒントでは、インデックスの名前を指定せずにテーブル名のみを指定した場合、実行ではどのインデックスも考慮されませんが、テーブル全体がスキャンされます。

### FORCE_INDEX(t1_name, idx1_name [, idx2_name ...])

`FORCE_INDEX(t1_name, idx1_name [, idx2_name ...])`ヒントは、指定されたインデックスのみを使用するようにオプティマイザに指示します。

`FORCE_INDEX(t1_name, idx1_name [, idx2_name ...])`の使用法と効果は、`USE_INDEX(t1_name, idx1_name [, idx2_name ...])`の使用法と効果と同じです。

次の4つのクエリは同様の効果を持ちます:

{{< copyable "sql" >}}

```sql
SELECT /*+ USE_INDEX(t, idx1) */ * FROM t;
SELECT /*+ FORCE_INDEX(t, idx1) */ * FROM t;
SELECT * FROM t use index(idx1);
SELECT * FROM t force index(idx1);
```

### IGNORE_INDEX(t1_name, idx1_name [, idx2_name ...])

`IGNORE_INDEX(t1_name, idx1_name [, idx2_name ...])`ヒントは、指定された `t1_name` テーブルの指定されたインデックス（複数可）を無視するようにオプティマイザに指示します。たとえば、次のヒントを適用することは、`select * from t t1 ignore index(idx1, idx2);` ステートメントを実行することと同じ効果を持ちます。

{{< copyable "sql" >}}

```sql
select /*+ IGNORE_INDEX(t1, idx1, idx2) */ * from t t1;
```

### ORDER_INDEX(t1_name, idx1_name [, idx2_name ...])

`ORDER_INDEX(t1_name, idx1_name [, idx2_name ...])`ヒントは、指定されたテーブルに対して指定されたインデックスのみを使用し、指定されたインデックスを順序立てて読み取るようにオプティマイザに指示します。

> **警告:**
>
> このヒントは SQL ステートメントの失敗を引き起こす可能性があります。まずテストを実行することをお勧めします。テスト中にエラーが発生した場合は、ヒントを削除してください。テストが正常に実行される場合は、それを使用し続けることができます。

このヒントは通常、次のようなシナリオで適用されます:

```sql
CREATE TABLE t(a INT, b INT, key(a), key(b));
EXPLAIN SELECT /*+ ORDER_INDEX(t, a) */ a FROM t ORDER BY a LIMIT 10;
```

```sql
+----------------------------+---------+-----------+---------------------+-------------------------------+
| id                         | estRows | task      | access object       | operator info                 |
+----------------------------+---------+-----------+---------------------+-------------------------------+
| Limit_10                   | 10.00   | root      |                     | offset:0, count:10            |
| └─IndexReader_14           | 10.00   | root      |                     | index:Limit_13                |
|   └─Limit_13               | 10.00   | cop[tikv] |                     | offset:0, count:10            |
|     └─IndexFullScan_12     | 10.00   | cop[tikv] | table:t, index:a(a) | keep order:true, stats:pseudo |
+----------------------------+---------+-----------+---------------------+-------------------------------+
```

このクエリに対してオプティマイザは、`Limit + IndexScan(keep order: true)`および`TopN + IndexScan(keep order: false)`の2種類のプランを生成します。`ORDER_INDEX`ヒントが使用されると、オプティマイザはインデックスを順に読み取る最初のプランを選択します。

> **注意:**
>
> - クエリ自体がインデックスを順に読み取る必要がない場合（つまり、ヒントなしでオプティマイザがいかなる状況でもインデックスを順に読み取るプランを生成しない場合）、`ORDER_INDEX`ヒントが使用されるとエラー `Can't find a proper physical plan for this query` が発生します。この場合は、対応する `ORDER_INDEX` ヒントを削除する必要があります。
> - パーティションテーブルのインデックスは順に読み取ることができないため、パーティションテーブルとその関連インデックスに `ORDER_INDEX` ヒントを使用しないでください。

### NO_ORDER_INDEX(t1_name, idx1_name [, idx2_name ...])

`NO_ORDER_INDEX(t1_name, idx1_name [, idx2_name ...])`ヒントは、指定されたテーブルに対して指定されたインデックスのみを使用し、指定されたインデックスを順に読み取らないようにオプティマイザに指示します。このヒントは通常次のシナリオで適用されます。

次の例は、`SELECT * FROM t t1 use index(idx1, idx2);` のクエリステートメントと同等の効果を持つことを示しています:

```sql
CREATE TABLE t(a INT, b INT, key(a), key(b));
EXPLAIN SELECT /*+ NO_ORDER_INDEX(t, a) */ a FROM t ORDER BY a LIMIT 10;
```

```sql
+----------------------------+----------+-----------+---------------------+--------------------------------+
| id                         | estRows  | task      | access object       | operator info                  |
+----------------------------+----------+-----------+---------------------+--------------------------------+
| TopN_7                     | 10.00    | root      |                     | test.t.a, offset:0, count:10   |
| └─IndexReader_14           | 10.00    | root      |                     | index:TopN_13                  |
|   └─TopN_13                | 10.00    | cop[tikv] |                     | test.t.a, offset:0, count:10   |
|     └─IndexFullScan_12     | 10000.00 | cop[tikv] | table:t, index:a(a) | keep order:false, stats:pseudo |
+----------------------------+----------+-----------+---------------------+--------------------------------+
```

`NO_ORDER_INDEX` ヒントの例と同様に、オプティマイザはこのクエリに対して `Limit + IndexScan(keep order: true)`および`TopN + IndexScan(keep order: false)`の2種類のプランを生成します。`NO_ORDER_INDEX` ヒントが使用されると、オプティマイザはインデックスを順に読み取らない後者のプランを選択します。

### AGG_TO_COP()
`AGG_TO_COP()`ヒントは、最適化機能にクエリブロック内の集約操作をコプロセッサにプッシュダウンするように指示します。最適化機能がプッシュダウンすべき適切な集約関数をプッシュダウンしない場合、このヒントを使用することを推奨します。例：

```sql
select /*+ AGG_TO_COP() */ sum(t1.a) from t t1;
```

### LIMIT_TO_COP()

`LIMIT_TO_COP()`ヒントは、指定されたクエリブロック内の`Limit`および`TopN`演算子をコプロセッサにプッシュダウンするように最適化機能に指示します。最適化機能がそのような操作を行わない場合、このヒントを使用することを推奨します。例：

```sql
SELECT /*+ LIMIT_TO_COP() */ * FROM t WHERE a = 1 AND b > 10 ORDER BY c LIMIT 1;
```

### READ_FROM_STORAGE(TIFLASH[t1_name [, tl_name ...]], TIKV[t2_name [, tl_name ...]])

`READ_FROM_STORAGE(TIFLASH[t1_name [, tl_name ...]], TIKV[t2_name [, tl_name ...]])`ヒントは、最適化機能に特定のテーブルを特定のストレージエンジンから読み取るように指示します。現在、このヒントでは2つのストレージエンジンパラメータ（`TIKV`および`TIFLASH`）がサポートされています。テーブルにエイリアスがある場合は、そのエイリアスを`READ_FROM_STORAGE()`のパラメータとして使用し、エイリアスが存在しない場合は元のテーブル名をパラメータとして使用します。例：

```sql
select /*+ READ_FROM_STORAGE(TIFLASH[t1], TIKV[t2]) */ t1.a from t t1, t t2 where t1.a = t2.a;
```

### USE_INDEX_MERGE(t1_name, idx1_name [, idx2_name ...])

`USE_INDEX_MERGE(t1_name, idx1_name [, idx2_name ...])`ヒントは、指定したインデックスマージメソッドを使用して特定のテーブルにアクセスするように最適化機能に指示します。インデックスマージには2つのタイプがあります：交差タイプと結合タイプ。詳細については、[Explain Statements Using Index Merge](/explain-index-merge.md)を参照してください。

インデックスのリストを明示的に指定すると、TiDBはリストからインデックスを選択してインデックスマージを構築します。インデックスのリストを指定しない場合、TiDBは利用可能なすべてのインデックスからインデックスマージを構築するためのインデックスを選択します。

交差タイプのインデックスマージでは、指定されたインデックスのリストがヒントの必須パラメータです。結合タイプのインデックスマージでは、指定されたインデックスのリストがヒントのオプションパラメータです。次の例を参照してください。

```sql
SELECT /*+ USE_INDEX_MERGE(t1, idx_a, idx_b, idx_c) */ * FROM t1 WHERE t1.a > 10 OR t1.b > 10;
```

同じテーブルに複数の`USE_INDEX_MERGE`ヒントがある場合、最適化機能はこれらのヒントによって指定されたインデックスの和集合からインデックスを選択しようとします。

> **注意:**
>
> `USE_INDEX_MERGE`のパラメータは、列名ではなくインデックス名を参照しています。主キーのインデックス名は`primary`です。

### LEADING(t1_name [, tl_name ...])

`LEADING(t1_name [, tl_name ...])`ヒントは、実行計画の生成時に、ヒントで指定されたテーブル名の順序に従って複数のテーブル結合の順序を決定するよう、最適化機能に指示します。例：

```sql
SELECT /*+ LEADING(t1, t2) */ * FROM t1, t2, t3 WHERE t1.id = t2.id and t2.id = t3.id;
```

上記の複数のテーブル結合を含むクエリでは、`LEADING()`ヒントで指定されたテーブル名の順序によって結合の順序が決定されます。最適化機能はまず`t1`と`t2`を結合してそしてその結果を`t3`と結合します。このヒントは[`STRAIGHT_JOIN`](#straight_join)よりも一般的です。

`LEADING`ヒントは以下の状況では効果を発揮しません：

+ 複数の`LEADING`ヒントが指定されている。
+ `LEADING`ヒントで指定されたテーブル名が存在しない。
+ `LEADING`ヒントで重複したテーブル名が指定されている。
+ 最適化機能が`LEADING`ヒントで指定された順序で結合操作を実行できない。
+ すでに`straight_join()`ヒントが存在する。
+ クエリにアウタージョインと直積が含まれている。
+ 同時に`MERGE_JOIN`、`INL_JOIN`、`INL_HASH_JOIN`、および`HASH_JOIN`ヒントのいずれかが使用されている。

これらの状況では警告が生成されます。

```sql
-- 複数の`LEADING`ヒントが指定されています。
SELECT /*+ LEADING(t1, t2) LEADING(t3) */ * FROM t1, t2, t3 WHERE t1.id = t2.id and t2.id = t3.id;

-- `LEADING`ヒントが効果を発揮しない理由を確認するには、`show warnings`を実行してください。
SHOW WARNINGS;
```

```sql
+---------+------+-------------------------------------------------------------------------------------------------------------------+
| Level   | Code | Message                                                                                                           |
+---------+------+-------------------------------------------------------------------------------------------------------------------+
| Warning | 1815 | We can only use one leading hint at most, when multiple leading hints are used, all leading hints will be invalid |
+---------+------+-------------------------------------------------------------------------------------------------------------------+
```

> **注意:**
>
> クエリステートメントがアウタージョインを含む場合、ヒントには結合順序を入れ替えることができるテーブルのみを指定できます。ヒントで結合順序を入れ替えることができないテーブルが含まれている場合、そのヒントは無効になります。例えば、`SELECT * FROM t1 LEFT JOIN (t2 JOIN t3 JOIN t4) ON t1.a = t2.a;`において、`t2`、`t3`、`t4`の結合順序を制御したい場合、`LEADING`ヒントに`t1`を指定することはできません。
ビュー `v` について、クエリ文から始まるリスト内の最初のビュー名（`ViewName@QueryBlockName [.ViewName@QueryBlockName .ViewName@QueryBlockName ...]`）は `v@SEL_1` です。ビュー `v` の最初のクエリブロックは `QB_NAME(v_1, v@SEL_1 .@SEL_1)` と宣言できます。または `QB_NAME(v_1, v)` と書くこともでき、`@SEL_1` を省略して `QB_NAME(v_1, v)` とすることができます：

    ```sql
    CREATE VIEW v AS SELECT /* コメント: 現在のクエリブロックの名前はデフォルトの @SEL_1 です */ * FROM t;

    -- グローバルヒントを指定します
    SELECT /*+ QB_NAME(v_1, v) USE_INDEX(t@v_1, idx) */ * FROM v;
    ```

- ネストされたビューとサブクエリを含む複雑な文の場合、以下の例はビュー `v1` と `v2` のそれぞれのクエリブロックの名前を指定しています：

    ```sql
    SELECT /* コメント: 現在のクエリブロックの名前はデフォルトの @SEL_1 です */ * FROM v2 JOIN (
        SELECT /* コメント: 現在のクエリブロックの名前はデフォルトの @SEL_2 です */ * FROM v2) vv;
    ```

    最初のビュー `v2` では、最初のクエリ文から始まるビュー名は `v2@SEL_1` です。2 番目のビュー `v2` では、最初のビュー名は `v2@SEL_2` です。次の例では、最初のビュー `v2` のみを考慮しています。

    ビュー `v2` の最初のクエリブロックは `QB_NAME(v2_1, v2@SEL_1 .@SEL_1)` と宣言でき、ビュー `v2` の2 番目のクエリブロックは `QB_NAME(v2_2, v2@SEL_1 .@SEL_2)` と宣言できます：

    ```sql
    CREATE VIEW v2 AS
        SELECT * FROM t JOIN /* コメント: ビュー `v2` の現在のクエリブロックの名前はデフォルトの @SEL_1 です。したがって、現在のクエリブロックビューリストは v2@SEL_1 .@SEL_1 です */ 
        (
            SELECT COUNT(*) FROM t1 JOIN v1 /* コメント: ビュー `v2` の現在のクエリブロックの名前はデフォルトの @SEL_2 です。したがって、現在のクエリブロックビューリストは v2@SEL_1 .@SEL_2 です */
        ) tt;
    ```

    ビュー `v1` では、直前の文から始まるビューリスト内の最初のビュー名は `v2@SEL_1 .v1@SEL_2` です。ビュー `v1` の最初のクエリブロックは `QB_NAME(v1_1, v2@SEL_1 .v1@SEL_2 .@SEL_1)` と宣言でき、2 番目のクエリブロックは `QB_NAME(v1_2, v2@SEL_1 .v1@SEL_2 .@SEL_2)` と宣言できます：

    ```sql
    CREATE VIEW v1 AS SELECT * FROM t JOIN /* コメント: ビュー `v1` の現在のクエリブロックの名前はデフォルトの @SEL_1 です。したがって、現在のクエリブロックビューリストは v2@SEL_1 .@SEL_2 .v1@SEL_1 です */
        (
            SELECT COUNT(*) FROM t1 JOIN t2 /* コメント: ビュー `v1` の現在のクエリブロックの名前はデフォルトの @SEL_2 です。したがって、現在のクエリブロックビューリストは v2@SEL_1 .@SEL_2 .v1@SEL_2 です */
        ) tt;
    ```

> **注意:**
>
> - ビューでグローバルヒントを使用する場合は、ビューで対応する `QB_NAME` ヒントを定義する必要があります。そうしないと、グローバルヒントは適用されません。
>
> - ヒントを使用してビュー内の複数のテーブル名を指定する場合は、同じヒント内のテーブル名が同じビューの同じクエリブロック内にあることを確認する必要があります。
>
> - ビューで `QB_NAME` ヒントを外側のクエリブロックに定義する場合:
>
>     - `QB_NAME` のビューリスト内の最初のアイテムの場合、`@SEL_` が明示的に宣言されていない場合、デフォルトは `QB_NAME` の定義されたクエリブロック位置と一致します。つまり、`SELECT /*+ QB_NAME(qb1, v2) */ * FROM v2 JOIN (SELECT /*+ QB_NAME(qb2, v2) */ * FROM v2) vv;` は `SELECT /*+ QB_NAME(qb1, v2@SEL_1) */ * FROM v2 JOIN (SELECT /*+ QB_NAME(qb2, v2@SEL_2) */ * FROM v2) vv;` と等価です。
>     - `QB_NAME` のビューリスト内の最初のアイテム以外では、`@SEL_1` のみを省略できます。つまり、現在のビューの最初のクエリブロックで `@SEL_1` が宣言されている場合、`@SEL_1` を省略できます。それ以外の場合は `@SEL_` を省略できません。前述の例について:
>
>         - ビュー `v2` の最初のクエリブロックは `QB_NAME(v2_1, v2)` と宣言できます。
>         - ビュー `v2` の2 番目のクエリブロックは `QB_NAME(v2_2, v2.@SEL_2)` と宣言できます。
>         - ビュー `v1` の最初のクエリブロックは `QB_NAME(v1_1, v2.v1@SEL_2)` と宣言できます。
>         - ビュー `v1` の2 番目のクエリブロックは `QB_NAME(v1_2, v2.v1@SEL_2 .@SEL_2)` と宣言できます。
```sql
select /*+ USE_TOJA(TRUE) */ t1.a, t1.b from t1 where t1.a in (select t2.a from t2) subq;
```

このヒントに加えて、`tidb_opt_insubq_to_join_and_agg` システム変数を設定することで、この機能を有効にするかどうかも制御できます。

### MAX_EXECUTION_TIME(N)

`MAX_EXECUTION_TIME(N)` ヒントは、サーバーがそれを中止する前にステートメントが許可されている実行時間に制限 `N` (ミリ秒単位のタイムアウト値) を配置します。次のヒントでは、`MAX_EXECUTION_TIME(1000)` はタイムアウトが 1000 ミリ秒（つまり1秒）であることを意味します。

{{< copyable "sql" >}}

```sql
select /*+ MAX_EXECUTION_TIME(1000) */ * from t1 inner join t2 where t1.id = t2.id;
```

このヒントに加えて、`global.max_execution_time` システム変数もステートメントの実行時間を制限することができます。

### MEMORY_QUOTA(N)

`MEMORY_QUOTA(N)` ヒントは、ステートメントが使用するメモリ量を制限 `N` (MB または GB 単位のしきい値値) に配置します。ステートメントのメモリ使用量がこの制限を超えると、TiDB はそのステートメントの制限を超えた動作に基づいてログメッセージを生成するか、ステートメントを単に中止します。

次のヒントでは、`MEMORY_QUOTA(1024 MB)` はメモリ使用量が 1024 MB に制限されていることを意味します：

{{< copyable "sql" >}}

```sql
select /*+ MEMORY_QUOTA(1024 MB) */ * from t;
```

このヒントに加えて、[`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query) システム変数もステートメントのメモリ使用量を制限することができます。

### READ_CONSISTENT_REPLICA()

`READ_CONSISTENT_REPLICA()` ヒントは、TiKV フォロワーノードから一貫性のあるデータを読み込む機能を有効にします。例：

{{< copyable "sql" >}}

```sql
select /*+ READ_CONSISTENT_REPLICA() */ * from t;
```

このヒントに加えて、`tidb_replica_read` 環境変数を `'follower'` または `'leader'` に設定することで、この機能を有効にするかどうかも制御できます。

### IGNORE_PLAN_CACHE()

`IGNORE_PLAN_CACHE()` ヒントは、現在の `prepare` ステートメントを処理する際にオプティマイザにプランキャッシュを使用しないように指示します。

このヒントは、[prepare-plan-cache](/sql-prepared-plan-cache.md) が有効になっているときに特定の種類のクエリのプランキャッシュを一時的に無効にするために使用されます。

次の例では、`prepare` ステートメントを実行するときに、プランキャッシュが強制的に無効にされます。

{{< copyable "sql" >}}

```sql
prepare stmt from 'select  /*+ IGNORE_PLAN_CACHE() */ * from t where t.id = ?';
```

### STRAIGHT_JOIN()

`STRAIGHT_JOIN()` ヒントは、ジョインプランを生成する際に、`FROM` 句内のテーブル名の順序でテーブルを結合することをオプティマイザに指示します。

{{< copyable "sql" >}}

```sql
SELECT /*+ STRAIGHT_JOIN() */ * FROM t t1, t t2 WHERE t1.a = t2.a;
```

> **注意:**
>
> - `STRAIGHT_JOIN` は `LEADING` よりも優先度が高くなります。両方のヒントが使用された場合、`LEADING` は効果を持ちません。
> - `STRAIGHT_JOIN` ヒントよりも汎用性が高い `LEADING` ヒントを使用することをお勧めします。

### NTH_PLAN(N)

`NTH_PLAN(N)` ヒントは、物理最適化中に見つかった `N` 番目の物理計画をオプティマイザに選択するように指示します。`N` は正の整数である必要があります。

指定した `N` が物理最適化の検索範囲を超えている場合、TiDB は警告を返し、このヒントを無視して最適な物理計画を選択します。

このヒントはカスケードプランナーが有効になっている場合は効果を発揮しません。

次の例では、オプティマイザは物理最適化中に見つかった 3 番目の物理計画を強制的に選択するように指示されます：

{{< copyable "sql" >}}

```sql
SELECT /*+ NTH_PLAN(3) */ count(*) from t where a > 5;
```

> **注意:**
> 
> `NTH_PLAN(N)` は主にテストに使用され、将来のバージョンでの互換性は保証されていません。このヒントを**慎重に**使用してください。

### RESOURCE_GROUP(resource_group_name)

`RESOURCE_GROUP(resource_group_name)` は[リソース制御](/tidb-resource-control.md)に使用され、リソースを分離します。このヒントは指定されたリソースグループを使用して現在のステートメントを一時的に実行します。指定したリソースグループが存在しない場合、このヒントは無視されます。

例：

```sql
SELECT /*+ RESOURCE_GROUP(rg1) */ * FROM t limit 10;
```

## ヒントが効果を持たない一般的な問題のトラブルシューティング

### ヒントが効果を持たないのは、MySQL のコマンドラインクライアントがヒントを削除しているためです

MySQL のコマンドラインクライアントは 5.7.7 より前のバージョンでは、デフォルトでオプティマイザのヒントを削除します。これらの以前のバージョンでヒント構文を使用したい場合は、クライアントの起動時に `--comments` オプションを追加してください。たとえば: `mysql -h 127.0.0.1 -P 4000 -uroot --comments`。

### ヒントが効果を持たないのは、データベース名が指定されていないためです

接続を作成する際にデータベース名が指定されていない場合、ヒントは効果を持たない可能性があります。例えば：

TiDB に接続する際に、`mysql -h127.0.0.1 -P4000 -uroot` コマンドに `-D` オプションを追加せずに接続し、次の SQL ステートメントを実行した場合：

```sql
SELECT /*+ use_index(t, a) */ a FROM test.t;
SHOW WARNINGS;
```

テーブル `t` のデータベースを TiDB が識別できないため、`use_index(t, a)` ヒントは効果を持ちません。

```sql
+---------+------+----------------------------------------------------------------------+
| Level   | Code | Message                                                              |
+---------+------+----------------------------------------------------------------------+
| Warning | 1815 | use_index(.t, a) is inapplicable, check whether the table(.t) exists |
+---------+------+----------------------------------------------------------------------+
1 row in set (0.00 sec)
```

### ヒントが効果を持たないのは、クロステーブルクエリでデータベース名が明示的に指定されていないためです

クロステーブルクエリを実行する際に、データベース名が明示的に指定されていない場合、ヒントが効果を持たない可能性があります。例：

```sql
USE test1;
CREATE TABLE t1(a INT, KEY(a));
USE test2;
CREATE TABLE t2(a INT, KEY(a));
SELECT /*+ use_index(t1, a) */ * FROM test1.t1, t2;
SHOW WARNINGS;
```

前述のステートメントでは、`t1` テーブルが現在の `test2` データベースに存在しないため、`use_index(t1, a)` ヒントは効果を持ちません。

```sql
+---------+------+----------------------------------------------------------------------------------+
| Level   | Code | Message                                                                          |
+---------+------+----------------------------------------------------------------------------------+
| Warning | 1815 | use_index(test2.t1, a) is inapplicable, check whether the table(test2.t1) exists |
+---------+------+----------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

この場合、`use_index(test1.t1, a)` を使用してデータベース名を明示的に指定する必要があります。

### ヒントが効果を持たないのは、間違った場所に配置されているためです

ヒントが特定のキーワードの直後に配置されていない場合、ヒントは効果を持たない可能性があります。例：

```sql
SELECT * /*+ use_index(t, a) */ FROM t;
SHOW WARNINGS;
```

警告は次のようになります：

```sql
+---------+------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Level   | Code | Message                                                                                                                                                                                                                 |
+---------+------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Warning | 1064 | You have an error in your SQL syntax; check the manual that corresponds to your TiDB version for the right syntax to use [parser:8066]Optimizer hint can only be followed by certain keywords like SELECT, INSERT, etc. |
+---------+------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.01 sec)
```

この場合、ヒントを `SELECT` キーワードの直後に配置する必要があります。詳細については、[構文](#syntax) セクションを参照してください。

### INL_JOIN ヒントが効果を持たないのは、結合キーの照合が互換性のないためです

2つのテーブルの結合キーの照合が互換性のない場合、`IndexJoin` 演算子を使用してクエリを実行することはできません。この場合、[`INL_JOIN` ヒント](#inl_joint1_name--tl_name-) は効果を持ちません。例：

```sql
CREATE TABLE t1 (k varchar(8), key(k)) COLLATE=utf8mb4_general_ci;
CREATE TABLE t2 (k varchar(8), key(k)) COLLATE=utf8mb4_bin;
EXPLAIN SELECT /*+ tidb_inlj(t1) */ * FROM t1, t2 WHERE t1.k=t2.k;
```

実行計画は次のようになります：

```sql
+-----------------------------+----------+-----------+----------------------+----------------------------------------------+
| id                          | estRows  | task      | access object        | operator info                                |
+-----------------------------+----------+-----------+----------------------+----------------------------------------------+
```
| HashJoin_19                 | 12487.50 | root      |                      | inner join, equal:[eq(test.t1.k, test.t2.k)] |
| ├─IndexReader_24(Build)     | 9990.00  | root      |                      | index:IndexFullScan_23                       |
| │ └─IndexFullScan_23        | 9990.00  | cop[tikv] | table:t2, index:k(k) | keep order:false, stats:pseudo               |
| └─IndexReader_22(Probe)     | 9990.00  | root      |                      | index:IndexFullScan_21                       |
|   └─IndexFullScan_21        | 9990.00  | cop[tikv] | table:t1, index:k(k) | keep order:false, stats:pseudo               |
+-----------------------------+----------+-----------+----------------------+----------------------------------------------+
5 行中 1 行が警告を含んでいます (0.00 sec)
```

前述の文において、`t1.k`と`t2.k`の照合は互換性がありません（それぞれ`utf8mb4_general_ci`および`utf8mb4_bin`）のため、`INL_JOIN`または`TIDB_INLJ`ヒントが効果を発揮しません。

```sql
SHOW WARNINGS;
+---------+------+----------------------------------------------------------------------------+
| Level   | Code | Message                                                                    |
+---------+------+----------------------------------------------------------------------------+
| Warning | 1815 | Optimizer Hint /*+ INL_JOIN(t1) */ or /*+ TIDB_INLJ(t1) */ is inapplicable |
+---------+------+----------------------------------------------------------------------------+
1 行が表示されました (0.00 sec)
```

### 結合順序のために`INL_JOIN`ヒントが効果を発揮しません

[`INL_JOIN(t1, t2)`](#inl_joint1_name--tl_name-)または`TIDB_INLJ(t1, t2)`ヒントは、`t1`および`t2`を他のテーブルと結合させるために`IndexJoin`演算子として直接結合させるのではなく、内部テーブルとして機能させることを明示的に指示します。たとえば：

```sql
EXPLAIN SELECT /*+ inl_join(t1, t3) */ * FROM t1, t2, t3 WHERE t1.id = t2.id AND t2.id = t3.id AND t1.id = t3.id;
+---------------------------------+----------+-----------+---------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| id                              | estRows  | task      | access object | operator info                                                                                                                                                           |
+---------------------------------+----------+-----------+---------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| IndexJoin_16                    | 15625.00 | root      |               | inner join, inner:TableReader_13, outer key:test.t2.id, test.t1.id, inner key:test.t3.id, test.t3.id, equal cond:eq(test.t1.id, test.t3.id), eq(test.t2.id, test.t3.id) |
| ├─IndexJoin_34(Build)           | 12500.00 | root      |               | inner join, inner:TableReader_31, outer key:test.t2.id, inner key:test.t1.id, equal cond:eq(test.t2.id, test.t1.id)                                                     |
| │ ├─TableReader_40(Build)       | 10000.00 | root      |               | data:TableFullScan_39                                                                                                                                                   |
| │ │ └─TableFullScan_39          | 10000.00 | cop[tikv] | table:t2      | keep order:false, stats:pseudo                                                                                                                                          |
| │ └─TableReader_31(Probe)       | 10000.00 | root      |               | data:TableRangeScan_30                                                                                                                                                  |
| │   └─TableRangeScan_30         | 10000.00 | cop[tikv] | table:t1      | range: decided by [test.t2.id], keep order:false, stats:pseudo                                                                                                          |
| └─TableReader_13(Probe)         | 12500.00 | root      |               | data:TableRangeScan_12                                                                                                                                                  |
|   └─TableRangeScan_12           | 12500.00 | cop[tikv] | table:t3      | range: decided by [test.t2.id test.t1.id], keep order:false, stats:pseudo                                                                                               |
+---------------------------------+----------+-----------+---------------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```

上記の例では、`t1`と`t3`は直接`IndexJoin`で結合されていません。

`t1`と`t3`の直接の`IndexJoin`を実行するためには、まず[`LEADING(t1, t3)`ヒント](#leadingt1_name--tl_name-)を使用して`t1`と`t3`の結合順序を指定し、次に`INL_JOIN`ヒントを使用して結合アルゴリズムを指定します。たとえば：

```sql
EXPLAIN SELECT /*+ leading(t1, t3), inl_join(t3) */ * FROM t1, t2, t3 WHERE t1.id = t2.id AND t2.id = t3.id AND t1.id = t3.id;
+---------------------------------+----------+-----------+---------------+---------------------------------------------------------------------------------------------------------------------+
| id                              | estRows  | task      | access object | operator info                                                                                                       |
+---------------------------------+----------+-----------+---------------+---------------------------------------------------------------------------------------------------------------------+
| Projection_12                   | 15625.00 | root      |               | test.t1.id, test.t1.name, test.t2.id, test.t2.name, test.t3.id, test.t3.name                                        |
| └─HashJoin_21                   | 15625.00 | root      |               | inner join, equal:[eq(test.t1.id, test.t2.id) eq(test.t3.id, test.t2.id)]                                           |
|   ├─TableReader_36(Build)       | 10000.00 | root      |               | data:TableFullScan_35                                                                                               |
|   │ └─TableFullScan_35          | 10000.00 | cop[tikv] | table:t2      | keep order:false, stats:pseudo                                                                                      |
|   └─IndexJoin_28(Probe)         | 12500.00 | root      |               | inner join, inner:TableReader_25, outer key:test.t1.id, inner key:test.t3.id, equal cond:eq(test.t1.id, test.t3.id) |
|     ├─TableReader_34(Build)     | 10000.00 | root      |               | data:TableFullScan_33                                                                                               |
|     │ └─TableFullScan_33        | 10000.00 | cop[tikv] | table:t1      | keep order:false, stats:pseudo                                                                                      |
|     └─TableReader_25(Probe)     | 10000.00 | root      |               | data:TableRangeScan_24                                                                                              |
|       └─TableRangeScan_24       | 10000.00 | cop[tikv] | table:t3      | range: decided by [test.t1.id], keep order:false, stats:pseudo                                                      |
+---------------------------------+----------+-----------+---------------+---------------------------------------------------------------------------------------------------------------------+
9 行が表示されました (0.01 秒)
```

### ヒントの使用により`このクエリに適切な物理プランが見つかりません`エラーが発生する

`このクエリに適切な物理プランが見つかりません`エラーは、次のシナリオで発生する可能性があります：

- クエリ自体がインデックスを順番に読む必要がない場合。つまり、このクエリに対しては、ヒントを使用せずにどの場合でも最適化プログラムがインデックスを順番に読む計画を生成しません。この場合、`ORDER_INDEX`ヒントが指定されていると、このエラーが発生します。この問題を解決するには、対応する`ORDER_INDEX`ヒントを削除します。
- クエリが`NO_JOIN`関連ヒントを使用してすべての可能な結合方法を除外している場合。

```sql
CREATE TABLE t1 (a INT);
CREATE TABLE t2 (a INT);
EXPLAIN SELECT /*+ NO_HASH_JOIN(t1), NO_MERGE_JOIN(t1) */ * FROM t1, t2 WHERE t1.a=t2.a;
ERROR 1815 (HY000): Internal : Can't find a proper physical plan for this query
```

- システム変数[`tidb_opt_enable_hash_join`](/system-variables.md#tidb_opt_enable_hash_join-new-in-v740)が`OFF`に設定されており、他のすべての結合種類も除外されている場合。

```sql
CREATE TABLE t1 (a INT);
CREATE TABLE t2 (a INT);
set tidb_opt_enable_hash_join=off;
EXPLAIN SELECT /*+ NO_MERGE_JOIN(t1) */ * FROM t1, t2 WHERE t1.a=t2.a;
ERROR 1815 (HY000): Internal : Can't find a proper physical plan for this query
```