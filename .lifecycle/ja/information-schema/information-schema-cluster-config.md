---
title: CLUSTER_CONFIG
summary: `CLUSTER_CONFIG`の`information_schema`テーブルの情報を学ぶ。
aliases: ['/docs/dev/system-tables/system-table-cluster-config/','/docs/dev/reference/system-databases/cluster-config/','/tidb/dev/system-table-cluster-config/']
---

# CLUSTER_CONFIG

`CLUSTER_CONFIG`クラスタ構成テーブルを使用して、クラスタ内のすべてのサーバー コンポーネントの現在の構成を取得できます。以前のTiDBリリースでは同様の情報を取得するために各インスタンスのHTTP APIエンドポイントにアクセスする必要がありましたが、これにより簡素化されます。

> **注意:**
>
> このテーブルはTiDB Self-Hostedにのみ適用され、[TiDB Cloud](https://docs.pingcap.com/tidbcloud/)では利用できません。

{{< copyable "sql" >}}

```sql
USE information_schema;
DESC cluster_config;
```

```sql
+----------+--------------+------+------+---------+-------+
| Field    | Type         | Null | Key  | Default | Extra |
+----------+--------------+------+------+---------+-------+
| TYPE     | varchar(64)  | YES  |      | NULL    |       |
| INSTANCE | varchar(64)  | YES  |      | NULL    |       |
| KEY      | varchar(256) | YES  |      | NULL    |       |
| VALUE    | varchar(128) | YES  |      | NULL    |       |
+----------+--------------+------+------+---------+-------+
```

フィールドの説明:

* `TYPE`: インスタンスの種類。`tidb`、`pd`、`tikv`のオプション値があります。
* `INSTANCE`: インスタンスのサービスアドレス。
* `KEY`: 構成項目の名前。
* `VALUE`: 構成項目の値。

次の例は、`CLUSTER_CONFIG`テーブルを使用して、TiKVインスタンスで`coprocessor`の構成をクエリする方法を示しています:

{{< copyable "sql" >}}

```sql
SELECT * FROM cluster_config WHERE type='tikv' AND `key` LIKE 'coprocessor%';
```

```sql
+------+-----------------+-----------------------------------+---------+
| TYPE | INSTANCE        | KEY                               | VALUE   |
+------+-----------------+-----------------------------------+---------+
| tikv | 127.0.0.1:20165 | coprocessor.batch-split-limit     | 10      |
| tikv | 127.0.0.1:20165 | coprocessor.region-max-keys       | 1440000 |
| tikv | 127.0.0.1:20165 | coprocessor.region-max-size       | 144MiB  |
| tikv | 127.0.0.1:20165 | coprocessor.region-split-keys     | 960000  |
| tikv | 127.0.0.1:20165 | coprocessor.region-split-size     | 96MiB   |
| tikv | 127.0.0.1:20165 | coprocessor.split-region-on-table | false   |
+------+-----------------+-----------------------------------+---------+
6 rows in set (0.00 sec)
```