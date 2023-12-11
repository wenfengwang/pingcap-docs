---
title: TIDB_HOT_REGIONS_HISTORY
summary: `TIDB_HOT_REGIONS_HISTORY`のinformation_schemaテーブルについて学ぶ。

# TIDB_HOT_REGIONS_HISTORY

`TIDB_HOT_REGIONS_HISTORY`テーブルは、PDによって定期的にローカルに記録された履歴のhot Regionsに関する情報を提供します。

> **注意:**
>
> このテーブルは[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスターでは利用できません。

<CustomContent platform="tidb">

[`hot-regions-write-interval`](/pd-configuration-file.md#hot-regions-write-interval-new-in-v540)を構成して、記録間隔を指定できます。デフォルト値は10分です。[`hot-regions-reserved-days`](/pd-configuration-file.md#hot-regions-reserved-days-new-in-v540)を構成して、hot Regionsに関する履歴情報を保持する期間を指定できます。デフォルト値は7日です。詳細は[PD設定ファイルの説明](/pd-configuration-file.md#hot-regions-write-interval-new-in-v540)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

デフォルトでは、記録間隔は10分で、hot Regionsに関する履歴情報を保持する期間は7日です。

</CustomContent>

{{< copyable "sql" >}}

```sql
USE information_schema;
DESC tidb_hot_regions_history;
```

```sql
+-------------+--------------+------+------+---------+-------+
| Field       | Type         | Null | Key  | Default | Extra |
+-------------+--------------+------+------+---------+-------+
| UPDATE_TIME | timestamp(6) | YES  |      | NULL    |       |
| DB_NAME     | varchar(64)  | YES  |      | NULL    |       |
| TABLE_NAME  | varchar(64)  | YES  |      | NULL    |       |
| TABLE_ID    | bigint(21)   | YES  |      | NULL    |       |
| INDEX_NAME  | varchar(64)  | YES  |      | NULL    |       |
| INDEX_ID    | bigint(21)   | YES  |      | NULL    |       |
| REGION_ID   | bigint(21)   | YES  |      | NULL    |       |
| STORE_ID    | bigint(21)   | YES  |      | NULL    |       |
| PEER_ID     | bigint(21)   | YES  |      | NULL    |       |
| IS_LEARNER  | tinyint(1)   | NO   |      | 0       |       |
| IS_LEADER   | tinyint(1)   | NO   |      | 0       |       |
| TYPE        | varchar(64)  | YES  |      | NULL    |       |
| HOT_DEGREE  | bigint(21)   | YES  |      | NULL    |       |
| FLOW_BYTES  | double       | YES  |      | NULL    |       |
| KEY_RATE    | double       | YES  |      | NULL    |       |
| QUERY_RATE  | double       | YES  |      | NULL    |       |
+-------------+--------------+------+------+---------+-------+
16 rows in set (0.00 sec)
```

`TIDB_HOT_REGIONS_HISTORY`テーブルのフィールドの説明は以下の通りです:

* UPDATE_TIME: hot Regionの更新時間
* DB_NAME: hot Regionが存在するオブジェクトのデータベース名
* TABLE_ID: hot Regionが存在するテーブルのID
* TABLE_NAME: hot Regionが存在するテーブル名
* INDEX_NAME: hot Regionが存在するインデックス名
* INDEX_ID: hot Regionが存在するインデックスのID
* REGION_ID: hot RegionのID
* STORE_ID: hot Regionが存在するストアのID
* PEER_ID: hot Regionに対応するPeerのID
* IS_LEARNER: PEERがLEARNERかどうか
* IS_LEADER: PEERがLEADERかどうか
* TYPE: hot Regionのタイプ
* HOT_DEGREE: hot Regionのhot degree
* FLOW_BYTES: Regionで書き込まれたバイト数と読み込まれたバイト数
* KEY_RATE: Regionで書き込まれたキー数と読み込まれたキー数
* QUERY_RATE: Regionで書き込まれたクエリ数と読み込まれたクエリ数

> **注意:**
>
> `UPDATE_TIME`、`REGION_ID`、`STORE_ID`、`PEER_ID`、`IS_LEARNER`、`IS_LEADER`、`TYPE`のフィールドはPDサーバーにプッシュダウンされて実行されます。テーブルの使用オーバーヘッドを減らすために、検索のための時間範囲を指定し、可能な条件をできるだけ指定する必要があります。例えば、`select * from tidb_hot_regions_history where store_id = 11 and update_time > '2020-05-18 20:40:00' and update_time < '2020-05-18 21:40:00' and type='write'`。

## 一般的なユーザーのシナリオ

* 特定の時間範囲内のhot Regionsをクエリします。`update_time`を実際の時間に置き換えてください。

    {{< copyable "sql" >}}

    ```sql
    SELECT * FROM INFORMATION_SCHEMA.TIDB_HOT_REGIONS_HISTORY WHERE update_time >'2021-08-18 21:40:00' and update_time <'2021-09-19 00:00:00';
    ```

    > **注意:**
    >
    > `UPDATE_TIME`はUnixタイムスタンプもサポートしています。例: `update_time >TIMESTAMP('2021-08-18 21:40:00')` または `update_time > FROM_UNIXTIME(1629294000.000)`。

* 特定の時間範囲内のテーブル内のhot Regionsをクエリします。`update_time`および`table_name`を実際の値に置き換えてください。

    {{< copyable "sql" >}}

    ```SQL
    SELECT * FROM INFORMATION_SCHEMA.TIDB_HOT_REGIONS_HISTORY WHERE update_time >'2021-08-18 21:40:00' and update_time <'2021-09-19 00:00:00' and TABLE_NAME = 'table_name';
    ```

* 特定の時間範囲内のhot Regionsの分布をクエリします。`update_time`および`table_name`を実際の値に置き換えてください。

    {{< copyable "sql" >}}

    ```sql
    SELECT count(region_id) cnt, store_id FROM INFORMATION_SCHEMA.TIDB_HOT_REGIONS_HISTORY WHERE update_time >'2021-08-18 21:40:00' and update_time <'2021-09-19 00:00:00' and table_name = 'table_name' GROUP BY STORE_ID ORDER BY cnt DESC;
    ```

* 特定の時間範囲内のhot Leader Regionsの分布をクエリします。`update_time`および`table_name`を実際の値に置き換えてください。

    {{< copyable "sql" >}}

    ```sql
    SELECT count(region_id) cnt, store_id FROM INFORMATION_SCHEMA.TIDB_HOT_REGIONS_HISTORY WHERE update_time >'2021-08-18 21:40:00' and update_time <'2021-09-19 00:00:00' and table_name = 'table_name' and is_leader=1 GROUP BY STORE_ID ORDER BY cnt DESC;
    ```

* 特定の時間範囲内のhot Index Regionsの分布をクエリします。`update_time`および`table_name`を実際の値に置き換えてください。

    {{< copyable "sql" >}}

    ```sql
    SELECT count(region_id) cnt, index_name, store_id FROM INFORMATION_SCHEMA.TIDB_HOT_REGIONS_HISTORY WHERE update_time >'2021-08-18 21:40:00' and update_time <'2021-09-19 00:00:00' and table_name = 'table_name' group by index_name, store_id order by index_name,cnt desc;
    ```

* 特定の時間範囲内のhot Index Leader Regionsの分布をクエリします。`update_time`および`table_name`を実際の値に置き換えてください。

    {{< copyable "sql" >}}

    ```sql
    SELECT count(region_id) cnt, index_name, store_id FROM INFORMATION_SCHEMA.Tidb_HOT_REGIONS_History WHERE UPDATE_TIME >2021-08-18 21:40:00 AND UPDATE_TIME <2022-09-19 00:00:00 AND TABLE_NAME = 'TABLE_NAME' AND IS_LEADER=1 GROUP BY STORE_ID ORDER BY cnt DESC;
    ```