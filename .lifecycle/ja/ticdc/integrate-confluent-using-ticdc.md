---
title: Confluent Cloud と Snowflake とのデータ統合
summary: TiDB データを Confluent Cloud、Snowflake、ksqlDB、SQL Server にストリーミングする方法を学んでください。

# Confluent Cloud と Snowflake とのデータ統合

Confluent は Apache Kafka 互換のストリーミングデータプラットフォームであり、強力なデータ統合機能を提供します。このプラットフォームでは、中断することなくリアルタイムのストリーミングデータにアクセスし、それを保存および管理できます。

TiDB v6.1.0 から、TiCDC は Avro 形式で Confluent にインクリメンタルデータをレプリケートできるようになりました。このドキュメントでは、[TiCDC](/ticdc/ticdc-overview.md) を使用して TiDB のインクリメンタルデータを Confluent にレプリケートし、さらに Confluent Cloud を介してデータを Snowflake、ksqlDB、および SQL Server にレプリケートする方法を紹介します。このドキュメントの構成は次のとおりです。

1. TiDB クラスタを TiCDC を含めて迅速にデプロイする。
2. TiDB から Confluent Cloud へデータをレプリケートするための changefeed を作成する。
3. Confluent Cloud から Snowflake、ksqlDB、および SQL Server へデータをレプリケートする Connectors を作成する。
4. go-tpc を使用して TiDB にデータを書き込み、それらの変更を Snowflake、ksqlDB、および SQL Server で観察する。

上記のステップは実験環境で実行されます。これらのステップを参照して、本番環境でクラスタをデプロイすることもできます。

## Confluent Cloud へのインクリメンタルデータのレプリケート

### Step 1. 環境をセットアップする

1. TiCDC を含めた TiDB クラスタをデプロイする。

    実験またはテスト環境では、TiUP Playground を使用して迅速に TiCDC を含めた TiDB クラスタをデプロイできます。

    ```shell
    tiup playground --host 0.0.0.0 --db 1 --pd 1 --kv 1 --tiflash 0 --ticdc 1
    # クラスタステータスを表示
    tiup status
    ```

    まだ TiUP がインストールされていない場合は、[TiUP のインストール](/tiup/tiup-overview.md#install-tiup)を参照してください。本番環境では、[Deploy TiCDC](/ticdc/deploy-ticdc.md) の手順に従って TiCDC をデプロイできます。

2. Confluent Cloud に登録して Confluent クラスタを作成する。

    Basic クラスタを作成し、インターネット経由でアクセス可能にします。詳細については、[Confluent Cloud のクイックスタート](https://docs.confluent.io/cloud/current/get-started/index.html)を参照してください。

### Step 2. アクセスキーペアを作成する

1. クラスタ API キーを作成する。

    [Confluent Cloud](https://confluent.cloud) にサインインします。**Data integration** > **API keys** > **Create key** を選択します。表示される **Select scope for API key** ページで、**Global access** を選択します。

    作成後、以下のようにキーペアファイルが生成されます。

    ```
    === Confluent Cloud API key: xxx-xxxxx ===

    API key:
    L5WWA4GK4NAT2EQV

    API secret:
    xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

    ブートストラップサーバー:
    xxx-xxxxx.ap-east-1.aws.confluent.cloud:9092
    ```

2. Schema Registry エンドポイントを記録する。

    Confluent Cloud コンソールで、 **Schema Registry** > **API endpoint** を選択します。Schema Registry エンドポイントを記録します。以下は例です。

    ```
    https://yyy-yyyyy.us-east-2.aws.confluent.cloud
    ```

3. Schema Registry API キーを作成する。

    Confluent Cloud コンソールで、**Schema Registry** > **API credentials** を選択します。**Edit** をクリックし、次に **Create key** をクリックします。

    作成後、以下のようにキーペアファイルが生成されます:

    ```
    === Confluent Cloud API key: yyy-yyyyy ===
    API key:
    7NBH2CAFM2LMGTH7
    API secret:
    xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    ```

    この手順はConfluent CLI を使用しても実行できます。詳細については、[Confluent CLI を Confluent Cloud クラスタに接続](https://docs.confluent.io/confluent-cli/current/connect.html)を参照してください。

### Step 3. Kafka changefeed を作成する

1. changefeed 構成ファイルを作成します。

    Avro および Confluent Connector に必要なように、各テーブルのインクリメンタルデータは独立したトピックに送信され、プライマリキーの値に基づいて各イベントにパーティションがディスパッチされる必要があります。 したがって、以下の内容で changefeed 構成ファイル `changefeed.conf` を作成する必要があります。

    ```
    [sink]
    dispatchers = [
    {matcher = ['*.*'], topic = "tidb_{schema}_{table}", partition="index-value"},
    ]
    ```

    構成ファイル内の `dispatchers` の詳細な説明については、[Kafka Sink の Topic と Partition Dispatcher のルールをカスタマイズする](/ticdc/ticdc-sink-to-kafka.md#customize-the-rules-for-topic-and-partition-dispatchers-of-kafka-sink)を参照してください。

2. Confluent Cloud にインクリメンタルデータをレプリケートする changefeed を作成します:

    ```shell
    tiup ctl:v<CLUSTER_VERSION> cdc changefeed create --server="http://127.0.0.1:8300" --sink-uri="kafka://<broker_endpoint>/ticdc-meta?protocol=avro&replication-factor=3&enable-tls=true&auto-create-topic=true&sasl-mechanism=plain&sasl-user=<broker_api_key>&sasl-password=<broker_api_secret>" --schema-registry="https://<schema_registry_api_key>:<schema_registry_api_secret>@<schema_registry_endpoint>" --changefeed-id="confluent-changefeed" --config changefeed.conf
    ```

    以下のフィールドの値を、[Step 2. アクセスキーペアを作成する](#step-2-create-an-access-key-pair) で作成または記録した値に置き換える必要があります。

    - `<broker_endpoint>`
    - `<broker_api_key>`
    - `<broker_api_secret>`
    - `<schema_registry_api_key>`
    - `<schema_registry_api_secret>`
    - `<schema_registry_endpoint>`

    すべての前述のフィールドの値を置き換えた後の構成ファイルは以下のようになります。

    ```shell
    tiup ctl:v<CLUSTER_VERSION> cdc changefeed create --server="http://127.0.0.1:8300" --sink-uri="kafka://xxx-xxxxx.ap-east-1.aws.confluent.cloud:9092/ticdc-meta?protocol=avro&replication-factor=3&enable-tls=true&auto-create-topic=true&sasl-mechanism=plain&sasl-user=L5WWA4GK4NAT2EQV&sasl-password=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" --schema-registry="https://7NBH2CAFM2LMGTH7:xxxxxxxxxxxxxxxxxx@yyy-yyyyy.us-east-2.aws.confluent.cloud" --changefeed-id="confluent-changefeed" --config changefeed.conf
    ```

    - changefeed を作成するコマンドを実行します。

        - changefeed が正常に作成された場合、changefeed ID などの changefeed 情報が表示されます。

            ```shell
            changefeed の作成に成功しました！
            ID: confluent-changefeed
            Info: {... changfeed info json struct ...}
            ```

        - コマンドを実行しても結果が返されない場合は、コマンドを実行するサーバーと Confluent Cloud のネットワーク接続を確認してください。詳細については、[Confluent Cloud への接続テスト](https://docs.confluent.io/cloud/current/networking/testing.html)を参照してください。

3. changefeed を作成した後、以下のコマンドを実行して changefeed の状態を確認します:

    ```shell
    tiup ctl:v<CLUSTER_VERSION> cdc changefeed list --server="http://127.0.0.1:8300"
    ```

    changefeed の管理については、[TiCDC の changefeed を管理する](/ticdc/ticdc-manage-changefeed.md)を参照してください。

### Step 4. 変更ログを生成するためのデータの書き込み

前述のステップが完了したら、TiCDC は TiDB クラスタのインクリメンタルデータの変更ログを Confluent Cloud に送信します。このセクションでは、変更ログを生成するために TiDB にデータを書き込む方法について説明します。

1. サービスのワークロードをシミュレートする。

    実験環境で変更ログを生成するために、go-tpc を使用して TiDB クラスタにデータを書き込むことができます。具体的には、以下のコマンドを実行して TiDB クラスタに `tpcc` というデータベースを作成し、TiUP bench を使用してこの新しいデータベースにデータを書き込みます。

    ```shell
    tiup bench tpcc -H 127.0.0.1 -P 4000 -D tpcc --warehouses 4 prepare
    tiup bench tpcc -H 127.0.0.1 -P 4000 -D tpcc --warehouses 4 run --time 300s
    ```

```
    go-tpcの詳細については、[How to Run TPC-C Test on TiDB](/benchmark/benchmark-tidb-using-tpcc.md) を参照してください。

    Confluent Cloud でデータを観察する。

    ![Confluent topics](/media/integrate/confluent-topics.png)

    Confluent Cloud コンソールで**Topics**をクリックします。ターゲットのトピックが作成され、データが受信されていることが確認できます。この時点で、TiDBデータベースの増分データはConfluent Cloudに正常にレプリケートされています。

## Snowflakeとデータを統合する

Snowflakeはクラウドネイティブのデータウェアハウスです。Confluentを使用すると、Snowflake Sink Connectorsを作成してTiDBの増分データをSnowflakeにレプリケートできます。

### 必要条件

- Snowflakeクラスターを登録および作成していること。[Getting Started with Snowflake](https://docs.snowflake.com/en/user-guide-getting-started.html)を参照してください。
- Snowflakeクラスターに接続する前にプライベートキーを生成していること。[Key Pair Authentication & Key Pair Rotation](https://docs.snowflake.com/en/user-guide/key-pair-auth.html)を参照してください。

### 統合手順

1. Snowflakeでデータベースとスキーマを作成します。

    Snowflakeコントロールコンソールで**Data** > **Database**を選択します。データベース名を`TPCC`、スキーマ名を`TiCDC`として作成します。

2. Confluent Cloud コンソールで**Data integration** > **Connectors** > **Snowflake Sink**を選択します。以下のページが表示されます。

    ![Add snowflake sink connector](/media/integrate/add-snowflake-sink-connector.png)

3. Snowflakeにレプリケートするトピックを選択して、次のページに進みます。

    ![Configuration](/media/integrate/configuration.png)

4. Snowflakeへの接続認証情報を指定します。前の手順で作成した値を使用して、**Database name** と **Schema name**を入力し、次のページに進みます。

    ![Configuration](/media/integrate/configuration.png)

5. **Configuration**ページで、**Input Kafka record value format**と**Input Kafka record key format**にそれぞれ`AVRO`を選択します。その後、**Continue**をクリックします。コネクタが作成され、ステータスが**Running**になるまで（数分かかる場合があります）待ちます。

    ![Data preview](/media/integrate/data-preview.png)

6. Snowflakeコンソールで**Data** > **Database** > **TPCC** > **TiCDC**を選択します。TiDBの増分データがSnowflakeにレプリケートされていることが確認できます（前の図を参照）。ただし、Snowflakeのテーブル構造はTiDBと異なり、データが増分で挿入されます。ほとんどのシナリオでは、SnowflakeのデータがTiDBのデータのレプリカとして保存されることが期待されます。次のセクションでこの問題に取り組みます。

### SnowflakeにおけるTiDBテーブルのデータレプリカの作成

前のセクションでは、TiDBの増分データの変更ログがSnowflakeにレプリケートされています。このセクションでは、SnowflakeのTASKとSTREAM機能を使用してこれらの変更ログを処理し、`INSERT`、`UPDATE`、`DELETE`のイベントタイプに応じて処理し、上流と同じ構造のテーブルに書き込むことで、SnowflakeにTiDBテーブルのデータレプリカを作成する方法について説明します。以下は`ITEM` テーブルを例にしたものです。

`ITEM` テーブルの構造は以下の通りです：

```
CREATE TABLE `item` (
  `i_id` int(11) NOT NULL,
  `i_im_id` int(11) DEFAULT NULL,
  `i_name` varchar(24) DEFAULT NULL,
  `i_price` decimal(5,2) DEFAULT NULL,
  `i_data` varchar(50) DEFAULT NULL,
  PRIMARY KEY (`i_id`)
);
```

Snowflakeでは、Confluent Snowflake Sink Connectorによって自動的に作成された`TIDB_TEST_ITEM`という名前のテーブルがあります。このテーブルの構造は以下の通りです：

```
create or replace TABLE TIDB_TEST_ITEM (
        RECORD_METADATA VARIANT,
        RECORD_CONTENT VARIANT
);
```

1. Snowflakeで、TiDBと同じ構造のテーブルを作成します：

    ```
    create or replace table TEST_ITEM (
        i_id INTEGER primary key,
        i_im_id INTEGER,
        i_name VARCHAR,
        i_price DECIMAL(36,2),
        i_data VARCHAR
    );
    ```

2. `TIDB_TEST_ITEM`のためのストリームを作成し、`append_only`を`true`に設定します。

    ```
    create or replace stream TEST_ITEM_STREAM on table TIDB_TEST_ITEM append_only=true;
    ```

    このようにすることで、作成されたストリームはリアルタイムで`INSERT`イベントだけをキャプチャします。具体的には、TiDBの`ITEM`の新しい変更ログが生成されると、その変更ログが`TIDB_TEST_ITEM`に挿入され、ストリームによってキャプチャされます。

3. ストリーム内のデータを処理します。イベントタイプに応じて、ストリームデータを`TEST_ITEM`テーブルで挿入、更新、または削除します。

    ```
    -- TEST_ITEMテーブルにデータをマージ
    merge into TEST_ITEM n
      using
          -- TEST_ITEM_STREAMをクエリ
          (SELECT RECORD_METADATA:key as k, RECORD_CONTENT:val as v from TEST_ITEM_STREAM) stm
          -- ストリームをテーブルと一致させ、i_idが等しい条件で
          on k:i_id = n.i_id
      -- TEST_ITEMテーブルにi_idとvが空で一致するレコードがある場合、このレコードを削除
      when matched and IS_NULL_VALUE(v) = true then
          delete

      -- TEST_ITEMテーブルにi_idとvが空でない一致するレコードがある場合、このレコードを更新
      when matched and IS_NULL_VALUE(v) = false then
          update set n.i_data = v:i_data, n.i_im_id = v:i_im_id, n.i_name = v:i_name, n.i_price = v:i_price

      -- TEST_ITEMテーブルにi_idと一致するレコードがない場合、このレコードを挿入
      when not matched then
          insert
              (i_data, i_id, i_im_id, i_name, i_price)
          values
              (v:i_data, v:i_id, v:i_im_id, v:i_name, v:i_price)
    ;
    ```

    上記の例では、Snowflakeの`MERGE INTO`ステートメントを使用して特定の条件でストリームとテーブルを一致させ、レコードを削除、更新、または挿入する操作を実行しています。この例では、以下の3つのシナリオについて、それぞれ3つの`WHERE`句を使用しています：

    - ストリームとテーブルが一致し、ストリーム内のデータが空の場合は、テーブルのレコードを削除します。
    - ストリームとテーブルが一致し、ストリーム内のデータが空でない場合は、テーブルのレコードを更新します。
    - ストリームとテーブルが一致せず、ストリーム内のデータが空でない場合は、テーブルに新しいレコードを挿入します。

4. ステップ3のステートメントを定期的に実行して、データが常に最新であるようにします。また、Snowflakeの`SCHEDULED TASK`機能を使用することもできます。

    ```
    -- 定期的にMERGE INTOステートメントを実行するためのTASKを作成
    create or replace task STREAM_TO_ITEM
        warehouse = test
        -- 1分ごとにTASKを実行
        schedule = '1 minute'
    when
        -- TEST_ITEM_STREAMにデータがない場合はTASKをスキップ
        system$stream_has_data('TEST_ITEM_STREAM')
    as
    -- TEST_ITEMテーブルにデータをマージ。ステートメントは前の例と同じです
    merge into TEST_ITEM n
      using
          (select RECORD_METADATA:key as k, RECORD_CONTENT:val as v from TEST_ITEM_STREAM) stm
          on k:i_id = n.i_id
      when matched and IS_NULL_VALUE(v) = true then
          delete
      when matched and IS_NULL_VALUE(v) = false then
          update set n.i_data = v:i_data, n.i_im_id = v:i_im_id, n.i_name = v:i_name, n.i_price = v:i_price
      when not matched then
          insert
              (i_data, i_id, i_im_id, i_name, i_price)
          values
              (v:i_data, v:i_id, v:i_im_id, v:i_name, v:i_price)
    ;
    ```

この時点で、特定のETL機能を備えたデータチャネルを確立しました。このデータチャネルを通じて、TiDBの増分データ変更ログをSnowflakeにレプリケートし、TiDBのデータのレプリカを維持し、Snowflake内のデータを使用できます。

最後のステップは、`TIDB_TEST_ITEM`テーブルの不要なデータを定期的にクリーンアップすることです：

```
-- TIDB_TEST_ITEMテーブルを2時間ごとにクリーンアップ
create or replace task TRUNCATE_TIDB_TEST_ITEM
    warehouse = test
    schedule = '120 minute'
when
    system$stream_has_data('TIDB_TEST_ITEM')
as
    TRUNCATE table TIDB_TEST_ITEM;
```

## ksqlDBとデータを統合する

ksqlDBはストリーム処理アプリケーション向けに特別に作られたデータベースです。Confluent Cloud上でksqlDBクラスターを作成し、TiCDCによってレプリケートされた増分データにアクセスできます。

1. Confluent Cloud コンソールで**ksqlDB**を選択し、指示に従ってksqlDBクラスターを作成します。
```
ksqlDBクラスターの状態が**Running**になるまでお待ちください。このプロセスには数分かかります。

2. ksqlDB Editorで、次のコマンドを実行して、`tidb_tpcc_orders`トピックにアクセスするストリームを作成します。

    ```sql
    CREATE STREAM orders (o_id INTEGER, o_d_id INTEGER, o_w_id INTEGER, o_c_id INTEGER, o_entry_d STRING, o_carrier_id INTEGER, o_ol_cnt INTEGER, o_all_local INTEGER) WITH (kafka_topic='tidb_tpcc_orders', partitions=3, value_format='AVRO');
    ```

3. 次のコマンドを実行して、ordersストリームのデータを確認します。

    ```sql
    SELECT * FROM ORDERS EMIT CHANGES;
    ```

    ![Select from orders](/media/integrate/select-from-orders.png)

    前述の図に示されているように、増分データがksqlDBにレプリケーションされたことが確認できます。ksqlDBとのデータ統合が完了しました。

## SQL Serverを使用したデータの統合

Microsoft SQL Serverは、Microsoftによって開発されたリレーショナルデータベース管理システム（RDBMS）です。Confluentを使用すると、SQL Serverシンクコネクタを作成して、TiDBの増分データをSQL Serverにレプリケーションできます。

1. SQL Serverに接続し、`tpcc`という名前のデータベースを作成します。

    ```shell
    [ec2-user@ip-172-1-1-1 bin]$ sqlcmd -S 10.61.43.14,1433 -U admin
    Password:
    1> create database tpcc
    2> go
    1> select name from master.dbo.sysdatabases
    2> go
    name
    ----------------------------------------------------------------------
    master
    tempdb
    model
    msdb
    rdsadmin
    tpcc
    (6 rows affected)
    ```

2. Confluent Cloudコンソールで、**Data integration** > **Connectors** > **Microsoft SQL Server Sink**を選択します。次に表示されるページは以下の通りです。

    ![Topic selection](/media/integrate/topic-selection.png)

3. SQL Serverにレプリケーションしたいトピックを選択して、次のページに移動します。

    ![Authentication](/media/integrate/authentication.png)

4. 接続情報と認証情報を入力し、次のページに進みます。

5. **Configuration**ページで、以下のフィールドを構成し、**Continue**をクリックします。

    | フィールド | 値 |
    | :- | :- |
    | Input Kafka record value format | AVRO |
    | Insert mode | UPSERT |
    | Auto create table | true |
    | Auto add columns | true |
    | PK mode | record_key |
    | Input Kafka record key format | AVRO |
    | Delete on null | true |

6. 構成が完了したら、**Continue**をクリックします。コネクタの状態が**Running**になるまでお待ちください。これには数分かかる場合があります。

    ![Results](/media/integrate/results.png)

7. SQL Serverに接続してデータを確認します。前述の図に示されているように、増分データがSQL Serverにレプリケーションされたことが確認できます。SQL Serverとのデータ統合が完了しました。