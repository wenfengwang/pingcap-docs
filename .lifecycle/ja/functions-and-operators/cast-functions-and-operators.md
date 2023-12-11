---
title: キャスト関数と演算子
summary: キャスト関数と演算子について学びます。
aliases: ['/docs/dev/functions-and-operators/cast-functions-and-operators/','/docs/dev/reference/sql/functions-and-operators/cast-functions-and-operators/']
---

# キャスト関数と演算子

キャスト関数と演算子は、あるデータ型から別のデータ型への値の変換を可能にします。TiDBはMySQL 5.7で利用可能な[キャスト関数と演算子](https://dev.mysql.com/doc/refman/5.7/en/cast-functions.html)をすべてサポートしています。

## キャスト関数と演算子のリスト

| 名前                                    | 説明                             |
| ---------------------------------------- | -------------------------------- |
| [`BINARY`](https://dev.mysql.com/doc/refman/8.0/en/cast-functions.html#operator_binary) | 文字列をバイナリ文字列にキャストします |
| [`CAST()`](https://dev.mysql.com/doc/refman/8.0/en/cast-functions.html#function_cast) | 値を特定の型にキャストします   |
| [`CONVERT()`](https://dev.mysql.com/doc/refman/8.0/en/cast-functions.html#function_convert) | 値を特定の型にキャストします   |

> **注意:**
>
> TiDBとMySQLでは、`SELECT CAST(MeN AS CHAR)`（またはその同等の形式である`SELECT CONVERT(MeM, CHAR)`）に対して一貫性のない結果が表示されます。ここで`MeN`は科学表記の倍精度浮動小数点数を表します。MySQLでは、`-15 <= N <= 14`のときは完全な数値が表示され、`N < -15`または`N > 14`のときは科学表記が表示されます。一方、TiDBでは常に完全な数値が表示されます。例えばMySQLでは、`SELECT CAST(3.1415e15 AS CHAR)`の結果は`3.1415e15`と表示されますが、TiDBでは`3141500000000000`と表示されます。