---
title: SHOW GRANTS | TiDB SQLステートメントリファレンス
summary: TiDBデータベースのSHOW GRANTSの使用法についての概要。
aliases: ['/docs/dev/sql-statements/sql-statement-show-grants/','/docs/dev/reference/sql/statements/show-grants/']
---

# SHOW GRANTS

このステートメントは、ユーザーに関連付けられた特権の一覧を表示します。MySQLと同様に、`USAGE`特権はTiDBへのログイン権限を示します。

## 概要

**ShowGrantsStmt:**

![ShowGrantsStmt](/media/sqlgram/ShowGrantsStmt.png)

**Username:**

![Username](/media/sqlgram/Username.png)

**UsingRoles:**

![UsingRoles](/media/sqlgram/UsingRoles.png)

**RolenameList:**

![RolenameList](/media/sqlgram/RolenameList.png)

**Rolename:**

![Rolename](/media/sqlgram/Rolename.png)

## 例

```sql
mysql> SHOW GRANTS;
+-------------------------------------------+
| Grants for User                           |
+-------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' |
+-------------------------------------------+
1 row in set (0.00 sec)

mysql> SHOW GRANTS FOR 'u1';
ERROR 1141 (42000): There is no such grant defined for user 'u1' on host '%'
mysql> CREATE USER u1;
Query OK, 1 row affected (0.04 sec)

mysql> GRANT SELECT ON test.* TO u1;
Query OK, 0 rows affected (0.04 sec)

mysql> SHOW GRANTS FOR u1;
+------------------------------------+
| Grants for u1@%                    |
+------------------------------------+
| GRANT USAGE ON *.* TO 'u1'@'%'     |
| GRANT Select ON test.* TO 'u1'@'%' |
+------------------------------------+
2 rows in set (0.00 sec)
```

## MySQL互換性

TiDBの`SHOW GRANTS`ステートメントはMySQLと完全に互換性があります。互換性の違いがある場合は、[バグを報告してください](https://docs.pingcap.com/tidb/stable/support)。

## 関連情報

* [SHOW CREATE USER](/sql-statements/sql-statement-show-create-user.md)
* [GRANT](/sql-statements/sql-statement-grant-privileges.md)