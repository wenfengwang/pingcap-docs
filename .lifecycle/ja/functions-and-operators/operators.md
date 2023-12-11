---
title: オペレータ
summary: オペレータの優先順位、比較関数とオペレータ、論理演算子、代入演算子について学びます。
aliases: ['/docs/dev/functions-and-operators/operators/','/docs/dev/reference/sql/functions-and-operators/operators/']
---

# オペレータ

このドキュメントでは、オペレータの優先順位、比較関数とオペレータ、論理演算子、代入演算子について説明します。

- [オペレータの優先順位](#operator-precedence)
- [比較関数とオペレータ](#comparison-functions-and-operators)
- [論理演算子](#logical-operators)
- [代入演算子](#assignment-operators)

| 名前 | 説明 |
| ---------------------------------------- | ---------------------------------------- |
| [AND, &&](https://dev.mysql.com/doc/refman/8.0/en/logical-operators.html#operator_and) | 論理積 |
| [=](https://dev.mysql.com/doc/refman/8.0/en/assignment-operators.html#operator_assign-equal) | 値の代入（`SET`ステートメントの一部として、または`UPDATE`ステートメントの`SET`句の一部として） |
| [:=](https://dev.mysql.com/doc/refman/8.0/en/assignment-operators.html#operator_assign-value) | 値の代入 |
| [BETWEEN ... AND ...](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_between) | 値が特定の範囲内にあるかどうかをチェック |
| [BINARY](https://dev.mysql.com/doc/refman/8.0/en/cast-functions.html#operator_binary) | 文字列をバイナリ文字列にキャスト |
| [&](https://dev.mysql.com/doc/refman/8.0/en/bit-functions.html#operator_bitwise-and) | ビットごとの論理積 |
| [~](https://dev.mysql.com/doc/refman/8.0/en/bit-functions.html#operator_bitwise-invert) | ビットごとの否定 |
| [\|](https://dev.mysql.com/doc/refman/8.0/en/bit-functions.html#operator_bitwise-or) | ビットごとの論理和 |
| [^](https://dev.mysql.com/doc/refman/8.0/en/bit-functions.html#operator_bitwise-xor) | ビットごとの排他的論理和 |
| [CASE](https://dev.mysql.com/doc/refman/8.0/en/flow-control-functions.html#operator_case) | CASE演算子 |
| [DIV](https://dev.mysql.com/doc/refman/8.0/en/arithmetic-functions.html#operator_div) | 整数除算 |
| [/](https://dev.mysql.com/doc/refman/8.0/en/arithmetic-functions.html#operator_divide) | 除算演算子 |
| [=](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_equal) | 等しい演算子 |
| [`<=>`](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_equal-to) | NULLセーフ等しい演算子 |
| [>](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_greater-than) | 大なり演算子 |
| [>=](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_greater-than-or-equal) | 大なりイコール演算子 |
| [IS](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_is) | 値をブール値と比較 |
| [IS NOT](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_is-not) | 値をブール値と比較 |
| [IS NOT NULL](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_is-not-null) | NOT NULL値のテスト |
| [IS NULL](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_is-null) | NULL値のテスト |
| [->](https://dev.mysql.com/doc/refman/8.0/en/json-search-functions.html#operator_json-column-path) | JSON列からパスを評価した後の値を返す。`JSON_EXTRACT()`と同等 |
| [->>](https://dev.mysql.com/doc/refman/8.0/en/json-search-functions.html#operator_json-inline-path) | JSON列からパスを評価し、結果をアンクォートした値を返す。`JSON_UNQUOTE(JSON_EXTRACT())`と同等 |
| [<<](https://dev.mysql.com/doc/refman/8.0/en/bit-functions.html#operator_left-shift) | 左シフト |
| [<](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_less-than) | 小なり演算子 |
| [<=](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_less-than-or-equal) | 小なりイコール演算子 |
| [LIKE](https://dev.mysql.com/doc/refman/8.0/en/string-comparison-functions.html#operator_like) | 単純なパターンマッチング |
| [ILIKE](https://www.postgresql.org/docs/current/functions-matching.html) | 大文字小文字を区別しない単純なパターンマッチング（TiDBでサポートされていますが、MySQLでサポートされていません） |
| [-](https://dev.mysql.com/doc/refman/8.0/en/arithmetic-functions.html#operator_minus) | マイナス演算子 |
| [%, MOD](https://dev.mysql.com/doc/refman/8.0/en/arithmetic-functions.html#operator_mod) | 剰余演算子 |
| [NOT, !](https://dev.mysql.com/doc/refman/8.0/en/logical-operators.html#operator_not) | 値を否定 |
| [NOT BETWEEN ... AND ...](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_not-between) | 値が特定の範囲内にないかどうかをチェック |
| [!=, `<>`](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_not-equal) | 等しくない演算子 |
| [NOT LIKE](https://dev.mysql.com/doc/refman/8.0/en/string-comparison-functions.html#operator_not-like) | 単純なパターンマッチングの否定 |
| [NOT REGEXP](https://dev.mysql.com/doc/refman/8.0/en/regexp.html#operator_not-regexp) | REGEXPの否定 |
| [\|\|, OR](https://dev.mysql.com/doc/refman/8.0/en/logical-operators.html#operator_or) | 論理和 |
| [+](https://dev.mysql.com/doc/refman/8.0/en/arithmetic-functions.html#operator_plus) | 加算演算子 |
| [REGEXP](https://dev.mysql.com/doc/refman/8.0/en/regexp.html#operator_regexp) | 正規表現を使用したパターンマッチング |
| [>>](https://dev.mysql.com/doc/refman/8.0/en/bit-functions.html#operator_right-shift) | 右シフト |
| [RLIKE](https://dev.mysql.com/doc/refman/8.0/en/regexp.html#operator_regexp) | REGEXPの同義語 |
| [*](https://dev.mysql.com/doc/refman/8.0/en/arithmetic-functions.html#operator_times) | 乗算演算子 |
| [-](https://dev.mysql.com/doc/refman/8.0/en/arithmetic-functions.html#operator_unary-minus) | 引数の符号を変更 |
| [XOR](https://dev.mysql.com/doc/refman/8.0/en/logical-operators.html#operator_xor) | 論理排他的論理和 |

## サポートされていないオペレータ

* [`SOUNDS LIKE`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#operator_sounds-like)

## オペレータの優先順位

オペレータの優先順位は、次のリストに示す通り、最も優先度が高いものから最も低いものまでです。同じ行に表示されているオペレータは、同じ優先順位を持っています。

```sql
INTERVAL
BINARY, COLLATE
!
-（単項マイナス）、~（単項ビット反転）
^
*, /, DIV, %, MOD
-, +
<<, >>
&
|
=（比較）、<=>, >=, >, <=, <, <>, !=, IS, LIKE, REGEXP, IN
BETWEEN, CASE, WHEN, THEN, ELSE
NOT
AND, &&
XOR
OR, ||
=（代入）、:=
```

詳細は、[オペレータの優先順位](https://dev.mysql.com/doc/refman/8.0/en/operator-precedence.html)を参照してください。

## 比較関数とオペレータ

| 名前 | 説明 |
| ---------------------------------------- | ---------------------------------------- |
| [BETWEEN ... AND ...](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_between) | 値が特定の範囲内にあるかどうかをチェック |
| [COALESCE()](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#function_coalesce) | 最初の非NULLの引数を返す |
| [=](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_equal) | 等しい演算子 |
| [`<=>`](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_equal-to) | NULLセーフ等しい演算子 |
| [>](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_greater-than) | 大なり演算子 |
| [>=](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_greater-than-or-equal) | 大なりイコール演算子 |

| [GREATEST()](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#function_greatest) | 最大の引数を返します |
| [IN()](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_in) | 値が値のセット内にあるかどうかをチェックします |
| [INTERVAL()](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#function_interval) | 最初の引数よりも小さい引数のインデックスを返します |
| [IS](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_is) | 値をブール値と比較します |
| [IS NOT](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_is-not) | 値をブール値と比較します |
| [IS NOT NULL](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_is-not-null) | NOT NULL 値のテスト |
| [IS NULL](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_is-null) | NULL 値のテスト |
| [ISNULL()](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#function_isnull) | 引数が NULL かどうかをテストします |
| [LEAST()](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#function_least) | 最小の引数を返します |
| [<](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_less-than) | 小なり演算子 |
| [<=](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_less-than-or-equal) | 以下演算子 |
| [LIKE](https://dev.mysql.com/doc/refman/8.0/en/string-comparison-functions.html#operator_like) | シンプルなパターンマッチング |
| [ILIKE](https://www.postgresql.org/docs/current/functions-matching.html) | 大文字小文字を区別しないシンプルなパターンマッチング（TiDB でサポートされていますが、MySQL ではサポートされていません） |
| [NOT BETWEEN ... AND ...](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_not-between) | 値が値の範囲内にないかどうかをチェックします |
| [!=, `<>`](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_not-equal) | 不等号演算子 |
| [NOT IN()](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html#operator_not-in) | 値が値のセット内にないかどうかをチェックします |
| [NOT LIKE](https://dev.mysql.com/doc/refman/8.0/en/string-comparison-functions.html#operator_not-like) | シンプルなパターンマッチングの否定 |
| [STRCMP()](https://dev.mysql.com/doc/refman/8.0/en/string-comparison-functions.html#function_strcmp) | 2 つの文字列を比較します |

詳細については、[Comparison Functions and Operators](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html) を参照してください。

## 論理演算子

| 名前 | 説明 |
| ---------------------------------------- | ------------- |
| [AND, &&](https://dev.mysql.com/doc/refman/8.0/en/logical-operators.html#operator_and) | 論理積 |
| [NOT, !](https://dev.mysql.com/doc/refman/8.0/en/logical-operators.html#operator_not) | 値を否定します |
| [\|\|, OR](https://dev.mysql.com/doc/refman/8.0/en/logical-operators.html#operator_or) | 論理和 |
| [XOR](https://dev.mysql.com/doc/refman/8.0/en/logical-operators.html#operator_xor) | 排他的論理和 |

詳細については、[MySQL Handling of GROUP BY](https://dev.mysql.com/doc/refman/8.0/en/group-by-handling.html) を参照してください。

## 代入演算子

| 名前 | 説明 |
| ---------------------------------------- | ---------------------------------------- |
| [=](https://dev.mysql.com/doc/refman/8.0/en/assignment-operators.html#operator_assign-equal) | 値を代入します（`SET` ステートメントの一部として、または `UPDATE` ステートメントの `SET` 句の一部として） |
| [:=](https://dev.mysql.com/doc/refman/8.0/en/assignment-operators.html#operator_assign-value) | 値を代入します |

詳細については、[Detection of Functional Dependence](https://dev.mysql.com/doc/refman/8.0/en/group-by-functional-dependence.html) を参照してください。

## MySQL 互換性

* MySQL では `ILIKE` 演算子はサポートされていません。