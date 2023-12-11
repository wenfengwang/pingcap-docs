---
title: GROUP BY修飾子
summary: TiDBのGROUP BY修飾子の使用方法を学びます。

# GROUP BY修飾子

v7.4.0から、TiDBの`GROUP BY`句は`WITH ROLLUP`修飾子をサポートしています。

`GROUP BY`句では、1つ以上の列をグループリストとして指定し、リストの後に`WITH ROLLUP`修飾子を追加することができます。すると、TiDBはグループリストの列に基づいて多次元降順グルーピングを実行し、出力の各グループに対するサマリー結果を提供します。

- グルーピングの方式:

    - 最初のグルーピング次元には、グループリストのすべての列が含まれます。
    - 次のグルーピング次元は、グルーピングリストの右端から開始し、一度に1つ以上の列を除外して新しいグループを形成します。

- 集計サマリー: 各次元に対して、クエリは集計操作を実行し、そしてこの次元の結果を以前のすべての次元の結果と集約します。つまり、詳細から全体まで異なる次元で集約データを取得できます。

このグルーピング方式では、グループリストに`N`個の列がある場合、TiDBは`N+1`グループでクエリ結果を集計します。

たとえば：

```sql
SELECT count(1) FROM t GROUP BY a,b,c WITH ROLLUP;
```

この例では、TiDBは`count(1)`の計算結果を4つのグループ（つまり、`{a, b, c}`、`{a, b}`、`{a}`、`{}`）で集計し、各グループのサマリー結果を出力します。

> **注記:**
>
> 現在、TiDBはCubeシンタックスをサポートしていません。

## 使用例

複数の列からデータを集計およびサマリーすることは、OLAP（オンライン解析処理）シナリオで一般的に使用されます。`WITH ROLLUP`修飾子を使用することで、集計結果から他の上位次元での超サマリー情報を表示する追加の行を取得できます。その後、超サマリー情報を使用して高度なデータ解析およびレポート生成を行うことができます。

## 前提条件

現在、TiDBは`WITH ROLLUP`構文をサポートする実行計画を生成するのはTiFlash MPPモードのみです。したがって、TiDBクラスタがTiFlashノードで展開され、対象のファクトテーブルが適切にTiFlashレプリカで構成されていることを確認してください。

<CustomContent platform="tidb">

詳細については、[TiFlashクラスタをスケールアウト](/scale-tidb-using-tiup.md#scale-out-a-tiflash-cluster)を参照してください。

</CustomContent>

## 例

`bank`という`year`、`month`、`day`、`profit`列を持つ利益テーブルがあるとします。

```sql
CREATE TABLE bank
(
    year    INT,
    month   VARCHAR(32),
    day     INT,
    profit  DECIMAL(13, 7)
);

ALTER TABLE bank SET TIFLASH REPLICA 1; -- テーブルにTiFlashレプリカを追加

INSERT INTO bank VALUES(2000, "Jan", 1, 10.3),(2001, "Feb", 2, 22.4),(2000,"Mar", 3, 31.6)
```

銀行の年間利益を取得する場合、次のように単純な`GROUP BY`句を使用できます：

```sql
SELECT year, SUM(profit) AS profit FROM bank GROUP BY year;
+------+--------------------+
| year | profit             |
+------+--------------------+
| 2001 | 22.399999618530273 |
| 2000 |  41.90000057220459 |
+------+--------------------+
2 rows in set (0.15 sec)
```

年間利益に加えて、銀行のレポートには通常、すべての年の総利益や月ごとの詳細な利益分析が含まれる必要があります。v7.4.0以前では、複数のクエリで異なる`GROUP BY`句を使用し、UNIONで結果を結合して集約されたサマリーを取得する必要がありました。v7.4.0からは、`WITH ROLLUP`修飾子を`GROUP BY`句に追加することで、1つのクエリで簡単に目的の結果を取得できます。

```sql
SELECT year, month, SUM(profit) AS profit from bank GROUP BY year, month WITH ROLLUP ORDER BY year desc, month desc;
+------+-------+--------------------+
| year | month | profit             |
+------+-------+--------------------+
| 2001 | Feb   | 22.399999618530273 |
| 2001 | NULL  | 22.399999618530273 |
| 2000 | Mar   | 31.600000381469727 |
| 2000 | Jan   | 10.300000190734863 |
| 2000 | NULL  |  41.90000057220459 |
| NULL | NULL  |  64.30000019073486 |
+------+-------+--------------------+
6 rows in set (0.025 sec)
```

上記の結果には、異なる次元での集計データが含まれています：年と月別、年ごと、および全体。結果において、`month`列に`NULL`値がない行は、その行の`profit`が年と月の両方のグループを集約して計算されていることを示します。`month`列が`NULL`値の行は、その行の`profit`が年間のすべての月を集約して計算されていることを示し、`year`列が`NULL`値の行は、その行の`profit`がすべての年を集約して計算されていることを示します。

具体的には：

- 最初の行の`profit`値は2次元グループ`{year, month}`から来ており、詳細な`{2000, "Jan"}`グループの集約結果を表しています。
- 2番目の行の`profit`値は1次元グループ`{year}`から来ており、中程度の`{2001}`グループの集約結果を表しています。
- 最後の行の`profit`値は0次元のグルーピング`{}`から来ており、全体の集約結果を表しています。

`WITH ROLLUP`の結果における`NULL`値は、集約演算子が適用される直前に生成されます。したがって、`SELECT`、`HAVING`、`ORDER BY`句で`NULL`値を使用して集約された結果をさらにフィルタリングできます。

たとえば、`HAVING`句で`NULL`を使用して2次元グループの集計結果のみをフィルタリングして表示することができます：

```sql
SELECT year, month, SUM(profit) AS profit FROM bank GROUP BY year, month WITH ROLLUP HAVING year IS NOT null AND month IS NOT null;
+------+-------+--------------------+
| year | month | profit             |
+------+-------+--------------------+
| 2000 | Mar   | 31.600000381469727 |
| 2000 | Jan   | 10.300000190734863 |
| 2001 | Feb   | 22.399999618530273 |
+------+-------+--------------------+
3 rows in set (0.02 sec)
```

`GROUP BY`リストの列にネイティブの`NULL`値が含まれている場合、`WITH ROLLUP`の集計結果がクエリ結果を誤解させる可能性があります。この問題に対処するには、`GROUPING()`関数を使用して、ネイティブの`NULL`値と`WITH ROLLUP`で生成された`NULL`値を区別することができます。この関数はグルーピング式をパラメータとして取り、そのグルーピング式が現在の結果で集計されているかどうかを示す`0`または`1`を返します。`1`は集計されたことを、`0`は集計されていないことを表します。

次の例は、`GROUPING()`関数の使用方法を示しています：

```sql
SELECT year, month, SUM(profit) AS profit, grouping(year) as grp_year, grouping(month) as grp_month FROM bank GROUP BY year, month WITH ROLLUP ORDER BY year DESC, month DESC;
+------+-------+--------------------+----------+-----------+
| year | month | profit             | grp_year | grp_month |
+------+-------+--------------------+----------+-----------+
| 2001 | Feb   | 22.399999618530273 |        0 |         0 |
| 2001 | NULL  | 22.399999618530273 |        0 |         1 |
| 2000 | Mar   | 31.600000381469727 |        0 |         0 |
| 2000 | Jan   | 10.300000190734863 |        0 |         0 |
| 2000 | NULL  |  41.90000057220459 |        0 |         1 |
| NULL | NULL  |  64.30000019073486 |        1 |         1 |
+------+-------+--------------------+----------+-----------+
6 rows in set (0.028 sec)
```

この出力から、`grp_year`および`grp_month`の結果から、行の集計次元に関する理解を得ることができ、`year`および`month`のグルーピング式においてネイティブの`NULL`値からの干渉を防ぐことができます。

`GROUPING()`関数は最大64個のグルーピング式をパラメータとして受け付けることができます。複数のパラメータの出力では、各パラメータが`0`または`1`の結果を生成し、これらのパラメータはそれぞれが`0`または`1`である64ビットの`UNSIGNED LONGLONG`を形成します。次の式を使用して、各パラメータのビット位置を取得できます：

```go
GROUPING(day, month, year):
  GROUPING(year)の結果
+ GROUPING(month)の結果 << 1
```
```sql
+ GROUPING(day) << 2

複数のパラメータを `GROUPING()` 関数で使用することで、任意の高次元で集計結果を効率的にフィルタリングできます。たとえば、`GROUPING(year, month)` を使用して、各年とすべての年の集計結果を素早くフィルタリングできます。

```sql
SELECT year, month, SUM(profit) AS profit, grouping(year) as grp_year, grouping(month) as grp_month FROM bank GROUP BY year, month WITH ROLLUP HAVING GROUPING(year, month) <> 0 ORDER BY year DESC, month DESC;
+------+-------+--------------------+----------+-----------+
| year | month | profit             | grp_year | grp_month |
+------+-------+--------------------+----------+-----------+
| 2001 | NULL  | 22.399999618530273 |        0 |         1 |
| 2000 | NULL  |  41.90000057220459 |        0 |         1 |
| NULL | NULL  |  64.30000019073486 |        1 |         1 |
+------+-------+--------------------+----------+-----------+
3 行がセットになっています (0.023 秒)
```

## ROLLUP 実行計画の解釈方法

多次元グループ化の要件を満たすために、多次元データ集計は `Expand` 演算子を使用してデータを複製します。各レプリカは特定の次元のグループに対応します。MPP のデータシャッフル機能を使用することで、`Expand` 演算子は複数の TiFlash ノード間で大容量のデータを迅速に再構成し、計算し、各ノードの計算能力を十分に活用します。

`Expand` 演算子の実装は `Projection` 演算子と類似していますが、`Expand` は複数レベルの `Projection` を含む、多レベルの投影操作式を持っています。元のデータの各行に対して、`Projection` 演算子は結果に 1 行のみを生成しますが、`Expand` 演算子は複数の結果行を生成します（行の数は投影操作式のレベル数に等しい）。

以下は実行計画の例です：

```sql
explain SELECT year, month, grouping(year), grouping(month), SUM(profit) AS profit FROM bank GROUP BY year, month WITH ROLLUP;
+----------------------------------------+---------+--------------+---------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| id                                     | estRows | task         | access object | operator info                                                                                                                                                                                                                        |
+----------------------------------------+---------+--------------+---------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| TableReader_44                         | 2.40    | root         |               | MppVersion: 2, data:ExchangeSender_43                                                                                                                                                                                                |
| └─ExchangeSender_43                    | 2.40    | mpp[tiflash] |               | ExchangeType: PassThrough                                                                                                                                                                                                            |
|   └─Projection_8                       | 2.40    | mpp[tiflash] |               | Column#6->Column#12, Column#7->Column#13, grouping(gid)->Column#14, grouping(gid)->Column#15, Column#9->Column#16                                                                                                                    |
|     └─Projection_38                    | 2.40    | mpp[tiflash] |               | Column#9, Column#6, Column#7, gid                                                                                                                                                                                                    |
|       └─HashAgg_36                     | 2.40    | mpp[tiflash] |               | group by:Column#6, Column#7, gid, funcs:sum(test.bank.profit)->Column#9, funcs:firstrow(Column#6)->Column#6, funcs:firstrow(Column#7)->Column#7, funcs:firstrow(gid)->gid, stream_count: 8                                           |
|         └─ExchangeReceiver_22          | 3.00    | mpp[tiflash] |               | stream_count: 8                                                                                                                                                                                                                      |
|           └─ExchangeSender_21          | 3.00    | mpp[tiflash] |               | ExchangeType: HashPartition, Compression: FAST, Hash Cols: [name: Column#6, collate: binary], [name: Column#7, collate: utf8mb4_bin], [name: gid, collate: binary], stream_count: 8                                                  |
|             └─Expand_20                | 3.00    | mpp[tiflash] |               | level-projection:[test.bank.profit, <nil>->Column#6, <nil>->Column#7, 0->gid],[test.bank.profit, Column#6, <nil>->Column#7, 1->gid],[test.bank.profit, Column#6, Column#7, 3->gid]; schema: [test.bank.profit,Column#6,Column#7,gid] |
|               └─Projection_16          | 3.00    | mpp[tiflash] |               | test.bank.profit, test.bank.year->Column#6, test.bank.month->Column#7                                                                                                                                                                |
|                 └─TableFullScan_17     | 3.00    | mpp[tiflash] | table:bank    | keep order:false, stats:pseudo                                                                                                                                                                                                       |
+----------------------------------------+---------+--------------+---------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
10 行がセットになっています (0.05 秒)
```

この実行計画の例では、`operator info` 列の `Expand_20` 行で `Expand` 演算子の多次元式を表示できます。これは 2 次元の式で構成されており、最後の行に `Expand` 演算子のスキーマ情報を表示できます。これは `schema: [test.bank.profit, Column#6, Column#7, gid]` です。

`Expand` 演算子のスキーマ情報では、`GID` が追加列として生成されます。その値は、異なる次元のグループ化ロジックに基づいて `Expand` 演算子によって計算され、その値は現在のデータレプリカと `grouping set` 間の関係を反映します。ほとんどの場合、`Expand` 演算子では 63 種類の ROLLUP のグループ化アイテムの組み合わせを表せる Bit-And 演算を使用し、64 次元のグループ化に対応します。このモードでは、TiDB は、現在のデータレプリカが複製される際に必要な次元の `grouping set` にグループ化式が含まれているかどうかに応じて `GID` 値を生成し、グループ化される列の順序で 64 ビット UINT64 値を埋めます。

前述の例では、グループ化リストの列の順序は `[year, month]` であり、ROLLUP 構文によって生成された次元グループは `{year, month}`、`{year}`、`{}` です。次元グループ `{year, month}` においては、`year` と `month` が必要な列ですので、TiDB はそれぞれに対応してビットの位置を 1 と 1 で埋めます。これにより、10 進数で 3 になる UINT64 が形成されます。そのため、プロジェクション式は `[test.bank.profit, Column#6, Column#7, 3->gid]` となります（ここで `column#6` は `year` に、`column#7` は `month` に対応します）。

以下は元のデータ行の例です：

```sql
+------+-------+------+------------+
| year | month | day  | profit     |
+------+-------+------+------------+
| 2000 | Jan   |    1 | 10.3000000 |
+------+-------+------+------------+
```

`Expand` 演算子が適用された後、以下の 3 行の結果を得られます：

```sql
+------------+------+-------+-----+
| profit     | year | month | gid |
+------------+------+-------+-----+
| 10.3000000 | 2000 | Jan   |  3  |
+------------+------+-------+-----+
| 10.3000000 | 2000 | NULL  |  1  |
+------------+------+-------+-----+
| 10.3000000 | NULL | NULL  |  0  |
+------------+------+-------+-----+
```

クエリの `SELECT` 句で `GROUPING` 関数を使用すると、`HAVING` や `ORDER BY` クローズで使用する場合と同様に、TiDB は論理最適化フェーズでこれを書き換え、`GROUP BY` 項目との関係を `grouping set` のロジックに関連する `GID` に変換し、この `GID` を新しい `GROUPING` 関数にメタデータとして埋め込みます。