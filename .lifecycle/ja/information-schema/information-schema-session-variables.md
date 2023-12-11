---
title: セッション変数
summary: `SESSION_VARIABLES` INFORMATION_SCHEMA テーブルについて学びます。

# SESSION_VARIABLES

`SESSION_VARIABLES` テーブルはセッション変数に関する情報を提供します。このテーブルのデータは `SHOW SESSION VARIABLES` 文の結果に類似しています。

```sql
USE INFORMATION_SCHEMA;
DESC SESSION_VARIABLES;
```

出力は以下の通りです:

```sql
+----------------+---------------+------+------+---------+-------+
| Field          | Type          | Null | Key  | Default | Extra |
+----------------+---------------+------+------+---------+-------+
| VARIABLE_NAME  | varchar(64)   | YES  |      | NULL    |       |
| VARIABLE_VALUE | varchar(1024) | YES  |      | NULL    |       |
+----------------+---------------+------+------+---------+-------+
2 行が返されました (0.00 秒)
```

`SESSION_VARIABLES` テーブルの最初の 10 行をクエリします:

```sql
SELECT * FROM SESSION_VARIABLES ORDER BY variable_name LIMIT 10;
```

出力は以下の通りです:

```sql
+-----------------------------------+------------------+
| VARIABLE_NAME                     | VARIABLE_VALUE   |
+-----------------------------------+------------------+
| allow_auto_random_explicit_insert | OFF              |
| auto_increment_increment          | 1                |
| auto_increment_offset             | 1                |
| autocommit                        | ON               |
| automatic_sp_privileges           | 1                |
| avoid_temporal_upgrade            | OFF              |
| back_log                          | 80               |
| basedir                           | /usr/local/mysql |
| big_tables                        | OFF              |
| bind_address                      | *                |
+-----------------------------------+------------------+
10 行が返されました (0.00 秒)
```  

`SESSION_VARIABLES` テーブルの列の説明は以下の通りです:

* `VARIABLE_NAME`: データベース内のセッションレベルの変数の名前。
* `VARIABLE_VALUE`: データベース内のセッションレベルの変数の値。