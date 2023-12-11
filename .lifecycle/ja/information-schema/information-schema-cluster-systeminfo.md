---
title: CLUSTER_SYSTEMINFO
summary: `CLUSTER_SYSTEMINFO` カーネルパラメーターテーブルの使用方法を学ぶ。
aliases: ['/docs/dev/system-tables/system-table-cluster-systeminfo/','/docs/dev/reference/system-databases/cluster-systeminfo/','/tidb/dev/system-table-cluster-systeminfo/']
---

# CLUSTER_SYSTEMINFO

`CLUSTER_SYSTEMINFO` カーネルパラメーターテーブルを使用して、クラスター内のすべてのインスタンスが配置されているサーバーのカーネル構成情報をクエリできます。現在は`sysctl`システムの情報をクエリできます。

> **注意:**
>
> このテーブルはTiDB自己ホスト型にのみ適用され、[TiDB Cloud](https://docs.pingcap.com/tidbcloud/)では利用できません。

{{< copyable "sql" >}}

```sql
USE information_schema;
DESC cluster_systeminfo;
```

```sql
+-------------+--------------+------+------+---------+-------+
| Field       | Type         | Null | Key  | Default | Extra |
+-------------+--------------+------+------+---------+-------+
| TYPE        | varchar(64)  | YES  |      | NULL    |       |
| INSTANCE    | varchar(64)  | YES  |      | NULL    |       |
| SYSTEM_TYPE | varchar(64)  | YES  |      | NULL    |       |
| SYSTEM_NAME | varchar(64)  | YES  |      | NULL    |       |
| NAME        | varchar(256) | YES  |      | NULL    |       |
| VALUE       | varchar(128) | YES  |      | NULL    |       |
+-------------+--------------+------+------+---------+-------+
6 rows in set (0.00 sec)
```

フィールドの説明:

* `TYPE`: [`information_schema.cluster_info`](/information-schema/information-schema-cluster-info.md) テーブルの `TYPE` フィールドに対応します。オプションの値は `tidb`, `pd`, `tikv` です。
* `INSTANCE`: [`information_schema.cluster_info`](/information-schema/information-schema-cluster-info.md) クラスター情報テーブルの `INSTANCE` フィールドに対応します。
* `SYSTEM_TYPE`: システムの種類。現在は `system` システムタイプをクエリできます。
* `SYSTEM_NAME`: システム名。現在は `sysctl` システム名をクエリできます。
* `NAME`: `sysctl` に対応する構成名です。
* `VALUE`: `sysctl` に対応する構成項目の値です。

次の例は、`CLUSTER_SYSTEMINFO` システム情報テーブルを使用して、クラスター内のすべてのサーバーのカーネルバージョンをクエリする方法を示しています。

```sql
SELECT * FROM cluster_systeminfo WHERE name LIKE '%kernel.osrelease%'
```

```sql
+------+-------------------+-------------+-------------+------------------+----------------------------+
| TYPE | INSTANCE          | SYSTEM_TYPE | SYSTEM_NAME | NAME             | VALUE                      |
+------+-------------------+-------------+-------------+------------------+----------------------------+
| tidb | 172.16.5.40:4008  | system      | sysctl      | kernel.osrelease | 3.10.0-862.14.4.el7.x86_64 |
| pd   | 172.16.5.40:20379 | system      | sysctl      | kernel.osrelease | 3.10.0-862.14.4.el7.x86_64 |
| tikv | 172.16.5.40:21150 | system      | sysctl      | kernel.osrelease | 3.10.0-862.14.4.el7.x86_64 |
+------+-------------------+-------------+-------------+------------------+----------------------------+
```