---
title: EXPLAIN | TiDB SQLステートメントリファレンス
summary: TiDBデータベースのEXPLAINの使用方法の概要。
aliases: ['/docs/dev/sql-statements/sql-statement-explain/','/docs/dev/reference/sql/statements/explain/']
---

# `EXPLAIN`

`EXPLAIN`ステートメントは、クエリの実行計画を実行せずに表示します。`EXPLAIN ANALYZE`を使用するとクエリが実行されます。`EXPLAIN`の出力が期待される結果と一致しない場合は、クエリ内の各テーブルで`ANALYZE TABLE`を実行することを検討してください。

ステートメント`DESC`および`DESCRIBE`は、このステートメントのエイリアスです。`EXPLAIN <tableName>`の代替の使用方法は[`SHOW [FULL] COLUMNS FROM`](/sql-statements/sql-statement-show-columns-from.md)で文書化されています。

TiDBは`EXPLAIN [options] FOR CONNECTION connection_id`ステートメントをサポートしています。ただし、このステートメントはMySQLの`EXPLAIN FOR`ステートメントとは異なります。詳細は[`EXPLAIN FOR CONNECTION`](#explain-for-connection)を参照してください。

## 概要

```ebnf+diagram
ExplainSym ::=
    'EXPLAIN'
|   'DESCRIBE'
|   'DESC'

ExplainStmt ::=
    ExplainSym ( TableName ColumnName? | 'ANALYZE'? ExplainableStmt | 'FOR' 'CONNECTION' NUM | 'FORMAT' '=' ( stringLit | ExplainFormatType ) ( 'FOR' 'CONNECTION' NUM | ExplainableStmt ) )

ExplainableStmt ::=
    SelectStmt
|   DeleteFromStmt
|   UpdateStmt
|   InsertIntoStmt
|   ReplaceIntoStmt
|   UnionStmt
```

## `EXPLAIN`の出力形式

> **Note:**
>
> TiDBにMySQLクライアントを使用して接続する場合、改行を含まないように結果を読むために、`pager less -S`コマンドを使用できます。その後、`EXPLAIN`結果が出力されたら、キーボードの右矢印<kbd>→</kbd>ボタンを押して出力を水平にスクロールできます。

> **Note:**
>
> 返された実行計画では、`IndexJoin`および`Apply`演算子の全てのプローブ側子ノードについて、v6.4.0以降の`estRows`の意味はv6.4.0より前と異なります。詳細は[TiDBクエリ実行計画の概要](/explain-overview.md#understand-explain-output)を参照してください。

現在、TiDBの`EXPLAIN`は5つの列を出力します：`id`、`estRows`、`task`、`access object`、`operator info`。実行計画内の各演算子はこれらの属性で説明され、`EXPLAIN`出力内の各行が演算子を説明します。各属性の説明は次の通りです。

| 属性名             | 説明 |
|:----------------|:----------------------------------------------------------------------------------------------------------|
| id            | 演算子IDは、全体の実行計画内の演算子の一意な識別子です。TiDB 2.1では、IDがフォーマットされ、演算子のツリー構造を表示します。データは子ノードから親ノードに流れます。各演算子には1つの親ノードのみがあります。 |
| estRows       | 演算子が予想される出力行数。この数値は統計情報と演算子のロジックに基づいて推定されます。`estRows`は、TiDB 4.0以前では`count`と呼ばれています。 |
| task          | 演算子が属するタスクの種類。現在、実行計画は2つのタスクに分割されています：**root**タスクはtidb-serverで実行され、**cop**タスクはTiKVまたはTiFlashで並列に実行されます。タスクレベルの実行計画のトポロジーは、ルートタスクに続いて多くのcopタスクがあります。ルートタスクはcopタスクの出力を入力として使用します。copタスクは、TiDBがTiKVまたはTiFlashにプッシュダウンするタスクを参照します。各copタスクはTiKVクラスタまたはTiFlashクラスタに分散され、複数のプロセスで実行されます。 |
| access object | 演算子によってアクセスされるデータ項目の情報。情報には`table`、`partition`、`index`（存在する場合）が含まれます。データを直接アクセスする演算子のみがこのような情報を持ちます。 |
| operator info | 演算子に関するその他の情報。各演算子の`operator info`は異なります。以下の例を参照してください。 |

## 例

{{< copyable "sql" >}}

```sql
EXPLAIN SELECT 1;
```

```sql
+-------------------+---------+------+---------------+---------------+
| id                | estRows | task | access object | operator info |
+-------------------+---------+------+---------------+---------------+
| Projection_3      | 1.00    | root |               | 1->Column#1   |
| └─TableDual_4     | 1.00    | root |               | rows:1        |
+-------------------+---------+------+---------------+---------------+
2 行が返されました (0.00 秒)
```

{{< copyable "sql" >}}

```sql
CREATE TABLE t1 (id INT NOT NULL PRIMARY KEY AUTO_INCREMENT, c1 INT NOT NULL);
```

```sql
クエリは正常に終了しました（0.10 秒で0件のデータが影響を受けました）。
```

{{< copyable "sql" >}}

```sql
INSERT INTO t1 (c1) VALUES (1), (2), (3);
```

```sql
クエリは正常に終了しました（0.02 秒で3件のデータが影響を受けました）。レコード数: 3、重複: 0、警告: 0
```

{{< copyable "sql" >}}

```sql
EXPLAIN SELECT * FROM t1 WHERE id = 1;
```

```sql
+-------------+---------+------+---------------+---------------+
| id          | estRows | task | access object | operator info |
+-------------+---------+------+---------------+---------------+
| Point_Get_1 | 1.00    | root | table:t1      | handle:1      |
+-------------+---------+------+---------------+---------------+
1 行が返されました (0.00 秒)
```

{{< copyable "sql" >}}

```sql
DESC SELECT * FROM t1 WHERE id = 1;
```

```sql
+-------------+---------+------+---------------+---------------+
| id          | estRows | task | access object | operator info |
+-------------+---------+------+---------------+---------------+
| Point_Get_1 | 1.00    | root | table:t1      | handle:1      |
+-------------+---------+------+---------------+---------------+
1 行が返されました (0.00 秒)
```

{{< copyable "sql" >}}

```sql
DESCRIBE SELECT * FROM t1 WHERE id = 1;
```

```sql
+-------------+---------+------+---------------+---------------+
| id          | estRows | task | access object | operator info |
+-------------+---------+------+---------------+---------------+
| Point_Get_1 | 1.00    | root | table:t1      | handle:1      |
+-------------+---------+------+---------------+---------------+
1 行が返されました (0.00 秒)
```

{{< copyable "sql" >}}

```sql
EXPLAIN INSERT INTO t1 (c1) VALUES (4);
```

```sql
+----------+---------+------+---------------+---------------+
| id       | estRows | task | access object | operator info |
+----------+---------+------+---------------+---------------+
| Insert_1 | N/A     | root |               | N/A           |
+----------+---------+------+---------------+---------------+
1 行が返されました (0.00 秒)
```

{{< copyable "sql" >}}

```sql
EXPLAIN UPDATE t1 SET c1=5 WHERE c1=3;
```

```sql
+---------------------------+---------+-----------+---------------+--------------------------------+
| id                        | estRows | task      | access object | operator info                  |
+---------------------------+---------+-----------+---------------+--------------------------------+
| Update_4                  | N/A     | root      |               | N/A                            |
| └─TableReader_8           | 0.00    | root      |               | data:Selection_7               |
|   └─Selection_7           | 0.00    | cop[tikv] |               | eq(test.t1.c1, 3)              |
|     └─TableFullScan_6     | 3.00    | cop[tikv] | table:t1      | keep order:false, stats:pseudo |
+---------------------------+---------+-----------+---------------+--------------------------------+
4 行が返されました (0.00 秒)
```

{{< copyable "sql" >}}

```sql
EXPLAIN DELETE FROM t1 WHERE c1=3;
```

```sql
+---------------------------+---------+-----------+---------------+--------------------------------+
| id                        | estRows | task      | access object | operator info                  |
+---------------------------+---------+-----------+---------------+--------------------------------+
| Delete_4                  | N/A     | root      |               | N/A                            |
| └─TableReader_8           | 0.00    | root      |               | data:Selection_7               |
|   └─Selection_7           | 0.00    | cop[tikv] |               | eq(test.t1.c1, 3)              |
```
|     └─TableFullScan_6     | 3.00    | cop[tikv] | table:t1      | keep order:false, stats:pseudo |
+---------------------------+---------+-----------+---------------+--------------------------------+
4 行セット (0.01 秒)

`EXPLAIN` の出力形式を指定するには、`FORMAT = xxx` 構文を使用できます。現在、TiDB は次の形式をサポートしています。

| FORMAT | 説明 |
| ------ | ------ |
| 指定なし  | 形式が指定されていない場合、`EXPLAIN` はデフォルトの形式、`row` を使用します。 |
| `row`  | `EXPLAIN` 文の結果を表形式で出力します。詳細については、[クエリ実行プランの理解](/explain-overview.md) を参照してください。 |
| `brief`  | `EXPLAIN` 文の出力で、演算子の ID が `FORMAT` を指定しない場合より簡略化されます。 |
| `dot`    | `EXPLAIN` 文は DOT 実行プランを出力し、このプランは `dot` プログラム（`graphviz` パッケージに含まれる）を使用して PNG ファイルに変換できます。 |
| `tidb_json` | `EXPLAIN` 文は JSON 形式で実行プランを出力し、演算子情報を JSON 配列に保存します。 |

<SimpleTab>

<div label="brief">

`FORMAT` が `"brief"` の場合の `EXPLAIN` の例は次の通りです:

{{< copyable "sql" >}}

```sql
EXPLAIN FORMAT = "brief" DELETE FROM t1 WHERE c1 = 3;
```

```sql
+-------------------------+---------+-----------+---------------+--------------------------------+
| id                      | estRows | task      | access object | operator info                  |
+-------------------------+---------+-----------+---------------+--------------------------------+
| Delete                  | N/A     | root      |               | N/A                            |
| └─TableReader           | 0.00    | root      |               | data:Selection                 |
|   └─Selection           | 0.00    | cop[tikv] |               | eq(test.t1.c1, 3)              |
|     └─TableFullScan     | 3.00    | cop[tikv] | table:t1      | keep order:false, stats:pseudo |
+-------------------------+---------+-----------+---------------+--------------------------------+
4 行セット (0.001 秒)
```

</div>

<div label="DotGraph">

MySQL 標準の結果形式に加えて、TiDB は DotGraph もサポートしており、次の例のように `FORMAT = "dot"` を指定する必要があります:

{{< copyable "sql" >}}

```sql
CREATE TABLE t(a bigint, b bigint);
EXPLAIN format = "dot" SELECT A.a, B.b FROM t A JOIN t B ON A.a > B.b WHERE A.a < 10;
```

```sql
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| dot contents                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|
digraph Projection_8 {
subgraph cluster8{
node [style=filled, color=lightgrey]
color=black
label = "root"
"Projection_8" -> "HashJoin_9"
"HashJoin_9" -> "TableReader_13"
"HashJoin_9" -> "Selection_14"
"Selection_14" -> "TableReader_17"
}
subgraph cluster12{
node [style=filled, color=lightgrey]
color=black
label = "cop"
"Selection_12" -> "TableFullScan_11"
}
subgraph cluster16{
node [style=filled, color=lightgrey]
color=black
label = "cop"
"Selection_16" -> "TableFullScan_15"
}
"TableReader_13" -> "Selection_12"
"TableReader_17" -> "Selection_16"
}
 |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 行セット (0.00 秒)
```

`dot` プログラムを持っている場合、次の方法を使用して PNG ファイルを生成できます:

```bash
dot xx.dot -T png -O

ここで、xx.dot は上記の文で返された結果です。
```

`dot` プログラムがない場合は、結果を [このウェブサイト](http://www.webgraphviz.com/) にコピーしてツリーダイアグラムを取得できます:

![Explain Dot](/media/explain_dot.png)

</div>

<div label="JSON">

JSON 形式で出力するには、`EXPLAIN` 文で `FORMAT = "tidb_json"` を指定する必要があります。次の例はその形式です:

```sql
CREATE TABLE t(id int primary key, a int, b int, key(a));
EXPLAIN FORMAT = "tidb_json" SELECT id FROM t WHERE a = 1;
```

```
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| TiDB_JSON                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| [
    {
        "id": "Projection_4",
        "estRows": "10.00",
        "taskType": "root",
        "operatorInfo": "test.t.id",
        "subOperators": [
            {
                "id": "IndexReader_6",
                "estRows": "10.00",
                "taskType": "root",
                "operatorInfo": "index:IndexRangeScan_5",
                "subOperators": [
                    {
                        "id": "IndexRangeScan_5",
                        "estRows": "10.00",
                        "taskType": "cop[tikv]",
                        "accessObject": "table:t, index:a(a)",
                        "operatorInfo": "range:[1,1], keep order:false, stats:pseudo"
                    }
                ]
            }
        ]
    }
]
 |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 行セット (0.01 秒)
```

出力では、`id`、`estRows`、`taskType`、`accessObject`、`operatorInfo` は、デフォルト形式の列と同じ意味を持ちます。`subOperators` は、サブノードを保存する配列です。サブノードのフィールドと意味は親ノードと同じです。フィールドが欠落している場合は、そのフィールドが空であることを意味します。

</div>

</SimpleTab>

## MySQL の互換性

* `EXPLAIN` の形式と潜在的な実行プランは、TiDB と MySQL で大幅に異なります。
* TiDB は `FORMAT=JSON` や `FORMAT=TREE` オプションをサポートしていません。
* TiDB における `FORMAT=tidb_json` は、デフォルトの `EXPLAIN` 結果の JSON 形式出力です。その形式とフィールドは、MySQL における `FORMAT=JSON` 出力と異なります。

### `EXPLAIN FOR CONNECTION`

`EXPLAIN FOR CONNECTION` は、現在実行中の SQL クエリまたは接続で最後に実行された SQL クエリの実行プランを取得するために使用されます。出力形式は `EXPLAIN` と同じです。ただし、TiDB における `EXPLAIN FOR CONNECTION` の実装は、MySQL と異なります。出力形式以外の違いは次のとおりです:

- 接続が休止状態の場合、MySQL では空の結果が返されますが、TiDB では前回の実行クエリプランが返されます。
- 現在のセッションの実行計画を取得しようとすると、MySQL はエラーを返しますが、TiDB は通常の結果を返します。
- MySQL では、ログインユーザがクエリを実行している接続と同じであるか、ログインユーザが **`PROCESS`** 権限を持っている必要があります。一方、TiDB では、ログインユーザがクエリを実行している接続と同じであるか、ログインユーザが **`SUPER`** 権限を持っている必要があります。

## 関連情報

* [クエリ実行プランの理解](/explain-overview.md)
* [EXPLAIN ANALYZE](/sql-statements/sql-statement-explain-analyze.md)
* [ANALYZE TABLE](/sql-statements/sql-statement-analyze-table.md)
* [TRACE](/sql-statements/sql-statement-trace.md)
```