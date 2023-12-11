---
title: SHOW WARNINGS | TiDB SQLステートメントリファレンス
summary: TiDBデータベースのSHOW WARNINGSの使用方法についての概要。
aliases: ['/docs/dev/sql-statements/sql-statement-show-warnings/','/docs/dev/reference/sql/statements/show-warnings/']
---

# SHOW WARNINGS

このステートメントは、現在のクライアント接続で以前に実行されたステートメントで発生した警告のリストを表示します。 MySQLと同様に、`sql_mode`はエラーと警告の発生を大きく左右します。

## 概要

**ShowWarningsStmt：**

![ShowWarningsStmt](/media/sqlgram/ShowWarningsStmt.png)

## 例

```sql
mysql> CREATE TABLE t1 (a INT UNSIGNED);
クエリが正常に完了しました。0行が変更されました（0.11秒）

mysql> INSERT INTO t1 VALUES (0);
クエリが正常に完了しました。1行が変更されました（0.02秒）

mysql> SELECT 1/a FROM t1;
+------+
| 1/a  |
+------+
| NULL |
+------+
1行がセットにあり、1つの警告があります（0.00秒）

mysql> SHOW WARNINGS;
+---------+------+---------------+
| Level   | Code | Message       |
+---------+------+---------------+
| Warning | 1365 | Division by 0 |
+---------+------+---------------+
1行がセットにあります（0.00秒）

mysql> INSERT INTO t1 VALUES (-1);
ERROR 1264 (22003): 行1の列'a'の値が範囲外です
mysql> SELECT * FROM t1;
+------+
| a    |
+------+
|    0 |
+------+
1行がセットにあります（0.00秒）

mysql> SET sql_mode='';
クエリが正常に完了しました。0行が変更されました（0.00秒）

mysql> INSERT INTO t1 VALUES (-1);
クエリが正常に完了しました。1行が変更されました、1つの警告があります（0.01秒）

mysql> SHOW WARNINGS;
+---------+------+---------------------------+
| Level   | Code | Message                   |
+---------+------+---------------------------+
| Warning | 1690 | constant -1 overflows int |
+---------+------+---------------------------+
1行がセットにあります（0.00秒）

mysql> SELECT * FROM t1;
+------+
| a    |
+------+
|    0 |
|    0 |
+------+
2行がセットにあります（0.00秒）

```

## MySQL互換性

TiDBの`SHOW WARNINGS`ステートメントは、MySQLと完全に互換性があります。互換性に違いがある場合は、[バグを報告してください](https://docs.pingcap.com/tidb/stable/support)。

## 関連項目

* [SHOW ERRORS](/sql-statements/sql-statement-show-errors.md)