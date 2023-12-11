---
title: TiDBクラウドへのスナップショットファイルのインポート
summary: Amazon AuroraまたはRDS for MySQLのスナップショットファイルをTiDB Cloudにインポートする方法について学びます。

# TiDBクラウドへのスナップショットファイルのインポート

Amazon AuroraまたはRDS for MySQLのスナップショットファイルをTiDBクラウドにインポートすることができます。`{db_name}.{table_name}/`フォルダ内の`.parquet`接尾辞を持つすべてのソースデータファイルは、[命名規則](/tidb-cloud/naming-conventions-for-data-import.md)に準拠している必要があります。

スナップショットファイルのインポートプロセスは、Parquetファイルのインポートと類似しています。詳細については、[Amazon S3またはGCSからTiDBクラウドにApache Parquetファイルをインポート](/tidb-cloud/import-parquet-files.md)を参照してください。