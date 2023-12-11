---
title: TiDB クエリ実行プランの概要
summary: `EXPLAIN` ステートメントによって返される実行プラン情報について学びます。
aliases: ['/docs/dev/query-execution-plan/','/docs/dev/reference/performance/understanding-the-query-execution-plan/','/docs/dev/index-merge/','/docs/dev/reference/performance/index-merge/','/tidb/dev/index-merge','/tidb/dev/query-execution-plan']
---

# TiDB クエリ実行プランの概要

> **注:**
>
> TiDB に MySQL クライアントを使用して接続する場合、出力結果を改行せずによりわかりやすく読み取るために、`pager less -S` コマンドを使用できます。次に、`EXPLAIN` 結果が出力された後、キーボードの右矢印 <kbd>→</kbd> ボタンを押して、出力を水平にスクロールすることができます。

SQL は宣言的な言語です。クエリの結果がどのように見えるかを記述しますが、実際にその結果を取得するための手法ではありません。TiDB は、クエリの実行方法の可能なすべての方法を検討し、どの順序でテーブルを結合し、どのような潜在的なインデックスを使用できるかを含めて実行する可能性のある方法をすべて検討します。クエリ実行計画を検討するプロセスは SQL 最適化として知られています。

`EXPLAIN` ステートメントは、指定されたステートメントに対する選択された実行プランを表示します。つまり、クエリの実行方法が数百から数千通り考慮された後、TiDB はこの _プラン_ が最も少ないリソースを消費し、最短の時間で実行されると信じています：

{{< copyable "sql" >}}

```sql
CREATE TABLE t (id INT NOT NULL PRIMARY KEY auto_increment, a INT NOT NULL, pad1 VARCHAR(255), INDEX(a));
INSERT INTO t VALUES (1, 1, 'aaa'),(2,2, 'bbb');
EXPLAIN SELECT * FROM t WHERE a = 1;
```

```sql
Query OK, 0 rows affected (0.96 sec)

Query OK, 2 rows affected (0.02 sec)
Records: 2  Duplicates: 0  Warnings: 0

+-------------------------------+---------+-----------+---------------------+---------------------------------------------+
| id                            | estRows | task      | access object       | operator info                               |
+-------------------------------+---------+-----------+---------------------+---------------------------------------------+
| IndexLookUp_10                | 10.00   | root      |                     |                                             |
| ├─IndexRangeScan_8(Build)     | 10.00   | cop[tikv] | table:t, index:a(a) | range:[1,1], keep order:false, stats:pseudo |
| └─TableRowIDScan_9(Probe)     | 10.00   | cop[tikv] | table:t             | keep order:false, stats:pseudo              |
+-------------------------------+---------+-----------+---------------------+---------------------------------------------+
3 rows in set (0.00 sec)
```

`EXPLAIN` は実際のクエリを実行しません。[`EXPLAIN ANALYZE`](/sql-statements/sql-statement-explain-analyze.md) を使用してクエリを実行し、`EXPLAIN` 情報を表示できます。これは、選択された実行プランが最適でない場合の診断に役立ちます。`EXPLAIN` のさらなる使用例については、次のドキュメントを参照してください：

* [インデックス](/explain-indexes.md)
* [結合](/explain-joins.md)
* [サブクエリ](/explain-subqueries.md)
* [集約](/explain-aggregation.md)
* [ビュー](/explain-views.md)
* [パーティション](/explain-partitions.md)

## EXPLAIN 出力の理解

以下は、上記の `EXPLAIN` ステートメントの出力を説明しています：

* `id` は、SQL ステートメントを実行するために必要な演算子またはサブタスクの名前を示します。詳細については[オペレータの概要](#operator-overview)を参照してください。
* `estRows` は、TiDB が処理すると予想する行数の見積もりを示します。この数値は、主キーまたはユニークキーに基づいたアクセス方法である場合など、辞書情報に基づくことがあります。また、CMSketch やヒストグラムなどの統計情報に基づくこともあります。
* `task` は、演算子が作業を実行している場所を示します。詳細については[タスクの概要](#task-overview)を参照してください。
* `access object` は、アクセスされているテーブル、パーティション、およびインデックスを示します。以上の例では、インデックスから列 `a` が使用されたことが示されています。これは、複合インデックスを持っている場合に役立ちます。
* `operator info` は、アクセスに関する追加情報を示します。詳細については[オペレータ情報の概要](#operator-info-overview)を参照してください。

> **注:**
>
> `IndexJoin` および `Apply` 演算子のプローブ側の子ノードすべての返された実行プランでは、v6.4.0 以前と v6.4.0 以降では、`estRows` の意味が異なります。
>
> v6.4.0 以前では、`estRows` はプローブ側演算子によって処理される見積もられた行数を、ビルド側演算子の各行ごとに意味していました。v6.4.0 以降では、`estRows` はプローブ側演算子によって処理される**合計行数**を意味します。`EXPLAIN ANALYZE` の結果で表示される実際の行数（`actRows` 列で示される）は合計行数を意味するため、v6.4.0 以降、`IndexJoin` および `Apply` 演算子のプローブ側子ノードの `estRows` と `actRows` の意味が一致します。
>
>
> 例：
>
> ```sql
> CREATE TABLE t1(a INT, b INT);
> CREATE TABLE t2(a INT, b INT, INDEX ia(a));
> EXPLAIN SELECT /*+ INL_JOIN(t2) */ * FROM t1 JOIN t2 ON t1.a = t2.a;
> EXPLAIN SELECT (SELECT a FROM t2 WHERE t2.a = t1.b LIMIT 1) FROM t1;
> ```
>
> ```sql
> -- v6.4.0 以前：
> +---------------------------------+----------+-----------+-----------------------+-----------------------------------------------------------------------------------------------------------------+
> | id                              | estRows  | task      | access object         | operator info                                                                                                   |
> +---------------------------------+----------+-----------+-----------------------+-----------------------------------------------------------------------------------------------------------------+
> | IndexJoin_12                    | 12487.50 | root      |                       | inner join, inner:IndexLookUp_11, outer key:test.t1.a, inner key:test.t2.a, equal cond:eq(test.t1.a, test.t2.a) |
> | ├─TableReader_24(Build)         | 9990.00  | root      |                       | data:Selection_23                                                                                               |
> | │ └─Selection_23                | 9990.00  | cop[tikv] |                       | not(isnull(test.t1.a))                                                                                          |
> | │   └─TableFullScan_22          | 10000.00 | cop[tikv] | table:t1              | keep order:false, stats:pseudo                                                                                  |
> | └─IndexLookUp_11(Probe)         | 1.25     | root      |                       |                                                                                                                 |
> |   ├─Selection_10(Build)         | 1.25     | cop[tikv] |                       | not(isnull(test.t2.a))                                                                                          |
> |   │ └─IndexRangeScan_8          | 1.25     | cop[tikv] | table:t2, index:ia(a) | range: decided by [eq(test.t2.a, test.t1.a)], keep order:false, stats:pseudo                                    |
> |   └─TableRowIDScan_9(Probe)     | 1.25     | cop[tikv] | table:t2              | keep order:false, stats:pseudo                                                                                  |
> +---------------------------------+----------+-----------+-----------------------+-----------------------------------------------------------------------------------------------------------------+
> +---------------------------------+----------+-----------+-----------------------+------------------------------------------------------------------------------+
> | id                              | estRows  | task      | access object         | operator info                                                                |
> +---------------------------------+----------+-----------+-----------------------+------------------------------------------------------------------------------+
> | Projection_12                   | 10000.00 | root      |                       | test.t2.a                                                                    |
> | └─Apply_14                      | 10000.00 | root      |                       | CARTESIAN left outer join                                                    |
> |   ├─TableReader_16(Build)       | 10000.00 | root      |                       | data:TableFullScan_15                                                        |
> |   │ └─TableFullScan_15          | 10000.00 | cop[tikv] | table:t1              | keep order:false, stats:pseudo                                               |
> |   └─Limit_17(Probe)             | 1.00     | root      |                       | offset:0, count:1                                                            |
> |     └─IndexReader_21            | 1.00     | root      |                       | index:Limit_20                                                               |
> |       └─Limit_20                | 1.00     | cop[tikv] |                       | offset:0, count:1                                                            |
> |         └─IndexRangeScan_19     | 1.00     | cop[tikv] | table:t2, index:ia(a) | range: decided by [eq(test.t2.a, test.t1.b)], keep order:false, stats:pseudo |
> ```
> +---------------------------------+----------+-----------+-----------------------+-----------------------------------+
>
> -- v6.4.0以降：
>
> -- v6.4.0以降の`IndexLookUp_11`、`Selection_10`、`IndexRangeScan_8`、`TableRowIDScan_9`の`estRows`列の値がv6.4.0よりも異なることがわかります。
> +---------------------------------+----------+-----------+-----------------------+-----------------------------------------------+
> | id                              | estRows  | task      | access object         | operator info                                 |
> +---------------------------------+----------+-----------+-----------------------+-----------------------------------------------+
> | IndexJoin_12                    | 12487.50 | root      |                       | inner join, inner:IndexLookUp_11, outer key:test.t1.a, inner key:test.t2.a, equal cond:eq(test.t1.a, test.t2.a) |
> | ├─TableReader_24(Build)         | 9990.00  | root      |                       | data:Selection_23                             |
> | │ └─Selection_23                | 9990.00  | cop[tikv] |                       | not(isnull(test.t1.a))                        |
> | │   └─TableFullScan_22          | 10000.00 | cop[tikv] | table:t1              | keep order:false, stats:pseudo                |
> | └─IndexLookUp_11(Probe)         | 12487.50 | root      |                       |                                               |
> |   ├─Selection_10(Build)         | 12487.50 | cop[tikv] |                       | not(isnull(test.t2.a))                        |
> |   │ └─IndexRangeScan_8          | 12500.00 | cop[tikv] | table:t2, index:ia(a) | range: decided by [eq(test.t2.a, test.t1.a)], keep order:false, stats:pseudo |
> |   └─TableRowIDScan_9(Probe)     | 12487.50 | cop[tikv] | table:t2              | keep order:false, stats:pseudo                |
> +---------------------------------+----------+-----------+-----------------------+-----------------------------------------------+
>
> -- v6.4.0以降：
>
> -- v6.4.0以降の`Limit_17`、`IndexReader_21`、`Limit_20`、および`IndexRangeScan_19`の`estRows`列の値がv6.4.0よりも異なることがわかります。
> +---------------------------------+----------+-----------+-----------------------+--------------------------------------+
> | id                              | estRows  | task      | access object         | operator info                        |
> +---------------------------------+----------+-----------+-----------------------+--------------------------------------+
> | Projection_12                   | 10000.00 | root      |                       | test.t2.a                            |
> | └─Apply_14                      | 10000.00 | root      |                       | CARTESIAN left outer join            |
> |   ├─TableReader_16(Build)       | 10000.00 | root      |                       | data:TableFullScan_15                |
> |   │ └─TableFullScan_15          | 10000.00 | cop[tikv] | table:t1              | keep order:false, stats:pseudo       |
> |   └─Limit_17(Probe)             | 10000.00 | root      |                       | offset:0, count:1                    |
> |     └─IndexReader_21            | 10000.00 | root      |                       | index:Limit_20                       |
> |       └─Limit_20                | 10000.00 | cop[tikv] |                       | offset:0, count:1                    |
> |         └─IndexRangeScan_19     | 10000.00 | cop[tikv] | table:t2, index:ia(a) | range: decided by [eq(test.t2.a, test.t1.b)], keep order:false, stats:pseudo |
> +---------------------------------+----------+-----------+-----------------------+--------------------------------------+
```
* `stats:pseudo`は、`estRows`に表示される推定値が正確でない可能性があることを示しています。TiDBは定期的に統計情報をバックグラウンドで更新します。また、`ANALYZE TABLE t`を実行することで手動で更新することもできます。

`EXPLAIN`文の実行後、異なるオペレーターは異なる情報を出力します。オプティマイザーの動作を制御し、それによって物理オペレーターの選択を制御するために、オプティマイザーヒントを使用することができます。例えば、`/*+ HASH_JOIN(t1, t2) */`は、オプティマイザーが`Hash Join`アルゴリズムを使用することを意味します。詳細については、[Optimizer Hints](/optimizer-hints.md)を参照してください。