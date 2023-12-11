---
title: CREATE USER | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースのCREATE USERの使用方法の概要について 
aliases: ['/docs/dev/sql-statements/sql-statement-create-user/','/docs/dev/reference/sql/statements/create-user/']
---

# CREATE USER

このステートメントは、パスワードで指定された新しいユーザーを作成します。MySQL権限システムでは、ユーザーはユーザー名と接続元のホストの組み合わせです。したがって、IPアドレス`192.168.1.1`からのみ接続できるユーザー`'newuser2'@'192.168.1.1'`を作成することが可能です。また、異なるホストからログインするために異なる権限を持つ2人のユーザーが同じユーザー部分を持つことも可能です。

## 概要

```ebnf+diagram
CreateUserStmt ::=
    'CREATE' 'USER' IfNotExists UserSpecList RequireClauseOpt ConnectionOptions PasswordOption LockOption AttributeOption ResourceGroupNameOption

IfNotExists ::=
    ('IF' 'NOT' 'EXISTS')?

UserSpecList ::=
    UserSpec ( ',' UserSpec )*

UserSpec ::=
    Username AuthOption

AuthOption ::=
    ( 'IDENTIFIED' ( 'BY' ( AuthString | 'PASSWORD' HashString ) | 'WITH' StringName ( 'BY' AuthString | 'AS' HashString )? ) )?

StringName ::=
    stringLit
|   Identifier

PasswordOption ::= ( 'PASSWORD' 'EXPIRE' ( 'DEFAULT' | 'NEVER' | 'INTERVAL' N 'DAY' )? | 'PASSWORD' 'HISTORY' ( 'DEFAULT' | N ) | 'PASSWORD' 'REUSE' 'INTERVAL' ( 'DEFAULT' | N 'DAY' ) | 'FAILED_LOGIN_ATTEMPTS' N | 'PASSWORD_LOCK_TIME' ( N | 'UNBOUNDED' ) )*

LockOption ::= ( 'ACCOUNT' 'LOCK' | 'ACCOUNT' 'UNLOCK' )?

AttributeOption ::= ( 'COMMENT' CommentString | 'ATTRIBUTE' AttributeString )?

ResourceGroupNameOption::= ( 'RESOURCE' 'GROUP' Identifier)?
```

## 使用例

パスワード`newuserpassword`のユーザーを作成します。

```sql
mysql> CREATE USER 'newuser' IDENTIFIED BY 'newuserpassword';
Query OK, 1 row affected (0.04 sec)
```

`192.168.1.1`にのみログインできるユーザーを作成します。

```sql
mysql> CREATE USER 'newuser2'@'192.168.1.1' IDENTIFIED BY 'newuserpassword';
Query OK, 1 row affected (0.02 sec)
```

TLS接続を使用してログインするように強制されるユーザーを作成します。

```sql
CREATE USER 'newuser3'@'%' IDENTIFIED BY 'newuserpassword' REQUIRE SSL;
Query OK, 1 row affected (0.02 sec)
```

ログイン時にX.509証明書を使用するように要求されるユーザーを作成します。

```sql
CREATE USER 'newuser4'@'%' IDENTIFIED BY 'newuserpassword' REQUIRE ISSUER '/C=US/ST=California/L=San Francisco/O=PingCAP';
Query OK, 1 row affected (0.02 sec)
```

作成時にロックされるユーザーを作成します。

```sql
CREATE USER 'newuser5'@'%' ACCOUNT LOCK;
```

```
Query OK, 1 row affected (0.02 sec)
```

コメント付きのユーザーを作成します。

```sql
CREATE USER 'newuser6'@'%' COMMENT 'このユーザーはテスト用に作成されました';
SELECT * FROM information_schema.user_attributes;
```

```
+-----------+------+---------------------------------------------------+
| USER      | HOST | ATTRIBUTE                                         |
+-----------+------+---------------------------------------------------+
| newuser6  | %    | {"comment": "このユーザーはテスト用に作成されました"} |
+-----------+------+---------------------------------------------------+
1 rows in set (0.00 sec)
```

`email`属性付きのユーザーを作成します。

```sql
CREATE USER 'newuser7'@'%' ATTRIBUTE '{"email": "user@pingcap.com"}';
SELECT * FROM information_schema.user_attributes;
```

```sql
+-----------+------+---------------------------------------------------+
| USER      | HOST | ATTRIBUTE                                         |
+-----------+------+---------------------------------------------------+
| newuser7  | %    | {"email": "user@pingcap.com"} |
+-----------+------+---------------------------------------------------+
1 rows in set (0.00 sec)
```

直近の5つのパスワードを再利用できないようにするユーザーを作成します。

```sql
CREATE USER 'newuser8'@'%' PASSWORD HISTORY 5;
```

```
Query OK, 1 row affected (0.02 sec)
```

パスワードを手動で期限切れにするユーザーを作成します。

```sql
CREATE USER 'newuser9'@'%' PASSWORD EXPIRE;
```

```
Query OK, 1 row affected (0.02 sec)
```

リソースグループ'rg1'を使用するユーザーを作成します。

```sql
CREATE USER 'newuser7'@'%' RESOURCE GROUP rg1;
SELECT USER, HOST, USER_ATTRIBUTES FROM MYSQL.USER WHERE USER='newuser7';
```

```sql
+----------+------+---------------------------+
| USER     | HOST | USER_ATTRIBUTES           |
+----------+------+---------------------------+
| newuser7 | %    | {"resource_group": "rg1"} |
+----------+------+---------------------------+
1 rows in set (0.00 sec)
```

## MySQL互換性

次の`CREATE USER`オプションは、TiDBではまだサポートされていません。

* TiDBは、`WITH MAX_QUERIES_PER_HOUR`、`WITH MAX_UPDATES_PER_HOUR`、および`WITH MAX_USER_CONNECTIONS`オプションをサポートしていません。
* TiDBは`DEFAULT ROLE`オプションをサポートしていません。

## 関連情報

<CustomContent platform="tidb">

* [MySQLとのセキュリティ互換性](/security-compatibility-with-mysql.md)
* [権限管理](/privilege-management.md)

</CustomContent>

* [DROP USER](/sql-statements/sql-statement-drop-user.md)
* [SHOW CREATE USER](/sql-statements/sql-statement-show-create-user.md)
* [ALTER USER](/sql-statements/sql-statement-alter-user.md)