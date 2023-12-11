---
title: ウィンドウ関数
summary: このドキュメントでは、TiDBでサポートされているウィンドウ関数について紹介します。
aliases: ['/docs/dev/functions-and-operators/window-functions/','/docs/dev/reference/sql/functions-and-operators/window-functions/']
---

# ウィンドウ関数

TiDBにおけるウィンドウ関数の使用方法は、MySQL 8.0と類似しています。詳細については、[MySQLウィンドウ関数](https://dev.mysql.com/doc/refman/8.0/en/window-functions.html)を参照してください。

ウィンドウ関数は、パーサーで追加のワードを予約するため、TiDBではウィンドウ関数を無効にするオプションが提供されています。アップグレード後にSQLステートメントの解析でエラーが発生する場合は、`tidb_enable_window_function=0`を設定してみてください。

[ここにリストされている](/tiflash/tiflash-supported-pushdown-calculations.md)ウィンドウ関数は、TiFlashにプッシュダウンできます。

`GROUP_CONCAT()`および`APPROX_PERCENTILE()`を除いて、TiDBはすべての[`GROUP BY`集約関数](/functions-and-operators/aggregate-group-by-functions.md)をサポートしています。さらに、TiDBは以下のウィンドウ関数をサポートしています:

| 関数名 | 機能の説明 |
| :-------------- | :------------------------------------- |
| [`CUME_DIST()`](https://dev.mysql.com/doc/refman/8.0/en/window-function-descriptions.html#function_cume-dist) | グループ内の値の累積分布を返します。 |
| [`DENSE_RANK()`](https://dev.mysql.com/doc/refman/8.0/en/window-function-descriptions.html#function_dense-rank) | パーティション内の現在の行のランクを、ギャップなしで返します。 |
| [`FIRST_VALUE()`](https://dev.mysql.com/doc/refman/8.0/en/window-function-descriptions.html#function_first-value) | 現在のウィンドウ内の最初の行の式の値を返します。 |
| [`LAG()`](https://dev.mysql.com/doc/refman/8.0/en/window-function-descriptions.html#function_lag) | パーティション内で現在の行よりも前のN行の式の値を返します。 |
| [`LAST_VALUE()`](https://dev.mysql.com/doc/refman/8.0/en/window-function-descriptions.html#function_last-value) | 現在のウィンドウ内の最後の行の式の値を返します。 |
| [`LEAD()`](https://dev.mysql.com/doc/refman/8.0/en/window-function-descriptions.html#function_lead) | パーティション内で現在の行よりも後のN行の式の値を返します。 |
| [`NTH_VALUE()`](https://dev.mysql.com/doc/refman/8.0/en/window-function-descriptions.html#function_nth-value) | 現在のウィンドウのN番目の行の式の値を返します。 |
| [`NTILE()`](https://dev.mysql.com/doc/refman/8.0/en/window-function-descriptions.html#function_ntile) | パーティションをNつのバケツに分割し、各行にバケツ番号を割り当て、パーティション内の現在の行のバケツ番号を返します。 |
| [`PERCENT_RANK()`](https://dev.mysql.com/doc/refman/8.0/en/window-function-descriptions.html#function_percent-rank) | 現在の行よりも小さい値のパーティション値の割合を返します。 |
| [`RANK()`](https://dev.mysql.com/doc/refman/8.0/en/window-function-descriptions.html#function_rank) | パーティション内の現在の行のランクを返します。ランクにはギャップがある場合があります。 |
| [`ROW_NUMBER()`](https://dev.mysql.com/doc/refman/8.0/en/window-function-descriptions.html#function_row-number) | パーティション内の現在の行の番号を返します。 |