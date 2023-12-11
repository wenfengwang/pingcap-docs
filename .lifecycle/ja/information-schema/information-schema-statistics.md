---
title: 統計
summary: `統計` information_schemaテーブルについて学びます。
---

# 統計

`統計` テーブルはテーブルのインデックスについての情報を提供します。

{{< copyable "sql" >}}

```sql
USE information_schema;
DESC statistics;
```

```sql
+---------------+---------------+------+------+---------+-------+
| Field         | Type          | Null | Key  | Default | Extra |
+---------------+---------------+------+------+---------+-------+
| TABLE_CATALOG | varchar(512)  | YES  |      | NULL    |       |
| TABLE_SCHEMA  | varchar(64)   | YES  |      | NULL    |       |
| TABLE_NAME    | varchar(64)   | YES  |      | NULL    |       |
| NON_UNIQUE    | varchar(1)    | YES  |      | NULL    |       |
| INDEX_SCHEMA  | varchar(64)   | YES  |      | NULL    |       |
| INDEX_NAME    | varchar(64)   | YES  |      | NULL    |       |
| SEQ_IN_INDEX  | bigint(2)     | YES  |      | NULL    |       |
| COLUMN_NAME   | varchar(21)   | YES  |      | NULL    |       |
| COLLATION     | varchar(1)    | YES  |      | NULL    |       |
| CARDINALITY   | bigint(21)    | YES  |      | NULL    |       |
| SUB_PART      | bigint(3)     | YES  |      | NULL    |       |
| PACKED        | varchar(10)   | YES  |      | NULL    |       |
| NULLABLE      | varchar(3)    | YES  |      | NULL    |       |
| INDEX_TYPE    | varchar(16)   | YES  |      | NULL    |       |
| COMMENT       | varchar(16)   | YES  |      | NULL    |       |
| INDEX_COMMENT | varchar(1024) | YES  |      | NULL    |       |
| IS_VISIBLE    | varchar(3)    | YES  |      | NULL    |       |
| Expression    | varchar(64)   | YES  |      | NULL    |       |
+---------------+---------------+------+------+---------+-------+
18 行が選択されました (0.00 秒)
```

`統計` テーブルの各フィールドは以下のように説明されています:

* `TABLE_CATALOG`: インデックスを含むテーブルが属するカタログの名前。この値は常に `def` です。
* `TABLE_SCHEMA`: インデックスを含むテーブルが属するデータベースの名前。
* `TABLE_NAME`: インデックスを含むテーブルの名前。
* `NON_UNIQUE`: インデックスが重複する値を含まない場合は `0`、インデックスが重複する値を含む場合は `1` です。
* `INDEX_SCHEMA`: インデックスが属するデータベースの名前。
* `INDEX_NAME`: インデックスの名前。インデックスが主キーの場合、この値は常に `PRIMARY` です。
* `SEQ_IN_INDEX`: インデックス内の列番号。`1` から開始します。
* `COLUMN_NAME`: 列の名前。`Expression` 列の説明を参照してください。
* `COLLATION`: インデックス内の列のソート方法。この値は `A`（昇順）、 `D`（降順）、または `NULL`（未ソート）になります。
* `CARDINALITY`: TiDB はこのフィールドを使用しません。このフィールドの値は常に `0` です。
* `SUB_PART`: インデックスのプレフィックス。列のプレフィックスの一部のみがインデックス化されている場合、値はインデックス化された文字数です。列全体がインデックス化されている場合は `NULL` です。
* `PACKED`: TiDB はこのフィールドを使用しません。この値は常に `NULL` です。
* `NULLABLE`: 列に `NULL` 値が含まれる可能性がある場合は `YES`、含まれない場合は `''` です。
* `INDEX_TYPE`: インデックスのタイプ。
* `COMMENT`: インデックスに関連するその他の情報。
* `INDEX_COMMENT`: インデックスを作成する際にコメント属性が指定された場合のコメント。
* `IS_VISIBLE`: オプティマイザがこのインデックスを使用できるかどうか。
* `Expression`: 非式部分のインデックス キーの場合は `NULL`、式部分のインデックス キーの場合は式そのものです。 [Expression Index](/sql-statements/sql-statement-create-index.md#expression-index) を参照してください。

次の文は同等です:

```sql
SELECT * FROM INFORMATION_SCHEMA.STATISTICS
  WHERE table_name = 'tbl_name'
  AND table_schema = 'db_name'

SHOW INDEX
  FROM tbl_name
  FROM db_name
```