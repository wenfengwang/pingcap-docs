---
title: TIDB_TRX
summary: `TIDB_TRX` INFORMATION_SCHEMAのテーブルについて学びます。

# TIDB_TRX

`TIDB_TRX`テーブルは、TiDBノードで現在実行中のトランザクションに関する情報を提供します。

```sql
USE INFORMATION_SCHEMA;
DESC TIDB_TRX;
```

出力は以下の通りです:

```sql
+-------------------------+-----------------------------------------------------------------+------+------+---------+-------+
| Field                   | Type                                                            | Null | Key  | Default | Extra |
+-------------------------+-----------------------------------------------------------------+------+------+---------+-------+
| ID                      | bigint(21) unsigned                                             | NO   | PRI  | NULL    |       |
| START_TIME              | timestamp(6)                                                    | YES  |      | NULL    |       |
| CURRENT_SQL_DIGEST      | varchar(64)                                                     | YES  |      | NULL    |       |
| CURRENT_SQL_DIGEST_TEXT | text                                                            | YES  |      | NULL    |       |
| STATE                   | enum('Idle','Running','LockWaiting','Committing','RollingBack') | YES  |      | NULL    |       |
| WAITING_START_TIME      | timestamp(6)                                                    | YES  |      | NULL    |       |
| MEM_BUFFER_KEYS         | bigint(64)                                                      | YES  |      | NULL    |       |
| MEM_BUFFER_BYTES        | bigint(64)                                                      | YES  |      | NULL    |       |
| SESSION_ID              | bigint(21) unsigned                                             | YES  |      | NULL    |       |
| USER                    | varchar(16)                                                     | YES  |      | NULL    |       |
| DB                      | varchar(64)                                                     | YES  |      | NULL    |       |
| ALL_SQL_DIGESTS         | text                                                            | YES  |      | NULL    |       |
| RELATED_TABLE_IDS       | text                                                            | YES  |      | NULL    |       |
+-------------------------+-----------------------------------------------------------------+------+------+---------+-------+
```

`TIDB_TRX`テーブルの各列フィールドの意味は以下の通りです:

* `ID`: トランザクションID。これはトランザクションの`start_ts`（開始タイムスタンプ）です。
* `START_TIME`: トランザクションの開始時刻。これはトランザクションの`start_ts`に対応する物理時刻です。
* `CURRENT_SQL_DIGEST`: トランザクションで現在実行中のSQL文のダイジェスト。
* `CURRENT_SQL_DIGEST_TEXT`: トランザクションによって現在実行されているSQL文の正規化された形式です。これは引数とフォーマットを除いたSQL文に対応します。`CURRENT_SQL_DIGEST`に対応します。
* `STATE`: トランザクションの現在の状態。可能な値は以下の通りです:
   * `Idle`: トランザクションはアイドル状態であり、つまりユーザーがクエリを入力するのを待っています。
   * `Running`: トランザクションはクエリを実行しています。
   * `LockWaiting`: トランザクションは悲観的ロックの取得を待っています。この状態になるのは、他のトランザクションによってブロックされているかどうかに関わらず、悲観的ロック操作の開始時です。
   * `Committing`: トランザクションはコミット処理中です。
   * `RollingBack`: トランザクションはロールバック中です。
* `WAITING_START_TIME`: `STATE`の値が`LockWaiting`の場合、この列は待機の開始時刻を示します。
* `MEM_BUFFER_KEYS`: 現在のトランザクションによってメモリバッファに書き込まれたキーの数。
* `MEM_BUFFER_BYTES`: 現在のトランザクションによってメモリバッファに書き込まれたキー値の総数（バイト単位）。
* `SESSION_ID`: このトランザクションが属するセッションのID。
* `USER`: トランザクションを実行するユーザーの名前。
* `DB`: トランザクションが実行されているセッションの現在のデフォルトデータベース名。
* `ALL_SQL_DIGESTS`: トランザクションで実行されたステートメントのダイジェスト一覧。リストはJSON形式の文字列配列として表示されます。各トランザクションは最大で最初の50ステートメントを記録します。[`TIDB_DECODE_SQL_DIGESTS`](/functions-and-operators/tidb-functions.md#tidb_decode_sql_digests)関数を使用して、この列の情報を対応する正規化されたSQL文のリストに変換できます。
* `RELATED_TABLE_IDS`: トランザクションがアクセスするテーブル、ビュー、その他のオブジェクトのID。

> **注意:**
>
> * 完全な情報は[PROCESS](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_process)特権を持つユーザーのみが取得できます。PROCESS特権がないユーザーは、現在のユーザーによって実行されたトランザクションの情報のみをクエリできます。
> * `CURRENT_SQL_DIGEST`および`ALL_SQL_DIGESTS`列の情報（SQLダイジェスト）は、正規化されたSQL文から計算されたハッシュ値です。`CURRENT_SQL_DIGEST_TEXT`列の情報および`TIDB_DECODE_SQL_DIGESTS`関数から返される結果は、内部的にはステートメントサマリーテーブルからクエリされますので、対応するステートメントが内部的に見つからない可能性があります。詳細なSQLダイジェストおよびステートメントサマリーテーブルの説明については、[ステートメントサマリーテーブル](/statement-summary-tables.md)を参照してください。
> * [`TIDB_DECODE_SQL_DIGESTS`](/functions-and-operators/tidb-functions.md#tidb_decode_sql_digests)関数の呼び出しには高いオーバーヘッドがかかります。大量のトランザクションの履歴的なSQL文をクエリするためにこの関数を呼び出すと、クエリに時間がかかる可能性があります。大規模なクラスターで多数の並行トランザクションがある場合は、`TIDB_TRX`テーブルのフルテーブルを直接クエリして`ALL_SQL_DIGEST`列と`tidb_decode_sql_digests`関数を使用することを避けてください。つまり、`SELECT *、tidb_decode_sql_digests(all_sql_digests) FROM TIDB_TRX`のようなSQLステートメントを避けてください。
> * 現在、`TIDB_TRX`テーブルではTiDB内部トランザクションの情報を表示することはサポートされていません。

## 例

`TIDB_TRX`テーブルを表示します:

```sql
SELECT * FROM INFORMATION_SCHEMA.TIDB_TRX\G
```

出力は以下の通りです:

```sql
*************************** 1. 行 ***************************
                     ID: 426789913200689153
             START_TIME: 2021-08-04 10:51:54.883000
     CURRENT_SQL_DIGEST: NULL
CURRENT_SQL_DIGEST_TEXT: NULL
                  STATE: Idle
     WAITING_START_TIME: NULL
        MEM_BUFFER_KEYS: 1
       MEM_BUFFER_BYTES: 29
             SESSION_ID: 7
                   USER: root
                     DB: test
        ALL_SQL_DIGESTS: ["e6f07d43b5c21db0fbb9a31feac2dc599787763393dd5acbfad80e247eb02ad5","04fa858fa491c62d194faec2ab427261cc7998b3f1ccf8f6844febca504cb5e9","b83710fa8ab7df8504920e8569e48654f621cf828afbe7527fd003b79f48da9e"]
*************************** 2. 行 ***************************
                     ID: 426789921471332353
             START_TIME: 2021-08-04 10:52:26.433000
     CURRENT_SQL_DIGEST: 38b03afa5debbdf0326a014dbe5012a62c51957f1982b3093e748460f8b00821
CURRENT_SQL_DIGEST_TEXT: update `t` set `v` = `v` + ? where `id` = ?
                  STATE: LockWaiting
     WAITING_START_TIME: 2021-08-04 10:52:35.106568
        MEM_BUFFER_KEYS: 0
       MEM_BUFFER_BYTES: 0
             SESSION_ID: 9
                   USER: root
                     DB: test
        ALL_SQL_DIGESTS: ["e6f07d43b5c21db0fbb9a31feac2dc599787763393dd5acbfad80e247eb02ad5","38b03afa5debbdf0326a014dbe5012a62c51957f1982b3093e748460f8b00821"]
2 行であと (0.01 秒)
```

この例のクエリ結果から、現在のノードには2つの実行中のトランザクションがあることがわかります。1つのトランザクションはアイドル状態です（`STATE`が`Idle`で`CURRENT_SQL_DIGEST`が`NULL`）し、このトランザクションは3つ（`ALL_SQL_DIGESTS`リストに3つのレコードがあり、これは実行された3つのSQLステートメントのダイジェストです）のステートメントを実行しました。もう1つのトランザクションはステートメントを実行してロックを待っています（`STATE`が`LockWaiting`で`WAITING_START_TIME`は待機ロックの開始時刻を示します）。このトランザクションは2つのステートメントを実行し、現在実行中のステートメントは``"update `t` set `v` = `v` + ? where `id` = ?"``の形式です。

```sql
SELECT id, all_sql_digests, tidb_decode_sql_digests(all_sql_digests) AS all_sqls FROM INFORMATION_SCHEMA.TIDB_TRX\G
```

出力は以下の通りです:

```sql
*************************** 1. 行 ***************************
             id: 426789913200689153
all_sql_digests: ["e6f07d43b5c21db0fbb9a31feac2dc599787763393dd5acbfad80e247eb02ad5","04fa858fa491c62d194faec2ab427261cc7998b3f1ccf8f6844febca504cb5e9","b83710fa8ab7df8504920e8569e48654f621cf828afbe7527fd003b79f48da9e"]
       all_sqls: ["begin","insert into `t` values ( ... )","update `t` set `v` = `v` + ?"]
*************************** 2. row ***************************
             id: 426789921471332353
all_sql_digests: ["e6f07d43b5c21db0fbb9a31feac2dc599787763393dd5acbfad80e247eb02ad5","38b03afa5debbdf0326a014dbe5012a62c51957f1982b3093e748460f8b00821"]
       all_sqls: ["begin","update `t` set `v` = `v` + ? where `id` = ?"]
```

このクエリでは、`TIDB_TRX`テーブルの`ALL_SQL_DIGESTS`列に対して[`TIDB_DECODE_SQL_DIGESTS`](/functions-and-operators/tidb-functions.md#tidb_decode_sql_digests)関数を呼び出し、システムの内部クエリを通じてSQLダイジェスト配列を正規化されたSQLステートメントの配列に変換します。これにより、過去にトランザクションによって実行されたステートメントの情報を視覚的に取得できます。ただし、前述のクエリは`TIDB_TRX`テーブル全体をスキャンし、各行ごとに`TIDB_DECODE_SQL_DIGESTS`関数を呼び出します。`TIDB_DECODE_SQL_DIGESTS`関数の呼び出しには高いオーバーヘッドがかかります。したがって、クラスタ内に多くの同時トランザクションが存在する場合は、この種のクエリを避けるようにしてください。

## CLUSTER_TIDB_TRX

`TIDB_TRX`テーブルは、単一のTiDBノードで実行されているトランザクションに関する情報のみを提供します。クラスタ全体のすべてのTiDBノードで実行されているトランザクションの情報を表示したい場合は、`CLUSTER_TIDB_TRX`テーブルをクエリする必要があります。`TIDB_TRX`テーブルのクエリ結果と比較して、`CLUSTER_TIDB_TRX`テーブルのクエリ結果には追加の`INSTANCE`フィールドが含まれます。`INSTANCE`フィールドには、クラスタ内の各ノードのIPアドレスとポートが表示され、トランザクションが存在するTiDBノードを識別するために使用されます。

```sql
USE INFORMATION_SCHEMA;
DESC CLUSTER_TIDB_TRX;
```

出力は次のとおりです。

```sql
+-------------------------+-----------------------------------------------------------------+------+------+---------+-------+
| Field                   | Type                                                            | Null | Key  | Default | Extra |
+-------------------------+-----------------------------------------------------------------+------+------+---------+-------+
| INSTANCE                | varchar(64)                                                     | YES  |      | NULL    |       |
| ID                      | bigint(21) unsigned                                             | NO   | PRI  | NULL    |       |
| START_TIME              | timestamp(6)                                                    | YES  |      | NULL    |       |
| CURRENT_SQL_DIGEST      | varchar(64)                                                     | YES  |      | NULL    |       |
| CURRENT_SQL_DIGEST_TEXT | text                                                            | YES  |      | NULL    |       |
| STATE                   | enum('Idle','Running','LockWaiting','Committing','RollingBack') | YES  |      | NULL    |       |
| WAITING_START_TIME      | timestamp(6)                                                    | YES  |      | NULL    |       |
| MEM_BUFFER_KEYS         | bigint(64)                                                      | YES  |      | NULL    |       |
| MEM_BUFFER_BYTES        | bigint(64)                                                      | YES  |      | NULL    |       |
| SESSION_ID              | bigint(21) unsigned                                             | YES  |      | NULL    |       |
| USER                    | varchar(16)                                                     | YES  |      | NULL    |       |
| DB                      | varchar(64)                                                     | YES  |      | NULL    |       |
| ALL_SQL_DIGESTS         | text                                                            | YES  |      | NULL    |       |
| RELATED_TABLE_IDS       | text                                                            | YES  |      | NULL    |       |
+-------------------------+-----------------------------------------------------------------+------+------+---------+-------+
14 rows in set (0.00 sec)
```