---
title: CLUSTER_LOAD
summary: `CLUSTER_LOAD`情報スキーマテーブルを学ぶ。
aliases: ['/docs/dev/system-tables/system-table-cluster-load/', '/docs/dev/reference/system-databases/cluster-load/', '/tidb/dev/system-table-cluster-load/']
---

# CLUSTER_LOAD

`CLUSTER_LOAD`クラスタ負荷テーブルは、TiDBクラスタの各インスタンスが配置されているサーバーの現在の負荷情報を提供します。

> **注記:**
>
> このテーブルはTiDB Self-Hostedにのみ適用され、[TiDB Cloud](https://docs.pingcap.com/tidbcloud/)では利用できません。

{{< copyable "sql" >}}

```sql
USE information_schema;
DESC cluster_load;
```

```sql
+-------------+--------------+------+------+---------+-------+
| Field       | Type         | Null | Key  | Default | Extra |
+-------------+--------------+------+------+---------+-------+
| TYPE        | varchar(64)  | YES  |      | NULL    |       |
| INSTANCE    | varchar(64)  | YES  |      | NULL    |       |
| DEVICE_TYPE | varchar(64)  | YES  |      | NULL    |       |
| DEVICE_NAME | varchar(64)  | YES  |      | NULL    |       |
| NAME        | varchar(256) | YES  |      | NULL    |       |
| VALUE       | varchar(128) | YES  |      | NULL    |       |
+-------------+--------------+------+------+---------+-------+
6 行が選択されました (0.00 sec)
```

フィールドの説明:

* `TYPE`: [`information_schema.cluster_info`](/information-schema/information-schema-cluster-info.md)テーブルの`TYPE`フィールドに対応します。オプションの値は`tidb`、`pd`、`tikv`です。
* `INSTANCE`: [`information_schema.cluster_info`](/information-schema/information-schema-cluster-info.md)クラスタ情報テーブルの`INSTANCE`フィールドに対応します。
* `DEVICE_TYPE`: ハードウェアタイプ。現在は、`cpu`、`memory`、`disk`、`net`タイプをクエリできます。
* `DEVICE_NAME`: ハードウェア名。`DEVICE_TYPE`によって`DEVICE_NAME`の値が異なります。
    * `cpu`: ハードウェア名はcpuです。
    * `disk`: ディスク名です。
    * `net`: ネットワークカード名です。
    * `memory`: ハードウェア名はmemoryです。
* `NAME`: 異なる負荷タイプ。たとえば、cpuには`load1`、`load5`、`load15`の3つの負荷タイプがあり、それぞれ1分、5分、15分の平均負荷を意味します。
* `VALUE`: ハードウェア負荷の値。たとえば、`1min`、`5min`、`15min`はそれぞれ1分、5分、15分の平均負荷を意味します。

次の例では、`CLUSTER_LOAD`テーブルを使用してcpuの現在の負荷情報をクエリする方法を示します:

{{< copyable "sql" >}}

```sql
SELECT * FROM cluster_load WHERE device_type='cpu' AND device_name='cpu';
```

```sql
+------+-----------------+-------------+-------------+--------+-------+
| TYPE | INSTANCE        | DEVICE_TYPE | DEVICE_NAME | NAME   | VALUE |
+------+-----------------+-------------+-------------+--------+-------+
| tidb | 0.0.0.0:4000    | cpu         | cpu         | load1  | 0.13  |
| tidb | 0.0.0.0:4000    | cpu         | cpu         | load5  | 0.25  |
| tidb | 0.0.0.0:4000    | cpu         | cpu         | load15 | 0.31  |
| pd   | 127.0.0.1:2379  | cpu         | cpu         | load1  | 0.13  |
| pd   | 127.0.0.1:2379  | cpu         | cpu         | load5  | 0.25  |
| pd   | 127.0.0.1:2379  | cpu         | cpu         | load15 | 0.31  |
| tikv | 127.0.0.1:20165 | cpu         | cpu         | load1  | 0.13  |
| tikv | 127.0.0.1:20165 | cpu         | cpu         | load5  | 0.25  |
| tikv | 127.0.0.1:20165 | cpu         | cpu         | load15 | 0.31  |
+------+-----------------+-------------+-------------+--------+-------+
9 行が選択されました (1.50 sec)
```