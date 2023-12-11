---
title: SHOW STATS_HEALTHY
summary: SHOW STATS_HEALTHYを使用したTiDBデータベースの概要。

# SHOW STATS_HEALTHY

`SHOW STATS_HEALTHY`ステートメントは、統計情報の精度がどの程度信頼されているかの推定値を表示します。パーセンテージが低い健康状態のテーブルは、最適でないクエリ実行計画を生成する可能性があります。

テーブルの健康状態は、`ANALYZE`テーブルコマンドを実行することで改善できます。健康状態が[`tidb_auto_analyze_ratio`](/system-variables.md#tidb_auto_analyze_ratio)のしきい値以下になると、`ANALYZE`は自動的に実行されます。

## 概要

**ShowStmt**

![ShowStmt](/media/sqlgram/ShowStmt.png)

**ShowTargetFiltertable**

![ShowTargetFilterable](/media/sqlgram/ShowTargetFilterable.png)

**ShowLikeOrWhereOpt**

![ShowLikeOrWhereOpt](/media/sqlgram/ShowLikeOrWhereOpt.png)

## 例

例としてデータをロードし、`ANALYZE`を実行します:

{{< copyable "sql" >}}

```sql
CREATE TABLE t1 (
 id INT NOT NULL PRIMARY KEY auto_increment,
 b INT NOT NULL,
 pad VARBINARY(255),
 INDEX(b)
);

INSERT INTO t1 SELECT NULL, FLOOR(RAND()*1000), RANDOM_BYTES(255) FROM dual;
INSERT INTO t1 SELECT NULL, FLOOR(RAND()*1000), RANDOM_BYTES(255) FROM t1 a JOIN t1 b JOIN t1 c LIMIT 100000;
INSERT INTO t1 SELECT NULL, FLOOR(RAND()*1000), RANDOM_BYTES(255) FROM t1 a JOIN t1 b JOIN t1 c LIMIT 100000;
INSERT INTO t1 SELECT NULL, FLOOR(RAND()*1000), RANDOM_BYTES(255) FROM t1 a JOIN t1 b JOIN t1 c LIMIT 100000;
INSERT INTO t1 SELECT NULL, FLOOR(RAND()*1000), RANDOM_BYTES(255) FROM t1 a JOIN t1 b JOIN t1 c LIMIT 100000;
INSERT INTO t1 SELECT NULL, FLOOR(RAND()*1000), RANDOM_BYTES(255) FROM t1 a JOIN t1 b JOIN t1 c LIMIT 100000;
INSERT INTO t1 SELECT NULL, FLOOR(RAND()*1000), RANDOM_BYTES(255) FROM t1 a JOIN t1 b JOIN t1 c LIMIT 100000;
SELECT SLEEP(1);
ANALYZE TABLE t1;
SHOW STATS_HEALTHY; # 健康状態は100%であるべきです
```

```sql
...
mysql> SHOW STATS_HEALTHY;
+---------+------------+----------------+---------+
| Db_name | Table_name | Partition_name | Healthy |
+---------+------------+----------------+---------+
| test    | t1         |                |     100 |
+---------+------------+----------------+---------+
1 row in set (0.00 sec)
```

レコードの約30%を削除する大量の更新を実行します。統計情報の健康状態を確認します。

{{< copyable "sql" >}}

```sql
DELETE FROM t1 WHERE id BETWEEN 101010 AND 201010; # レコードの約30%を削除する
SHOW STATS_HEALTHY; 
```

```sql
mysql> SHOW STATS_HEALTHY;
+---------+------------+----------------+---------+
| Db_name | Table_name | Partition_name | Healthy |
+---------+------------+----------------+---------+
| test    | t1         |                |      50 |
+---------+------------+----------------+---------+
1 row in set (0.00 sec)
```

## MySQL互換性

このステートメントはMySQL構文のTiDB拡張です。

## 関連情報

* [ANALYZE](/sql-statements/sql-statement-analyze-table.md)
* [統計情報の概要](/statistics.md)