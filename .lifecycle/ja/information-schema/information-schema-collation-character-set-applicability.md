---
title: COLLATION_CHARACTER_SET_APPLICABILITY
summary: `COLLATION_CHARACTER_SET_APPLICABILITY` INFORMATION_SCHEMA テーブルの情報を学びます。
---

# COLLATION_CHARACTER_SET_APPLICABILITY

`COLLATION_CHARACTER_SET_APPLICABILITY` テーブルは、照合順序を該当する文字セット名にマップします。`COLLATIONS` テーブルと同様に、MySQL との互換性のためにのみ含まれています。

```sql
USE INFORMATION_SCHEMA;
DESC COLLATION_CHARACTER_SET_APPLICABILITY;
```

出力は以下の通りです：

```sql
+--------------------+-------------+------+------+---------+-------+
| Field              | Type        | Null | Key  | Default | Extra |
+--------------------+-------------+------+------+---------+-------+
| COLLATION_NAME     | varchar(32) | NO   |      | NULL    |       |
| CHARACTER_SET_NAME | varchar(32) | NO   |      | NULL    |       |
+--------------------+-------------+------+------+---------+-------+
2 rows in set (0.00 sec)
```

`COLLATION_CHARACTER_SET_APPLICABILITY` テーブルで `utf8mb4` 文字セットの照合マッピングを表示します：

```sql
SELECT * FROM COLLATION_CHARACTER_SET_APPLICABILITY WHERE character_set_name='utf8mb4';
```

出力は以下の通りです：

```sql
+--------------------+--------------------+
| COLLATION_NAME     | CHARACTER_SET_NAME |
+--------------------+--------------------+
| utf8mb4_bin        | utf8mb4            |
| utf8mb4_general_ci | utf8mb4            |
| utf8mb4_unicode_ci | utf8mb4            |
+--------------------+--------------------+
3 rows in set (0.00 sec)
```

`COLLATION_CHARACTER_SET_APPLICABILITY` テーブルのカラムの説明は以下の通りです：

* `COLLATION_NAME`: 照合順序の名前。
* `CHARACTER_SET_NAME`: その照合順序が属する文字セットの名前。