---
title: COLLATIONS
summary: `COLLATIONS`のinformation_schemaテーブルについて学ぶ。

# COLLATIONS

`COLLATIONS`テーブルは、`CHARACTER_SETS`テーブル内の文字セットに対応する照合順序のリストを提供します。現在、このテーブルはMySQLとの互換性のためにのみ含まれています。

{{< copyable "sql" >}}

```sql
USE information_schema;
DESC collations;
```

```sql
+--------------------+-------------+------+------+---------+-------+
| Field              | Type        | Null | Key  | Default | Extra |
+--------------------+-------------+------+------+---------+-------+
| COLLATION_NAME     | varchar(32) | YES  |      | NULL    |       |
| CHARACTER_SET_NAME | varchar(32) | YES  |      | NULL    |       |
| ID                 | bigint(11)  | YES  |      | NULL    |       |
| IS_DEFAULT         | varchar(3)  | YES  |      | NULL    |       |
| IS_COMPILED        | varchar(3)  | YES  |      | NULL    |       |
| SORTLEN            | bigint(3)   | YES  |      | NULL    |       |
+--------------------+-------------+------+------+---------+-------+
6 rows in set (0.00 sec)
```

{{< copyable "sql" >}}

```sql
SELECT * FROM collations WHERE character_set_name='utf8mb4';
```

```sql
+--------------------+--------------------+------+------------+-------------+---------+
| COLLATION_NAME     | CHARACTER_SET_NAME | ID   | IS_DEFAULT | IS_COMPILED | SORTLEN |
+--------------------+--------------------+------+------------+-------------+---------+
| utf8mb4_bin        | utf8mb4            |   46 | Yes        | Yes         |       1 |
| utf8mb4_general_ci | utf8mb4            |   45 |            | Yes         |       1 |
| utf8mb4_unicode_ci | utf8mb4            |  224 |            | Yes         |       1 |
+--------------------+--------------------+------+------------+-------------+---------+
3 rows in set (0.001 sec)
```

`COLLATIONS`テーブルの列の説明は以下の通りです:

* `COLLATION_NAME`: 照合順序の名前
* `CHARACTER_SET_NAME`: 照合順序が属する文字セットの名前
* `ID`: 照合順序のID
* `IS_DEFAULT`: この照合順序が所属する文字セットのデフォルトの照合順序かどうか
* `IS_COMPILED`: 文字セットがサーバーにコンパイルされているかどうか
* `SORTLEN`: 照合順序が文字を並べ替える際に割り当てられるメモリの最小長