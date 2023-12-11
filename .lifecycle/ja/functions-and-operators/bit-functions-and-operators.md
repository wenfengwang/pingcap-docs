---
title: ビット関数と演算子
summary: ビット関数と演算子について学びます。
aliases: ['/docs/dev/functions-and-operators/bit-functions-and-operators/','/docs/dev/reference/sql/functions-and-operators/bit-functions-and-operators/']
---

# ビット関数と演算子

TiDB は MySQL 5.7 で利用可能な[ビット関数と演算子](https://dev.mysql.com/doc/refman/5.7/en/bit-functions.html)をすべてサポートしています。

**ビット関数と演算子:**

| 名前 | 説明 |
| :------| :------------- |
| [`BIT_COUNT()`](https://dev.mysql.com/doc/refman/8.0/en/bit-functions.html#function_bit-count) | 1 として設定されているビットの数を返す |
| [&](https://dev.mysql.com/doc/refman/8.0/en/bit-functions.html#operator_bitwise-and) | ビットごとの AND |
| [~](https://dev.mysql.com/doc/refman/8.0/en/bit-functions.html#operator_bitwise-invert) | ビットごとの反転 |
| [\|](https://dev.mysql.com/doc/refman/8.0/en/bit-functions.html#operator_bitwise-or) | ビットごとの OR |
| [^](https://dev.mysql.com/doc/refman/8.0/en/bit-functions.html#operator_bitwise-xor) | ビットごとの XOR |
| [<<](https://dev.mysql.com/doc/refman/8.0/en/bit-functions.html#operator_left-shift) | 左シフト |
| [>>](https://dev.mysql.com/doc/refman/8.0/en/bit-functions.html#operator_right-shift) | 右シフト |