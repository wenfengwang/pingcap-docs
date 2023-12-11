---
title: CLIENT_ERRORS_SUMMARY_BY_USER
summary: `CLIENT_ERRORS_SUMMARY_BY_USER` INFORMATION_SCHEMAテーブルについて学びます。

# CLIENT_ERRORS_SUMMARY_BY_USER

`CLIENT_ERRORS_SUMMARY_BY_USER`テーブルは、TiDBサーバに接続するクライアントに返されたSQLエラーや警告の要約を提供します。これには、次のものが含まれます。

* 書式が誤ったSQL文。
* ゼロによる除算エラー。
* 範囲外または重複したキー値の挿入を試みること。
* 権限エラー。
* 存在しないテーブル。

クライアントエラーは、MySQLサーバプロトコルを介してクライアントに返され、アプリケーションが適切な対応を取ることが期待されています。 `INFORMATION_SCHEMA.CLIENT_ERRORS_SUMMARY_BY_USER`テーブルは、TiDBサーバによって返されたエラーがアプリケーションによって適切に処理（または記録）されていないシナリオを検査するための便利な方法を提供します。

`CLIENT_ERRORS_SUMMARY_BY_USER`はユーザ単位でエラーを要約するため、あるユーザサーバが他のサーバよりも多くのエラーを生成しているシナリオを診断するのに役立ちます。可能なシナリオには次のものがあります。

* 権限エラー。
* 不足しているテーブル、またはリレーションオブジェクト。
* 不正確なSQL構文、またはアプリケーションとTiDBのバージョン間の非互換性。

要約されたカウントは`FLUSH CLIENT_ERRORS_SUMMARY`ステートメントでリセットできます。この要約は各TiDBサーバに対してローカルであり、メモリにのみ保持されます。 TiDBサーバが再起動されると要約は失われます。

```sql
USE INFORMATION_SCHEMA;
DESC CLIENT_ERRORS_SUMMARY_BY_USER;
```

出力は次のようになります。

```sql
+---------------+---------------+------+------+---------+-------+
| Field         | Type          | Null | Key  | Default | Extra |
+---------------+---------------+------+------+---------+-------+
| USER          | varchar(64)   | NO   |      | NULL    |       |
| ERROR_NUMBER  | bigint(64)    | NO   |      | NULL    |       |
| ERROR_MESSAGE | varchar(1024) | NO   |      | NULL    |       |
| ERROR_COUNT   | bigint(64)    | NO   |      | NULL    |       |
| WARNING_COUNT | bigint(64)    | NO   |      | NULL    |       |
| FIRST_SEEN    | timestamp     | YES  |      | NULL    |       |
| LAST_SEEN     | timestamp     | YES  |      | NULL    |       |
+---------------+---------------+------+------+---------+-------+
7 rows in set (0.00 sec)
```

フィールドの説明：

* `USER`：認証されたユーザ。
* `ERROR_NUMBER`：返されたMySQL互換のエラー番号。
* `ERROR_MESSAGE`：エラーメッセージは、エラー番号に一致します（プリペアドステートメント形式）。
* `ERROR_COUNT`：このエラーがユーザに返された回数。
* `WARNING_COUNT`：この警告がユーザに返された回数。
* `FIRST_SEEN`：このエラー（または警告）がユーザに送信された最初の時間。
* `LAST_SEEN`：このエラー（または警告）がユーザに最近送信された時間。

次の例は、クライアントがローカルTiDBサーバに接続したときに警告が生成されることを示しています。 要約は`FLUSH CLIENT_ERRORS_SUMMARY`を実行した後にリセットされます。

```sql
SELECT 0/0;
SELECT * FROM CLIENT_ERRORS_SUMMARY_BY_USER;
FLUSH CLIENT_ERRORS_SUMMARY;
SELECT * FROM CLIENT_ERRORS_SUMMARY_BY_USER;
```

出力は次のようになります。

```sql
+-----+
| 0/0 |
+-----+
| NULL |
+-----+
1行がセットされ、1つの警告があります（0.00秒）

+------+--------------+---------------+-------------+---------------+---------------------+---------------------+
| USER | ERROR_NUMBER | ERROR_MESSAGE | ERROR_COUNT | WARNING_COUNT | FIRST_SEEN          | LAST_SEEN           |
+------+--------------+---------------+-------------+---------------+---------------------+---------------------+
| root |         1365 | ゼロによる除算 |           0 |             1 | 2021-03-18 13:05:36 | 2021-03-18 13:05:36 |
+------+--------------+---------------+-------------+---------------+---------------------+---------------------+
1行がセットされました（0.00秒）

クエリはOKで、0行が影響を受けませんでした（0.00秒）

空のセット（0.00秒）
```