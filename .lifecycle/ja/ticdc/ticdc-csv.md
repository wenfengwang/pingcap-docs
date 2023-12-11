---
title: TiCDC CSV プロトコル
summary: TiCDC CSV プロトコルの概念と使用方法について学ぶ。

# TiCDC CSV プロトコル

下流のシンクとしてクラウドストレージサービスを使用する場合、DML イベントを CSV 形式でクラウドストレージサービスに送信できます。

## CSV の使用

CSV プロトコルを使用する際の構成例は次のとおりです：

```shell
cdc cli changefeed create --server=http://127.0.0.1:8300 --changefeed-id="csv-test" --sink-uri="s3://bucket/prefix" --config changefeed.toml
```

`changefeed.toml` ファイルの構成は次のようになります：

```toml
[sink]
protocol = "csv"
terminator = "\n"

[sink.csv]
delimiter = ','
quote = '"'
null = '\N'
include-commit-ts = true
```

## トランザクション制約

- 1 つの CSV ファイル内で、行の `commit-ts` はその後続の行の `commit-ts` 以下となります。
- 単一テーブルの同じトランザクションは同じ CSV ファイルに保存されます。
- 同じトランザクションの複数のテーブルは異なる CSV ファイルに保存できます。

## データ保存パス構造

データの保存パス構造の詳細については、[ストレージパス構造](/ticdc/ticdc-sink-to-cloud-storage.md#storage-path-structure) を参照してください。

## データ形式の定義

CSV ファイルにおいて、各列は次のように定義されます：

- 列 1: 操作タイプの指示子。`I`、`U`、`D` を含みます。`I` は `INSERT`、`U` は `UPDATE`、`D` は `DELETE` を意味します。
- 列 2: テーブル名。
- 列 3: スキーマ名。
- 列 4: ソーストランザクションの `commit-ts`。この列はオプションです。
- 列 5 から最後の列: 変更されるデータを表す 1 つ以上の列。

テーブル `hr.employee` が次のように定義されているとします：

```sql
CREATE TABLE `employee` (
  `Id` int NOT NULL,
  `LastName` varchar(20) DEFAULT NULL,
  `FirstName` varchar(30) DEFAULT NULL,
  `HireDate` date DEFAULT NULL,
  `OfficeLocation` varchar(20) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

このテーブルの DML イベントは、次のように CSV 形式で保存されます：

```shell
"I","employee","hr",433305438660591626,101,"Smith","Bob","2014-06-04","New York"
"U","employee","hr",433305438660591627,101,"Smith","Bob","2015-10-08","Los Angeles"
"D","employee","hr",433305438660591629,101,"Smith","Bob","2017-03-13","Dallas"
"I","employee","hr",433305438660591630,102,"Alex","Alice","2017-03-14","Shanghai"
"U","employee","hr",433305438660591630,102,"Alex","Alice","2018-06-15","Beijing"
```

## データ型のマッピング

| MySQL のデータ型                                       | CSV のデータ型 | 例                                 | 説明                           |
|--------------------------------------------------------|----------------|---------------------------------|---------------------------------------|
| `BOOLEAN`/`TINYINT`/`SMALLINT`/`INT`/`MEDIUMINT`/`BIGINT` | Integer        | `123`                          | -                           |
| `FLOAT`/`DOUBLE`                                        | Float          | `153.123`                      |  -                                   |
| `NULL`                                                 | Null           | `\N`                          | -                                       |
| `TIMESTAMP`/`DATETIME`                                  | String         | `"1973-12-30 15:30:00.123456"` | フォーマット: `yyyy-MM-dd HH:mm:ss.%06d` |
| `DATE`                                                 | String         | `"2000-01-01"`                 | フォーマット: `yyyy-MM-dd`                     |
| `TIME`                                                 | String         | `"23:59:59"`                   | フォーマット: `yyyy-MM-dd`                     |
| `YEAR`                                                 | Integer        | `1970`                         |  -                                    |
| `VARCHAR`/`JSON`/`TINYTEXT`/`MEDIUMTEXT`/`LONGTEXT`/`TEXT`/`CHAR` | String         | `"test"`                       | UTF-8 エンコード                     |
| `VARBINARY`/`TINYBLOB`/`MEDIUMBLOB`/`LONGBLOB`/`BLOB`/`BINARY`  | String         | `"6Zi/5pav"` または `"e998bfe696af"`          | Base64 または 16 進エンコード                 |
| `BIT`                                                  | Integer        | `81`                           | -                                       |
| `DECIMAL`                                              | String         | `"129012.1230000"`             | -                                       |
| `ENUM`                                                 | String         | `"a"`                          | -                                      |
| `SET`                                                  | String         | `"a,b"`                        | -                                      |