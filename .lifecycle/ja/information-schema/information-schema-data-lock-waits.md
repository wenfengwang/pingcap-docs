---
title: DATA_LOCK_WAITS
summary: `DATA_LOCK_WAITS`のinformation_schemaテーブルについて。

# DATA_LOCK_WAITS

`DATA_LOCK_WAITS`テーブルは、クラスター内のすべてのTiKVノードで実行中のロック待ち情報を表示します。この情報には、悲観的トランザクションのロック待ち情報と、ブロックされている楽観的トランザクションの情報が含まれます。

{{< copyable "sql" >}}

```sql
USE information_schema;
DESC data_lock_waits;
```

```sql
+------------------------+---------------------+------+------+---------+-------+
| Field                  | Type                | Null | Key  | Default | Extra |
+------------------------+---------------------+------+------+---------+-------+
| KEY                    | text                | NO   |      | NULL    |       |
| KEY_INFO               | text                | YES  |      | NULL    |       |
| TRX_ID                 | bigint(21) unsigned | NO   |      | NULL    |       |
| CURRENT_HOLDING_TRX_ID | bigint(21) unsigned | NO   |      | NULL    |       |
| SQL_DIGEST             | varchar(64)         | YES  |      | NULL    |       |
| SQL_DIGEST_TEXT        | text                | YES  |      | NULL    |       |
+------------------------+---------------------+------+------+---------+-------+
```

`DATA_LOCK_WAITS`テーブルのそれぞれの列フィールドの意味は次の通りです:

* `KEY`: ロックを待っている16進数形式のキー。
* `KEY_INFO`: `KEY`の詳細情報。[KEY_INFO](#key_info)セクションを参照してください。
* `TRX_ID`: ロックを待っているトランザクションのID。このIDはトランザクションの`start_ts`でもあります。
* `CURRENT_HOLDING_TRX_ID`: 現在そのロックを保持しているトランザクションのID。このIDはトランザクションの`start_ts`でもあります。
* `SQL_DIGEST`: ロック待ちトランザクションで現在ブロックされているSQLステートメントのダイジェスト。
* `SQL_DIGEST_TEXT`: ロック待ちトランザクションで現在ブロックされている正規化されたSQLステートメント（引数とフォーマットのないSQLステートメント）です。これは`SQL_DIGEST`に対応しています。

> **警告:**
>
> * [PROCESS](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_process)権限を持つユーザーのみ、このテーブルをクエリできます。
> * 現在、楽観的トランザクションの場合、`SQL_DIGEST`および`SQL_DIGEST_TEXT`フィールドは`null`（利用不可）です。ブロッキングを引き起こすSQLステートメントを特定するためには、このテーブルを[`CLUSTER_TIDB_TRX`](/information-schema/information-schema-tidb-trx.md)と結合して楽観的トランザクションのすべてのSQLステートメントを取得することができます。 
> * `DATA_LOCK_WAITS`テーブルの情報はクエリ中にすべてのTiKVノードからリアルタイムで取得されます。現在、`WHERE`条件のあるクエリでも、情報収集は引き続きすべてのTiKVノードで実行されます。クラスターが大きく負荷が高い場合、このテーブルをクエリすると性能の乱れの潜在的リスクが発生する可能性があります。そのため、実際の状況に応じて使用してください。
> * 異なるTiKVノードからの情報は、同じ時点のスナップショットであることを保証されません。
> * `SQL_DIGEST`列の情報（SQLダイジェスト）は、正規化されたSQLステートメントから計算されたハッシュ値です。`SQL_DIGEST_TEXT`列の情報は内部的にステートメントサマリーテーブルからクエリされるため、対応するステートメントが内部で見つからない可能性があります。SQLダイジェストとステートメントサマリーテーブルの詳細な説明については、[ステートメントサマリーテーブル](/statement-summary-tables.md)を参照してください。

## `KEY_INFO`

`KEY_INFO`列には、`KEY`列の詳細情報が表示されます。情報はJSON形式で表示されます。各フィールドの説明は次のとおりです:

* `"db_id"`: キーが属するスキーマのID。
* `"db_name"`: キーが属するスキーマの名前。
* `"table_id"`: キーが属するテーブルのID。
* `"table_name"`: キーが属するテーブルの名前。
* `"partition_id"`: キーが存在するパーティションのID。
* `"partition_name"`: キーが存在するパーティションの名前。
* `"handle_type"`: 行キー（つまり、データの行を格納するキー）のハンドルタイプ。可能な値は次のとおりです:
   * `"int"`: ハンドルタイプがintで、つまりハンドルは行IDです。
   * `"common"`: ハンドルタイプがint64ではない。これは、クラスター化インデックスが有効なときに非intのプライマリキーで表示されます。
   * `"unknown"`: ハンドルタイプは現在サポートされていません。
* `"handle_value"`: ハンドル値。
* `"index_id"`: インデックスキー（インデックスを格納するキー）のインデックスID。
* `"index_name"`: インデックスキーが属するインデックスの名前。
* `"index_values"`: インデックスキーのインデックス値。

上記の各フィールドにおいて、フィールドの情報が適用されないか、現在利用不可の場合、そのフィールドはクエリ結果から省略されます。例えば、行キー情報には`index_id`、`index_name`、`index_values`が含まれず、インデックスキーには`handle_type`および`handle_value`が含まれません。非パーティションテーブルには`partition_id`、`partition_name`が表示されず、削除されたテーブルのキー情報には`table_name`、`db_id`、`db_name`、`index_name`のようなスキーマ情報を含まず、テーブルがパーティション化されたテーブルかどうかを区別することができません。

> **注意:**
>
> キーがパーティション設定されたテーブルから来た場合、クエリ中にキーが属するスキーマの情報がいくつかの理由で（例:キーが属するテーブルが削除されたため）クエリできない場合、`table_id`フィールドにキーが属するパーティションのIDが表示される可能性があります。これは、TiDBが異なるパーティションのキーを、いくつかの独立したテーブルのキーと同じ方法でエンコードするためです。したがって、スキーマ情報が欠落している場合、TiDBはそのキーがパーティションされていないテーブルに属するか、テーブルの1つのパーティションに属するかを確認できません。

## 例

{{< copyable "sql" >}}

```sql
select * from information_schema.data_lock_waits\G
```

```sql
*************************** 1. row ***************************
                   KEY: 7480000000000000355F728000000000000001
              KEY_INFO: {"db_id":1,"db_name":"test","table_id":53,"table_name":"t","handle_type":"int","handle_value":"1"}
                TRX_ID: 426790594290122753
CURRENT_HOLDING_TRX_ID: 426790590082449409
            SQL_DIGEST: 38b03afa5debbdf0326a014dbe5012a62c51957f1982b3093e748460f8b00821
       SQL_DIGEST_TEXT: update `t` set `v` = `v` + ? where `id` = ?
1 row in set (0.01 sec)
```

上記のクエリ結果は、ID `426790594290122753`のトランザクションが、ダイジェストが`"38b03afa5debbdf0326a014dbe5012a62c51957f1982b3093e748460f8b00821"`であるステートメントを実行し、形式が``update `t` set `v` = `v` + ? where `id` = ?``であるステートメントの悲観的ロックを得ようとしていること、ただし、このキーのロックはID `426790590082449409`のトランザクションによって保持されています。
```