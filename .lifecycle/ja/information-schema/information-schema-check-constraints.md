---
title: CHECK_CONSTRAINTS
summary: `CHECK_CONSTRAINTS` INFORMATION_SCHEMAテーブルの情報を学ぶ。

# CHECK\_CONSTRAINTS

`CHECK_CONSTRAINTS`テーブルはテーブル上の[`CHECK`制約](/constraints.md#check)に関する情報を提供します。

```sql
USE INFORMATION_SCHEMA;
DESC CHECK_CONSTRAINTS;
```

出力は以下の通りです。

```sql
+--------------------+-------------+------+-----+---------+-------+
| Field              | Type        | Null | Key | Default | Extra |
+--------------------+-------------+------+-----+---------+-------+
| CONSTRAINT_CATALOG | varchar(64) | NO   |     | NULL    |       |
| CONSTRAINT_SCHEMA  | varchar(64) | NO   |     | NULL    |       |
| CONSTRAINT_NAME    | varchar(64) | NO   |     | NULL    |       |
| CHECK_CLAUSE       | longtext    | NO   |     | NULL    |       |
+--------------------+-------------+------+-----+---------+-------+
4 rows in set (0.00 sec)
```

以下の例は、`CREATE TABLE`ステートメントを使用して`CHECK`制約を追加します。

```sql
CREATE TABLE test.t1 (id INT PRIMARY KEY, CHECK (id%2 = 0));
SELECT * FROM CHECK_CONSTRAINTS\G
```

出力は以下の通りです。

```sql
*************************** 1. row ***************************
CONSTRAINT_CATALOG: def
 CONSTRAINT_SCHEMA: test
   CONSTRAINT_NAME: t1_chk_1
      CHECK_CLAUSE: (`id` % 2 = 0)
1 row in set (0.00 sec)
```

`CHECK_CONSTRAINTS`テーブルのフィールドは以下のように説明されています。

* `CONSTRAINT_CATALOG`: 制約のカタログ、つまり常に`def`です。
* `CONSTRAINT_SCHEMA`: 制約のスキーマ。
* `CONSTRAINT_NAME`: 制約の名前。
* `CHECK_CLAUSE`: チェック制約の節。