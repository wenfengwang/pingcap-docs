---
title: SET [NAMES|CHARACTER SET] |  TiDB SQLステートメントリファレンス
summary: TiDBデータベースでのSET [NAMES|CHARACTER SET]の使用法の概要。
aliases: ['/docs/dev/sql-statements/sql-statement-set-names/','/docs/dev/reference/sql/statements/set-names/']
---

# SET [NAMES|CHARACTER SET]

ステートメント`SET NAMES`、`SET CHARACTER SET`、`SET CHARSET`は、現在の接続の変数`character_set_client`、`character_set_results`、`character_set_connection`を変更します。

## 概要

**SetNamesStmt:**

![SetNamesStmt](/media/sqlgram/SetNamesStmt.png)

**VariableAssignmentList:**

![VariableAssignmentList](/media/sqlgram/VariableAssignmentList.png)

**VariableAssignment:**

![VariableAssignment](/media/sqlgram/VariableAssignment.png)

**CharsetName:**

![CharsetName](/media/sqlgram/CharsetName.png)

**StringName:**

![StringName](/media/sqlgram/StringName.png)

**CharsetKw:**

![CharsetKw](/media/sqlgram/CharsetKw.png)

**CharsetNameOrDefault:**

![CharsetNameOrDefault](/media/sqlgram/CharsetNameOrDefault.png)

## 例

```sql
mysql> SHOW VARIABLES LIKE 'character_set%';
+--------------------------+--------------------------------------------------------+
| Variable_name            | Value                                                  |
+--------------------------+--------------------------------------------------------+
| character_sets_dir       | /usr/local/mysql-5.6.25-osx10.8-x86_64/share/charsets/ |
| character_set_connection | utf8mb4                                                |
| character_set_system     | utf8                                                   |
| character_set_results    | utf8mb4                                                |
| character_set_client     | utf8mb4                                                |
| character_set_database   | utf8mb4                                                |
| character_set_filesystem | binary                                                 |
| character_set_server     | utf8mb4                                                |
+--------------------------+--------------------------------------------------------+
8 rows in set (0.01 sec)

mysql> SET NAMES utf8;
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW VARIABLES LIKE 'character_set%';
+--------------------------+--------------------------------------------------------+
                    | Variable_name            | Value                                                  |
                    +--------------------------+--------------------------------------------------------+
| character_sets_dir   | /usr/local/mysql-5.6.25-osx10.8-x86_64/share/charsets/ |
| character_set_connection | utf8                                                   |
| character_set_system     | utf8                                                   |
| character_set_results | utf8                                                   |
| character_set_client  | utf8                                                   |
| character_set_server   | utf8mb4                                                 |
| character_set_database | utf8mb4                                                 |
| character_set_filesystem| binary                                           |
                    +--------------------------+--------------------------------------------------------+
8 rows in set (0.01 sec)

mysql> SET CHARACTER SET utf8mb4;
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW VARIABLES LIKE 'character_set%';
        +--------------------------+--------------------------------------------------------+
        | Variable_name            | Value                                                  |
        +--------------------------+--------------------------------------------------------+
        | character_set_connection | utf8mb4                                                |
        | character_set_system     | utf8                                                   |
        | character_set_results    | utf8mb4                                                |
        | character_set_client     | utf8mb4                                                |
        | character_sets_dir       | /usr/local/mysql-5.6.25-osx10.8-x86_64/share/charsets/ |
        | character_set_database   | utf8mb4                                                |
        | character_set_filesystem | binary                                                 |
        | character_set_server     | utf8mb4                                                |
        +--------------------------+--------------------------------------------------------+
        8 rows in set (0.00 sec)
```

## MySQL互換性

TiDBの`SET [NAMES|CHARACTER SET]`ステートメントはMySQLと完全に互換性があります。互換性の違いが見つかった場合は、[バグを報告](https://docs.pingcap.com/tidb/stable/support)してください。

## 関連項目

* [SHOW \[GLOBAL|SESSION\] VARIABLES](/sql-statements/sql-statement-show-variables.md)
* [`SET <variable>`](/sql-statements/sql-statement-set-variable.md)
* [文字セットと照合順序のサポート](/character-set-and-collation.md)