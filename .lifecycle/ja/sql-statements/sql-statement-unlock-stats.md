---
title: 統計のロック解除
summary: TiDBデータベースの統計のロック解除の概要。

# 統計のロック解除

`UNLOCK STATS`は、テーブルの統計をロック解除するために使用されます。

> **警告:**
>
> 現在のバージョンで統計をロックすることは実験的な機能です。本番環境で使用することは推奨されません。

## 概要

```ebnf+diagram
UnlockStatsStmt ::=
    'UNLOCK' 'STATS' (TableNameList) | (TableName 'PARTITION' PartitionNameList)

TableNameList ::=
    TableName (',' TableName)*

TableName ::=
    Identifier ( '.' Identifier )?

PartitionNameList ::=
    Identifier ( ',' Identifier )*
```

## 例

[LOCK STATS](/sql-statements/sql-statement-lock-stats.md)の例を参照し、テーブル `t` を作成し、その統計をロックします。

テーブル `t` の統計をロック解除し、`ANALYZE`を正常に実行できます。

```sql
mysql> UNLOCK STATS t;
Query OK, 0 rows affected (0.01 sec)

mysql> ANALYZE TABLE t;
Query OK, 0 rows affected, 1 warning (0.03 sec)

mysql> SHOW WARNINGS;
+-------+------+-----------------------------------------------------------------------------------------------------------------------------------------+
| Level | Code | Message                                                                                                                                 |
+-------+------+-----------------------------------------------------------------------------------------------------------------------------------------+
| Note  | 1105 | Analyze use auto adjusted sample rate 1.000000 for table test.t, reason to use this rate is "use min(1, 110000/8) as the sample-rate=1" |
+-------+------+-----------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

[LOCK STATS](/sql-statements/sql-statement-lock-stats.md)の例を参照し、テーブル `t` を作成し、そのパーティション `p1` の統計をロックします。

パーティション `p1` の統計をロック解除し、`ANALYZE`を正常に実行できます。

```sql
mysql> UNLOCK STATS t PARTITION p1;
Query OK, 0 rows affected (0.00 sec)

mysql> ANALYZE TABLE t PARTITION p1;
Query OK, 0 rows affected, 1 warning (0.01 sec)

mysql> SHOW WARNINGS;
+-------+------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Level | Code | Message                                                                                                                                                              |
+-------+------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Note  | 1105 | Analyze use auto adjusted sample rate 1.000000 for table test.t's partition p1, reason to use this rate is "TiDB assumes that the table is empty, use sample-rate=1" |
+-------+------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

## MySQL互換性

この文は、TiDBによるMySQL構文の拡張です。

## 関連項目

* [統計](/statistics.md#lock-statistics)
* [LOCK STATS](/sql-statements/sql-statement-lock-stats.md)
* [SHOW STATS_LOCKED](/sql-statements/sql-statement-show-stats-locked.md)