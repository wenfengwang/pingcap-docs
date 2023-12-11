---
title: ALTER USER | TiDB SQLステートメントリファレンス
summary: TiDBデータベースのALTER USERの使用方法の概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-alter-user/','/docs/dev/reference/sql/statements/alter-user/']
---

# ALTER USER

このステートメントは、TiDB権限システム内の既存のユーザーを変更します。MySQL権限システムでは、ユーザーはユーザー名と接続元のホストの組み合わせです。したがって、IPアドレス`192.168.1.1`からのみ接続が可能な`'newuser2'@'192.168.1.1'`というユーザーを作成することが可能です。また、異なるホストからログインした場合の権限を異なるものにすることも可能です。

## 概要

```ebnf+diagram
AlterUserStmt ::=
    'ALTER' 'USER' IfExists (UserSpecList RequireClauseOpt ConnectionOptions PasswordOption LockOption AttributeOption | 'USER' '(' ')' 'IDENTIFIED' 'BY' AuthString) ResourceGroupNameOption

UserSpecList ::=
    UserSpec ( ',' UserSpec )*

UserSpec ::=
    Username AuthOption

Username ::=
    StringName ('@' StringName | singleAtIdentifier)? | 'CURRENT_USER' OptionalBraces

AuthOption ::=
    ( 'IDENTIFIED' ( 'BY' ( AuthString | 'PASSWORD' HashString ) | 'WITH' StringName ( 'BY' AuthString | 'AS' HashString )? ) )?

PasswordOption ::= ( 'PASSWORD' 'EXPIRE' ( 'DEFAULT' | 'NEVER' | 'INTERVAL' N 'DAY' )? | 'PASSWORD' 'HISTORY' ( 'DEFAULT' | N ) | 'PASSWORD' 'REUSE' 'INTERVAL' ( 'DEFAULT' | N 'DAY' ) | 'FAILED_LOGIN_ATTEMPTS' N | 'PASSWORD_LOCK_TIME' ( N | 'UNBOUNDED' ) )*

LockOption ::= ( 'ACCOUNT' 'LOCK' | 'ACCOUNT' 'UNLOCK' )?

AttributeOption ::= ( 'COMMENT' CommentString | 'ATTRIBUTE' AttributeString )?

ResourceGroupNameOption::= ( 'RESOURCE' 'GROUP' Identifier)?
```

## 例

```sql
mysql> CREATE USER 'newuser' IDENTIFIED BY 'newuserpassword';
Query OK, 1 row affected (0.01 sec)

mysql> SHOW CREATE USER 'newuser';
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| CREATE USER for newuser@%                                                                                                                                            |
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| CREATE USER 'newuser'@'%' IDENTIFIED WITH 'mysql_native_password' AS '*5806E04BBEE79E1899964C6A04D68BCA69B1A879' REQUIRE NONE PASSWORD EXPIRE DEFAULT ACCOUNT UNLOCK |
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

### ユーザーの基本情報を変更する

ユーザー`newuser`のパスワードを変更します：

```
mysql> ALTER USER 'newuser' IDENTIFIED BY 'newnewpassword';
Query OK, 0 rows affected (0.02 sec)

mysql> SHOW CREATE USER 'newuser';
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| CREATE USER for newuser@%                                                                                                                                            |
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| CREATE USER 'newuser'@'%' IDENTIFIED WITH 'mysql_native_password' AS '*FB8A1EA1353E8775CA836233E367FBDFCB37BE73' REQUIRE NONE PASSWORD EXPIRE DEFAULT ACCOUNT UNLOCK |
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

ユーザー`newuser`をロックします：

```sql
ALTER USER 'newuser' ACCOUNT LOCK;
```

```
Query OK, 0 rows affected (0.02 sec)
```

`newuser`の属性を変更します：

```sql
ALTER USER 'newuser' ATTRIBUTE '{"newAttr": "value", "deprecatedAttr": null}';
SELECT * FROM information_schema.user_attributes;
```

```sql
+-----------+------+--------------------------+
| USER      | HOST | ATTRIBUTE                |
+-----------+------+--------------------------+
| newuser   | %    | {"newAttr": "value"}     |
+-----------+------+--------------------------+
1 row in set (0.00 sec)
```

`ALTER USER ... COMMENT`を使用して、`newuser`のコメントを変更します：

```sql
ALTER USER 'newuser' COMMENT 'Here is the comment';
SELECT * FROM information_schema.user_attributes;
```

```sql
+-----------+------+--------------------------------------------------------+
| USER      | HOST | ATTRIBUTE                                              |
+-----------+------+--------------------------------------------------------+
| newuser   | %    | {"comment": "Here is the comment", "newAttr": "value"} |
+-----------+------+--------------------------------------------------------+
1 rows in set (0.00 sec)
```

`ALTER USER ... ATTRIBUTE`を使用して、`newuser`のコメントを削除します：

```sql
ALTER USER 'newuser' ATTRIBUTE '{"comment": null}';
SELECT * FROM information_schema.user_attributes;
```

```sql
+-----------+------+---------------------------+
| USER      | HOST | ATTRIBUTE                 |
+-----------+------+---------------------------+
| newuser   | %    | {"newAttr": "value"}      |
+-----------+------+---------------------------+
1 rows in set (0.00 sec)
```

`ALTER USER ... PASSWORD EXPIRE NEVER`を使用して、`newuser`の自動パスワード有効期限ポリシーを無期限に変更します：

```sql
ALTER USER 'newuser' PASSWORD EXPIRE NEVER;
```

```
Query OK, 0 rows affected (0.02 sec)
```

`ALTER USER ... PASSWORD REUSE INTERVAL ... DAY`を使用して、`newuser`のパスワード再利用ポリシーを過去90日間に使用されたパスワードの再利用を禁止するように変更します：

```sql
ALTER USER 'newuser' PASSWORD REUSE INTERVAL 90 DAY;
```

```
Query OK, 0 rows affected (0.02 sec)
```

### ユーザーにバインドされたリソースグループを変更する

`ALTER USER ... RESOURCE GROUP`を使用して、ユーザー`newuser`のリソースグループを`rg1`に変更します。

```sql
ALTER USER 'newuser' RESOURCE GROUP rg1;
```

```
Query OK, 0 rows affected (0.02 sec)
```

現在のユーザーにバインドされたリソースグループを表示します：

```sql
SELECT USER, JSON_EXTRACT(User_attributes, "$.resource_group") FROM mysql.user WHERE user = "newuser";
```

```
+---------+---------------------------------------------------+
| USER    | JSON_EXTRACT(User_attributes, "$.resource_group") |
+---------+---------------------------------------------------+
| newuser | "rg1"                                             |
+---------+---------------------------------------------------+
1 row in set (0.02 sec)
```

ユーザーをリソースグループにバインド解除します。つまり、ユーザーを`default`リソースグループにバインドします。

```sql
ALTER USER 'newuser' RESOURCE GROUP `default`;
SELECT USER, JSON_EXTRACT(User_attributes, "$.resource_group") FROM mysql.user WHERE user = "newuser";
```

```
+---------+---------------------------------------------------+
| USER    | JSON_EXTRACT(User_attributes, "$.resource_group") |
+---------+---------------------------------------------------+
| newuser | "default"                                         |
+---------+---------------------------------------------------+
1 row in set (0.02 sec)
```

## 関連項目

<CustomContent platform="tidb">

* [MySQLとのセキュリティ互換性](/security-compatibility-with-mysql.md)

</CustomContent>

* [CREATE USER](/sql-statements/sql-statement-create-user.md)
* [DROP USER](/sql-statements/sql-statement-drop-user.md)
* [SHOW CREATE USER](/sql-statements/sql-statement-show-create-user.md)