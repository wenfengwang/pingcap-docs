---
title: GRANT <privileges> | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースのGRANT <privileges>の使用方法の概要。
aliases: ['/docs/dev/sql-statements/sql-statement-grant-privileges/','/docs/dev/reference/sql/statements/grant-privileges/']
---

# `GRANT <privileges>`

このステートメントは、TiDBの事前存在ユーザーに特権を割り当てます。TiDBの特権システムはMySQLに従い、資格情報はデータベース/テーブルのパターンに基づいて割り当てられます。このステートメントを実行するには、`GRANT OPTION`特権と割り当てるすべての特権が必要です。

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
|    'ALTER' 'ROUTINE'?
|   'CREATE' ( 'USER' | 'TEMPORARY' 'TABLES' | 'VIEW' | 'ROLE' | 'ROUTINE' )?
|   'TRIGGER'
|   'DELETE'
|   'DROP' 'ROLE'?
|    'PROCESS'
|   'EXECUTE'
|   'INDEX'
|   'INSERT'
|   'SELECT'
|   'SUPER'
|   'SHOW' ( 'DATABASES' | 'VIEW' )
|   'UPDATE'
|    'GRANT' 'OPTION'
|    'REFERENCES'
|    'REPLICATION' ( 'SLAVE' | 'CLIENT' )
|    'USAGE'
|   'RELOAD'
|   'FILE'
|   'CONFIG'
|   'LOCK' 'TABLES'
|   'EVENT'
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
クエリが正常終了しました (0.02 秒)

mysql> GRANT ALL ON test.* TO 'newuser';
クエリが正常終了しました (0.03 秒)

mysql> SHOW GRANTS FOR 'newuser';
+-------------------------------------------------+
| Grants for newuser@%                            |
+-------------------------------------------------+
| GRANT USAGE ON *.* TO 'newuser'@'%'             |
| GRANT ALL PRIVILEGES ON test.* TO 'newuser'@'%' |
+-------------------------------------------------+
2 行が返されました (0.00 秒)
```

## MySQL互換性

* MySQLと同様に、`USAGE`特権はTiDBサーバーにログインする機能を示します。
* 現在、列レベルの特権はサポートされていません。
* MySQLと同様に、`NO_AUTO_CREATE_USER` SQLモードが存在しない場合、`GRANT`ステートメントはユーザーが存在しないときに自動的に空のパスワードを持つ新しいユーザーを作成します。このSQLモードを削除すると（デフォルトで有効になっています）、セキュリティリスクが発生します。
* TiDBでは、`GRANT <privileges>`ステートメントが正常に実行された後、実行結果は現在の接続で即座に有効になります。一方、[MySQLでは、一部の特権について、実行結果は後続の接続でのみ有効になります](https://dev.mysql.com/doc/refman/8.0/en/privilege-changes.html)。詳細は[TiDB #39356](https://github.com/pingcap/tidb/issues/39356)を参照してください。

## 関連項目

* [`GRANT <role>`](/sql-statements/sql-statement-grant-role.md)
* [`REVOKE <privileges>`](/sql-statements/sql-statement-revoke-privileges.md)
* [SHOW GRANTS](/sql-statements/sql-statement-show-grants.md)

<CustomContent platform="tidb">

* [特権管理](/privilege-management.md)

</CustomContent>