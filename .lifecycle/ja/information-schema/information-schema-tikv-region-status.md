---
title: TIKV_REGION_STATUS
summary: `TIKV_REGION_STATUS`の`information_schema`テーブルの情報を学ぶ。

# TIKV_REGION_STATUS

`TIKV_REGION_STATUS`テーブルは、PDのAPI経由でTiKVリージョンの基本情報を表示します。リージョンID、開始および終了キー値、読み取りおよび書き込みトラフィックなどが含まれます。

> **注意:**
>
> このテーブルは[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスターでは利用できません。

{{< copyable "sql" >}}

```sql
USE information_schema;
DESC tikv_region_status;
```

```sql
+---------------------------+-------------+------+------+---------+-------+
| Field                     | Type        | Null | Key  | Default | Extra |
+---------------------------+-------------+------+------+---------+-------+
| REGION_ID                 | bigint(21)  | YES  |      | NULL    |       |
| START_KEY                 | text        | YES  |      | NULL    |       |
| END_KEY                   | text        | YES  |      | NULL    |       |
| TABLE_ID                  | bigint(21)  | YES  |      | NULL    |       |
| DB_NAME                   | varchar(64) | YES  |      | NULL    |       |
| TABLE_NAME                | varchar(64) | YES  |      | NULL    |       |
| IS_INDEX                  | tinyint(1)  | NO   |      | 0       |       |
| INDEX_ID                  | bigint(21)  | YES  |      | NULL    |       |
| INDEX_NAME                | varchar(64) | YES  |      | NULL    |       |
| EPOCH_CONF_VER            | bigint(21)  | YES  |      | NULL    |       |
| EPOCH_VERSION             | bigint(21)  | YES  |      | NULL    |       |
| WRITTEN_BYTES             | bigint(21)  | YES  |      | NULL    |       |
| READ_BYTES                | bigint(21)  | YES  |      | NULL    |       |
| APPROXIMATE_SIZE          | bigint(21)  | YES  |      | NULL    |       |
| APPROXIMATE_KEYS          | bigint(21)  | YES  |      | NULL    |       |
| REPLICATIONSTATUS_STATE   | varchar(64) | YES  |      | NULL    |       |
| REPLICATIONSTATUS_STATEID | bigint(21)  | YES  |      | NULL    |       |
+---------------------------+-------------+------+------+---------+-------+
17 行のセット (0.00 秒)
```

`TIKV_REGION_STATUS`テーブルの各列の説明は次のとおりです。

* `REGION_ID`: リージョンのID。
* `START_KEY`: リージョンの開始キーの値。
* `END_KEY`: リージョンの終了キーの値。
* `TABLE_ID`: リージョンが属するテーブルのID。
* `DB_NAME`: `TABLE_ID`が属するデータベースの名前。
* `TABLE_NAME`: リージョンが属するテーブルの名前。
* `IS_INDEX`: リージョンデータがインデックスであるかどうか。0はインデックスでないことを示し、1はインデックスであることを示します。現在のリージョンにテーブルデータとインデックスデータの両方が含まれている場合、複数の行がレコードされ、`IS_INDEX`はそれぞれ0と1です。
* `INDEX_ID`: リージョンが属するインデックスのID。`IS_INDEX`が0の場合、この列の値はNULLです。
* `INDEX_NAME`: リージョンが属するインデックスの名前。`IS_INDEX`が0の場合、この列の値はNULLです。
* `EPOCH_CONF_VER`: リージョン構成のバージョン番号。ピアが追加または削除されると、バージョン番号が増加します。
* `EPOCH_VERSION`: リージョンの現在のバージョン番号。リージョンが分割されるかマージされると、バージョン番号が増加します。
* `WRITTEN_BYTES`: リージョンに書き込まれたデータ（バイト単位）の量。
* `READ_BYTES`: リージョンから読み取られたデータ（バイト単位）の量。
* `APPROXIMATE_SIZE`: リージョンのおおよそのデータサイズ（MB）。
* `APPROXIMATE_KEYS`: リージョン内のおおよそのキー数。
* `REPLICATIONSTATUS_STATE`: リージョンの現在のレプリケーション状態。状態は`UNKNOWN`、`SIMPLE_MAJORITY`、または`INTEGRITY_OVER_LABEL`のいずれかです。
* `REPLICATIONSTATUS_STATEID`: `REPLICATIONSTATUS_STATE`に対応する識別子。

また、`EPOCH_CONF_VER`、`WRITTEN_BYTES`、`READ_BYTES`の列に対して`EPOCH_CONF_VER`、`WRITTEN_BYTES`、`READ_BYTES`の順で`ORDER BY X LIMIT Y`操作を使用して、pd-ctlで`top confver`、`top read`、`top write`の操作を実装できます。

以下のSQL文を使用して、書き込みデータが最も多いトップ3のリージョンをクエリできます。

```sql
SELECT * FROM tikv_region_status ORDER BY written_bytes DESC LIMIT 3;
```