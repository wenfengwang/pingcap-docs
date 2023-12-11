---
title: ロック機能
summary: TiDBのユーザーレベルのロック機能について学びます。

# ロック機能

TiDBはMySQL 5.7で利用可能なほとんどのユーザーレベルの[ロック機能](https://dev.mysql.com/doc/refman/5.7/en/locking-functions.html)をサポートしています。

## サポートされている機能

| 名前                                                                                                                 | 説明                                                           |
|:---------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------|
| [`GET_LOCK(lockName, timeout)`](https://dev.mysql.com/doc/refman/8.0/en/locking-functions.html#function_get-lock)    | アドバイザリーロックを取得します。`lockName`パラメーターは64文字を超えてはいけません。タイムアウトまでの最大`timeout`秒待機し、タイムアウトして失敗した場合は失敗を返します。         |
| [`IS_FREE_LOCK(lockName)`](https://dev.mysql.com/doc/refman/8.0/en/locking-functions.html#function_is-free-lock) | ロックが空いているかどうかを確認します。 |
| [`IS_USED_LOCK(lockName)`](https://dev.mysql.com/doc/refman/8.0/en/locking-functions.html#function_is-used-lock) | ロックが使用中かどうかを確認します。使用中の場合は、対応する接続IDを返します。 |
| [`RELEASE_ALL_LOCKS()`](https://dev.mysql.com/doc/refman/8.0/en/locking-functions.html#function_release-all-locks)   | 現在のセッションが保持しているすべてのロックを解放します。                        |
| [`RELEASE_LOCK(lockName)`](https://dev.mysql.com/doc/refman/8.0/en/locking-functions.html#function_release-lock)     | 以前に取得したロックを解放します。`lockName`パラメーターは64文字を超えてはいけません。 |

## MySQL互換性

* TiDBで許可される最小のタイムアウトは1秒で、最大のタイムアウトは1時間（3600秒）です。これはMySQLと異なり、0秒および無制限のタイムアウト（`timeout=-1`）の両方が許可されています。TiDBは範囲外の値を自動的に最も近い許容値に変換し、`timeout=-1`を3600秒に変換します。
* TiDBはユーザーレベルのロックによって引き起こされるデッドロックを自動的に検出しません。デッドロックしたセッションは1時間を超えるとタイムアウトしますが、影響を受けるセッションの1つで`KILL`を使用して手動で解決することもできます。また、ユーザーレベルのロックを常に同じ順序で取得することによってデッドロックを防ぐこともできます。
* ロックはクラスタ内のすべてのTiDBサーバーに影響します。これはMySQLクラスターおよびグループレプリケーションとは異なり、ロックは単一のサーバーにローカルです。
* `IS_USED_LOCK()`は、他のセッションから呼び出され、ロックを保持しているプロセスのIDを返すことができない場合に`1`を返します。