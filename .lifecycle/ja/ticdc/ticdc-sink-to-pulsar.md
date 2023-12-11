---
title: Pulsarへのデータレプリケーション
summary: TiCDCを使用してPulsarにデータをレプリケートする方法について学びます。

# Pulsarへのデータレプリケーション

このドキュメントでは、TiCDCを使用して増分データをPulsarにレプリケートするチェンジフィードの作成方法について説明します。

## Pulsarへの増分データレプリケーションのためのレプリケーションタスクの作成

以下のコマンドを実行して、レプリケーションタスクを作成してください。

```shell
cdc cli changefeed create \
    --server=http://127.0.0.1:8300 \
    --sink-uri="pulsar://127.0.0.1:6650/persistent://public/default/yktest?protocol=canal-json" \
    --config=./t_changefeed.toml \
    --changefeed-id="simple-replication-task"
```

```shell

チェンジフィードの作成に成功しました！
ID: simple-replication-task
情報: {"upstream_id":7277814241002263370,"namespace":"default","id":"simple-replication-task","sink_uri":"pulsar://127.0.0.1:6650/consumer-test?protocol=canal-json","create_time":"2023-11-28T14:42:32.000904+08:00","start_ts":444203257406423044,"config":{"memory_quota":1073741824,"case_sensitive":false,"force_replicate":false,"ignore_ineligible_table":false,"check_gc_safe_point":true,"enable_sync_point":false,"bdr_mode":false,"sync_point_interval":600000000000,"sync_point_retention":86400000000000,"filter":{"rules":["pulsar_test.*"]},"mounter":{"worker_num":16},"sink":{"protocol":"canal-json","csv":{"delimiter":",","quote":"\"","null":"\\N","include_commit_ts":false,"binary_encoding_method":"base64"},"dispatchers":[{"matcher":["pulsar_test.*"],"partition":"","topic":"test_{schema}_{table}"}],"encoder_concurrency":16,"terminator":"\r\n","date_separator":"day","enable_partition_separator":true,"enable_kafka_sink_v2":false,"only_output_updated_columns":false,"delete_only_output_handle_key_columns":false,"pulsar_config":{"connection-timeout":30,"operation-timeout":30,"batching-max-messages":1000,"batching-max-publish-delay":10,"send-timeout":30},"advance_timeout":150},"consistent":{"level":"none","max_log_size":64,"flush_interval":2000,"use_file_backend":false},"scheduler":{"enable_table_across_nodes":false,"region_threshold":100000,"write_key_threshold":0},"integrity":{"integrity_check_level":"none","corruption_handle_level":"warn"}},"state":"normal","creator_version":"v7.5.0","resolved_ts":444203257406423044,"checkpoint_ts":444203257406423044,"checkpoint_time":"2023-09-12 14:42:31.410"}
```

各パラメータの意味は以下のとおりです：

- `--server`: TiCDCクラスタ内のTiCDCサーバーのアドレス。
- `--changefeed-id`: レプリケーションタスクのID。フォーマットは正規表現 `^[a-zA-Z0-9]+(\-[a-zA-Z0-9]+)*$` に一致する必要があります。IDが指定されていない場合、TiCDCは自動的にUUID（バージョン4形式）をIDとして生成します。
- `--sink-uri`: レプリケーションタスクのダウンストリームアドレス。[Sink URIを使用してPulsarを構成する](#sink-uri)を参照してください。
- `--start-ts`: チェンジフィードの開始TSO。TiCDCクラスターはこのTSOからデータを取得し始めます。デフォルト値は現在時刻です。
- `--target-ts`: チェンジフィードのターゲットTSO。TiCDCクラスターはこのTSOでデータの取得を停止します。デフォルトでは空であり、TiCDCは自動的にデータの取得を停止しません。
- `--config`: チェンジフィードの構成ファイル。[TiCDCチェンジフィード構成パラメータ](/ticdc/ticdc-changefeed-config.md)を参照してください。

## Sink URIおよびチェンジフィードの構成を使用してPulsarを構成する

Sink URIを使用してTiCDCのターゲットシステムの接続情報を指定し、チェンジフィード構成を使用してPulsarに関連するパラメータを構成できます。

### Sink URI

Sink URIは以下の形式に従います：

```shell
[scheme]://[userinfo@][host]:[port][/path]?[query_parameters]
```

構成例1：

```shell
--sink-uri="pulsar://127.0.0.1:6650/persistent://abc/def/yktest?protocol=canal-json"
```

構成例2：

```shell
--sink-uri="pulsar://127.0.0.1:6650/yktest?protocol=canal-json"
```

URIで構成可能なパラメータは以下のとおりです：

| パラメータ           | 説明                                                   |
| :------------------ | :------------------------------------------------------------ |
| `127.0.0.1`          | ダウンストリームPulsarがサービスを提供するIPアドレス。             |
| `6650`               | ダウンストリームPulsarの接続ポート。                              |
| `persistent://abc/def/yktest`   |  上記の構成例1に示されているように、このパラメータはPulsarのテナント、名前空間、およびトピックを指定するために使用されます。   |
| `yktest`    | 上記の構成例2に示されているように、指定するトピックがPulsarのデフォルトテナント`public`のデフォルト名前空間`default`にある場合、URIをトピック名のみで構成できます。たとえば、`yktest`とします。これは`persistent://public/default/yktest`とトピックを指定することと等価です。 |

### チェンジフィードの構成パラメータ

以下はチェンジフィードの構成パラメータの例です：

```toml
[sink]
# `dispatchers`は一致ルールを指定するために使用されます。
# 注意：ダウンストリームMQがPulsarの場合、`partition`のルーティングルールが`ts`、`index-value`、`table`、または`default`のいずれでも指定されていない場合、各Pulsarメッセージは、キーとして設定した文字列を使用してルーティングされます。
# たとえば、マッチャーのルーティングルールを文字列`code`と指定すると、そのマッチャーに一致するすべてのPulsarメッセージは、キーとして`code`を使用してルーティングされます。
# dispatchers = [
#    {matcher = ['test1.*', 'test2.*'], topic = "トピック表現1", partition = "ts" },
#    {matcher = ['test3.*', 'test4.*'], topic = "トピック表現2", partition = "index-value" },
#    {matcher = ['test1.*', 'test5.*'], topic = "トピック表現3", partition = "table"},
#    {matcher = ['test6.*'], partition = "default"},
#    {matcher = ['test7.*'], partition = "test123"}
# ]

# `protocol`はメッセージのエンコード形式を指定するために使用されます。
# ダウンストリームがPulsarの場合、プロトコルはcanal-jsonのみです。
# protocol = "canal-json"

# 次のパラメータは、ダウンストリームがPulsarの場合にのみ効果を発揮します。
[sink.pulsar-config]
# Pulsarサーバーでの認証はトークンを使用して行います。トークンの値を指定してください。
authentication-token = "xxxxxxxxxxxxx"
# Pulsarサーバー認証にトークンを使用する場合、トークンの場所を指定してください。
token-from-file="/data/pulsar/token-file.txt"
# Pulsarは基本アカウントとパスワードを使用してアイデンティティを認証します。アカウントを指定してください。
basic-user-name="root"
# Pulsarは基本アカウントとパスワードを使用してアイデンティティを認証します。パスワードを指定してください。
basic-password="password"
# Pulsar TLS暗号化認証の証明書パス。
auth-tls-certificate-path="/data/pulsar/certificate"
# Pulsar TLS暗号化認証の秘密キーパス。
auth-tls-private-key-path="/data/pulsar/certificate.key"
# Pulsar TLS暗号化認証の信頼される証明書ファイルのパス。
tls-trust-certs-file-path="/data/pulsar/tls-trust-certs-file"
# Pulsar oauth2発行者URL。詳細については、Pulsarのウェブサイトを参照してください：https://pulsar.apache.org/docs/2.10.x/client-libraries-go/#tls-encryption-and-authentication
oauth2.oauth2-issuer-url="https://xxxx.auth0.com"
# Pulsar oauth2オーディエンス
oauth2.oauth2-audience="https://xxxx.auth0.com/api/v2/"
# Pulsar oauth2秘密キー
oauth2.oauth2-private-key="/data/pulsar/privateKey"
# Pulsar oauth2クライアントID
oauth2.oauth2-client-id="0Xx...Yyxeny"
# Pulsar oauth2 oauth2-scope
oauth2.oauth2-scope="xxxx"
# TiCDCでキャッシュされるPulsarプロデューサーの数。デフォルト値は10240です。各Pulsarプロデューサーは1つのトピックに対応します。必要なトピックの数がデフォルト値よりも大きい場合は、数を増やす必要があります。
pulsar-producer-cache-size=10240
# Pulsarデータの圧縮方法。デフォルトでは圧縮は使用されません。オプションの値は "lz4", "zlib", "zstd" です。
compression-type=""
# PulsarクライアントがサーバーとのTCP接続を確立するタイムアウト。デフォルトで値は5秒です。
connection-timeout=5
# Pulsarクライアントがトピックの作成や購読などの操作を開始するタイムアウト。デフォルトで値は30秒です。
operation-timeout=30
# Pulsarプロデューサーが送信する単一のバッチ内のメッセージの最大数。デフォルトで値は1000です。
batching-max-messages=1000
# Pulsarプロデューサーのメッセージをバッチ処理する間隔。デフォルトで値は10ミリ秒です。
batching-max-publish-delay=10
# Pulsarプロデューサーがメッセージを送信するタイムアウト。デフォルトで値は30秒です。
send-timeout=30

### ベストプラクティス

* changefeedを作成する際に`protocol`パラメータを指定する必要があります。現在、Pulsarへのデータレプリケーションをサポートするのは`canal-json`プロトコルのみです。
* `pulsar-producer-cache-size`パラメータは、Pulsarクライアントにキャッシュされるプロデューサーの数を示します。Pulsarの各プロデューサーは1つのトピックにしか対応できないため、TiCDCはLRUメソッドを採用してプロデューサーをキャッシュし、デフォルトの制限値は10240です。デフォルト値よりも多くのトピックをレプリケートする必要がある場合は、数を増やす必要があります。

### Pulsar用のTiCDC認証と認可

次のサンプル設定は、Pulsarでトークン認証を使用する場合の構成です。

- トークン

    シンクURI: 

    ```shell
    --sink-uri="pulsar://127.0.0.1:6650/persistent://public/default/yktest?protocol=canal-json"
    ```

    構成パラメータ: 

    ```shell
    [sink.pulsar-config]
    authentication-token = "xxxxxxxxxxxxx"
    ```

- ファイルからのトークン

    シンクURI: 

    ```shell
    --sink-uri="pulsar://127.0.0.1:6650/persistent://public/default/yktest?protocol=canal-json"
    ```

    構成パラメータ: 

    ```toml
    [sink.pulsar-config]
    # PulsarはPulsarサーバーでの認証にトークンを使用します。TiCDCサーバーから読み込まれるトークンファイルへのパスを指定します。
    token-from-file="/data/pulsar/token-file.txt"
    ```

- TLS暗号化認証

    シンクURI: 

    ```shell
    --sink-uri="pulsar+ssl://127.0.0.1:6650/persistent://public/default/yktest?protocol=canal-json"
    ```

    構成パラメータ: 

    ```toml
    [sink.pulsar-config]
    # Pulsar TLS暗号化認証の証明書パス
    auth-tls-certificate-path="/data/pulsar/certificate"
    # Pulsar TLS暗号化認証の秘密キーパス
    auth-tls-private-key-path="/data/pulsar/certificate.key"
    # Pulsar TLS暗号化認証の信頼される証明書ファイルへのパス
    tls-trust-certs-file-path="/data/pulsar/tls-trust-certs-file"
    ```

- OAuth2認証

    シンクURI: 

    ```shell
    --sink-uri="pulsar+ssl://127.0.0.1:6650/persistent://public/default/yktest?protocol=canal-json"
    ```

    構成パラメータ: 

    ```toml
    [sink.pulsar-config]
    # Pulsar oauth2 issuer-url。詳細は、PulsarのWebサイトを参照してください：https://pulsar.apache.org/docs/2.10.x/client-libraries-go/#oauth2-authentication
    oauth2.oauth2-issuer-url="https://xxxx.auth0.com"
    # Pulsar oauth2オーディエンス
    oauth2.oauth2-audience="https://xxxx.auth0.com/api/v2/"
    # Pulsar oauth2プライベートキー
    oauth2.oauth2-private-key="/data/pulsar/privateKey"
    # Pulsar oauth2クライアントID
    oauth2.oauth2-client-id="0Xx...Yyxeny"
    # Pulsar oauth2 oauth2スコープ
    oauth2.oauth2-scope="xxxx"
    ```

## Pulsarシンクでのトピックとパーティションのディスパッチルールのカスタマイズ

### Matcher用の一致ルール

次のサンプル設定ファイルの`dispatchers`構成項目を使用する場合を示します:

```toml
[sink]
dispatchers = [
  {matcher = ['test1.*', 'test2.*'], topic = "Topic expression 1", partition = "ts" },
  {matcher = ['test3.*', 'test4.*'], topic = "Topic expression 2", partition = "index-value" },
  {matcher = ['test1.*', 'test5.*'], topic = "Topic expression 3", partition = "table"},
  {matcher = ['test6.*'], partition = "default"},
  {matcher = ['test7.*'], partition = "test123"}
]
```

- 一致ルールに一致するテーブルは、対応するトピック表現で指定されたポリシーに従ってディスパッチされます。例えば、テーブル`test3.aa`は`Topic expression 2`に従ってディスパッチされ、テーブル`test5.aa`は`Topic expression 3`に従ってディスパッチされます。
- 複数の一致ルールに一致するテーブルは、最初に一致するトピック表現に従ってディスパッチされます。例えば、テーブル`test1.aa`は`Topic expression 1`に従ってディスパッチされます。
- いずれの一致ルールにも一致しないテーブルは、`-sink-uri`で指定されたデフォルトトピックに対応するデータ変更イベントが送信されます。例えば、テーブル`test10.aa`はデフォルトトピックに送信されます。
- 一致ルールに一致するテーブルでトピックディスパッチャが指定されていない場合、`-sink-uri`で指定されたデフォルトトピックに対応するデータ変更が送信されます。例えば、テーブル`test6.abc`はデフォルトトピックに送信されます。

### トピックディスパッチャ

`topic = "xxx"`を使用してトピックディスパッチャを指定し、トピック表現を使用して柔軟なトピックディスパッチポリシーを実装できます。トータルトピック数は1000未満と推奨されています。

トピック表現の形式は`[prefix]{schema}[middle][{table}][suffix]`です。以下にそれぞれのパーツの意味を示します:

- `prefix`: オプション。トピック名のプレフィックスを表します。
- `{schema}`: オプション。データベース名を表します。
- `middle`: オプション。データベース名とテーブル名のセパレータを表します。
- `{table}`: オプション。テーブル名を表します。
- `suffix`: オプション。トピック名のサフィックスを表します。

`prefix`、`middle`、`suffix`は大文字と小文字のアルファベット（`a-z`、`A-Z`）、数字（`0-9`）、ピリオド（`.`）、アンダースコア（`_`）、ハイフン（`-`）のみをサポートします。`{schema}`と`{table}`は小文字である必要があります。大文字を含む`{Schema}`や`{TABLE}`などのプレースホルダは無効です。

以下にいくつかの例を示します:

- `matcher = ['test1.table1', 'test2.table2'], topic = "hello_{schema}_{table}"`
    - テーブル`test1.table1`に対応するデータ変更イベントは`hello_test1_table1`というトピックにディスパッチされます。
    - テーブル`test2.table2`に対応するデータ変更イベントは`hello_test2_table2`というトピックにディスパッチされます。

- `matcher = ['test3.*', 'test4.*'], topic = "hello_{schema}_world"`
    - `test3`以下のすべてのテーブルのデータ変更イベントは`hello_test3_world`というトピックにディスパッチされます。
    - `test4`以下のすべてのテーブルのデータ変更イベントは`hello_test4_world`というトピックにディスパッチされます。

- `matcher = ['*.*'], topic = "{schema}_{table}"`
    - TiCDCがリスンしているすべてのテーブルは、`databaseName_tableName`のルールに従って個別のトピックにディスパッチされます。例えば、`test.account`の場合、そのデータ変更ログは`test_account`というトピックにディスパッチされます。

### DDLイベントのディスパッチ

#### データベースレベルのDDLイベント

`CREATE DATABASE`や`DROP DATABASE`など、特定のテーブルに関連しないDDLステートメントは、データベースレベルのDDLステートメントと呼ばれます。データベースレベルのDDLステートメントに対応するイベントは、`--sink-uri`で指定されたデフォルトトピックに送信されます。

#### テーブルレベルのDDLイベント

`ALTER TABLE`や`CREATE TABLE`など、特定のテーブルに関連するDDLステートメントは、テーブルレベルのDDLステートメントと呼ばれます。テーブルレベルのDDLステートメントに対応するイベントは、`dispatchers`の設定に従って適切なトピックに送信されます。

例えば、`dispatchers`構成が`matcher = ['test.*'], topic = {schema}_{table}`のような場合、DDLイベントは以下のようにディスパッチされます:

- DDLイベントが単一のテーブルに関連する場合、DDLイベントはそのまま適切なトピックに送信されます。例えば、`DROP TABLE test.table1`のDDLイベントは`test_table1`というトピックに送信されます。
- もしDDLイベントに複数のテーブルが関与している場合（`RENAME TABLE`、`DROP TABLE`、`DROP VIEW`のいずれかは複数のテーブルが関与する可能性があります）、単一のDDLイベントは複数のイベントに分割され、適切なトピックに送信されます。たとえば、DDLイベント`RENAME TABLE test.table1 TO test.table10, test.table2 TO test.table20`の処理は次のようになります：

    - `RENAME TABLE test.table1 TO test.table10`のDDLイベントは`test_table1`という名前のトピックに送信されます。
    - `RENAME TABLE test.table2 TO test.table20`のDDLイベントは`test_table2`という名前のトピックに送信されます。

### パーティションディスパッチャー

現在、TiCDCは専用サブスクリプションモデルを使用してメッセージを消費するコンシューマのみをサポートしています。つまり、各コンシューマはトピック内のすべてのパーティションからメッセージを消費できます。

`partition = "xxx"`を使用してパーティションディスパッチャーを指定できます。次のパーティションディスパッチャーがサポートされています：`default`、`ts`、`index-value`、`table`。他の文字列を記入すると、TiCDCはその文字列をPulsarサーバーに送信されるメッセージのキーとして渡します。

ディスパッチのルールは以下の通りです：

- `default`：デフォルトでは、イベントはスキーマ名とテーブル名によってディスパッチされます（`table`を指定した場合と同じ）。
- `ts`：行の変更の`commitTs`を使用してハッシュ計算を行い、イベントをディスパッチします。
- `index-value`：テーブルの主キーまたは一意のインデックスの値を使用してハッシュ計算を行い、イベントをディスパッチします。
- `table`：スキーマ名とテーブル名を使用してハッシュ計算を行い、イベントをディスパッチします。
- その他の自己定義文字列：自己定義文字列は、Pulsarメッセージのキーとして直接使用され、Pulsarプロデューサーはこのキー値を使用してディスパッチします。