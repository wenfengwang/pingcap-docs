---
title: TIKV_STORE_STATUS
summary: `TIKV_STORE_STATUS` INFORMATION_SCHEMAテーブルの使用方法について学びます。

# TIKV_STORE_STATUS

`TIKV_STORE_STATUS`テーブルは、PDのAPIを介してTiKVノードの基本情報を示します。クラスターで割り当てられたID、アドレスとポート、ステータス、容量、および現在のノードのRegionリーダーの数などが含まれます。

> **注:**
>
> このテーブルは[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスターでは利用できません。

```sql
USE INFORMATION_SCHEMA;
DESC TIKV_STORE_STATUS;
```

出力は次のようになります:

```sql
+-------------------+-------------+------+------+---------+-------+
| Field             | Type        | Null | Key  | Default | Extra |
+-------------------+-------------+------+------+---------+-------+
| STORE_ID          | bigint(21)  | YES  |      | NULL    |       |
| ADDRESS           | varchar(64) | YES  |      | NULL    |       |
| STORE_STATE       | bigint(21)  | YES  |      | NULL    |       |
| STORE_STATE_NAME  | varchar(64) | YES  |      | NULL    |       |
| LABEL             | json        | YES  |      | NULL    |       |
| VERSION           | varchar(64) | YES  |      | NULL    |       |
| CAPACITY          | varchar(64) | YES  |      | NULL    |       |
| AVAILABLE         | varchar(64) | YES  |      | NULL    |       |
| LEADER_COUNT      | bigint(21)  | YES  |      | NULL    |       |
| LEADER_WEIGHT     | double      | YES  |      | NULL    |       |
| LEADER_SCORE      | double      | YES  |      | NULL    |       |
| LEADER_SIZE       | bigint(21)  | YES  |      | NULL    |       |
| REGION_COUNT      | bigint(21)  | YES  |      | NULL    |       |
| REGION_WEIGHT     | double      | YES  |      | NULL    |       |
| REGION_SCORE      | double      | YES  |      | NULL    |       |
| REGION_SIZE       | bigint(21)  | YES  |      | NULL    |       |
| START_TS          | datetime    | YES  |      | NULL    |       |
| LAST_HEARTBEAT_TS | datetime    | YES  |      | NULL    |       |
| UPTIME            | varchar(64) | YES  |      | NULL    |       |
+-------------------+-------------+------+------+---------+-------+
19 rows in set (0.00 sec)
```

`TIKV_STORE_STATUS`テーブルの列の説明は次のとおりです:

* `STORE_ID`: ストアのID。
* `ADDRESS`: ストアのアドレス。
* `STORE_STATE`: ストアの状態の識別子。`STORE_STATE_NAME`に対応します。
* `STORE_STATE_NAME`: ストアの状態の名前。名前は`Up`、`Offline`、または`Tombstone`です。
* `LABEL`: ストアに設定されたラベル。
* `VERSION`: ストアのバージョン番号。
* `CAPACITY`: ストアのストレージ容量。
* `AVAILABLE`: ストアの残りのストレージ容量。
* `LEADER_COUNT`: ストア上のリーダーの数。
* `LEADER_WEIGHT`: ストアのリーダーウェイト。
* `LEADER_SCORE`: ストアのリーダースコア。
* `LEADER_SIZE`: ストア上のすべてのリーダーのおおよその合計データサイズ（MB）。
* `REGION_COUNT`: ストア上のリージョンの数。
* `REGION_WEIGHT`: ストアのリージョンウェイト。
* `REGION_SCORE`: ストアのリージョンスコア。
* `REGION_SIZE`: ストア上のすべてのリージョンのおおよその合計データサイズ（MB）。
* `START_TS`: ストアが開始されたタイムスタンプ。
* `LAST_HEARTBEAT_TS`: ストアが最後に送信したハートビートのタイムスタンプ。
* `UPTIME`: ストアが開始してからの合計時間。