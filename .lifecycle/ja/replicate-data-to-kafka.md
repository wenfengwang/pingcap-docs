---
title: Apache KafkaとApache Flinkを使用してデータを統合する
summary: TiDBデータをApache KafkaとApache Flinkに複製する方法について学びます。

# Apache KafkaとApache Flinkを使用してデータを統合する

このドキュメントでは、[TiCDC](/ticdc/ticdc-overview.md)を使用してTiDBデータをApache KafkaとApache Flinkにレプリケートする方法について説明します。このドキュメントの構成は次のとおりです。

1. TiCDCが含まれたTiDBクラスタを迅速に展開し、KafkaクラスタとFlinkクラスタを作成します。
2. TiDBからKafkaへのデータをレプリケートするchangefeedを作成します。
3. go-tpcを使用してTiDBにデータを書き込みます。
4. Kafkaコンソールコンシューマーでデータを確認し、指定されたKafkaトピックにデータがレプリケートされていることを確認します。
5. （オプション）FlinkクラスタをKafkaデータを消費するように構成します。

上記の手順は研究室環境で実行されます。これらの手順を参考にして、本番環境にクラスタを展開することもできます。

## ステップ1. 環境をセットアップする

1. TiCDCが含まれたTiDBクラスタを展開します。

    研究室やテスト環境では、TiUP Playgroundを使用して迅速にTiCDCが含まれたTiDBクラスタを展開できます。

    ```shell
    tiup playground --host 0.0.0.0 --db 1 --pd 1 --kv 1 --tiflash 0 --ticdc 1
    # クラスタステータスを表示
    tiup status
    ```

    TiUPがまだインストールされていない場合は、[TiUPのインストール](/tiup/tiup-overview.md#install-tiup)を参照してください。本番環境では、[TiCDCの展開](/ticdc/deploy-ticdc.md)に従ってTiCDCを展開できます。

2. Kafkaクラスタを作成します。

    - 研究室環境: [Apache Kafkaクイックスタート](https://kafka.apache.org/quickstart)を参照してKafkaクラスタを開始します。
    - 本番環境: [本番環境でのKafkaの実行](https://docs.confluent.io/platform/current/kafka/deployment.html)を参照してKafka本番クラスタを展開します。

3. （オプション）Flinkクラスタを作成します。

    - 研究室環境: [Apache Flink入門](https://nightlies.apache.org/flink/flink-docs-release-1.15/docs/try-flink/local_installation/)を参照してFlinkクラスタを開始します。
    - 本番環境: [Apache Kafkaの展開](https://nightlies.apache.org/flink/flink-docs-release-1.15/docs/deployment/overview/)を参照してFlink本番クラスタを展開します。

## ステップ2. Kafka changefeedを作成する

1. changefeed設定ファイルを作成します。

    Flinkで必要とされるのは、各テーブルの増分データが独立したトピックに送信され、プライマリキーの値に基づいて各イベントにパーティションが割り当てられることです。したがって、次の内容のchangefeed設定ファイル`changefeed.conf`を作成する必要があります。

    ```
    [sink]
    dispatchers = [
    {matcher = ['*.*'], topic = "tidb_{schema}_{table}", partition="index-value"},
    ]
    ```

    設定ファイル内の`dispatchers`の詳細については、[Kafkaシンクのトピックおよびパーティションディスパッチャのルールをカスタマイズする](/ticdc/ticdc-sink-to-kafka.md#customize-the-rules-for-topic-and-partition-dispatchers-of-kafka-sink)を参照してください。

2. Kafkaへの増分データのレプリケートのためのchangefeedを作成します。

    ```shell
    tiup ctl:v<CLUSTER_VERSION> cdc changefeed create --server="http://127.0.0.1:8300" --sink-uri="kafka://127.0.0.1:9092/kafka-topic-name?protocol=canal-json" --changefeed-id="kafka-changefeed" --config="changefeed.conf"
    ```

    - changefeedが正常に作成された場合、changefeedの情報（changefeed IDなど）が次のように表示されます：

        ```shell
        changefeedの作成に成功しました！
        ID: kafka-changefeed
        Info: {... changfeed info json struct ...}
        ```

    - コマンドを実行しても結果が返されない場合は、コマンドを実行するサーバとsink URIで指定されたKafkaマシンとのネットワーク接続を確認してください。

    本番環境では、Kafkaクラスタには複数のブローカーノードがあります。したがって、sink URIに複数のブローカーのアドレスを追加できます。これにより、Kafkaクラスタへの安定したアクセスが確保されます。Kafkaクラスタがダウンしても、changefeedは動作し続けます。Kafkaクラスタに3つのブローカーノードがあり、それぞれのIPアドレスが127.0.0.1:9092、127.0.0.2:9092、127.0.0.3:9092であるとします。次のようなsink URIを使用してchangefeedを作成できます。

    ```shell
    tiup ctl:v<CLUSTER_VERSION> cdc changefeed create --server="http://127.0.0.1:8300" --sink-uri="kafka://127.0.0.1:9092,127.0.0.2:9092,127.0.0.3:9092/kafka-topic-name?protocol=canal-json&partition-num=3&replication-factor=1&max-message-bytes=1048576" --config="changefeed.conf"
    ```

3. changefeedを作成した後、次のコマンドを実行してchangefeedの状態を確認します：

    ```shell
    tiup ctl:v<CLUSTER_VERSION> cdc changefeed list --server="http://127.0.0.1:8300"
    ```

    changefeedの管理については、[TiCDCのchangefeedの管理](/ticdc/ticdc-manage-changefeed.md)を参照してください。

## ステップ3. 変更ログを生成するためにデータを書き込む

前述の手順が完了した後、TiCDCはTiDBクラスタの増分データの変更ログをKafkaに送信します。このセクションでは、変更ログを生成するためにTiDBにデータを書き込む方法について説明します。

1. サービス負荷をシミュレートします。

    研究室環境で変更ログを生成するために、go-tpcを使用してTiDBクラスタにデータを書き込むことができます。具体的には、次のコマンドを実行して、TiUPベンチを使用して`tpcc`データベースを作成し、新しいデータベースにデータを書き込みます。

    ```shell
    tiup bench tpcc -H 127.0.0.1 -P 4000 -D tpcc --warehouses 4 prepare
    tiup bench tpcc -H 127.0.0.1 -P 4000 -D tpcc --warehouses 4 run --time 300s
    ```

    go-tpcについての詳細については、[TiDBでTPC-Cテストを実行する方法](/benchmark/benchmark-tidb-using-tpcc.md)を参照してください。

2. Kafkaトピックでデータを消費します。

    changefeedが正常に動作する場合、データはKafkaトピックに書き込まれます。`kafka-console-consumer.sh`を実行します。そうすると、データが正常にKafkaトピックに書き込まれているのを確認できます。

    ```shell
    ./bin/kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --from-beginning --topic `${topic-name}`
    ```

この時点で、TiDBデータベースの増分データがKafkaに正常にレプリケートされます。次に、Flinkを使用してKafkaデータを消費するか、特定のサービスシナリオ用にKafkaコンシューマクライアントを開発することができます。

## （オプション）ステップ4. Flinkを使用してKafkaデータを消費するように構成する

1. Flink Kafkaコネクタをインストールします。

    Flinkエコシステムでは、Flink Kafkaコネクタを使用してKafkaデータを消費し、データをFlinkに出力します。ただし、Flink Kafkaコネクタは自動的にはインストールされません。これを使用するには、Flinkのインストール後にFlink Kafkaコネクタとその依存関係をFlinkのインストールディレクトリの`lib`ディレクトリに追加し、新しいプラグインをロードするためにFlinkクラスタを再起動する必要があります。

    - [flink-connector-kafka-1.15.0.jar](https://repo.maven.apache.org/maven2/org/apache/flink/flink-connector-kafka/1.15.0/flink-connector-kafka-1.15.0.jar)
    - [flink-sql-connector-kafka-1.15.0.jar](https://repo.maven.apache.org/maven2/org/apache/flink/flink-sql-connector-kafka/1.15.0/flink-sql-connector-kafka-1.15.0.jar)
    - [kafka-clients-3.2.0.jar](https://repo.maven.apache.org/maven2/org/apache/kafka/kafka-clients/3.2.0/kafka-clients-3.2.0.jar)

2. テーブルを作成します。

    Flinkがインストールされているディレクトリで、次のコマンドを実行してFlink SQLクライアントを起動します。

    ```shell
    [root@flink flink-1.15.0]# ./bin/sql-client.sh
    ```

    次に、次のコマンドを実行して`tpcc_orders`という名前のテーブルを作成します。

    ```sql
    CREATE TABLE tpcc_orders (
        o_id INTEGER,
        o_d_id INTEGER,
        o_w_id INTEGER,
        o_c_id INTEGER,
```
    o_entry_d STRING,
    o_carrier_id INTEGER,
    o_ol_cnt INTEGER,
    o_all_local INTEGER
) WITH (
'connector' = 'kafka',
'topic' = 'tidb_tpcc_orders',
'properties.bootstrap.servers' = '127.0.0.1:9092',
'properties.group.id' = 'testGroup',
'format' = 'canal-json',
'scan.startup.mode' = 'earliest-offset',
'properties.auto.offset.reset' = 'earliest'
)

環境の実際の値で `topic` および `properties.bootstrap.servers` を置換してください。

3. テーブルのデータをクエリします。

    次のコマンドを実行して、`tpcc_orders` テーブルのデータをクエリしてください。

    ```sql
    SELECT * FROM tpcc_orders;
    ```

    このコマンドを実行すると、テーブルに新しいデータがあることが次の図で示されているように確認できます。

    ![SQL クエリ結果](/media/integrate/sql-query-result.png)

Kafka とのデータ統合が完了しました。
```