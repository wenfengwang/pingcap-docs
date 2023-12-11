---
title: デフォルトのロールを設定する | TiDB SQL ステートメントリファレンス
summary: TiDB データベースで SET DEFAULT ROLE の使用方法の概要です。

---

# `SET DEFAULT ROLE`

このステートメントは特定のロールをユーザーにデフォルトで適用するためのものです。そのため、`SET ROLE <ロール名>` や `SET ROLE ALL` を実行せずに、自動的にそのロールに関連する権限を持つことができます。

## 概要

**SetDefaultRoleStmt:**

![SetDefaultRoleStmt](/media/sqlgram/SetDefaultRoleStmt.png)

**SetDefaultRoleOpt:**

![SetDefaultRoleOpt](/media/sqlgram/SetDefaultRoleOpt.png)

**RolenameList:**

![RolenameList](/media/sqlgram/RolenameList.png)

**UsernameList:**

![UsernameList](/media/sqlgram/UsernameList.png)

## 例

`root` ユーザーとして TiDB に接続します:

```shell
mysql -h 127.0.0.1 -P 4000 -u root
```

新しいロール `analyticsteam` と新しいユーザー `jennifer` を作成します:

```sql
CREATE ROLE analyticsteam;
Query OK, 0 rows affected (0.02 sec)

GRANT SELECT ON test.* TO analyticsteam;
Query OK, 0 rows affected (0.02 sec)

CREATE USER jennifer;
Query OK, 0 rows affected (0.01 sec)

GRANT analyticsteam TO jennifer;
Query OK, 0 rows affected (0.01 sec)
```

`jennifer` ユーザーとして TiDB に接続します:

```shell
mysql -h 127.0.0.1 -P 4000 -u jennifer
```

デフォルトで `jennifer` は `analyticsteam` ロールに関連する特権を使用するために、`SET ROLE analyticsteam` ステートメントを実行する必要があることに注意してください:

```sql
SHOW GRANTS;
+---------------------------------------------+
| Grants for User                             |
+---------------------------------------------+
| GRANT USAGE ON *.* TO 'jennifer'@'%'        |
| GRANT 'analyticsteam'@'%' TO 'jennifer'@'%' |
+---------------------------------------------+
2 rows in set (0.00 sec)

SHOW TABLES in test;
ERROR 1044 (42000): Access denied for user 'jennifer'@'%' to database 'test'
SET ROLE analyticsteam;
Query OK, 0 rows affected (0.00 sec)

SHOW GRANTS;
+---------------------------------------------+
| Grants for User                             |
+---------------------------------------------+
| GRANT USAGE ON *.* TO 'jennifer'@'%'        |
| GRANT Select ON test.* TO 'jennifer'@'%'    |
| GRANT 'analyticsteam'@'%' TO 'jennifer'@'%' |
+---------------------------------------------+
3 rows in set (0.00 sec)

SHOW TABLES IN test;
+----------------+
| Tables_in_test |
+----------------+
| t1             |
+----------------+
1 row in set (0.00 sec)
```

`root` ユーザーとして TiDB に接続します:

```shell
mysql -h 127.0.0.1 -P 4000 -u root
```

`SET DEFAULT ROLE` ステートメントを使用して、`analyticsteam` ロールを `jennifer` に関連付けることができます:

```sql
SET DEFAULT ROLE analyticsteam TO jennifer;
Query OK, 0 rows affected (0.02 sec)
```

`jennifer` ユーザーとして TiDB に接続します:

```shell
mysql -h 127.0.0.1 -P 4000 -u jennifer
```

これにより、`jennifer` ユーザーは `analyticsteam` ロールに関連する権限を持ち、`SET ROLE` ステートメントを実行する必要はありません:

```sql
SHOW GRANTS;
+---------------------------------------------+
| Grants for User                             |
+---------------------------------------------+
| GRANT USAGE ON *.* TO 'jennifer'@'%'        |
| GRANT Select ON test.* TO 'jennifer'@'%'    |
| GRANT 'analyticsteam'@'%' TO 'jennifer'@'%' |
+---------------------------------------------+
3 rows in set (0.00 sec)

SHOW TABLES IN test;
+----------------+
| Tables_in_test |
+----------------+
| t1             |
+----------------+
1 row in set (0.00 sec)
```

`SET DEFAULT ROLE` は関連するロールを自動的に `GRANT` しません。`jennifer` が許可されていないロールに対して `SET DEFAULT ROLE` を試みると、次のエラーが発生します:

```sql
SET DEFAULT ROLE analyticsteam TO jennifer;
ERROR 3530 (HY000): `analyticsteam`@`%` is is not granted to jennifer@%
```

## MySQL 互換性

TiDB における `SET DEFAULT ROLE` ステートメントは MySQL 8.0 でのロール機能と完全に互換性があります。互換性の違いがある場合は、[バグを報告](https://docs.pingcap.com/tidb/stable/support)してください。

## 関連項目

* [`CREATE ROLE`](/sql-statements/sql-statement-create-role.md)
* [`DROP ROLE`](/sql-statements/sql-statement-drop-role.md)
* [`GRANT <role>`](/sql-statements/sql-statement-grant-role.md)
* [`REVOKE <role>`](/sql-statements/sql-statement-revoke-role.md)
* [`SET ROLE`](/sql-statements/sql-statement-set-role.md)

<CustomContent platform="tidb">

* [ロールベースのアクセス制御](/role-based-access-control.md)

</CustomContent>