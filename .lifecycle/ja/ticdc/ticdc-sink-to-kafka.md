---
title: Kafkaへデータを複製する
summary: TiCDCを使用してApache Kafkaへデータを複製する方法について学びます。

# Kafkaへデータを複製する

このドキュメントでは、TiCDCを使用してインクリメンタルデータをApache Kafkaに複製する変更フィードを作成する方法について説明します。

## 複製タスクを作成する

次のコマンドを実行して複製タスクを作成します。

```shell
cdc cli changefeed create \
    --server=http://10.0.10.25:8300 \
    --sink-uri="kafka://127.0.0.1:9092/topic-name?protocol=canal-json&kafka-version=2.4.0&partition-num=6&max-message-bytes=67108864&replication-factor=1" \
    --changefeed-id="simple-replication-task"
```

```shell
変更フィードを作成しました！
ID: simple-replication-task
Info: {"sink-uri":"kafka://127.0.0.1:9092/topic-name?protocol=canal-json&kafka-version=2.4.0&partition-num=6&max-message-bytes=67108864&replication-factor=1","opts":{},"create-time":"2023-11-28T22:04:08.103600025+08:00","start-ts":415241823337054209,"target-ts":0,"admin-job-type":0,"sort-engine":"unified","sort-dir":".","config":{"case-sensitive":false,"filter":{"rules":["*.*"],"ignore-txn-start-ts":null,"ddl-allow-list":null},"mounter":{"worker-num":16},"sink":{"dispatchers":null},"scheduler":{"type":"table-number","polling-time":-1}},"state":"normal","history":null,"error":null}
```

- `--server`: TiCDCクラスタ内の任意のTiCDCサーバーのアドレス。
- `--changefeed-id`: 複製タスクのID。フォーマットは`^[a-zA-Z0-9]+(\-[a-zA-Z0-9]+)*$`の正規表現と一致する必要があります。このIDが指定されていない場合、TiCDCは自動的にUUID（バージョン4形式）をIDとして生成します。
- `--sink-uri`: 複製タスクのダウンストリームアドレス。詳細については、[kafkaの`sink-uri`を設定する](#kafkaのためのsink-uriを設定する)を参照してください。
- `--start-ts`: 変更フィードの開始TSOを指定します。このTSOからTiCDCクラスタはデータの取得を開始します。デフォルト値は現在時刻です。
- `--target-ts`: 変更フィードの終了TSOを指定します。このTSOまでTiCDCクラスタはデータの取得を停止します。デフォルト値は空で、TiCDCは自動的にデータの取得を停止しません。
- `--config`: 変更フィードの構成ファイルを指定します。詳細については、[TiCDC変更フィード構成パラメータ](/ticdc/ticdc-changefeed-config.md)を参照してください。

## Kafkaのためのsink-uriを設定する

Sink URIはTiCDCターゲットシステムの接続情報を指定するために使用されます。フォーマットは次のようになります。

```shell
[scheme]://[userinfo@][host]:[port][/path]?[query_parameters]
```

サンプル構成:

```shell
--sink-uri="kafka://127.0.0.1:9092/topic-name?protocol=canal-json&kafka-version=2.4.0&partition-num=6&max-message-bytes=67108864&replication-factor=1"
```

以下は、Kafka用に構成可能なsink URIパラメータと値の説明です:

| パラメータ/パラメータ値               | 説明                                                        |
| :------------------ | :------------------------------------------------------------ |
| `127.0.0.1`          | ダウンストリームKafkaサービスのIPアドレス。                                 |
| `9092`               | ダウンストリームKafkaのポート番号。                                          |
| `topic-name` | 可変。Kafkaトピックの名称。 |
| `kafka-version`      | ダウンストリームKafkaのバージョン（オプション、デフォルトでは`2.4.0`）。現在、最初にサポートされるKafkaバージョンは`0.11.0.2`で、最新バージョンは`3.2.0`です。この値はダウンストリームKafkaの実際のバージョンと一致している必要があります。                      |
| `kafka-client-id`    | 複製タスクのKafkaクライアントIDを指定します（オプション。デフォルトでは`TiCDC_sarama_producer_replication ID`）。 |
| `partition-num`      | ダウンストリームKafkaのパーティション数（オプション。この値は実際のパーティションの数より大きくてはなりません。それ以外の場合、複製タスクは正常に作成できません。デフォルトでは`3`）。 |
| `max-message-bytes`  | 一度にKafkaブローカーに送信されるデータの最大サイズ（オプション、デフォルトでは`10MB`）。 v5.0.6およびv4.0.6から、デフォルト値は`64MB`および`256MB`から`10MB`に変更されています。 |
| `replication-factor` | 保存可能なKafkaメッセージレプリカの数（オプション、デフォルトでは`1`）。この値はKafkaの[`min.insync.replicas`](https://kafka.apache.org/33/documentation.html#brokerconfigs_min.insync.replicas)の値以上である必要があります。 |
| `required-acks` | `Produce`リクエストで使用されるパラメータで、ブローカーに応答する前に受け取るレプリカの数を通知します。値のオプションは `0`（`NoResponse`：応答なし、`TCP ACK`のみが提供される）、`1`（`WaitForLocal`：ローカルコミットが正常に送信された後に応答します）、`-1`（`WaitForAll`：全てのレプリカが正常にコミットされた後に応答します。ブローカーの[`min.insync.replicas`](https://kafka.apache.org/33/documentation.html#brokerconfigs_min.insync.replicas)構成項目で最小数のレプリカを構成できます）（オプション、デフォルト値は`-1`）。    |
| `compression` | メッセージの送信時に使用される圧縮アルゴリズム（値のオプションは `none`、`lz4`、`gzip`、`snappy`、`zstd`；デフォルトは`none`）。Snappy圧縮ファイルは[公式Snappy形式](https://github.com/google/snappy)である必要があります。他のバリアントのSnappy圧縮はサポートされていません。|
| `protocol` | メッセージのKafkaへの出力に使用されるプロトコル。値のオプションは `canal-json`、`open-protocol`、`canal`、`avro`、`maxwell`です。   |
| `auto-create-topic` | `topic-name`がKafkaクラスタに存在しない場合に、TiCDCがトピックを自動的に作成するかどうかを決定します（オプション、デフォルトでは`true`）。 |
| `enable-tidb-extension` | オプション。出力プロトコルが`canal-json`の場合、`true`の場合、TiCDCは[WATERMARKイベント](/ticdc/ticdc-canal-json.md#watermark-event)を送信し、Kafkaメッセージに[TiDB拡張フィールド](/ticdc/ticdc-canal-json.md#tidb-extension-field)を追加します。 v6.1.0では、このパラメータは `avro`プロトコルにも適用されます。`true`の場合、TiCDCはKafkaメッセージに[3つのTiDB拡張フィールド](/ticdc/ticdc-avro-protocol.md#tidb-extension-fields)を追加します。 |
| `max-batch-size` | v4.0.9で新規追加されました。メッセージプロトコルが1つのKafkaメッセージに複数のデータ変更を出力できる場合、このパラメータは1つのKafkaメッセージにおけるデータ変更の最大数を指定します。現在、このパラメータはKafkaの`protocol`が`open-protocol`の場合のみ有効です（オプション、デフォルトでは`16`）。 |
| `enable-tls` | ダウンストリームKafkaインスタンスに接続する際にTLSを使用するかどうか（オプション、デフォルトでは`false`）。 |
| `ca` | ダウンストリームKafkaインスタンスに接続するために必要なCA証明書ファイルのパス（オプション）。  |
| `cert` | ダウンストリームKafkaインスタンスに接続するために必要な証明書ファイルのパス（オプション）。 |
| `key` | ダウンストリームKafkaインスタンスに接続するために必要な証明書キーファイルのパス（オプション）。 |
| `insecure-skip-verify` | ダウンストリームKafkaインスタンスに接続する際に証明書検証をスキップするかどうか（オプション、デフォルトでは`false`）。 |
| `sasl-user` | ダウンストリームKafkaインスタンスに接続するために必要なSASL/PLAINまたはSASL/SCRAM認証のidentity（authcid）（オプション）。 |
| `sasl-password` | ダウンストリームKafkaインスタンスに接続するために必要なSASL/PLAINまたはSASL/SCRAM認証のパスワード（オプション。特殊文字が含まれる場合はURLエンコードする必要があります）。 |
| `sasl-mechanism` | ダウンストリームKafkaインスタンスに接続するために必要なSASL認証の名前。値は`plain`、`scram-sha-256`、`scram-sha-512`、`gssapi`になります。 |
| `sasl-gssapi-auth-type` | gssapi認証タイプ。値は`user`または`keytab`になります（オプション）。 |
| `sasl-gssapi-keytab-path` | gssapiキータブパス（オプション）。|
| `sasl-gssapi-kerberos-config-path` | gssapiケルベロス構成パス（オプション）。 |
| `sasl-gssapi-service-name` | gssapiサービス名（オプション）。 |
| `sasl-gssapi-user` | GSSAPI認証のユーザー名（任意）。 |
| `sasl-gssapi-password` | GSSAPI認証のパスワード（任意）。特殊文字を含む場合、URLエンコードする必要があります。 |
| `sasl-gssapi-realm` | GSSAPIレルム名（任意）。 |
| `sasl-gssapi-disable-pafxfast` | GSSAPI PA-FX-FASTを無効にするかどうか（任意）。 |
| `dial-timeout` | ダウンストリームKafkaとの接続を確立するタイムアウト。デフォルト値は `10s` です。 |
| `read-timeout` | ダウンストリームKafkaからの応答を取得するタイムアウト。デフォルト値は `10s` です。 |
| `write-timeout` | ダウンストリームKafkaにリクエストを送信するタイムアウト。デフォルト値は `10s` です。 |
| `avro-decimal-handling-mode` | `avro`プロトコルでのみ有効です。AvroがDECIMALフィールドをどのように処理するかを決定します。値は `string` または `precise` にでき、DECIMALフィールドを文字列または精密な浮動小数点数にマッピングするかを示します。  |
| `avro-bigint-unsigned-handling-mode` | `avro`プロトコルでのみ有効です。AvroがBIGINT UNSIGNEDフィールドをどのように処理するかを決定します。値は `string` または `long` にでき、BIGINT UNSIGNEDフィールドを64ビット符号付き数または文字列にマッピングするかを示します。  |

### ベストプラクティス

* 独自のKafkaトピックを作成することが推奨されています。最低限、トピックがKafkaブローカーに送信できる各メッセージの最大データ量と、ダウンストリームKafkaパーティションの数を設定する必要があります。変更フィードを作成する場合、これら2つの設定はそれぞれ `max-message-bytes` と `partition-num` に対応しています。
* まだ存在しないトピックを持つ変更フィードを作成する場合、TiCDCは`partition-num`と`replication-factor`パラメータを使用してトピックを作成しようとします。これらのパラメータを明示的に指定することを推奨します。
* ほとんどの場合、`canal-json`プロトコルを使用することを推奨します。

> **注意:**
>
> `protocol` が `open-protocol` の場合、TiCDCは`max-message-bytes` を超えるメッセージを生成しないようにします。ただし、1つの変更が`max-message-bytes`を超える長さになる場合、サイレントな失敗を避けるために、TiCDCはこのメッセージを出力し、ログに警告を出力しようとします。

### TiCDCはKafkaの認証と認可を使用します

Kafka SASL認証を使用する場合の例は次のとおりです。

- SASL/PLAIN

  ```shell
  --sink-uri="kafka://127.0.0.1:9092/topic-name?kafka-version=2.4.0&sasl-user=alice-user&sasl-password=alice-secret&sasl-mechanism=plain"
  ```

- SASL/SCRAM

  SCRAM-SHA-256とSCRAM-SHA-512は、PLAINメソッドと類似しています。対応する認証メソッドとして`sasl-mechanism`を指定するだけです。

- SASL/GSSAPI

  SASL/GSSAPI `user` 認証:

  ```shell
  --sink-uri="kafka://127.0.0.1:9092/topic-name?kafka-version=2.4.0&sasl-mechanism=gssapi&sasl-gssapi-auth-type=user&sasl-gssapi-kerberos-config-path=/etc/krb5.conf&sasl-gssapi-service-name=kafka&sasl-gssapi-user=alice/for-kafka&sasl-gssapi-password=alice-secret&sasl-gssapi-realm=example.com"
  ```

  `sasl-gssapi-user`と`sasl-gssapi-realm`の値は、Kerberosで指定された[principal](https://web.mit.edu/kerberos/krb5-1.5/krb5-1.5.4/doc/krb5-user/What-is-a-Kerberos-Principal_003f.html)に関連しています。たとえば、principalが`alice/for-kafka@example.com`に設定されている場合、`sasl-gssapi-user`と`sasl-gssapi-realm`はそれぞれ`alice/for-kafka`と`example.com`に指定されます。

  SASL/GSSAPI `keytab` 認証:

  ```shell
  --sink-uri="kafka://127.0.0.1:9092/topic-name?kafka-version=2.4.0&sasl-mechanism=gssapi&sasl-gssapi-auth-type=keytab&sasl-gssapi-kerberos-config-path=/etc/krb5.conf&sasl-gssapi-service-name=kafka&sasl-gssapi-user=alice/for-kafka&sasl-gssapi-keytab-path=/var/lib/secret/alice.key&sasl-gssapi-realm=example.com"
  ```

  SASL/GSSAPI認証メソッドの詳細については、[GSSAPIの構成](https://docs.confluent.io/platform/current/kafka/authentication_sasl/authentication_sasl_gssapi.html)を参照してください。

- TLS/SSL暗号化

    KafkaブローカーがTLS/SSL暗号化を有効にしている場合、`--sink-uri`に`-enable-tls=true`パラメータを追加する必要があります。自己署名証明書を使用する場合、`--sink-uri`で`ca`、`cert`、`key`を指定する必要があります。

- ACL認可

    TiCDCが正常に機能するために必要な最小限のアクセス許可のセットは次のとおりです。

    - トピック[resource type](https://docs.confluent.io/platform/current/kafka/authorization.html#resources)に対する`Create`、`Write`、`Describe`権限。
    - クラスタリソースタイプに対する`DescribeConfigs`権限。

### TiCDCをKafka Connect（Confluentプラットフォーム）と統合する

Confluentが提供する[data connectors](https://docs.confluent.io/current/connect/managing/connectors.html)を使用して、リレーショナルデータベースまたは非リレーショナルデータベースにデータをストリーミングするには、`avro`プロトコルを使用し、`schema-registry`に対するURLを提供する必要があります。

サンプル構成:

```shell
--sink-uri="kafka://127.0.0.1:9092/topic-name?&protocol=avro&replication-factor=3" --schema-registry="http://127.0.0.1:8081" --config changefeed_config.toml
```

```shell
[sink]
dispatchers = [
 {matcher = ['*.*'], topic = "tidb_{schema}_{table}"},
]
```

詳しい統合ガイドは、[TiDBをConfluentプラットフォームと統合するためのクイックスタートガイド](/ticdc/integrate-confluent-using-ticdc.md)を参照してください。 

## Kafka SinkのTopicおよびPartitionディスパッチャーのルールをカスタマイズする

### Matcherルール

次の`dispatchers`の構成を例として取り上げます。

```toml
[sink]
dispatchers = [
  {matcher = ['test1.*', 'test2.*'], topic = "Topic expression 1", partition = "ts" },
  {matcher = ['test3.*', 'test4.*'], topic = "Topic expression 2", partition = "index-value" },
  {matcher = ['test1.*', 'test5.*'], topic = "Topic expression 3", partition = "table"},
  {matcher = ['test6.*'], partition = "ts"}
]
```

- マッチャールールに一致するテーブルは、対応するトピック式で指定されたポリシーに従ってディスパッチされます。たとえば、`test3.aa`テーブルは「Topic expression 2」に従ってディスパッチされます。`test5.aa`テーブルは「Topic expression 3」に従ってディスパッチされます。
- 複数のマッチャールールに一致するテーブルは、最初に一致したトピック式に従ってディスパッチされます。たとえば、`test1.aa`テーブルは「Topic expression 1」に従ってディスパッチされます。
- どのマッチャールールにも一致しないテーブルについては、`--sink-uri`で指定されたデフォルトトピックに対応するデータ変更イベントが送信されます。たとえば、`test10.aa`テーブルはデフォルトトピックに送信されます。
- マッチャールールに一致するがトピックディスパッチャーが指定されていないテーブルについては、`--sink-uri`で指定されたデフォルトトピックに対応するデータ変更が送信されます。たとえば、`test6.aa`テーブルはデフォルトトピックに送信されます。

### トピックディスパッチャー

`topic = "xxx"` を使用してトピックディスパッチャーを指定し、トピック式を使用して柔軟なトピックディスパッチポリシーを実装できます。トピックの総数は1000未満であることが推奨されます。

トピック式のフォーマットは、`[prefix][{schema}][middle][{table}][suffix]`です。

- `prefix`: オプション。トピック名のプレフィックスを示します。
- `[{schema}]`: オプション。スキーマ名に一致させるために使用されます。
- `middle`: オプション。スキーマ名とテーブル名の区切り記号を示します。
- `{table}`: オプション。テーブル名に一致させるために使用されます。
- `suffix`: オプション。トピック名のサフィックスを示します。
`prefix`, `middle`, および `suffix` には、次の文字のみを含めることができます: `a-z`、`A-Z`、`0-9`、`.`、`_` および `-`。`{schema}` および `{table}` はいずれも小文字である必要があります。`{Schema}` や `{TABLE}` のようなプレースホルダは無効です。

いくつかの例:

- `matcher = ['test1.table1', 'test2.table2'], topic = "hello_{schema}_{table}"`
    - `test1.table1` に対応するデータ変更イベントは `hello_test1_table1` というトピックに送信されます。
    - `test2.table2` に対応するデータ変更イベントは `hello_test2_table2` というトピックに送信されます。
- `matcher = ['test3.*', 'test4.*'], topic = "hello_{schema}_world"`
    - `test3` 内のすべてのテーブルに対応するデータ変更イベントは `hello_test3_world` というトピックに送信されます。
    - `test4` 内のすべてのテーブルに対応するデータ変更イベントは `hello_test4_world` というトピックに送信されます。
- `matcher = ['test5.*', 'test6.*'], topic = "hard_code_topic_name"`
    - `test5` および `test6` 内のすべてのテーブルに対応するデータ変更イベントは `hard_code_topic_name` というトピックに送信されます。トピック名を直接指定できます。
- `matcher = ['*.*'], topic = "{schema}_{table}"`
    - TiCDC によってリッスンされるすべてのテーブルは、"schema_table" のルールに従って別々のトピックにディスパッチされます。たとえば、`test.account` テーブルの場合、TiCDC はそのデータ変更ログを `test_account` という名前のトピックに送信します。

### DDL イベントをディスパッチする

#### スキーマレベルの DDL

特定のテーブルに関連しない DDL はスキーマレベルの DDL と呼ばれます。`create database` や `drop database` のようなイベントは、`--sink-uri` で指定されたデフォルトのトピックに送信されます。

#### テーブルレベルの DDL

特定のテーブルに関連する DDL はテーブルレベルの DDL と呼ばれます。`alter table` や `create table` のようなイベントは、ディスパッチャの構成に応じて対応するトピックに送信されます。

たとえば、`matcher = ['test.*'], topic = {schema}_{table}` のようなディスパッチャの例では、DDL イベントは次のようにディスパッチされます:

- 単一のテーブルが DDL イベントに関与する場合、DDL イベントはそのまま対応するトピックに送信されます。たとえば、`drop table test.table1` のような DDL イベントは `test_table1` という名前のトピックに送信されます。
- 複数のテーブルが DDL イベントに関与する場合 (`rename table` / `drop table` / `drop view` などは複数のテーブルに関与する可能性があります)、DDL イベントは複数のイベントに分割され、対応するトピックに送信されます。たとえば、`rename table test.table1 to test.table10, test.table2 to test.table20` のような DDL イベントは `rename table test.table1 to test.table10` が `test_table1` というトピックに送信され、`rename table test.table2 to test.table20` が `test.table2` というトピックに送信されます。

### パーティションディスパッチャ

`partition = "xxx"` を使用してパーティションディスパッチャを指定できます。`default`、`index-value`、`columns`、`table`、および `ts` の 5 つのディスパッチャをサポートしています。ディスパッチャのルールは次のとおりです:

- `default`: デフォルトで `table` ディスパッチャルールを使用します。スキーマ名とテーブル名を使用してパーティション番号を計算し、テーブルからのデータが同じパーティションに送信されるようにします。その結果、単一のテーブルからのデータは1つのパーティションのみに存在し、順序が保証されます。ただし、このディスパッチャルールは送信スループットを制限し、消費速度を向上させることはできません。
- `index-value`: プライマリキー、ユニークインデックス、または明示的に指定されたインデックスを使用してパーティション番号を計算し、テーブルデータを複数のパーティションに分散させます。単一のテーブルからのデータは複数のパーティションに送信され、各パーティション内のデータは順序が保証されます。消費速度を向上させるには、消費者を追加することができます。
- `columns`: 明示的に指定された列の値を使用してパーティション番号を計算し、テーブルデータを複数のパーティションに分散させます。単一のテーブルからのデータは複数のパーティションに送信され、各パーティション内のデータは順序が保証されます。消費速度を向上させるには、消費者を追加することができます。
- `table`: スキーマ名とテーブル名を使用してパーティション番号を計算します。
- `ts`: 行の変更の commitTs を使用してパーティション番号を計算し、テーブルデータを複数のパーティションに分散させます。単一のテーブルからのデータは複数のパーティションに送信され、各パーティション内のデータは順序が保証されます。消費速度を向上させるには、消費者を追加することができます。ただし、データアイテムの複数の変更が異なるパーティションに送信される可能性があり、異なる消費者の進捗状況が異なる可能性があるため、データの整合性に影響を与える可能性があります。そのため、消費者はデータを commitTs で並べ替える必要があります。

次のような `dispatchers` の構成を取り上げてみましょう:

```toml
[sink]
dispatchers = [
    {matcher = ['test.*'], partition = "index-value"},
    {matcher = ['test1.*'], partition = "index-value", index-name = "index1"},
    {matcher = ['test2.*'], partition = "columns", columns = ["id", "a"]},
    {matcher = ['test3.*'], partition = "table"},
]
```

- `test` データベースのテーブルは `index-value` ディスパッチャを使用し、プライマリキーまたはユニークインデックスの値を使用してパーティション番号を計算します。プライマリキーが存在する場合はプライマリキーを使用し、それ以外の場合は最も短いユニークインデックスを使用します。
- `test1` のテーブルは `index-value` ディスパッチャを使用し、`index1` という名前のインデックスのすべての列の値を使用してパーティション番号を計算します。指定されたインデックスが存在しない場合はエラーが報告されます。なお、`index-name` で指定されたインデックスはユニークインデックスである必要があります。
- `test2` のテーブルは `columns` ディスパッチャを使用し、`id` および `a` の列の値を使用してパーティション番号を計算します。いずれかの列が存在しない場合はエラーが報告されます。
- `test3` のテーブルは `table` ディスパッチャを使用します。
- `test4` のテーブルは上記のいずれのルールにも一致しないため、`default` ディスパッチャ（つまり `table` ディスパッチャ）を使用します。

テーブルが複数のディスパッチャルールに一致する場合、最初に一致したルールが優先されます。

> **注意:**
>
> v6.1.0 以降、構成の意味を明確にするために、パーティションディスパッチャを指定するための構成は `dispatcher` から `partition` に変更され、`partition` は `dispatcher` のエイリアスとなりました。たとえば、次の 2 つのルールは完全に同等です。
>
> ```
> [sink]
> dispatchers = [
>    {matcher = ['*.*'], dispatcher = "index-value"},
>    {matcher = ['*.*'], partition = "index-value"},
> ]
> ```
>
> ただし、`dispatcher` と `partition` は同じルール内で同時に現れることはできません。たとえば、次のルールは無効です。
>
> ```
> {matcher = ['*.*'], dispatcher = "index-value", partition = "table"},
> ```

## カラムセレクタ

カラムセレクタ機能は、イベントから列を選択し、選択された列に関連するデータ変更のみをダウンストリームに送信することをサポートしています。

次のような `column-selectors` の構成を取り上げてみましょう:

```toml
[sink]
column-selectors = [
    {matcher = ['test.t1'], columns = ['a', 'b']},
    {matcher = ['test.*'], columns = ["*", "!b"]},
    {matcher = ['test1.t1'], columns = ['column*', '!column1']},
    {matcher = ['test3.t'], columns = ["column?", "!column1"]},
]
```

- `test.t1` のテーブルでは、`a` および `b` の列のみが送信されます。
- `test` データベースのテーブル（`t1` を除く）では、`b` 以外のすべての列が送信されます。
- `test1.t1` のテーブルでは、`column1` を除く `column` で始まるすべての列が送信されます。
- `test3.t` のテーブルでは、`column1` を除く `column` で始まる 7 文字の列が送信されます。
- いずれのルールにも一致しないテーブルでは、すべての列が送信されます。

> **注意:**
>
> `column-selectors` ルールでフィルタリングされた後、テーブル内のデータは複製するためにプライマリキーまたはユニークキーを持っている必要があります。それ以外の場合、cf の作成時または実行中にエラーが報告されます。

## 単一の大規模なテーブルの負荷を複数の TiCDC ノードにスケールアウトする

この機能は、データのボリュームおよび毎分の変更行数に応じて、単一の大規模なテーブルのデータ複製範囲を複数の範囲に分割し、それぞれの範囲内のデータボリュームおよび変更行数のほぼ同じになるようにします。そして、これらの範囲を複数の TiCDC ノードに複製することで、単一の大規模なテーブルを複数の TiCDC ノードで同時に複製できるようにします。この機能は次の 2 つの問題を解決することができます:

- 単一の TiCDC ノードでは大規模な単一のテーブルをタイムリーに複製できません。
- TiCDC ノードが消費するリソース（CPU やメモリなど）が均等に分散されていない問題を解消できます。

> **警告:**
>
> TiCDC v7.0.0 では、大規模な単一のテーブルの負荷をスケールアウトする機能は Kafka changefeed のみをサポートしています。

構成の例:

```toml
[scheduler]
```toml
enable-table-across-nodes = true
region-threshold = 100000
write-key-threshold = 30000
```

指定されたSQL文で、テーブルが含むリージョンの数をクエリできます。

```sql
SELECT COUNT(*) FROM INFORMATION_SCHEMA.TIKV_REGION_STATUS WHERE DB_NAME="database1" AND TABLE_NAME="table1" AND IS_INDEX=0;
```

## Kafkaトピックのサイズ制限を超過するメッセージの処理

Kafkaトピックは受信できるメッセージのサイズに制限があります。この制限は[`max.message.bytes`](https://kafka.apache.org/documentation/#topicconfigs_max.message.bytes)パラメータによって制御されます。TiCDC Kafka Sinkがこの制限を超えるデータを送信すると、チェンジフィードはエラーを報告し、データのレプリケーションを続行できません。この問題を解決するために、TiCDCは新しい構成`large-message-handle-option`を追加し、以下の解決策を提供します。

現在、この機能では2つのエンコードプロトコルがサポートされています: Canal-JSONとOpen Protocol。Canal-JSONプロトコルを使用する場合は、`sink-uri`内で`enable-tidb-extension=true`を指定する必要があります。

### TiCDCデータの圧縮

v7.4.0から、TiCDC Kafka Sinkは、エンコード直後にデータを圧縮し、圧縮されたデータのサイズとメッセージサイズの制限を比較することをサポートしています。この機能により、メッセージのサイズ制限を超過する発生を効果的に低減できます。

以下に構成例を示します。

```toml
[sink.kafka-config.large-message-handle]
# この設定はv7.4.0で導入されました。
# デフォルトでは"none"で、圧縮機能は無効です。
# 可能な値は"none"、"lz4"、"snappy"です。デフォルト値は"none"です。
large-message-handle-compression = "none"
```

この機能は、Kafkaプロデューサーの圧縮機能とは異なります:
* `large-message-handle-compression`で指定された圧縮アルゴリズムは、単一のKafkaメッセージを圧縮します。圧縮は、メッセージサイズの制限と比較する前に行われます。
* `sink-uri`で圧縮アルゴリズムを構成できます。圧縮は、複数のKafkaメッセージを含むデータ送信要求全体に適用されます。圧縮は、メッセージサイズの制限と比較した後に行われます。

`large-message-handle-compression`が有効になっている場合、コンシューマが受信したメッセージは、特定の圧縮プロトコルを使用してエンコードされます。コンシューマアプリケーションは、データをデコードする際に、指定された圧縮プロトコルを使用する必要があります。

### ハンドルキーのみ送信

v7.3.0から、TiCDC Kafka Sinkは、メッセージのサイズが制限を超えた場合にハンドルキーのみを送信できるようサポートしています。これにより、メッセージのサイズが大幅に削減され、Kafkaトピックの制限を超えた場合の変更フィードのエラーやタスクの失敗を回避できます。ハンドルキーは以下のように定義されます:
* レプリケーションするテーブルに主キーがある場合、主キーがハンドルキーとなります。
* テーブルに主キーがないがNOT NULLのユニークキーがある場合、NOT NULLのユニークキーがハンドルキーとなります。

以下に構成例を示します。

```toml
[sink.kafka-config.large-message-handle]
# large-message-handle-optionはv7.3.0で導入されました。
# デフォルトは"none"です。メッセージのサイズが制限を超えた場合、変更フィードは失敗します。
# "handle-key-only"に設定すると、メッセージのサイズが制限を超えた場合、データフィールドにはハンドルキーのみが送信されます。メッセージのサイズが依然として制限を超えた場合、変更フィードは失敗します。
large-message-handle-option = "claim-check"
```

### ハンドルキーのみメッセージを処理

ハンドルキーのみのメッセージ形式は以下の通りです。

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
        "id": 4
    },
    "mysqlType": {
        "id": "int"
    },
    "data": [
        {
          "id": "2"
        }
    ],
    "old": null,
    "_tidb": {     // TiDBの拡張フィールド
        "commitTs": 163963314122145239,
        "onlyHandleKey": true
    }
}
```

Kafkaコンシューマがメッセージを受信すると、まず`onlyHandleKey`フィールドをチェックします。このフィールドが存在し、`true`である場合、メッセージは完全なデータのハンドルキーのみを含むことを意味します。この場合、完全なデータを取得するにはアップストリームのTiDBをクエリし、[`tidb_snapshot`を使用して履歴データを読み取る](/read-historical-data.md)必要があります。

> **警告:**
>
> Kafkaコンシューマがデータを処理し、TiDBをクエリする際、データはGCによって削除されている可能性があります。このような状況を避けるために、TiDBクラスタの[GCライフタイムを変更](/system-variables.md#tidb_gc_life_time-new-in-v50)する必要があります。

### 大きなメッセージを外部ストレージに送信

v7.4.0から、TiCDC Kafka Sinkは、メッセージのサイズが制限を超えた場合に大きなメッセージを外部ストレージに送信できるようサポートしています。同時に、TiCDCはKafkaに大きなメッセージのアドレスを含むメッセージを送信します。これにより、Kafkaトピックの制限を超えたことによる変更フィードの失敗を回避できます。

以下に構成例を示します。

```toml
[sink.kafka-config.large-message-handle]
# large-message-handle-optionはv7.3.0で導入されました。
# デフォルトは"none"です。メッセージのサイズが制限を超えた場合、変更フィードは失敗します。
# "handle-key-only"に設定すると、メッセージのサイズが制限を超えた場合、データフィールドにはハンドルキーのみが送信されます。メッセージのサイズが依然として制限を超えた場合、変更フィードは失敗します。
# "claim-check"に設定すると、メッセージのサイズが制限を超えた場合、メッセージは外部ストレージに送信されます。
large-message-handle-option = "claim-check"
claim-check-storage-uri = "s3://claim-check-bucket"
```

`large-message-handle-option`を`"claim-check"`に設定する場合、`claim-check-storage-uri`を有効な外部ストレージアドレスに設定する必要があります。そうでない場合、チェンジフィードの作成に失敗します。

> **ヒント:**
>
> TiCDCにおけるAmazon S3、GCS、Azure Blob StorageのURIパラメータの詳細については、[外部ストレージサービスのURI形式](/external-storage-uri.md)を参照してください。

TiCDCは外部ストレージサービス上のメッセージをクリーンアップしません。データコンシューマは、外部ストレージサービスを自身で管理する必要があります。

### 外部ストレージから大きなメッセージを消費する

Kafkaコンシューマは、外部ストレージに格納されている大きなメッセージのアドレスを含むメッセージを受信します。メッセージ形式は以下の通りです。

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
        "id": 4
    },
    "mysqlType": {
        "id": "int"
    },
    "data": [
        {
          "id": "2"
        }
    ],
    "old": null,
    "_tidb": {     // TiDBの拡張フィールド
        "commitTs": 163963314122145239,
        "claimCheckLocation": "s3:/claim-check-bucket/${uuid}.json"
    }
}
```

メッセージが`claimCheckLocation`フィールドを含む場合、Kafkaコンシューマは、フィールドによって提供されたアドレスに従ってJSON形式で格納されている大きなメッセージデータを読み込みます。メッセージ形式は以下の通りです。

```json
{
  key: "xxx",
  value: "xxx",
}
```

`key`と`value`フィールドには、エンコードされた大きなメッセージが含まれており、これらの2つのパートのデータを解析して大きなメッセージの内容を復元できます。