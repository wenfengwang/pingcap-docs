---
title: TiCDCチェンジフィードのCLIおよび構成パラメータ
summary: TiCDCチェンジフィードのCLIおよび構成パラメータの定義を学びます。
---

# TiCDCチェンジフィードのCLIおよび構成パラメータ

## チェンジフィードのCLIパラメータ

このセクションでは、リフィケーション（チェンジフィード）タスクを作成する方法を示すことで、TiCDCチェンジフィードのコマンドラインパラメータを紹介します。

```shell
cdc cli changefeed create --server=http://10.0.10.25:8300 --sink-uri="mysql://root:123456@127.0.0.1:3306/" --changefeed-id="simple-replication-task"
```

```shell
チェンジフィードの作成に成功しました！
ID: simple-replication-task
情報: {"upstream_id":7178706266519722477,"namespace":"default","id":"simple-replication-task","sink_uri":"mysql://root:xxxxx@127.0.0.1:4000/?time-zone=","create_time":"2023-11-28T15:05:46.679218+08:00","start_ts":438156275634929669,"engine":"unified","config":{"case_sensitive":false,"enable_old_value":true,"force_replicate":false,"ignore_ineligible_table":false,"check_gc_safe_point":true,"enable_sync_point":true,"bdr_mode":false,"sync_point_interval":30000000000,"sync_point_retention":3600000000000,"filter":{"rules":["test.*"],"event_filters":null},"mounter":{"worker_num":16},"sink":{"protocol":"","schema_registry":"","csv":{"delimiter":",","quote":"\"","null":"\\N","include_commit_ts":false},"column_selectors":null,"transaction_atomicity":"none","encoder_concurrency":16,"terminator":"\r\n","date_separator":"none","enable_partition_separator":false},"consistent":{"level":"none","max_log_size":64,"flush_interval":2000,"storage":""}},"state":"normal","creator_version":"v7.5.0"}
```

- `--changefeed-id`: レプリケーションタスクのID。書式は `^[a-zA-Z0-9]+(\-[a-zA-Z0-9]+)*$` 正規表現と一致する必要があります。このIDが指定されていない場合、TiCDCは自動的にIDとしてUUID（バージョン4フォーマット）を生成します。
- `--sink-uri`: レプリケーションタスクのダウンストリームアドレス。`--sink-uri` を以下の形式に従って構成します。現在、スキームは `mysql`、`tidb`、`kafka` をサポートしています。

    ```
    [scheme]://[userinfo@][host]:[port][/path]?[query_parameters]
    ```

    シンクURIに `! * ' ( ) ; : @ & = + $ , / ? % # [ ]` などの特殊文字が含まれる場合は、[URIエンコーダ](https://www.urlencoder.org/) などで特殊文字をエスケープする必要があります。

- `--start-ts`: チェンジフィードの開始TSOを指定します。このTSOからTiCDCクラスターはデータの取得を開始します。デフォルト値は現在の時刻です。
- `--target-ts`: チェンジフィードの終了TSOを指定します。TiCDCクラスターはこのTSOまでデータの取得を停止します。デフォルト値は空であり、TiCDCは自動的にデータの取得を停止しません。
- `--config`: チェンジフィードの構成ファイルを指定します。

## チェンジフィードの構成パラメータ

このセクションでは、レプリケーションタスクの構成を紹介します。
```
# v6.1.0から、TiDBは2種類のイベントディスパッチャーであるパーティションとトピックをサポートしています。詳細については、<partition and topic link>を参照してください。
# マッチャーの一致構文は、フィルタールールの構文と同じです。マッチャールールの詳細については、<>を参照してください。
# 注意: この設定項目はダウンストリームがMQの場合のみ有効です。
# 注意: ダウンストリームのMQがPulsarの場合、`partition`のルーティングルールが`ts`、`index-value`、`table`、または`default`のいずれでもない場合、各Pulsarメッセージは、キーとして設定した文字列を使用してルーティングされます。
# たとえば、マッチャーのルーティングルールを文字列 `code` とした場合、そのマッチャーに一致するすべての Pulsar メッセージは、キーとして `code` でルーティングされます。
# dispatchers = [
#    {matcher = ['test1.*', 'test2.*'], topic = "Topic expression 1", partition = "index-value"},
#    {matcher = ['test3.*', 'test4.*'], topic = "Topic expression 2", partition = "index-value", index-name="index1"},
#    {matcher = ['test1.*', 'test5.*'], topic = "Topic expression 3", partition = "table"},
#    {matcher = ['test6.*'], partition = "columns", columns = "['a', 'b']"}
#    {matcher = ['test7.*'], partition = "ts"}
# ]

# v7.5.0から、column-selectors が導入され、ダウンストリームがKafkaの場合にのみ有効です。
# column-selectors は、レプリケーション用に特定の列を選択するために使用されます。
# column-selectors = [
#     {matcher = ['test.t1'], columns = ['a', 'b']},
#     {matcher = ['test.*'], columns = ["*", "!b"]},
#     {matcher = ['test1.t1'], columns = ['column*', '!column1']},
#     {matcher = ['test3.t'], columns = ["column?", "!column1"]},
# ]

# プロトコル設定項目は、メッセージのエンコーディングに使用されるプロトコル形式を指定します。
# ダウンストリームがKafkaの場合、プロトコルには、canal-json、avro、あるいは open-protocol のいずれかしか指定できません。
# ダウンストリームがPulsarの場合、プロトコルには canal-json のみ指定できます。
# ダウンストリームがストレージサービスの場合、プロトコルには canal-json または csv のいずれかしか指定できません。
# 注意: この設定項目はダウンストリームがKafka、Pulsar、またはストレージサービスの場合のみ有効です。
# protocol = "canal-json"

# v7.2.0から、`delete-only-output-handle-key-columns`パラメーターは、DELETE イベントの出力を指定します。このパラメーターは、canal-json と open-protocol プロトコルのみ有効です。
# このパラメーターは、`force-replicate`と互換性があります。このパラメーターと`force-replicate`の両方が `true` に設定されている場合、TiCDC は changefeed を作成する際にエラーを報告します。
# デフォルト値は false で、すべての列を出力します。true に設定すると、プライマリキーカラムかユニークインデックスカラムのみが出力されます。
# Avro プロトコルは、このパラメーターによって制御されず、常にプライマリキーカラムまたはユニークインデックスカラムのみが出力されます。
# CSV プロトコルは、このパラメーターによって制御されず、常にすべての列が出力されます。
delete-only-output-handle-key-columns = false

# スキーマレジストリの URL。
# 注意: この設定項目はダウンストリームがMQの場合のみ有効です。
# schema-registry = "http://localhost:80801/subjects/{subject-name}/versions/{version-number}/schema"

# データをエンコードする際に使用するエンコーダースレッドの数を指定します。
# 注意: この設定項目はダウンストリームがMQの場合のみ有効です。
# デフォルト値は 32 です。
# encoder-concurrency = 32

# kafka-go sink ライブラリを使用する kafka-sink-v2 を有効にするかどうかを指定します。
# 注意: この設定項目はダウンストリームがMQの場合のみ有効です。
# デフォルト値は false です。
# enable-kafka-sink-v2 = false

# v7.1.0から、この設定項目は更新された列のみを出力するかどうかを指定します。
# 注意: この設定項目は open-protocol および canal-json を使用する MQ ダウンストリームにのみ適用されます。
# デフォルト値は false です。
# only-output-updated-columns = false

############ ストレージシンク設定項目 ############
# 以下の3つの設定項目は、データをストレージシンクにレプリケートする際にのみ使用され、MQ または MySQL シンクにデータをレプリケートする場合は無視できます。
# 2つのデータ変更イベントを区切るために使用される行終端記号。デフォルト値は空の文字列で、"\r\n" が使用されます。
# terminator = ''
# ファイルディレクトリで使用される日付区切り文字のタイプ。値のオプションは `none`、`year`、`month`、`day` です。`day` がデフォルト値で、日付でファイルが分割されます。<https://docs.pingcap.com/tidb/stable/ticdc-sink-to-cloud-storage#data-change-records>を参照してください。
# 注意: この設定項目はダウンストリームがストレージサービスの場合のみ有効です。
date-separator = 'day'
# 分割テーブルのデータを個々のディレクトリに格納するかどうかを指定します。デフォルト値は true で、テーブル内のパーティションは個々のディレクトリに格納されます。<https://github.com/pingcap/tiflow/issues/8724>におけるダウンストリームの分割テーブルでの潜在的なデータ損失を避けるために、`true` を設定することをお勧めします。使用例については、<https://docs.pingcap.com/tidb/dev/ticdc-sink-to-cloud-storage#data-change-records)>を参照してください。
# 注意: この設定項目はダウンストリームがストレージサービスの場合のみ有効です。
enable-partition-separator = true

# v6.5.0から、TiCDCはストレージサービスへのデータ変更の保存を CSV 形式でサポートしています。MQ または MySQL シンクにデータをレプリケートする場合は、以下の設定を無視してください。
# [sink.csv]
# CSV ファイルでフィールドを区切るために使用される文字。値は ASCII 文字である必要があり、デフォルト値は `,` です。
# delimiter = ','
# CSV ファイルでフィールドを囲むために使用される引用文字。デフォルト値は `"` です。値が空の場合、引用符は使用されません。
# quote = '"'
# CSV 列が null の場合に表示される文字。デフォルト値は `\N` です。
# null = '\N'
# CSV 行に commit-ts を含めるかどうか。デフォルト値は false です。
# include-commit-ts = false
# 2進データのエンコーディング方法。'base64' または 'hex' が指定できます。デフォルト値は 'base64' です。
# binary-encoding-method = 'base64'

# redo log を使用する changefeed に対するレプリケーション整合性設定を指定します。詳細については、https://docs.pingcap.com/tidb/stable/ticdc-sink-to-mysql#eventually-consistent-replication-in-disaster-scenarios を参照してください。
# 注意: 一貫性関連の構成項目は、ダウンストリームがデータベースで redo log 機能が有効の場合のみ有効です。
[consistent]
# データの整合性レベル。使用できるオプションは "none" と "eventual" です。"none" は redo log が無効を意味します。
# デフォルト値は "none" です。
level = "none"
# MB 単位の最大 redo log サイズ。
# デフォルト値は 64 です。
max-log-size = 64
# redo log のフラッシュ間隔。デフォルト値は 2000 ミリ秒です。
flush-interval = 2000
# redo log のストレージ URI。
# デフォルト値は空です。
storage = ""
# redo log をファイルに保存するかどうかを指定します。
# デフォルト値は false です。
use-file-backend = false

[integrity]
# 単一行のデータのチェックサム検証を有効にするかどうか。デフォルト値は "none" で、機能は無効になります。値のオプションは "none" と "correctness" です。
integrity-check-level = "none"
# 単一行のデータのチェックサム検証が失敗した場合の Changefeed のログレベルを指定します。デフォルト値は "warn" です。値のオプションは "warn" と "error" です。
corruption-handle-level = "warn"

# 以下の構成項目はダウンストリームが Kafka の場合のみ有効です。
[sink.kafka-config]
# Kafka SASL 認証のメカニズム。デフォルト値は空で、SASL 認証は使用されません。
sasl-mechanism = "OAUTHBEARER"
# Kafka SASL OAUTHBEARER 認証で使用されるクライアント ID。デフォルト値は空です。OAUTHBEARER 認証が使用される場合、このパラメーターが必要です。
sasl-oauth-client-id = "producer-kafka"
# Kafka SASL OAUTHBEARER 認証で使用されるクライアントシークレット。デフォルト値は空です。OAUTHBEARER 認証が使用される場合、このパラメーターが必要です。
sasl-oauth-client-secret = "cHJvZHVjZXIta2Fma2E="
# トークンを取得するための Kafka SASL OAUTHBEARER 認証の token URL。デフォルト値は空です。OAUTHBEARER 認証が使用される場合、このパラメーターが必要です。
sasl-oauth-token-url = "http://127.0.0.1:4444/oauth2/token"
# Kafka SASL OAUTHBEARER 認証でのスコープ。デフォルト値は空です。OAUTHBEARER 認証が使用される場合、このパラメーターはオプションです。
sasl-oauth-scopes = ["producer.kafka", "consumer.kafka"]
# Kafka SASL OAUTHBEARER 認証で使用される grant-type。デフォルト値は "client_credentials" です。OAUTHBEARER 認証が使用される場合、このパラメーターはオプションです。
# Kafka SASL OAUTHBEARER認証の受信者。デフォルト値は空です。このパラメータは、OAUTHBEARER認証が使用されている場合にオプションです。
sasl-oauth-audience = "kafka"

# 次のパラメータは、下流がPulsarの場合にのみ有効です。
[sink.pulsar-config]
# Pulsarサーバーでの認証は、トークンを使用して行われます。トークンの値を指定してください。
authentication-token = "xxxxxxxxxxxxx"
# Pulsarサーバー認証用のトークンを使用する場合は、トークンが保存されているファイルのパスを指定してください。
token-from-file="/data/pulsar/token-file.txt"
# Pulsarは基本アカウントとパスワードを使用してアイデンティティを認証します。アカウントを指定してください。
basic-user-name="root"
# Pulsarは基本アカウントとパスワードを使用してアイデンティティを認証します。パスワードを指定してください。
basic-password="password"
# Pulsar TLS暗号化認証用の証明書パス。
auth-tls-certificate-path="/data/pulsar/certificate"
# Pulsar TLS暗号化認証用の秘密鍵パス。
auth-tls-private-key-path="/data/pulsar/certificate.key"
# Pulsar TLS暗号化認証の信頼される証明書ファイルへのパス。
tls-trust-certs-file-path="/data/pulsar/tls-trust-certs-file"
# Pulsar oauth2発行者URL。詳細は、Pulsarのウェブサイトを参照してください: https://pulsar.apache.org/docs/2.10.x/client-libraries-go/#tls-encryption-and-authentication
oauth2.oauth2-issuer-url="https://xxxx.auth0.com"
# Pulsar oauth2オーディエンス
oauth2.oauth2-audience="https://xxxx.auth0.com/api/v2/"
# Pulsar oauth2秘密鍵
oauth2.oauth2-private-key="/data/pulsar/privateKey"
# Pulsar oauth2クライアントID
oauth2.oauth2-client-id="0Xx...Yyxeny"
# Pulsar oauth2 oauth2スコープ
oauth2.oauth2-scope="xxxx"
# TiCDC内のキャッシュされたPulsarプロデューサーの数。デフォルト値は10240です。各Pulsarプロデューサーは1つのトピックに対応します。必要なトピックの数がデフォルト値よりも大きい場合は、数を増やす必要があります。
pulsar-producer-cache-size=10240
# Pulsarデータ圧縮方式。デフォルトでは圧縮は使用されません。オプション値は「lz4」「zlib」「zstd」です。
compression-type=""
# PulsarクライアントがサーバーとのTCP接続を確立するためのタイムアウト。デフォルト値は5秒です。
connection-timeout=5
# Pulsarクライアントがトピックの作成や購読などの操作を開始するためのタイムアウト。デフォルト値は30秒です。
operation-timeout=30
# Pulsarプロデューサーが送信する単一バッチ内のメッセージの最大数。デフォルト値は1000です。
batching-max-messages=1000
# Pulsarプロデューサーがバッチングのためにメッセージを保存する間隔。デフォルト値は10ミリ秒です。
batching-max-publish-delay=10
# Pulsarプロデューサーがメッセージを送信するタイムアウト。デフォルト値は30秒です。
send-timeout=30