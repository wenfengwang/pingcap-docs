---
title: MEMORY_USAGE
summary: `MEMORY_USAGE`情報スキーマシステムテーブルの学習。

# MEMORY_USAGE

`MEMORY_USAGE`テーブルは、現在のTiDBインスタンスのメモリ使用状況を表します。

```sql
USE information_schema;
DESC memory_usage;
```

```sql
+--------------------+-------------+------+------+---------+-------+
| Field              | Type        | Null | Key  | Default | Extra |
+--------------------+-------------+------+------+---------+-------+
| MEMORY_TOTAL       | bigint(21)  | NO   |      | NULL    |       |
| MEMORY_LIMIT       | bigint(21)  | NO   |      | NULL    |       |
| MEMORY_CURRENT     | bigint(21)  | NO   |      | NULL    |       |
| MEMORY_MAX_USED    | bigint(21)  | NO   |      | NULL    |       |
| CURRENT_OPS        | varchar(50) | YES  |      | NULL    |       |
| SESSION_KILL_LAST  | datetime    | YES  |      | NULL    |       |
| SESSION_KILL_TOTAL | bigint(21)  | NO   |      | NULL    |       |
| GC_LAST            | datetime    | YES  |      | NULL    |       |
| GC_TOTAL           | bigint(21)  | NO   |      | NULL    |       |
| DISK_USAGE         | bigint(21)  | NO   |      | NULL    |       |
| QUERY_FORCE_DISK   | bigint(21)  | NO   |      | NULL    |       |
+--------------------+-------------+------+------+---------+-------+
11 rows in set (0.000 sec)
```

{{< copyable "sql" >}}

```sql
SELECT * FROM information_schema.memory_usage;
```

```sql
+--------------+--------------+----------------+-----------------+-------------+---------------------+--------------------+---------------------+----------+------------+------------------+
| MEMORY_TOTAL | MEMORY_LIMIT | MEMORY_CURRENT | MEMORY_MAX_USED | CURRENT_OPS | SESSION_KILL_LAST   | SESSION_KILL_TOTAL | GC_LAST             | GC_TOTAL | DISK_USAGE | QUERY_FORCE_DISK |
+--------------+--------------+----------------+-----------------+-------------+---------------------+--------------------+---------------------+----------+------------+------------------+
|  33674170368 |  10737418240 |     5097644032 |     10826604544 | NULL        | 2022-10-17 22:47:47 |                  1 | 2022-10-17 22:47:47 |       20 |          0 |                0 |
+--------------+--------------+----------------+-----------------+-------------+---------------------+--------------------+---------------------+----------+------------+------------------+
2 rows in set (0.002 sec)
```

`MEMORY_USAGE`テーブルの列は以下のように説明されます：

* MEMORY_TOTAL: TiDBの利用可能な合計メモリ（バイト単位）。
* MEMORY_LIMIT: TiDBのメモリ使用量制限（バイト単位）。この値は、システム変数[`tidb_server_memory_limit`](/system-variables.md#tidb_server_memory_limit-new-in-v640)の値と同じです。
* MEMORY_CURRENT: TiDBの現在のメモリ使用量（バイト単位）。
* MEMORY_MAX_USED: TiDBの起動から現在までの間に使用された最大メモリ量（バイト単位）。
* CURRENT_OPS: "shrinking" | null。"shrinking"は、TiDBがメモリ使用量を縮小する操作を実行していることを意味します。
* SESSION_KILL_LAST: セッションが終了した最後の時刻のタイムスタンプ。
* SESSION_KILL_TOTAL: TiDBが起動してから現在までの間にセッションが終了した回数。
* GC_LAST: メモリ使用量によってGolang GCが最後にトリガーされた時刻のタイムスタンプ。
* GC_TOTAL: TiDBが起動してから現在までの間にメモリ使用量によってGolang GCがトリガーされた回数。
* DISK_USAGE: 現在のデータスピル操作のディスク使用量（バイト単位）。
* QUERY_FORCE_DISK: TiDBが起動してから現在までの間にデータがディスクにスピルされた回数。