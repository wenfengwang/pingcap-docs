---
title: ALTER DATABASE | TiDB SQLステートメントリファレンス
summary: TiDBデータベースのALTER DATABASEの使用方法についての概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-alter-database/','/docs/dev/reference/sql/statements/alter-database/']
---

# ALTER DATABASE

`ALTER DATABASE`は、現在のデータベースのデフォルトの文字セットと照合順序を指定または変更するために使用されます。`ALTER SCHEMA`は`ALTER DATABASE`と同じ効果があります。

## 構文

```ebnf+diagram
AlterDatabaseStmt ::=
    'ALTER' 'DATABASE' DBName? DatabaseOptionList

DatabaseOption ::=
    DefaultKwdOpt ( CharsetKw '='? CharsetName | 'COLLATE' '='? CollationName | 'ENCRYPTION' '='? EncryptionOpt )
```

## 例

テストデータベースのスキーマをutf8mb4文字セットを使用するように変更します:

{{< copyable "sql" >}}

```sql
ALTER DATABASE test DEFAULT CHARACTER SET = utf8mb4;
```

```sql
Query OK, 0 rows affected (0.00 sec)
```

現在、TiDBは一部の文字セットと照合順序のみをサポートしています。詳細は[文字セットと照合順序のサポート](/character-set-and-collation.md)を参照してください。

## MySQL互換性

TiDBの`ALTER DATABASE`ステートメントはMySQLと完全に互換性があります。互換性の違いがある場合は、[バグを報告してください](https://docs.pingcap.com/tidb/stable/support)。

## 関連項目

* [CREATE DATABASE](/sql-statements/sql-statement-create-database.md)
* [SHOW DATABASES](/sql-statements/sql-statement-show-databases.md)