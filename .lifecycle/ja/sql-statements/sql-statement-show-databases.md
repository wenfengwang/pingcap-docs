---
title: SHOW DATABASES | TiDB SQLステートメントリファレンス
summary: TiDBデータベースのSHOW DATABASESの使用概要。
aliases: ['/docs/dev/sql-statements/sql-statement-show-databases/','/docs/dev/reference/sql/statements/show-databases/']
---

# SHOW DATABASES

このステートメントは、現在のユーザーが権限を持つデータベースの一覧を表示します。現在のユーザーがアクセス権を持っていないデータベースは一覧から非表示になります。`information_schema`データベースは常にデータベースの一覧の先頭に表示されます。

`SHOW SCHEMAS`はこのステートメントのエイリアスです。

## 概要

**ShowDatabasesStmt:**

![ShowDatabasesStmt](/media/sqlgram/ShowDatabasesStmt.png)

**ShowLikeOrWhereOpt:**

![ShowLikeOrWhereOpt](/media/sqlgram/ShowLikeOrWhereOpt.png)

## 例

```sql
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| INFORMATION_SCHEMA |
| PERFORMANCE_SCHEMA |
| mysql              |
| test               |
+--------------------+
4 rows in set (0.00 sec)

mysql> CREATE DATABASE mynewdb;
Query OK, 0 rows affected (0.10 sec)

mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| INFORMATION_SCHEMA |
| PERFORMANCE_SCHEMA |
| mynewdb            |
| mysql              |
| test               |
+--------------------+
5 rows in set (0.00 sec)
```

## MySQL互換性

TiDBの`SHOW DATABASES`ステートメントはMySQLと完全に互換性があります。互換性に関する問題がある場合は、[バグを報告してください](https://docs.pingcap.com/tidb/stable/support)。

## 関連項目

* [SHOW SCHEMAS](/sql-statements/sql-statement-show-schemas.md)
* [DROP DATABASE](/sql-statements/sql-statement-drop-database.md)
* [CREATE DATABASE](/sql-statements/sql-statement-create-database.md)