---
title: SHOW ANALYZE STATUS
summary: SHOW ANALYZE STATUSのTiDBデータベースにおける使用方法の概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-show-analyze-status/']
---

# SHOW ANALYZE STATUS

`SHOW ANALYZE STATUS`ステートメントは、TiDBによって実行されている統計収集タスクと、過去のタスクレコードの一部を表示します。

TiDB v6.1.0から、`SHOW ANALYZE STATUS`ステートメントは、クラスタレベルのタスクを表示できるようになりました。TiDBを再起動しても、このステートメントを使用して再起動前のタスクレコードを表示できます。TiDB v6.1.0より前では、`SHOW ANALYZE STATUS`ステートメントはインスタンスレベルのタスクのみを表示でき、タスクレコードはTiDBを再起動すると消去されます。

TiDB v6.1.0から、システムテーブル`mysql.analyze_jobs`を使用して、過去7日間のタスク履歴を表示できます。

TiDB v7.3.0から、システムテーブル`mysql.analyze_jobs`または`SHOW ANALYZE STATUS`を使用して、現在の`ANALYZE`タスクの進行状況を表示できます。

## 構文

```ebnf+diagram
ShowAnalyzeStatusStmt ::= 'SHOW' 'ANALYZE' 'STATUS' ShowLikeOrWhereOpt

ShowLikeOrWhereOpt ::= 'LIKE' SimpleExpr | 'WHERE' Expression
```

## 例

```sql
mysql> create table t(x int, index idx(x)) partition by hash(x) partitions 2;
Query OK, 0 rows affected (0.69 sec)

mysql> set @@tidb_analyze_version = 1;
Query OK, 0 rows affected (0.00 sec)

mysql> analyze table t;
Query OK, 0 rows affected (0.20 sec)

mysql> show analyze status;
+--------------+------------+----------------+-------------------+----------------+---------------------+---------------------+----------+-------------+----------------+------------+------------------+----------+---------------------+
| Table_schema | Table_name | Partition_name | Job_info          | Processed_rows | Start_time          | End_time            | State    | Fail_reason | Instance       | Process_ID | Remaining_seconds| Progress | Estimated_total_rows|
+--------------+------------+----------------+-------------------+----------------+---------------------+---------------------+----------+-------------+----------------+------------+------------------+----------+---------------------+
| test         | t          | p1             | analyze index idx |              0 | 2022-05-27 11:29:46 | 2022-05-27 11:29:46 | finished | NULL        | 127.0.0.1:4000 | NULL       | NULL             | NULL     | NULL                |
| test         | t          | p0             | analyze index idx |              0 | 2022-05-27 11:29:46 | 2022-05-27 11:29:46 | finished | NULL        | 127.0.0.1:4000 | NULL       | NULL             | NULL     | NULL                |
| test         | t          | p1             | analyze columns   |              0 | 2022-05-27 11:29:46 | 2022-05-27 11:29:46 | finished | NULL        | 127.0.0.1:4000 | NULL       | NULL             | NULL     | NULL                |
| test         | t          | p0             | analyze columns   |              0 | 2022-05-27 11:29:46 | 2022-05-27 11:29:46 | finished | NULL        | 127.0.0.1:4000 | NULL       | NULL             | NULL     | NULL                |
| test         | t1         | p0             | analyze columns   |       28523259 | 2022-05-27 11:29:46 | 2022-05-27 11:29:46 | running  | NULL        | 127.0.0.1:4000 | 690208308  | 0s               | 0.9843   | 28978290            |
+--------------+------------+----------------+-------------------+----------------+---------------------+---------------------+----------+-------------+----------------+------------+------------------+----------+---------------------+
4 rows in set (0.01 sec)

mysql> set @@tidb_analyze_version = 2;
Query OK, 0 rows affected (0.00 sec)

mysql> analyze table t;
Query OK, 0 rows affected, 2 warnings (0.03 sec)

mysql> show analyze status;
+--------------+------------+----------------+--------------------------------------------------------------------+----------------+---------------------+---------------------+----------+-------------+----------------+------------+--------------------+----------+----------------------+
| Table_schema | Table_name | Partition_name | Job_info                                                           | Processed_rows | Start_time          | End_time            | State    | Fail_reason | Instance       | Process_ID | Remaining_seconds  | Progress | Estimated_total_rows |
+--------------+------------+----------------+--------------------------------------------------------------------+----------------+---------------------+---------------------+----------+-------------+----------------+------------+--------------------+----------+----------------------+
| test         | t          | p1             | analyze table all columns with 256 buckets, 500 topn, 1 samplerate |              0 | 2022-05-27 11:30:12 | 2022-05-27 11:30:12 | finished | NULL        | 127.0.0.1:4000 | NULL       | NULL               | NULL     | NULL                 |
| test         | t          | p0             | analyze table all columns with 256 buckets, 500 topn, 1 samplerate |              0 | 2022-05-27 11:30:12 | 2022-05-27 11:30:12 | finished | NULL        | 127.0.0.1:4000 | NULL       | NULL               | NULL     | NULL                 |
| test         | t          | p1             | analyze index idx                                                  |              0 | 2022-05-27 11:29:46 | 2022-05-27 11:29:46 | finished | NULL        | 127.0.0.1:4000 | NULL       | NULL               | NULL     | NULL                 |
| test         | t          | p0             | analyze index idx                                                  |              0 | 2022-05-27 11:29:46 | 2022-05-27 11:29:46 | finished | NULL        | 127.0.0.1:4000 | NULL       | NULL               | NULL     | NULL                 |
| test         | t          | p1             | analyze columns                                                    |              0 | 2022-05-27 11:29:46 | 2022-05-27 11:29:46 | finished | NULL        | 127.0.0.1:4000 | NULL       | NULL               | NULL     | NULL                 |
| test         | t          | p0             | analyze columns                                                    |              0 | 2022-05-27 11:29:46 | 2022-05-27 11:29:46 | finished | NULL        | 127.0.0.1:4000 | NULL       | NULL               | NULL     | NULL                 |
+--------------+------------+----------------+--------------------------------------------------------------------+----------------+---------------------+---------------------+----------+-------------+----------------+------------+--------------------+----------+----------------------+
6 rows in set (0.00 sec)
```

## MySQLの互換性

このステートメントは、MySQL構文のTiDBの拡張機能です。

## 関連情報

* [ANALYZE_STATUSテーブル](/information-schema/information-schema-analyze-status.md)