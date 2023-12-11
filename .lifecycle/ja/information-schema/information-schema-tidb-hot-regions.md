---
title: TIDB_HOT_REGIONS
summary: `TIDB_HOT_REGIONS`のinformation_schemaテーブルについて学びます。

# TIDB_HOT_REGIONS

`TIDB_HOT_REGIONS`テーブルは、現在のホットリージョンに関する情報を提供します。過去のホットリージョンに関する情報については、[TIDB_HOT_REGIONS_HISTORY](/information-schema/information-schema-tidb-hot-regions-history.md)を参照してください。

> **注意:**
>
> このテーブルは、TiDB Self-Hostedにのみ適用され、[TiDB Cloud](https://docs.pingcap.com/tidbcloud/)では利用できません。

{{< copyable "sql" >}}

```sql
USE information_schema;
DESC tidb_hot_regions;
```

```
+----------------+-------------+------+------+---------+-------+
| Field          | Type        | Null | Key  | Default | Extra |
+----------------+-------------+------+------+---------+-------+
| TABLE_ID       | bigint(21)  | YES  |      | NULL    |       |
| INDEX_ID       | bigint(21)  | YES  |      | NULL    |       |
| DB_NAME        | varchar(64) | YES  |      | NULL    |       |
| TABLE_NAME     | varchar(64) | YES  |      | NULL    |       |
| INDEX_NAME     | varchar(64) | YES  |      | NULL    |       |
| REGION_ID      | bigint(21)  | YES  |      | NULL    |       |
| TYPE           | varchar(64) | YES  |      | NULL    |       |
| MAX_HOT_DEGREE | bigint(21)  | YES  |      | NULL    |       |
| REGION_COUNT   | bigint(21)  | YES  |      | NULL    |       |
| FLOW_BYTES     | bigint(21)  | YES  |      | NULL    |       |
+----------------+-------------+------+------+---------+-------+
10 rows in set (0.00 sec)
```

`TIDB_HOT_REGIONS`テーブルの各列の説明は以下の通りです：

* `TABLE_ID`: ホットリージョンが存在するテーブルのID。
* `INDEX_ID`: ホットリージョンが存在するインデックスのID。
* `DB_NAME`: ホットリージョンが存在するオブジェクトのデータベース名。
* `TABLE_NAME`: ホットリージョンが存在するテーブルの名前。
* `INDEX_NAME`: ホットリージョンが存在するインデックスの名前。
* `REGION_ID`: ホットリージョンのID。
* `TYPE`: ホットリージョンのタイプ。
* `MAX_HOT_DEGREE`: リージョンの最大ホット度。
* `REGION_COUNT`: インスタンス内のホットリージョンの数。
* `FLOW_BYTES`: リージョンで書き込まれたバイト数および読み込まれたバイト数。