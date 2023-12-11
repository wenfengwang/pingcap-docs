---
title: dbtをTiDB Cloudに統合する
summary: TiDB Cloudでのdbtのユースケースを学ぶ
---

# dbtをTiDB Cloudと統合する

[Data build tool (dbt)](https://www.getdbt.com/)は、SQLステートメントを通じてデータを変換するのに役立つ人気のあるオープンソースのデータ変換ツールです。[dbt-tidb](https://github.com/pingcap/dbt-tidb)プラグインを使用することで、TiDB Cloudで作業するアナリティクスエンジニアは、テーブルやビューを作成するプロセスを考えることなく、直接SQLを使用してデータを整形できます。

本書は、dbtプロジェクトを例にとり、TiDB Cloudでdbtを使用する方法を紹介します。

## ステップ1: dbtとdbt-tidbをインストールする

dbtとdbt-tidbは、1つのコマンドを使用してインストールできます。次のコマンドでは、dbt-tidbをインストールする際にdbtが依存関係としてインストールされます。

```shell
pip install dbt-tidb
```

dbtを個別にインストールすることもできます。dbtのドキュメントの[dbtのインストール方法](https://docs.getdbt.com/docs/get-started/installation)を参照してください。

## ステップ2: デモプロジェクトを作成する

dbtの機能を試すために、dbt-labが提供するデモプロジェクトである[jaffle_shop](https://github.com/dbt-labs/jaffle_shop)を使用できます。プロジェクトをGitHubから直接クローンできます。

```shell
git clone https://github.com/dbt-labs/jaffle_shop && \
cd jaffle_shop
```

`jaffle_shop`ディレクトリ内のすべてのファイルは次のように構成されています。

```shell
.
├── LICENSE
├── README.md
├── dbt_project.yml
├── etc
│ ├── dbdiagram_definition.txt
│ └── jaffle_shop_erd.png
├── models
│ ├── customers.sql
│ ├── docs.md
│ ├── orders.sql
│ ├── overview.md
│ ├── schema.yml
│ └── staging
│ ├── schema.yml
│ ├── stg_customers.sql
│ ├── stg_orders.sql
│ └── stg_payments.sql
└── seeds
├── raw_customers.csv
├── raw_orders.csv
└── raw_payments.csv
```

このディレクトリには次のようなものがあります。

- `dbt_project.yml`は、プロジェクトの構成ファイルであり、プロジェクト名とデータベース構成ファイルの情報を保持しています。

- `models`ディレクトリには、プロジェクトのSQLモデルとテーブルのスキーマが含まれています。データアナリストがこのセクションを作成します。モデルの詳細については、[SQLモデル](https://docs.getdbt.com/docs/build/sql-models)を参照してください。

- `seeds`ディレクトリには、データベースエクスポートツールによってダンプされたCSVファイルが保存されています。たとえば、[TiDB Cloudデータをエクスポート](https://docs.pingcap.com/tidbcloud/export-data-from-tidb-cloud)してDumplingを使用してCSVファイルにエクスポートできます。`jaffle_shop`プロジェクトでは、これらのCSVファイルは処理するための生データとして使用されます。

## ステップ3: プロジェクトの構成

プロジェクトを構成するには、次の手順を実行します。

1. グローバル構成を完了する。

    [プロファイルフィールドの説明](#description-of-profile-fields)を参照して、デフォルトのグローバルプロファイルである`~/.dbt/profiles.yml`を編集し、TiDB Cloudとの接続を構成します。

    ```shell
    sudo vi ~/.dbt/profiles.yml
    ```

    エディタで、次の構成を追加します。

    ```yaml
    jaffle_shop_tidb:
      target: dev
      outputs:
        dev:
          type: tidb
          server: gateway01.ap-southeast-1.prod.aws.tidbcloud.com
          port: 4000
          schema: analytics
          username: xxxxxxxxxxx.root
          password: "your_password"
    ```

    `server`、`port`、`username`の値は、クラスタの接続ダイアログから取得できます。このダイアログを開くには、プロジェクトの[**Clusters**](https://tidbcloud.com/console/clusters)ページに移動し、ターゲットクラスタの名前をクリックしてその概要ページに移動し、右上隅の**Connect**をクリックします。

2. プロジェクトの構成を完了する。

    `jaffle_shop`プロジェクトディレクトリで、プロジェクト構成ファイル`dbt_project.yml`を編集し、`profile`フィールドを`jaffle_shop_tidb`に変更します。この構成により、プロジェクトは`~/.dbt/profiles.yml`ファイルで指定されたデータベースからクエリを実行できます。

    ```shell
    vi dbt_project.yml
    ```

    エディタで、構成を次のように更新します。

    ```yaml
    name: 'jaffle_shop'

    config-version: 2
    version: '0.1'

    profile: 'jaffle_shop_tidb'                   # ここで修正してください

    model-paths: ["models"]                       # モデルパス
    seed-paths: ["seeds"]                         # シードパス
    test-paths: ["tests"]
    analysis-paths: ["analysis"]
    macro-paths: ["macros"]

    target-path: "target"
    clean-targets:
      - "target"
      - "dbt_modules"
      - "logs"

    require-dbt-version: [">=1.0.0", "<2.0.0"]

    models:
      jaffle_shop:
        materialized: table
        staging:
          materialized: view
    ```

3. 構成を検証する。

    次のコマンドを実行して、データベースとプロジェクトの構成が正しいかどうかを確認します。

    ```shell
    dbt debug
    ```

## ステップ4: (オプション) CSVファイルをロードする

> **注意:**
>
> このステップはオプションです。処理するデータがすでにターゲットデータベースにある場合は、このステップをスキップできます。

プロジェクトを作成し構成したら、CSVデータをロードし、CSVをターゲットデータベースのテーブルとして実体化する時がきました。

1. CSVデータをロードし、ターゲットデータベースのテーブルとしてCSVを実体化します。

    ```shell
    dbt seed
    ```

    次のは、実行例です。

    ```shell
    Running with dbt=1.0.1
    Partial parse save file not found. Starting full parse.
    Found 5 models, 20 tests, 0 snapshots, 0 analyses, 172 macros, 0 operations, 3 seed files, 0 sources, 0 exposures, 0 metrics

    Concurrency: 1 threads (target='dev')

    1 of 3 START seed file analytics.raw_customers.................................. [RUN]
    1 of 3 OK loaded seed file analytics.raw_customers.............................. [INSERT 100 in 0.19s]
    2 of 3 START seed file analytics.raw_orders..................................... [RUN]
    2 of 3 OK loaded seed file analytics.raw_orders................................. [INSERT 99 in 0.14s]
    3 of 3 START seed file analytics.raw_payments................................... [RUN]
    3 of 3 OK loaded seed file analytics.raw_payments............................... [INSERT 113 in 0.24s]
    ```

    結果に示されているように、seedファイルが開始され、`analytics.raw_customers`、`analytics.raw_orders`、`analytics.raw_payments`の3つのテーブルにデータがロードされました。

2. TiDB Cloudで結果を検証する。

    `show databases`コマンドを使用して、dbtが作成した新しい`analytics`データベースをリストします。`show tables`コマンドは、`analytics`データベースに3つのテーブルが存在することを示します。

    ```sql
    mysql> SHOW DATABASES;
    +--------------------+
    | Database           |
    +--------------------+
    | INFORMATION_SCHEMA |
    | METRICS_SCHEMA     |
    | PERFORMANCE_SCHEMA |
    | analytics          |
    | io_replicate       |
    | mysql              |
    | test               |
    +--------------------+
    7 rows in set (0.00 sec)

    mysql> USE ANALYTICS;
    mysql> SHOW TABLES;
    +---------------------+
    | Tables_in_analytics |
    +---------------------+
    | raw_customers       |
    | raw_orders          |
    | raw_payments        |
    +---------------------+
    3 rows in set (0.00 sec)

    mysql> SELECT * FROM raw_customers LIMIT 10;
    +------+------------+-----------+
    | id   | first_name | last_name |
    +------+------------+-----------+
    |    1 | Michael    | P.        |
    |    2 | Shawn      | M.        |
    |    3 | Kathleen   | P.        |
    |    4 | Jimmy      | C.        |
    |    5 | Katherine  | R.        |
    |    6 | Sarah      | R.        |
    |    7 | Martin     | M.        |
    |    8 | Frank      | R.        |
    |    9 | Jennifer   | F.        |
    |   10 | Henry      | W.        |
    +------+------------+-----------+
    10 rows in set (0.10 sec)
    ```

## ステップ5: データ変換

これで、設定されたプロジェクトを実行し、データ変換を完了する準備ができました。

1. `dbt`プロジェクトを実行してデータ変換を完了します:

    ```shell
    dbt run
    ```

    以下は出力の例です:

    ```shell
    Running with dbt=1.0.1
    Found 5 models, 20 tests, 0 snapshots, 0 analyses, 170 macros, 0 operations, 3 seed files, 0 sources, 0 exposures, 0 metrics

    Concurrency: 1 threads (target='dev')

    1 of 5 START view model analytics.stg_customers................................. [RUN]
    1 of 5 OK created view model analytics.stg_customers............................ [SUCCESS 0 in 0.31s]
    2 of 5 START view model analytics.stg_orders.................................... [RUN]
    2 of 5 OK created view model analytics.stg_orders............................... [SUCCESS 0 in 0.23s]
    3 of 5 START view model analytics.stg_payments.................................. [RUN]
    3 of 5 OK created view model analytics.stg_payments............................. [SUCCESS 0 in 0.29s]
    4 of 5 START table model analytics.customers.................................... [RUN]
    4 of 5 OK created table model analytics.customers............................... [SUCCESS 0 in 0.76s]
    5 of 5 START table model analytics.orders....................................... [RUN]
    5 of 5 OK created table model analytics.orders.................................. [SUCCESS 0 in 0.63s]

    Finished running 3 view models, 2 table models in 2.27s.

    Completed successfully

    Done. PASS=5 WARN=0 ERROR=0 SKIP=0 TOTAL=5
    ```

    この結果では、2つのテーブル(`analytics.customers`および`analytics.orders`)と3つのビュー(`analytics.stg_customers`、`analytics.stg_orders`、`analytics.stg_payments`)が正常に作成されたことが示されています。

2. 変換が成功したかどうかを確認するためにTiDB Cloudに移動します。

    ```sql
    mysql> USE ANALYTICS;
    mysql> SHOW TABLES;
    +---------------------+
    | Tables_in_analytics |
    +---------------------+
    | customers           |
    | orders              |
    | raw_customers       |
    | raw_orders          |
    | raw_payments        |
    | stg_customers       |
    | stg_orders          |
    | stg_payments        |
    +---------------------+
    8 rows in set (0.00 sec)

    mysql> SELECT * FROM customers LIMIT 10;
    +-------------+------------+-----------+-------------+-------------------+------------------+-------------------------+
    | customer_id | first_name | last_name | first_order | most_recent_order | number_of_orders | customer_lifetime_value |
    +-------------+------------+-----------+-------------+-------------------+------------------+-------------------------+
    |           1 | Michael    | P.        | 2018-01-01  | 2018-02-10        |                2 |                 33.0000 |
    |           2 | Shawn      | M.        | 2018-01-11  | 2018-01-11        |                1 |                 23.0000 |
    |           3 | Kathleen   | P.        | 2018-01-02  | 2018-03-11        |                3 |                 65.0000 |
    |           4 | Jimmy      | C.        | NULL        | NULL              |             NULL |                    NULL |
    |           5 | Katherine  | R.        | NULL        | NULL              |             NULL |                    NULL |
    |           6 | Sarah      | R.        | 2018-02-19  | 2018-02-19        |                1 |                  8.0000 |
    |           7 | Martin     | M.        | 2018-01-14  | 2018-01-14        |                1 |                 26.0000 |
    |           8 | Frank      | R.        | 2018-01-29  | 2018-03-12        |                2 |                 45.0000 |
    |           9 | Jennifer   | F.        | 2018-03-17  | 2018-03-17        |                1 |                 30.0000 |
    |          10 | Henry      | W.        | NULL        | NULL              |             NULL |                    NULL |
    +-------------+------------+-----------+-------------+-------------------+------------------+-------------------------+
    10 rows in set (0.00 sec)
    ```

    出力から、さらに5つのテーブルやビューが追加され、テーブルやビューのデータが変換されたことが示されています。この例では、顧客テーブルからの一部のデータのみが表示されています。

## ステップ6: ドキュメントを生成する

`dbt`を使用すると、プロジェクトの全体構造を表示し、すべてのテーブルやビューを記述する視覚的なドキュメントを生成できます。

視覚的なドキュメントを生成するには、次の手順を実行します:

1. ドキュメントを生成する:

    ```shell
    dbt docs generate
    ```

2. サーバーを起動する:

    ```shell
    dbt docs serve
    ```

3. ブラウザからドキュメントにアクセスするには、<http://localhost:8080>に移動します。

## プロファイルフィールドの説明

| オプション           | 説明                                                             | 必須? | 例                                           |
|------------------|-------------------------------------------------------------------------|-----------|---------------------------------------------------|
| `type`             | 使用する特定のアダプタ                                             | 必須  | `tidb`                                            |
| `server`           | 接続するTiDB Cloudクラスターのエンドポイント                         | 必須  | `gateway01.ap-southeast-1.prod.aws.tidbcloud.com` |
| `port`             | 使用するポート                                                         | 必須  | `4000`                                            |
| `schema`           | データを正規化するスキーマ (データベース)                      | 必須  | `analytics`                                       |
| `username`         | TiDB Cloudクラスターに接続するためのユーザー名               | 必須  | `xxxxxxxxxxx.root`                                |
| `password`         | TiDB Cloudクラスターに認証するためのパスワード       | 必須  | `"your_password"`                                 |
| `retries`          | TiDB Cloudクラスターへの接続の再試行回数 (デフォルトは1)    | オプション  | `2`                                               |

## サポートされる関数

`dbt-tidb`では、以下の関数を直接使用できます。それらの使用方法についての詳細については、[dbt-util](https://github.com/dbt-labs/dbt-utils)を参照してください。

以下の関数がサポートされています:

- `bool_or`
- `cast_bool_to_text`
- `dateadd`
- `datediff`。`datediff`は`dbt-util`とは少し異なり、切り捨てではなく切り下げを行います。
- `date_trunc`
- `hash`
- `safe_cast`
- `split_part`
- `last_day`
- `cast_bool_to_text`
- `concat`
- `escape_single_quotes`
- `except`
- `intersect`
- `length`
- `position`
- `replace`
- `right`