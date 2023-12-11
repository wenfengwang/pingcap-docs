---
title: インデックスマージを使用したステートメントの説明
summary: `EXPLAIN`ステートメントによってTiDBで返される実行計画情報について学びます。

# インデックスマージを使用したステートメントの説明

インデックスマージは、TiDB v4.0で導入されたテーブルアクセス方法です。この方法を使用することで、TiDBのオプティマイザはテーブルごとに複数のインデックスを使用し、それぞれのインデックスで返された結果をマージすることができます。一部のシナリオでは、この方法によってクエリがフルテーブルスキャンを回避することで効率的になります。

TiDBのインデックスマージには、2つのタイプがあります。そのうち、前者は`AND`式に対応し、後者は`OR`式に対応します。ユニオン型のインデックスマージはTiDB v4.0で実験的な機能として導入され、v5.4.0でGAになりました。一方、インターセクション型はTiDB v6.5.0で導入され、[`USE_INDEX_MERGE`](/optimizer-hints.md#use_index_merget1_name-idx1_name--idx2_name-)ヒントが指定されている場合のみ使用できます。

## インデックスマージの有効化

v5.4.0以降のTiDBバージョンでは、インデックスマージはデフォルトで有効になっています。それ以外の場合、インデックスマージが有効になっていない場合は、変数[`tidb_enable_index_merge`](/system-variables.md#tidb_enable_index_merge-new-in-v40)を`ON`に設定する必要があります。

```sql
SET session tidb_enable_index_merge = ON;
```

## 例

```sql
CREATE TABLE t(a int, b int, c int, d int, INDEX idx_a(a), INDEX idx_b(b), INDEX idx_c(c), INDEX idx_d(d));
```

```sql
EXPLAIN SELECT /*+ NO_INDEX_MERGE() */ * FROM t WHERE a = 1 OR b = 1;

+-------------------------+----------+-----------+---------------+--------------------------------------+
| id                      | estRows  | task      | access object | operator info                        |
+-------------------------+----------+-----------+---------------+--------------------------------------+
| TableReader_7           | 19.99    | root      |               | data:Selection_6                     |
| └─Selection_6           | 19.99    | cop[tikv] |               | or(eq(test.t.a, 1), eq(test.t.b, 1)) |
|   └─TableFullScan_5     | 10000.00 | cop[tikv] | table:t       | keep order:false, stats:pseudo       |
+-------------------------+----------+-----------+---------------+--------------------------------------+
EXPLAIN SELECT /*+ USE_INDEX_MERGE(t) */ * FROM t WHERE a > 1 OR b > 1;
+-------------------------------+---------+-----------+-------------------------+------------------------------------------------+
| id                            | estRows | task      | access object           | operator info                                  |
+-------------------------------+---------+-----------+-------------------------+------------------------------------------------+
| IndexMerge_8                  | 5555.56 | root      |                         | type: union                                    |
| ├─IndexRangeScan_5(Build)     | 3333.33 | cop[tikv] | table:t, index:idx_a(a) | range:(1,+inf], keep order:false, stats:pseudo |
| ├─IndexRangeScan_6(Build)     | 3333.33 | cop[tikv] | table:t, index:idx_b(b) | range:(1,+inf], keep order:false, stats:pseudo |
| └─TableRowIDScan_7(Probe)     | 5555.56 | cop[tikv] | table:t                 | keep order:false, stats:pseudo                 |
+-------------------------------+---------+-----------+-------------------------+------------------------------------------------+
```

上記のクエリでは、フィルタ条件は`WHERE`句で`OR`をコネクタとして使用しています。インデックスマージを使用しない場合、テーブルごとに1つのインデックスしか使用できません。`a = 1`はインデックス`a`にプッシュダウンすることはできず、`b = 1`もインデックス`b`にプッシュダウンすることはできません。`t`に多くのデータが存在する場合、フルテーブルスキャンは効率的ではありません。このようなシナリオに対応するために、TiDBにはインデックスマージが導入されています。

上記のクエリでは、オプティマイザはユニオン型のインデックスマージを選択してテーブルにアクセスします。インデックスマージを使用すると、オプティマイザはテーブルごとに複数のインデックスを使用し、それぞれのインデックスで返された結果をマージし、上記の出力の後半にある後者の実行計画を生成できます。

出力では、`operator info`の`type: union`情報が含まれる`IndexMerge_8`オペレータは、このオペレータがユニオン型のインデックスマージであることを示しています。`IndexRangeScan_5`および`IndexRangeScan_6`は条件に一致する`RowID`を範囲に応じてスキャンし、その後`TableRowIDScan_7`オペレータがこれらの`RowID`に基づいて条件に一致するデータを正確に読み取ります。

`IndexRangeScan`/`TableRangeScan`など、特定の範囲のデータを対象とするスキャン操作では、他のスキャン操作（`IndexFullScan`/`TableFullScan`など）と比較して、結果の`operator info`列にスキャン範囲に関する追加情報が含まれます。上記の例では、`IndexRangeScan_5`オペレータの`range:(1,+inf]`は、このオペレータがデータを1から正の無限大までスキャンすることを示しています。

```sql
EXPLAIN SELECT /*+ NO_INDEX_MERGE() */ * FROM t WHERE a > 1 AND b > 1 AND c = 1;  -- インデックスマージを使用しない

+--------------------------------+---------+-----------+-------------------------+---------------------------------------------+
| id                             | estRows | task      | access object           | operator info                               |
+--------------------------------+---------+-----------+-------------------------+---------------------------------------------+
| IndexLookUp_19                 | 1.11    | root      |                         |                                             |
| ├─IndexRangeScan_16(Build)     | 10.00   | cop[tikv] | table:t, index:idx_c(c) | range:[1,1], keep order:false, stats:pseudo |
| └─Selection_18(Probe)          | 1.11    | cop[tikv] |                         | gt(test.t.a, 1), gt(test.t.b, 1)            |
|   └─TableRowIDScan_17          | 10.00   | cop[tikv] | table:t                 | keep order:false, stats:pseudo              |
+--------------------------------+---------+-----------+-------------------------+---------------------------------------------+

EXPLAIN SELECT /*+ USE_INDEX_MERGE(t, idx_a, idx_b, idx_c) */ * FROM t WHERE a > 1 AND b > 1 AND c = 1;  -- インデックスマージを使用

+-------------------------------+---------+-----------+-------------------------+------------------------------------------------+
| id                            | estRows | task      | access object           | operator info                                  |
+-------------------------------+---------+-----------+-------------------------+------------------------------------------------+
| IndexMerge_9                  | 1.11    | root      |                         | type: intersection                             |
| ├─IndexRangeScan_5(Build)     | 3333.33 | cop[tikv] | table:t, index:idx_a(a) | range:(1,+inf], keep order:false, stats:pseudo |
| ├─IndexRangeScan_6(Build)     | 3333.33 | cop[tikv] | table:t, index:idx_b(b) | range:(1,+inf], keep order:false, stats:pseudo |
| ├─IndexRangeScan_7(Build)     | 10.00   | cop[tikv] | table:t, index:idx_c(c) | range:[1,1], keep order:false, stats:pseudo    |
| └─TableRowIDScan_8(Probe)     | 1.11    | cop[tikv] | table:t                 | keep order:false, stats:pseudo                 |
+-------------------------------+---------+-----------+-------------------------+------------------------------------------------+
```

上記の例から、フィルタ条件が`AND`を連結子として使用する`WHERE`句であることがわかります。インデックスマージが有効になる前は、オプティマイザは`idx_a`、`idx_b`、または`idx_c`のいずれか1つしか選択できませんでした。

フィルタ条件のうちいずれかが選択率が低い場合、オプティマイザは対応するインデックスを直接選択して理想的な実行効率を達成できます。ただし、データの分布が以下の3つの条件をすべて満たす場合は、インターセクション型のインデックスマージを考慮することができます。

- テーブル全体のデータサイズが大きく、直接テーブル全体を読み取るのが効率的でない。
- 3つのフィルタ条件のそれぞれの選択率が非常に高く、単一のインデックスを使用した`IndexLookUp`の実行効率が理想的でない。
- 3つのフィルタ条件の全体的な選択率が低い。

インターセクション型のインデックスマージを使用してテーブルにアクセスする場合、オプティマイザはテーブルごとに複数のインデックスを選択し、それぞれのインデックスで返された結果をマージして、上記の例の出力で後者の`IndexMerge`の実行計画を生成できます。`operator info`の`type: intersection`情報が含まれる`IndexMerge_9`オペレータは、このオペレータがインターセクション型のインデックスマージであることを示しています。実行計画のその他の部分は、前述のユニオン型のインデックスマージの例と似ています。

> **注記:**
>
> - インデックスマージ機能はv5.4.0以降でデフォルトで有効になっています。つまり、[`tidb_enable_index_merge`](/system-variables.md#tidb_enable_index_merge-new-in-v40)は`ON`です。
- 最適化機能に対して`tidb_enable_index_merge`の設定に関係なく、Index Mergeを適用させたい場合は、SQLヒント [`USE_INDEX_MERGE`](/optimizer-hints.md#use_index_merget1_name-idx1_name--idx2_name-) を使用できます。また、フィルタリング条件にプッシュダウンできない式が含まれている場合に、Index Mergeを有効にするには、SQLヒント [`USE_INDEX_MERGE`](/optimizer-hints.md#use_index_merget1_name-idx1_name--idx2_name-) を使用する必要があります。

- クエリプランに対して単一のインデックススキャンメソッド（フルテーブルスキャン以外）を選択できる場合、Optimizerは自動的にIndex Mergeを使用しません。Index Mergeを使用するためには、Optimizerヒントを使用する必要があります。

- 現時点では[一時テーブル](/temporary-tables.md)ではIndex Mergeはサポートされていません。

- インターセクションタイプのIndex MergeはOptimizerによって自動的に選択されません。選択するには、**テーブル名とインデックス名**を使用して、[`USE_INDEX_MERGE`](/optimizer-hints.md#use_index_merget1_name-idx1_name--idx2_name-) ヒントを明示する必要があります。