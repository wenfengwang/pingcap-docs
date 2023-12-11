---
title: ParquetファイルからTiDBへのデータ移行
summary: ParquetファイルからTiDBへのデータ移行方法について学びます。

# ParquetファイルからTiDBへのデータ移行

このドキュメントでは、Apache HiveからParquetファイルを生成し、TiDB Lightningを使用してParquetファイルからTiDBにデータを移行する方法について説明します。

Amazon AuroraからParquetファイルをエクスポートする場合は、[Amazon AuroraからTiDBへのデータ移行](/migrate-aurora-to-tidb.md)を参照してください。

## 前提条件

- [TiUPを使用してTiDB Lightningをインストール](/migration-tools.md)します。
- TiDB Lightningに必要な対象データベースの特権を取得します ([TiDB Lightningの特権要件](/tidb-lightning/tidb-lightning-faq.md#what-are-the-privilege-requirements-for-the-target-database)を参照)。

## ステップ1. Parquetファイルの準備

このセクションでは、TiDB Lightningで読み取ることができるHiveからParquetファイルをエクスポートする方法について説明します。

Hiveの各テーブルは、`STORED AS PARQUET LOCATION '/path/in/hdfs'` を注釈付けしてParquetファイルにエクスポートできます。したがって、`test`という名前のテーブルをエクスポートする必要がある場合は、次の手順を実行してください：

1. Hiveで次のSQLステートメントを実行します：

    ```sql
    CREATE TABLE temp STORED AS PARQUET L

OCATION '/path/in/hdfs' AS SELECT * FROM test;
    ```

    上記のステートメントを実行した後、テーブルデータはHDFSシステムに正常にエクスポートされます。

2. `hdfs dfs -get`コマンドを使用してParquetファイルをローカルファイルシステムにエクスポートします：

    ```shell
    hdfs dfs -get /path/in/hdfs /path/in/local
    ```

    エクスポートが完了したら、HDFSでエクスポートしたParquetファイルを削除する必要がある場合は、一時テーブル (`temp`) を直接削除できます：

    ```sql
    DROP TABLE temp;
    ```

3. HiveからエクスポートされたParquetファイルには`.parquet`サフィックスが付いていない可能性があり、TiDB Lightningによって正しく識別されないことがあります。したがって、ファイルをインポートする前にエクスポートしたファイルの名前を変更し、`.parquet`サフィックスを追加する必要があります。

4. すべてのParquetファイルを統一されたディレクトリに配置します。たとえば、`/data/my_datasource/`または`s3://my-bucket/sql-backup`です。TiDB Lightningは、このディレクトリおよびそのサブディレクトリ内のすべての`.parquet`ファイルを再帰的に検索します。

## ステップ2. 対象テーブルスキーマの作成

ParquetファイルからTiDBにデータをインポートする前に、対象テーブルスキーマを作成する必要があります。対象テーブルスキーマを作成する方法は、次の2つの方法のいずれかを使用できます：

* **方法1**：TiDB Lightningを使用して対象テーブルスキーマを作成します。

    必要なDDLステートメントを含むSQLファイルを作成します：

    - `${db_name}-schema-create.sql` ファイルに `CREATE DATABASE` ステートメントを追加します。
    - `${db_name}.${table_name}-schema.sql` ファイルに `CREATE TABLE` ステートメントを追加します。

* **方法2**：対象テーブルスキーマを手動で作成します。

## ステップ3. 構成ファイルの作成

次の内容で `tidb-lightning.toml` ファイルを作成します：

```toml
[lightning]
# ログ
level = "info"
file = "tidb-lightning.log"

[tikv-importer]
# "local": デフォルトバックエンド。大量のデータ（1 TiB以上）をインポートする場合は、ローカルバックエンドを推奨します。インポート中、対象のTiDBクラスタはサービスを提供できません。
backend = "local"
# "tidb": "tidb"バックエンドは、1 TiB未満のデータをインポートすることを推奨します。インポート中、対象のTiDBクラスタは通常サービスを提供できます。
# インポートモードについての詳細情報は、<https://docs.pingcap.com/tidb/stable/tidb-lightning-overview#tidb-lightning-architecture> を参照してください。
# ソートされたKey-Valueファイルの一時保存ディレクトリを設定します。ディレクトリは空でなければならず、ストレージスペースはインポートするデータセットのサイズよりも大きくなければなりません。インポートパフォーマンスを向上させるためには、`data-source-dir`とは異なるディレクトリを使用し、I/Oを排他的に使用できるフラッシュストレージを使用することをお勧めします。
sorted-kv-dir = "${sorted-kv-dir}"

[mydumper]
# データソースのディレクトリ。
data-source-dir = "${data-path}" # ローカルパスまたはS3パス。例：'s3://my-bucket/sql-backup'。

[tidb]
# 対象クラスタ。
host = ${host}            # 例：172.16.32.1
port = ${port}            # 例：4000
user = "${user_name}"     # 例： "root"
password = "${password}"  # 例： "rootroot"
status-port = ${status-port} # インポート中、TiDB LightningはTiDBのステータスポートからテーブルスキーマ情報を取得する必要があります。例：10080
pd-addr = "${ip}:${port}" # PDクラスタのアドレス。例：172.16.31.3:2379。TiDB LightningはPDからいくつかの情報を取得します。backend = "local"の場合は、status-portとpd-addrを正しく指定する必要があります。さもなければ、インポートが異常になります。
```

構成ファイルの詳細については、[TiDB Lightningの構成](/tidb-lightning/tidb-lightning-configuration.md)を参照してください。

## ステップ4. データのインポート

1. `tidb-lightning`を実行します。

    - Amazon S3からデータをインポートする場合は、TiDB Lightningを実行する前に、S3バックエンドストレージにアクセスする権限を持つアカウントのSecretKeyとAccessKeyを環境変数として設定する必要があります。

        ```shell
        export AWS_ACCESS_KEY_ID=${access_key}
        export AWS_SECRET_ACCESS_KEY=${secret_key}
        ```

        上記の方法に加えて、TiDB Lightningは`~/.aws/credentials`から認証情報ファイルを読み取ることもサポートしています。

    - コマンドラインでプログラムを起動する場合、プロセスは`SIGHUP`シグナルを受信した後、予期せず終了することがあります。この場合は、プログラムを`nohup`や`screen`ツールを使用して実行することをお勧めします。たとえば：

        ```shell
        nohup tiup tidb-lightning -config tidb-lightning.toml > nohup.out 2>&1 &
        ```

2. インポートが開始した後、次のいずれかの方法でインポートの進捗状況を確認できます：

    - `grep`を使用してログ内のキーワード`progress`を検索します。デフォルトで進捗が5分ごとに更新されます。
    - [モニタリングダッシュボード](/tidb-lightning/monitor-tidb-lightning.md)で進捗状況を確認します。
    - [TiDB Lightningウェブインターフェース](/tidb-lightning/tidb-lightning-web-interface.md)で進捗状況を確認します。

    TiDB Lightningがインポートを完了すると、自動的に終了します。

3. インポートが成功したかどうかを確認します。

    `tidb-lightning.log`に最終行に `the whole procedure completed` が含まれているかどうかを確認します。含まれている場合は、インポートに成功しています。含まれていない場合は、インポートでエラーが発生しています。エラーメッセージに従ってエラーを解決してください。

    > **注意：**
    >
    > インポートが成功したかどうかに関係なく、ログの最終行に `tidb lightning exit` が表示されます。これはTiDB Lightningが正常に終了したことを意味しますが、インポートが成功したことを必ずしも意味しません。

インポートが失敗した場合は、トラブルシューティングのために[TiDB Lightning FAQ](/tidb-lightning/tidb-lightning-faq.md)を参照してください。