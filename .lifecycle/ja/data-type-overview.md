---
title: データ型
summary: TiDBでサポートされているデータ型について学びましょう。
aliases: ['/docs/dev/data-type-overview/','/docs/dev/reference/sql/data-types/overview/']
---

# データ型

TiDBは`SPATIAL`タイプを除くすべてのMySQLのデータ型をサポートしています。これには、[数値型](/data-type-numeric.md)、[文字列型](/data-type-string.md)、[日付時刻型](/data-type-date-and-time.md)、および[JSON型](/data-type-json.md)が含まれます。

データ型の定義は、`T(M[, D])`と指定されます。ここで：

- `T`は特定のデータ型を示します。
- `M`は整数型の最大表示幅を示します。浮動小数点型および固定小数点型の場合、`M`は格納できる桁数（精度）の合計です。文字列型の場合、`M`は最大長です。Mの最大許容値はデータ型に依存します。
- `D`は浮動小数点型および固定小数点型に適用され、小数点以下の桁数（スケール）を示します。
- `fsp`は`TIME`、 `DATETIME`、`TIMESTAMP`の型に適用され、秒未満の精度を表します。与えられた`fsp`の値は0から6の範囲内でなければなりません。値が0の場合、少数部がないことを示します。省略された場合は、デフォルトの精度が0です。
