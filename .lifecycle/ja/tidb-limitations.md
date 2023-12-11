---
title: TiDBの制限
summary: TiDBの使用制限について学ぶ
aliases: ['/docs/dev/tidb-limitations/']
---

# TiDBの制限

このドキュメントでは、TiDBの一般的な使用制限について、識別子の最大長やサポートされる最大データベース数、テーブル数、インデックス数、パーティションテーブル数、およびシーケンス数について説明します。

## 識別子長に関する制限

| 識別子の種類 | 最大長（許容される文字数） |
|:---------|:--------------|
| データベース | 64 |
| テーブル   | 64 |
| カラム      | 64 |
| インデックス    | 64 |
| ビュー       | 64 |
| シーケンス | 64 |

## データベース、テーブル、ビュー、接続の総数に関する制限

| 種類  | 最大数  |
|:----------|:----------|
| データベース | 無制限 |
| テーブル       | 無制限 |
| ビュー         | 無制限 |
| 接続        | 無制限 |

## 単一データベースに関する制限

| 種類       | 上限   |
|:----------|:----------|
| テーブル        | 無制限  |

## 単一テーブルに関する制限

| 種類       | 上限（デフォルト値）   |
|:----------|:----------|
| カラム       | デフォルトは1017で、4096まで調整可能    |
| インデックス   | デフォルトは64で、512まで調整可能        |
| 行           | 無制限 |
| サイズ       | 無制限 |
| パーティション | 8192     |

<CustomContent platform="tidb">

* `Columns`の上限は、[`table-column-count-limit`](/tidb-configuration-file.md#table-column-count-limit-new-in-v50)を介して変更できます。
* `Indexes`の上限は、[`index-limit`](/tidb-configuration-file.md#index-limit-new-in-v50)を介して変更できます。

</CustomContent>

## 単一行に関する制限

| 種類       | 上限（デフォルト値）   |
|:----------|:----------|
| サイズ       | デフォルトは6 MiBで、120 MiBまで調整可能   |

<CustomContent platform="tidb">

[`txn-entry-size-limit`](/tidb-configuration-file.md#txn-entry-size-limit-new-in-v50)構成項目を介して、サイズ制限を調整できます。

</CustomContent>

## データ型に関する制限

| 種類       | 上限   |
|:----------|:----------|
| CHAR       | 256 文字      |
| BINARY     | 256 文字      |
| VARBINARY  | 65535 文字   |
| VARCHAR    | 16383 文字   |
| TEXT       | デフォルトは6 MiBで、120 MiBまで調整可能     |
| BLOB       | デフォルトは6 MiBで、120 MiBまで調整可能    |

## SQLステートメントに関する制限

| 種類       | 上限   |
|:----------|:----------|
| 単一トランザクション内のSQLステートメントの最大数 | 楽観的トランザクションが使用され、トランザクションの再試行が有効になっている場合、上限は5000です。 |

<CustomContent platform="tidb">

[`stmt-count-limit`](/tidb-configuration-file.md#stmt-count-limit)構成項目を介して、この制限を変更できます。

</CustomContent>

## TiKVバージョンの制限

クラスタ内でTiDBコンポーネントのバージョンがv6.2.0以上の場合、TiKVのバージョンもv6.2.0以上である必要があります。