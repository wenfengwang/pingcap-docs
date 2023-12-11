---
title: インポート
summary: TiDBでのIMPORT INTOの使用概要。
---

# IMPORT INTO

`IMPORT INTO`ステートメントは、TiDB Lightningの[Physical Import Mode](/tidb-lightning/tidb-lightning-physical-import-mode.md)を介して、TiDBの空のテーブルに`CSV`、`SQL`、`PARQUET`などの形式でデータをインポートするために使用されます。

> **注意:**
>
> このステートメントはTiDB Self-Hostedにのみ適用され、[TiDB Cloud](https://docs.pingcap.com/tidbcloud/)では使用できません。

`IMPORT INTO`は、Amazon S3、GCS、およびTiDBローカルストレージに保存されたファイルからデータをインポートすることをサポートしています。

- Amazon S3、GCS、Azure Blob Storageに保存されたデータファイルについて、`IMPORT INTO`は[TiDBバックエンドタスク分散実行フレームワーク](/tidb-distributed-execution-framework.md)で実行をサポートしています。

    - このフレームワークが有効になっている場合（[tidb_enable_dist_task](/system-variables.md#tidb_enable_dist_task-new-in-v710)が `ON` ）、 `IMPORT INTO`はデータインポートジョブを複数のサブジョブに分割し、これらのサブジョブを異なるTiDBノードに分散して実行し、インポート効率を向上させます。
    - このフレームワークが無効になっている場合、`IMPORT INTO`は現在のユーザが接続しているTiDBノードでのみ実行をサポートしています。

- TiDBにローカルに保存されたデータファイルについては、`IMPORT INTO`は、現在のユーザが接続しているTiDBノードでのみ実行をサポートしています。そのため、データファイルをTiDBにローカルに保存する必要があります。プロキシやロードバランサーを介してTiDBにアクセスする場合は、TiDBにローカルに保存されたデータファイルをインポートすることはできません。

## 制限事項

- 現在、`IMPORT INTO`は10 TiB以内のデータのインポートをサポートしています。
- `IMPORT INTO`はデータベース内の既存の空のテーブルにのみデータのインポートをサポートしています。
- `IMPORT INTO`はトランザクションやロールバックをサポートしていません。明示的なトランザクション（`BEGIN`/`END`）内で`IMPORT INTO`を実行すると、エラーが返されます。
- `IMPORT INTO`の実行は、インポートが完了するまで現在の接続をブロックします。ステートメントを非同期で実行するには、`DETACHED`オプションを追加できます。
- `IMPORT INTO`は、[バックアップ＆リストア](/br/backup-and-restore-overview.md)、[`FLASHBACK CLUSTER TO TIMESTAMP`](/sql-statements/sql-statement-flashback-to-timestamp.md)、インデックスの追加の高速化[/system-variables.md#tidb_ddl_enable_fast_reorg-new-in-v630](/acceleration of adding indexes)、TiDB Lightningを使用したデータのインポート、TiCDCを使用したデータのレプリケーション、[PITR（Point-in-Time Recovery）](/br/br-log-architecture.md)などの機能と同時に動作することをサポートしていません。
- クラスタで同時に1つの`IMPORT INTO`ジョブしか実行できません。`IMPORT INTO`は実行中のジョブを事前にチェックしますが、これは厳格な制限ではありません。複数のインポートジョブを開始することができますが、それによってデータの不整合やインポートの失敗が発生する可能性があるため避ける必要があります。
- データのインポートプロセス中に、ターゲットテーブルでDDLまたはDML操作を行わないでください。また、ターゲットデータベースの[`FLASHBACK DATABASE`](/sql-statements/sql-statement-flashback-database.md)を実行しないでください。これらの操作は、インポートの失敗やデータの不整合を引き起こす可能性があります。さらに、インポートが完了するまで、読み取り操作を行うことは**推奨されません**。読み取りおよび書き込み操作は、インポートが完了した後にのみ実行してください。
- インポートプロセスはシステムリソースを大幅に消費します。より良いパフォーマンスを得るためには、少なくとも32コアと64 GiBのメモリを備えたTiDBノードを使用することを推奨します。インポート中にTiDBはデータをTiDB [一時ディレクトリ](/tidb-configuration-file.md#temp-dir-new-in-v630)にソートして書き込むため、フラッシュメモリなどの高性能ストレージメディアを構成することをお勧めします。詳細については、[Physical Import Mode limitations](/tidb-lightning/tidb-lightning-physical-import-mode.md#requirements-and-restrictions)を参照してください。
- TiDB [一時ディレクトリ](/tidb-configuration-file.md#temp-dir-new-in-v630)には、少なくとも90 GiBの利用可能なスペースが必要です。インポートするデータのボリュームと同じかそれ以上のストレージ容量を割り当てることを推奨します。
- 1つのインポートジョブは、1つのターゲットテーブルにのみデータをインポートできます。複数のターゲットテーブルにデータをインポートするには、ターゲットテーブルのインポートが完了した後に、次のターゲットテーブルのために新しいジョブを作成する必要があります。
- `IMPORT INTO`は、TiDBクラスタのアップグレード中はサポートされません。
- データインポートに[Global Sort](/tidb-global-sort.md)機能が使用される場合、エンコード後の単一行のデータサイズは32 MiBを超えてはなりません。
- Global Sort機能がデータインポートに使用される場合、インポートタスクが完了する前にターゲットTiDBクラスタが削除されると、グローバルソートに使用される一時データがAmazon S3に残る可能性があります。この場合は、S3ストレージコストの増加を避けるために、残留データを手動で削除する必要があります。
- インポートするデータには、プライマリキーまたはユニークインデックスの競合レコードが含まれていないことを確認してください。さもなければ、競合が発生し、インポートタスクが失敗する可能性があります。
-既知の問題: TiDBノード構成ファイルのPDアドレスがクラスタの現在のPDトポロジと一貫していない場合、「IMPORT INTO」タスクが失敗する可能性があります。この不一致は、以前にPDがスケールアウトされたが、TiDB構成ファイルがそれに応じて更新されなかった場合や、TiDBノードが構成ファイルの更新後に再起動されなかった場合などに発生する可能性があります。

## インポートの前提条件

データをインポートする前に、次の要件を満たしていることを確認してください：

- インポート対象のテーブルがTiDBで既に作成され、空であること。
- インポート対象のクラスタには、インポートするデータを格納する十分なスペースが確保されていること。
- 現在のセッションに接続しているTiDBノードの[一時ディレクトリ](/tidb-configuration-file.md#temp-dir-new-in-v630)には、少なくとも90 GiBの利用可能なスペースがあります。[`tidb_enable_dist_task`](/system-variables.md#tidb_enable_dist_task-new-in-v710)が有効になっている場合、クラスタ内の各TiDBノードの一時ディレクトリに十分なディスクスペースがあることも確認してください。

## 必要な権限

`IMPORT INTO`を実行するには、対象のテーブルに対して`SELECT`、`UPDATE`、`INSERT`、`DELETE`、`ALTER`権限が必要です。TiDBローカルストレージからファイルをインポートするには、`FILE`権限も必要です。

## 構文

```ebnf+diagram
ImportIntoStmt ::=
    'IMPORT' 'INTO' TableName ColumnNameOrUserVarList? SetClause? 'FROM' fileLocation Format? WithOptions?

ColumnNameOrUserVarList ::=
    '(' ColumnNameOrUserVar (',' ColumnNameOrUserVar)* ')'

SetClause ::=
    'SET' SetItem (',' SetItem)*

SetItem ::=
    ColumnName '=' Expr

Format ::=
    'CSV' | 'SQL' | 'PARQUET'

WithOptions ::=
    'WITH' OptionItem (',' OptionItem)*

OptionItem ::=
    optionName '=' optionVal | optionName
```

## パラメータの説明

### ColumnNameOrUserVarList

データファイル内の各フィールドがターゲットテーブルの列に対応するかどうかを指定します。これを使用して、特定のフィールドをスキップするための変数にフィールドをマップしたり、`SetClause`で使用することもできます。

- このパラメータが指定されていない場合、データファイルの各行のフィールド数は、ターゲットテーブルの列数と一致する必要があります。フィールドは、順番に対応する列にインポートされます。
- このパラメータが指定されている場合、指定された列または変数の数は、データファイルの各行のフィールド数と一致する必要があります。

### `SetClause`

インポート先の列の値がどのように計算されるかを指定します。 `SET`式の右側では、`ColumnNameOrUserVarList`で指定した変数を参照できます。

`SET`式の左側では、`ColumnNameOrUserVarList`に含まれていない列名のみを参照できます。ターゲット列名が既に`ColumnNameOrUserVarList`に存在する場合、`SET`式は無効です。

### `fileLocation`

データファイルの保存場所を指定します。これは、Amazon S3、GCS、またはAzure Blob Storage URIパス、またはTiDBのローカルファイルパスになります。

- Amazon S3、GCS、Azure Blob Storage URIパス：URIの構成の詳細については、[外部ストレージサービスのURI形式](/external-storage-uri.md)を参照してください。
- TiDBローカルファイルパス：絶対パスである必要があり、ファイルの拡張子は`.csv`、`.sql`、または`.parquet`である必要があります。このパスに対応するファイルが現在のユーザーによって接続されているTiDBノードに保存されており、ユーザーには`FILE`権限が必要です。

> **注意:**
>
> 対象のクラスタで[SEM](/system-variables.md#tidb_enable_enhanced_security)が有効になっている場合、 `fileLocation`はローカルファイルパスとして指定できません。

`fileLocation`パラメータでは、単一のファイルを指定するか、インポート用に複数のファイルを一致させるために `*` ワイルドカードを使用できます。ワイルドカードはファイル名のみに使用できます。ディレクトリには一致せず、サブディレクトリ内のファイルを再帰的に一致させません。Amazon S3に保存されたファイルの例を挙げると、パラメータを次のように構成できます。
- 単一のファイルをインポートします： `s3://<bucket-name>/path/to/data/foo.csv`
- 指定されたパス内のすべてのファイルをインポートします： `s3://<bucket-name>/path/to/data/*`
- 指定されたパス内で`.csv`のサフィックスを持つすべてのファイルをインポートします： `s3://<bucket-name>/path/to/data/*.csv`
- 指定されたパス内で`foo`の接頭辞を持つすべてのファイルをインポートします： `s3://<bucket-name>/path/to/data/foo*`
- 指定されたパス内で`foo`の接頭辞と`.csv`のサフィックスを持つすべてのファイルをインポートします： `s3://<bucket-name>/path/to/data/foo*.csv`

### フォーマット

`IMPORT INTO` ステートメントは、3つのデータファイルフォーマット、`CSV`、`SQL`、`PARQUET` をサポートしています。指定されていない場合、デフォルトのフォーマットは `CSV` です。

### WithOptions

`WithOptions` を使用して、インポートオプションを指定し、データインポートプロセスを制御することができます。たとえば、バックエンドで非同期にインポートを実行するために、`IMPORT INTO` ステートメントに `WITH DETACHED` オプションを追加してインポートを有効にすることができます。

サポートされているオプションは以下の通りです：

| オプション名 | サポートされているデータフォーマット | 説明 |
|:---|:---|:---|
| `CHARACTER_SET='<string>'` | CSV | データファイルの文字セットを指定します。デフォルトの文字セットは `utf8mb4` です。サポートされている文字セットには `binary`、`utf8`、`utf8mb4`、`gb18030`、`gbk`、`latin1`、`ascii` が含まれています。 |
| `FIELDS_TERMINATED_BY='<string>'` | CSV | フィールドのセパレータを指定します。デフォルトのセパレータは`,`です。 |
| `FIELDS_ENCLOSED_BY='<char>'` | CSV | フィールドのデリミタを指定します。デフォルトのデリミタは`"`です。 |
| `FIELDS_ESCAPED_BY='<char>'` | CSV | フィールドのエスケープ文字を指定します。デフォルトのエスケープ文字は`\`です。 |
| `FIELDS_DEFINED_NULL_BY='<string>'` | CSV | フィールド内で `NULL` を表す値を指定します。デフォルト値は`\N`です。 |
| `LINES_TERMINATED_BY='<string>'` | CSV | 行の終端記号を指定します。デフォルトでは、`IMPORT INTO` は自動的に`\n`、`\r`、または`\r\n`を行終端記号として識別します。行終端記号がこれらのいずれかである場合、このオプションを明示的に指定する必要はありません。 |
| `SKIP_ROWS=<number>` | CSV | スキップする行数を指定します。デフォルト値は`0`です。CSVファイルのヘッダーをスキップするためにこのオプションを使用できます。インポートのソースファイルをワイルドカードで指定する場合、このオプションは`fileLocation`でワイルドカードに一致するすべてのソースファイルに適用されます。 |
| `SPLIT_FILE` | CSV | 1つのCSVファイルを複数の約256 MiBの小さなチャンクに分割してパラレル処理を改善するために使用します。このパラメータは**非圧縮の**CSVファイルにのみ適用され、TiDB Lightning [`strict-format`](/tidb-lightning/tidb-lightning-data-source.md#strict-format)と同じ使用制限があります。 |
| `DISK_QUOTA='<string>'` | すべてのフォーマット | データソーティング中に使用できるディスクスペースのしきい値を指定します。デフォルト値は、TiDBの[temporary directory](/tidb-configuration-file.md#temp-dir-new-in-v630)内のディスクスペースの80%です。ディスク総容量を取得できない場合、デフォルト値は50 GiBです。明示的に`DISK_QUOTA`を指定する場合は、TiDBの一時ディレクトリ内のディスク容量の80%を超えないようにします。 |
| `DISABLE_TIKV_IMPORT_MODE` | すべてのフォーマット | インポートプロセス中にTiKVをインポートモードに切り替えることを無効にするかどうかを指定します。デフォルトでは、TiKVをインポートモードに切り替えることは無効になっています。クラスタ内で読み書き操作が進行中の場合、このオプションを有効にしてインポートプロセスの影響を回避することができます。 |
| `THREAD=<number>` | すべてのフォーマット | インポートの並列処理を指定します。デフォルト値はCPUコア数の50%で、最小値は1です。このオプションを明示的に指定してリソース使用量を制御することができますが、値がCPUコア数を超えないようにします。データを含まない新しいクラスタにデータをインポートする場合、インポート性能を改善するためにこの並列処理を適切に増やすことをおすすめします。既存のプロダクション環境で使用されているクラスタの場合、この並列処理をアプリケーションの要件に応じて調整することをおすすめします。 |
| `MAX_WRITE_SPEED='<string>'` | すべてのフォーマット | TiKVノードへの書き込み速度を制御します。デフォルトでは速度制限はありません。たとえば、このオプションを`1MiB`として指定すると、書き込み速度を1 MiB/sに制限することができます。 |
| `CHECKSUM_TABLE='<string>'` | すべてのフォーマット | インポート後にターゲットテーブルでのチェックサムチェックを実行するかどうかを構成します。サポートされる値には`"required"`（デフォルト）、`"optional"`、`"off"`が含まれます。`"required"`はインポート後にチェックサムチェックを実行します。チェックサムチェックが失敗した場合、TiDBはエラーを返し、インポートは終了します。`"optional"`はインポート後にチェックサムチェックを実行します。エラーが発生した場合、TiDBは警告を返し、エラーを無視します。`"off"`はインポート後にチェックサムチェックを実行しません。 |
| `DETACHED` | すべてのフォーマット | `IMPORT INTO` を非同期で実行するかどうかを制御します。このオプションを有効にすると、`IMPORT INTO` を即座にインポートジョブの情報（`Job_ID`など）を返し、ジョブはバックエンドで非同期に実行されます。 |
| `CLOUD_STORAGE_URI` | すべてのフォーマット | エンコードされたKVデータの[Global Sort](/tidb-global-sort.md)が格納されているターゲットアドレスを指定します。`CLOUD_STORAGE_URI`が指定されていない場合、`IMPORT INTO` はシステム変数[`tidb_cloud_storage_uri`](/system-variables.md#tidb_cloud_storage_uri-new-in-v740)の値に基づいてGlobal Sortを使用するかどうかを決定します。このシステム変数がターゲットストレージアドレスを指定している場合、`IMPORT INTO` はこのアドレスをGlobal Sortに使用します。`CLOUD_STORAGE_URI`に非空の値が指定されている場合、`IMPORT INTO` はその値をターゲットストレージアドレスとして使用します。`CLOUD_STORAGE_URI`に空の値が指定されている場合、ローカルソーティングが強制されます。現時点では、ターゲットストレージアドレスはS3のみをサポートしています。URIの構成の詳細については、[Amazon S3 URI format](/external-storage-uri.md#amazon-s3-uri-format)を参照してください。この機能を使用する場合、すべてのTiDBノードがターゲットS3バケットに対して読み書きアクセス権を持っている必要があります。 |

## 圧縮されたファイル

`IMPORT INTO` は、圧縮された `CSV` および `SQL` ファイルをインポートすることをサポートしています。ファイルが圧縮されているかどうか、および圧縮形式はファイルの拡張子に基づいて自動的に判別することができます：

| 拡張子 | 圧縮形式 |
|:---|:---|
| `.gz`、`.gzip` | gzip 圧縮形式 |
| `.zstd`、`.zst` | ZStd 圧縮形式 |
| `.snappy` | snappy 圧縮形式 |

> **注意:**
>
> Snappy圧縮形式のファイルは[公式のSnappy形式](https://github.com/google/snappy)である必要があります。他のバリアントのSnappy圧縮形式はサポートされていません。

## Global Sort

> **警告:**
>
> Global Sort 機能は実験的なものです。本番環境で使用することはお勧めしません。

`IMPORT INTO` は、ソースデータファイルのデータインポートジョブを複数のサブジョブに分割し、それぞれのサブジョブが独自にデータをエンコードしてソートし、インポートする前にインポートします。これらのサブジョブのエンコードされたKV範囲に重複があると、インポートのパフォーマンスと安定性が低下することにつながるため、KVの圧縮を続行する必要があります。

次のシナリオでは、KV範囲で重複が発生することがあります:

- 各サブジョブに割り当てられたデータファイル内の行にオーバーラップする主キー範囲がある場合、各サブジョブのエンコードによって生成されたデータKVも重複します。
    - `IMPORT INTO` は、通常はファイル名の辞書順によって並べ替えられたデータファイルの走査順に基づいてサブジョブを分割します。
- ターゲットテーブルに多くのインデックスがある場合、またはインデックス列の値がデータファイル内で散在している場合、各サブジョブのエンコードによって生成されたインデックスKVも重複します。

バックエンドタスク分散実行フレームワークが有効にされている場合、`IMPORT INTO` ステートメントで `CLOUD_STORAGE_URI` オプションを指定するか、エンコードされたKVデータのターゲットストレージアドレスをシステム変数[`tidb_cloud_storage_uri`](/system-variables.md#tidb_cloud_storage_uri-new-in-v740)を使用して指定することで [Global Sort](/tidb-global-sort.md) 機能を有効にすることができます。現時点では、Global Sort ストレージアドレスとしてS3のみがサポートされています。Global Sort を有効にすると、`IMPORT INTO` はエンコードされたKVデータをクラウドストレージに書き込み、クラウドストレージでグローバルソートを実行し、その後世界的にソートされたインデックスとテーブルデータを並列してTiKVにインポートします。これにより、KVの重複による問題を防止し、インポートの安定性が向上します。

Global Sortは大量のメモリリソースを消費します。データインポート前に、インポート効率に影響を与えないように頻繁にgolang GCをトリガーしないようにするために、[`tidb_server_memory_limit_gc_trigger`](/system-variables.md#tidb_server_memory_limit_gc_trigger-new-in-v640)および[`tidb_server_memory_limit`](/system-variables.md#tidb_server_memory_limit-new-in-v640)変数を設定することをお勧めします。
```
SET GLOBAL tidb_server_memory_limit_gc_trigger=0.99;
SET GLOBAL tidb_server_memory_limit='88%';
```

> **注意:**
>
> - キー値範囲のオーバーラップが低い場合、Global Sort を有効にすると、インポートのパフォーマンスが低下する可能性があります。これは、Global Sort を有効にすると、TiDB が Global Sort 操作とその後のインポートを続行する前に、すべてのサブジョブでのローカルソートの完了を待つ必要があるためです。
> - Global Sort を使用してインポートを完了すると、Global Sort のクラウドストレージに格納されたファイルがバックグラウンドスレッドで非同期にクリーンアップされます。

## 出力

`IMPORT INTO` がインポートを完了するか、`DETACHED` モードが有効になっている場合、`IMPORT INTO` は出力で現在のジョブ情報を返します。次の例に示すように、各フィールドの説明については [`SHOW IMPORT JOB(s)`](/sql-statements/sql-statement-show-import-job.md) を参照してください。

`IMPORT INTO` がインポートを完了すると、次のような出力が表示されます。

```sql
IMPORT INTO t FROM '/path/to/small.csv';
+--------+--------------------+--------------+----------+-------+----------+------------------+---------------+----------------+----------------------------+----------------------------+----------------------------+------------+
| Job_ID | Data_Source        | Target_Table | Table_ID | Phase | Status   | Source_File_Size | Imported_Rows | Result_Message | Create_Time                | Start_Time                 | End_Time                   | Created_By |
+--------+--------------------+--------------+----------+-------+----------+------------------+---------------+----------------+----------------------------+----------------------------+----------------------------+------------+
|  60002 | /path/to/small.csv | `test`.`t`   |      363 |       | finished | 16B              |             2 |                | 2023-06-08 16:01:22.095698 | 2023-06-08 16:01:22.394418 | 2023-06-08 16:01:26.531821 | root@%     |
+--------+--------------------+--------------+----------+-------+----------+------------------+---------------+----------------+----------------------------+----------------------------+----------------------------+------------+
```

`DETACHED` モードが有効になっている場合、`IMPORT INTO` ステートメントを実行すると、すぐに出力でジョブ情報が表示されます。出力から、ジョブのステータスが `pending` であることがわかります。これは、実行を待っていることを意味します。

```sql
IMPORT INTO t FROM '/path/to/small.csv' WITH DETACHED;
+--------+--------------------+--------------+----------+-------+---------+------------------+---------------+----------------+----------------------------+------------+----------+------------+
| Job_ID | Data_Source        | Target_Table | Table_ID | Phase | Status  | Source_File_Size | Imported_Rows | Result_Message | Create_Time                | Start_Time | End_Time | Created_By |
+--------+--------------------+--------------+----------+-------+---------+------------------+---------------+----------------+----------------------------+------------+----------+------------+
|  60001 | /path/to/small.csv | `test`.`t`   |      361 |       | pending | 16B              |          NULL |                | 2023-06-08 15:59:37.047703 | NULL       | NULL     | root@%     |
+--------+--------+--------------------+--------------+----------+-------+---------+------------------+---------------+----------------+----------------------------+------------+----------+------------+
```

## インポートジョブの表示と管理

`DETACHED` モードで有効になっているインポートジョブについては、[`SHOW IMPORT`](/sql-statements/sql-statement-show-import-job.md) を使用して現在のジョブの進行状況を表示できます。

インポートジョブが開始された後、[`CANCEL IMPORT JOB <job-id>`](/sql-statements/sql-statement-cancel-import-job.md) を使用してキャンセルすることができます。

## 例

### ヘッダー付きの CSV ファイルをインポート

```sql
IMPORT INTO t FROM '/path/to/file.csv' WITH skip_rows=1;
```

### `DETACHED` モードで非同期にファイルをインポート

```sql
IMPORT INTO t FROM '/path/to/file.csv' WITH DETACHED;
```

### データファイルの特定のフィールドをインポートしない

データファイルが CSV 形式で、その内容が次のとおりであるとします。

```
id,name,age
1,Tom,23
2,Jack,44
```

並びに、インポート対象のテーブルのスキーマが `CREATE TABLE t(id int primary key, name varchar(100))` であるとします。データファイルからテーブル `t` に `age` フィールドをインポートしないようにするには、次の SQL ステートメントを実行します。

```sql
IMPORT INTO t(id, name, @1) FROM '/path/to/file.csv' WITH skip_rows=1;
```

### ワイルドカード `*` を使用して複数のデータファイルをインポート

`/path/to/` ディレクトリに `file-01.csv`、`file-02.csv`、`file-03.csv` という名前の 3 つのファイルがあるとします。`IMPORT INTO` を使用してこれらの 3 つのファイルを対象テーブル `t` にインポートするには、次の SQL ステートメントを実行します。

```sql
IMPORT INTO t FROM '/path/to/file-*.csv'
```

### Amazon S3、GCS、または Azure Blob Storage からデータファイルをインポート

- Amazon S3 からデータファイルをインポート:

    ```sql
    IMPORT INTO t FROM 's3://bucket-name/test.csv?access-key=XXX&secret-access-key=XXX';
    ```

- GCS からデータファイルをインポート:

    ```sql
    IMPORT INTO t FROM 'gs://import/test.csv?credentials-file=${credentials-file-path}';
    ```

- Azure Blob Storage からデータファイルをインポート:

    ```sql
    IMPORT INTO t FROM 'azure://import/test.csv?credentials-file=${credentials-file-path}';
    ```

Amazon S3、GCS、または Azure Blob Storage の URI パス構成の詳細については、[外部ストレージサービスの URI フォーマット](/external-storage-uri.md) を参照してください。

### SetClause を使用して列の値を計算

データファイルが CSV 形式で、その内容が次のとおりであるとします。

```
id,name,val
1,phone,230
2,book,440
```

並びに、インポート対象のテーブルのスキーマが `CREATE TABLE t(id int primary key, name varchar(100), val int)` であるとします。インポート時に `val` 列の値を 100 倍にしたい場合は、次の SQL ステートメントを実行します。

```sql
IMPORT INTO t(id, name, @1) SET val=@1*100 FROM '/path/to/file.csv' WITH skip_rows=1;
```

### SQL 形式のデータファイルをインポート

```sql
IMPORT INTO t FROM '/path/to/file.sql' FORMAT 'sql';
```

### TiKV への書き込み速度を制限

TiKV ノードへの書き込み速度を 10 MiB/s に制限するには、次の SQL ステートメントを実行します。

```sql
IMPORT INTO t FROM 's3://bucket/path/to/file.parquet?access-key=XXX&secret-access-key=XXX' FORMAT 'parquet' WITH MAX_WRITE_SPEED='10MiB';
```

## MySQL 互換性

このステートメントは、MySQL 構文の TiDB 拡張です。

## 関連情報

* [`SHOW IMPORT JOB(s)`](/sql-statements/sql-statement-show-import-job.md)
* [`CANCEL IMPORT JOB`](/sql-statements/sql-statement-cancel-import-job.md)