---
title: 関数と演算子リファレンス
summary: 関数と演算子の使用方法を学ぶ
aliases: ['/docs/dev/functions-and-operators/functions-and-operators-overview/','/docs/dev/reference/sql/functions-and-operators/reference/']
---

# 関数と演算子リファレンス

TiDBにおける関数と演算子の使用法はMySQLと類似しています。[MySQLの関数と演算子](https://dev.mysql.com/doc/refman/8.0/en/functions.html)を参照してください。

SQL文では、`SELECT`文の`ORDER BY`および`HAVING`節、`SELECT`/`DELETE`/`UPDATE`文の`WHERE`節、および`SET`文で表現を使用できます。

リテラル、カラム名、NULL、組み込み関数、演算子などを使用して式を記述できます。TiDBがTiKVにプッシュダウンできる式については、[プッシュダウン用の式のリスト](/functions-and-operators/expressions-pushed-down.md)を参照してください。