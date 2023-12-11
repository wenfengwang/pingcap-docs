---
title: クラスターハードウェア
summary: `CLUSTER_HARDWARE` の information_schema テーブルについて学びます。
aliases: ['/docs/dev/system-tables/system-table-cluster-hardware/','/docs/dev/reference/system-databases/cluster-hardware/','/tidb/dev/system-table-cluster-hardware/']
---

# クラスターハードウェア

`CLUSTER_HARDWARE` ハードウェアシステムテーブルは、クラスターの各インスタンスが配置されているサーバーのハードウェア情報を提供します。

> **注意:**
>
> このテーブルは TiDB Self-Hosted にのみ適用され、[TiDB Cloud](https://docs.pingcap.com/tidbcloud/) では利用できません。

{{< copyable "sql" >}}

```sql
USE information_schema;
DESC cluster_hardware;
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
6 rows in set (0.00 sec)
```

フィールドの説明:

* `TYPE`: [`information_schema.cluster_info`](/information-schema/information-schema-cluster-info.md) テーブルの `TYPE` フィールドに対応します。オプションの値は `tidb`、`pd`、`tikv` です。
* `INSTANCE`: [`information_schema.cluster_info`](/information-schema/information-schema-cluster-info.md) クラスター情報テーブルの `INSTANCE` フィールドに対応します。
* `DEVICE_TYPE`: ハードウェアタイプ。現在、`cpu`、`memory`、`disk`、`net` タイプをクエリできます。
* `DEVICE_NAME`: ハードウェア名。`DEVICE_TYPE` によって `DEVICE_NAME` の値が異なります。
    * `cpu`: ハードウェア名は cpu です。
    * `memory`: ハードウェア名は memory です。
    * `disk`: ディスク名です。
    * `net`: ネットワークカード名です。
* `NAME`: ハードウェアの異なる情報名です。例えば、cpu には 2 つの情報名があります: `cpu-logical-cores` と `cpu-physical-cores` で、それぞれ論理コア数と物理コア数を意味します。
* `VALUE`: 対応するハードウェア情報の値です。例えば、ディスク容量や CPU コア数です。

以下の例は、`CLUSTER_HARDWARE` テーブルを使用して CPU 情報をクエリする方法を示しています:

{{< copyable "sql" >}}

```sql
SELECT * FROM cluster_hardware WHERE device_type='cpu' AND device_name='cpu' AND name LIKE '%cores';
```

```sql
+------+-----------------+-------------+-------------+--------------------+-------+
| TYPE | INSTANCE        | DEVICE_TYPE | DEVICE_NAME | NAME               | VALUE |
+------+-----------------+-------------+-------------+--------------------+-------+
| tidb | 0.0.0.0:4000    | cpu         | cpu         | cpu-logical-cores  | 16    |
| tidb | 0.0.0.0:4000    | cpu         | cpu         | cpu-physical-cores | 8     |
| pd   | 127.0.0.1:2379  | cpu         | cpu         | cpu-logical-cores  | 16    |
| pd   | 127.0.0.1:2379  | cpu         | cpu         | cpu-physical-cores | 8     |
| tikv | 127.0.0.1:20165 | cpu         | cpu         | cpu-logical-cores  | 16    |
| tikv | 127.0.0.1:20165 | cpu         | cpu         | cpu-physical-cores | 8     |
+------+-----------------+-------------+-------------+--------------------+-------+
6 rows in set (0.03 sec)
```