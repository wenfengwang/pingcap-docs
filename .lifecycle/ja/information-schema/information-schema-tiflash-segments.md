---
title: TIFLASH_SEGMENTS
summary: `TIFLASH_SEGMENTS` の information_schema テーブルの情報を学びます。

# TIFLASH_SEGMENTS

> **警告:**
>
> このテーブルはプロダクション環境で使用しないでください。テーブルのフィールドは安定しておらず、新しいリリースでは予告なく変更される可能性があります。

> **注意:**
>
> このテーブルは [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) クラスターでは利用できません。

`TIFLASH_SEGMENTS` テーブルは、TiFlash 内のデータテーブルに関する統計情報を提供します。

```sql
USE information_schema;
DESC tiflash_segments;
```

```sql
+-------------------------------+-------------+------+------+---------+-------+
| Field                         | Type        | Null | Key  | Default | Extra |
+-------------------------------+-------------+------+------+---------+-------+
| DATABASE                      | varchar(64) | YES  |      | NULL    |       |
| TABLE                         | varchar(64) | YES  |      | NULL    |       |
| TIDB_DATABASE                 | varchar(64) | YES  |      | NULL    |       |
| TIDB_TABLE                    | varchar(64) | YES  |      | NULL    |       |
| TABLE_ID                      | bigint(64)  | YES  |      | NULL    |       |
| IS_TOMBSTONE                  | bigint(64)  | YES  |      | NULL    |       |
| SEGMENT_ID                    | bigint(64)  | YES  |      | NULL    |       |
| RANGE                         | varchar(64) | YES  |      | NULL    |       |
| EPOCH                         | bigint(64)  | YES  |      | NULL    |       |
| ROWS                          | bigint(64)  | YES  |      | NULL    |       |
| SIZE                          | bigint(64)  | YES  |      | NULL    |       |
| DELTA_RATE                    | double      | YES  |      | NULL    |       |
| DELTA_MEMTABLE_ROWS           | bigint(64)  | YES  |      | NULL    |       |
| DELTA_MEMTABLE_SIZE           | bigint(64)  | YES  |      | NULL    |       |
| DELTA_MEMTABLE_COLUMN_FILES   | bigint(64)  | YES  |      | NULL    |       |
| DELTA_MEMTABLE_DELETE_RANGES  | bigint(64)  | YES  |      | NULL    |       |
| DELTA_PERSISTED_PAGE_ID       | bigint(64)  | YES  |      | NULL    |       |
| DELTA_PERSISTED_ROWS          | bigint(64)  | YES  |      | NULL    |       |
| DELTA_PERSISTED_SIZE          | bigint(64)  | YES  |      | NULL    |       |
| DELTA_PERSISTED_COLUMN_FILES  | bigint(64)  | YES  |      | NULL    |       |
| DELTA_PERSISTED_DELETE_RANGES | bigint(64)  | YES  |      | NULL    |       |
| DELTA_CACHE_SIZE              | bigint(64)  | YES  |      | NULL    |       |
| DELTA_INDEX_SIZE              | bigint(64)  | YES  |      | NULL    |       |
| STABLE_PAGE_ID                | bigint(64)  | YES  |      | NULL    |       |
| STABLE_ROWS                   | bigint(64)  | YES  |      | NULL    |       |
| STABLE_SIZE                   | bigint(64)  | YES  |      | NULL    |       |
| STABLE_DMFILES                | bigint(64)  | YES  |      | NULL    |       |
| STABLE_DMFILES_ID_0           | bigint(64)  | YES  |      | NULL    |       |
| STABLE_DMFILES_ROWS           | bigint(64)  | YES  |      | NULL    |       |
| STABLE_DMFILES_SIZE           | bigint(64)  | YES  |      | NULL    |       |
| STABLE_DMFILES_SIZE_ON_DISK   | bigint(64)  | YES  |      | NULL    |       |
| STABLE_DMFILES_PACKS          | bigint(64)  | YES  |      | NULL    |       |
| TIFLASH_INSTANCE              | varchar(64) | YES  |      | NULL    |       |
+-------------------------------+-------------+------+------+---------+-------+
33 行が返されました (0.00 秒)
```

`TIFLASH_SEGMENTS` テーブルのフィールドは以下のように説明されます：

- `DATABASE`: TiFlash 内のデータベース名。セグメントはこのデータベース内のテーブルに属しています。
- `TABLE`: TiFlash 内のテーブル名。セグメントはこのテーブルに属しています。
- `TIDB_DATABASE`: TiDB 内のデータベース名。セグメントはこのデータベース内のテーブルに属しています。
- `TIDB_TABLE`: TiDB 内のテーブル名。セグメントはこのテーブルに属しています。
- `TABLE_ID`: セグメントが属するテーブルの内部 ID。この ID は TiDB クラスター内で一意です。
- `IS_TOMBSTONE`: セグメントが属するテーブルがリサイクル可能かどうかを示します。`1` はテーブルがリサイクル可能であることを示し、`0` はテーブルが通常の状態であることを示します。
- `SEGMENT_ID`: セグメント ID、テーブル内で一意です。
- `RANGE`: セグメントが含むデータの範囲。
- `EPOCH`: セグメントの更新バージョン。各セグメントのバージョン番号は単調に増加します。
- `ROWS`: セグメント内の総行数。
- `SIZE`: セグメントデータの総サイズ（バイト単位）。
- `DELTA_RATE`: Delta レイヤ内の総行数とセグメント内の総行数の比率。
- `DELTA_MEMTABLE_ROWS`: Delta レイヤにキャッシュされた総行数。
- `DELTA_MEMTABLE_SIZE`: Delta レイヤにキャッシュされたデータの総サイズ（バイト単位）。
- `DELTA_MEMTABLE_COLUMN_FILES`: Delta レイヤにキャッシュされた Column Files の数。
- `DELTA_MEMTABLE_DELETE_RANGES`: Delta レイヤにキャッシュされた Delete Ranges の数。
- `DELTA_PERSISTED_PAGE_ID`: Delta レイヤのディスク上に保存されたデータの ID。
- `DELTA_PERSISTED_ROWS`: Delta レイヤの永続データの総行数。
- `DELTA_PERSISTED_SIZE`: Delta レイヤの永続データの総サイズ（バイト単位）。
- `DELTA_PERSISTED_COLUMN_FILES`: Delta レイヤの永続 Column Files の数。
- `DELTA_PERSISTED_DELETE_RANGES`: Delta レイヤの永続 Delete Ranges の数。
- `DELTA_CACHE_SIZE`: Delta レイヤ内のキャッシュサイズ（バイト単位）。
- `DELTA_INDEX_SIZE`: Delta レイヤ内のインデックスのサイズ（バイト単位）。
- `STABLE_PAGE_ID`: Stable レイヤのデータのディスク保存 ID。
- `STABLE_ROWS`: Stable レイヤの総行数。
- `STABLE_SIZE`: Stable レイヤのデータの総サイズ（バイト単位）。
- `STABLE_DMFILES`: Stable レイヤの DMFiles の数。
- `STABLE_DMFILES_ID_0`: Stable レイヤの最初の DMFile のディスク保存 ID。
- `STABLE_DMFILES_ROWS`: Stable レイヤの DMFile の総行数。
- `STABLE_DMFILES_SIZE`: Stable レイヤの DMFile のデータの総サイズ（バイト単位）。
- `STABLE_DMFILES_SIZE_ON_DISK`: Stable レイヤ内の DMFile が占有するディスク容量（バイト単位）。
- `STABLE_DMFILES_PACKS`: Stable レイヤの DMFile のパック数。
- `TIFLASH_INSTANCE`: TiFlash インスタンスのアドレス。