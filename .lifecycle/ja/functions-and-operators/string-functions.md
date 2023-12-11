---
title: 文字列関数
summary: TiDBの文字列関数について学ぶ。
aliases: ['/docs/dev/functions-and-operators/string-functions/','/docs/dev/reference/sql/functions-and-operators/string-functions/']
---

# 文字列関数

TiDBは、MySQL 5.7で利用可能なほとんどの[string functions](https://dev.mysql.com/doc/refman/5.7/en/string-functions.html)と、MySQL 8.0で利用可能ないくつかの[string functions](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html)、およびOracle 21で利用可能な一部の[functions](https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlqr/SQL-Functions.html#GUID-93EC62F8-415D-4A7E-B050-5D5B2C127009)をサポートしています。

<CustomContent platform="tidb">

OracleとTiDBの機能と構文の比較については、[OracleとTiDBの機能と構文の比較](/oracle-functions-to-tidb.md)を参照してください。

</CustomContent>

## サポートされる関数

### [`ASCII()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_ascii)

左端の文字の数値を返します。

### [`BIN()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_bin)

数値のバイナリ表現を含む文字列を返します。

### [`BIT_LENGTH()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_bit-length)

引数の長さをビット単位で返します。

### [`CHAR()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_char)

渡された整数ごとの文字を返します。

### [`CHAR_LENGTH()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_char-length)

引数内の文字数を返します。

### [`CHARACTER_LENGTH()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_character-length)

`CHAR_LENGTH()` の同義語です。

### [`CONCAT()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_concat)

連結された文字列を返します。

### [`CONCAT_WS()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_concat-ws)

区切り文字付きで連結を返します。

### [`ELT()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_elt)

指定されたインデックス番号の文字列を返します。

### [`EXPORT_SET()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_export-set)

値ビットごとに、ON文字列またはOFF文字列を取得します。

### [`FIELD()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_field)

最初の引数の位置を返します。

### [`FIND_IN_SET()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_find-in-set)

第一引数が第二引数内でどこに位置するかを返します。

### [`FORMAT()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_format)

指定された小数点以下の桁数でフォーマットされた数値を返します。

### [`FROM_BASE64()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_from-base64)

Base-64文字列をデコードして結果を返します。

### [`HEX()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_hex)

10進数または文字列の16進数表現を返します。

### [`INSERT()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_insert)

指定された位置から指定された数の文字列を挿入します。

### [`INSTR()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_instr)

サブストリングの最初の出現位置を返します。

### [`LCASE()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_lcase)

`LOWER()` の同義語です。

### [`LEFT()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_left)

指定された文字数の左端の文字を返します。

### [`LENGTH()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_length)

文字列のバイト長を返します。

### [`LIKE`](https://dev.mysql.com/doc/refman/8.0/en/string-comparison-functions.html#operator_like)

シンプルなパターンマッチングです。

### [`LOCATE()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_locate)

サブストリングの最初の出現位置を返します。

### [`LOWER()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_lower)

引数を小文字で返します。

### [`LPAD()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_lpad)

指定された文字列で左端を埋めた文字列を返します。

### [`LTRIM()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_ltrim)

先頭の空白を削除します。

### [`MAKE_SET()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_make-set)

対応するビットがセットされたカンマ区切りの文字列を返します。

### [`MID()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_mid)

指定された位置からのサブストリングを返します。

### [`NOT LIKE`](https://dev.mysql.com/doc/refman/8.0/en/string-comparison-functions.html#operator_not-like)

単純なパターンマッチングの否定です。

### [`NOT REGEXP`](https://dev.mysql.com/doc/refman/8.0/en/regexp.html#operator_not-regexp)

`REGEXP` の否定です。

### [`OCT()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_oct)

数値の8進表現を含む文字列を返します。

### [`OCTET_LENGTH()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_octet-length)

`LENGTH()` の同義語です。

### [`ORD()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_ord)

引数の左端の文字の文字コードを返します。

### [`POSITION()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_position)

`LOCATE()` の同義語です。

### [`QUOTE()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_quote)

SQLステートメントで使用するために引数をエスケープします。

### [`REGEXP`](https://dev.mysql.com/doc/refman/8.0/en/regexp.html#operator_regexp)

正規表現を使用したパターンマッチングです。

### [`REGEXP_INSTR()`](https://dev.mysql.com/doc/refman/8.0/en/regexp.html#function_regexp-instr)

正規表現に一部互換性がある部分的にMySQL互換の文字列の開始インデクスを返します（詳細については[MySQLとの正規表現の互換性](#regular-expression-compatibility-with-mysql)を参照してください）。

### [`REGEXP_LIKE()`](https://dev.mysql.com/doc/refman/8.0/en/regexp.html#function_regexp-like)

文字列が正規表現に一部互換性があるかどうかを返します（詳細については[MySQLとの正規表現の互換性](#regular-expression-compatibility-with-mysql)を参照してください）。

### [`REGEXP_REPLACE()`](https://dev.mysql.com/doc/refman/8.0/en/regexp.html#function_regexp-replace)

正規表現に一部互換性がある部分的にMySQL互換のサブストリングを置換します（詳細については[MySQLとの正規表現の互換性](#regular-expression-compatibility-with-mysql)を参照してください）。

### [`REGEXP_SUBSTR()`](https://dev.mysql.com/doc/refman/8.0/en/regexp.html#function_regexp-substr)

正規表現に一部互換性がある部分的にMySQL互換のサブストリングを返します（詳細については[MySQLとの正規表現の互換性](#regular-expression-compatibility-with-mysql)を参照してください）。

### [`REPEAT()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_repeat)

指定された回数だけ文字列を繰り返します。

### [`REPLACE()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_replace)

指定された文字列の出現箇所を置換します。

### [`REVERSE()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_reverse)

文字列内の文字を逆順にします。

### [`RIGHT()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_right)

指定された右端の文字数を返します。

### [`RLIKE`](https://dev.mysql.com/doc/refman/8.0/en/regexp.html#operator_regexp)

`REGEXP` の同義語です。

### [`RPAD()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_rpad)
指定された回数だけ文字列を追加します。

### [`RTRIM()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_rtrim)

末尾のスペースを削除します。

### [`SPACE()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_space)

指定された数のスペースからなる文字列を返します。

### [`STRCMP()`](https://dev.mysql.com/doc/refman/8.0/en/string-comparison-functions.html#function_strcmp)

2 つの文字列を比較します。

### [`SUBSTR()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_substr)

指定された部分文字列を返します。

### [`SUBSTRING()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_substring)

指定された部分文字列を返します。

### [`SUBSTRING_INDEX()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_substring-index)

指定された区切り文字の出現回数より前の部分文字列を返します。

### [`TO_BASE64()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_to-base64)

引数を base-64 文字列に変換して返します。

### [`TRANSLATE()`](https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/TRANSLATE.html#GUID-80F85ACB-092C-4CC7-91F6-B3A585E3A690)

文字列内のすべての文字を別の文字に置き換えます。Oracle と異なり空の文字列を `NULL` として扱いません。

### [`TRIM()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_trim)

先頭および末尾のスペースを削除します。

### [`UCASE()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_ucase)

`UPPER()` の別名です。

### [`UNHEX()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_unhex)

数値の 16 進表現を含む文字列を返します。

### [`UPPER()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_upper)

大文字に変換します。

### [`WEIGHT_STRING()`](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html#function_weight-string)

入力文字列の weight 文字列を返します。

## サポートされていない関数

* `LOAD_FILE()`
* `MATCH()`
* `SOUNDEX()`

## MySQL との正規表現の互換性

以下のセクションでは、MySQL との正規表現の互換性について説明します。

### 構文の互換性

MySQL は International Components for Unicode (ICU) を使用して正規表現を実装しており、TiDB は RE2 を使用しています。両ライブラリ間の構文の違いについては、[ICU ドキュメント](https://unicode-org.github.io/icu/userguide/) および [RE2 Syntax](https://github.com/google/re2/wiki/Syntax) を参照してください。

### `match_type` の互換性

TiDB と MySQL の `match_type` の値オプションは次のとおりです:

- TiDB での値オプションは `"c"`、`"i"`、`"m"`、`"s"` で、MySQL での値オプションは `"c"`、`"i"`、`"m"`、`"n"`、`"u"` です。
- TiDB の `"s"` は MySQL の `"n"` に対応します。TiDB で `"s"` が設定されると、`.` が行終端記号 (`\n`) にもマッチします。

    例: MySQL での `SELECT REGEXP_LIKE(a, b, "n") FROM t1` は TiDB での `SELECT REGEXP_LIKE(a, b, "s") FROM t1` と同じです。

- TiDB は MySQL の Unix 専用行終端記号を表す `"u"` をサポートしていません。

### データ型の互換性

バイナリ文字列型のサポートについて、TiDB と MySQL の間には次の違いがあります:

- MySQL は 8.0.22 から正規表現関数でバイナリ文字列をサポートしていません。詳細については[MySQL ドキュメント](https://dev.mysql.com/doc/refman/8.0/en/regexp.html)を参照してください。ただし、パラメータや戻り値がすべてバイナリ文字列である場合は、MySQL でも正規関数が機能します。それ以外の場合はエラーが報告されます。
- 現在のところ、TiDB は任意の状況でバイナリ文字列を使用することを禁止しており、必ずエラーが報告されます。

### その他の互換性

空の文字列の置換における TiDB と MySQL のサポートの違い:

次の例では `REGEXP_REPLACE("", "^$", "123")` を取り上げます:

- MySQL は空の文字列を置換せず、結果として `""` を返します。
- TiDB は空の文字列を置換し、結果として `"123"` を返します。