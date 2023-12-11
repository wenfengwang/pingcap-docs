---
title: JSON 関数
summary: JSON 関数について学ぶ
aliases: ['/docs/dev/functions-and-operators/json-functions/','/docs/dev/reference/sql/functions-and-operators/json-functions/']
---

# JSON 関数

TiDB は MySQL 5.7 の GA リリースで提供されたほとんどの JSON 関数をサポートしています。

## JSON 値を作成する関数

| 関数名                     | 説明 |
| --------------------------------- | ----------- |
| [JSON_ARRAY([val[, val] ...])](https://dev.mysql.com/doc/refman/8.0/en/json-creation-functions.html#function_json-array)  | (可能なら空の)値のリストを評価し、それらの値を含む JSON 配列を返します |
| [JSON_OBJECT(key, val[, key, val] ...)](https://dev.mysql.com/doc/refman/8.0/en/json-creation-functions.html#function_json-object)   | (可能なら空の)キーと値のペアのリストを評価し、それらのペアを含む JSON オブジェクトを返します  |
| [JSON_QUOTE(string)](https://dev.mysql.com/doc/refman/8.0/en/json-creation-functions.html#function_json-quote) | 文字列をクォートした JSON 値として返します |

## JSON 値を検索する関数

| 関数名                     | 説明 |
| --------------------------------- | ----------- |
| [JSON_CONTAINS(target, candidate[, path])](https://dev.mysql.com/doc/refman/8.0/en/json-search-functions.html#function_json-contains) | 指定された candidate JSON ドキュメントが target JSON ドキュメントに含まれるかどうかを 1 または 0 で示します |
| [JSON_CONTAINS_PATH(json_doc, one_or_all, path[, path] ...)](https://dev.mysql.com/doc/refman/8.0/en/json-search-functions.html#function_json-contains-path) | 指定されたパスまたは複数のパスにおいて JSON ドキュメントにデータが含まれるかどうかを示す 0 または 1 を返します |
| [JSON_EXTRACT(json_doc, path[, path] ...)](https://dev.mysql.com/doc/refman/8.0/en/json-search-functions.html#function_json-extract)| JSON ドキュメントからデータを取得し、`path` 引数で選択されたドキュメントの部分から選択します |
| [->](https://dev.mysql.com/doc/refman/8.0/en/json-search-functions.html#operator_json-column-path)  | JSON カラムから値を返します。評価されるパス後の JSON カラムからの値を返します。`JSON_EXTRACT(doc, path_literal)` の別名です   |
| [->>](https://dev.mysql.com/doc/refman/8.0/en/json-search-functions.html#operator_json-inline-path)  | JSON カラムから値を返します。評価されるパス後の JSON カラムからの値を返し、結果をアンクォートします。`JSON_UNQUOTE(JSON_EXTRACT(doc, path_literal))` の別名です |
| [JSON_KEYS(json_doc[, path])](https://dev.mysql.com/doc/refman/8.0/en/json-search-functions.html#function_json-keys) | JSON オブジェクトのトップレベル値からキーを JSON 配列として返し、path 引数が指定されている場合は選択されたパスからトップレベルのキーを返します |
| [JSON_SEARCH(json_doc, one_or_all, search_str[, escape_char[, path] ...])](https://dev.mysql.com/doc/refman/8.0/en/json-search-functions.html#function_json-search) | JSON ドキュメントを対象として文字列の一致を検索します |
| [value MEMBER OF(json_array)](https://dev.mysql.com/doc/refman/8.0/en/json-search-functions.html#operator_member-of) | 与えられた値が JSON 配列の要素である場合は 1 を返し、それ以外の場合は 0 を返します |
| [JSON_OVERLAPS(json_doc1, json_doc2)](https://dev.mysql.com/doc/refman/8.0/en/json-search-functions.html#function_json-overlaps) | 2 つの JSON ドキュメントにオーバーラップする部分があるかどうかを示します。ある場合は 1 を、ない場合は 0 を返します。 |

...
| [JSON_ARRAYAGG(key)](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_json-arrayagg) | キーの集約を提供します。 |
| [JSON_OBJECTAGG(key, value)](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html#function_json-objectagg) | 指定されたキーに対する値の集約を提供します。 |

## 関連情報

* [JSON 関数リファレンス](https://dev.mysql.com/doc/refman/8.0/en/json-function-reference.html)
* [JSON データ型](/data-type-json.md)