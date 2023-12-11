---
title: 制御フロー関数
summary: 制御フロー関数について学びます。
aliases: ['/docs/dev/functions-and-operators/control-flow-functions/','/docs/dev/reference/sql/functions-and-operators/control-flow-functions/']
---

# 制御フロー関数

TiDBは、MySQL 5.7で使用可能な[制御フロー関数](https://dev.mysql.com/doc/refman/5.7/en/flow-control-functions.html)をすべてサポートしています。

| 名前                                                                                                      | 説明                         |
|:---------------------------------------------------------------------------------------------------------|:-----------------------------|
| [`CASE`](https://dev.mysql.com/doc/refman/8.0/en/flow-control-functions.html#operator_case)                | CASE演算子                  |
| [`IF()`](https://dev.mysql.com/doc/refman/8.0/en/flow-control-functions.html#function_if)                  | if/else構文                 |
| [`IFNULL()`](https://dev.mysql.com/doc/refman/8.0/en/flow-control-functions.html#function_ifnull)          | NULL if/else構文            |
| [`NULLIF()`](https://dev.mysql.com/doc/refman/8.0/en/flow-control-functions.html#function_nullif)          | expr1 = expr2の場合はNULLを返す |