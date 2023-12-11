---
title: DROP USER | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースのDROP USERの使用方法の概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-drop-user/','/docs/dev/reference/sql/statements/drop-user/']
---

# DROP USER

このステートメントは、TiDBシステムデータベースからユーザーを削除します。オプションのキーワード`IF EXISTS`は、ユーザーが存在しない場合にエラーを抑制するために使用できます。このステートメントには`CREATE USER`権限が必要です。

## 概要

```ebnf+diagram
DropUserStmt ::=
    'DROP' 'USER' ( 'IF' 'EXISTS' )? UsernameList

Username ::=
    StringName ('@' StringName | singleAtIdentifier)? | 'CURRENT_USER' OptionalBraces
```

## 例

```sql
mysql> DROP USER idontexist;
ERROR 1396 (HY000): Operation DROP USER failed for idontexist@%

mysql> DROP USER IF EXISTS 'idontexist';
Query OK, 0 rows affected (0.01 sec)

mysql> CREATE USER 'newuser' IDENTIFIED BY 'mypassword';
Query OK, 1 row affected (0.02 sec)

mysql> GRANT ALL ON test.* TO 'newuser';
Query OK, 0 rows affected (0.03 sec)

mysql> SHOW GRANTS FOR 'newuser';
+-------------------------------------------------+
| Grants for newuser@%                            |
+-------------------------------------------------+
| GRANT USAGE ON *.* TO 'newuser'@'%'             |
| GRANT ALL PRIVILEGES ON test.* TO 'newuser'@'%' |
+-------------------------------------------------+
2 rows in set (0.00 sec)

mysql> REVOKE ALL ON test.* FROM 'newuser';
Query OK, 0 rows affected (0.03 sec)

mysql> SHOW GRANTS FOR 'newuser';
+-------------------------------------+
| Grants for newuser@%                |
+-------------------------------------+
| GRANT USAGE ON *.* TO 'newuser'@'%' |
+-------------------------------------+
1 row in set (0.00 sec)

mysql> DROP USER 'newuser';
Query OK, 0 rows affected (0.14 sec)

mysql> SHOW GRANTS FOR 'newuser';
ERROR 1141 (42000): There is no such grant defined for user 'newuser' on host '%'
```

## MySQL互換性

* `IF EXISTS`で存在しないユーザーを削除した場合、TiDBでは警告が生成されません。 [Issue #10196](https://github.com/pingcap/tidb/issues/10196).

## 関連項目

* [CREATE USER](/sql-statements/sql-statement-create-user.md)
* [ALTER USER](/sql-statements/sql-statement-alter-user.md)
* [SHOW CREATE USER](/sql-statements/sql-statement-show-create-user.md)

<CustomContent platform="tidb">

* [権限管理](/privilege-management.md)

</CustomContent>