---
title: バイナリログコンシュマークライアントユーザーガイド
summary: バイナリログコンシュマークライアントを使用して、KafkaからTiDBのセカンダリバイナリログデータをコンシュームし、データを特定の形式で出力します。
aliases: ['/tidb/dev/binlog-slave-client','/docs/dev/tidb-binlog/binlog-slave-client/','/docs/dev/reference/tidb-binlog/binlog-slave-client/','/docs/dev/reference/tools/tidb-binlog/binlog-slave-client/']
---

# バイナリログコンシュマークライアントユーザーガイド

バイナリログコンシュマークライアントは、TiDBのセカンダリバイナリログデータをKafkaからコンシュームし、データを特定の形式で出力するために使用されます。現在、DrainerはMySQL、TiDB、ファイル、およびKafkaなど、複数のダウンストリーミングをサポートしています。ただし、ユーザーは時々、データを他の形式に出力するためのカスタマイズされた要件を持っています。例えば、ElasticsearchやHiveなどの形式にデータを出力するために、この機能が導入されました。

## Drainerの設定

Drainerの設定ファイルを変更し、データをKafkaに出力するように設定します：

```
[syncer]
db-type = "kafka"

[syncer.to]
# Kafkaのアドレス
kafka-addrs = "127.0.0.1:9092"
# Kafkaのバージョン
kafka-version = "2.4.0"
```

## カスタマイズ開発

### データ形式

まず、DrainerによってKafkaに出力されるデータの形式情報を取得する必要があります：

```
// `Column`はデータ型に基づいて対応する変数に列データを格納します。
message Column {
  // データがnullであるかどうかを示します
  optional bool is_null = 1 [ default = false ];
  // `int`データを格納します
  optional int64 int64_value = 2;
  // `uint`、`enum`、`set`データを格納します
  optional uint64 uint64_value = 3;
  // `float`および`double`データを格納します
  optional double double_value = 4;
  // `bit`、`blob`、`binary`および`json`データを格納します
  optional bytes bytes_value = 5;
  // `date`、`time`、`decimal`、`text`、`char`データを格納します
  optional string string_value = 6;
}

// `ColumnInfo`は列の情報を格納し、列名、タイプ、および主キーであるかどうかを含みます。
message ColumnInfo {
  optional string name = 1 [ (gogoproto.nullable) = false ];
  // MySQLの小文字の列フィールドタイプ
  // https://dev.mysql.com/doc/refman/8.0/en/data-types.html
  // `numeric`タイプの場合: int bigint smallint tinyint float double decimal bit
  // `string`タイプの場合: text longtext mediumtext char tinytext varchar
  // blob longblob mediumblob binary tinyblob varbinary
  // enum set
  // `json`タイプの場合: json
  optional string mysql_type = 2 [ (gogoproto.nullable) = false ];
  optional bool is_primary_key = 3 [ (gogoproto.nullable) = false ];
}

// `Row`は行の実際のデータを格納します。
message Row { repeated Column columns = 1; }

// `MutationType`はDMLタイプを示します。
enum MutationType {
  Insert = 0;
  Update = 1;
  Delete = 2;
}

// `Table`はテーブル内の変更を含みます。
message Table {
  optional string schema_name = 1;
  optional string table_name = 2;
  repeated ColumnInfo column_info = 3;
  repeated TableMutation mutations = 4;
}

// `TableMutation`は行の変更を格納します。
message TableMutation {
  required MutationType type = 1;
  // 変更後のデータ
  required Row row = 2;
  // 変更前のデータ。`Update` MutationTypeの場合にのみ有効です。
  optional Row change_row = 3;
}

// `DMLData`はトランザクション内でDMLによって引き起こされたすべての変更を格納します。
message DMLData {
  // `tables`にはトランザクション内のすべてのテーブル変更が含まれます。
  repeated Table tables = 1;
}

// `DDLData`はDDL情報を格納します。
message DDLData {
  // 現在使用されているデータベース
  optional string schema_name = 1;
  // 関連テーブル
  optional string table_name = 2;
  // `ddl_query`は元のDDLステートメントクエリです。
  optional bytes ddl_query = 3;
}

// `BinlogType`はDMLおよびDDLを含むバイナリログの種類を示します。
enum BinlogType {
  DML = 0; // `dml_data`を持っています
  DDL = 1; // `ddl_query`を持っています
}

// `Binlog`はトランザクション内のすべての変更を格納します。Kafkaは、構造データのシリアル化結果を格納します。
message Binlog {
  optional BinlogType type = 1 [ (gogoproto.nullable) = false ];
  optional int64 commit_ts = 2 [ (gogoproto.nullable) = false ];
  optional DMLData dml_data = 3;
  optional DDLData ddl_data = 4;
}
```

データ形式の定義については、[`secondary_binlog.proto`](https://github.com/pingcap/tidb/blob/master/pkg/tidb-binlog/proto/proto/secondary_binlog.proto)を参照してください。

### ドライバー

[タイDB-ツール](https://github.com/pingcap/tidb-tools/)プロジェクトは、[ドライバー](https://github.com/pingcap/tidb/tree/master/pkg/tidb-binlog/driver)を提供し、Kafka内のバイナリログデータを読み取るために使用されます。以下の機能があります：

* Kafkaデータを読み取る
* `commit ts`に基づいてKafkaに格納されたバイナリログを検出する

ドライバーを使用する際には、以下の情報を構成する必要があります：

* `KafkaAddr`：Kafkaクラスターのアドレス
* `CommitTS`：バイナリログの読み取りを開始する`commit ts`
* `Offset`：データの読み取りを開始するKafka`offset`。`CommitTS`が設定されている場合、このパラメータを構成する必要はありません。
* `ClusterID`：TiDBクラスターのクラスターID
* `Topic`：Kafkaのトピック名。Topicが空の場合、Drainerでデフォルトの名前`<ClusterID>_obinlog`が使用されます。

パッケージ内のドライバーコードを引用し、ドライバーが提供する例のコードを参照して、ドライバーの使用方法やバイナリログデータの解析方法を学んでください。

現在、以下の2つの例が提供されています：

* MySQLにデータをレプリケートするためのドライバーの使用。この例は、バイナリログをSQLに変換する方法を示しています。
* データをプリントするためのドライバーの使用

> **注意:**
>
> - 例のコードはドライバーの使用方法のみを示しています。本番環境でドライバーを使用する場合は、コードを最適化する必要があります。
> - 現在、Golangバージョンのドライバーおよび例のコードのみが利用可能です。他の言語を使用する場合は、バイナリログprotoファイルに基づいて対応する言語のコードファイルを生成し、Kafka内のバイナリログデータを読み取り、データを解析し、データをダウンストリームに出力するアプリケーションを開発する必要があります。また、他の言語の例のコードを最適化し、[TiDB-Tools](https://github.com/pingcap/tidb-tools)に提出することも歓迎します。