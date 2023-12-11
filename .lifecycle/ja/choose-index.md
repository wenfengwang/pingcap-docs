---
title: インデックスの選択
summary: TiDBのクエリ最適化のための最適なインデックスを選択します。

# インデックスの選択

ストレージエンジンからのデータの読み取りはSQLの実行中に最も時間のかかるステップの1つです。現在、TiDBは異なるストレージエンジンと異なるインデックスからデータを読み取ることをサポートしています。クエリ実行のパフォーマンスは、適切なインデックスを選択するかどうかに大きく依存しています。

このドキュメントでは、テーブルにアクセスするためのインデックスの選択方法と、インデックス選択を制御するための関連方法を紹介します。

## テーブルのアクセス

インデックスの選択を紹介する前に、TiDBがテーブルにアクセスする方法、各方法をトリガーする条件、各方法の違い、長所と短所を理解することが重要です。

### テーブルにアクセスするための演算子

| 演算子 | トリガー条件 | 適用シナリオ | 説明 |
| :------- | :------- | :------- | :---- |
| PointGet / BatchPointGet | 1つ以上の単一ポイント範囲でテーブルにアクセスする場合。 | すべてのシナリオ | トリガーされた場合、通常は最も高速な演算子と見なされます。なぜなら、演算を行うためにcoprocessorインターフェースではなく、直接kvgetインターフェースを呼び出すからです。 |
| TableReader | なし | すべてのシナリオ | このTableReader演算子はTiKV用のものです。通常は、TiKVレイヤーからテーブルデータを直接スキャンする効率が最も低い演算子と見なされます。`_tidb_rowid`列の範囲クエリがある場合、または選択肢として他のテーブルアクセス演算子がない場合にのみ選択できます。 |
| TableReader | テーブルにレプリカがTiFlashノードにある場合。 | 読み取る列が少なく、評価する行が多い。 | このTableReader演算子はTiFlash用です。TiFlashは列ベースのストレージです。少数の列を計算し、多数の行を計算する必要がある場合は、この演算子を選択することを推奨します。 |
| IndexReader | テーブルに1つ以上のインデックスがあり、計算に必要な列がインデックスに含まれている場合。 | インデックスの範囲クエリが小さい場合、またはインデックス列に順序の要件がある場合。 | 複数のインデックスが存在する場合、コストの推定に基づいて適切なインデックスが選択されます。 |
| IndexLookupReader | テーブルに1つ以上のインデックスがあり、計算に必要な列がインデックスに完全に含まれていない場合。 | IndexReaderと同じ。 | インデックスが計算列を完全にカバーしていないため、TiDBはインデックスを読み取った後、テーブルから行を取得する必要があります。IndexReader演算子と比較して追加コストがかかります。 |
| IndexMerge | テーブルに複数のインデックスか多値インデックスがある場合。 | 多値インデックスまたは複数のインデックスを使用する場合。 | このオペレータを使用するには、[optimizer hints](/optimizer-hints.md)を指定するか、コストの推定に基づいてこのオペレータを自動的に選択させます。詳細については、[Explain Statements Using Index Merge](/explain-index-merge.md)を参照してください。 |

> **注意:**
>
> TableReader演算子は`_tidb_rowid`列インデックスに基づいており、TiFlashは列保存インデックスを使用しているため、インデックスの選択はテーブルへのアクセスのための演算子の選択と同じです。

## インデックスの選択規則

TiDBは、規則またはコストに基づいてインデックスを選択します。基本的な規則には、事前規則とスカイライン剪定があります。インデックスの選択時にはまず事前規則を試し、インデックスが事前規則を満たさない場合は、スカイライン剪定を使用して適切でないインデックスを除外し、その後テーブルへのアクセス演算子ごとのコスト推定に基づいて最小コストのインデックスを選択します。

### 規則に基づく選択

#### 事前規則

以下のヒューリスティック事前規則を使用して、TiDBはインデックスを選択します:

+ 規則1: インデックスが「完全一致のユニークインデックス + テーブルから行を取得する必要がない（つまり、インデックスによって生成されるプランがIndexReader演算子である）」を満たす場合、TiDBは直接このインデックスを選択します。

+ 規則2: インデックスが「完全一致のユニークインデックス + テーブルから行を取得する必要がある（つまり、インデックスによって生成されるプランがIndexReader演算子である）」を満たす場合、TiDBはテーブルから取得する行の数が最も少ないインデックスを候補のインデックスとして選択します。

+ 規則3: インデックスが「通常のインデックス + テーブルから行を取得する必要がなく + 読み取られる行数が特定の閾値の値よりも少ない」を満たす場合、TiDBは読み取られる行数が最も少ないインデックスを候補のインデックスとして選択します。

+ 規則4: 規則2と3に基づいてそれぞれ1つの候補のインデックスが選択される場合は、その候補のインデックスを選択します。規則2と3に基づいてそれぞれ2つの候補のインデックスが選択される場合は、テーブルから取得される行数（インデックスで取得される行数 + テーブルから取得する行数）が少ないインデックスを選択します。

上記の規則における「完全一致のインデックス」とは、それぞれのインデックス列が同じ条件を持っていることを意味します。`EXPLAIN FORMAT = 'verbose' ...`ステートメントを実行すると、事前規則がインデックスにマッチすると、TiDBはプリルールに一致するインデックスを警告レベルのノートとして出力します。

以下の例では、規則2の条件「ユニークインデックスに完全一致 + テーブルから行を取得する必要がある」を満たすインデックス`idx_b`があるため、TiDBはインデックス`idx_b`をアクセスパスとして選択し、`SHOW WARNING`はインデックス`idx_b`がプリルールに一致することを示すノートを返します。

```sql
mysql> CREATE TABLE t(a INT PRIMARY KEY, b INT, c INT, UNIQUE INDEX idx_b(b));
クエリ OK, 0 行が選択されました。実行時間: 0.01 秒

mysql> EXPLAIN FORMAT = 'verbose' SELECT b, c FROM t WHERE b = 3 OR b = 6;
+-------------------+---------+---------+------+-------------------------+------------------------------+
| id                | estRows | estCost | task | access object           | operator info                |
+-------------------+---------+---------+------+-------------------------+------------------------------+
| Batch_Point_Get_5 | 2.00    | 8.80    | root | table:t, index:idx_b(b) | keep order:false, desc:false |
+-------------------+---------+---------+------+-------------------------+------------------------------+
1 行が選択されました。警告 1件 (0.00 秒)
mysql> SHOW WARNINGS;
+-------+------+-------------------------------------------------------------------------------------------+
| レベル | コード | メッセージ                                                                  |
+-------+------+-------------------------------------------------------------------------------------------+
| Note  | 1105 | unique index idx_b of t is selected since the path only has point ranges with double scan |
+-------+------+-------------------------------------------------------------------------------------------+
1 行が選択されました。 (0.00 秒)
```

### スカイライン剪定

スカイライン剪定は、インデックスに対するヒューリスティックなフィルタリングルールであり、誤った推定による間違ったインデックスの選択の確率を減らすことができます。インデックスを判断するには、次の3つの次元が必要です:

- インデックス列がカバーするアクセス条件の数。ここでの「アクセス条件」とは、列の範囲に変換できるwhere条件のことです。そして、インデックス列セットがカバーするアクセス条件が多いほど、この次元で優れています。

- インデックスをテーブルへのアクセスに選択する際に、テーブルから行を取得する必要があるかどうか（つまり、インデックスによって生成されるプランがIndexReader演算子またはIndexLookupReader演算子であるかどうか）。テーブルから行を取得しないインデックスの方がこの次元で優れています。どちらのインデックスもテーブルから行を取得する必要がある場合は、インデックス列がどれだけのフィルタ条件をカバーするかを比較します。フィルタ条件とは、インデックスに基づいて判断できる`where`条件のことです。インデックスの列セットがより多くのアクセス条件をカバーすればするほど、取得される行の数が少なくなり、この次元でインデックスが優れています。

- インデックスが特定の順序を満たすかどうかの選択。インデックスの読み取りが特定の列セットの順序を保証できるため、クエリの順序を満たすインデックスは、この次元で優れています。

上記の3つの次元において、インデックス`idx_a`が全ての3つの次元でインデックス`idx_b`よりも劣らないとともに、少なくとも1つの次元で`idx_b`よりも優れている場合、`idx_a`が選好されます。`EXPLAIN FORMAT = 'verbose' ...` ステートメントを実行すると、スカイライン剪定でいくつかのインデックスが除外された場合、TiDBはスカイライン剪定の除外後の残りのインデックスをリストアップした警告レベルのノートを出力します。

以下の例では、インデックス`idx_b`と`idx_e`はどちらも`idx_b_c`よりも劣るため、スカイライン剪定によって除外されます。`SHOW WARNING`の返された結果は、スカイライン剪定の除外後の残りのインデックスを表示します。

```sql
mysql> CREATE TABLE t(a INT PRIMARY KEY, b INT, c INT, d INT, e INT, INDEX idx_b(b), INDEX idx_b_c(b, c), INDEX idx_e(e));
クエリ OK, 0 行が選択されました。実行時間: 0.01 秒

mysql> EXPLAIN FORMAT = 'verbose' SELECT * FROM t WHERE b = 2 AND c > 4;
+-------------------------------+---------+---------+-----------+------------------------------+----------------------------------------------------+
| id                            | estRows | estCost | task      | access object                | operator info                                      |
+-------------------------------+---------+---------+-----------+------------------------------+----------------------------------------------------+
| IndexLookUp_10                | 33.33   | 738.29  | root      |                              |                                                    |
| ├─IndexRangeScan_8(Build)     | 33.33   | 2370.00 | cop[tikv] | table:t, index:idx_b_c(b, c) | range:(2 4,2 +inf], keep order:false, stats:pseudo |
```
| └─TableRowIDScan_9(Probe)     | 33.33   | 2370.00 | cop[tikv] | table:t                      | keep order:false, stats:pseudo                     |
+-------------------------------+---------+---------+-----------+------------------------------+----------------------------------------------------+
3 rows in set, 1 warning (0.00 sec)

mysql> SHOW WARNINGS;
+-------+------+------------------------------------------------------------------------------------------+
| Level | Code | Message                                                                                  |
+-------+------+------------------------------------------------------------------------------------------+
| Note  | 1105 | [t,idx_b_c] remain after pruning paths for t given Prop{SortItems: [], TaskTp: rootTask} |
+-------+------+------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

### コスト推定に基づく選択

スカイラインプルーニングルールを使用して不適切なインデックスを除外した後、インデックスの選択は完全にコスト推定に基づいています。テーブルへのアクセスのコスト推定には次の考慮事項が必要です。

- ストレージエンジン内のインデックスデータの1行あたりの平均長。
- インデックスによって生成されたクエリ範囲内の行の数。
- テーブルから行を取得するコスト。
- クエリ実行中にインデックスによって生成される範囲の数。

これらの要因とコストモデルに従って、オプティマイザは最も低いコストでテーブルにアクセスするためにインデックスを選択します。

#### コスト推定に基づく選択の一般的なチューニング問題

1. 推定された行数が正確ではありませんか？

    これは通常、古いまたは正確でない統計情報のためです。`analyze table`ステートメントを再実行するか、`analyze table`ステートメントのパラメータを修正できます。

2. 統計情報は正確であり、TiFlashからの読み取りが速いのに、なぜオプティマイザがTiKVからの読み取りを選択するのですか？

    現在、TiFlashとTiKVを区別するコストモデルはまだ粗いです。`tidb_opt_seek_factor`パラメータの値を減らすと、オプティマイザはTiFlashを選択するようになります。

3. 統計情報は正確です。インデックスAはテーブルから行を取得する必要がありますが、実際には行を取得しないインデックスBよりも速く実行されます。なぜオプティマイザはインデックスBを選択するのですか？

    この場合、テーブルから行を取得するためのコスト推定が大きすぎる可能性があります。`tidb_opt_network_factor`パラメータの値を減らすと、テーブルから行を取得するコストを削減できます。

## インデックスの制御

インデックスの選択は、[オプティマイザヒント](/optimizer-hints.md)を介して単一のクエリで制御できます。

- `USE_INDEX` / `IGNORE_INDEX`は、オプティマイザに特定のインデックスの使用/非使用を強制します。`FORCE_INDEX`と`USE_INDEX`は同じ効果があります。

- `READ_FROM_STORAGE`は、オプティマイザに指定したテーブルのTiKV / TiFlashストレージエンジンを選択してクエリを実行させます。

## 複数値インデックスの使用

[複数値インデックス](/sql-statements/sql-statement-create-index.md#multi-valued-indexes)は通常のインデックスとは異なります。現在、TiDBは複数値インデックスへのアクセスに[IndexMerge](/explain-index-merge.md)を使用しています。したがって、データアクセスに複数値インデックスを使用するには、システム変数`tidb_enable_index_merge`の値が`ON`に設定されていることを確認してください。

複数値インデックスの制限事項については、[`CREATE INDEX`](/sql-statements/sql-statement-create-index.md#limitations)を参照してください。

### サポートされるシナリオ

現在、TiDBは`json_member_of`、`json_contains`、`json_overlaps`の条件から自動的に変換されるIndexMergeを使用して、複数値インデックスへのアクセスをサポートしています。コストに基づいてIndexMergeを自動的に選択するか、オプティマイザヒント[`use_index_merge`](/optimizer-hints.md#use_index_merget1_name-idx1_name--idx2_name-)または[`use_index`](/optimizer-hints.md#use_indext1_name-idx1_name--idx2_name-)を使用して複数値インデックスの選択を指定できます。次の例を参照してください。

```sql
mysql> CREATE TABLE t1 (j JSON, INDEX idx((CAST(j->'$.path' AS SIGNED ARRAY)))); -- '$.path'を使用して複数値インデックスを作成
Query OK, 0 rows affected (0.04 sec)

mysql> EXPLAIN SELECT /*+ use_index_merge(t1, idx) */ * FROM t1 WHERE (1 MEMBER OF (j->'$.path'));
+---------------------------------+---------+-----------+-----------------------------------------------------------------------------+------------------------------------------------------------------------+
| id                              | estRows | task      | access object                                                               | operator info                                                          |
+---------------------------------+---------+-----------+-----------------------------------------------------------------------------+------------------------------------------------------------------------+
| Selection_5                     | 8000.00 | root      |                                                                             | json_memberof(cast(1, json BINARY), json_extract(test.t1.j, "$.path")) |
| └─IndexMerge_8                  | 10.00   | root      |                                                                             | type: union                                                            |
|   ├─IndexRangeScan_6(Build)     | 10.00   | cop[tikv] | table:t1, index:idx(cast(json_extract(`j`, _utf8'$.path') as signed array)) | range:[1,1], keep order:false, stats:pseudo                            |
|   └─TableRowIDScan_7(Probe)     | 10.00   | cop[tikv] | table:t1                                                                    | keep order:false, stats:pseudo                                         |
+---------------------------------+---------+-----------+-----------------------------------------------------------------------------+------------------------------------------------------------------------+
4 rows in set, 1 warning (0.00 sec)

mysql> EXPLAIN SELECT /*+ use_index_merge(t1, idx) */ * FROM t1 WHERE JSON_CONTAINS((j->'$.path'), '[1, 2, 3]');
+-------------------------------+---------+-----------+-----------------------------------------------------------------------------+---------------------------------------------+
| id                            | estRows | task      | access object                                                               | operator info                               |
+-------------------------------+---------+-----------+-----------------------------------------------------------------------------+---------------------------------------------+
| IndexMerge_9                  | 10.00   | root      |                                                                             | type: intersection                          |
| ├─IndexRangeScan_5(Build)     | 10.00   | cop[tikv] | table:t1, index:idx(cast(json_extract(`j`, _utf8'$.path') as signed array)) | range:[1,1], keep order:false, stats:pseudo |
| ├─IndexRangeScan_6(Build)     | 10.00   | cop[tikv] | table:t1, index:idx(cast(json_extract(`j`, _utf8'$.path') as signed array)) | range:[2,2], keep order:false, stats:pseudo |
| ├─IndexRangeScan_7(Build)     | 10.00   | cop[tikv] | table:t1, index:idx(cast(json_extract(`j`, _utf8'$.path') as signed array)) | range:[3,3], keep order:false, stats:pseudo |
| └─TableRowIDScan_8(Probe)     | 10.00   | cop[tikv] | table:t1                                                                    | keep order:false, stats:pseudo              |
+-------------------------------+---------+-----------+-----------------------------------------------------------------------------+---------------------------------------------+
5 rows in set (0.00 sec)

mysql> EXPLAIN SELECT /*+ use_index_merge(t1, idx) */ * FROM t1 WHERE JSON_OVERLAPS((j->'$.path'), '[1, 2, 3]');
+---------------------------------+---------+-----------+-----------------------------------------------------------------------------+----------------------------------------------------------------------------------+
| id                              | estRows | task      | access object                                                               | operator info                                                                    |
+---------------------------------+---------+-----------+-----------------------------------------------------------------------------+----------------------------------------------------------------------------------+
| Selection_5                     | 8000.00 | root      |                                                                             | json_overlaps(json_extract(test.t1.j, "$.path"), cast("[1, 2, 3]", json BINARY)) |
| └─IndexMerge_10                 | 10.00   | root      |                                                                             | type: union                                                                      |
|   ├─IndexRangeScan_6(Build)     | 10.00   | cop[tikv] | table:t1, index:idx(cast(json_extract(`j`, _utf8'$.path') as signed array)) | range:[1,1], keep order:false, stats:pseudo                                      |
|   ├─IndexRangeScan_7(Build)     | 10.00   | cop[tikv] | table:t1, index:idx(cast(json_extract(`j`, _utf8'$.path') as signed array)) | range:[2,2], keep order:false, stats:pseudo                                      |
|   ├─IndexRangeScan_8(Build)     | 10.00   | cop[tikv] | table:t1, index:idx(cast(json_extract(`j`, _utf8'$.path') as signed array)) | range:[3,3], keep order:false, stats:pseudo                                      |
|   └─TableRowIDScan_9(Probe)     | 10.00   | cop[tikv] | table:t1                                                                    | keep order:false, stats:pseudo                                                   |
+---------------------------------+---------+-----------+-----------------------------------------------------------------------------+----------------------------------------------------------------------------------+
6 rows in set, 1 warning (0.00 sec)
```

複合複数値インデックスもIndexMergeを介してアクセスできます:

```sql
mysql> CREATE TABLE t2 (a INT, j JSON, b INT, INDEX idx(a, (CAST(j->'$.path' AS SIGNED ARRAY)), b));
Query OK, 0 rows affected (0.04 sec)

```sql
mysql> EXPLAIN SELECT /*+ use_index_merge(t2, idx) */ * FROM t2 WHERE a=1 AND (1メンバー OF (j->'$.path')) AND b=2;
+---------------------------------+---------+-----------+-----------------------------------------------------------------------------------+------------------------------------------------------------------------+
| id                              | estRows | task      | access object                                                                     | operator info                                                          |
+---------------------------------+---------+-----------+-----------------------------------------------------------------------------------+------------------------------------------------------------------------+
| Selection_5                     | 0.01    | root      |                                                                                   | json_memberof(cast(1, json BINARY), json_extract(test.t2.j,  "$.path")) |
| └─IndexMerge_8                  | 0.00    | root      |                                                                                   | type: union                                                            |
|   ├─IndexRangeScan_6(Build)     | 0.00    | cop[tikv] | table:t2, index:idx(a, cast(json_extract(`j`, _utf8'$.path') as signed array), b) | range:[1 1 2,1 1 2], keep order:false, stats:pseudo                    |
|   └─TableRowIDScan_7(Probe)     | 0.00    | cop[tikv] | table:t2                                                                          | keep order:false, stats:pseudo                                         |
+---------------------------------+---------+-----------+-----------------------------------------------------------------------------------+------------------------------------------------------------------------+
4 rows in set, 1 warning (0.00 sec)

mysql> EXPLAIN SELECT /*+ use_index_merge(t2, idx) */ * FROM t2 WHERE a=1 AND JSON_CONTAINS((j->'$.path'), '[1, 2, 3]');
+-------------------------------+---------+-----------+-----------------------------------------------------------------------------------+-------------------------------------------------+
| id                            | estRows | task      | access object                                                                     | operator info                                   |
+-------------------------------+---------+-----------+-----------------------------------------------------------------------------------+-------------------------------------------------+
| IndexMerge_9                  | 0.10    | root      |                                                                                   | type: intersection                              |
| ├─IndexRangeScan_5(Build)     | 0.10    | cop[tikv] | table:t2, index:idx(a, cast(json_extract(`j`, _utf8'$.path') as signed array), b) | range:[1 1,1 1], keep order:false, stats:pseudo |
| ├─IndexRangeScan_6(Build)     | 0.10    | cop[tikv] | table:t2, index:idx(a, cast(json_extract(`j`, _utf8'$.path') as signed array), b) | range:[1 2,1 2], keep order:false, stats:pseudo |
| ├─IndexRangeScan_7(Build)     | 0.10    | cop[tikv] | table:t2, index:idx(a, cast(json_extract(`j`, _utf8'$.path') as signed array), b) | range:[1 3,1 3], keep order:false, stats:pseudo |
| └─TableRowIDScan_8(Probe)     | 0.10    | cop[tikv] | table:t2                                                                          | keep order:false, stats:pseudo                  |
+-------------------------------+---------+-----------+-----------------------------------------------------------------------------------+-------------------------------------------------+
5 rows in set (0.00 sec)

mysql> EXPLAIN SELECT /*+ use_index_merge(t2, idx) */ * FROM t2 WHERE a=1 AND JSON_OVERLAPS((j->'$.path'), '[1, 2, 3]');
+---------------------------------+---------+-----------+-----------------------------------------------------------------------------------+----------------------------------------------------------------------------------+
| id                              | estRows | task      | access object                                                                     | operator info                                                                    |
+---------------------------------+---------+-----------+-----------------------------------------------------------------------------------+----------------------------------------------------------------------------------+
| Selection_5                     | 8.00    | root      |                                                                                   | json_overlaps(json_extract(test.t2.j, "$.path"), cast("[1, 2, 3]", json BINARY)) |
| └─IndexMerge_10                 | 0.10    | root      |                                                                                   | type: union                                                                      |
|   ├─IndexRangeScan_6(Build)     | 0.10    | cop[tikv] | table:t2, index:idx(a, cast(json_extract(`j`, _utf8'$.path') as signed array), b) | range:[1 1,1 1], keep order:false, stats:pseudo                                  |
|   ├─IndexRangeScan_7(Build)     | 0.10    | cop[tikv] | table:t2, index:idx(a, cast(json_extract(`j`, _utf8'$.path') as signed array), b) | range:[1 2,1 2], keep order:false, stats:pseudo                                  |
|   ├─IndexRangeScan_8(Build)     | 0.10    | cop[tikv] | table:t2, index:idx(a, cast(json_extract(`j`, _utf8'$.path') as signed array), b) | range:[1 3,1 3], keep order:false, stats:pseudo                                  |
|   └─TableRowIDScan_9(Probe)     | 0.10    | cop[tikv] | table:t2                                                                          | keep order:false, stats:pseudo                                                   |
+---------------------------------+---------+-----------+-----------------------------------------------------------------------------------+----------------------------------------------------------------------------------+
6 rows in set, 1 warning (0.00 sec)
```

For `OR` conditions composed of multiple `member of` expressions that can access the same multi-valued index, IndexMerge can be used to access the multi-valued index:

```sql
mysql> CREATE TABLE t3 (a INT, j JSON, INDEX idx(a, (CAST(j AS SIGNED ARRAY))));
Query OK, 0 rows affected (0.04 sec)

mysql> EXPLAIN SELECT /*+ use_index_merge(t3, idx) */ * FROM t3 WHERE ((a=1 AND (1メンバー OF (j)))) OR ((a=2 AND (2メンバー OF (j))));
+---------------------------------+---------+-----------+---------------------------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------+
| id                              | estRows | task      | access object                                     | operator info                                                                                                                                    |
+---------------------------------+---------+-----------+---------------------------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------+
| Selection_5                     | 0.08    | root      |                                                   | or(and(eq(test.t3.a, 1), json_memberof(cast(1, json BINARY), test.t3.j)), and(eq(test.t3.a, 2), json_memberof(cast(2, json BINARY), test.t3.j))) |
| └─IndexMerge_9                  | 0.10    | root      |                                                   | type: union                                                                                                                                      |
|   ├─IndexRangeScan_6(Build)     | 0.10    | cop[tikv] | table:t3, index:idx(a, cast(`j` as signed array)) | range:[1 1,1 1], keep order:false, stats:pseudo                                                                                                  |
|   ├─IndexRangeScan_7(Build)     | 0.10    | cop[tikv] | table:t3, index:idx(a, cast(`j` as signed array)) | range:[2 2,2 2], keep order:false, stats:pseudo                                                                                                  |
|   └─TableRowIDScan_8(Probe)     | 0.10    | cop[tikv] | table:t3                                          | keep order:false, stats:pseudo                                                                                                                   |
+---------------------------------+---------+-----------+---------------------------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------+
```
```
      └─TableRowIDScan_7          | 10.00   | cop[tikv] | table:t                                                                    | keep order:false, stats:pseudo                                                                                                                 |
+---------------------------------+---------+-----------+----------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------------------------+
5 rows in set, 6 warnings (0.01 sec)
```

現時点では、TiDBは複数のインデックスを同時にアクセスするための以下の計画を生成するのではなく、1つのインデックスのみを使用してアクセスすることをサポートしています。

```sql
Selection
└─IndexMerge
  ├─IndexRangeScan(k1)
  ├─IndexRangeScan(k2)
  ├─IndexRangeScan(ka)
  └─Selection
    └─TableRowIDScan
```

### サポートされていないシナリオ

複数の異なるインデックスに対応する複数の式で構成される `OR` 条件の場合、多値インデックスを使用することはできません。

```sql
mysql> create table t(j1 json, j2 json, a int, INDEX k1((CAST(j1->'$.path' AS SIGNED ARRAY))), INDEX k2((CAST(j2->'$.path' AS SIGNED ARRAY))), INDEX ka(a));
Query OK, 0 rows affected (0.03 sec)

mysql> explain select /*+ use_index_merge(t, k1, k2, ka) */ * from t where (1 member of (j1->'$.path')) or (2 member of (j2->'$.path'));
+-------------------------+----------+-----------+---------------+----------------------------------------------------------------------------------------------------------------------------------------------------+
| id                      | estRows  | task      | access object | operator info                                                                                                                                      |
+-------------------------+----------+-----------+---------------+----------------------------------------------------------------------------------------------------------------------------------------------------+
| Selection_5             | 8000.00  | root      |               | or(json_memberof(cast(1, json BINARY), json_extract(test.t.j1, "$.path")), json_memberof(cast(2, json BINARY), json_extract(test.t.j2, "$.path"))) |
| └─TableReader_7         | 10000.00 | root      |               | data:TableFullScan_6                                                                                                                               |
|   └─TableFullScan_6     | 10000.00 | cop[tikv] | table:t       | keep order:false, stats:pseudo                                                                                                                     |
+-------------------------+----------+-----------+---------------+----------------------------------------------------------------------------------------------------------------------------------------------------+
3 rows in set, 3 warnings (0.00 sec)

mysql> explain select /*+ use_index_merge(t, k1, k2, ka) */ * from t where (1 member of (j1->'$.path')) or (a = 3);
+-------------------------+----------+-----------+---------------+---------------------------------------------------------------------------------------------+
| id                      | estRows  | task      | access object | operator info                                                                               |
+-------------------------+----------+-----------+---------------+---------------------------------------------------------------------------------------------+
| Selection_5             | 8000.00  | root      |               | or(json_memberof(cast(1, json BINARY), json_extract(test.t.j1, "$.path")), eq(test.t.a, 3)) |
| └─TableReader_7         | 10000.00 | root      |               | data:TableFullScan_6                                                                        |
|   └─TableFullScan_6     | 10000.00 | cop[tikv] | table:t       | keep order:false, stats:pseudo                                                              |
+-------------------------+----------+-----------+---------------+---------------------------------------------------------------------------------------------+
3 rows in set, 3 warnings (0.00 sec)
```

前述のシナリオの回避策は、`Union All` を使用してクエリを書き直すことです。

以下は、まだサポートされていないいくつかのより複雑なシナリオです。

```sql
mysql> CREATE TABLE t4 (j JSON, INDEX idx((CAST(j AS SIGNED ARRAY))));
Query OK, 0 rows affected (0.04 sec)

-- クエリに複数の json_contains 式で構成される OR 条件が含まれる場合、IndexMergeを使用してインデックスをアクセスできません。
mysql> EXPLAIN SELECT /*+ use_index_merge(t3, idx) */ * FROM t3 WHERE (json_contains(j, '[1, 2]')) OR (json_contains(j, '[3, 4]'));
+-------------------------+----------+-----------+---------------+------------------------------------------------------------------------------------------------------------------+
| id                      | estRows  | task      | access object | operator info                                                                                                    |
+-------------------------+----------+-----------+---------------+------------------------------------------------------------------------------------------------------------------+
| TableReader_7           | 9600.00  | root      |               | data:Selection_6                                                                                                 |
| └─Selection_6           | 9600.00  | cop[tikv] |               | or(json_contains(test.t3.j, cast("[1, 2]", json BINARY)), json_contains(test.t3.j, cast("[3, 4]", json BINARY))) |
|   └─TableFullScan_5     | 10000.00 | cop[tikv] | table:t3      | keep order:false, stats:pseudo                                                                                   |
+-------------------------+----------+-----------+---------------+------------------------------------------------------------------------------------------------------------------+
3 rows in set, 1 warning (0.00 sec)

mysql> SHOW WARNINGS;
+---------+------+----------------------------+
| Level   | Code | Message                    |
+---------+------+----------------------------+
| Warning | 1105 | IndexMerge is inapplicable |
+---------+------+----------------------------+
1 row in set (0.00 sec)

mysql> EXPLAIN SELECT /*+ use_index_merge(t3, idx) */ * FROM t3 WHERE (json_contains(j, '[1, 2]')) OR (json_contains(j, '[3, 4]'));
+-------------------------+----------+-----------+---------------+------------------------------------------------------------------------------------------------------------------+
| id                      | estRows  | task      | access object | operator info                                                                                                    |
+-------------------------+----------+-----------+---------------+------------------------------------------------------------------------------------------------------------------+
| TableReader_7           | 9600.00  | root      |               | data:Selection_6                                                                                                 |
| └─Selection_6           | 9600.00  | cop[tikv] |               | or(json_contains(test.t3.j, cast("[1, 2]", json BINARY)), json_contains(test.t3.j, cast("[3, 4]", json BINARY))) |
|   └─TableFullScan_5     | 10000.00 | cop[tikv] | table:t3      | keep order:false, stats:pseudo                                                                                   |
+-------------------------+----------+-----------+---------------+------------------------------------------------------------------------------------------------------------------+
3 rows in set, 1 warning (0.01 sec)

-- マルチレベルの OR / AND のネストで構成されるより複雑な式を含むクエリが含まれる場合、IndexMergeを使用してインデックスにアクセスすることはできません。
mysql> EXPLAIN SELECT /*+ use_index_merge(t3, idx) */ * FROM t3 WHERE ((1 member of (j)) AND (2 member of (j))) OR ((3 member of (j)) AND (4 member of (j)));
+-------------------------+----------+-----------+---------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| id                      | estRows  | task      | access object | operator info                                                                                                                                                                                                |
+-------------------------+----------+-----------+---------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Selection_5             | 8000.00  | root      |               | or(and(json_memberof(cast(1, json BINARY), test.t3.j), json_memberof(cast(2, json BINARY), test.t3.j)), and(json_memberof(cast(3, json BINARY), test.t3.j), json_memberof(cast(4, json BINARY), test.t3.j))) |
| └─TableReader_7         | 10000.00 | root      |               | data:TableFullScan_6                                                                                                                                                                                         |
|   └─TableFullScan_6     | 10000.00 | cop[tikv] | table:t3      | keep order:false, stats:pseudo                                                                                                                                                                               |
+-------------------------+----------+-----------+---------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
3 rows in set, 2 warnings (0.00 sec)
```

現在の実装によって制限されているマルチバリューインデックスのために、 [`use_index`](/optimizer-hints.md#use_indext1_name-idx1_name--idx2_name-) を使用すると、`Can't find a proper physical plan for this query` エラーが返される可能性がありますが、 [`use_index_merge`](/optimizer-hints.md#use_index_merget1_name-idx1_name--idx2_name-) を使用するとそのようなエラーは返されません。そのため、マルチバリューインデックスを使用する場合は、`use_index_merge` を使用することをお勧めします。

```sql
mysql> EXPLAIN SELECT /*+ use_index(t3, idx) */ * FROM t3 WHERE ((1 member of (j)) AND (2 member of (j))) OR ((3 member of (j)) AND (4 member of (j)));
ERROR 1815 (HY000): Internal : Cant find a proper physical plan for this query

mysql> EXPLAIN SELECT /*+ use_index_merge(t3, idx) */ * FROM t3 WHERE ((1 member of (j)) AND (2 member of (j))) OR ((3 member of (j)) AND (4 member of (j)));
+-------------------------+----------+-----------+---------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| id                      | estRows  | task      | access object | operator info                                                                                                                                                                                                |
+-------------------------+----------+-----------+---------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```
| Selection_5             | 8000.00  | root      |               | or(and(json_memberof(cast(1, json BINARY), test.t3.j), json_memberof(cast(2, json BINARY), test.t3.j)), and(json_memberof(cast(3, json BINARY), test.t3.j), json_memberof(cast(4, json BINARY), test.t3.j))) |
| └─TableReader_7         | 10000.00 | root      |               | data:TableFullScan_6                                                                                                                                                                                         |
|   └─TableFullScan_6     | 10000.00 | cop[tikv] | table:t3      | keep order:false, stats:pseudo                                                                                                                                                                               |
+-------------------------+----------+-----------+---------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
3 rows in set, 2 warnings (0.00 sec)