---
title: 情報関数
summary: 情報関数について学ぶ
aliases: ['/docs/dev/functions-and-operators/information-functions/','/docs/dev/reference/sql/functions-and-operators/information-functions/']
---

# 情報関数

TiDBはMySQL 5.7で利用可能なほとんどの[情報関数](https://dev.mysql.com/doc/refman/5.7/en/information-functions.html)をサポートしています。

## TiDBでサポートされているMySQL関数

| 名前 | 説明 |
|:-----|:------------|
| [`BENCHMARK()`](https://dev.mysql.com/doc/refman/8.0/en/information-functions.html#function_benchmark) | 式をループ内で実行します |
| [`CONNECTION_ID()`](https://dev.mysql.com/doc/refman/8.0/en/information-functions.html#function_connection-id) | 接続のID（スレッドID）を返します |
| [`CURRENT_USER()`, `CURRENT_USER`](https://dev.mysql.com/doc/refman/8.0/en/information-functions.html#function_current-user) | 認証されたユーザー名とホスト名を返します |
| [`DATABASE()`](https://dev.mysql.com/doc/refman/8.0/en/information-functions.html#function_database) | デフォルト（現在の）データベース名を返します |
| [`FOUND_ROWS()`](https://dev.mysql.com/doc/refman/8.0/en/information-functions.html#function_found-rows) | `LIMIT`句がある`SELECT`に対して、`LIMIT`句がない場合の返される行の数 |
| [`LAST_INSERT_ID()`](https://dev.mysql.com/doc/refman/8.0/en/information-functions.html#function_last-insert-id) | 最後の`INSERT`の`AUTOINCREMENT`列の値を返します |
| [`ROW_COUNT()`](https://dev.mysql.com/doc/refman/8.0/en/information-functions.html#function_row-count) | 影響を受けた行の数 |
| [`SCHEMA()`](https://dev.mysql.com/doc/refman/8.0/en/information-functions.html#function_schema) | `DATABASE()`の同義語 |
| [`SESSION_USER()`](https://dev.mysql.com/doc/refman/8.0/en/information-functions.html#function_session-user) | `USER()`の同義語 |
| [`SYSTEM_USER()`](https://dev.mysql.com/doc/refman/8.0/en/information-functions.html#function_system-user) | `USER()`の同義語 |
| [`USER()`](https://dev.mysql.com/doc/refman/8.0/en/information-functions.html#function_user) | クライアントによって提供されたユーザー名とホスト名を返します |
| [`VERSION()`](https://dev.mysql.com/doc/refman/8.0/en/information-functions.html#function_version) | MySQLサーバーバージョンを示す文字列を返します |

## TiDB固有の関数

以下の関数はTiDBのみでサポートされており、MySQLには同等の関数がありません。

| 名前 | 説明 |
|:-----|:------------|
| [`CURRENT_RESOURCE_GROUP()`](/functions-and-operators/tidb-functions.md#current_resource_group)  | 現在のセッションがバウンドされているリソースグループの名前を返します |

## サポートされていない関数

* `CHARSET()`
* `COERCIBILITY()`
* `COLLATION()`