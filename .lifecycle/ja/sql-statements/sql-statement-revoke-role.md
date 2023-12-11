---
title: REVOKE <role> | TiDB SQLステートメントリファレンス
summary: TiDBデータベースのREVOKE <role>の使用概要です。

# `REVOKE <role>`

このステートメントは、指定されたユーザー（またはユーザーのリスト）から以前に割り当てられたロールを削除します。

## 概要

```ebnf+diagram
RevokeRoleStmt ::=
    'REVOKE' RolenameList 'FROM' UsernameList

RolenameList ::=
    Rolename ( ',' Rolename )*

UsernameList ::=
    Username ( ',' Username )*
```

## 例

`root`ユーザーとしてTiDBに接続します。

```shell
mysql -h 127.0.0.1 -P 4000 -u root
```

新しいロール`analyticsteam`と新しいユーザー`jennifer`を作成します。

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

`jennifer`ユーザーとしてTiDBに接続します。

```shell
mysql -h 127.0.0.1 -P 4000 -u jennifer
```

デフォルトで`jennifer`は`analyticsteam`ロールに関連付けられた特権を使用するために`SET ROLE analyticsteam`を実行する必要があることに注意してください。

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

`root`ユーザーとしてTiDBに接続します。

```shell
mysql -h 127.0.0.1 -P 4000 -u root
```

ステートメント`SET DEFAULT ROLE`を使用して、`analyticsteam`ロールを`jennifer`に関連付けることができます。

```sql
SET DEFAULT ROLE analyticsteam TO jennifer;
Query OK, 0 rows affected (0.02 sec)
```

`jennifer`ユーザーとしてTiDBに接続します。

```shell
mysql -h 127.0.0.1 -P 4000 -u jennifer
```

これにより、ユーザー`jennifer`は`analyticsteam`ロールに関連付けられた特権を持ち、`jennifer`は`SET ROLE`ステートメントを実行する必要はありません。

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

`root`ユーザーとしてTiDBに接続します。

```shell
mysql -h 127.0.0.1 -P 4000 -u root
```

`jennifer`から`analyticsteam`ロールを取り消します。

```sql
REVOKE analyticsteam FROM jennifer;
Query OK, 0 rows affected (0.01 sec)
```

`jennifer`ユーザーとしてTiDBに接続します。

```shell
mysql -h 127.0.0.1 -P 4000 -u jennifer
```

`jennifer`の特権を表示します。

```sql
SHOW GRANTS;
+--------------------------------------+
| Grants for User                      |
+--------------------------------------+
| GRANT USAGE ON *.* TO 'jennifer'@'%' |
+--------------------------------------+
1 row in set (0.00 sec)
```

## MySQL互換性

TiDBの`REVOKE <role>`ステートメントは、MySQL 8.0のロール機能と完全に互換性があります。互換性に違いがある場合は、[バグを報告してください](https://docs.pingcap.com/tidb/stable/support)。

## 関連項目

* [`CREATE ROLE`](/sql-statements/sql-statement-create-role.md)
* [`DROP ROLE`](/sql-statements/sql-statement-drop-role.md)
* [`GRANT <role>`](/sql-statements/sql-statement-grant-role.md)
* [`SET ROLE`](/sql-statements/sql-statement-set-role.md)
* [`SET DEFAULT ROLE`](/sql-statements/sql-statement-set-default-role.md)

<CustomContent platform="tidb">

* [ロールベースのアクセス制御](/role-based-access-control.md)

</CustomContent>
```