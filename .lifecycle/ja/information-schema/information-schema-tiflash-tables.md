---
title: TIFLASH_TABLES
summary: `TIFLASH_TABLES`のinformation_schemaテーブルについて学びます。

# TIFLASH_TABLES

> **警告:**
>
> このテーブルは本番環境で使用しないでください。テーブルのフィールドが不安定であり、事前の予告なしにTiDBの新しいリリースで変更される可能性があります。

> **注意:**
>
> このテーブルは[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスターでは利用できません。

`TIFLASH_TABLES`テーブルはTiFlash内のデータテーブルに関する統計情報を提供します。

```sql
USE information_schema;
DESC tiflash_tables;
```

```sql
+-------------------------------------------+--------------+------+------+---------+-------+
| Field                                     | Type         | Null | Key  | Default | Extra |
+-------------------------------------------+--------------+------+------+---------+-------+
| DATABASE                                  | varchar(64)  | YES  |      | NULL    |       |
| TABLE                                     | varchar(64)  | YES  |      | NULL    |       |
| TIDB_DATABASE                             | varchar(64)  | YES  |      | NULL    |       |
| TIDB_TABLE                                | varchar(64)  | YES  |      | NULL    |       |
| TABLE_ID                                  | bigint(64)   | YES  |      | NULL    |       |
| IS_TOMBSTONE                              | bigint(64)   | YES  |      | NULL    |       |
| SEGMENT_COUNT                             | bigint(64)   | YES  |      | NULL    |       |
| TOTAL_ROWS                                | bigint(64)   | YES  |      | NULL    |       |
| TOTAL_SIZE                                | bigint(64)   | YES  |      | NULL    |       |
| TOTAL_DELETE_RANGES                       | bigint(64)   | YES  |      | NULL    |       |
| DELTA_RATE_ROWS                           | double       | YES  |      | NULL    |       |
| DELTA_RATE_SEGMENTS                       | double       | YES  |      | NULL    |       |
| DELTA_PLACED_RATE                         | double       | YES  |      | NULL    |       |
| DELTA_CACHE_SIZE                          | bigint(64)   | YES  |      | NULL    |       |
| DELTA_CACHE_RATE                          | double       | YES  |      | NULL    |       |
| DELTA_CACHE_WASTED_RATE                   | double       | YES  |      | NULL    |       |
| DELTA_INDEX_SIZE                          | bigint(64)   | YES  |      | NULL    |       |
| AVG_SEGMENT_ROWS                          | double       | YES  |      | NULL    |       |
| AVG_SEGMENT_SIZE                          | double       | YES  |      | NULL    |       |
| DELTA_COUNT                               | bigint(64)   | YES  |      | NULL    |       |
| TOTAL_DELTA_ROWS                          | bigint(64)   | YES  |      | NULL    |       |
| TOTAL_DELTA_SIZE                          | bigint(64)   | YES  |      | NULL    |       |
| AVG_DELTA_ROWS                            | double       | YES  |      | NULL    |       |
| AVG_DELTA_SIZE                            | double       | YES  |      | NULL    |       |
| AVG_DELTA_DELETE_RANGES                   | double       | YES  |      | NULL    |       |
| STABLE_COUNT                              | bigint(64)   | YES  |      | NULL    |       |
| TOTAL_STABLE_ROWS                         | bigint(64)   | YES  |      | NULL    |       |
| TOTAL_STABLE_SIZE                         | bigint(64)   | YES  |      | NULL    |       |
| TOTAL_STABLE_SIZE_ON_DISK                 | bigint(64)   | YES  |      | NULL    |       |
| AVG_STABLE_ROWS                           | double       | YES  |      | NULL    |       |
| AVG_STABLE_SIZE                           | double       | YES  |      | NULL    |       |
| TOTAL_PACK_COUNT_IN_DELTA                 | bigint(64)   | YES  |      | NULL    |       |
| MAX_PACK_COUNT_IN_DELTA                   | bigint(64)   | YES  |      | NULL    |       |
| AVG_PACK_COUNT_IN_DELTA                   | double       | YES  |      | NULL    |       |
| AVG_PACK_ROWS_IN_DELTA                    | double       | YES  |      | NULL    |       |
| AVG_PACK_SIZE_IN_DELTA                    | double       | YES  |      | NULL    |       |
| TOTAL_PACK_COUNT_IN_STABLE                | bigint(64)   | YES  |      | NULL    |       |
| AVG_PACK_COUNT_IN_STABLE                  | double       | YES  |      | NULL    |       |
| AVG_PACK_ROWS_IN_STABLE                   | double       | YES  |      | NULL    |       |
| AVG_PACK_SIZE_IN_STABLE                   | double       | YES  |      | NULL    |       |
| STORAGE_STABLE_NUM_SNAPSHOTS              | bigint(64)   | YES  |      | NULL    |       |
| STORAGE_STABLE_OLDEST_SNAPSHOT_LIFETIME   | double       | YES  |      | NULL    |       |
| STORAGE_STABLE_OLDEST_SNAPSHOT_THREAD_ID  | bigint(64)   | YES  |      | NULL    |       |
| STORAGE_STABLE_OLDEST_SNAPSHOT_TRACING_ID | varchar(128) | YES  |      | NULL    |       |
| STORAGE_DELTA_NUM_SNAPSHOTS               | bigint(64)   | YES  |      | NULL    |       |
| STORAGE_DELTA_OLDEST_SNAPSHOT_LIFETIME    | double       | YES  |      | NULL    |       |
| STORAGE_DELTA_OLDEST_SNAPSHOT_THREAD_ID   | bigint(64)   | YES  |      | NULL    |       |
| STORAGE_DELTA_OLDEST_SNAPSHOT_TRACING_ID  | varchar(128) | YES  |      | NULL    |       |
| STORAGE_META_NUM_SNAPSHOTS                | bigint(64)   | YES  |      | NULL    |       |
| STORAGE_META_OLDEST_SNAPSHOT_LIFETIME     | double       | YES  |      | NULL    |       |
| STORAGE_META_OLDEST_SNAPSHOT_THREAD_ID    | bigint(64)   | YES  |      | NULL    |       |
| STORAGE_META_OLDEST_SNAPSHOT_TRACING_ID   | varchar(128) | YES  |      | NULL    |       |
| BACKGROUND_TASKS_LENGTH                   | bigint(64)   | YES  |      | NULL    |       |
| TIFLASH_INSTANCE                          | varchar(64)  | YES  |      | NULL    |       |
+-------------------------------------------+--------------+------+------+---------+-------+
54 rows in set (0.00 sec)
```

`TIFLASH_TABLES`テーブルのフィールドは以下の通りです。

- `DATABASE`: TiFlash内のテーブルが属するデータベースの名前。
- `TABLE`: TiFlash内のテーブルの名前。
- `TIDB_DATABASE`: TiDB内のテーブルが属するデータベースの名前。
- `TIDB_TABLE`: TiDB内のテーブルの名前。
- `TABLE_ID`: テーブルの内部IDであり、TiDBクラスター内で一意です。
- `IS_TOMBSTONE`: テーブルをリサイクルできるかを示します。`1`はテーブルをリサイクルできることを示し、`0`はテーブルが正常な状態であることを示します。
- `SEGMENT_COUNT`: テーブル内のセグメントの数。セグメントはTiFlashのデータ管理単位です。
- `TOTAL_ROWS`: テーブル内の総行数。
- `TOTAL_SIZE`: テーブルの総サイズ（バイト単位）。
- `TOTAL_DELETE_RANGES`: テーブル内の削除範囲の総数。
- `DELTA_RATE_ROWS`: テーブルのDeltaレイヤー内の総行数に対するテーブルの総行数の比率。
- `DELTA_RATE_SEGMENTS`: テーブル内で非空のDeltaレイヤーを含むセグメントの割合。
- `DELTA_PLACED_RATE`: テーブルのDeltaレイヤーでインデックス構築が完了した行の割合。
- `DELTA_CACHE_SIZE`: テーブルのDeltaレイヤー内のキャッシュのサイズ（バイト単位）。
- `DELTA_CACHE_RATE`: テーブルのDeltaレイヤー内のキャッシュデータの割合。
- `DELTA_CACHE_WASTED_RATE`: テーブルのDeltaレイヤー内の無効なキャッシュデータの割合。
- `DELTA_INDEX_SIZE`: Deltaレイヤー内のインデックスが占めるメモリのサイズ（バイト単位）。
- `AVG_SEGMENT_ROWS`: テーブルの全セグメントに含まれる行の平均数。
- `AVG_SEGMENT_SIZE`: テーブルの全セグメントの平均サイズ（バイト単位）。
- `DELTA_COUNT`: テーブル内で非空のDeltaレイヤーを含むセグメントの数。
- `TOTAL_DELTA_ROWS`: Deltaレイヤー内の総行数。
- `TOTAL_DELTA_SIZE`: Deltaレイヤー内のデータの総サイズ（バイト単位）。
- `AVG_DELTA_ROWS`: 全Deltaレイヤーのデータの平均行数。
- `AVG_DELTA_SIZE`: 全Deltaレイヤーのデータの平均サイズ（バイト単位）。
- `AVG_DELTA_DELETE_RANGES`: 全Deltaレイヤーの削除範囲操作の平均数。
- `STABLE_COUNT`: テーブル内で非空のStableレイヤーを含むセグメントの数。
- `TOTAL_STABLE_ROWS`: すべてのStableレイヤー内の総行数。
- `TOTAL_STABLE_SIZE`: すべての安定層におけるデータの合計サイズ（バイト単位）。
- `TOTAL_STABLE_SIZE_ON_DISK`: すべての安定層におけるデータのディスク上の使用スペース（バイト単位）。
- `AVG_STABLE_ROWS`: すべての安定層におけるデータの平均行数。
- `AVG_STABLE_SIZE`: すべての安定層におけるデータの平均サイズ（バイト単位）。
- `TOTAL_PACK_COUNT_IN_DELTA`: すべてのデルタ層における列ファイルの合計数。
- `MAX_PACK_COUNT_IN_DELTA`: 単一のデルタ層における列ファイルの最大数。
- `AVG_PACK_COUNT_IN_DELTA`: すべてのデルタ層における列ファイルの平均数。
- `AVG_PACK_ROWS_IN_DELTA`: すべてのデルタ層におけるすべての列ファイルの平均行数。
- `AVG_PACK_SIZE_IN_DELTA`: すべてのデルタ層におけるすべての列ファイルのデータの平均サイズ（バイト単位）。
- `TOTAL_PACK_COUNT_IN_STABLE`: すべての安定層におけるパックの合計数。
- `AVG_PACK_COUNT_IN_STABLE`: すべての安定層におけるパックの平均数。
- `AVG_PACK_ROWS_IN_STABLE`: すべての安定層におけるすべてのパックの平均行数。
- `AVG_PACK_SIZE_IN_STABLE`: 安定層におけるすべてのパックのデータの平均サイズ（バイト単位）。
- `STORAGE_STABLE_NUM_SNAPSHOTS`: 安定層におけるスナップショットの数。
- `STORAGE_STABLE_OLDEST_SNAPSHOT_LIFETIME`: 安定層における最も古いスナップショットの期間（秒単位）。
- `STORAGE_STABLE_OLDEST_SNAPSHOT_THREAD_ID`: 安定層における最も古いスナップショットのスレッドID。
- `STORAGE_STABLE_OLDEST_SNAPSHOT_TRACING_ID`: 安定層における最も古いスナップショットのトレースID。
- `STORAGE_DELTA_NUM_SNAPSHOTS`: デルタ層におけるスナップショットの数。
- `STORAGE_DELTA_OLDEST_SNAPSHOT_LIFETIME`: デルタ層における最も古いスナップショットの期間（秒単位）。
- `STORAGE_DELTA_OLDEST_SNAPSHOT_THREAD_ID`: デルタ層における最も古いスナップショットのスレッドID。
- `STORAGE_DELTA_OLDEST_SNAPSHOT_TRACING_ID`: デルタ層における最も古いスナップショットのトレースID。
- `STORAGE_META_NUM_SNAPSHOTS`: メタ情報におけるスナップショットの数。
- `STORAGE_META_OLDEST_SNAPSHOT_LIFETIME`: メタ情報における最も古いスナップショットの期間（秒単位）。
- `STORAGE_META_OLDEST_SNAPSHOT_THREAD_ID`: メタ情報における最も古いスナップショットのスレッドID。
- `STORAGE_META_OLDEST_SNAPSHOT_TRACING_ID`: メタ情報における最も古いスナップショットのトレースID。
- `BACKGROUND_TASKS_LENGTH`: バックグラウンドにおけるタスクキューの長さ。
- `TIFLASH_INSTANCE`: TiFlashインスタンスのアドレス。