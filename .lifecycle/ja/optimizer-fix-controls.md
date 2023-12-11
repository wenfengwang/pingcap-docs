---
title: オプティマイザー修正コントロール
summary: オプティマイザー修正コントロール機能について、「tidb_opt_fix_control」を使用してTiDBオプティマイザーをより細かく制御する方法について学びます。

# オプティマイザー修正コントロール

この製品はイテレーションを重ねるごとにTiDBオプティマイザーの動作が変化し、それによりより合理的な実行計画が生成されます。ただし、特定のシナリオでは新しい動作が予期しない結果をもたらすことがあります。例：

- 一部の動作は特定のシナリオに依存する。ほとんどのシナリオに改善をもたらす変更が他のシナリオには逆戻りをもたらす可能性があります。
- 時々、動作の詳細の変更とその結果との関係が非常に複雑です。特定の動作の改善が全体的な逆戻りを引き起こす可能性があります。

そのため、TiDBではオプティマイザー修正コントロール機能を提供し、一連の修正の値を設定することでTiDBオプティマイザーの動作を細かく制御できるようにしています。このドキュメントではオプティマイザー修正コントロール機能とその使用方法、TiDBが現在サポートしているすべての修正を説明しています。

## `tidb_opt_fix_control`の概要

v7.1.0から、TiDBは[`tidb_opt_fix_control`](/system-variables.md#tidb_opt_fix_control-new-in-v710)システム変数を提供して、オプティマイザーの動作をより詳細に制御します。

各修正は、TiDBオプティマイザーの特定の目的のための動作を調整するために使用される制御項目です。GitHub Issueに対応する番号で示されます。たとえば、修正`44262`の場合、[Issue 44262](https://github.com/pingcap/tidb/issues/44262)で何を制御しているかを確認できます。

[`tidb_opt_fix_control`](/system-variables.md#tidb_opt_fix_control-new-in-v710)システム変数は、複数の修正を1つの値として受け入れます。修正はコンマ（`,`）で区切られた形式`"<#issue1>:<value1>,<#issue2>:<value2>,...,<#issueN>:<valueN>"`であり、`<#issueN>`は修正番号です。例：

```sql
SET SESSION tidb_opt_fix_control = '44262:ON,44389:ON';
```

## オプティマイザー修正コントロール参照

### [`44262`](https://github.com/pingcap/tidb/issues/44262) <span class="version-mark">v7.2.0で新規追加</span>

- デフォルト値: `OFF`
- 可能な値: `ON`, `OFF`
- この変数は、[グローバル統計情報](/statistics.md#collect-statistics-of-partitioned-tables-in-dynamic-pruning-mode)が欠落している場合に、パーティションテーブルにアクセスするための[ダイナミックプルーニングモード](/partitioned-table.md#dynamic-pruning-mode)の使用を許可するかどうかを制御します。

### [`44389`](https://github.com/pingcap/tidb/issues/44389) <span class="version-mark">v7.2.0で新規追加</span>

- デフォルト値: `OFF`
- 可能な値: `ON`, `OFF`
- `c = 10 and (a = 'xx' or (a = 'kk' and b = 1))`などのフィルタに対して、この変数は`IndexRangeScan`に対してより包括的なスキャン範囲を構築しようとするかどうかを制御します。

### [`44823`](https://github.com/pingcap/tidb/issues/44823) <span class="version-mark">v7.3.0で新規追加</span>

- デフォルト値: `200`
- 可能な値: `[0, 2147483647]`
- メモリを節約するため、プランキャッシュはこの変数の指定数を超えるパラメータを持つクエリをキャッシュしません。`0`は無制限を意味します。

### [`44830`](https://github.com/pingcap/tidb/issues/44830) <span class="version-mark">v7.3.0で新規追加</span>

- デフォルト値: `OFF`
- 可能な値: `ON`, `OFF`
- この変数は、Plan Cacheが物理的な最適化中に生成された`PointGet`演算子を含む実行計画をキャッシュすることを許可するかどうかを制御します。

### [`44855`](https://github.com/pingcap/tidb/issues/44855) <span class="version-mark">v7.3.0で新規追加</span>

- デフォルト値: `OFF`
- 可能な値: `ON`, `OFF`
- 特定のシナリオでは、`IndexJoin`演算子の`Probe`側が`Selection`演算子を含んでいる場合、TiDBは`IndexScan`の行数を非常に過大評価します。これにより、`IndexJoin`の代わりにサブ最適なクエリプランが選択される可能性があります。
- この問題を緩和するために、TiDBは改善策を導入しましたが、潜在的なクエリプランのフォールバックリスクのため、この改善はデフォルトで無効になっています。
- この変数は、前述の改善を有効にするかどうかを制御します。

### [`45132`](https://github.com/pingcap/tidb/issues/45132) <span class="version-mark">v7.4.0で新規追加</span>

- デフォルト値: `1000`
- 可能な値: `[0, 2147483647]`
- この変数は、オプティマイザーのヒューリスティック戦略がアクセスパスを選択する基準を設定します。アクセスパス（たとえば`Index_A`など）の推定行数が他のアクセスパス（デフォルトで`1000`倍）よりもはるかに小さい場合、オプティマイザーはコスト比較をスキップして直接`Index_A`を選択します。
- `0`はこのヒューリスティック戦略を無効にします。