---
title: CLIENT_ERRORS_SUMMARY_BY_HOST
summary: `CLIENT_ERRORS_SUMMARY_BY_HOST` INFORMATION_SCHEMA テーブルについて学びます。

# CLIENT_ERRORS_SUMMARY_BY_HOST

テーブル `CLIENT_ERRORS_SUMMARY_BY_HOST` は、TiDB サーバーに接続するクライアントに返された SQL エラーや警告のサマリを提供します。これには以下が含まれます。

* 不正な SQL ステートメント。
* ゼロによる除算エラー。
* 範囲外または重複したキー値の挿入を試行すること。
* 権限エラー。
* 存在しないテーブル。

これらのエラーは、MySQL サーバープロトコルを介してクライアントに返され、アプリケーションが適切な対応を取ることが期待されています。`INFORMATION_SCHEMA.CLIENT_ERRORS_SUMMARY_BY_HOST` テーブルは、TiDB サーバーによって返されたエラーをアプリケーションが適切に処理(または記録)していない状況を検査するための有用な手段を提供します。

`CLIENT_ERRORS_SUMMARY_BY_HOST` がリモートホストごとにエラーをまとめるため、1つのアプリケーションサーバが他のサーバよりも多くのエラーを生成しているシナリオを診断するのに役立ちます。可能なシナリオには以下があります。

* 時代遅れの MySQL クライアントライブラリ。
* 時代遅れのアプリケーション (おそらく新しいデプロイの際にこのサーバが見落とされた可能性があります)。
* ユーザ権限の "ホスト" 部分の誤った使用。
* タイムアウトや切断された接続が多く発生する信頼性のないネットワーク接続。

サマリされた回数は、`FLUSH CLIENT_ERRORS_SUMMARY` ステートメントを使用してリセットできます。サマリは各 TiDB サーバにローカルであり、メモリ内だけで保持されます。TiDB サーバが再起動すると、サマリは失われます。

```sql
USE INFORMATION_SCHEMA;
DESC CLIENT_ERRORS_SUMMARY_BY_HOST;
```

出力は以下のようになります。

```sql
+---------------+---------------+------+------+---------+-------+
| Field         | Type          | Null | Key  | Default | Extra |
+---------------+---------------+------+------+---------+-------+
| HOST          | varchar(255)  | NO   |      | NULL    |       |
| ERROR_NUMBER  | bigint(64)    | NO   |      | NULL    |       |
| ERROR_MESSAGE | varchar(1024) | NO   |      | NULL    |       |
| ERROR_COUNT   | bigint(64)    | NO   |      | NULL    |       |
| WARNING_COUNT | bigint(64)    | NO   |      | NULL    |       |
| FIRST_SEEN    | timestamp     | YES  |      | NULL    |       |
| LAST_SEEN     | timestamp     | YES  |      | NULL    |       |
+---------------+---------------+------+------+---------+-------+
7 rows in set (0.00 sec)
```

フィールドの説明:

* `HOST`: クライアントのリモートホスト。
* `ERROR_NUMBER`: 返された MySQL 互換のエラー番号。
* `ERROR_MESSAGE`: エラーメッセージ。これはエラー番号と一致します (プリペアドステートメント形式)。
* `ERROR_COUNT`: このエラーがクライアントホストに返された回数。
* `WARNING_COUNT`: この警告がクライアントホストに返された回数。
* `FIRST_SEEN`: このエラー (または警告) がクライアントホストから最初に見られた時刻。
* `LAST_SEEN`: このエラー (または警告) がクライアントホストから最後に見られた時刻。

以下の例では、クライアントがローカル TiDB サーバに接続した際に警告が生成されます。`FLUSH CLIENT_ERRORS_SUMMARY` を実行した後、サマリがリセットされます。

```sql
SELECT 0/0;
SELECT * FROM CLIENT_ERRORS_SUMMARY_BY_HOST;
FLUSH CLIENT_ERRORS_SUMMARY;
SELECT * FROM CLIENT_ERRORS_SUMMARY_BY_HOST;
```

出力は以下のようになります。

```sql
+-----+
| 0/0 |
+-----+
| NULL |
+-----+
1 行が返され、1 つの警告があります (0.00 秒)

+-----------+--------------+---------------+-------------+---------------+---------------------+---------------------+
| HOST      | ERROR_NUMBER | ERROR_MESSAGE | ERROR_COUNT | WARNING_COUNT | FIRST_SEEN          | LAST_SEEN           |
+-----------+--------------+---------------+-------------+---------------+---------------------+---------------------+
| 127.0.0.1 |         1365 | 0 による除算   |           0 |             1 | 2021-03-18 12:51:54 | 2021-03-18 12:51:54 |
+-----------+--------------+---------------+-------------+---------------+---------------------+---------------------+
1 行が返されます (0.00 秒)

クエリが OK ですが、0 行が変更されました (0.00 秒)

空のセット (0.00 秒)
```