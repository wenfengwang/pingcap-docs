---
title: KEY_COLUMN_USAGE
summary: `KEY_COLUMN_USAGE`のinformation_schemaテーブルの情報を学ぶ。

# KEY_COLUMN_USAGE

`KEY_COLUMN_USAGE`テーブルは、列のキー制約（主キー制約など）を記述します。

{{< copyable "sql" >}}

```sql
USE information_schema;
DESC key_column_usage;
```

```
+-------------------------------+--------------+------+------+---------+-------+
| Field                         | Type         | Null | Key  | Default | Extra |
+-------------------------------+--------------+------+------+---------+-------+
| CONSTRAINT_CATALOG            | varchar(512) | NO   |      | NULL    |       |
| CONSTRAINT_SCHEMA             | varchar(64)  | NO   |      | NULL    |       |
| CONSTRAINT_NAME               | varchar(64)  | NO   |      | NULL    |       |
| TABLE_CATALOG                 | varchar(512) | NO   |      | NULL    |       |
| TABLE_SCHEMA                  | varchar(64)  | NO   |      | NULL    |       |
| TABLE_NAME                    | varchar(64)  | NO   |      | NULL    |       |
| COLUMN_NAME                   | varchar(64)  | NO   |      | NULL    |       |
| ORDINAL_POSITION              | bigint(10)   | NO   |      | NULL    |       |
| POSITION_IN_UNIQUE_CONSTRAINT | bigint(10)   | YES  |      | NULL    |       |
| REFERENCED_TABLE_SCHEMA       | varchar(64)  | YES  |      | NULL    |       |
| REFERENCED_TABLE_NAME         | varchar(64)  | YES  |      | NULL    |       |
| REFERENCED_COLUMN_NAME        | varchar(64)  | YES  |      | NULL    |       |
+-------------------------------+--------------+------+------+---------+-------+
12 行 (0.00 秒)
```

{{< copyable "sql" >}}

```sql
SELECT * FROM key_column_usage WHERE table_schema='mysql' and table_name='user';
```

```
*************************** 1. 行 ***************************
           CONSTRAINT_CATALOG: def
            CONSTRAINT_SCHEMA: mysql
              CONSTRAINT_NAME: PRIMARY
                TABLE_CATALOG: def
                 TABLE_SCHEMA: mysql
                   TABLE_NAME: user
                  COLUMN_NAME: Host
             ORDINAL_POSITION: 1
POSITION_IN_UNIQUE_CONSTRAINT: NULL
      REFERENCED_TABLE_SCHEMA: NULL
        REFERENCED_TABLE_NAME: NULL
       REFERENCED_COLUMN_NAME: NULL
*************************** 2. 行 ***************************
           CONSTRAINT_CATALOG: def
            CONSTRAINT_SCHEMA: mysql
              CONSTRAINT_NAME: PRIMARY
                TABLE_CATALOG: def
                 TABLE_SCHEMA: mysql
                   TABLE_NAME: user
                  COLUMN_NAME: User
             ORDINAL_POSITION: 2
POSITION_IN_UNIQUE_CONSTRAINT: NULL
      REFERENCED_TABLE_SCHEMA: NULL
        REFERENCED_TABLE_NAME: NULL
       REFERENCED_COLUMN_NAME: NULL
2 行 (0.00 秒)
```

`KEY_COLUMN_USAGE`テーブルの列の説明は以下の通りです。

* `CONSTRAINT_CATALOG`: 制約が属するカタログの名前。値は常に `def` です。
* `CONSTRAINT_SCHEMA`: 制約が属するスキーマの名前。
* `CONSTRAINT_NAME`: 制約の名前。
* `TABLE_CATALOG`: テーブルが属するカタログの名前。値は常に `def` です。
* `TABLE_SCHEMA`: テーブルが属するスキーマの名前。
* `TABLE_NAME`: 制約のあるテーブルの名前。
* `COLUMN_NAME`: 制約のある列の名前。
* `ORDINAL_POSITION`: 制約内での列の位置。位置番号は `1` から始まります。
* `POSITION_IN_UNIQUE_CONSTRAINT`: ユニーク制約や主キー制約は空です。外部キー制約の場合、この列は参照されるテーブルのキーの位置です。
* `REFERENCED_TABLE_SCHEMA`: 制約で参照されているスキーマの名前。現在のTiDBでは、この列の値はすべての制約で `nil` ですが、外部キー制約の場合は除きます。
* `REFERENCED_TABLE_NAME`: 制約で参照されているテーブルの名前。現在のTiDBでは、この列の値はすべての制約で `nil` ですが、外部キー制約の場合は除きます。
* `REFERENCED_COLUMN_NAME`: 制約で参照されている列の名前。現在のTiDBでは、この列の値はすべての制約で `nil` ですが、外部キー制約の場合は除きます。
