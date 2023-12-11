---
title: SHOW [FULL] PROCESSLIST | TiDB SQLステートメントリファレンス
summary: TiDBデータベースのSHOW [FULL] PROCESSLISTの使用方法の概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-show-processlist/','/docs/dev/reference/sql/statements/show-processlist/']
---

# SHOW [FULL] PROCESSLIST

このステートメントは、現在のセッションをTiDBサーバーに接続しているものをリストアップします。オプションのキーワード`FULL`が指定されていない限り、`Info`列にはクエリのテキストが含まれ、これは切り詰められます。

## 概要

**ShowProcesslistStmt:**

![ShowProcesslistStmt](/media/sqlgram/ShowProcesslistStmt.png)

**OptFull:**

![OptFull](/media/sqlgram/OptFull.png)

## 例

```sql
mysql> SHOW PROCESSLIST;
+------+------+-----------------+------+---------+------+------------+------------------+
| Id   | User | Host            | db   | Command | Time | State      | Info             |
+------+------+-----------------+------+---------+------+------------+------------------+
|    5 | root | 127.0.0.1:45970 | test | Query   |    0 | autocommit | SHOW PROCESSLIST |
+------+------+-----------------+------+---------+------+------------+------------------+
1 行が返されました (0.00 秒)
```

## MySQL互換性

* TiDBの`State`列は非記述的です。値としての状態を単一の値で表現することはTiDBではより複雑です。なぜなら、クエリは並列で実行され、各goroutineは常に異なる状態を持っているからです。

## 関連項目

* [KILL \[TIDB\]](/sql-statements/sql-statement-kill.md)