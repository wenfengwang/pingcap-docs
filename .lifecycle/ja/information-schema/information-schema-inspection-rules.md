---
title: INSPECTION_RULES
summary: `INSPECTION_RULES`の information_schema テーブルに関する情報を学びます。

# INSPECTION_RULES

`INSPECTION_RULES` テーブルは、検査結果に実行される診断テストに関する情報を提供します。 例の使用法については [検査結果](/information-schema/information-schema-inspection-result.md) を参照してください。

> **注意:**
>
> このテーブルは TiDB セルフホスト環境にのみ適用され、[TiDB Cloud](https://docs.pingcap.com/tidbcloud/) では利用できません。

{{< copyable "sql" >}}

```sql
USE information_schema;
DESC inspection_rules;
```

```
+---------+--------------+------+------+---------+-------+
| Field   | Type         | Null | Key  | Default | Extra |
+---------+--------------+------+------+---------+-------+
| NAME    | varchar(64)  | YES  |      | NULL    |       |
| TYPE    | varchar(64)  | YES  |      | NULL    |       |
| COMMENT | varchar(256) | YES  |      | NULL    |       |
+---------+--------------+------+------+---------+-------+
3 行が選択されました (0.00 秒)
```

{{< copyable "sql" >}}

```sql
SELECT * FROM inspection_rules;
```

```
+-----------------+------------+---------+
| NAME            | TYPE       | COMMENT |
+-----------------+------------+---------+
| config          | inspection |         |
| version         | inspection |         |
| node-load       | inspection |         |
| critical-error  | inspection |         |
| threshold-check | inspection |         |
| ddl             | summary    |         |
| gc              | summary    |         |
| pd              | summary    |         |
| query-summary   | summary    |         |
| raftstore       | summary    |         |
| read-link       | summary    |         |
| rocksdb         | summary    |         |
| stats           | summary    |         |
| wait-events     | summary    |         |
| write-link      | summary    |         |
+-----------------+------------+---------+
15 行が選択されました (0.00 秒)
```