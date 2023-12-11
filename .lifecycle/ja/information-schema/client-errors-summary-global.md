---
title: CLIENT_ERRORS_SUMMARY_GLOBAL
summary: `CLIENT_ERRORS_SUMMARY_GLOBAL`のINFORMATION_SCHEMAテーブルについて学ぶ。

# CLIENT_ERRORS_SUMMARY_GLOBAL

`CLIENT_ERRORS_SUMMARY_GLOBAL`テーブルは、TiDBサーバに接続するクライアントに返されたすべてのSQLエラーや警告のグローバルなサマリーを提供します。これには次のものが含まれます。

* 不正なSQL文。
* ゼロでの除算エラー。
* 範囲外または重複キー値の挿入を試みる。
* 権限エラー。
* テーブルが存在しない。

クライアントエラーは、MySQLサーバープロトコルを介してクライアントに返され、アプリケーションが適切なアクションを取ることが期待されています。`INFORMATION_SCHEMA.CLIENT_ERRORS_SUMMARY_GLOBAL`テーブルは高レベルの概要を提供し、TiDBサーバによって返されたエラーを正しく処理（または記録）していないシナリオで有用です。

サマリーのカウントは、`FLUSH CLIENT_ERRORS_SUMMARY`ステートメントでリセットできます。サマリーは各TiDBサーバにローカルであり、メモリ内にのみ保持されます。TiDBサーバが再起動されると、サマリーは失われます。

```sql
USE INFORMATION_SCHEMA;
DESC CLIENT_ERRORS_SUMMARY_GLOBAL;
```

出力は次のようになります。

```sql
+---------------+---------------+------+------+---------+-------+
| Field         | Type          | Null | Key  | Default | Extra |
+---------------+---------------+------+------+---------+-------+
| ERROR_NUMBER  | bigint(64)    | NO   |      | NULL    |       |
| ERROR_MESSAGE | varchar(1024) | NO   |      | NULL    |       |
| ERROR_COUNT   | bigint(64)    | NO   |      | NULL    |       |
| WARNING_COUNT | bigint(64)    | NO   |      | NULL    |       |
| FIRST_SEEN    | timestamp     | YES  |      | NULL    |       |
| LAST_SEEN     | timestamp     | YES  |      | NULL    |       |
+---------------+---------------+------+------+---------+-------+
6 行が選択されました (0.00 秒)
```

フィールドの説明:

* `ERROR_NUMBER`: 返されたMySQL互換のエラー番号。
* `ERROR_MESSAGE`: エラーメッセージはエラー番号に一致します（プリペアドステートメント形式）。
* `ERROR_COUNT`: このエラーが返された回数。
* `WARNING_COUNT`: この警告が返された回数。
* `FIRST_SEEN`: このエラー（または警告）が最初に送信された時間。
* `LAST_SEEN`: このエラー（または警告）が最も最近に送信された時間。

次の例では、ローカルのTiDBサーバに接続した際に警告が生成されます。`FLUSH CLIENT_ERRORS_SUMMARY`を実行した後にサマリーがリセットされます。

```sql
SELECT 0/0;
SELECT * FROM CLIENT_ERRORS_SUMMARY_GLOBAL;
FLUSH CLIENT_ERRORS_SUMMARY;
SELECT * FROM CLIENT_ERRORS_SUMMARY_GLOBAL;
```

出力は次のようになります。

```sql
+-----+
| 0/0 |
+-----+
| NULL |
+-----+
1 行が選択され、1 つの警告があります (0.00 秒)

+--------------+---------------+-------------+---------------+---------------------+---------------------+
| ERROR_NUMBER | ERROR_MESSAGE | ERROR_COUNT | WARNING_COUNT | FIRST_SEEN          | LAST_SEEN           |
+--------------+---------------+-------------+---------------+---------------------+---------------------+
|         1365 | ゼロでの除算 |           0 |             1 | 2021-03-18 13:10:51 | 2021-03-18 13:10:51 |
+--------------+---------------+-------------+---------------+---------------------+---------------------+
1 行が選択されました (0.00 秒)

クエリは正常に完了しました。影響を受けた行は 0 行です (0.00 秒)

空のセット (0.00 秒)
```