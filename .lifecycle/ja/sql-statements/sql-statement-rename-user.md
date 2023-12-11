---
title: ユーザーの名前変更
summary: TiDBデータベースでのRENAME USERの使用方法の概要。

# ユーザーの名前変更

`RENAME USER`は、既存のユーザーの名前を変更するために使用されます。

## 概要

```ebnf+diagram
RenameUserStmt ::=
    'RENAME' 'USER' UserToUser ( ',' UserToUser )*
UserToUser ::=
    Username 'TO' Username
Username ::=
    StringName ('@' StringName | singleAtIdentifier)? | 'CURRENT_USER' OptionalBraces
```

## 例

```sql
CREATE USER 'newuser' IDENTIFIED BY 'mypassword';
```

```sql
Query OK, 1 row affected (0.02 sec)
```

```sql
SHOW GRANTS FOR 'newuser';
```

```sql
+-------------------------------------+
| Grants for newuser@%                |
+-------------------------------------+
| GRANT USAGE ON *.* TO 'newuser'@'%' |
+-------------------------------------+
1 row in set (0.00 sec)
```

```sql
RENAME USER 'newuser' TO 'testuser';
```

```sql
Query OK, 0 rows affected (0.08 sec)
```

```sql
SHOW GRANTS FOR 'testuser';
```

```sql
+--------------------------------------+
| Grants for testuser@%                |
+--------------------------------------+
| GRANT USAGE ON *.* TO 'testuser'@'%' |
+--------------------------------------+
1 row in set (0.00 sec)
```

```sql
SHOW GRANTS FOR 'newuser';
```

```sql
ERROR 1141 (42000): ホスト '%' のユーザー 'newuser' には定義された権限がありません
```

## MySQL互換性

`RENAME USER`はMySQLと完全に互換性があると期待されています。互換性の違いを見つけた場合は、[バグを報告](https://docs.pingcap.com/tidb/stable/support)してください。

## 関連情報

* [CREATE USER](/sql-statements/sql-statement-create-user.md)
* [SHOW GRANTS](/sql-statements/sql-statement-show-grants.md)
* [DROP USER](/sql-statements/sql-statement-drop-user.md)