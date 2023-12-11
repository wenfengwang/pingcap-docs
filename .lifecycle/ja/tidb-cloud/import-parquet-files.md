---
title: Amazon S3やGCSからApache ParquetファイルをTiDB Cloudにインポートする
summary: Amazon S3やGCSからApache ParquetファイルをTiDB Cloudにインポートする方法について学びます。

# Amazon S3やGCSからApache ParquetファイルをTiDB Cloudにインポートする

TiDB Cloudでは、圧縮されていないものとSnappy圧縮された[Apache Parquet](https://parquet.apache.org/)形式のデータファイルの両方をインポートできます。このドキュメントでは、Amazon Simple Storage Service（Amazon S3）またはGoogle Cloud Storage（GCS）からParquetファイルをTiDB Cloudにインポートする方法について説明します。

> **注意:**
>
> - TiDB Cloudでは空のテーブルにのみParquetファイルをインポートできます。既にデータが含まれている既存のテーブルにデータをインポートする場合は、このドキュメントに従って一時的な空のテーブルにデータをインポートし、その後`INSERT SELECT`ステートメントを使用してデータを対象の既存テーブルにコピーできます。
> - TiDB専用クラスタにチェンジフィードがある場合、クラスタにデータをインポートできません（**データのインポート**ボタンが無効になります）、現在の**物理インポートモード**を使用するためです。このモードでは、インポートされたデータは変更ログを生成しないため、チェンジフィードでインポートされたデータを検出できません。
> - TiDB専用クラスタのみが、GCSからParquetファイルをインポートすることをサポートしています。

## ステップ1. Parquetファイルの準備

> **注意:**
>
> 現在、TiDB Cloudは以下のデータ型を含むParquetファイルのインポートをサポートしていません。インポートするParquetファイルにこれらのデータ型が含まれる場合は、まずサポートされているデータ型（たとえば`STRING`）を使用してParquetファイルを再生成する必要があります。あるいは、AWS Glueなどのサービスを使用してデータ型を簡単に変換することもできます。

> - `LIST`
> - `NEST STRUCT`
> - `BOOL`
> - `ARRAY`
> - `MAP`

1. Parquetファイルが256 MBを超える場合、約256 MBのファイルに分割することを検討してください。

    TiDB Cloudは非常に大きなParquetファイルをインポートできますが、約256 MBの複数の入力ファイルが最適です。これは、TiDB Cloudが複数のファイルを並列で処理できるためで、インポート速度が大幅に向上することがあります。

2. Parquetファイルの名前を以下のように付けます。

    - Parquetファイルに1つのテーブル全体のデータが含まれている場合、ファイルの名前を`${db_name}.${table_name}.parquet`形式で付けます。これにより、データをインポートするときに`${db_name}.${table_name}`テーブルにマッピングされます。
    - 1つのテーブルのデータが複数のParquetファイルに分かれている場合、これらのParquetファイルに数字の接尾辞を追加します。たとえば、`${db_name}.${table_name}.000001.parquet`およ`${db_name}.${table_name}.000002.parquet`のようにします。数字の接尾辞は連続している必要はありませんが、昇順である必要があります。また、すべての接尾辞が同じ長さであることを確認するために、数字の前に余分なゼロを追加する必要があります。

    > **注意:**
    >
    > 一部のケースで（たとえば、Parquetファイルリンクを他のプログラムで使用している場合）、前述の規則に従ってParquetファイル名を更新できない場合は、ファイル名を変更せずに **マッピング設定** を使用してソースデータを単一のターゲットテーブルにインポートすることができます。

## ステップ2. ターゲットテーブルのスキーマを作成する

Parquetファイルはスキーマ情報を含まないため、TiDB CloudにParquetファイルからデータをインポートする前に、以下の方法のいずれかを使用してテーブルスキーマを作成する必要があります。

- 方法1: TiDB Cloudでソースデータのターゲットデータベースとテーブルを作成します。

- 方法2: ParquetファイルがあるAmazon S3またはGCSディレクトリで、以下の方法でソースデータのターゲットテーブルスキーマファイルを作成します:

    1. ソースデータのためのデータベーススキーマファイルを作成します。

        Parquetファイルが[ステップ1](#step-1-prepare-the-parquet-files)の命名規則に従っている場合、データベーススキーマファイルはデータインポートにはオプションです。それ以外の場合、データベーススキーマファイルは必須です。

        各データベーススキーマファイルは`${db_name}-schema-create.sql`形式でなければならず、`CREATE DATABASE`DDLステートメントを含まなければなりません。このファイルによって、TiDB Cloudはデータをインポートするときに`${db_name}`データベースを作成します。

        たとえば、次のステートメントが含まれる`mydb-scehma-create.sql`ファイルを作成した場合、TiDB Cloudはデータをインポートするときに`mydb`データベースを作成します。

        ```sql
        CREATE DATABASE mydb;
        ```

    2. ソースデータのためのテーブルスキーマファイルを作成します。

        Amazon S3またはGCSディレクトリにテーブルスキーマファイルが含まれていない場合、TiDB Cloudはデータをインポートするときに対応するテーブルを作成しません。

        各テーブルスキーマファイルは`${db_name}.${table_name}-schema.sql`形式でなければならず、`CREATE TABLE`DDLステートメントを含まなければなりません。このファイルによって、TiDB Cloudはデータをインポートするときに`${db_name}`データベース内の`${db_table}`テーブルを作成します。

        たとえば、次のステートメントが含まれる`mydb.mytable-schema.sql`ファイルを作成した場合、TiDB Cloudはデータをインポートするときに`mydb`データベース内の`mytable`テーブルを作成します。

        ```sql
        CREATE TABLE mytable (
        ID INT,
        REGION VARCHAR(20),
        COUNT INT );
        ```

        > **注意:**
        >
        > 各`${db_name}.${table_name}-schema.sql`ファイルは単一のDDLステートメントのみを含んでいる必要があります。ファイルに複数のDDLステートメントが含まれている場合は、最初のステートメントのみが有効です。

## ステップ3. クロスアカウントアクセスを構成する

TiDB CloudがAmazon S3またはGCSバケットのParquetファイルにアクセスできるようにするには、次のいずれかを行います:

- ParquetファイルがAmazon S3にある場合、[Amazon S3アクセスを構成](/tidb-cloud/config-s3-and-gcs-access.md#configure-amazon-s3-access)します。

    AWSアクセスキーまたはRole ARNを使用してバケットにアクセスできます。完了したら、アクセスキー（アクセスキーIDとシークレットアクセスキーを含む）またはRole ARNの値をメモしておく必要があります（[ステップ4](#step-4-import-parquet-files-to-tidb-cloud)で使用します）。

- ParquetファイルがGCSにある場合、[GCSアクセス](/tidb-cloud/config-s3-and-gcs-access.md#configure-gcs-access)を構成します。

## ステップ4. TiDB CloudにParquetファイルをインポートする

ParquetファイルをTiDB Cloudにインポートするためには、以下の手順を実行します:

1. ターゲットクラスタの**インポート**ページを開きます。

    1. [TiDB Cloudコンソール](https://tidbcloud.com/)にログインして、プロジェクトの[**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動します。

        > **Tip:**
        >
        > 複数のプロジェクトを持っている場合は、左下の`<MDSvgIcon name="icon-left-projects" />`をクリックして別のプロジェクトに切り替えることができます。

    2. ターゲットクラスタの概要ページに移動し、左のナビゲーションペインで **インポート** をクリックします。

2. **インポート**ページで:
   - TiDB Dedicatedクラスタの場合、右上隅の**データのインポート**をクリックします。
   - TiDB Serverlessクラスタの場合、アップロードエリア上の**S3からデータをインポート**リンクをクリックします。

3. ソースParquetファイルの次の情報を提供します:

    - **ロケーション**: **Amazon S3**を選択します。
    - **データ形式**: **Parquet**を選択します。
    - **バケットURI**: ParquetファイルがあるバケットURIを選択します。URIの末尾に`/`を付ける必要があることに注意してください。たとえば、 `s3://sampledate/ingest/`です。
    - **バケットアクセス**（このフィールドはAWS S3の場合だけ表示されます）: AWSアクセスキーまたはRole ARNを使用してバケットにアクセスすることができます。詳細については[Amazon S3アクセスの構成](/tidb-cloud/config-s3-and-gcs-access.md#configure-amazon-s3-access)を参照してください。
        - **AWSアクセスキー**: AWSアクセスキーIDおよびAWSシークレットアクセスキーを入力します。
        - **AWS Role ARN**: Role ARNの値を入力します。

4. **事前作成済みのテーブルにインポート**または**S3からスキーマおよびデータをインポート**を選択できます。

    - **事前作成済みのテーブルにインポート**を選択すると、事前にTiDBでテーブルを作成し、データをインポートするテーブルを選択できます。この場合、最大1000のテーブルをインポートできます。テーブルを作成するには、左のナビゲーションペインで**Chat2Qury**をクリックします。Chat2Quryの使用方法についての詳細は、[AIパワードのChat2Queryでデータを管理](/tidb-cloud/explore-data-with-chat2query.md)を参照してください。
- **S3 からスキーマとデータをインポート** では、S3 に保存されているテーブルを作成するための SQL スクリプトをインポートし、それに対応するテーブルデータを TiDB にインポートできます。

5. ソースファイルが命名規則に適合していない場合は、単一の対象テーブルと CSV ファイルの間のカスタムマッピングルールを指定できます。その後、提供されたカスタムマッピングルールを使用して、データソースファイルが再スキャンされます。マッピングを変更するには、**詳細設定** をクリックしてから **マッピング設定** をクリックします。 **マッピング設定** は **事前に作成されたテーブルにインポート** を選択した場合のみ使用できます。

    - **対象データベース**: 選択した対象データベースの名前を入力します。

    - **対象テーブル**: 選択した対象テーブルの名前を入力します。このフィールドは特定のテーブル名のみを受け入れるため、ワイルドカードはサポートされていません。

    - **ソースファイルの URI と名前**: 次の形式でソースファイルの URI と名前を入力してください。 `s3://[bucket_name]/[data_source_folder]/[file_name].parquet`。例：`s3://sampledate/ingest/TableName.01.parquet`。ソースファイルの一致させるためにワイルドカードを使用することもできます。例：

        - `s3://[bucket_name]/[data_source_folder]/my-data?.parquet`: このフォルダ内の `my-data` で始まり1文字（たとえば `my-data1.parquet` および `my-data2.parquet`）を含むすべての Parquet ファイルが同じ対象テーブルにインポートされます。
        - `s3://[bucket_name]/[data_source_folder]/my-data*.parquet`: `my-data` で始まるフォルダ内のすべての Parquet ファイルが同じ対象テーブルにインポートされます。

      注意：`?` と `*` のみがサポートされています。

        > **注意:**
        >
        > URI にはデータソースフォルダが含まれている必要があります。

6. **インポートの開始** をクリックします。

7. インポートの進行状況が **完了** になったら、インポートされたテーブルを確認します。

インポートタスクを実行すると、TiDB Cloud がサポートされていない変換や無効な変換を検出した場合、インポートジョブを自動的に終了し、インポートエラーを報告します。

インポートエラーが発生した場合は、次のようにします。

1. 部分的にインポートされたテーブルを削除します。
2. テーブルスキーマファイルを確認します。エラーがあればテーブルスキーマファイルを修正します。
3. Parquet ファイルのデータ型を確認します。

    Parquet ファイルにサポートされていないデータ型（たとえば `NEST STRUCT`、`ARRAY`、`MAP`）が含まれている場合は、Parquet ファイルを [サポートされているデータ型](#supported-data-types)（たとえば `STRING`）を使用して再生成する必要があります。

4. インポートタスクをもう一度実行してください。

## サポートされているデータ型

次の表に、TiDB Cloud にインポートできるサポートされている Parquet データ型がリストされています。

| Parquet 原始タイプ | Parquet 論理タイプ | TiDB または MySQL のタイプ |
|---|---|---|
| DOUBLE | DOUBLE | DOUBLE<br />FLOAT |
| FIXED_LEN_BYTE_ARRAY(9) | DECIMAL(20,0) | BIGINT UNSIGNED |
| FIXED_LEN_BYTE_ARRAY(N) | DECIMAL(p,s) | DECIMAL<br />NUMERIC |
| INT32 | DECIMAL(p,s) | DECIMAL<br />NUMERIC |
| INT32 | N/A | INT<br />MEDIUMINT<br />YEAR |
| INT64 | DECIMAL(p,s) | DECIMAL<br />NUMERIC |
| INT64 | N/A | BIGINT<br />INT UNSIGNED<br />MEDIUMINT UNSIGNED |
| INT64 | TIMESTAMP_MICROS | DATETIME<br />TIMESTAMP |
| BYTE_ARRAY | N/A | BINARY<br />BIT<br />BLOB<br />CHAR<br />LINESTRING<br />LONGBLOB<br />MEDIUMBLOB<br />MULTILINESTRING<br />TINYBLOB<br />VARBINARY |
| BYTE_ARRAY | STRING | ENUM<br />DATE<br />DECIMAL<br />GEOMETRY<br />GEOMETRYCOLLECTION<br />JSON<br />LONGTEXT<br />MEDIUMTEXT<br />MULTIPOINT<br />MULTIPOLYGON<br />NUMERIC<br />POINT<br />POLYGON<br />SET<br />TEXT<br />TIME<br />TINYTEXT<br />VARCHAR |
| SMALLINT | N/A | INT32 |
| SMALLINT UNSIGNED | N/A | INT32 |
| TINYINT | N/A | INT32 |
| TINYINT UNSIGNED | N/A | INT32 |

## トラブルシューティング

### データインポート中の警告の解決

**インポートの開始** をクリックした後、`対応するソースファイルが見つかりません` などの警告メッセージが表示される場合は、正しいソースファイルを提供したり、既存のファイルを [データインポートの命名規則](/tidb-cloud/naming-conventions-for-data-import.md) に従って名前を変更したり、変更を行うために **詳細設定** を使用してこれを解決してください。

これらの問題を解決した後、データを再度インポートする必要があります。

### インポートされたテーブルにゼロ行がある

インポートの進行状況が **完了** になったら、インポートされたテーブルを確認してください。行数がゼロの場合、入力したバケット URI に一致するデータファイルがないことを意味します。この場合は、正しいソースファイルを提供したり、既存のファイルを [データインポートの命名規則](/tidb-cloud/naming-conventions-for-data-import.md) に従って名前を変更したり、変更を行うために **詳細設定** を使用してこれを解決してください。その後、これらのテーブルを再度インポートしてください。