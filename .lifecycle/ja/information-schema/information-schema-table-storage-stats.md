---
title: TABLE_STORAGE_STATS
summary: `TABLE_STORAGE_STATS` INFORMATION_SCHEMAテーブルの使用方法について学びます。

# TABLE_STORAGE_STATS

`TABLE_STORAGE_STATS`テーブルは、ストレージエンジン（TiKV）によって保存されたテーブルのサイズに関する情報を提供します。

```sql
USE INFORMATION_SCHEMA;
DESC TABLE_STORAGE_STATS;
```

以下の出力が表示されます:

```sql
+--------------------+-------------+------+------+---------+-------+
| Field              | Type        | Null | Key  | Default | Extra |
+--------------------+-------------+------+------+---------+-------+
| TABLE_SCHEMA       | varchar(64) | YES  |      | NULL    |       |
| TABLE_NAME         | varchar(64) | YES  |      | NULL    |       |
| TABLE_ID           | bigint(21)  | YES  |      | NULL    |       |
| PEER_COUNT         | bigint(21)  | YES  |      | NULL    |       |
| REGION_COUNT       | bigint(21)  | YES  |      | NULL    |       |
| EMPTY_REGION_COUNT | bigint(21)  | YES  |      | NULL    |       |
| TABLE_SIZE         | bigint(64)  | YES  |      | NULL    |       |
| TABLE_KEYS         | bigint(64)  | YES  |      | NULL    |       |
+--------------------+-------------+------+------+---------+-------+
8 rows in set (0.00 sec)
```

```sql
CREATE TABLE test.t1 (id INT);
INSERT INTO test.t1 VALUES (1);
SELECT * FROM TABLE_STORAGE_STATS WHERE table_schema = 'test' AND table_name = 't1'\G
```

以下の出力が表示されます:

```sql
*************************** 1. row ***************************
      TABLE_SCHEMA: test
        TABLE_NAME: t1
          TABLE_ID: 56
        PEER_COUNT: 1
      REGION_COUNT: 1
EMPTY_REGION_COUNT: 1
        TABLE_SIZE: 1
        TABLE_KEYS: 0
1 row in set (0.00 sec)
```

`TABLE_STORAGE_STATS`テーブルのフィールドは以下のように説明されています:

* `TABLE_SCHEMA`: テーブルが属するスキーマの名前。
* `TABLE_NAME`: テーブルの名前。
* `TABLE_ID`: テーブルのID。
* `PEER_COUNT`: テーブルのレプリカの数。
* `REGION_COUNT`: リージョンの数。
* `EMPTY_REGION_COUNT`: このテーブルにデータを含まないリージョンの数。
* `TABLE_SIZE`: テーブルの合計サイズ（単位：MiB）。
* `TABLE_KEYS`: テーブル内のレコードの合計数。