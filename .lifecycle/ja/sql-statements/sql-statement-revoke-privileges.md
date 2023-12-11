---
title: REVOKE <権限> | TiDB SQLステートメントリファレンス
summary: TiDBデータベースのREVOKE <権限>の使用方法の概要。
aliases: ['/docs/dev/sql-statements/sql-statement-revoke-privileges/','/docs/dev/reference/sql/statements/revoke-privileges/']
---

# `REVOKE <権限>`

このステートメントは、既存のユーザーから権限を削除します。このステートメントを実行するには、`GRANT OPTION`権限と取り消すすべての権限が必要です。

## 概要

```ebnf+diagram
GrantStmt ::=
    'GRANT' PrivElemList 'ON' ObjectType PrivLevel 'TO' UserSpecList RequireClauseOpt WithGrantOptionOpt

PrivElemList ::=
    PrivElem ( ',' PrivElem )*

PrivElem ::=
    PrivType ( '(' ColumnNameList ')' )?

PrivType ::=
    'ALL' 'PRIVILEGES'?
|   'ALTER' 'ROUTINE'?
|   'CREATE' ( 'USER' | 'TEMPORARY' 'TABLES' | 'VIEW' | 'ROLE' | 'ROUTINE' )?
|    'TRIGGER'
|   'DELETE'
|    'DROP' 'ROLE'?
|    'PROCESS'
|    'EXECUTE'
|   'INDEX'
|   'INSERT'
|   'SELECT'
|   'SUPER'
|    'SHOW' ( 'DATABASES' | 'VIEW' )
|   'UPDATE'
|   'GRANT' 'OPTION'
|   'REFERENCES'
|   'REPLICATION' ( 'SLAVE' | 'CLIENT' )
|   'USAGE'
|    'RELOAD'
|   'FILE'
|   'CONFIG'
|   'LOCK' 'TABLES'
|    'EVENT'
|   'SHUTDOWN'

ObjectType ::=
    'TABLE'?

PrivLevel ::=
    '*' ( '.' '*' )?
|    Identifier ( '.' ( '*' | Identifier ) )?

UserSpecList ::=
    UserSpec ( ',' UserSpec )*
```

## 例

```sql
mysql> CREATE USER 'newuser' IDENTIFIED BY 'mypassword';
クエリが実行されました。1行が変更されました（0.02秒）

mysql> GRANT ALL ON test.* TO 'newuser';
クエリが実行されました。0行が変更されました（0.03秒）

mysql> SHOW GRANTS FOR 'newuser';
+-------------------------------------------------+
| Grants for newuser@%                            |
+-------------------------------------------------+
| GRANT USAGE ON *.* TO 'newuser'@'%'             |
| GRANT ALL PRIVILEGES ON test.* TO 'newuser'@'%' |
+-------------------------------------------------+
2行がセットにあります（0.00秒）

mysql> REVOKE ALL ON test.* FROM 'newuser';
クエリが実行されました。0行が変更されました（0.03秒）

mysql> SHOW GRANTS FOR 'newuser';
+-------------------------------------+
| Grants for newuser@%                |
+-------------------------------------+
| GRANT USAGE ON *.* TO 'newuser'@'%' |
+-------------------------------------+
1行がセットにあります（0.00秒）

mysql> DROP USER 'newuser';
クエリが実行されました。0行が変更されました（0.14秒）

mysql> SHOW GRANTS FOR 'newuser';
ERROR 1141 (42000): ホスト'%​​'のユーザー'newuser'に定義された権限はありません
```

## MySQL互換性

* TiDBでは、`REVOKE <権限>`ステートメントが正常に実行された後、実行結果は現在の接続にすぐに反映されます。一方、[MySQLでは、一部の権限については、実行結果が以降の接続でのみ有効になります](https://dev.mysql.com/doc/refman/8.0/en/privilege-changes.html)。詳細は[TiDB#39356](https://github.com/pingcap/tidb/issues/39356)を参照してください。

## 関連項目

* [`GRANT <権限>`](/sql-statements/sql-statement-grant-privileges.md)
* [SHOW GRANTS](/sql-statements/sql-statement-show-grants.md)

<CustomContent platform="tidb">

* [権限管理](/privilege-management.md)

</CustomContent>