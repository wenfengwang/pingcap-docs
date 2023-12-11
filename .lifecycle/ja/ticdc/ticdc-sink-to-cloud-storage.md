---
title: ストレージサービスへのデータの複製
summary: TiCDCを使用してデータをストレージサービスに複製する方法と、複製されたデータの保存パスについて学びます。
---

# ストレージサービスへのデータの複製

TiDB v6.5.0から、TiCDCはAmazon S3、GCS、Azure Blob Storage、NFSを含むストレージサービスに行の変更イベントを保存することをサポートしています。このドキュメントでは、TiCDCを使用してこのようなストレージサービスに増分データを複製する changefeed の作成方法と、データの保存方法について説明します。このドキュメントの構成は以下の通りです：

- [ストレージサービスへのデータの複製方法](#ストレージサービスへのデータの複製).
- [ストレージサービスでデータが保存される方法](#保存パスの構造).

## ストレージサービスへのデータの複製

次のコマンドを実行して changefeed タスクを作成します：

```shell
cdc cli changefeed create \
    --server=http://10.0.10.25:8300 \
    --sink-uri="s3://logbucket/storage_test?protocol=canal-json" \
    --changefeed-id="simple-replication-task"
```

出力は次のようになります：

```shell
Info: {"upstream_id":7171388873935111376,"namespace":"default","id":"simple-replication-task","sink_uri":"s3://logbucket/storage_test?protocol=canal-json","create_time":"2023-11-28T18:52:05.566016967+08:00","start_ts":437706850431664129,"engine":"unified","config":{"case_sensitive":false,"enable_old_value":true,"force_replicate":false,"ignore_ineligible_table":false,"check_gc_safe_point":true,"enable_sync_point":false,"sync_point_interval":600000000000,"sync_point_retention":86400000000000,"filter":{"rules":["*.*"],"event_filters":null},"mounter":{"worker_num":16},"sink":{"protocol":"canal-json","schema_registry":"","csv":{"delimiter":",","quote":"\"","null":"\\N","include_commit_ts":false},"column_selectors":null,"transaction_atomicity":"none","encoder_concurrency":16,"terminator":"\r\n","date_separator":"none","enable_partition_separator":false},"consistent":{"level":"none","max_log_size":64,"flush_interval":2000,"storage":""}},"state":"normal","creator_version":"v7.5.0"}
```

- `--server`: TiCDC クラスタ内の任意の TiCDC サーバのアドレス。
- `--changefeed-id`: changefeed のID。フォーマットは `^[a-zA-Z0-9]+(\-[a-zA-Z0-9]+)*$` 正規表現に一致する必要があります。指定されていない場合は、TiCDC は自動的に UUID (version 4 フォーマット) をIDとして生成します。
- `--sink-uri`: changefeed のダウンストリーム アドレス。詳細は[シンク URI の構成](#シンク-URI-の構成)を参照してください。
- `--start-ts`: changefeed の開始 TSO。TiCDC はこの TSO からデータを取得し始めます。デフォルト値は現在の時間です。
- `--target-ts`: changefeed の終了 TSO。TiCDC はこの TSO までデータを取得するのを停止します。デフォルト値は空であり、TiCDC はデータを自動的に停止しません。
- `--config`: changefeed の構成ファイル。詳細は[TiCDC changefeed configuration parameters](/ticdc/ticdc-changefeed-config.md)を参照してください。

## シンク URI の構成

このセクションでは、Amazon S3、GCS、Azure Blob Storage、NFSを含むストレージサービスのためのシンク URI の構成方法について説明します。シンク URI は、TiCDC ターゲットシステムの接続情報を指定するために使用されます。フォーマットは次の通りです：

```shell
[scheme]://[host]/[path]?[query_parameters]
```
URI 内の `[query_parameters]` では、次のパラメータを構成できます：

| パラメータ | 説明 | デフォルト値 | 値の範囲 |
| :---------| :---------- | :------------ | :---------- |
| `worker-count` | ダウンストリームのクラウドストレージへデータ変更を保存する並行性。  | `16` | `[1, 512]` |
| `flush-interval` | ダウンストリームのクラウドストレージへデータ変更を保存する間隔。   | `5s` | `[2s, 10m]` |
| `file-size` | このパラメータの値を超えるバイト数のデータ変更ファイルはクラウドストレージへ保存されます。 | `67108864` | `[1048576, 536870912]` |
| `protocol` | ダウンストリームに送信されるメッセージのプロトコル形式。  | N/A |  `canal-json` および `csv` |
| `enable-tidb-extension` | `protocol` を `canal-json` に設定し、`enable-tidb-extension` を `true` にした場合、TiCDC は[WATERMARK イベント](/ticdc/ticdc-canal-json.md#watermark-event)を送信し、[Canal-JSON メッセージにTiDB拡張フィールド](/ticdc/ticdc-canal-json.md#tidb-extension-field)を追加します。 | `false` | `false` および `true` |

> **注意:**
>
> `flush-interval` または `file-size` のいずれかの要件を満たしたときにデータ変更ファイルがダウンストリームに保存されます。
> `protocol` パラメータは必須です。TiCDC がchangefeed を作成する際にこのパラメータを受信しない場合、 `CDC:ErrSinkUnknownProtocol` エラーが返されます。

### 外部ストレージ用のシンク URI の構成

次に、Amazon S3 の例の構成を示します：

```shell
--sink-uri="s3://bucket/prefix?protocol=canal-json"
```

次に、GCS の例の構成を示します：

```shell
--sink-uri="gcs://bucket/prefix?protocol=canal-json"
```

次に、Azure Blob Storage の例の構成を示します：

```shell
--sink-uri="azure://bucket/prefix?protocol=canal-json"
```

> **ヒント:**
>
> TiCDCにおけるAmazon S3、GCS、Azure Blob StorageのURIパラメータの詳細については、[外部ストレージサービスのURIフォーマット](/external-storage-uri.md)を参照してください。

### NFS用のシンク URI の構成

次に、NFSの例の構成を示します：

```shell
--sink-uri="file:///my-directory/prefix?protocol=canal-json"
```

## データ保存パスの構造

このセクションでは、データ変更レコード、メタデータ、DDLイベントの保存パスの構造について説明します。

### データ変更レコード

データ変更レコードは、次のパスに保存されます：

```shell
{scheme}://{prefix}/{schema}/{table}/{table-version-separator}/{partition-separator}/{date-separator}/CDC{num}.{extension}
```

- `scheme`: ストレージのタイプを指定します。例: `s3`、`gcs`、`azure`、または `file`。
- `prefix`: ユーザー定義の親ディレクトリを指定します。例: <code>s3://**bucket/bbb/ccc**</code>。
- `schema`: スキーマ名を指定します。例: <code>s3://bucket/bbb/ccc/**test**</code>。
- `table`: テーブル名を指定します。例: <code>s3://bucket/bbb/ccc/test/**table1**</code>。
- `table-version-separator`: パスをテーブルバージョンごとにセパレートするセパレータを指定します。例: <code>s3://bucket/bbb/ccc/test/table1/**9999**</code>。
- `partition-separator`: パスをテーブルパーティションごとにセパレートするセパレータを指定します。例: <code>s3://bucket/bbb/ccc/test/table1/9999/**20**</code>。
- `date-separator`: ファイルをトランザクションのコミット日によって分類します。デフォルト値は`day`です。値の選択肢は次の通りです：
    - `none`: `date-separator` なし。例: `s3://bucket/bbb/ccc/test/table1/9999` にすべての`test.table1`バージョンが`9999`のファイルが保存されます。
    - `year`: セパレータはトランザクションのコミット日の年です。例: <code>s3://bucket/bbb/ccc/test/table1/9999/**2022**</code>。
    - `month`: セパレータはトランザクションのコミット日の年と月です。例: <code>s3://bucket/bbb/ccc/test/table1/9999/**2022-01**</code>。
    - `day`: セパレータはトランザクションのコミット日の年、月、日です。例: <code>s3://bucket/bbb/ccc/test/table1/9999/**2022-01-02**</code>。
- `num`: データ変更を記録するファイルのシリアル番号を保存します。例: <code>s3://bucket/bbb/ccc/test/table1/9999/2022-01-02/CDC**000005**.csv</code>。
- `extension`: ファイルの拡張子を指定します。TiDB v6.5.0ではCSVおよびCanal-JSONフォーマットがサポートされています。

> **注意:**

> テーブルバージョンは、上流テーブルでDDL操作が行われた後にのみ変更され、新しいテーブルバージョンは、上流のTiDBがDDLの実行を完了したTSOです。ただし、テーブルバージョンの変更はテーブルスキーマの変更を意味しません。たとえば、列にコメントを追加しても、スキーマファイルの内容が変更されるわけではありません。

### インデックスファイル

インデックスファイルは、書き込まれたデータが間違って上書きされないようにするために使用されます。これは、データ変更レコードと同じパスに保存されます。

```shell
{scheme}://{prefix}/{schema}/{table}/{table-version-separator}/{partition-separator}/{date-separator}/meta/CDC.index
```

インデックスファイルには、現在のディレクトリで使用されている最大のファイル名が記録されます。例えば:

```
CDC000005.csv
```

この例では、このディレクトリ内の`CDC000001.csv`から`CDC000004.csv`までのファイルが使用されています。TiCDCクラスターでテーブルのスケジューリングやノードの再起動が発生すると、新しいノードはインデックスファイルを読み込み、`CDC000005.csv`が使用されているかどうかを判断します。使用されていない場合、新しいノードは`CDC000005.csv`からファイルを書き込みます。使用されている場合、他のノードによって書き込まれたデータの上書きを防ぐために`CDC000006.csv`から書き込みを開始します。

### メタデータ

メタデータは、次のパスに保存されます。

```shell
{protocol}://{prefix}/metadata
```

メタデータはJSON形式のファイルで以下のようになります。

```json
{
    "checkpoint-ts":433305438660591626
}
```

- `checkpoint-ts`: `commit-ts`よりも小さいトランザクションは、下流のターゲットストレージに書き込まれます。

### DDLイベント

### テーブルレベルのDDLイベント

上流のテーブルのDDLイベントでテーブルバージョンが変更された場合、TiCDCは自動的に以下を実行します:

- データ変更レコードを書き込む新しいパスに切り替えます。たとえば、`test.table1`のバージョンが`441349361156227074`に変更されると、TiCDCは`test/table1/441349361156227074/2022-01-02/`パスにデータ変更レコードを書き込むように切り替えます。
- テーブルスキーマ情報を保存するために、以下のパスにスキーマファイルを生成します:

    ```shell
    {scheme}://{prefix}/{schema}/{table}/meta/schema_{table-version}_{hash}.json
    ```

`schema_441349361156227074_3131721815.json`のスキーマファイルを例に取ると、このファイル内のテーブルスキーマ情報は以下のようになります:

```json
{
    "Table":"table1",
    "Schema":"test",
    "Version":1,
    "TableVersion":441349361156227074,
    "Query":"ALTER TABLE test.table1 ADD OfficeLocation blob(20)",
    "Type":5,
    "TableColumns":[
        {
            "ColumnName":"Id",
            "ColumnType":"INT",
            "ColumnNullable":"false",
            "ColumnIsPk":"true"
        },
        {
            "ColumnName":"LastName",
            "ColumnType":"CHAR",
            "ColumnLength":"20"
        },
        {
            "ColumnName":"FirstName",
            "ColumnType":"VARCHAR",
            "ColumnLength":"30"
        },
        {
            "ColumnName":"HireDate",
            "ColumnType":"DATETIME"
        },
        {
            "ColumnName":"OfficeLocation",
            "ColumnType":"BLOB",
            "ColumnLength":"20"
        }
    ],
    "TableColumnsTotal":"5"
}
```

- `Table`: テーブル名。
- `Schema`: スキーマ名。
- `Version`: ストレージシンクのプロトコルバージョン。
- `TableVersion`: テーブルバージョン。
- `Query`: DDLステートメント。
- `Type`: DDLタイプ。
- `TableColumns`: 1つ以上のマップの配列で、それぞれがソーステーブルの列を記述しています。
    - `ColumnName`: 列名。
    - `ColumnType`: 列のタイプ。詳細は[データ型](#データ型)を参照してください。
    - `ColumnLength`: 列の長さ。詳細は[データ型](#データ型)を参照してください。
    - `ColumnPrecision`: 列の精度。詳細は[データ型](#データ型)を参照してください。
    - `ColumnScale`: 小数点以下の桁数（スケール）。詳細は[データ型](#データ型)を参照してください。
    - `ColumnNullable`: このオプションの値が`true`の場合、列にNULLを許可します。
    - `ColumnIsPk`: このオプションの値が`true`の場合、列は主キーの一部です。
- `TableColumnsTotal`: `TableColumns`配列のサイズ。

### データベースレベルのDDLイベント

上流のデータベースでデータベースレベルのDDLイベントが発生すると、TiCDCは自動的に以下のパスにデータベーススキーマ情報を格納するためのスキーマファイルを生成します:

```shell
{scheme}://{prefix}/{schema}/meta/schema_{table-version}_{hash}.json
```

`schema_441349361156227000_3131721815.json`のスキーマファイルを例に取ると、このファイル内のデータベーススキーマ情報は以下のようになります:

```json
{
  "Table": "",
  "Schema": "schema1",
  "Version": 1,
  "TableVersion": 441349361156227000,
  "Query": "CREATE DATABASE `schema1`",
  "Type": 1,
  "TableColumns": null,
  "TableColumnsTotal": 0
}
```

### データ型

このセクションでは、`schema_{table-version}_{hash}.json`ファイル（以下、「スキーマファイル」と呼ぶ）で使用されるデータ型について説明します。これらのデータ型は`T(M[, D])`と定義されます。詳細については、[データ型の概要](/data-type-overview.md)を参照してください。

#### 整数型

TiDBの整数型は`IT[(M)] [UNSIGNED]`と定義されます。ここで、

- `IT`は整数型であり、`TINYINT`、`SMALLINT`、`MEDIUMINT`、`INT`、`BIGINT`、または`BIT`になります。
- `M`はタイプの表示幅です。

スキーマファイルでは、整数型は以下のように定義されます:

```json
{
    "ColumnName":"COL1",
    "ColumnType":"{IT} [UNSIGNED]",
    "ColumnPrecision":"{M}"
}
```

#### 10進数型

TiDBの10進数型は`DT[(M,D)][UNSIGNED]`と定義されます。ここで、

- `DT`は浮動小数点型で、`FLOAT`、`DOUBLE`、`DECIMAL`、または `NUMERIC`になります。
- `M`はデータ型の精度、または小数点以下の桁数です。
- `D`は小数点以下の桁数です。

スキーマファイルでは、10進数型は以下のように定義されます:

```json
{
    "ColumnName":"COL1",
    "ColumnType":"{DT} [UNSIGNED]",
    "ColumnPrecision":"{M}",
    "ColumnScale":"{D}"
}
```

#### 日付および時刻型

TiDBの日付型は`DT`と定義されます。ここで、

- `DT`は`DATE`または`YEAR`の日付型です。

日付型は、スキーマファイルでは以下のように定義されます:

```json
{
    "ColumnName":"COL1",
    "ColumnType":"{DT}"
}
```

TiDBの時刻型は`TT[(M)]`と定義されます。ここで、

- `TT`は`TIME`、`DATETIME`、または`TIMESTAMP`の時刻型です。
- `M`は0から6の範囲の秒の精度です。

時刻型は、スキーマファイルでは以下のように定義されます:

```json
{
    "ColumnName":"COL1",
    "ColumnType":"{TT}",
    "ColumnScale":"{M}"
}
```

#### 文字列型

TiDBの文字列型は`ST[(M)]`と定義されます。ここで、

- `ST`は`CHAR`、`VARCHAR`、`TEXT`、`BINARY`、`BLOB`、または`JSON`の文字列型です。
- `M`は文字列の最大長です。

文字列型は、スキーマファイルでは以下のように定義されます:

```json
{
    "ColumnName":"COL1",
    "ColumnType":"{ST}",
    "ColumnLength":"{M}"
}
```

#### 列挙型および集合型

列挙型および集合型は、スキーマファイルでは以下のように定義されます:

```json
{
    "ColumnName":"COL1",
    "ColumnType":"{ENUM/SET}",
}
```