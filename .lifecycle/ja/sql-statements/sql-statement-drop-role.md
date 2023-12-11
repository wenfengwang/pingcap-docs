---
title: DROP ROLE | TiDB SQLステートメントリファレンス
summary: TiDBデータベースのDROP ROLEの使用方法の概要です。

# DROP ROLE

この文は、以前に `CREATE ROLE` で作成された役割を削除します。

## 概要

```ebnf+diagram
DropRoleStmt ::=
    'DROP' 'ROLE' ( 'IF' 'EXISTS' )? RolenameList

RolenameList ::=
    Rolename ( ',' Rolename )*
```

## 例

`root` ユーザーとしてTiDBに接続します：

```shell
mysql -h 127.0.0.1 -P 4000 -u root
```

新しい役割 `analyticsteam` と新しいユーザー `jennifer` を作成します：

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

`jennifer` ユーザーとしてTiDBに接続します：

```shell
mysql -h 127.0.0.1 -P 4000 -u jennifer
```

デフォルトでは、`jennifer` は `analyticsteam` 役割に関連付けられた特権を使用するために `SET ROLE analyticsteam` を実行する必要があります：

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

`root` ユーザーとしてTiDBに接続します：

```shell
mysql -h 127.0.0.1 -P 4000 -u root
```

`SET DEFAULT ROLE` 文を使用して、`analyticsteam` 役割を `jennifer` に関連付けることができます：

```sql
SET DEFAULT ROLE analyticsteam TO jennifer;
Query OK, 0 rows affected (0.02 sec)
```

`jennifer` ユーザーとしてTiDBに接続します：

```shell
mysql -h 127.0.0.1 -P 4000 -u jennifer
```

これにより、`jennifer` ユーザーは `analyticsteam` 役割に関連付けられた特権を持ち、`jennifer` が `SET ROLE` 文を実行する必要はありません：

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

`root` ユーザーとしてTiDBに接続します：

```shell
mysql -h 127.0.0.1 -P 4000 -u root
```

`analyticsteam` の役割を削除します：

```sql
DROP ROLE analyticsteam;
Query OK, 0 rows affected (0.02 sec)
```

`jennifer` はもはや `analyticsteam` のデフォルトの役割を関連付けておらず、`analyticsteam` の役割を設定することはできません。

`jennifer` ユーザーとしてTiDBに接続します：

```shell
mysql -h 127.0.0.1 -P 4000 -u jennifer
```

`jennifer` の特権を表示します：

```sql
SHOW GRANTS;
+--------------------------------------+
| Grants for User                      |
+--------------------------------------+
| GRANT USAGE ON *.* TO 'jennifer'@'%' |
+--------------------------------------+
1 row in set (0.00 sec)

SET ROLE analyticsteam;
ERROR 3530 (HY000): `analyticsteam`@`%` is is not granted to jennifer@%
```

## MySQL互換性

TiDBの `DROP ROLE` ステートメントは、MySQL 8.0のロール機能と完全に互換性があります。互換性に違いがある場合は、[バグを報告してください](https://docs.pingcap.com/tidb/stable/support)。

## 関連情報

* [`CREATE ROLE`](/sql-statements/sql-statement-create-role.md)
* [`GRANT <role>`](/sql-statements/sql-statement-grant-role.md)
* [`REVOKE <role>`](/sql-statements/sql-statement-revoke-role.md)
* [`SET ROLE`](/sql-statements/sql-statement-set-role.md)
* [`SET DEFAULT ROLE`](/sql-statements/sql-statement-set-default-role.md)

<CustomContent platform="tidb">

* [基于角色的访问控制](/role-based-access-control.md)

</CustomContent>