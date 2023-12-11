---
title: TiCDC Avro Protocol
summary: TiCDC Avroプロトコルの概念と使用方法について学ぶ。

# TiCDC Avro Protocol

Avroは[Apache Avro™](https://avro.apache.org/)によって定義されたデータ交換フォーマットプロトコルであり、[Confluent Platform](https://docs.confluent.io/platform/current/platform.html)によってデフォルトのデータ交換フォーマットとして選択されています。本書では、TiCDCにおけるAvroデータ形式の実装、TiDB拡張フィールド、Avroデータ形式の定義、およびAvroと[Confluent Schema Registry](https://docs.confluent.io/platform/current/schema-registry/index.html)との相互作用について説明します。

> **警告:**
>
> v7.3.0から、[有効なインデックスなしでテーブルを複製](/ticdc/ticdc-manage-changefeed.md#replicate-tables-without-a-valid-index)するようにTiCDCを有効にすると、Avroプロトコルを使用するチェンジフィードを作成するとTiCDCはエラーを報告します。

## Avroの使用

メッセージキュー（MQ）を下流シンクとして使用する場合、`sink-uri`でAvroを指定できます。TiCDCはTiDB DMLイベントをキャプチャし、これらのイベントからAvroメッセージを作成し、メッセージを下流に送信します。Avroはスキーマの変更を検出すると、最新のスキーマをSchema Registryに登録します。

次のは、Avroを使用した設定例です：

{{< copyable "shell-regular" >}}

```shell
cdc cli changefeed create --server=http://127.0.0.1:8300 --changefeed-id="kafka-avro" --sink-uri="kafka://127.0.0.1:9092/topic-name?protocol=avro" --schema-registry=http://127.0.0.1:8081 --config changefeed_config.toml
```

```shell
[sink]
dispatchers = [
 {matcher = ['*.*'], topic = "tidb_{schema}_{table}"},
]
```

`--schema-registry`の値は`https`プロトコルと`username:password`認証をサポートしており、例えば`--schema-registry=https://username:password@schema-registry-uri.com`です。ユーザ名とパスワードはURLエンコードする必要があります。

## TiDB拡張フィールド

デフォルトでは、AvroはDMLイベントで変更された行のデータのみを収集し、データ変更のタイプやTiDB固有のCommitTS（トランザクションの一意な識別子）を収集しません。この問題を解決するために、TiCDCはAvroプロトコルメッセージに次の3つのTiDB拡張フィールドを導入しています。`sink-uri`で`enable-tidb-extension`を`true`に設定すると（デフォルトは`false`）、TiCDCはメッセージ生成時にAvroメッセージにこれら3つのフィールドを追加します。

- `_tidb_op`: DMLタイプ。"c"は挿入を示し、"u"は更新を示します。
- `_tidb_commit_ts`: トランザクションの一意な識別子。
- `_tidb_commit_physical_time`: トランザクション識別子内の物理的タイムスタンプ。

次のは、構成の例です：

{{< copyable "shell-regular" >}}

```shell
cdc cli changefeed create --server=http://127.0.0.1:8300 --changefeed-id="kafka-avro-enable-extension" --sink-uri="kafka://127.0.0.1:9092/topic-name?protocol=avro&enable-tidb-extension=true" --schema-registry=http://127.0.0.1:8081 --config changefeed_config.toml
```

```shell
[sink]
dispatchers = [
 {matcher = ['*.*'], topic = "tidb_{schema}_{table}"},
]
```

## データ形式の定義

TiCDCはDMLイベントをKafkaイベントに変換し、イベントのキーと値はAvroプロトコルに従ってエンコードされます。

### キーデータ形式

```
{
    "name":"{{TableName}}",
    "namespace":"{{Namespace}}",
    "type":"record",
    "fields":[
        {{ColumnValueBlock}},
        {{ColumnValueBlock}},
    ]
}
```

- `{{TableName}}`はイベントが発生したテーブルの名前を示します。
- `{{Namespace}}`はAvroの名前空間です。
- `{{ColumnValueBlock}}`は各列のデータの形式を定義します。

キーの`fields`にはプライマリキー列またはユニークインデックス列のみが含まれます。

### 値データ形式

```
{
    "name":"{{TableName}}",
    "namespace":"{{Namespace}}",
    "type":"record",
    "fields":[
        {{ColumnValueBlock}},
        {{ColumnValueBlock}},
    ]
}
```

値のデータ形式はデフォルトではキーのデータ形式と同じです。しかし、値の`fields`には、プライマリキー列だけでなくすべての列が含まれます。

[`enable-tidb-extension`](#tidb-extension-fields)を有効にした後、値のデータ形式は次のようになります：

```
{
    "name":"{{TableName}}",
    "namespace":"{{Namespace}}",
    "type":"record",
    "fields":[
        {{ColumnValueBlock}},
        {{ColumnValueBlock}},
        {
            "name":"_tidb_op",
            "type":"string"
        },
        {
            "name":"_tidb_commit_ts",
            "type":"long"
        },
        {
            "name":"_tidb_commit_physical_time",
            "type":"long"
        }
    ]
}
```

`enable-tidb-extension`を無効にした値のデータ形式と比較すると、3つの新しいフィールドが追加されます：`_tidb_op`、`_tidb_commit_ts`、およ `_tidb_commit_physical_time`。

### 列データ形式

列データは、キー/値データ形式の`{{ColumnValueBlock}}`部分です。TiCDCはSQLタイプに基づいて列データ形式を生成します。基本的な列データ形式は次の通りです：

```
{
    "name":"{{ColumnName}}",
    "type":{
        "connect.parameters":{
            "tidb_type":"{{TIDB_TYPE}}"
        },
        "type":"{{AVRO_TYPE}}"
    }
}
```

1つの列がNULLである可能性がある場合、列データ形式は以下のようになります：

```
{
    "default":null,
    "name":"{{ColumnName}}",
    "type":[
        "null",
        {
            "connect.parameters":{
                "tidb_type":"{{TIDB_TYPE}}"
            },
            "type":"{{AVRO_TYPE}}"
        }
    ]
}
```

- `{{ColumnName}}`は列名を示します。
- `{{TIDB_TYPE}}`はSQLタイプとの1対1のマッピングではないTiDBのタイプを示します。
- `{{AVRO_TYPE}}`は[avro spec](https://avro.apache.org/docs/current/spec.html)でのタイプを示します。

| SQL TYPE   | TIDB_TYPE | AVRO_TYPE | 説明                                                                                                                   |
|------------|-----------|-----------|-------------------------------------------------------------------------------------------------------------------------|
| BOOL       | INT       | int       |                                                                                                                         |
| TINYINT    | INT       | int       | 無符号の場合、TIDB_TYPEはINT UNSIGNEDです。                                                                            |
| SMALLINT   | INT       | int       | 無符号の場合、TIDB_TYPEはINT UNSIGNEDです。                                                                            |
| MEDIUMINT  | INT       | int       | 無符号の場合、TIDB_TYPEはINT UNSIGNEDです。                                                                            |
| INT        | INT       | int       | 無符号の場合、TIDB_TYPEはINT UNSIGNEDで、AVRO_TYPEはlongです。                                                      |
| BIGINT     | BIGINT    | long      | 無符号の場合、TIDB_TYPEはBIGINT UNSIGNEDです。 `avro-bigint-unsigned-handling-mode`がstringの場合、AVRO_TYPEはstringです。 |
| TINYBLOB   | BLOB      | bytes     |  -                                                                                                                     |
| BLOB       | BLOB      | bytes     |  -                                                                                                                     |
| MEDIUMBLOB | BLOB      | bytes     |  -                                                                                                                     |
| LONGBLOB   | BLOB      | bytes     |  -                                                                                                                     |
| BINARY     | BLOB      | bytes     |  -                                                                                                                     |
| VARBINARY  | BLOB      | bytes     |  -                                                                                                                     |
| TINYTEXT   | TEXT      | string    |  -                                                                                                                     |
| TEXT       | TEXT      | string    |  -                                                                                                                     |
| MEDIUMTEXT | TEXT      | string    |  -                                                                                                                     |
| LONGTEXT   | TEXT      | string    |  -                                                                                                                     |
| CHAR       | TEXT      | string    |  -                                                                                                                     |
| VARCHAR    | TEXT      | string    |  -                                                                                                                     |
| FLOAT      | FLOAT     | double    |  -                                                                                                                     |
| DOUBLE     | DOUBLE    | double    |  -                                                                                                                     |
| DATE       | DATE      | string    |  -                                                                                                                     |
| DATETIME   | DATETIME  | string    |  -                                                                                                                     |
| TIMESTAMP  | TIMESTAMP | string    |  -                                                                                                                     |
| TIME       | TIME      | string    |  -                                                                                                                     |
| YEAR       | YEAR      | int       |  -                                                                                                                     |
| BIT        | BIT       | bytes     |  -                                                                                                                     |
| JSON       | JSON      | string    |  -                                                                                                                     |
| ENUM       | ENUM      | string    |  -                                                                                                                     |
| SET        | SET       | string    |  -                                                                                                                     |
| DECIMAL    | DECIMAL   | バイト     | `avro-decimal-handling-mode` が string の場合、AVRO_TYPE は string です。                                                         |

Avroプロトコルでは、他にも2つの `sink-uri` パラメータがカラムデータの形式に影響を与える可能性があります: `avro-decimal-handling-mode` と `avro-bigint-unsigned-handling-mode`。

- `avro-decimal-handling-mode` は、Avroが10進数フィールドをどのように扱うかを制御します。

    - string: Avroは10進数フィールドを文字列として扱います。
    - precise: Avroは10進数フィールドをバイトとして扱います。

- `avro-bigint-unsigned-handling-mode` は、AvroがBIGINT UNSIGNEDフィールドをどのように扱うかを制御します。

    - string: AvroはBIGINT UNSIGNEDフィールドを文字列として扱います。
    - long: AvroはBIGINT UNSIGNEDフィールドを64ビットの符号付き整数として扱います。値が`9223372036854775807`を超えると、オーバーフローが発生します。

次のは構成の例です:

{{< copyable "shell-regular" >}}

```shell
cdc cli changefeed create --server=http://127.0.0.1:8300 --changefeed-id="kafka-avro-string-option" --sink-uri="kafka://127.0.0.1:9092/topic-name?protocol=avro&avro-decimal-handling-mode=string&avro-bigint-unsigned-handling-mode=string" --schema-registry=http://127.0.0.1:8081 --config changefeed_config.toml
```

```shell
[sink]
dispatchers = [
 {matcher = ['*.*'], topic = "tidb_{schema}_{table}"},
]
```

ほとんどのSQLタイプはベースのカラムデータの形式にマッピングされます。他のいくつかのSQLタイプは、追加情報を提供するためにベースデータ形式を拡張します。

BIT(64)

```
{
    "name":"{{ColumnName}}",
    "type":{
        "connect.parameters":{
            "tidb_type":"BIT",
            "length":"64"
        },
        "type":"bytes"
    }
}
```

ENUM/SET(a,b,c)

```
{
    "name":"{{ColumnName}}",
    "type":{
        "connect.parameters":{
            "tidb_type":"ENUM/SET",
            "allowed":"a,b,c"
        },
        "type":"string"
    }
}
```

DECIMAL(10, 4)

```
{
    "name":"{{ColumnName}}",
    "type":{
        "connect.parameters":{
            "tidb_type":"DECIMAL",
        },
        "logicalType":"decimal",
        "precision":10,
        "scale":4,
        "type":"bytes"
    }
}
```

## DDL イベントとスキーマ変更

AvroはDDLイベントを下流に生成しません。DMLイベントが発生するたびにスキーマの変更をチェックします。スキーマが変更されると、Avroは新しいスキーマを生成してSchema Registryに登録します。スキーマ変更が互換性チェックをパスしない場合、登録に失敗します。TiCDCはスキーマの互換性の問題を解決しません。

なお、スキーマの変更が互換性チェックをパスし新しいバージョンが登録されたとしても、データのプロデューサやコンシューマはシステムが正常に動作するようにするためにアップグレードする必要があります。

Confluent Schema Registryのデフォルトの互換性ポリシーが `BACKWARD` であり、ソーステーブルに空でない列を追加する場合を想定します。この場合、Avroは新しいスキーマを生成しますが、Schema Registryに互換性の問題により登録に失敗します。このとき、チェンジフィードはエラー状態に入ります。

スキーマに関する詳細については、[Schema Registry 関連ドキュメント](https://docs.confluent.io/platform/current/schema-registry/avro.html) を参照してください。

## トピック分散

Schema Registryは3種類の[Subject Name Strategies](https://docs.confluent.io/platform/current/schema-registry/serdes-develop/index.html#subject-name-strategy) をサポートしています: TopicNameStrategy、RecordNameStrategy、TopicRecordNameStrategy。現在、TiCDC AvroはTopicNameStrategyのみをサポートしており、つまりKafkaトピックは1つのデータ形式のみを受信できます。そのため、TiCDC Avroは同じトピックに複数のテーブルをマッピングすることを禁止します。チェンジフィードを作成する際、構成された分散ルールに `{schema}` と `{table}` のプレースホルダが含まれていない場合、エラーが報告されます。

## 互換性

TiCDCクラスタをv7.0.0にアップグレードする際、Avroを使用して複製されたテーブルに `FLOAT` データ型が含まれている場合、互換性ポリシーを `None` に手動で調整する必要があります。これにより、チェンジフィードがスキーマを正常に更新できます。それ以外の場合、アップグレード後にチェンジフィードはスキーマを更新できず、エラー状態に入ります。詳細については、[#8490](https://github.com/pingcap/tiflow/issues/8490) を参照してください。