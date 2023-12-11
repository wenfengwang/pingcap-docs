---
title: MEMORY_USAGE_OPS_HISTORY
summary: `MEMORY_USAGE_OPS_HISTORY`情報スキーマシステムテーブルについて学びます。

# MEMORY_USAGE_OPS_HISTORY

`MEMORY_USAGE_OPS_HISTORY`テーブルは、メモリ関連の操作の履歴と現在のTiDBインスタンスの実行基準を記述しています。

```sql
USE information_schema;
DESC memory_usage_ops_history;
```

```sql
+----------------+---------------------+------+------+---------+-------+
| Field          | Type                | Null | Key  | Default | Extra |
+----------------+---------------------+------+------+---------+-------+
| TIME           | datetime            | NO   |      | NULL    |       |
| OPS            | varchar(20)         | NO   |      | NULL    |       |
| MEMORY_LIMIT   | bigint(21)          | NO   |      | NULL    |       |
| MEMORY_CURRENT | bigint(21)          | NO   |      | NULL    |       |
| PROCESSID      | bigint(21) unsigned | YES  |      | NULL    |       |
| MEM            | bigint(21) unsigned | YES  |      | NULL    |       |
| DISK           | bigint(21) unsigned | YES  |      | NULL    |       |
| CLIENT         | varchar(64)         | YES  |      | NULL    |       |
| DB             | varchar(64)         | YES  |      | NULL    |       |
| USER           | varchar(16)         | YES  |      | NULL    |       |
| SQL_DIGEST     | varchar(64)         | YES  |      | NULL    |       |
| SQL_TEXT       | varchar(256)        | YES  |      | NULL    |       |
+----------------+---------------------+------+------+---------+-------+
12 行が返されました (0.000 秒)
```

{{< copyable "sql" >}}

```sql
SELECT * FROM information_schema.memory_usage_ops_history;
```

```sql
+---------------------+-------------+--------------+----------------+---------------------+------------+------+-----------------+------+------+------------------------------------------------------------------+----------------------------------------------------------------------+
| TIME                | OPS         | MEMORY_LIMIT | MEMORY_CURRENT | PROCESSID           | MEM        | DISK | CLIENT          | DB   | USER | SQL_DIGEST                                                       | SQL_TEXT                                                             |
+---------------------+-------------+--------------+----------------+---------------------+------------+------+-----------------+------+------+------------------------------------------------------------------+----------------------------------------------------------------------+
| 2022-10-17 22:46:25 | SessionKill |  10737418240 |    10880237568 | 6718275530455515543 | 7905028235 |    0 | 127.0.0.1:34394 | test | root | 146b3d812852663a20635fbcf02be01688f52c8d433dafec0d496a14f0b59df6 | desc analyze select * from t t1 join t t2 on t1.a=t2.a order by t1.a |
+---------------------+-------------+--------------+----------------+---------------------+------------+------+-----------------+------+------+------------------------------------------------------------------+----------------------------------------------------------------------+
2 行が返されました (0.002 秒)
```

`MEMORY_USAGE_OPS_HISTORY`テーブルの列は以下のように説明されています:

* `TIME`: セッション終了時のタイムスタンプ。
* `OPS`: "SessionKill"
* `MEMORY_LIMIT`: タイムスタンプ時点でのTiDBのメモリ使用量の制限（バイト単位）。その値はシステム変数`tidb_server_memory_limit`の値と同じです。
* `MEMORY_CURRENT`: TiDBの現在のメモリ使用量（バイト単位）。
* `PROCESSID`: 終了したセッションの接続ID。
* `MEM`: 終了したセッションのメモリ使用量（バイト単位）。
* `DISK`: 終了したセッションのディスク使用量（バイト単位）。
* `CLIENT`: 終了したセッションのクライアント接続アドレス。
* `DB`: 終了したセッションに接続されたデータベースの名前。
* `USER`: 終了したセッションのユーザー名。
* `SQL_DIGEST`: 終了したセッションで実行されていたSQLステートメントのダイジェスト。
* `SQL_TEXT`: 終了したセッションで実行されていたSQLステートメント。