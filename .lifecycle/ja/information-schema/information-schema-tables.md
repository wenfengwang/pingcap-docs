---
title: TABLES
summary: `TABLES`のinformation_schemaテーブルについて学ぶ。

# TABLES

`TABLES`テーブルはデータベース内のテーブルに関する情報を提供します：

{{< コピー可能 "sql" >}}

```sql
USE information_schema;
DESC tables;
```

```sql
+---------------------------+---------------+------+------+----------+-------+
| Field                     | Type          | Null | Key  | Default  | Extra |
+---------------------------+---------------+------+------+----------+-------+
| TABLE_CATALOG             | varchar(512)  | YES  |      | NULL     |       |
| TABLE_SCHEMA              | varchar(64)   | YES  |      | NULL     |       |
| TABLE_NAME                | varchar(64)   | YES  |      | NULL     |       |
| TABLE_TYPE                | varchar(64)   | YES  |      | NULL     |       |
| ENGINE                    | varchar(64)   | YES  |      | NULL     |       |
| VERSION                   | bigint(21)    | YES  |      | NULL     |       |
| ROW_FORMAT                | varchar(10)   | YES  |      | NULL     |       |
| TABLE_ROWS                | bigint(21)    | YES  |      | NULL     |       |
| AVG_ROW_LENGTH            | bigint(21)    | YES  |      | NULL     |       |
| DATA_LENGTH               | bigint(21)    | YES  |      | NULL     |       |
| MAX_DATA_LENGTH           | bigint(21)    | YES  |      | NULL     |       |
| INDEX_LENGTH              | bigint(21)    | YES  |      | NULL     |       |
| DATA_FREE                 | bigint(21)    | YES  |      | NULL     |       |
| AUTO_INCREMENT            | bigint(21)    | YES  |      | NULL     |       |
| CREATE_TIME               | datetime      | YES  |      | NULL     |       |
| UPDATE_TIME               | datetime      | YES  |      | NULL     |       |
| CHECK_TIME                | datetime      | YES  |      | NULL     |       |
| TABLE_COLLATION           | varchar(32)   | NO   |      | utf8_bin |       |
| CHECKSUM                  | bigint(21)    | YES  |      | NULL     |       |
| CREATE_OPTIONS            | varchar(255)  | YES  |      | NULL     |       |
| TABLE_COMMENT             | varchar(2048) | YES  |      | NULL     |       |
| TIDB_TABLE_ID             | bigint(21)    | YES  |      | NULL     |       |
| TIDB_ROW_ID_SHARDING_INFO | varchar(255)  | YES  |      | NULL     |       |
+---------------------------+---------------+------+------+----------+-------+
23 rows in set (0.00 sec)
```

{{< コピー可能 "sql" >}}

```sql
SELECT * FROM tables WHERE table_schema='mysql' AND table_name='user'\G
```

```sql
*************************** 1. row ***************************
            TABLE_CATALOG: def
             TABLE_SCHEMA: mysql
               TABLE_NAME: user
               TABLE_TYPE: BASE TABLE
                   ENGINE: InnoDB
                  VERSION: 10
               ROW_FORMAT: Compact
               TABLE_ROWS: 0
           AVG_ROW_LENGTH: 0
              DATA_LENGTH: 0
          MAX_DATA_LENGTH: 0
             INDEX_LENGTH: 0
                DATA_FREE: 0
           AUTO_INCREMENT: NULL
              CREATE_TIME: 2020-07-05 09:25:51
              UPDATE_TIME: NULL
               CHECK_TIME: NULL
          TABLE_COLLATION: utf8mb4_bin
                 CHECKSUM: NULL
           CREATE_OPTIONS: 
            TABLE_COMMENT: 
            TIDB_TABLE_ID: 5
TIDB_ROW_ID_SHARDING_INFO: NULL
1 row in set (0.00 sec)
```

以下のステートメントは同等です：

```sql
SELECT table_name FROM INFORMATION_SCHEMA.TABLES
  WHERE table_schema = 'db_name'
  [AND table_name LIKE 'wild']

SHOW TABLES
  FROM db_name
  [LIKE 'wild']
```

`TABLES`テーブルの列の説明は以下の通りです：

* `TABLE_CATALOG`: テーブルが属するカタログの名前。値は常に`def`です。
* `TABLE_SCHEMA`: テーブルが属するスキーマの名前です。
* `TABLE_NAME`: テーブルの名前です。
* `TABLE_TYPE`: テーブルのタイプです。
* `ENGINE`: ストレージエンジンの種類です。値は現在`InnoDB`です。
* `VERSION`: バージョンです。値はデフォルトで`10`です。
* `ROW_FORMAT`: 行フォーマットです。値は現在`Compact`です。
* `TABLE_ROWS`: 統計情報によるテーブル内の行数です。
* `AVG_ROW_LENGTH`: テーブルの平均行長です。`AVG_ROW_LENGTH` = `DATA_LENGTH` / `TABLE_ROWS`です。
* `DATA_LENGTH`: データの長さです。`DATA_LENGTH` = `TABLE_ROWS` \* タプル内の列のストレージ長の合計です。TiKVのレプリカは考慮されません。
* `MAX_DATA_LENGTH`: 最大データの長さです。値は現在`0`で、つまりデータ長さに上限がありません。
* `INDEX_LENGTH`: インデックスの長さです。`INDEX_LENGTH` = `TABLE_ROWS` \* インデックスタプル内の列の長さの合計です。TiKVのレプリカは考慮されません。
* `DATA_FREE`: データのフラグメントです。値は現在`0`です。
* `AUTO_INCREMENT`: オートインクリメントプライマリキーの現在のステップです。
* `CREATE_TIME`: テーブルが作成された時間です。
* `UPDATE_TIME`: テーブルが更新された時間です。
* `CHECK_TIME`: テーブルがチェックされた時間です。
* `TABLE_COLLATION`: テーブル内の文字列の照合順序です。
* `CHECKSUM`: チェックサムです。
* `CREATE_OPTIONS`: オプションを作成します。
* `TABLE_COMMENT`: テーブルのコメントやノートです。

テーブル内のほとんどの情報はMySQLと同じです。TiDBで新たに定義された列は2つだけです：

* `TIDB_TABLE_ID`: テーブルの内部IDを示します。このIDはTiDBクラスター内で一意です。
* `TIDB_ROW_ID_SHARDING_INFO`: テーブルのシャーディングタイプを示します。可能な値は次のとおりです：
    - `"NOT_SHARDED"`: テーブルはシャーディングされていません。
    - `"NOT_SHARDED(PK_IS_HANDLE)"`: 整数プライマリキーを行IDとして定義するテーブルはシャーディングされていません。
    - `"PK_AUTO_RANDOM_BITS={bit_number}"`: 整数プライマリキーを行IDとして定義し、プライマリキーに`AUTO_RANDOM`属性が付与されているため、テーブルはシャーディングされています。
    - `"SHARD_BITS={bit_number}"`: `SHARD_ROW_ID_BITS={bit_number}`を使用してテーブルがシャーディングされています。
    - NULL: テーブルはシステムテーブルまたはビューであり、そのためシャーディングされません。