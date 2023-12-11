---
title: SET PASSWORD | TiDB SQL ステートメントリファレンス
summary: TiDB データベースの SET PASSWORD の使用方法の概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-set-password/','/docs/dev/reference/sql/statements/set-password/']
---

# SET PASSWORD

このステートメントは、TiDB システムデータベース内のユーザーアカウントのユーザーパスワードを変更します。

## 概要

**SetStmt:**

![SetStmt](/media/sqlgram/SetStmt.png)

## 例

```sql
mysql> SET PASSWORD='test'; -- パスワードを変更する
Query OK, 0 rows affected (0.01 sec)

mysql> CREATE USER 'newuser' IDENTIFIED BY 'test';
Query OK, 1 row affected (0.00 sec)

mysql> SHOW CREATE USER 'newuser';
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| CREATE USER for newuser@%                                                                                                                                            |
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| CREATE USER 'newuser'@'%' IDENTIFIED WITH 'mysql_native_password' AS '*94BDCEBE19083CE2A1F959FD02F964C7AF4CFC29' REQUIRE NONE PASSWORD EXPIRE DEFAULT ACCOUNT UNLOCK |
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> SET PASSWORD FOR newuser = 'test';
Query OK, 0 rows affected (0.01 sec)

mysql> SHOW CREATE USER 'newuser';
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| CREATE USER for newuser@%                                                                                                                                            |
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| CREATE USER 'newuser'@'%' IDENTIFIED WITH 'mysql_native_password' AS '*94BDCEBE19083CE2A1F959FD02F964C7AF4CFC29' REQUIRE NONE PASSWORD EXPIRE DEFAULT ACCOUNT UNLOCK |
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> SET PASSWORD FOR newuser = PASSWORD('test'); -- 以前の MySQL リリースからの非推奨構文
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW CREATE USER 'newuser';
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| CREATE USER for newuser@%                                                                                                                                            |
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| CREATE USER 'newuser'@'%' IDENTIFIED WITH 'mysql_native_password' AS '*94BDCEBE19083CE2A1F959FD02F964C7AF4CFC29' REQUIRE NONE PASSWORD EXPIRE DEFAULT ACCOUNT UNLOCK |
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

## MySQL 互換性

TiDB での `SET PASSWORD` ステートメントは MySQL と完全に互換性があります。互換性に違いがある場合は、[バグを報告してください](https://docs.pingcap.com/tidb/stable/support)。

## 関連項目

* [CREATE USER](/sql-statements/sql-statement-create-user.md)

<CustomContent platform="tidb">

* [権限管理](/privilege-management.md)

</CustomContent>