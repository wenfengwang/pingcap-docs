---
title: その他の機能
summary: TiDBのその他の機能について学びます。
aliases: ['/docs/dev/functions-and-operators/miscellaneous-functions/','/docs/dev/reference/sql/functions-and-operators/miscellaneous-functions/']
---

# その他の機能

TiDBはMySQL 5.7で利用可能なほとんどの[その他の機能](https://dev.mysql.com/doc/refman/5.7/en/miscellaneous-functions.html)をサポートしています。

## サポートされている機能

| 名前 | 説明  |
|:------------|:-----------------------------------------------------------------------------------------------|
| [`ANY_VALUE()`](https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html#function_any-value)              | `ONLY_FULL_GROUP_BY`の値を抑制します     |
| [`BIN_TO_UUID()`](https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html#function_bin-to-uuid)          | バイナリ形式のUUIDをテキスト形式に変換します    |
| [`DEFAULT()`](https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html#function_default)                  | テーブルのカラムのデフォルト値を返します      |
| [`INET_ATON()`](https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html#function_inet-aton)              | IPアドレスの数値値を返します         |
| [`INET_NTOA()`](https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html#function_inet-ntoa)              | 数値値からIPアドレスを返します        |
| [`INET6_ATON()`](https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html#function_inet6-aton)            | IPv6アドレスの数値値を返します       |
| [`INET6_NTOA()`](https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html#function_inet6-ntoa)            | 数値値からIPv6アドレスを返します      |
| [`IS_IPV4()`](https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html#function_is-ipv4)                  | 引数がIPv4アドレスであるかどうか   |
| [`IS_IPV4_COMPAT()`](https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html#function_is-ipv4-compat)    | 引数がIPv4互換アドレスであるかどうか    |
| [`IS_IPV4_MAPPED()`](https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html#function_is-ipv4-mapped)    | 引数がIPv4マップアドレスであるかどうか   |
| [`IS_IPV6()`](https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html#function_is-ipv6)                  | 引数がIPv6アドレスであるかどうか   |
| [`NAME_CONST()`](https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html#function_name-const)            | カラム名を変更するために使用できます               |
| [`SLEEP()`](https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html#function_sleep)                      | 指定した秒数だけスリープします。ただし、[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスターでは、`SLEEP()`関数は最大スリープ時間が300秒までサポートされる制限があります。       |
| [`UUID()`](https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html#function_uuid)                        | ユニバーサルユニーク識別子（UUID）を返します       |
| [`UUID_TO_BIN()`](https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html#function_uuid-to-bin)          | テキスト形式のUUIDをバイナリ形式に変換します    |
| [`VALUES()`](https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html#function_values)                    | 挿入時に使用される値を定義します       |

## サポートされていない機能

| 名前 | 説明  |
|:------------|:-----------------------------------------------------------------------------------------------|
| [`UUID_SHORT()`](https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html#function_uuid-short)            | TiDBにはない特定の仮定に基づいた一意のUUIDを提供します [TiDB #4620](https://github.com/pingcap/tidb/issues/4620) |
| [`MASTER_WAIT_POS()`](https://dev.mysql.com/doc/refman/8.0/en/miscellaneous-functions.html#function_master-pos-wait)  | MySQLレプリケーションに関連します |