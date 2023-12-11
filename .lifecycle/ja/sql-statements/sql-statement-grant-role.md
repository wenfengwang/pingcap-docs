---
title: GRANT <role> | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースでのGRANT <role>の使用方法の概要です。

# `GRANT <role>`

以前に作成したロールを既存のユーザーに割り当てます。ユーザーは、その後`SET ROLE <ロール名>`ステートメントを使用してロールの権限を仮定するか、`SET ROLE ALL`を使用して割り当てられたすべてのロールを仮定できます。

## 概要

```ebnf+diagram
GrantRoleStmt ::=
    'GRANT' RolenameList 'TO' UsernameList

RolenameList ::=
    Rolename ( ',' Rolename )*

UsernameList ::=
    Username ( ',' Username )*
```

## 例

`root`ユーザーとしてTiDBに接続します：

```shell
mysql -h 127.0.0.1 -P 4000 -u root
```

新しいロール `analyticsteam` と新しいユーザー `jennifer` を作成します：

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

`jennifer`ユーザーとしてTiDBに接続します：

```shell
mysql -h 127.0.0.1 -P 4000 -u jennifer
```

デフォルトでは、`jennifer`ユーザーは`SET ROLE analyticsteam`を実行する必要があります。これにより、`analyticsteam`ロールに関連付けられた特権を使用できます：

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

`root`ユーザーとしてTiDBに接続します：

```shell
mysql -h 127.0.0.1 -P 4000 -u root
```

ステートメント`SET DEFAULT ROLE`を使用して、ロール`analyticsteam`を`jennifer`に関連付けることができます：

```sql
SET DEFAULT ROLE analyticsteam TO jennifer;
Query OK, 0 rows affected (0.02 sec)
```

`jennifer`ユーザーとしてTiDBに接続します：

```shell
mysql -h 127.0.0.1 -P 4000 -u jennifer
```

この後、ユーザー`jennifer`はロール`analyticsteam`に関連付けられた特権を持ち、`jennifer`は`SET ROLE`ステートメントを実行する必要はありません：

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

## MySQLの互換性

TiDBの`GRANT <role>`ステートメントは、MySQL 8.0のロール機能と完全に互換性があります。互換性の違いがある場合は、[バグを報告してください](https://docs.pingcap.com/tidb/stable/support)。

## 関連項目

* [`GRANT <privileges>`](/sql-statements/sql-statement-grant-privileges.md)
* [`CREATE ROLE`](/sql-statements/sql-statement-create-role.md)
* [`DROP ROLE`](/sql-statements/sql-statement-drop-role.md)
* [`REVOKE <role>`](/sql-statements/sql-statement-revoke-role.md)
* [`SET ROLE`](/sql-statements/sql-statement-set-role.md)
* [`SET DEFAULT ROLE`](/sql-statements/sql-statement-set-default-role.md)

<CustomContent platform="tidb">

* [ロールベースのアクセス制御](/role-based-access-control.md)

</CustomContent>