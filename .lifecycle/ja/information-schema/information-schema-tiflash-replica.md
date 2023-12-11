---
title: TIFLASH_REPLICA
summary: `TIFLASH_REPLICA` INFORMATION_SCHEMA テーブルの情報を取得します。

# TIFLASH_REPLICA

`TIFLASH_REPLICA` テーブルは、TiFlash のレプリカに関する情報を提供します。

```sql
USE INFORMATION_SCHEMA;
DESC TIFLASH_REPLICA;
```

出力は次のとおりです：

```sql
+-----------------+-------------+------+------+---------+-------+
| Field           | Type        | Null | Key  | Default | Extra |
+-----------------+-------------+------+------+---------+-------+
| TABLE_SCHEMA    | varchar(64) | YES  |      | NULL    |       |
| TABLE_NAME      | varchar(64) | YES  |      | NULL    |       |
| TABLE_ID        | bigint(21)  | YES  |      | NULL    |       |
| REPLICA_COUNT   | bigint(64)  | YES  |      | NULL    |       |
| LOCATION_LABELS | varchar(64) | YES  |      | NULL    |       |
| AVAILABLE       | tinyint(1)  | YES  |      | NULL    |       |
| PROGRESS        | double      | YES  |      | NULL    |       |
+-----------------+-------------+------+------+---------+-------+
7 行が返されました (0.01 秒)
```

`TIFLASH_REPLICA` テーブルのフィールドは以下のように説明されています：

- `TABLE_SCHEMA`: テーブルが属するデータベースの名前。
- `TABLE_NAME`: テーブルの名前。
- `TABLE_ID`: テーブルの内部IDであり、TiDBクラスタ内で一意です。
- `REPLICA_COUNT`: TiFlash レプリカの数。
- `LOCATION_LABELS`: TiFlash レプリカが作成されたときに設定された LocationLabelList。
- `AVAILABLE`: テーブルの TiFlash レプリカが利用可能かどうかを示します。値が `1`（利用可能）の場合、TiDB オプティマイザはクエリのコストに基づいてクエリを TiKV または TiFlash に適切にプッシュダウンできます。値が `0`（利用不可）の場合、TiDB はクエリを TiFlash にプッシュダウンしません。このフィールドの値が一度 `1`（利用可能）になると、それ以上変更されません。
- `PROGRESS`: TiFlash レプリカのレプリケーションの進行状況で、小数点以下2桁で分単位の精度があります。このフィールドの範囲は `[0, 1]` です。`AVAILABLE` フィールドが `1` で `PROGRESS` が 1 よりも小さい場合、TiFlash レプリカは TiKV よりも遅れており、TiFlash にプッシュダウンされたクエリはデータレプリケーションの待機時間のタイムアウトによって失敗する可能性があります。
