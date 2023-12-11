---
title: TiCDC Canal-JSON プロトコル
summary: TiCDC Canal-JSON プロトコルの概念と使用方法について学びます。

# TiCDC Canal-JSON プロトコル

Canal-JSON は、[Alibaba Canal](https://github.com/alibaba/canal) で定義されたデータ交換形式プロトコルです。このドキュメントでは、TiCDC での Canal-JSON データ形式の実装、TiDB 拡張フィールド、Canal-JSON データ形式の定義、公式 Canal との比較について学ぶことができます。

## Canal-JSON の使用

下流の Sink としてメッセージキュー (MQ) を使用する場合、`sink-uri` で Canal-JSON を指定できます。TiCDC は Event を基本単位として Canal-JSON メッセージをラップおよび構築し、TiDB のデータ変更イベントを下流に送信します。

次の 3 種類の Event があります。

* DDL Event: DDL 変更レコードを表します。上流の DDL ステートメントが正常に実行された後に送信されます。DDL Event はインデックスが 0 の MQ Partition に送信されます。
* DML Event: 行データ変更レコードを表します。行の変更が発生するとこのタイプの Event が送信されます。変更後の行の情報が含まれます。
* WATERMARK Event: 特定の時点を表します。この時点までに受信したイベントが完了したことを示します。これは TiDB 拡張フィールドに適用され、`sink-uri` で `enable-tidb-extension` を `true` に設定した場合にのみ有効です。

以下は、`Canal-JSON` を使用する例です。

{{< copyable "shell-regular" >}}

```shell
cdc cli changefeed create --server=http://127.0.0.1:8300 --changefeed-id="kafka-canal-json" --sink-uri="kafka://127.0.0.1:9092/topic-name?kafka-version=2.4.0&protocol=canal-json"
```

## TiDB 拡張フィールド

Canal-JSON プロトコルはもともと MySQL 向けに設計されています。TiDB 固有の CommitTS トランザクションの一意な識別子など、重要なフィールドが含まれていません。この問題を解決するために、TiCDC は Canal-JSON プロトコル形式に TiDB 拡張フィールドを追加します。`sink-uri` で `enable-tidb-extension` を `true` に設定した後 (`false` がデフォルト)、TiCDC は Canal-JSON メッセージの生成時に次のように動作します。

* TiCDC は `_tidb` というフィールドを含む DML Event および DDL Event メッセージを送信します。
* TiCDC は WATERMARK Event メッセージを送信します。

以下は例です。

{{< copyable "shell-regular" >}}

```shell
cdc cli changefeed create --server=http://127.0.0.1:8300 --changefeed-id="kafka-canal-json-enable-tidb-extension" --sink-uri="kafka://127.0.0.1:9092/topic-name?kafka-version=2.4.0&protocol=canal-json&enable-tidb-extension=true"
```

## メッセージ形式の定義

このセクションでは、DDL Event、DML Event、WATERMARK Event の形式、およびコンシューマ側でデータ解決がどのように行われるかについて説明します。

### DDL Event

TiCDC は DDL Event を次の Canal-JSON 形式にエンコードします。

```json
{
    "id": 0,
    "database": "test",
    "table": "",
    "pkNames": null,
    "isDdl": true,
    "type": "QUERY",
    "es": 1639633094670,
    "ts": 1639633095489,
    "sql": "drop database if exists test",
    "sqlType": null,
    "mysqlType": null,
    "data": null,
    "old": null,
    "_tidb": {     // TiDB 拡張フィールド
        "commitTs": 163963309467037594
    }
}
```

各フィールドの説明は以下の通りです。

| Field      | Type   | Description                                                             |
|:----------|:-------|:-------------------------------------------------------------------------|
| id        | Number | TiCDC ではデフォルト値は 0 です。                                            |
| database  | String | 行が位置するデータベースの名前                                              |
| table     | String | 行が位置するテーブルの名前                                                  |
| pkNames   | Array  | 主キーを構成するすべての列の名前                                           |
| isDdl     | Bool   | メッセージが DDL イベントかどうか                                            |
| type      | String | Canal-JSON で定義されたイベントの種類                                         |
| es        | Number | メッセージを生成したイベントの発生時刻 (13 ビットのミリ秒単位のタイムスタンプ)          |
| ts        | Number | TiCDC がメッセージを生成した時刻 (13 ビットのミリ秒単位のタイムスタンプ)             |
| sql       | String | `isDdl` が `true` の場合、対応する DDL ステートメントを記録                             |
| sqlType   | Object | `isDdl` が `false` の場合、各列のデータ型が Java でどのように表現されているかを記録      |
| mysqlType | object | `isDdl` が `false` の場合、各列のデータ型が MySQL でどのように表現されているかを記録    |
| data      | Object | `isDdl` が `false` の場合、各列の名前とデータ値を記録                                  |
| old       | Object | メッセージが更新イベントによって生成された場合のみ、更新前の各列の名前とデータ値を記録      |
| _tidb     | Object | TiDB 拡張フィールド。`enable-tidb-extension` を `true` に設定した場合にのみ存在します。`commitTs` の値は行の変更を引き起こしたトランザクションの TSO です。|

### DML Event

TiCDC は、DML データ変更イベントの 1 行を次のようにエンコードします。

```json
{
    "id": 0,
    "database": "test",
    "table": "tp_int",
    "pkNames": [
        "id"
    ],
    "isDdl": false,
    "type": "INSERT",
    "es": 1639633141221,
    "ts": 1639633142960,
    "sql": "",
    "sqlType": {
        "c_bigint": -5,
        "c_int": 4,
        "c_mediumint": 4,
        "c_smallint": 5,
        "c_tinyint": -6,
        "id": 4
    },
    "mysqlType": {
        "c_bigint": "bigint",
        "c_int": "int",
        "c_mediumint": "mediumint",
        "c_smallint": "smallint",
        "c_tinyint": "tinyint",
        "id": "int"
    },
    "data": [
        {
            "c_bigint": "9223372036854775807",
            "c_int": "2147483647",
            "c_mediumint": "8388607",
            "c_smallint": "32767",
            "c_tinyint": "127",
            "id": "2"
        }
    ],
    "old": null,
    "_tidb": {     // TiDB 拡張フィールド
        "commitTs": 163963314122145239
    }
}
```

### WATERMARK Event

TiCDC は、`enable-tidb-extension` を `true` に設定した場合にのみ WATERMARK Event を送信します。`type` フィールドの値は `TIDB_WATERMARK` です。この Event には `_tidb` フィールドが含まれ、そのフィールドには `watermarkTs` というパラメータのみが含まれます。`watermarkTs` の値は Event の送信時に記録された TSO です。

このタイプの Event を受信すると、`commitTs` が `watermarkTs` よりも小さいすべての Event が送信されています。TiCDC は「少なくとも一度」のセマンティクスを提供するため、データが繰り返し送信される場合があります。`commitTs` が `watermarkTs` よりも小さい後続の Event を受信した場合、この Event を安全に無視できます。

以下は WATERMARK Event の例です。

```json
{
    "id": 0,
    "database": "",
    "table": "",
    "pkNames": null,
    "isDdl": false,
    "type": "TIDB_WATERMARK",
    "es": 1640007049196,
    "ts": 1640007050284,
    "sql": "",
    "sqlType": null,
    "mysqlType": null,
    "data": null,
    "old": null,
    "_tidb": {     // TiDB 拡張フィールド
        "watermarkTs": 429918007904436226
    }
}
```

### コンシューマ側でのデータ解決

上記の例からわかるように、Canal-JSON は統一されたデータ形式を持ち、異なる Event タイプに対して異なるフィールド埋め込みルールがあります。統一した方法でこの JSON 形式のデータを解決し、その後フィールドの値をチェックすることで Event タイプを判別できます。

* `isDdl` が `true` の場合、メッセージには DDL Event が含まれています。
* `isDdl` が `false` の場合、さらに `type` フィールドを確認する必要があります。`type` が `TIDB_WATERMARK` の場合、WATERMARK Event です。それ以外の場合、DML Event です。

## フィールドの説明

Canal-JSON 形式では、`mysqlType` フィールドおよび `sqlType` フィールドに対応するデータ型が記録されています。

### MySQL データ型フィールド
`mysqlType` フィールドでは、Canal-JSON フォーマットは、各カラムの MySQL タイプの文字列を記録します。詳細は[TiDB データ型](/data-type-overview.md)を参照してください。

### SQL タイプ フィールド

`sqlType` フィールドでは、Canal-JSON フォーマットは、各カラムの Java SQL タイプを記録します。これは JDBC のデータの対応するデータ型です。その値は MySQL タイプと特定のデータ値から計算できます。マッピングは以下の通りです:

| MySQL タイプ | Java SQL タイプ コード |
| :----------| :----------------- |
| Boolean    | -6                 |
| Float      | 7                  |
| Double     | 8                  |
| Decimal    | 3                  |
| Char       | 1                  |
| Varchar    | 12                 |
| Binary     | 2004               |
| Varbinary  | 2004               |
| Tinytext   | 2005               |
| Text       | 2005               |
| Mediumtext | 2005               |
| Longtext   | 2005               |
| Tinyblob   | 2004               |
| Blob       | 2004               |
| Mediumblob | 2004               |
| Longblob   | 2004               |
| Date       | 91                 |
| Datetime   | 93                 |
| Timestamp  | 93                 |
| Time       | 92                 |
| Year       | 12                 |
| Enum       | 4                  |
| Set        | -7                 |
| Bit        | -7                 |
| JSON       | 12                 |

## 整数型

整数型には、`Unsigned` 制約と値の範囲を考慮する必要があり、それぞれ異なる Java SQL タイプ コードに対応します。以下の表に示すように:

| MySQL タイプ 文字列  | 値の範囲                                 | Java SQL タイプ コード |
| :------------------| :------------------------------------------ | :----------------- |
| tinyint            | [-128, 127]                                 | -6                 |
| tinyint unsigned   | [0, 127]                                    | -6                 |
| tinyint unsigned   | [128, 255]                                  | 5                  |
| smallint           | [-32768, 32767]                             | 5                  |
| smallint unsigned  | [0, 32767]                                  | 5                  |
| smallint unsigned  | [32768, 65535]                              | 4                  |
| mediumint          | [-8388608, 8388607]                         | 4                  |
| mediumint unsigned | [0, 8388607]                                | 4                  |
| mediumint unsigned | [8388608, 16777215]                         | 4                  |
| int                | [-2147483648, 2147483647]                   | 4                  |
| int unsigned       | [0, 2147483647]                             | 4                  |
| int unsigned       | [2147483648, 4294967295]                    | -5                 |
| bigint             | [-9223372036854775808, 9223372036854775807] | -5                 |
| bigint unsigned    | [0, 9223372036854775807]                    | -5                 |
| bigint unsigned    | [9223372036854775808, 18446744073709551615] | 3                  |

以下の表は、TiCDC の Java SQL タイプとそのコードのマッピング関係を示しています。

| Java SQL タイプ | Java SQL タイプ コード |
| :-------------| :------------------|
| CHAR          | 1                  |
| DECIMAL       | 3                  |
| INTEGER       | 4                  |
| SMALLINT      | 5                  |
| REAL          | 7                  |
| DOUBLE        | 8                  |
| VARCHAR       | 12                 |
| DATE          | 91                 |
| TIME          | 92                 |
| TIMESTAMP     | 93                 |
| BLOB          | 2004               |
| CLOB          | 2005               |
| BIGINT        | -5                 |
| TINYINT       | -6                 |
| Bit           | -7                 |

Java SQL タイプに関する詳細については、[Java SQL Class Types](https://docs.oracle.com/javase/8/docs/api/java/sql/Types.html)を参照してください。

## バイナリと Blob タイプ

TiCDC は、[バイナリタイプ](/data-type-string.md#binary-type)を、Canal-JSON フォーマットで以下のようにそれぞれのバイトを文字表現に変換してエンコードします:

- 表示可能な文字は ISO/IEC 8859-1 文字エンコーディングを使用して表されます。
- 非表示可能な文字および HTML で特別な意味を持つ文字は、それぞれの UTF-8 のエスケープシーケンスを使用して表されます。

以下の表は、詳細な表現情報を示しています。

| 文字タイプ                | 値の範囲 | 文字表現 |
| :---------------------------| :-----------| :---------------------|
| 制御文字          | [0, 31]     | UTF-8 エスケープ（たとえば `\u0000` から `\u001F`） |
| 横タブ              | [9]         | `\t`                    |
| 改行                   | [10]        | `\n`                    |
| キャリッジ・リターン              | [13]       | `\r`                    |
| 表示可能文字        | [32, 127]   | 文字表現（たとえば `A`） |
| アンパサンド                   | [38]        | `\u0026`                |
| 小なり記号              | [60]        | `\u0038`                |
| 大なり記号           | [62]        | `\u003E`                |
| 拡張制御文字 | [128, 159]  | 文字表現   |
| ISO 8859-1（Latin-1）        | [160, 255]  | 文字表現   |

### エンコーディングの例

たとえば、以下の 16 バイト `[5 7 10 15 36 50 43 99 120 60 38 255 254 45 55 70]` が `c_varbinary` という `VARBINARY` カラムに格納されている場合に、Canal-JSON の `Update` イベントで以下のようにエンコードされます:

```json
{
    ...
    "data": [
        {
            ...
            "c_varbinary": "\u0005\u0007\n\u000f$2+cx\u003c\u0026ÿþ-7F"
        }
    ]
    ...
}
```

## TiCDC Canal-JSON と公式 Canal の比較

TiCDC が Canal-JSON データ形式を実装する方法、`Update` イベントと `mysqlType` フィールドを含め、公式 Canal とは異なります。以下の表は、主な違いを示しています。

| 項目            | TiCDC Canal-JSON                                                                                                                             | Canal                                |
|:----------------|:---------------------------------------------------------------------------------------------------------------------------------------------|:-------------------------------------|
| `Update` タイプのイベント  | デフォルトでは、`old` フィールドにすべてのカラムデータが含まれます。`only_output_updated_columns` が `true` の場合、`old` フィールドには変更されたカラムデータのみが含まれます。  | `old` フィールドには変更されたカラムデータのみが含まれます    |
| `mysqlType` フィールド  | パラメータを持つタイプの場合、タイプパラメータの情報が含まれません                                                         | パラメータを持つタイプの場合、タイプパラメータの完全な情報が含まれます    |

### `Update` タイプのイベント

`Update` タイプのイベントの場合:

- TiCDC では、`old` フィールドにすべてのカラムデータが含まれます
- 公式 Canal では、`old` フィールドには変更されたカラムデータのみが含まれます
```
            "c_smallint": "32767",
            "c_tinyint": "127",           // Modified column
            "id": "2"
        }
    ]
}
```


公式のCanalでは、出力イベントメッセージの`old`フィールドには、変更された列のデータのみが含まれます。以下に示すように。

```json
{
    "id": 0,
    ...
    "type": "UPDATE",
    ...
    "sqlType": {
        ...
    },
    "mysqlType": {
        ...
    },
    "data": [
        {
            "c_bigint": "9223372036854775807",
            "c_int": "0",
            "c_mediumint": "8388607",
            "c_smallint": "32767",
            "c_tinyint": "0",
            "id": "2"
        }
    ],
    "old": [                              // Canalでは、このフィールドに変更された列のデータのみが含まれます。
        {
            "c_int": "2147483647",        // Modified column
            "c_tinyint": "127",           // Modified column
        }
    ]
}
```

### `mysqlType`フィールド

`mysqlType`フィールドについて、タイプにパラメータが含まれる場合、公式のCanalにはタイプパラメータの完全な情報が含まれます。一方、TiCDCにはそのような情報は含まれません。

次の例では、テーブル定義のSQLステートメントに、`decimal`、`char`、`varchar`、`enum`の各列用のパラメータが含まれています。TiCDCと公式のCanalによって生成されたCanal-JSON形式を比較することで、TiCDCは`mysqlType`フィールドに基本的なMySQL情報のみを含んでいることがわかります。タイプパラメータの完全な情報が必要な場合は、他の手段で実装する必要があります。

以下のSQLステートメントが上流のTiDBで順次実行されたと仮定します。

```sql
create table t (
    id     int auto_increment,
    c_decimal    decimal(10, 4) null,
    c_char       char(16)      null,
    c_varchar    varchar(16)   null,
    c_binary     binary(16)    null,
    c_varbinary  varbinary(16) null,
    c_enum enum('a','b','c') null,
    c_set  set('a','b','c')  null,
    c_bit  bit(64)            null,
    constraint pk
        primary key (id)
);

insert into t (c_decimal, c_char, c_varchar, c_binary, c_varbinary, c_enum, c_set, c_bit)
values (123.456, "abc", "abc", "abc", "abc", 'a', 'a,b', b'1000001');
```

TiCDCの出力は以下のようになります。

```json
{
    "id": 0,
    ...
    "isDdl": false,
    "sqlType": {
        ...
    },
    "mysqlType": {
        "c_binary": "binary",
        "c_bit": "bit",
        "c_char": "char",
        "c_decimal": "decimal",
        "c_enum": "enum",
        "c_set": "set",
        "c_varbinary": "varbinary",
        "c_varchar": "varchar",
        "id": "int"
    },
    "data": [
        {
            ...
        }
    ],
    "old": null,
}
```

公式Canalの出力は以下の通りです。

```json
{
    "id": 0,
    ...
    "isDdl": false,
    "sqlType": {
        ...
    },
    "mysqlType": {
        "c_binary": "binary(16)",
        "c_bit": "bit(64)",
        "c_char": "char(16)",
        "c_decimal": "decimal(10, 4)",
        "c_enum": "enum('a','b','c')",
        "c_set": "set('a','b','c')",
        "c_varbinary": "varbinary(16)",
        "c_varchar": "varchar(16)",
        "id": "int"
    },
    "data": [
        {
            ...
        }
    ],
    "old": null,
}
```

## TiCDC Canal-JSONの変更

### `Delete`イベントの`old`フィールドの変更

v5.4.0から、`Delete`イベントの`old`フィールドが変更されました。

以下は`Delete`イベントメッセージです。v5.4.0以前では、`old`フィールドには"data"フィールドと同じ内容が含まれていました。v5.4.0以降のバージョンでは、`old`フィールドはnullに設定されます。削除されたデータは"data"フィールドを使用して取得できます。

```
{
    "id": 0,
    "database": "test",
    ...
    "type": "DELETE",
    ...
    "sqlType": {
        ...
    },
    "mysqlType": {
        ...
    },
    "data": [
        {
            "c_bigint": "9223372036854775807",
            "c_int": "0",
            "c_mediumint": "8388607",
            "c_smallint": "32767",
            "c_tinyint": "0",
            "id": "2"
        }
    ],
    "old": null,
    // 以下はv5.4.0以前の例です。`old`フィールドには"data"フィールドと同じ内容が含まれていました。
    "old": [
        {
            "c_bigint": "9223372036854775807",
            "c_int": "0",
            "c_mediumint": "8388607",
            "c_smallint": "32767",
            "c_tinyint": "0",
            "id": "2"
        }
    ]
}
```