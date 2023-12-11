---
title: データベースの作成を表示
summary: TiDBデータベースでのSHOW CREATE DATABASEの使用の概要。

# データベースの作成を表示

`SHOW CREATE DATABASE`は、既存のデータベースを再作成するための正確なSQLステートメントを表示するために使用されます。 `SHOW CREATE SCHEMA`はそれの同義語です。

## 概要

**ShowCreateDatabaseStmt:**

```ebnf+diagram
ShowCreateDatabaseStmt ::=
    "SHOW" "CREATE" "DATABASE" | "SCHEMA" ("IF" "NOT" "EXISTS")? DBName
```

## 例

```sql
CREATE DATABASE test;
```

```sql
Query OK, 0 rows affected (0.12 sec)
```

```sql
SHOW CREATE DATABASE test;
```

```sql
+----------+------------------------------------------------------------------+
| Database | Create Database                                                  |
+----------+------------------------------------------------------------------+
| test     | CREATE DATABASE `test` /*!40100 DEFAULT CHARACTER SET utf8mb4 */ |
+----------+------------------------------------------------------------------+
1 row in set (0.00 sec)
```

```sql
SHOW CREATE SCHEMA IF NOT EXISTS test;
```

```sql
+----------+-------------------------------------------------------------------------------------------+
| Database | Create Database                                                                           |
+----------+-------------------------------------------------------------------------------------------+
| test     | CREATE DATABASE /*!32312 IF NOT EXISTS*/ `test` /*!40100 DEFAULT CHARACTER SET utf8mb4 */ |
+----------+-------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

## MySQL互換性

`SHOW CREATE DATABASE`は、MySQLと完全に互換性があると期待されます。互換性の違いが見つかった場合、[バグを報告](https://docs.pingcap.com/tidb/stable/support)することができます。

## 関連項目

* [CREATE TABLE](/sql-statements/sql-statement-create-table.md)
* [DROP TABLE](/sql-statements/sql-statement-drop-table.md)
* [SHOW TABLES](/sql-statements/sql-statement-show-tables.md)
* [SHOW COLUMNS FROM](/sql-statements/sql-statement-show-columns-from.md)