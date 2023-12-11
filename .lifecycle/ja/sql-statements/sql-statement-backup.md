---
title: バックアップ | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースのバックアップの使用方法の概要。
aliases: ['/docs/dev/sql-statements/sql-statement-backup/']
---

# バックアップ

このステートメントはTiDBクラスターの分散バックアップを実行するために使用されます。

> **警告:**
>
> - この機能は実験的なものです。本番環境で使用しないことをお勧めします。この機能は予告なく変更または削除される可能性があります。バグを見つけた場合は、GitHubの[issue](https://github.com/pingcap/tidb/issues)に報告できます。
> - この機能は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスターでは利用できません。

`BACKUP`ステートメントは、[BR ツール](https://docs.pingcap.com/tidb/stable/backup-and-restore-overview)と同じエンジンを使用しますが、バックアッププロセスはTiDB自体によって、別個のBRツールではなく駆動されます。 BRのすべての利点と警告はこのステートメントにも適用されます。

`BACKUP`を実行するには、`BACKUP_ADMIN`または`SUPER`権限が必要です。また、バックアップを実行するTiDBノードとクラスター内のすべてのTiKVノードは、宛先に対して読み取りまたは書き込みの権限を持っている必要があります。 [Security Enhanced Mode](/system-variables.md#tidb_enable_enhanced_security)が有効になっている場合は、ローカルストレージ（`local://`で始まるストレージパス）は許可されていません。

`BACKUP`ステートメントは、バックアップタスク全体が終了、失敗、またはキャンセルされるまでブロックされます。 `BACKUP`を実行するためには、持続的な接続が準備されている必要があります。タスクは[`KILL TIDB QUERY`](/sql-statements/sql-statement-kill.md)ステートメントを使用してキャンセルできます。

`BACKUP`または[`RESTORE`](/sql-statements/sql-statement-restore.md)タスクは同時に1つだけ実行できます。同じTiDBサーバーですでに`BACKUP`または`RESTORE`ステートメントが実行されている場合は、新しい`BACKUP`実行は前のすべてのタスクが終了するまで待機します。

`BACKUP`は "tikv" ストレージエンジンとのみ使用できます。 "unistore"エンジンとの使用は失敗します。

## 概要

```ebnf+diagram
BackupStmt ::=
    "BACKUP" BRIETables "TO" stringLit BackupOption*

BRIETables ::=
    "DATABASE" ( '*' | DBName (',' DBName)* )
|   "TABLE" TableNameList

BackupOption ::=
    "RATE_LIMIT" '='? LengthNum "MB" '/' "SECOND"
|   "CONCURRENCY" '='? LengthNum
|   "CHECKSUM" '='? Boolean
|   "SEND_CREDENTIALS_TO_TIKV" '='? Boolean
|   "LAST_BACKUP" '='? BackupTSO
|   "SNAPSHOT" '='? ( BackupTSO | LengthNum TimestampUnit "AGO" )

Boolean ::=
    NUM | "TRUE" | "FALSE"

BackupTSO ::=
    LengthNum | stringLit
```

## 例

### データベースのバックアップ

{{< copyable "sql" >}}

```sql
BACKUP DATABASE `test` TO 'local:///mnt/backup/2020/04/';
```

```sql
+------------------------------+-----------+-----------------+---------------------+---------------------+
| Destination                  | Size      | BackupTS        | Queue Time          | Execution Time      |
+------------------------------+-----------+-----------------+---------------------+---------------------+
| local:///mnt/backup/2020/04/ | 248665063 | 416099531454472 | 2020-04-12 23:09:48 | 2020-04-12 23:09:48 |
+------------------------------+-----------+-----------------+---------------------+---------------------+
1 row in set (58.453 sec)
```

上記の例では、`test`データベースがローカルファイルシステムにバックアップされます。データは、`/mnt/backup/2020/04/`ディレクトリにSSTファイルとして保存され、すべてのTiDBおよびTiKVノードに分散されます。

上記の結果の1行目は次のように説明されます。

| カラム | 説明 |
| :-------- | :--------- |
| `Destination` | 宛先のURL |
| `Size` | バックアップアーカイブの合計サイズ（バイト単位） |
| `BackupTS` | バックアップが作成されたスナップショットのTSO（[増分バックアップ](#incremental-backup)に有用） |
| `Queue Time` | `BACKUP`タスクがキューイングされたタイムスタンプ（現在のタイムゾーンで） |
| `Execution Time` | `BACKUP`タスクが実行を開始したタイムスタンプ（現在のタイムゾーンで） |

### テーブルのバックアップ

{{< copyable "sql" >}}

```sql
BACKUP TABLE `test`.`sbtest01` TO 'local:///mnt/backup/sbtest01/';
```

{{< copyable "sql" >}}

```sql
BACKUP TABLE sbtest02, sbtest03, sbtest04 TO 'local:///mnt/backup/sbtest/';
```

### クラスタ全体のバックアップ

{{< copyable "sql" >}}

```sql
BACKUP DATABASE * TO 'local:///mnt/backup/full/';
```

システムテーブル（`mysql.*`、`INFORMATION_SCHEMA.*`、`PERFORMANCE_SCHEMA.*`、…）はバックアップに含まれません。

### 外部ストレージ

BRはS3やGCSにデータをバックアップすることをサポートしています。

{{< copyable "sql" >}}

```sql
BACKUP DATABASE `test` TO 's3://example-bucket-2020/backup-05/?access-key={YOUR_ACCESS_KEY}&secret-access-key={YOUR_SECRET_KEY}';
```

<CustomContent platform="tidb">

URIの構文については、[外部ストレージサービスのURI形式](/external-storage-uri.md)で詳しく説明されています。

</CustomContent>

<CustomContent platform="tidb-cloud">

URIの構文については、[external storage URI](https://docs.pingcap.com/tidb/stable/external-storage-uri)で詳しく説明されています。

</CustomContent>

クレデンシャルを配布すべきでないクラウド環境で実行する場合は、`SEND_CREDENTIALS_TO_TIKV`オプションを`FALSE`に設定してください。

{{< copyable "sql" >}}

```sql
BACKUP DATABASE `test` TO 's3://example-bucket-2020/backup-05/'
    SEND_CREDENTIALS_TO_TIKV = FALSE;
```

### パフォーマンスの微調整

平均アップロード速度をTiKVノードあたりに制限するために`RATE_LIMIT`を使用します。

デフォルトでは、各TiKVノードは4つのバックアップスレッドを実行します。この値は`CONCURRENCY`オプションで調整できます。

バックアップが完了する前に、データに対してクラスタでチェックサムを実行して、正当性を検証します。不要であると確信している場合は、このステップを`CHECKSUM`オプションで無効にできます。

{{< copyable "sql" >}}

```sql
BACKUP DATABASE `test` TO 's3://example-bucket-2020/backup-06/'
    RATE_LIMIT = 120 MB/SECOND
    CONCURRENCY = 8
    CHECKSUM = FALSE;
```

### スナップショット

タイムスタンプ、TSO、または相対時間を指定して、過去のデータをバックアップします。

{{< copyable "sql" >}}

```sql
-- 相対時間
BACKUP DATABASE `test` TO 'local:///mnt/backup/hist01'
    SNAPSHOT = 36 HOUR AGO;

-- タイムスタンプ（現在のタイムゾーンで）
BACKUP DATABASE `test` TO 'local:///mnt/backup/hist02'
    SNAPSHOT = '2020-04-01 12:00:00';

-- タイムスタンプ（Oracle）
BACKUP DATABASE `test` TO 'local:///mnt/backup/hist03'
    SNAPSHOT = 415685305958400;
```

相対時間には以下の単位がサポートされています：

* MICROSECOND
* SECOND
* MINUTE
* HOUR
* DAY
* WEEK

SQL標準に従って、単位は常に単数形であることに注意してください。

### 増分バックアップ

`LAST_BACKUP`オプションを指定して、最後のバックアップから現在のスナップショットまでの変更のみをバックアップします。

{{< copyable "sql" >}}

```sql
-- タイムスタンプ（現在のタイムゾーンで）
BACKUP DATABASE `test` TO 'local:///mnt/backup/hist02'
    LAST_BACKUP = '2020-04-01 12:00:00';

-- タイムスタンプ（Oracle）
BACKUP DATABASE `test` TO 'local:///mnt/backup/hist03'
    LAST_BACKUP = 415685305958400;
```

## MySQL互換性

このステートメントはMySQL構文に対するTiDBの拡張です。

## 関連項目

* [RESTORE](/sql-statements/sql-statement-restore.md)
* [SHOW BACKUPS](/sql-statements/sql-statement-show-backups.md)