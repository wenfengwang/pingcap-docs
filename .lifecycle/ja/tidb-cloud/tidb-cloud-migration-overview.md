---
title: TiDBクラウドの移行とインポートの概要
summary: TiDBクラウド向けのデータ移行とインポートシナリオの概要について学びます。
aliases: ['/tidbcloud/export-data-from-tidb-cloud']
---

# TiDBクラウドの移行とインポートの概要

あなたはTiDBクラウドにさまざまなデータソースからデータを移行できます。このドキュメントでは、データ移行のシナリオの概要について説明します。

## MySQL互換データベースからのデータ移行

MySQL互換データベースからのデータ移行では、フルデータ移行と増分データ移行を実行できます。移行のシナリオと方法は次のとおりです。

- Data Migrationを使用したMySQL互換データベースの移行

    TiDBはMySQLと高い互換性があります。TiDBクラウドコンソールでData Migrationを使用して、任意のMySQL互換データベースからデータをスムーズにTiDBクラウドに移行できます。詳細については、[Data Migrationを使用したMySQL互換データベースのTiDB Cloudへの移行](/tidb-cloud/migrate-from-mysql-using-data-migration.md)を参照してください。

- AWS DMSを使用した移行

    PostgreSQL、Oracle、SQL Serverなどの異種データベースをTiDBクラウドに移行する場合は、AWS Database Migration Service (AWS DMS) を使用することをお勧めします。

    - [AWS DMSを使用してMySQL互換データベースをTiDBクラウドに移行する方法](/tidb-cloud/migrate-from-mysql-using-aws-dms.md)
    - [AWS DMSを使用してAmazon RDS for Oracleからの移行方法](/tidb-cloud/migrate-from-oracle-using-aws-dms.md)

- MySQLシャードの移行とマージ

    アプリケーションがデータストレージにMySQLシャードを使用している場合、これらのシャードをTiDBクラウドに1つのテーブルとして移行できます。詳細については、[大規模データセットのMySQLシャードの移行とマージをTiDB Cloudに移行](/tidb-cloud/migrate-sql-shards.md)を参照してください。

- TiDBセルフホストからの移行

    DumplingとTiCDCを使用して、TiDBセルフホストクラスタからTiDBクラウド（AWS）にデータを移行できます。詳細については、[TiDBセルフホストからTiDBクラウドに移行](/tidb-cloud/migrate-from-op-tidb.md)を参照してください。

## ファイルからTiDBクラウドにデータをインポート

SQL、CSV、Parquet、またはAuroraスナップショット形式のデータファイルがある場合、これらのファイルを一括でTiDBクラウドにインポートできます。インポートのシナリオと方法は次のとおりです。

- ローカルのCSVファイルをTiDBクラウドにインポート

    ローカルのCSVファイルをTiDBクラウドにインポートできます。詳細については、[TiDBクラウドへのローカルファイルのインポート](/tidb-cloud/tidb-cloud-import-local-files.md)を参照してください。

- サンプルデータ（SQLファイル）をTiDBクラウドにインポート

    サンプルデータ（SQLファイル）をTiDBクラウドにインポートして、TiDBクラウドのインターフェースとインポートプロセスにすばやく慣れることができます。詳細については、[TiDBクラウドにサンプルデータをインポート](/tidb-cloud/import-sample-data.md)を参照してください。

- Amazon S3またはGCSからCSVファイルをTiDBクラウドにインポート

    Amazon S3またはGCSからCSVファイルをTiDBクラウドにインポートできます。詳細については、[Amazon S3またはGCSからTiDBクラウドにCSVファイルをインポート](/tidb-cloud/import-csv-files.md)を参照してください。

- Amazon S3またはGCSからApache ParquetファイルをTiDBクラウドにインポート

    Amazon S3またはGCSからApache ParquetファイルをTiDBクラウドにインポートできます。詳細については、[Amazon S3またはGCSからTiDBクラウドにApache Parquetファイルをインポート](/tidb-cloud/import-parquet-files.md)を参照してください。

## 参照

### Amazon S3アクセスおよびGCSアクセスの構成

ソースデータがAmazon S3またはGoogle Cloud Storage（GCS）バケットに格納されている場合、TiDBクラウドにデータをインポートまたは移行する前に、バケットへのアクセスを構成する必要があります。詳細については、[Amazon S3アクセスおよびGCSアクセスの構成](/tidb-cloud/config-s3-and-gcs-access.md)を参照してください。

### データインポートのための命名規則

データが正常にインポートされるようにするには、命名規則に準拠したスキーマファイルとデータファイルを準備する必要があります。詳細については、[データインポートのための命名規則](/tidb-cloud/naming-conventions-for-data-import.md)を参照してください。

### Amazon S3からデータをインポート時のアクセス拒否エラーの解決

Amazon S3からTiDBクラウドにデータをインポートする際に発生するアクセス拒否エラーを解決することができます。詳細については、[Amazon S3からのデータインポート時のアクセス拒否エラーのトラブルシューティング](/tidb-cloud/troubleshoot-import-access-denied-error.md)を参照してください。