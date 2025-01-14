---
title: CREATE DATABASE | TiDB SQLステートメントリファレンス
summary: TiDBデータベースのCREATE DATABASEの使用方法の概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-create-database/','/docs/dev/reference/sql/statements/create-database/']
---

# CREATE DATABASE

このステートメントは、TiDBで新しいデータベースを作成します。 MySQLの用語「データベース」は、SQL標準におけるスキーマに最も近いものです。

## 概要

```ebnf+diagram
CreateDatabaseStmt ::=
    'CREATE' 'DATABASE' IfNotExists DBName DatabaseOptionListOpt

IfNotExists ::=
    ( 'IF' 'NOT' 'EXISTS' )?

DBName ::=
    Identifier

DatabaseOptionListOpt ::=
    DatabaseOptionList?

DatabaseOptionList ::=
    DatabaseOption ( ','? DatabaseOption )*

DatabaseOption ::=
    DefaultKwdOpt ( CharsetKw '='? CharsetName | 'COLLATE' '='? CollationName | 'ENCRYPTION' '='? EncryptionOpt )
|   DefaultKwdOpt PlacementPolicyOption

PlacementPolicyOption ::=
    "PLACEMENT" "POLICY" EqOpt PolicyName
|   "PLACEMENT" "POLICY" (EqOpt | "SET") "DEFAULT"
```

## 構文

`CREATE DATABASE` ステートメントは、データベースを作成し、データベースのデフォルトのプロパティ（デフォルトの文字セットおよび照合順序など）を指定するために使用されます。 `CREATE SCHEMA` は `CREATE DATABASE` の同義語です。

```sql
CREATE {DATABASE | SCHEMA} [IF NOT EXISTS] db_name
    [create_specification] ...

create_specification:
    [DEFAULT] CHARACTER SET [=] charset_name
  | [DEFAULT] COLLATE [=] collation_name
```

既存のデータベースを作成し、`IF NOT EXISTS` を指定しない場合、エラーが表示されます。

`create_specification` オプションは、データベース内で特定の `CHARACTER SET` および `COLLATE` を指定するために使用されます。現在、TiDBは一部の文字セットと照合順序のみをサポートしています。詳細は、[文字セットおよび照合順序のサポート](/character-set-and-collation.md)を参照してください。

## 例

```sql
mysql> CREATE DATABASE mynewdatabase;
Query OK, 0 rows affected (0.09 sec)

mysql> USE mynewdatabase;
Database changed
mysql> CREATE TABLE t1 (a int);
Query OK, 0 rows affected (0.11 sec)

mysql> SHOW TABLES;
+-------------------------+
| Tables_in_mynewdatabase |
+-------------------------+
| t1                      |
+-------------------------+
1 row in set (0.00 sec)
```

## MySQL互換性

TiDBの `CREATE DATABASE` ステートメントは、MySQLと完全に互換性があります。互換性の違いがある場合は、[バグを報告してください](https://docs.pingcap.com/tidb/stable/support)。

## 関連項目

* [USE](/sql-statements/sql-statement-use.md)
* [ALTER DATABASE](/sql-statements/sql-statement-alter-database.md)
* [DROP DATABASE](/sql-statements/sql-statement-drop-database.md)
* [SHOW DATABASES](/sql-statements/sql-statement-show-databases.md)