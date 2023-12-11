---
title: LOAD DATA | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースのLOAD DATAの使用法の概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-load-data/','/docs/dev/reference/sql/statements/load-data/','/tidb/dev/sql-statement-operate-load-data-job','/tidb/dev/sql-statement-show-load-data']
---

# LOAD DATA

`LOAD DATA`ステートメントは、TiDBテーブルにデータをバッチで読み込みます。

TiDB v7.0.0から、`LOAD DATA` SQLステートメントは以下の機能をサポートしています。

- S3およびGCSからのデータのインポート
- 新しい`FIELDS DEFINED NULL BY`パラメータの追加

> **警告:**
>
> 新しい`FIELDS DEFINED NULL BY`パラメータおよびS3およびGCSからのデータのインポートは実験的な機能です。本番環境で使用をお勧めしません。この機能は事前の通知なしに変更または削除される可能性があります。バグが見つかった場合は、GitHubで[issue](https://github.com/pingcap/tidb/issues)を報告してください。

## 概要

```ebnf+diagram
LoadDataStmt ::=
    'LOAD' 'DATA' LocalOpt 'INFILE' stringLit DuplicateOpt 'INTO' 'TABLE' TableName CharsetOpt Fields Lines IgnoreLines ColumnNameOrUserVarListOptWithBrackets LoadDataSetSpecOpt

LocalOpt ::= ('LOCAL')?

Fields ::=
    ('TERMINATED' 'BY' stringLit
    | ('OPTIONALLY')? 'ENCLOSED' 'BY' stringLit
    | 'ESCAPED' 'BY' stringLit
    | 'DEFINED' 'NULL' 'BY' stringLit ('OPTIONALLY' 'ENCLOSED')?)?
```

## パラメータ

### `LOCAL`

`LOCAL`を使用して、クライアント上のデータファイルの場所を指定できます。ファイルパラメータは、クライアント上のファイルシステムパスである必要があります。

TiDB Cloudを使用している場合、`LOAD DATA`ステートメントを使用してローカルのデータファイルを読み込むには、TiDB Cloudに接続する際に接続文字列に`--local-infile`オプションを追加する必要があります。

- TiDB Serverlessの接続文字列の例：

    ```
    mysql --connect-timeout 15 -u '<user_name>' -h <host_name> -P 4000 -D test --ssl-mode=VERIFY_IDENTITY --ssl-ca=/etc/ssl/cert.pem -p<your_password> --local-infile
    ```

- TiDB Dedicatedの接続文字列の例：

    ```
    mysql --connect-timeout 15 --ssl-mode=VERIFY_IDENTITY --ssl-ca=<CA_path> --tls-version="TLSv1.2" -u root -h <host_name> -P 4000 -D test -p<your_password> --local-infile
    ```

### S3およびGCSストレージ

<CustomContent platform="tidb">

`LOCAL`を指定しない場合、ファイルパラメータは[外部ストレージ](/br/backup-and-restore-storages.md)で詳細に説明されているように有効なS3またはGCSパスである必要があります。

</CustomContent>

<CustomContent platform="tidb-cloud">

`LOCAL`を指定しない場合、ファイルパラメータは[外部ストレージ](https://docs.pingcap.com/tidb/stable/backup-and-restore-storages)で詳細に説明されているように有効なS3またはGCSパスである必要があります。

</CustomContent>

データファイルがS3またはGCSに格納されている場合、個々のファイルをインポートするか、ワイルドカード文字`*`を使用して複数のファイルを一括インポートすることができます。ワイルドカードはサブディレクトリ内のファイルを再帰的に処理しません。以下はいくつかの例です：

- 単一のファイルをインポート：`s3://<bucket-name>/path/to/data/foo.csv`
- 指定されたパス内のすべてのファイルをインポート：`s3://<bucket-name>/path/to/data/*`
- 指定されたパス内で`.csv`で終わるすべてのファイルをインポート：`s3://<bucket-name>/path/to/data/*.csv`
- 指定されたパス内で`foo`で始まるすべてのファイルをインポート：`s3://<bucket-name>/path/to/data/foo*`
- 指定されたパス内で`foo`で始まり`.csv`で終わるすべてのファイルをインポート：`s3://<bucket-name>/path/to/data/foo*.csv`

### `Fields`、`Lines`および`Ignore Lines`

`Fields`および`Lines`パラメータを使用して、データ形式の処理方法を指定できます。

- `FIELDS TERMINATED BY`：データの区切り文字を指定します。
- `FIELDS ENCLOSED BY`：データの囲み文字を指定します。
- `LINES TERMINATED BY`：行の終端文字を指定します。特定の文字で行を終了させたい場合に使用します。

`DEFINED NULL BY`を使用して、データファイル内でNULL値がどのように表されるかを指定できます。

- MySQLの動作と一貫して、`ESCAPED BY`が`null`でない場合、つまりデフォルト値`\`が使用される場合、`\N`はNULL値と見なされます。
- `DEFINED NULL BY`を使用する場合、例えば、`DEFINED NULL BY 'my-null'`のように使用すると、`my-null`はNULL値と見なされます。
- `DEFINED NULL BY ... OPTIONALLY ENCLOSED`を使用する場合、例えば `DEFINED NULL BY 'my-null' OPTIONALLY ENCLOSED`のように使用すると、`my-null`および`"my-null"`（`ENCLOSED BY '"'`を想定）はNULL値と見なされます。
- `DEFINED NULL BY`または`DEFINED NULL BY ... OPTIONALLY ENCLOSED`を使用せずに`ENCLOSED BY`を使用する場合、例えば`ENCLOSED BY '"'`のように使用すると、`NULL`がNULL値と見なされます。この動作はMySQLと一貫しています。
- それ以外の場合、NULL値と見なされません。

以下のデータ形式を例に取ると：

```
"bob","20","street 1"\r\n
"alice","33","street 1"\r\n
```

`bob`、`20`、`street 1`を抽出する場合、フィールドの区切り文字を`','`、囲み文字を`'\"'`と指定します：

```sql
FIELDS TERMINATED BY ',' ENCLOSED BY '\"' LINES TERMINATED BY '\r\n'
```
上記のパラメータを指定しない場合、インポートされたデータはデフォルトで以下のように処理されます：

```sql
FIELDS TERMINATED BY '\t' ENCLOSED BY '' ESCAPED BY '\\'
LINES TERMINATED BY '\n' STARTING BY ''
```

また、`IGNORE <number> LINES`パラメータを構成することで、ファイルの最初の`number`行を無視できます。たとえば、`IGNORE 1 LINES`を構成した場合、ファイルの最初の行は無視されます。

## 例

以下の例では、`LOAD DATA`を使用してデータをインポートしています。カンマはフィールドの区切り文字として指定されています。データを囲む二重引用符は無視されます。ファイルの最初の行は無視されます。

<CustomContent platform="tidb">

`ERROR 1148 (42000): the used command is not allowed with this TiDB version`が表示される場合は、トラブルシューティングのために[ERROR 1148 (42000): the used command is not allowed with this TiDB version](/error-codes.md#mysql-native-error-messages)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

`ERROR 1148 (42000): the used command is not allowed with this TiDB version`が表示される場合は、トラブルシューティングのために[ERROR 1148 (42000): the used command is not allowed with this TiDB version](https://docs.pingcap.com/tidb/stable/error-codes#mysql-native-error-messages)を参照してください。

</CustomContent>

```sql
LOAD DATA LOCAL INFILE '/mnt/evo970/data-sets/bikeshare-data/2017Q4-capitalbikeshare-tripdata.csv' INTO TABLE trips FIELDS TERMINATED BY ',' ENCLOSED BY '\"' LINES TERMINATED BY '\r\n' IGNORE 1 LINES (duration, start_date, end_date, start_station_number, start_station, end_station_number, end_station, bike_number, member_type);
```

```sql
Query OK, 815264 rows affected (39.63 sec)
Records: 815264  Deleted: 0  Skipped: 0  Warnings: 0
```

`LOAD DATA`は、`FIELDS ENCLOSED BY`および`FIELDS TERMINATED BY`のパラメータとして16進数ASCII文字表現またはバイナリASCII文字表現を使用することもサポートしています。次の例を参照してください。

```sql
LOAD DATA LOCAL INFILE '/mnt/evo970/data-sets/bikeshare-data/2017Q4-capitalbikeshare-tripdata.csv' INTO TABLE trips FIELDS TERMINATED BY x'2c' ENCLOSED BY b'100010' LINES TERMINATED BY '\r\n' IGNORE 1 LINES (duration, start_date, end_date, start_station_number, start_station, end_station_number, end_station, bike_number, member_type);
```

上記の例では、`x'2c'`は`,`文字の16進数表現であり、`b'100010'`は`"`文字のバイナリ表現です。

## MySQL互換性

`LOAD DATA`ステートメントの構文はMySQLと互換性がありますが、文字セットオプションは解析されますが無視されます。構文の互換性の違いがある場合は、[バグを報告](https://docs.pingcap.com/tidb/stable/support)できます。
<CustomContent platform="tidb">

> **注意:**
>
> - TiDB v4.0.0より前のバージョンでは、`LOAD DATA`は20000行ごとにコミットされます。
> - TiDB v4.0.0からv6.6.0までのバージョンでは、TiDBはデフォルトで1つのトランザクションですべての行をコミットします。
> - TiDB v4.0.0またはそれ以前のバージョンからのアップグレード後には、`ERROR 8004 (HY000) at line 1: Transaction is too large, size: 100000058`というエラーが発生する可能性があります。このエラーを解決する推奨される方法は、`tidb.toml`ファイルの中の[`txn-total-size-limit`](/tidb-configuration-file.md#txn-total-size-limit)の値を増やすことです。この制限を増やすことができない場合は、[`tidb_dml_batch_size`](/system-variables.md#tidb_dml_batch_size)を`20000`に設定してアップグレード前の動作を復元することもできます。v7.0.0以降、`tidb_dml_batch_size`は`LOAD DATA`ステートメントには影響しません。
> - トランザクション内でどれだけの行がコミットされたとしても、`LOAD DATA`ステートメントは明示的なトランザクションによる[`ROLLBACK`](/sql-statements/sql-statement-rollback.md)ステートメントでロールバックされません。
> - `LOAD DATA`ステートメントは、TiDBのトランザクションモード構成に関係なく、常に楽観的トランザクションモードで実行されます。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **注意:**
>
> - TiDB v4.0.0より前のバージョンでは、`LOAD DATA`は20000行ごとにコミットされます。
> - TiDB v4.0.0からv6.6.0までのバージョンでは、TiDBはデフォルトで1つのトランザクションですべての行をコミットします。
> - TiDB v7.0.0からは、`LOAD DATA`ステートメントの`WITH batch_size=<number>`パラメータによってコミットされる行数が制御されます。デフォルトは1回のコミットあたり1000行です。
> - TiDB v4.0.0またはそれ以前のバージョンからのアップグレード後には、[`tidb_dml_batch_size`](/system-variables.md#tidb_dml_batch_size)を`20000`に設定することでアップグレード前の動作を復元できます。
> - トランザクション内でどれだけの行がコミットされたとしても、`LOAD DATA`ステートメントは明示的なトランザクションによる[`ROLLBACK`](/sql-statements/sql-statement-rollback.md)ステートメントでロールバックされません。
> - `LOAD DATA`ステートメントは、TiDBのトランザクションモード構成に関係なく、常に楽観的トランザクションモードで実行されます。

</CustomContent>

## 関連項目

<CustomContent platform="tidb">

* [INSERT](/sql-statements/sql-statement-insert.md)
* [TiDB楽観的トランザクションモデル](/optimistic-transaction.md)
* [TiDB悲観的トランザクションモード](/pessimistic-transaction.md)

</CustomContent>

<CustomContent platform="tidb-cloud">

* [INSERT](/sql-statements/sql-statement-insert.md)
* [TiDB楽観的トランザクションモデル](/optimistic-transaction.md)
* [TiDB悲観的トランザクションモード](/pessimistic-transaction.md)

</CustomContent>