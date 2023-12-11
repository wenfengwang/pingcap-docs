---
title: SHOW STATS_LOCKED
summary: SHOW STATS_LOCKEDについてのTiDBデータベースの使用概要。

# SHOW STATS_LOCKED

`SHOW STATS_LOCKED`は統計情報がロックされているテーブルを表示します。

> **警告:**
>
> 現在のバージョンでは統計情報のロックは実験的な機能です。本番環境での使用は推奨されません。

## 概要

```ebnf+diagram
ShowStatsLockedStmt ::= 'SHOW' 'STATS_LOCKED' ShowLikeOrWhereOpt

ShowLikeOrWhereOpt ::= 'LIKE' SimpleExpr | 'WHERE' Expression
```

## 例

`t`という名前のテーブルを作成し、データを挿入します。`t`テーブルの統計情報がロックされていない場合、`ANALYZE`ステートメントを正常に実行できます。

```sql
mysql> CREATE TABLE t(a INT, b INT);
Query OK, 0 rows affected (0.03 sec)

mysql> INSERT INTO t VALUES (1,2), (3,4), (5,6), (7,8);
Query OK, 4 rows affected (0.00 sec)
Records: 4  Duplicates: 0  Warnings: 0

mysql> ANALYZE TABLE t;
Query OK, 0 rows affected, 1 warning (0.02 sec)

mysql> SHOW WARNINGS;
+-------+------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Level | Code | Message                                                                                                                                                                                                               |
+-------+------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Note  | 1105 | 統計メタデータの行数はPDで取得した行数と比較してはるかに小さいため、テーブル test.t に対して自動調整されたサンプルレート 1.000000 が使用されます。このレートを使用した理由は "Row count in stats_meta is much smaller compared with the row count got by PD, use min(1, 15000/4) as the sample-rate=1"
+-------+------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

テーブル `t`の統計情報をロックし、`SHOW STATS_LOCKED`を実行します。出力にはテーブル `t`の統計情報がロックされていることが表示されます。

```sql
mysql> LOCK STATS t;
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW STATS_LOCKED;
+---------+------------+----------------+--------+
| Db_name | Table_name | Partition_name | Status |
+---------+------------+----------------+--------+
| test    | t          |                | locked |
+---------+------------+----------------+--------+
1 row in set (0.01 sec)
```

## MySQL互換性

このステートメントはMySQLの構文にTiDBの拡張機能です。

## 関連項目

* [Statistics](/statistics.md#lock-statistics)
* [LOCK STATS](/sql-statements/sql-statement-lock-stats.md)
* [UNLOCK STATS](/sql-statements/sql-statement-unlock-stats.md)