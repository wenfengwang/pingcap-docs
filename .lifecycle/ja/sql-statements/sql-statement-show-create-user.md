---
title: SHOW CREATE USER | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースのSHOW CREATE USERの使用方法の概要。
aliases: ['/docs/dev/sql-statements/sql-statement-show-create-user/','/docs/dev/reference/sql/statements/show-create-user/']
---

# SHOW CREATE USER

このステートメントは、`CREATE USER`構文を使用してユーザーを再作成する方法を示します。

## 概要

**ShowCreateUserStmt:**

![ShowCreateUserStmt](/media/sqlgram/ShowCreateUserStmt.png)

**ユーザー名:**

![ユーザー名](/media/sqlgram/Username.png)

## 例

```sql
mysql> SHOW CREATE USER 'root';
+--------------------------------------------------------------------------------------------------------------------------+
| CREATE USER for root@%                                                                                                   |
+--------------------------------------------------------------------------------------------------------------------------+
| CREATE USER 'root'@'%' IDENTIFIED WITH 'mysql_native_password' AS '' REQUIRE NONE PASSWORD EXPIRE DEFAULT ACCOUNT UNLOCK |
+--------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> SHOW GRANTS FOR 'root';
+-------------------------------------------+
| Grants for root@%                         |
+-------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' |
+-------------------------------------------+
1 row in set (0.00 sec)
```

## MySQL compatibility

* `SHOW CREATE USER`の出力はMySQLに合わせて設計されていますが、`CREATE`オプションのいくつかはTiDBでまだサポートされていません。まだサポートされていないオプションはパースされますが無視されます。詳細については[security compatibility]を参照してください。

## 関連項目

* [CREATE USER](/sql-statements/sql-statement-create-user.md)
* [SHOW GRANTS](/sql-statements/sql-statement-show-grants.md)
* [DROP USER](/sql-statements/sql-statement-drop-user.md)