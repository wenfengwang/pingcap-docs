---
title: DROP DATABASE | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースに対するDROP DATABASEの使用法の概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-drop-database/','/docs/dev/reference/sql/statements/drop-database/']
---

# DROP DATABASE

`DROP DATABASE` ステートメントは指定されたデータベーススキーマとその中で作成されたすべてのテーブルとビューを永久に削除します。削除されたデータベースに関連付けられているユーザー権限は影響を受けません。

## 構文

```ebnf+diagram
DropDatabaseStmt ::=
    'DROP' 'DATABASE' IfExists DBName

IfExists ::= ( 'IF' 'EXISTS' )?
```

## 例

```sql
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| INFORMATION_SCHEMA |
| PERFORMANCE_SCHEMA |
| mysql              |
| test               |
+--------------------+
4 rows in set (0.00 sec)

mysql> DROP DATABASE test;
Query OK, 0 rows affected (0.25 sec)

mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| INFORMATION_SCHEMA |
| PERFORMANCE_SCHEMA |
| mysql              |
+--------------------+
3 rows in set (0.00 sec)
```

## MySQL互換性

TiDBの`DROP DATABASE` ステートメントはMySQLと完全に互換性があります。互換性の違いがある場合は、[バグを報告してください](https://docs.pingcap.com/tidb/stable/support)。

## 関連情報

* [CREATE DATABASE](/sql-statements/sql-statement-create-database.md)
* [ALTER DATABASE](/sql-statements/sql-statement-alter-database.md)