---
title: TIDB_INDEXES
summary: `TIDB_INDEXES`情報スキーマテーブルについて学ぶ。

# TIDB_INDEXES

`TIDB_INDEXES`テーブルはすべてのテーブルのINDEX情報を提供します。

{{< copyable "sql" >}}

```sql
USE information_schema;
DESC tidb_indexes;
```

```
+---------------+---------------+------+------+---------+-------+
| Field         | Type          | Null | Key  | Default | Extra |
+---------------+---------------+------+------+---------+-------+
| TABLE_SCHEMA  | varchar(64)   | YES  |      | NULL    |       |
| TABLE_NAME    | varchar(64)   | YES  |      | NULL    |       |
| NON_UNIQUE    | bigint(21)    | YES  |      | NULL    |       |
| KEY_NAME      | varchar(64)   | YES  |      | NULL    |       |
| SEQ_IN_INDEX  | bigint(21)    | YES  |      | NULL    |       |
| COLUMN_NAME   | varchar(64)   | YES  |      | NULL    |       |
| SUB_PART      | bigint(21)    | YES  |      | NULL    |       |
| INDEX_COMMENT | varchar(2048) | YES  |      | NULL    |       |
| Expression    | varchar(64)   | YES  |      | NULL    |       |
| INDEX_ID      | bigint(21)    | YES  |      | NULL    |       |
| IS_VISIBLE    | varchar(64)   | YES  |      | NULL    |       |
| CLUSTERED     | varchar(64)   | YES  |      | NULL    |       |
+---------------+---------------+------+------+---------+-------+
12 rows in set (0.00 sec)
```

`INDEX_ID`は、各インデックスにTiDBが割り当てた一意のIDです。他のテーブルやAPIから取得した`INDEX_ID`と結合操作を行う際に使用できます。

たとえば、[`SLOW_QUERY`テーブル](/information-schema/information-schema-slow-query.md)で遅いクエリに関係する`TABLE_ID`と`INDEX_ID`を取得し、次のSQLステートメントを使用して特定のインデックス情報を取得できます。

```sql
SELECT
 tidb_indexes.*
FROM
 tidb_indexes,
 tables
WHERE
  tidb_indexes.table_schema = tables.table_schema
 AND tidb_indexes.table_name = tidb_indexes.table_name
 AND tables.tidb_table_id = ?
 AND index_id = ?
```

`TIDB_INDEXES`テーブルのフィールドは次のように説明されます:

* `TABLE_SCHEMA`: インデックスが属するスキーマの名前。
* `TABLE_NAME`: インデックスが属するテーブルの名前。
* `NON_UNIQUE`: インデックスがユニークならば、値は`0`、そうでなければ値は`1`。
* `KEY_NAME`: インデックス名。インデックスが主キーの場合、名前は`PRIMARY`。
* `SEQ_IN_INDEX`: インデックス内の列の連番。`1`から始まります。
* `COLUMN_NAME`: インデックスが配置されている列の名前。
* `SUB_PART`: インデックスの接頭辞長。列が部分的にインデックス化されている場合、`SUB_PART`の値はインデックス化された文字数です。それ以外の場合、値は`NULL`です。
* `INDEX_COMMENT`: インデックスのコメント。インデックスが作成された際に設定されます。
* `INDEX_ID`: インデックスID。
* `IS_VISIBLE`: インデックスが可視であるかどうか。
* `CLUSTERED`: [クラスタ化インデックス](/clustered-indexes.md)であるかどうか。