---
title: CREATE ROLE | TiDB SQLステートメントリファレンス
summary: TiDBデータベースのCREATE ROLEの使用法の概要。
---

# CREATE ROLE

このステートメントは、ユーザーに割り当てることができる、ロールベースのアクセス制御の一部として新しいロールを作成します。

## 概要

```ebnf+diagram
CreateRoleStmt ::=
    'CREATE' 'ROLE' IfNotExists RoleSpec (',' RoleSpec)*

IfNotExists ::=
    ('IF' 'NOT' 'EXISTS')?

RoleSpec ::=
    Rolename
```

## 例

`root`ユーザーとしてTiDBに接続：

```shell
mysql -h 127.0.0.1 -P 4000 -u root
```

新しいロール`analyticsteam`と新しいユーザー`jennifer`を作成：

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

`jennifer`ユーザーとしてTiDBに接続：

```shell
mysql -h 127.0.0.1 -P 4000 -u jennifer
```

デフォルトでは`jennifer`は`analyticsteam`ロールに関連付けられた特権を使用するために`SET ROLE analyticsteam`を実行する必要があります：

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
| GRANT SELECT ON test.* TO 'jennifer'@'%'    |
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

`root`ユーザーとしてTiDBに接続：

```shell
mysql -h 127.0.0.1 -P 4000 -u root
```

ステートメント`SET DEFAULT ROLE`を使用して、ロール`analyticsteam`を`jennifer`に関連付けることができます：

```sql
SET DEFAULT ROLE analyticsteam TO jennifer;
Query OK, 0 rows affected (0.02 sec)
```

`jennifer`ユーザーとしてTiDBに接続：

```shell
mysql -h 127.0.0.1 -P 4000 -u jennifer
```

これにより、`jennifer`ユーザーは`analyticsteam`ロールに関連付けられた特権を持ち、`jennifer`は`SET ROLE`ステートメントを実行する必要はありません：

```sql
SHOW GRANTS;
+---------------------------------------------+
| Grants for User                             |
+---------------------------------------------+
| GRANT USAGE ON *.* TO 'jennifer'@'%'        |
| GRANT SELECT ON test.* TO 'jennifer'@'%'    |
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

## MySQL互換性

TiDBの`CREATE ROLE`ステートメントは、MySQL 8.0のロール機能と完全に互換性があります。互換性の違いがある場合は、[バグを報告してください](https://docs.pingcap.com/tidb/stable/support)。

## 関連項目

* [`DROP ROLE`](/sql-statements/sql-statement-drop-role.md)
* [`GRANT <role>`](/sql-statements/sql-statement-grant-role.md)
* [`REVOKE <role>`](/sql-statements/sql-statement-revoke-role.md)
* [`SET ROLE`](/sql-statements/sql-statement-set-role.md)
* [`SET DEFAULT ROLE`](/sql-statements/sql-statement-set-default-role.md)

<CustomContent platform="tidb">

* [ロールベースアクセス制御](/role-based-access-control.md)

</CustomContent>