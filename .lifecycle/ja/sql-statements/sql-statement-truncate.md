---
title: TRUNCATE | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースにおけるTRUNCATEの使用概要についての概要。
aliases: ['/docs/dev/sql-statements/sql-statement-truncate/','/docs/dev/reference/sql/statements/truncate/']
---

# TRUNCATE

`TRUNCATE`ステートメントは、非トランザクショナルな方法でテーブルからすべてのデータを削除します。 `TRUNCATE`は、以前の定義で`DROP TABLE` + `CREATE TABLE`とセマンティック的に同じと考えることができます。

`TRUNCATE TABLE tableName`と`TRUNCATE tableName`の両方が有効な構文です。

## シノプシス 

**TruncateTableStmt:**

![TruncateTableStmt](/media/sqlgram/TruncateTableStmt.png)

**OptTable:**

![OptTable](/media/sqlgram/OptTable.png)

**TableName:**

![TableName](/media/sqlgram/TableName.png)

## 例

```sql
mysql> CREATE TABLE t1 (a INT NOT NULL PRIMARY KEY);
クエリは成功しました。変更された行: 0 行に影響を与えました。(0.11秒)

mysql> INSERT INTO t1 VALUES (1),(2),(3),(4),(5);
クエリは成功しました。変更された行: 5 行に影響を与えました。(0.01秒)
レコード: 5  重複: 0  警告: 0

mysql> SELECT * FROM t1;
+---+
| a |
+---+
| 1 |
| 2 |
| 3 |
| 4 |
| 5 |
+---+
5行が選択されました。(0.00秒)

mysql> TRUNCATE t1;
クエリは成功しました。変更された行: 0 行に影響を与えました。(0.11秒)

mysql> SELECT * FROM t1;
空のセット。(0.00秒)

mysql> INSERT INTO t1 VALUES (1),(2),(3),(4),(5);
クエリは成功しました。変更された行: 5 行に影響を与えました。(0.01秒)
レコード: 5  重複: 0  警告: 0

mysql> TRUNCATE TABLE t1;
クエリは成功しました。変更された行: 0 行に影響を与えました。(0.11秒)
```

## MySQL互換性

TiDBの`TRUNCATE`ステートメントは、MySQLと完全に互換性があります。互換性に違いがある場合は、[バグを報告してください](https://docs.pingcap.com/tidb/stable/support)。

## 関連項目

* [DROP TABLE](/sql-statements/sql-statement-drop-table.md)
* [DELETE](/sql-statements/sql-statement-delete.md)
* [CREATE TABLE](/sql-statements/sql-statement-create-table.md)
* [SHOW CREATE TABLE](/sql-statements/sql-statement-show-create-table.md)