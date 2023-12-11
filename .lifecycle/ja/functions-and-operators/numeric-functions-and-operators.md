---
title: 数値関数と演算子
summary: 数値関数と演算子について学びます。
aliases: ['/docs/dev/functions-and-operators/numeric-functions-and-operators/','/docs/dev/reference/sql/functions-and-operators/numeric-functions-and-operators/']
---

# 数値関数と演算子

TiDBはMySQL 5.7で利用可能な[数値関数と演算子](https://dev.mysql.com/doc/refman/5.7/en/numeric-functions.html)をすべてサポートしています。

## 算術演算子

| 名前                                                                                                   | 説明                     |
|:--------------------------------------------------------------------------------------------------------|:--------------------------|
| [`+`](https://dev.mysql.com/doc/refman/8.0/en/arithmetic-functions.html#operator_plus)                 | 加算演算子               |
| [`-`](https://dev.mysql.com/doc/refman/8.0/en/arithmetic-functions.html#operator_minus)                | 減算演算子               |
| [`*`](https://dev.mysql.com/doc/refman/8.0/en/arithmetic-functions.html#operator_times)                | 乗算演算子               |
| [`/`](https://dev.mysql.com/doc/refman/8.0/en/arithmetic-functions.html#operator_divide)               | 除算演算子               |
| [`DIV`](https://dev.mysql.com/doc/refman/8.0/en/arithmetic-functions.html#operator_div)                | 整数除算                 |
| [`%`, `MOD`](https://dev.mysql.com/doc/refman/8.0/en/arithmetic-functions.html#operator_mod)           | 剰余演算子               |
| [`-`](https://dev.mysql.com/doc/refman/8.0/en/arithmetic-functions.html#operator_unary-minus)          | 引数の符号を変更する     |

## 数学関数

| 名前                                                                                                              | 説明                                                 |
|:-------------------------------------------------------------------------------------------------------------------|:------------------------------------------------------|
| [`POW()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_pow)                       | 指定された累乗の引数を返す                           |
| [`POWER()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_power)                   | 指定された累乗の引数を返す                           |
| [`EXP()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_exp)                       | 累乗する                                             |
| [`SQRT()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_sqrt)                     | 引数の平方根を返す                                   |
| [`LN()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_ln)                         | 引数の自然対数を返す                                 |
| [`LOG()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_log)                       | 最初の引数の自然対数を返す                           |
| [`LOG2()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_log2)                     | 対数2の引数を返す                                    |
| [`LOG10()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_log10)                   | 対数10の引数を返す                                   |
| [`PI()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_pi)                         | 円周率を返す                                         |
| [`TAN()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_tan)                       | 引数の正接を返す                                     |
| [`COT()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_cot)                       | 余接を返す                                           |
| [`SIN()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_sin)                       | 引数の正弦を返す                                     |
| [`COS()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_cos)                       | 余弦を返す                                           |
| [`ATAN()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_atan)                     | アークタンジェントを返す                             |
| [`ATAN2(), ATAN()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_atan2)           | 2つの引数のアークタンジェントを返す                   |
| [`ASIN()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_asin)                     | アークサインを返す                                   |
| [`ACOS()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_acos)                     | アークコサインを返す                                 |
| [`RADIANS()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_radians)               | 引数をラジアンに変換する                             |
| [`DEGREES()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_degrees)               | ラジアンを度に変換する                               |
| [`MOD()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_mod)                       | 余りを返す                                           |
| [`ABS()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_abs)                       | 絶対値を返す                                       |
| [`CEIL()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_ceil)                     | 引数以上の最小の整数値を返す                         |
| [`CEILING()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_ceiling)             | 引数以上の最小の整数値を返す                         |
| [`FLOOR()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_floor)                 | 引数以下の最大の整数値を返す                         |
| [`ROUND()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_round)                 | 引数を四捨五入する                                   |
| [`RAND()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_rand)                   | ランダムな浮動小数点値を返す                         |
| [`SIGN()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_sign)                   | 引数の符号を返す                                     |
| [`CONV()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_conv)                   | 異なる数値ベース間で数値を変換する                   |
| [`TRUNCATE()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_truncate)           | 指定された十進数の位まで切り捨てる                   |
| [`CRC32()`](https://dev.mysql.com/doc/refman/8.0/en/mathematical-functions.html#function_crc32)                 | 循環冗長検査値を計算する                             |