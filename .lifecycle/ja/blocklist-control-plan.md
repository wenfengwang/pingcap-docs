---
title: 最適化ルールおよび式プッシュダウンのブロックリスト
summary: 最適化ルールと式プッシュダウンの挙動を制御するためのブロックリストについて学びます。
---

# 最適化ルールおよび式プッシュダウンのブロックリスト

このドキュメントでは、TiDBの挙動を制御するために最適化ルールのブロックリストおよび式プッシュダウンのブロックリストを使用する方法について紹介します。

## 最適化ルールのブロックリスト

最適化ルールのブロックリストは、最適化ルールを調整する方法の一つで、主に手動でいくつかの最適化ルールを無効にするために使用されます。

### 重要な最適化ルール

|**最適化ルール**|**ルール名**|**説明**|
| :--- | :--- | :--- |
| カラムプルーニング | column_prune | 上位エグゼキューターによって必要とされていない場合、1つの演算子はカラムをプルーニングします。 |
| 相関副問い合わせの除去 | decorrelate | 相関副問い合わせを非相関結合または集約に書き換えようとします。 |
| 集約の除去 | aggregation_eliminate | 実行計画から不要な集約演算子を削除しようとします。 |
| プロジェクションの除去 | projection_eliminate | 実行計画から不要なプロジェクション演算子を削除します。 |
| Max/Minの除去 | max_min_eliminate | 集約の一部のmax/min関数を、`order by` + `limit 1`形式に書き直します。 |
| プレディケートのプッシュダウン | predicate_push_down | プレディケートをデータソースに近い演算子までプッシュしようとします。 |
| 外部結合の除去 | outer_join_eliminate | 実行計画から不要な左結合または右結合を削除しようとします。 |
| パーティションプルーニング | partition_processor | 述語によって拒絶されたパーティションを剪定し、パーティション化されたテーブルのクエリを`UnionAll + Partition Datasource`形式に書き直します。 |
| 集約のプッシュダウン | aggregation_push_down | 集約をその子にプッシュしようとします。 |
| TopNのプッシュダウン | topn_push_down | TopN演算子をデータソースに近い場所にプッシュしようとします。 |
| 結合の並べ替え | join_reorder | マルチテーブル結合の順序を決定します。 |
| ウィンドウ関数からTopNまたはLimitの派生 | derive_topn_from_window | ウィンドウ関数からTopN演算子またはLimit演算子を派生します。 |

### 最適化ルールの無効化

特定のクエリにおいて、いくつかのルールがサブ最適な実行計画を導く場合は、最適化ルールのブロックリストを使用していくつかのルールを無効にできます。

#### 使い方

> **注意：**
>
> 以下の操作をすべて実行するには、データベースの`スーパー権限`が必要です。各最適化ルールには名前があります。例えば、カラムプルーニングの名前は`column_prune`です。すべての最適化ルールの名前は、[重要な最適化ルール](#important-optimization-rules)テーブルの2番目の列にあります。

- 一部のルールを無効にしたい場合は、その名前を`mysql.opt_rule_blacklist`テーブルに記述します。例えば：

    {{< copyable "sql" >}}

    ```sql
    INSERT INTO mysql.opt_rule_blacklist VALUES("join_reorder"), ("topn_push_down");
    ```

    上記のSQL文を実行すると、上述の操作がすぐに効果を発揮します。有効範囲には、対応するTiDBサーバーのすべての旧い接続が含まれます。

    {{< copyable "sql" >}}

    ```sql
    admin reload opt_rule_blacklist;
    ```

    > **注意：**
    >
    > `admin reload opt_rule_blacklist`は、上記のステートメントが実行されたTiDBサーバーでのみ効果があります。クラスターのすべてのTiDBサーバーに効果をもたせたい場合は、各TiDBサーバーでこのコマンドを実行してください。

- ルールを再度有効にする場合は、テーブル内の対応するデータを削除し、その後`admin reload`ステートメントを実行します：

    {{< copyable "sql" >}}

    ```sql
    DELETE FROM mysql.opt_rule_blacklist WHERE name IN ("join_reorder", "topn_push_down");
    ```

    {{< copyable "sql" >}}

    ```sql
    admin reload opt_rule_blacklist;
    ```

## 式プッシュダウンのブロックリスト

式プッシュダウンのブロックリストは、式プッシュダウンを調整する方法の一つで、主に手動で特定のデータ型のいくつかの式を無効にするために使用されます。

### プッシュダウンをサポートする式

プッシュダウンをサポートする式に関する詳細は、[TiKVへのプッシュダウンにサポートされる式](/functions-and-operators/expressions-pushed-down.md#supported-expressions-for-pushdown-to-tikv)を参照してください。

### 特定の式のプッシュダウンの無効化

式プッシュダウンにより誤った結果が得られた場合は、ブロックリストを使用してアプリケーションの素早い復旧を行うことができます。具体的には、`mysql.expr_pushdown_blacklist`テーブルにいくつかのサポートされている関数や演算子を追加して、特定の式のプッシュダウンを無効にできます。

`mysql.expr_pushdown_blacklist`のスキーマは次のように示されます：

{{< copyable "sql" >}}

```sql
DESC mysql.expr_pushdown_blacklist;
```

```sql
+------------+--------------+------+------+-------------------+-------+
| Field      | Type         | Null | Key  | Default           | Extra |
+------------+--------------+------+------+-------------------+-------+
| name       | char(100)    | NO   |      | NULL              |       |
| store_type | char(100)    | NO   |      | tikv,tiflash,tidb |       |
| reason     | varchar(200) | YES  |      | NULL              |       |
+------------+--------------+------+------+-------------------+-------+
3 rows in set (0.00 sec)
```

上記の各フィールドの説明は次の通りです：

+ `name`: プッシュダウンを無効にする関数の名前。
+ `store_type`: 求められたコンポーネントによってプッシュダウンを行わないようにする関数を指定します。利用可能なコンポーネントは、`tidb`、`tikv`、`tiflash`です。`store_type`は大文字小文字を区別しません。複数のコンポーネントを指定する必要がある場合は、各コンポーネントをカンマで区切ってください。
    - `store_type`が`tidb`の場合、その関数がTiDBメモリテーブルの読み取り中に他のTiDBサーバーで実行されるかどうかを示します。
    - `store_type`が`tikv`の場合、その関数がTiKVサーバーのCoprocessorコンポーネントで実行されるかどうかを示します。
    - `store_type`が`tiflash`の場合、その関数がTiFlash ServerのCoprocessorコンポーネントで実行されるかどうかを示します。
+ `reason`: この関数がブロックリストに追加された理由を記録します。

### 使い方

このセクションでは、式プッシュダウンのブロックリストの使用方法について説明します。

#### ブロックリストへの追加

ブロックリストに1つ以上の式（関数または演算子）を追加するには、次の手順を実行します：

1. 対応する関数名または演算子名と、プッシュダウンを無効にしたいコンポーネントのセットを`mysql.expr_pushdown_blacklist`テーブルに挿入します。

2. `admin reload expr_pushdown_blacklist`を実行します。

### ブロックリストから削除

1つまたは複数の式をブロックリストから削除するには、次の手順を実行します：

1. `mysql.expr_pushdown_blacklist`テーブルから、対忙する関数名または演算子名と、プッシュダウンを無効にしたいコンポーネントのセットを削除します。

2. `admin reload expr_pushdown_blacklist`を実行します。

> **注意：**
>
> `admin reload expr_pushdown_blacklist`は、このステートメントが実行されたTiDBサーバーでのみ効果があります。クラスターのすべてのTiDBサーバーに効果をもたせたい場合は、各TiDBサーバーでこのコマンドを実行してください。

## 式ブロックリストの使用例

以下の例では、`<` および `>` 演算子がブロックリストに追加され、その後`>`演算子がブロックリストから削除されます。

ブロックリストが有効になっているかどうかを判断するには、`EXPLAIN`の結果を観察してください（詳細は[TiDBクエリ実行計画の概要](/explain-overview.md)を参照）。

1. 次のSQL文の`WHERE`句における述語`a < 2`および`a > 2`は、TiKVにプッシュダウンすることができます。

    {{< copyable "sql" >}}

    ```sql
    EXPLAIN SELECT * FROM t WHERE a < 2 AND a > 2;
    ```

    ```sql
    +-------------------------+----------+-----------+---------------+------------------------------------+
    | id                      | estRows  | task      | access object | operator info                      |
    +-------------------------+----------+-----------+---------------+------------------------------------+
    | TableReader_7           | 0.00     | root      |               | data:Selection_6                   |
    | └─Selection_6           | 0.00     | cop[tikv] |               | gt(ssb_1.t.a, 2), lt(ssb_1.t.a, 2) |
    |   └─TableFullScan_5     | 10000.00 | cop[tikv] | table:t       | keep order:false, stats:pseudo     |
    +-------------------------+----------+-----------+---------------+------------------------------------+
    3 rows in set (0.00 sec)
    ```

2. `mysql.expr_pushdown_blacklist`テーブルに式を挿入し、`admin_reload expr_pushdown_blacklist`を実行します。

    {{< copyable "sql" >}}

    ```sql
    INSERT INTO mysql.expr_pushdown_blacklist VALUES('<','tikv',''), ('>','tikv','');
    ```

    ```sql```
```
    クエリは正常に完了しました。2行が影響を受けました（0.01秒）
    レコード数：2  重複：0  警告：0
    ```

    {{< copyable "sql" >}}

    ```sql
    admin reload expr_pushdown_blacklist;
    ```

    ```sql
    クエリは正常に完了しました。0行が影響を受けました（0.00秒）
    ```

3. 実行プランを再度確認すると、`<` および `>` 演算子が TiKV Coprocessor にプッシュダウンされていないことがわかります。

    {{< copyable "sql" >}}

    ```sql
    EXPLAIN SELECT * FROM t WHERE a < 2 and a > 2;
    ```

    ```sql
    +-------------------------+----------+-----------+---------------+------------------------------------+
    | id                      | estRows  | task      | access object | operator info                      |
    +-------------------------+----------+-----------+---------------+------------------------------------+
    | Selection_7             | 10000.00 | root      |               | gt(ssb_1.t.a, 2), lt(ssb_1.t.a, 2) |
    | └─TableReader_6         | 10000.00 | root      |               | data:TableFullScan_5               |
    |   └─TableFullScan_5     | 10000.00 | cop[tikv] | table:t       | keep order:false, stats:pseudo     |
    +-------------------------+----------+-----------+---------------+------------------------------------+
    3 行が返されました（0.00秒）
    ```

4. ブロックリストから一つの式（ここでは `>` ）を削除し、`admin reload expr_pushdown_blacklist` を実行します。

    {{< copyable "sql" >}}

    ```sql
    DELETE FROM mysql.expr_pushdown_blacklist WHERE name = '>';
    ```

    ```sql
    クエリは正常に完了しました。1行が影響を受けました（0.01秒）
    ```

    {{< copyable "sql" >}}

    ```sql
    admin reload expr_pushdown_blacklist;
    ```

    ```sql
    クエリは正常に完了しました。0行が影響を受けました（0.00秒）
    ```

5. 実行プランを再度確認すると、`<` はプッシュダウンされず、`>` が TiKV Coprocessor にプッシュダウンされていることがわかります。

    {{< copyable "sql" >}}

    ```sql
    EXPLAIN SELECT * FROM t WHERE a < 2 AND a > 2;
    ```

    ```sql
    +---------------------------+----------+-----------+---------------+--------------------------------+
    | id                        | estRows  | task      | access object | operator info                  |
    +---------------------------+----------+-----------+---------------+--------------------------------+
    | Selection_8               | 0.00     | root      |               | lt(ssb_1.t.a, 2)               |
    | └─TableReader_7           | 0.00     | root      |               | data:Selection_6               |
    |   └─Selection_6           | 0.00     | cop[tikv] |               | gt(ssb_1.t.a, 2)               |
    |     └─TableFullScan_5     | 10000.00 | cop[tikv] | table:t       | keep order:false, stats:pseudo |
    +---------------------------+----------+-----------+---------------+--------------------------------+
    4 行が返されました（0.00秒）
    ```