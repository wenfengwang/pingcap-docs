---
title: TiFlashでサポートされるPush-Down計算
summary: TiFlashでサポートされるPush-Down計算について学びます。

# TiFlashでサポートされるPush-Down計算

このドキュメントはTiFlashでサポートされるPush-Down計算を紹介します。

## Push-Down演算子

TiFlashは次の演算子のPush-Downをサポートしています:

* TableScan: テーブルからデータを読み込みます。
* Selection: データをフィルタリングします。
* HashAgg: [ハッシュ集約](/explain-aggregation.md#hash-aggregation)アルゴリズムに基づくデータ集約を実行します。
* StreamAgg: [ストリーム集約](/explain-aggregation.md#stream-aggregation)アルゴリズムに基づくデータ集約を実行します。StreamAggは`GROUP BY`条件なしでの集約のみをサポートします。
* TopN: TopNの計算を実行します。
* Limit: Limitの計算を実行します。
* Project: プロジェクションの計算を実行します。
* HashJoin: [ハッシュ結合](/explain-joins.md#hash-join)アルゴリズムを使用して結合を実行しますが、以下の条件があります:
    * この演算子は[MPPモード](/tiflash/use-tiflash-mpp-mode.md)でのみPushDownが可能です。
    * インナージョイン、レフトジョイン、セミジョイン、アンチセミジョイン、レフトセミジョイン、アンチレフトセミジョインのサポートされる結合です。
    * 前述の結合はEqui JoinとNon-Equi Join（Cartesian JoinまたはNull-aware Semi Join）の両方をサポートしています。Cartesian JoinまたはNull-aware Semi Joinの計算時には、Shuffle Hash Joinアルゴリズムの代わりにBroadcastアルゴリズムが使用されます。
* [ウィンドウ関数](/functions-and-operators/window-functions.md): 現在、TiFlashは`ROW_NUMBER()`、`RANK()`、`DENSE_RANK()`、`LEAD()`、`LAG()`、`FIRST_VALUE()`、`LAST_VALUE()`をサポートしています。

TiDBでは、演算子はツリー構造で構成されています。演算子をTiFlashにPushDownするためには、以下の前提条件をすべて満たす必要があります:

+ すべての子演算子がTiFlashにPushDownできること。
+ 演算子に式が含まれている場合（ほとんどの演算子には式が含まれています）、その演算子のすべての式がTiFlashにPushDownできること。

## Push-Down式

TiFlashは以下のPush-Down式をサポートしています:

| 式の種類 | 操作 |
| :-------------- | :------------------------------------- |
| [数値関数と演算子](/functions-and-operators/numeric-functions-and-operators.md) | `+`, `-`, `/`, `*`, `%`, `>=`, `<=`, `=`, `!=`, `<`, `>`, `ROUND()`, `ABS()`, `FLOOR(int)`, `CEIL(int)`, `CEILING(int)`, `SQRT()`, `LOG()`, `LOG2()`, `LOG10()`, `LN()`, `EXP()`, `POW()`, `SIGN()`, `RADIANS()`, `DEGREES()`, `CONV()`, `CRC32()`, `GREATEST(int/real)`, `LEAST(int/real)` |
| [制御フロー関数](/functions-and-operators/control-flow-functions.md)および[演算子](/functions-and-operators/operators.md) | `AND`, `OR`, `NOT`, `CASE WHEN`, `IF()`, `IFNULL()`, `ISNULL()`, `IN`, `LIKE`, `ILIKE`, `COALESCE`, `IS` |
| [ビット演算](/functions-and-operators/bit-functions-and-operators.md) | `&` (bitand), <code>\|</code> (bitor), `~` (bitneg), `^` (bitxor) |
| [文字列関数](/functions-and-operators/string-functions.md) | `SUBSTR()`, `CHAR_LENGTH()`, `REPLACE()`, `CONCAT()`, `CONCAT_WS()`, `LEFT()`, `RIGHT()`, `ASCII()`, `LENGTH()`, `TRIM()`, `LTRIM()`, `RTRIM()`, `POSITION()`, `FORMAT()`, `LOWER()`, `UCASE()`, `UPPER()`, `SUBSTRING_INDEX()`, `LPAD()`, `RPAD()`, `STRCMP()` |
| [正規表現関数と演算子](/functions-and-operators/string-functions.md) | `REGEXP`, `REGEXP_LIKE()`, `REGEXP_INSTR()`, `REGEXP_SUBSTR()`, `REGEXP_REPLACE()` |
| [日付関数](/functions-and-operators/date-and-time-functions.md) | `DATE_FORMAT()`, `TIMESTAMPDIFF()`, `FROM_UNIXTIME()`, `UNIX_TIMESTAMP(int)`, `UNIX_TIMESTAMP(decimal)`, `STR_TO_DATE(date)`, `STR_TO_DATE(datetime)`, `DATEDIFF()`, `YEAR()`, `MONTH()`, `DAY()`, `EXTRACT(datetime)`, `DATE()`, `HOUR()`, `MICROSECOND()`, `MINUTE()`, `SECOND()`, `SYSDATE()`, `DATE_ADD/ADDDATE(datetime, int)`, `DATE_ADD/ADDDATE(string, int/real)`, `DATE_SUB/SUBDATE(datetime, int)`, `DATE_SUB/SUBDATE(string, int/real)`, `QUARTER()`, `DAYNAME()`, `DAYOFMONTH()`, `DAYOFWEEK()`, `DAYOFYEAR()`, `LAST_DAY()`, `MONTHNAME()`, `TO_SECONDS()`, `TO_DAYS()`, `FROM_DAYS()`, `WEEKOFYEAR()`
| [JSON関数](/functions-and-operators/json-functions.md) | `JSON_LENGTH()`, `->`, `->>`, `JSON_EXTRACT()` |
| [変換関数](/functions-and-operators/cast-functions-and-operators.md) | `CAST(int AS DOUBLE), CAST(int AS DECIMAL)`, `CAST(int AS STRING)`, `CAST(int AS TIME)`, `CAST(double AS INT)`, `CAST(double AS DECIMAL)`, `CAST(double AS STRING)`, `CAST(double AS TIME)`, `CAST(string AS INT)`, `CAST(string AS DOUBLE), CAST(string AS DECIMAL)`, `CAST(string AS TIME)`, `CAST(decimal AS INT)`, `CAST(decimal AS STRING)`, `CAST(decimal AS TIME)`, `CAST(time AS INT)`, `CAST(time AS DECIMAL)`, `CAST(time AS STRING)`, `CAST(time AS REAL)` |
| [集約関数](/functions-and-operators/aggregate-group-by-functions.md) | `MIN()`, `MAX()`, `SUM()`, `COUNT()`, `AVG()`, `APPROX_COUNT_DISTINCT()`, `GROUP_CONCAT()` |
| [その他の関数](/functions-and-operators/miscellaneous-functions.md) | `INET_NTOA()`, `INET_ATON()`, `INET6_NTOA()`, `INET6_ATON()` |

## 制限事項

* ビット、セット、ジオメトリのタイプを含む式はTiFlashにPushDownできません。

* `DATE_ADD()`、`DATE_SUB()`、`ADDDATE()`、`SUBDATE()`関数は以下のインターバルタイプのみをサポートします。他のインターバルタイプが使用された場合、TiFlashはエラーを報告します。

    * DAY
    * WEEK
    * MONTH
    * YEAR
    * HOUR
    * MINUTE
    * SECOND

クエリでサポートされていないPush-Down計算が発生する場合、TiDBは残りの計算を完了する必要があり、TiFlashのアクセラレーション効果に大きく影響することがあります。現在サポートされていない演算子や式は将来のバージョンでサポートされる可能性があります。

`MAX()`などの関数は、集約関数として使用される場合はPushDownがサポートされますが、ウィンドウ関数として使用される場合はサポートされません。

## 例

このセクションでは、演算子や式をTiFlashにPushDownするいくつかの例を提供しています。

### 例1: 演算子をTiFlashにPushDown

```sql
CREATE TABLE t(id INT PRIMARY KEY, a INT);
ALTER TABLE t SET TIFLASH REPLICA 1;

EXPLAIN SELECT * FROM t LIMIT 3;

+------------------------------+---------+--------------+---------------+--------------------------------+
| id                           | estRows | task         | access object | operator info                  |
+------------------------------+---------+--------------+---------------+--------------------------------+
| Limit_9                      | 3.00    | root         |               | offset:0, count:3              |
| └─TableReader_17             | 3.00    | root         |               | data:ExchangeSender_16         |
|   └─ExchangeSender_16        | 3.00    | mpp[tiflash] |               | ExchangeType: PassThrough      |
|     └─Limit_15               | 3.00    | mpp[tiflash] |               | offset:0, count:3              |
|       └─TableFullScan_14     | 3.00    | mpp[tiflash] | table:t       | keep order:false, stats:pseudo |
+------------------------------+---------+--------------+---------------+--------------------------------+
5 rows in set (0.18 sec)
```

上記の例では、オペレータ `Limit` がデータをフィルタリングするためにTiFlashにPushDownされており、ネットワーク経由で転送されるデータの量を減らし、ネットワークのオーバーヘッドを減らすのに役立ちます。これは、`Limit_15`オペレータの`task`列の`mpp[tiflash]`の値で示されています。

### 例2: 式をTiFlashにPushDown

```sql
CREATE TABLE t(id INT PRIMARY KEY, a INT);
ALTER TABLE t SET TIFLASH REPLICA 1;
INSERT INTO t(id,a) VALUES (1,2),(2,4),(11,2),(12,4),(13,4),(14,7);

EXPLAIN SELECT MAX(id + a) FROM t GROUP BY a;
```
```

+------------------------------------+---------+--------------+---------------+---------------------------------------------------------------------------+
| id                                 | estRows | task         | access object | operator info                                                             |
+------------------------------------+---------+--------------+---------------+---------------------------------------------------------------------------+
| TableReader_45                     | 4.80    | root         |               | data:ExchangeSender_44                                                    |
| └─ExchangeSender_44                | 4.80    | mpp[tiflash] |               | ExchangeType: PassThrough                                                 |
|   └─Projection_39                  | 4.80    | mpp[tiflash] |               | Column#3                                                                  |
|     └─HashAgg_37                   | 4.80    | mpp[tiflash] |               | group by:Column#9, funcs:max(Column#8)->Column#3                          |
|       └─Projection_46              | 6.00    | mpp[tiflash] |               | plus(test.t.id, test.t.a)->Column#8, test.t.a                             |
|         └─ExchangeReceiver_23      | 6.00    | mpp[tiflash] |               |                                                                           |
|           └─ExchangeSender_22      | 6.00    | mpp[tiflash] |               | ExchangeType: HashPartition, Hash Cols: [name: test.t.a, collate: binary] |
|             └─TableFullScan_21     | 6.00    | mpp[tiflash] | table:t       | keep order:false, stats:pseudo                                            |
+------------------------------------+---------+--------------+---------------+---------------------------------------------------------------------------+
8 rows in set (0.18 sec)
```

前述の例では、`id + a` の式が事前計算のためにTiFlashにプッシュダウンされます。これにより、ネットワーク経由で転送されるデータ量が削減され、全体の計算パフォーマンスが向上します。`task` 列の `operator` 列が `plus(test.t.id, test.t.a)` の値である行の `task` 列で `mpp[tiflash]` の値が示されています。

### 例3: プッシュダウンの制限

```sql
CREATE TABLE t(id INT PRIMARY KEY, a INT);
ALTER TABLE t SET TIFLASH REPLICA 1;
INSERT INTO t(id,a) VALUES (1,2),(2,4),(11,2),(12,4),(13,4),(14,7);

EXPLAIN SELECT id FROM t WHERE TIME(now()+ a) < '12:00:00';

+-----------------------------+---------+--------------+---------------+--------------------------------------------------------------------------------------------------+
| id                          | estRows | task         | access object | operator info                                                                                    |
+-----------------------------+---------+--------------+---------------+--------------------------------------------------------------------------------------------------+
| Projection_4                | 4.80    | root         |               | test.t.id                                                                                        |
| └─Selection_6               | 4.80    | root         |               | lt(cast(time(cast(plus(20230110083056, test.t.a), var_string(20))), var_string(10)), "12:00:00") |
|   └─TableReader_11          | 6.00    | root         |               | data:ExchangeSender_10                                                                           |
|     └─ExchangeSender_10     | 6.00    | mpp[tiflash] |               | ExchangeType: PassThrough                                                                        |
|       └─TableFullScan_9     | 6.00    | mpp[tiflash] | table:t       | keep order:false, stats:pseudo                                                                   |
+-----------------------------+---------+--------------+---------------+--------------------------------------------------------------------------------------------------+
5 rows in set, 3 warnings (0.20 sec)
```

前述の例では、TiFlashで`TableFullScan` のみが実行されます。その他の関数は `root` で計算およびフィルタリングされ、TiFlashにプッシュダウンされません。

次のコマンドを実行して、TiFlashに完全にプッシュダウンできない演算子および式を特定できます。

```sql
SHOW WARNINGS;

+---------+------+------------------------------------------------------------------------------------------------------------------------------------+
| Level   | Code | Message                                                                                                                            |
+---------+------+------------------------------------------------------------------------------------------------------------------------------------+
| Warning | 1105 | Scalar function 'time'(signature: Time, return type: time) is not supported to push down to storage layer now.                     |
| Warning | 1105 | Scalar function 'cast'(signature: CastDurationAsString, return type: var_string(10)) is not supported to push down to tiflash now. |
| Warning | 1105 | Scalar function 'cast'(signature: CastDurationAsString, return type: var_string(10)) is not supported to push down to tiflash now. |
+---------+------+------------------------------------------------------------------------------------------------------------------------------------+
3 rows in set (0.18 sec)
```

前述の例の式は、`Time` および `Cast` 関数がTiFlashにプッシュダウンできないため、TiFlashに完全にプッシュダウンすることができません。

### 例4: ウィンドウ関数

```sql
CREATE TABLE t(id INT PRIMARY KEY, c1 VARCHAR(100));
ALTER TABLE t SET TIFLASH REPLICA 1;
INSERT INTO t VALUES(1,"foo"),(2,"bar"),(3,"bar foo"),(10,"foo"),(20,"bar"),(30,"bar foo");

EXPLAIN SELECT id, ROW_NUMBER() OVER (PARTITION BY id > 10) FROM t;
+----------------------------------+----------+--------------+---------------+---------------------------------------------------------------------------------------------------------------+
| id                               | estRows  | task         | access object | operator info                                                                                                 |
+----------------------------------+----------+--------------+---------------+---------------------------------------------------------------------------------------------------------------+
| TableReader_30                   | 10000.00 | root         |               | MppVersion: 1, data:ExchangeSender_29                                                                         |
| └─ExchangeSender_29              | 10000.00 | mpp[tiflash] |               | ExchangeType: PassThrough                                                                                     |
|   └─Projection_7                 | 10000.00 | mpp[tiflash] |               | test.t.id, Column#5, stream_count: 4                                                                          |
|     └─Window_28                  | 10000.00 | mpp[tiflash] |               | row_number()->Column#5 over(partition by Column#4 rows between current row and current row), stream_count: 4  |
|       └─Sort_14                  | 10000.00 | mpp[tiflash] |               | Column#4, stream_count: 4                                                                                     |
|         └─ExchangeReceiver_13    | 10000.00 | mpp[tiflash] |               | stream_count: 4                                                                                               |
|           └─ExchangeSender_12    | 10000.00 | mpp[tiflash] |               | ExchangeType: HashPartition, Compression: FAST, Hash Cols: [name: Column#4, collate: binary], stream_count: 4 |
|             └─Projection_10      | 10000.00 | mpp[tiflash] |               | test.t.id, gt(test.t.id, 10)->Column#4                                                                        |
|               └─TableFullScan_11 | 10000.00 | mpp[tiflash] | table:t       | keep order:false, stats:pseudo                                                                                |
+----------------------------------+----------+--------------+---------------+---------------------------------------------------------------------------------------------------------------+
9 rows in set (0.0073 sec)

```

この出力では、`Window` 操作の `task` 列に `mpp[tiflash]` の値があることから、`ROW_NUMBER() OVER (PARTITION BY id > 10)` 操作がTiFlashにプッシュダウンできることが示されています。

```sql
CREATE TABLE t(id INT PRIMARY KEY, c1 VARCHAR(100));
ALTER TABLE t SET TIFLASH REPLICA 1;
INSERT INTO t VALUES(1,"foo"),(2,"bar"),(3,"bar foo"),(10,"foo"),(20,"bar"),(30,"bar foo");

EXPLAIN SELECT id, MAX(id) OVER (PARTITION BY id > 10) FROM t;
+-----------------------------+----------+-----------+---------------+------------------------------------------------------------+
| id                          | estRows  | task      | access object | operator info                                              |
+-----------------------------+----------+-----------+---------------+------------------------------------------------------------+
| Projection_6                | 10000.00 | root      |               | test.t1.id, Column#5                                       |
| └─Shuffle_14                | 10000.00 | root      |               | execution info: concurrency:5, data sources:[Projection_8] |
|   └─Window_7                | 10000.00 | root      |               | max(test.t1.id)->Column#5 over(partition by Column#4)      |
|     └─Sort_13               | 10000.00 | root      |               | Column#4                                                   |
|       └─Projection_8        | 10000.00 | root      |               | test.t1.id, gt(test.t1.id, 10)->Column#4                   |
|         └─TableReader_10    | 10000.00 | root      |               | data:TableFullScan_9                                       |
|           └─TableFullScan_9 | 10000.00 | cop[tikv] | table:t1      | keep order:false, stats:pseudo                             |
+-----------------------------+----------+-----------+---------------+------------------------------------------------------------+
7 rows in set (0.0010 sec)
```
```
In this output, you can see that the `Window` operation has a value of `root` in the `task` column, indicating that the `MAX(id) OVER (PARTITION BY id > 10)` operation cannot be pushed down to TiFlash. This is because `MAX()` is only supported for push-down as an aggregate function and not as a window function.
```