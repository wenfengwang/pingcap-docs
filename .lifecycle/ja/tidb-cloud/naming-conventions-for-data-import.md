---
title: データインポートの命名規則
summary: CSV、Parquet、Auroraスナップショット、およびSQLファイルのデータインポート時の命名規則について学びます。
---

# データインポートの命名規則

TiDB Cloudには、次のファイル形式でデータをインポートできます: CSV、Parquet、Auroraスナップショット、およびSQL。データが正常にインポートされるようにするには、次の2種類のファイルを準備する必要があります。

- **スキーマファイル**。データベーススキーマファイル（オプション）とテーブルスキーマファイルの両方をSQL形式（.sql）で準備します。テーブルスキーマファイルが提供されない場合は、対象のデータベースで対応するテーブルを事前に手動で作成する必要があります。
- **データファイル**。データをインポートするための命名規則に準拠したデータファイルを準備します。データファイルの名前が要件を満たさない場合は、[**ファイルパターン**](#file-pattern)を使用してインポートタスクを実行することをお勧めします。それ以外の場合、インポートタスクはインポートしたいデータファイルをスキャンすることができません。

## スキーマファイルの命名規則

このセクションでは、データベースおよびテーブルのスキーマファイルの命名規則について説明します。スキーマファイルの命名規則は、次のすべてのソースファイルタイプ（CSV、Parquet、Auroraスナップショット、SQL）について同じです。

スキーマファイルの命名規則は以下の通りです。

- データベーススキーマファイル（オプション）: `${db_name}-schema-create.sql`
- テーブルスキーマファイル: `${db_name}.${table_name}-schema.sql`

以下はデータベーススキーマファイルの例です:

- 名前: `import_db-schema-create.sql`
- ファイル内容:

    ```sql
    CREATE DATABASE import_db;
    ```

以下はテーブルスキーマファイルの例です:

- 名前: `import_db.test_table-schema.sql`
- ファイル内容:

    ```sql
    CREATE TABLE test_table (
        id INTEGER PRIMARY KEY,
        val VARCHAR(255)
    );
    ```

## データファイルの命名規則

このセクションでは、データファイルの命名規則について説明します。ソースファイルのタイプに応じて、データファイルの命名規則が異なります。

### CSV

CSVファイルをインポートする場合は、データファイルを次のように命名します:

`${db_name}.${table_name}${suffix}.csv.${compress}`

`${suffix}`はオプションであり、次の形式のいずれかにできます。ここで *`xxx`* は任意の数字です:

- *`.xxx`*、例: `.01`
- *`._xxx_xxx_xxx`*、例: `._0_0_01`
- *`_xxx_xxx_xxx`*、例: `_0_0_01`

`${compress}`は圧縮形式であり、オプションです。TiDB Cloudは以下の形式をサポートしています: `.gzip`、`.gz`、`.zstd`、`.zst`および`.snappy`。

例として、すべての以下のファイルの対象データベースとテーブルは `import_db` および `test_table` です:

- `import_db.test_table.csv`
- `import_db.test_table.01.csv`
- `import_db.test_table._0_0_01.csv`
- `import_db.test_table_0_0_01.csv`
- `import_db.test_table_0_0_01.csv.gz`

### Parquet

Parquetファイルをインポートする場合は、データファイルを次のように命名します:

`${db_name}.${table_name}${suffix}.parquet` （`${suffix}`はオプション）

例:

- `import_db.test_table.parquet`
- `import_db.test_table.01.parquet`

### Auroraスナップショット

Auroraスナップショットファイルでは、`${db_name}.${table_name}/` フォルダ内の`.parquet`サフィックスを持つすべてのファイルが命名規則に準拠しています。データファイル名には、"a-z, 0-9, - , _ , ."から構成される任意の接頭辞と「.parquet」サフィックスを含めることができます。

例:

- `import_db.test_table/mydata.parquet`
- `import_db.test_table/part001/mydata.parquet`
- `import_db.test_table/part002/mydata-part002.parquet`

### SQL

SQLファイルをインポートする場合は、データファイルを次のように命名します:

`${db_name}.${table_name}${suffix}.sql.${compress}`

`${suffix}`はオプションであり、次の形式のいずれかにできます。ここで *`xxx`* は任意の数字です:

- *`.xxx`*、例: `.01`
- *`._xxx_xxx_xxx`*、例: `._0_0_01`
- *`_xxx_xxx_xxx`*、例: `_0_0_01`

`${compress}`は圧縮形式であり、オプションです。TiDB Cloudは以下の形式をサポートしています: `.gzip`、`.gz`、`.zstd`、`.zst`および`.snappy`。

例:

- `import_db.test_table.sql`
- `import_db.test_table.01.sql`
- `import_db.test_table.01.sql.gz`

SQLファイルがデフォルト設定でTiDB Dumplingを介してエクスポートされた場合、デフォルトで命名規則に準拠しています。

## ファイルパターン

CSVまたはParquetのソースデータファイルが命名規則に準拠していない場合は、ファイルパターン機能を使用してソースデータファイルとターゲットテーブルの名前マッピング関係を確立することができます。この機能はAuroraスナップショットおよびSQLデータファイルをサポートしていません。

- CSVファイルの場合、「ステップ4. TiDB CloudへのCSVファイルのインポート」の**ファイルパターン**を参照してください（/tidb-cloud/import-csv-files.md#step-4-import-csv-files-to-tidb-cloud）
- Parquetファイルの場合、「ステップ4. TiDB CloudへのParquetファイルのインポート」の**ファイルパターン**を参照してください（/tidb-cloud/import-parquet-files.md#step-4-import-parquet-files-to-tidb-cloud）