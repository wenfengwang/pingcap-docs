---
title: RESTORE | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースのRESTOREの使用方法の概要。
aliases: ['/docs/dev/sql-statements/sql-statement-restore/']
---

# RESTORE

このステートメントは、以前に[`BACKUP`ステートメント](/sql-statements/sql-statement-backup.md)で生成されたバックアップアーカイブから分散リストアを実行します。

> **警告:**
>
> - この機能は実験的なものです。本番環境では使用しないでください。この機能は変更される可能性があり、事前の通知なしに削除されることがあります。バグを見つけた場合は、GitHubの[issue](https://github.com/pingcap/tidb/issues)で報告できます。
> - この機能は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスターでは利用できません。

`RESTORE`ステートメントは、リストアプロセスがTiDB自体によって駆動される点を除いて、[BRツール](https://docs.pingcap.com/tidb/stable/backup-and-restore-overview)と同じエンジンを使用します。BRの利点と注意点はここでも適用されます。特に、**`RESTORE`は現在ACIDコンプライアンスではありません**。`RESTORE`を実行する前に、次の要件が満たされていることを確認してください。

* クラスターは「オフライン」であり、現在のTiDBセッションが、リストアされるすべてのテーブルにアクセスするための唯一のアクティブなSQL接続であること。
* 完全なリストアを実行している場合、既存のテーブルがすでに存在しないことを確認してください。既存のデータが上書きされ、データとインデックスの不整合を引き起こす可能性があります。
* 増分リストアを実行している場合、バックアップが作成された`LAST_BACKUP`のタイムスタンプ時点でテーブルがまさに同じ状態であることを確認してください。

`RESTORE`を実行するには、`RESTORE_ADMIN`または`SUPER`権限のどちらかが必要です。また、リストアを実行しているTiDBノードとクラスター内のすべてのTiKVノードが、宛先からの読み取り権限を持っている必要があります。

`RESTORE`ステートメントはブロッキングされ、リストアタスクが完了、失敗、またはキャンセルされるまでにのみ終了します。`RESTORE`を実行するためには、長時間の接続が用意されている必要があります。タスクは[`KILL TIDB QUERY`](/sql-statements/sql-statement-kill.md)ステートメントを使用してキャンセルできます。

`BACKUP`および`RESTORE`タスクは同時に1つだけ実行できます。もし同じTiDBサーバーで既に`BACKUP`または`RESTORE`タスクが実行中であれば、新しい`RESTORE`実行は、以前のすべてのタスクが完了するまで待機します。

`RESTORE`は「tikv」ストレージエンジンでのみ使用できます。「unistore」エンジンで`RESTORE`を使用すると失敗します。

## 構文

```ebnf+diagram
RestoreStmt ::=
    "RESTORE" BRIETables "FROM" stringLit RestoreOption*

BRIETables ::=
    "DATABASE" ( '*' | DBName (',' DBName)* )
|   "TABLE" TableNameList

RestoreOption ::=
    "RATE_LIMIT" '='? LengthNum "MB" '/' "SECOND"
|   "CONCURRENCY" '='? LengthNum
|   "CHECKSUM" '='? Boolean
|   "SEND_CREDENTIALS_TO_TIKV" '='? Boolean

Boolean ::=
    NUM | "TRUE" | "FALSE"
```

## 例

### バックアップアーカイブからのリストア

{{< copyable "sql" >}}

```sql
RESTORE DATABASE * FROM 'local:///mnt/backup/2020/04/';
```

```sql
+------------------------------+-----------+----------+---------------------+---------------------+
| Destination                  | Size      | BackupTS | Queue Time          | Execution Time      |
+------------------------------+-----------+----------+---------------------+---------------------+
| local:///mnt/backup/2020/04/ | 248665063 | 0        | 2020-04-21 17:16:55 | 2020-04-21 17:16:55 |
+------------------------------+-----------+----------+---------------------+---------------------+
1 row in set (28.961 sec)
```

上記の例では、すべてのデータがローカルファイルシステムのバックアップアーカイブからリストアされます。データは`/mnt/backup/2020/04/`ディレクトリからSSTファイルとして読み込まれ、すべてのTiDBおよびTiKVノードに分散されます。

上記の結果の最初の行は、次のように説明されています。

| カラム | 説明 |
| :-------- | :--------- |
| `Destination` | データを読み込む宛先URL |
| `Size` | バックアップアーカイブの合計サイズ（バイト単位） |
| `BackupTS` |（使用されていません） |
| `Queue Time` | `RESTORE`タスクがキューに入れられたタイムスタンプ（現在のタイムゾーンで） |
| `Execution Time` | `RESTORE`タスクが実行を開始したタイムスタンプ（現在のタイムゾーンで） |

### 部分的なリストア

リストアするデータベースまたはテーブルを指定できます。バックアップアーカイブから一部のデータベースやテーブルが欠落している場合、それらは無視され、したがって`RESTORE`は何もせずに完了します。

{{< copyable "sql" >}}

```sql
RESTORE DATABASE `test` FROM 'local:///mnt/backup/2020/04/';
```

{{< copyable "sql" >}}

```sql
RESTORE TABLE `test`.`sbtest01`, `test`.`sbtest02` FROM 'local:///mnt/backup/2020/04/';
```

### 外部ストレージ

BRは、S3またはGCSからデータをリストアすることをサポートしています。

{{< copyable "sql" >}}

```sql
RESTORE DATABASE * FROM 's3://example-bucket-2020/backup-05/';
```

<CustomContent platform="tidb">

外部ストレージサービスのURIフォーマットについては、[URI Formats of External Storage Services](/external-storage-uri.md)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

外部ストレージのURIフォーマットについては、[external storage URI](https://docs.pingcap.com/tidb/stable/external-storage-uri)を参照してください。

</CustomContent>

資格情報が配布されてはならないクラウド環境で実行する場合は、`SEND_CREDENTIALS_TO_TIKV`オプションを`FALSE`に設定してください。

{{< copyable "sql" >}}

```sql
RESTORE DATABASE * FROM 's3://example-bucket-2020/backup-05/'
    SEND_CREDENTIALS_TO_TIKV = FALSE;
```

### パフォーマンスの微調整

`RATE_LIMIT`を使用して、TiKVノードごとの平均ダウンロード速度を制限して、ネットワーク帯域幅を減らすことができます。

デフォルトでは、TiDBノードは128個のリストアスレッドを実行します。この値は`CONCURRENCY`オプションで調整できます。

リストアが完了する前には、`RESTORE`はアーカイブからのデータに対するチェックサムを実行して、正当性を検証します。このステップは不要であると確信している場合は、`CHECKSUM`オプションで無効にできます。

{{< copyable "sql" >}}

```sql
RESTORE DATABASE * FROM 's3://example-bucket-2020/backup-06/'
    RATE_LIMIT = 120 MB/SECOND
    CONCURRENCY = 64
    CHECKSUM = FALSE;
```

### 増分リストア

増分リストアを実行するための特別な構文はありません。TiDBはバックアップアーカイブが完全なものか増分のものかを認識し、適切なアクションを取ります。正しい順序で各増分リストアを適用するだけです。

例えば、次のようにバックアップタスクが作成された場合：

{{< copyable "sql" >}}

```sql
BACKUP DATABASE `test` TO 's3://example-bucket/full-backup'  SNAPSHOT = 413612900352000;
BACKUP DATABASE `test` TO 's3://example-bucket/inc-backup-1' SNAPSHOT = 414971854848000 LAST_BACKUP = 413612900352000;
BACKUP DATABASE `test` TO 's3://example-bucket/inc-backup-2' SNAPSHOT = 416353458585600 LAST_BACKUP = 414971854848000;
```

それぞれの増分リストアを次のように適用する必要があります：

{{< copyable "sql" >}}

```sql
RESTORE DATABASE * FROM 's3://example-bucket/full-backup';
RESTORE DATABASE * FROM 's3://example-bucket/inc-backup-1';
RESTORE DATABASE * FROM 's3://example-bucket/inc-backup-2';
```

## MySQL互換性

このステートメントは、MySQLの構文のTiDB拡張です。

## 関連情報

* [BACKUP](/sql-statements/sql-statement-backup.md)
* [SHOW RESTORES](/sql-statements/sql-statement-show-backups.md)