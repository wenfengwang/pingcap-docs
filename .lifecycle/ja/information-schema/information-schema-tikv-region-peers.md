---
title: TIKV_REGION_PEERS
summary: `TIKV_REGION_PEERS` INFORMATION_SCHEMA テーブルの使用方法について学びます。

# TIKV_REGION_PEERS

`TIKV_REGION_PEERS` テーブルは、TiKV の単一のリージョンノードの詳細情報を表示します。リージョンノードが学習者かリーダーかなどが含まれます。

> **注意:**
>
> このテーブルは [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) クラスターでは利用できません。

```sql
USE INFORMATION_SCHEMA;
DESC TIKV_REGION_PEERS;
```

出力は以下のようになります:

```sql
+--------------+-------------+------+------+---------+-------+
| Field        | Type        | Null | Key  | Default | Extra |
+--------------+-------------+------+------+---------+-------+
| REGION_ID    | bigint(21)  | YES  |      | NULL    |       |
| PEER_ID      | bigint(21)  | YES  |      | NULL    |       |
| STORE_ID     | bigint(21)  | YES  |      | NULL    |       |
| IS_LEARNER   | tinyint(1)  | NO   |      | 0       |       |
| IS_LEADER    | tinyint(1)  | NO   |      | 0       |       |
| STATUS       | varchar(10) | YES  |      | 0       |       |
| DOWN_SECONDS | bigint(21)  | YES  |      | 0       |       |
+--------------+-------------+------+------+---------+-------+
7 rows in set (0.01 sec)
```

たとえば、`WRITTEN_BYTES` の値が最大の3つのリージョンに対する特定の TiKV アドレスをクエリするには、次の SQL 文を使用できます:

```sql
SELECT
  address,
  tikv.address,
  region.region_id
FROM
  TIKV_STORE_STATUS tikv,
  TIKV_REGION_PEERS peer,
  (SELECT * FROM tikv_region_status ORDER BY written_bytes DESC LIMIT 3) region
WHERE
  region.region_id = peer.region_id
  AND peer.is_leader = 1
  AND peer.store_id = tikv.store_id;
```

`TIKV_REGION_PEERS` テーブルのフィールドは以下のように説明されます:

* REGION_ID: リージョン ID。
* PEER_ID: リージョンのピアの ID。
* STORE_ID: リージョンが存在する TiKV ストアの ID。
* IS_LEARNER: ピアが学習者であるかどうか。
* IS_LEADER: ピアがリーダーであるかどうか。
* STATUS: ピアの状態:
    * PENDING: 一時的に利用できません。
    * DOWN: オフラインになり変換されました。このピアはもはやサービスを提供しません。
    * NORMAL: 通常運転中。
* DOWN_SECONDS: オフライン状態の期間（秒単位）。