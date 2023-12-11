---
title: TiDB CloudのAirbyteとの統合
summary: Airbyte TiDBコネクタの使用方法を学ぶ

# TiDB CloudとAirbyteの統合

[Airbyte](https://airbyte.com/)は、オープンソースのデータ統合エンジンで、データウェアハウス、データレイク、およびデータベースに抽出、ロード、変換（ELT）パイプラインを構築し、データを統合することができます。このドキュメントでは、AirbyteをTiDB Cloudにソースまたはデスティネーションとして接続する方法について説明します。

## Airbyteの展開

Airbyteをわずかな手順でローカルに展開できます。

1. ワークスペースに[Docker](https://www.docker.com/products/docker-desktop)をインストールします。

2. Airbyteのソースコードをクローンします。

    ```shell
    git clone https://github.com/airbytehq/airbyte.git && \
    cd airbyte
    ```

3. docker-composeを使用してDockerイメージを実行します。

    ```shell
    docker-compose up
    ```

Airbyteのバナーが表示されたら、ユーザー名（`airbyte`）とパスワード（`password`）を使用してUIにアクセスするために<http://localhost:8000>に移動できます。

```
airbyte-server      |     ___    _      __          __
airbyte-server      |    /   |  (_)____/ /_  __  __/ /____
airbyte-server      |   / /| | / / ___/ __ \/ / / / __/ _ \
airbyte-server      |  / ___ |/ / /  / /_/ / /_/ / /_/  __/
airbyte-server      | /_/  |_/_/_/  /_.___/\__, /\__/\___/
airbyte-server      |                     /____/
airbyte-server      | --------------------------------------
airbyte-server      |  Now ready at http://localhost:8000/
airbyte-server      | --------------------------------------
```

## TiDBコネクタの設定

便利なことに、TiDBをソースとしてもデスティネーションとしても設定する手順は同じです。

1. サイドバーで**Sources**または**Destinations**をクリックし、新しいTiDBコネクタを作成するためにTiDBタイプを選択します。

2. 以下のパラメータを入力します。

    - ホスト: TiDB Cloudクラスターのエンドポイント
    - ポート: データベースのポート
    - データベース: データを同期したいデータベース
    - ユーザー名: データベースへのアクセス用のユーザー名
    - パスワード: ユーザー名のパスワード

    パラメータの値は、クラスターの接続ダイアログから取得できます。ダイアログを開くには、プロジェクトの[**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスターの名前をクリックして概要ページに移動し、右上隅の**Connect**をクリックします。

3. **SSL接続**を有効にし、**JDBC URL Params**でTLSプロトコルを**TLSv1.2**または**TLSv1.3**に設定します。

    > 注意:
    >
    > - TiDB CloudはTLS接続をサポートしています。例えば、`enabledTLSProtocols=TLSv1.2`のように、TLSプロトコルを**TLSv1.2**および**TLSv1.3**から選択できます。
    > - JDBCを介してTiDB CloudへのTLS接続を無効にしたい場合は、JDBC URL ParamsでuseSSLを`false`に設定してSSL接続を閉じる必要があります。例えば、`useSSL=false`。
    > - TiDB ServerlessはTLS接続のみをサポートしています。

4. **Set up source**または**destination**をクリックして、コネクタの作成を完了します。以下のスクリーンショットは、TiDBをソースとして設定した構成を示しています。

![TiDBソースの構成](/media/tidb-cloud/integration-airbyte-parameters.jpg)

TiDBからSnowflakeへ、またはCSVファイルからTiDBへなど、さまざまなソースとデスティネーションの組み合わせを使用できます。

TiDBコネクタの詳細については、[TiDBソース](https://docs.airbyte.com/integrations/sources/tidb)および[TiDBデスティネーション](https://docs.airbyte.com/integrations/destinations/tidb)を参照してください。

## 接続の設定

ソースとデスティネーションを設定した後、接続を構築し、構成することができます。

次の手順では、TiDBをソースとしてもデスティネーションとしても使用します。他のコネクタには異なるパラメータがある場合があります。

1. サイドバーで**Connections**をクリックし、**New Connection**をクリックします。
2. 以前に設定したソースとデスティネーションを選択します。
3. **Set up**コネクションパネルに移動し、接続の名前（`${source_name} - ${destination-name}`など）を作成します。
4. **Replication frequency**を**Every 24 hours**に設定して、接続を1日に1回データをレプリケートするようにします。
5. **Destination Namespace**を**Custom format**に設定し、**Namespace Custom Format**を**test**に設定して、すべてのデータを`test`データベースに格納できるようにします。
6. **Sync mode**を**Full refresh | Overwrite**に設定します。

    > **ヒント:**
    >
    > TiDBコネクタは増分およびフルリフレッシュの両方の同期をサポートしています。
    >
    > - 増分モードでは、Airbyteは最後の同期ジョブ以降にソースに追加されたレコードのみを読み取ります。増分モードを使用した最初の同期は、フルリフレッシュモードと同等です。
    > - フルリフレッシュモードでは、Airbyteはソースのすべてのレコードを読み取り、すべての同期タスクでデスティネーションにレプリケートします。Airbyteの**Namespace**という名前の各テーブルごとに同期モードを個別に設定できます。

    ![接続の設定](/media/tidb-cloud/integration-airbyte-connection.jpg)

7. **Normalization & Transformation**を**Normalized tabular data**に設定して、デフォルトの正規化モードを使用するか、ジョブ用のdbtファイルを設定できます。正規化についての詳細は、[Transformations and Normalization](https://docs.airbyte.com/operator-guides/transformation-and-normalization/transformations-with-dbt)を参照してください。
8. **Set up connection**をクリックします。
9. 接続が確立されたら、**ENABLED**をクリックして同期タスクを有効にするか、**Sync now**をクリックしてすぐに同期できます。

![データの同期](/media/tidb-cloud/integration-airbyte-sync.jpg)

## 制限事項

- TiDBコネクタはChange Data Capture (CDC)機能をサポートしていません。
- TiDBデスティネーションは、デフォルトの正規化モードでは`timestamp`タイプを`varchar`タイプに変換します。これは、Airbyteがタイムスタンプタイプを文字列に変換して送信するためであり、TiDBが`cast ('2020-07-28 14:50:15+1:00' as timestamp)`をサポートしていないためです。
- 一部の大規模なELTミッションでは、TiDBの[transaction restrictions](/develop/dev-guide-transaction-restraints.md#large-transaction-restrictions)のパラメータを増やす必要があります。

## 関連項目

[TiDB CloudからSnowflakeにデータを移行するためにAirbyteを使用する](https://www.pingcap.com/blog/using-airbyte-to-migrate-data-from-tidb-cloud-to-snowflake/)