---
title: ダンプリングの概要
summary: Dumpling ツールを使用して TiDB からデータをエクスポートします。
aliases: ['/docs/dev/mydumper-overview/','/docs/dev/reference/tools/mydumper/','/tidb/dev/mydumper-overview/']
---

# ダンプリングを使用してデータをエクスポート

このドキュメントは、データエクスポートツールである [ダンプリング](https://github.com/pingcap/tidb/tree/master/dumpling) を紹介します。ダンプリングは、TiDB/MySQL に格納されているデータを SQL または CSV データファイルとしてエクスポートし、論理的なフルバックアップまたはエクスポートを行うことができます。また、ダンプリングは Amazon S3 にデータをエクスポートすることもサポートしています。

<CustomContent platform="tidb">

[Dumpling](/tiup/tiup-overview.md) を取得するには、`tiup install dumpling` を実行してください。その後、`tiup dumpling ...` を実行してダンプリングを使用することができます。

ダンプリングのインストールパッケージは TiDB ツールキットに含まれています。TiDB ツールキットをダウンロードする場合は、[TiDB Tools のダウンロード](/download-ecosystem-tools.md) を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

以下のコマンドを使用してダンプリングをインストールできます：

```bash
curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
source ~/.bash_profile
tiup install dumpling
```

上記のコマンドでは、`~/.bash_profile` をプロファイルファイルのパスに変更する必要があります。

</CustomContent>

ダンプリングの詳細な使用方法については、`--help` オプションを使用するか、[ダンプリングのオプションリスト](#option-list-of-dumpling) を参照してください。

ダンプリングを使用する際には、実行中のクラスタでエクスポートコマンドを実行する必要があります。

<CustomContent platform="tidb">

TiDB には、必要に応じて使用できる他のツールも提供されています。

- SST ファイル（キーと値のペア）のバックアップや、遅延に敏感でない増分データのバックアップには、[BR](/br/backup-and-restore-overview.md) を参照してください。
- 増分データのリアルタイムバックアップには、[TiCDC](/ticdc/ticdc-overview.md) を参照してください。
- すべてのエクスポートされたデータは、[TiDB Lightning](/tidb-lightning/tidb-lightning-overview.md) を使用して、TiDB に再インポートすることができます。

</CustomContent>

> **注記:**
>
> PingCAP は以前、TiDB 固有の拡張を含む [mydumper プロジェクト](https://github.com/maxbube/mydumper) のフォークをメンテナンスしていました。v7.5.0 以降は、[Mydumper](https://docs.pingcap.com/tidb/v4.0/mydumper-overview) は非推奨とされ、その機能の大部分が [ダンプリング](/dumpling-overview.md) に置き換えられました。mydumper の代わりにダンプリングを使用することを強くお勧めします。

ダンプリングには、以下の利点があります：

- SQL や CSV を含む複数の形式でデータをエクスポートできます。
- データをフィルタリングしやすくする [table-filter](https://github.com/pingcap/tidb-tools/blob/master/pkg/table-filter/README.md) 機能をサポートしています。
- データを Amazon S3 クラウドストレージにエクスポートすることができます。
- TiDB にはさらに最適化が施されています：
    - 単一の TiDB SQL ステートメントのメモリ制限を設定できます。
    - Dumpling が直接 PD に接続できる場合、TiDB v4.0.0 およびそれ以降のバージョンでは、TiDB GC 時間の自動調整をサポートします。
    - 1 つのテーブルからの同時データエクスポートのパフォーマンスを最適化するために、TiDB の隠し列 `_tidb_rowid` を使用します。
    - TiDB では、データバックアップの一貫性を確保するために、[`tidb_snapshot`](/read-historical-data.md#how-tidb-reads-data-from-history-versions) の値を設定して、データバックアップの時点を指定できます。これにより、`FLUSH TABLES WITH READ LOCK` を使用してバックアップの一貫性を保証する必要がありません。

> **注記:**
>
> ダンプリングは、以下のシナリオで PD に接続できません：
>
> - TiDB クラスタが Kubernetes 上で実行されている場合（ダンプリング自体が Kubernetes 環境内で実行されていない限り）。
> - TiDB クラウド上で TiDB クラスタが実行されている場合。
>
> このような場合は、[TiDB GC 時間を手動で調整](#manually-set-the-tidb-gc-time) して、エクスポートの失敗を回避する必要があります。

## TiDB や MySQL からデータをエクスポート

### 必要な権限

- PROCESS: クラスタ情報をクエリして PD アドレスを取得し、その後 PD を介して GC を制御するために必要です。
- SELECT: テーブルをエクスポートする際に必要です。
- RELOAD: `consistency flush` を使用する場合に必要です。なお、この権限は TiDB のみサポートしています。アップストリームが RDS データベースまたはマネージドサービスの場合は、この権限を無視できます。
- LOCK TABLES: `consistency lock` を使用する場合に必要です。この権限は、エクスポートするすべてのデータベースおよびテーブルに付与する必要があります。
- REPLICATION CLIENT: メタデータをエクスポートしてデータスナップショットを記録する際に必要です。この権限はオプションであり、メタデータをエクスポートする必要がない場合は無視できます。

### SQL ファイルへのエクスポート

このドキュメントでは、ホスト 127.0.0.1:4000 に TiDB インスタンスがあり、この TiDB インスタンスにはパスワードのない root ユーザーがいると仮定しています。

ダンプリングはデフォルトで SQL ファイルにデータをエクスポートします。`--filetype sql` フラグを追加することで、SQL ファイルにデータをエクスポートできます：

{{< copyable "shell-regular" >}}

```shell
dumpling -u root -P 4000 -h 127.0.0.1 --filetype sql -t 8 -o /tmp/test -r 200000 -F 256MiB
```

上記のコマンドについて：

+ `-h`、`-P`、`-u` オプションは、それぞれアドレス、ポート、ユーザーを意味します。認証にパスワードが必要な場合は、`-p $YOUR_SECRET_PASSWORD` を使用してパスワードをダンプリングに渡すことができます。
+ `-o`（または `--output`）オプションは、ストレージのエクスポートディレクトリを指定します。絶対ローカルファイルパスまたは [外部ストレージ URI](/external-storage-uri.md) をサポートしています。
+ `-t` オプションは、エクスポートのスレッド数を指定します。スレッド数を増やすと、Dumpling の並行性とエクスポート速度が向上し、データベースのメモリ消費量も増加します。したがって、数を大きく設定することは推奨されていません。通常、64 より小さい値です。
+ `-r` オプションは、テーブル内の同時処理を有効にしてエクスポートを高速化します。デフォルト値は `0` で、無効を意味します。0 より大きい値は有効で、その値は `INT` 型です。ソースデータベースが TiDB の場合、 `-r` 値が 0 より大きいと、TiDB リージョン情報が分割に使用され、メモリ使用量が削減されます。指定した `-r` 値は分割アルゴリズムに影響を与えません。ソースデータベースが MySQL で、プライマリキーが `INT` 型の場合、`-r` を指定することで、テーブル内の同時処理も有効にできます。
+ `-F` オプションは、単一ファイルの最大サイズを指定します（ここでの単位は `MiB` です。`5GiB` や `8KB` などの入力も受け付け可能です）。この値は、このファイルを TiDB インスタンスにロードする際に TiDB Lightning を使用する予定がある場合は、256 MiB 以下に保つことをお勧めします。

> **注記:**
>
> 単一のエクスポートされたテーブルのサイズが 10 GB を超える場合、**`-r` と `-F` オプションを使用することを強くお勧めします**。

#### ストレージサービスの URI フォーマット

このセクションでは、Amazon S3、GCS、および Azure Blob Storage の URI フォーマットについて説明します。URI フォーマットは次のようになります：

```shell
[scheme]://[host]/[path]?[parameters]
```

詳細については、[外部ストレージサービスの URI フォーマット](/external-storage-uri.md) を参照してください。

### CSV ファイルへのエクスポート

`--filetype csv` 引数を追加することで、データを CSV ファイルにエクスポートできます。

CSV ファイルにデータをエクスポートする際、`--sql <SQL>` を使用して SQL ステートメントでレコードをフィルタリングできます。たとえば、次のコマンドで、`test.sbtest1` で `id < 100` に一致するレコードすべてをエクスポートできます：

{{< copyable "shell-regular" >}}

```shell
./dumpling -u root -P 4000 -h 127.0.0.1 -o /tmp/test --filetype csv --sql 'select * from `test`.`sbtest1` where id < 100' -F 100MiB --output-filename-template 'test.sbtest1.{{.Index}}'
```

上記のコマンドについて：

- `--sql` オプションは CSV ファイルにエクスポートする際のみ使用できます。上記のコマンドは、すべてのエクスポートするテーブルの `SELECT * FROM <table-name> WHERE id < 100` ステートメントを実行します。指定したフィールドを持たないテーブルの場合、エクスポートに失敗します。

<CustomContent platform="tidb">

- `--sql` オプションを使用する場合、Dumpling はエクスポートされたテーブルとスキーマ情報を取得できません。 `--output-filename-template` オプションを使用すると、CSV ファイルのファイル名形式を指定できます。このオプションを使用すると [TiDB Lightning](/tidb-lightning/tidb-lightning-overview.md) を使ってデータファイルをインポートしやすくなります。 例えば、`--output-filename-template='test.sbtest1.{{.Index}}'` は、エクスポートされた CSV ファイルが `test.sbtest1.000000000` または `test.sbtest1.000000001` と名前が付けられることを指定します。

</CustomContent>

<CustomContent platform="tidb-cloud">

- `--sql` オプションを使用する場合、Dumpling はエクスポートされたテーブルとスキーマ情報を取得できません。 `--output-filename-template` オプションを使用すると、CSV ファイルのファイル名形式を指定できます。 例えば、`--output-filename-template='test.sbtest1.{{.Index}}'` は、エクスポートされた CSV ファイルが `test.sbtest1.000000000` または `test.sbtest1.000000001` と名前が付けられることを指定します。

</CustomContent>

- `--csv-separator` および `--csv-delimiter` などのオプションを使用して CSV ファイルの形式を構成できます。詳細については[Dumpling オプションリスト](#option-list-of-dumpling)を参照してください。

> **注意:**
>
> *Dumpling* は *文字列* と *キーワード* を区別しません。 インポートされるデータがブール型の場合、 `true` の値は `1` に変換され、 `false` の値は `0` に変換されます。

### エクスポートされたデータファイルを圧縮します

Dumpling でエクスポートされた CSV および SQL のデータおよびテーブル構造ファイルを圧縮するには、`--compress <フォーマット>` オプションを使用します。 このパラメーターは次の圧縮アルゴリズムをサポートしています。`gzip`、`snappy` および `zstd` です。 圧縮はデフォルトで無効になっています。

- このオプションは個々のデータおよびテーブル構造ファイルのみを圧縮します。 フォルダ全体を圧縮して単一の圧縮パッケージを生成することはできません。
- このオプションはディスクスペースを節約できますが、エクスポート速度を遅くし、CPU 消費を増加させます。 エクスポート速度が重要なシナリオでは、このオプションを慎重に使用してください。
- TiDB Lightning v6.5.0 以降のバージョンでは、Dumpling によってエクスポートされた圧縮ファイルを追加の構成なしでデータソースとして使用できます。

> **注意:**
>
> Snappy 圧縮ファイルは[公式の Snappy フォーマット](https://github.com/google/snappy)である必要があります。 その他のバリアントの Snappy 圧縮はサポートされていません。

### エクスポートファイルの形式

- `metadata`: エクスポートされたファイルの開始時刻とマスターバイナリログの位置。

    {{< copyable "shell-regular" >}}

    ```shell
    cat metadata
    ```

    ```shell
    Started dump at: 2020-11-10 10:40:19
    SHOW MASTER STATUS:
            Log: tidb-binlog
            Pos: 420747102018863124
    Finished dump at: 2020-11-10 10:40:20
    ```

- `{schema}-schema-create.sql`: スキーマを作成するために使用される SQL ファイル

    {{< copyable "shell-regular" >}}

    ```shell
    cat test-schema-create.sql
    ```

    ```shell
    CREATE DATABASE `test` /*!40100 DEFAULT CHARACTER SET utf8mb4 */;
    ```

- `{schema}.{table}-schema.sql`: テーブルを作成するために使用される SQL ファイル

    {{< copyable "shell-regular" >}}

    ```shell
    cat test.t1-schema.sql
    ```

    ```shell
    CREATE TABLE `t1` (
      `id` int(11) DEFAULT NULL
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;
    ```

- `{schema}.{table}.{0001}.{sql|csv}`: データソースファイル

    {{< copyable "shell-regular" >}}

    ```shell
    cat test.t1.0.sql
    ```

    ```shell
    /*!40101 SET NAMES binary*/;
    INSERT INTO `t1` VALUES
    (1);
    ```

- `*-schema-view.sql`, `*-schema-trigger.sql`, `*-schema-post.sql`: その他のエクスポートされたファイル

### Amazon S3 クラウドストレージにデータをエクスポート

v4.0.8 以降、Dumpling はクラウドストレージにデータをエクスポートすることをサポートしています。 Amazon S3 にデータをバックアップする必要がある場合は、`-o` パラメーターで Amazon S3 ストレージパスを指定する必要があります。

指定されたリージョンで Amazon S3 バケットを作成する必要があります ([Amazon ドキュメント - S3 バケットの作成方法](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/create-bucket.html) を参照)。 バケット内にフォルダを作成する必要がある場合は、[Amazon ドキュメント - フォルダの作成](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/create-folder.html)を参照してください。

Amazon S3 バックエンドストレージにアクセスする権限を持つアカウントの `SecretKey` と `AccessKey` を環境変数として Dumpling ノードに渡す必要があります。

{{< copyable "shell-regular" >}}

```shell
export AWS_ACCESS_KEY_ID=${AccessKey}
export AWS_SECRET_ACCESS_KEY=${SecretKey}
```

Dumpling は `~/.aws/credentials` から資格情報ファイルを読み取ることもサポートしています。 外部ストレージサービスの URI パラメーターの記述についての詳細については、 [外部ストレージサービスの URI フォーマット](/external-storage-uri.md)を参照してください。

```shell
./dumpling -u root -P 4000 -h 127.0.0.1 -r 200000 -o "s3://${Bucket}/${Folder}"
```

### エクスポートされたデータをフィルタリングする

#### `--where` オプションを使用してデータをフィルタリングする

デフォルトでは、Dumpling は `mysql`、`sys`、`INFORMATION_SCHEMA`、`PERFORMANCE_SCHEMA`、`METRICS_SCHEMA` および `INSPECTION_SCHEMA` を含むシステムデータベースを除くすべてのデータベースをエクスポートします。 `--where <SQL where expression>` を使用してエクスポート対象のレコードを選択できます。

```shell
./dumpling -u root -P 4000 -h 127.0.0.1 -o /tmp/test --where "id < 100"
```

上記のコマンドは、各テーブルから `id < 100` に一致するデータをエクスポートします。 `--where` パラメーターを `--sql` と同時に使用することはできません。

#### `--filter` オプションを使用してデータをフィルタリングする

`--filter` オプションを使って特定のデータベースやテーブルをフィルタリングできます。 テーブルフィルターの構文は `.gitignore` に似ています。 詳細については、[Table Filter](/table-filter.md)を参照してください。

{{< copyable "shell-regular" >}}

```shell
./dumpling -u root -P 4000 -h 127.0.0.1 -o /tmp/test -r 200000 --filter "employees.*" --filter "*.WorkOrder"
```

上記のコマンドは、`employees` データベース内のすべてのテーブルおよびすべてのデータベースの `WorkOrder` テーブルをエクスポートします。

#### `-B` または `-T` オプションを使用してデータをフィルタリングする

Dumpling は `-B` オプションで特定のデータベースをエクスポートしたり、`-T` オプションで特定のテーブルをエクスポートしたりすることができます。

> **注意:**
>
> - `--filter` オプションと `-T` オプションを同時に使用することはできません。
> - `-T` オプションは `データベース名.テーブル名` のような完全な形式の入力のみを受け入れることができ、テーブル名のみの入力は受け入れません。 例: Dumpling は `-T WorkOrder` を認識できません。

例:

- `-B employees` は `employees` データベースをエクスポートします。
- `-T employees.WorkOrder` は `employees.WorkOrder` テーブルをエクスポートします。

### 並列処理を使用してエクスポート効率を向上させる

デフォルトでは、エクスポートされたファイルは `./export-<現在のローカル時間>` ディレクトリに保存されます。一般的に使用されるオプションは次のとおりです。

- `-t` オプションはエクスポートのスレッド数を指定します。 スレッド数を増やすと、Dumpling の並列処理とエクスポート速度が向上し、データベースのメモリ消費量も増加します。 したがって、数値を大きく設定することは推奨されません。
- `-r` オプションは、テーブル内のコンカレンシーを有効にしてエクスポートを高速化します。 デフォルト値は `0` で、無効を意味します。 0 より大きい値が指定されている場合は有効になります。 値は `INT` 型です。元のデータベースが TiDB の場合、`-r` の値を 0 より大きくすると、TiDB リージョン情報が使用されて分割され、メモリの使用量が減少します。 特定の `-r` の値は分割アルゴリズムに影響を与えません。 元のデータベースが MySQL で、プライマリキーが `INT` 型の場合、`-r` を指定することでテーブル内のコンカレンシーを有効にすることもできます。
- `--compress <format>` オプションはダンプの圧縮形式を指定します。 `gzip`、`snappy`、`zstd` の圧縮アルゴリズムをサポートしています。 優先することでデータのダンプを高速化できますが、ストレージがボトルネックである場合やストレージ容量に制限がある場合に有利です。 欠点は CPU 使用量の増加です。 各ファイルを個別に圧縮します。

上記のオプションを指定することで、Dumpling はデータエクスポートの速度を向上させることができます。

### Dumpling のデータ一貫性オプションを調整する

> **注意:**
>
> データ一貫性オプションのデフォルト値は `auto` です。 ほとんどのシナリオでは、Dumpling のデフォルトのデータ一貫性オプションを調整する必要はありません。
```
ダンプリングは、`--consistency <一貫性レベル>` オプションを使用して、データのエクスポートにおける「一貫性保証」の方法を制御します。一貫性のためにスナップショットを使用する場合、`--snapshot` オプションを使用してバックアップするタイムスタンプを指定できます。次の一貫性レベルを使用することもできます。

- `flush`: [`FLUSH TABLES WITH READ LOCK`](https://dev.mysql.com/doc/refman/8.0/en/flush.html#flush-tables-with-read-lock) を使用して、一時的にレプリカデータベースの DML と DDL 操作を中断し、バックアップ接続のグローバルな一貫性を保証し、バイナリログの位置（POS）情報を記録します。すべてのバックアップ接続がトランザクションを開始した後にロックが解除されます。オフピーク時または MySQL レプリカデータベースでフルバックアップを実行することを推奨します。
- `snapshot`: 指定したタイムスタンプの一貫性のあるスナップショットを取得してエクスポートします。
- `lock`: エクスポートするすべてのテーブルに読み取りロックを追加します。
- `none`: 一貫性を保証しません。
- `auto`: MySQL には `flush`、TiDB には `snapshot` を使用します。

すべてが完了したら、エクスポートされたファイルを `/tmp/test` に表示できます:

{{< copyable "shell-regular" >}}

```shell
ls -lh /tmp/test | awk '{print $5 "\t" $9}'
```

```
140B  metadata
66B   test-schema-create.sql
300B  test.sbtest1-schema.sql
190K  test.sbtest1.0.sql
300B  test.sbtest2-schema.sql
190K  test.sbtest2.0.sql
300B  test.sbtest3-schema.sql
190K  test.sbtest3.0.sql
```

### TiDB の歴史データスナップショットのエクスポート

Dumpling は、`--snapshot` オプションを指定して、特定の [tidb_snapshot](/read-historical-data.md#how-tidb-reads-data-from-history-versions) のデータをエクスポートできます。

`--snapshot` オプションには TSO（`SHOW MASTER STATUS` コマンドの `Position` フィールドによって出力される）または `datetime` データ型の有効な時間（`YYYY-MM-DD hh:mm:ss` 形式）を設定できます。例:

{{< copyable "shell-regular" >}}

```shell
./dumpling --snapshot 417773951312461825
./dumpling --snapshot "2020-07-02 17:12:45"
```

TSO が `417773951312461825` および時間が `2020-07-02 17:12:45` のときの TiDB の歴史データスナップショットがエクスポートされます。

### 大きなテーブルのエクスポートメモリ使用量を制御

Dumpling が TiDB から大きな単一のテーブルをエクスポートしている場合、エクスポートされるデータサイズが大きすぎて Out of Memory (OOM) が発生し、接続が中断されてエクスポートに失敗する可能性があります。次のパラメータを使用して TiDB のメモリ使用量を減らすことができます:

+ `-r` を設定してエクスポートするデータをチャンクに分割します。これにより、TiDB のデータスキャンのメモリオーバーヘッドが減少し、並列テーブルデータダンプが可能になり、エクスポート効率が向上します。上流データベースが TiDB v3.0 以降のバージョンの場合、`-r` の値が 0 より大きいと、TiDB リージョン情報がスプリットに使用され、特定の `-r` の値はスプリットアルゴリズムに影響しません。
+ `--tidb-mem-quota-query` の値を `8589934592` (8 GB) またはそれ以下に減らします。`--tidb-mem-quota-query` は、TiDB の単一のクエリステートメントのメモリ使用量を制御します。
+ `--params "tidb_distsql_scan_concurrency=5"` パラメータを調整します。[`tidb_distsql_scan_concurrency`](/system-variables.md#tidb_distsql_scan_concurrency) は、TiDB におけるスキャン操作の並行性を制御するセッション変数です。

### TiDB GC 時間の手動設定

TiDB からデータをエクスポートする場合（1 TB を超える）、TiDB バージョンが v4.0.0 以上であり、Dumpling が TiDB クラスタの PD アドレスにアクセスできる場合、Dumpling はオリジナルのクラスタに影響を与えることなく自動的に GC 時間を延長します。

ただし、次のいずれかのシナリオで、Dumpling は自動的に GC 時間を調整できません:

- データサイズが非常に大きい（1 TB を超える）場合。
- Dumpling が直接 PD に接続できない場合。たとえば、TiDB クラスタが TiDB クラウド上にある場合や、Dumpling とは別に分離された Kubernetes 上にある場合。

このようなシナリオでは、エクスポート中に GC によるエクスポートの失敗を回避するために、事前に GC 時間を手動で延長する必要があります。

GC 時間を手動で調整するには、次の SQL 文を使用します:

```sql
SET GLOBAL tidb_gc_life_time = '720h';
```

Dumpling を終了した後、エクスポートが成功したかどうかに関わらず、GC 時間をオリジナルの値に戻す必要があります（デフォルト値は `10m` です）。

```sql
SET GLOBAL tidb_gc_life_time = '10m';
```

## Dumpling のオプションリスト

| オプション                  | 用途                                                                                                                                                                                                                                                                                                                          | デフォルト値                                  |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------- |
| `-V` または `--version`      | Dumpling のバージョンを出力し、直ちに終了します                                                                                                                                                                                                                                                                                |
| `-B` または `--database`     | 指定したデータベースをエクスポートします                                                                                                                                                                                                                                                                                        |
| `-T` または `--tables-list`  | 指定したテーブルをエクスポートします                                                                                                                                                                                                                                                                                            |
| `-f` または `--filter`       | フィルタパターンに一致するテーブルをエクスポートします。フィルタの構文については、[table-filter](/table-filter.md) を参照してください。                                                                                                                                                                                   | `[\*.\*,!/^(mysql&#124;sys&#124;INFORMATION_SCHEMA&#124;PERFORMANCE_SCHEMA&#124;METRICS_SCHEMA&#124;INSPECTION_SCHEMA)$/.\*]`（システムスキーマを除くすべてのデータベースまたはテーブルをエクスポート） |
| `--case-sensitive`           | テーブルフィルタが大文字・小文字を区別するかどうか                                                                                                                                                                                                                                                                           | false（大文字と小文字を区別しない）          |
| `-h` または `--host`         | 接続するデータベースホストの IP アドレス                                                                                                                                                                                                                                                                                        | "127.0.0.1"                                  |
| `-t` または `--threads`      | 並列バックアップスレッドの数                                                                                                                                                                                                                                                                                                    | 4                                            |
| `-r` または `--rows`         | エクスポートするためのテーブルにインテーブル並行を有効にします。デフォルト値は `0` で、無効を意味します。`0` より大きい値は有効にし、その値は `INT` 型です。ソースデータベースが TiDB の場合、`-r` の値が 0 より大きいと、TiDB リージョン情報が使われてスプリットされ、メモリ使用量が減少します。具体的な `-r` の値はスプリットアルゴリズムに影響しません。ソースデータベースが MySQL であり、プライマリキーが `INT` 型の場合、`-r` を指定すると、インテーブル並行が有効になります。                                                                                                                                                                                                               |
| `-L` または `--logfile`      | ログ出力先のアドレス。空の場合、ログはコンソールに出力されます                                                                                                                                                                                                                                                                | ""                                           |
| `--loglevel`                 | ログレベル {debug,info,warn,error,dpanic,panic,fatal}                                                                                                                                                                                                                                                                           | "info"                                       |
| `--logfmt`                   | ログの出力形式 {text,json}                                                                                                                                                                                                                                                                                                      | "text"                                       |
| `-d` または `--no-data`      | データをエクスポートしません（スキーマのみが必要なシナリオに適しています）                                                                                                                                                                                                                                                     |
| `--no-header`                | ヘッダを生成せずにテーブルの CSV ファイルをエクスポート                                                                                                                                                                                                                                                                        |
| `-W` または `--no-views`     | ビューをエクスポートしません                                                                                                                                                                                                                                                                                                    | true                                         |
| `-m` または `--no-schemas`   | データのみがエクスポートされるスキーマをエクスポートしません                                                                                                                                                                                                                                                                  |
| `-s` または `--statement-size`| `INSERT` ステートメントのサイズを制御します。単位はバイトです                                                                                                                                                                                                                                                                    |
| `-F` または `--filesize`      | 分割テーブルのファイルサイズ。`128B`、`64KiB`、`32MiB`、`1.5GiB` などの単位が指定されている必要があります。                                                                                                                                                                                                                  |
| `--filetype`                 | エクスポートされるファイルの種類（csv/sql）                                                                                                                                                                                                                                                                                     | "sql"                                        |
| `-o` または `--output`       | データのエクスポートのための絶対ローカルファイルパスまたは[外部ストレージ URI](/external-storage-uri.md) を指定します。                                                                                                                                                                                                                                  | "./export-${time}"                           |
| `-S` または `--sql`          | 指定された SQL ステートメントに従ってデータをエクスポートします。このコマンドは同時エクスポートをサポートしません。                                                                                                                                                                                                          |
| `--consistency`              | flush: ダンプの前に FTWRL を使用します <br/> snapshot: 特定の TSO の TiDB データをダンプします <br/> lock: エクスポートするすべてのテーブルに `lock tables read` を実行します <br/> none: ロックを追加せずにダンプしますが、一貫性を保証できません <br/> auto: MySQL では `flush`、TiDB では `snapshot` を使用します | "auto"                                       |
| `--snapshot`                 | スナップショット TSO；`consistency=snapshot` の場合にのみ有効です                                                                                                                                                                                                                                                               |
| `--where`                    | `where` 条件を通じてテーブルバックアップの範囲を指定します                                                                                                                                                                                                                                                                      |
| `-p` または `--password`     | 接続するデータベースホストのパスワード                                                                                                                                                                                                                                                                                          |
```
| `-P` または `--port`             | 接続されているデータベースホストのポート                                                                                                                                                                                                                                                                                            | 4000                                       |
| `-u` または `--user`             | 接続されているデータベースホストのユーザー名                                                                                                                                                                                                                                                                                        | "root"                                     |
| `--dump-empty-database`      | 空のデータベースの `CREATE DATABASE` ステートメントをエクスポートする                                                                                                                                                                                                                                                                     | true                                       |
| `--ca`                       | TLS接続のための証明機関ファイルのアドレス                                                                                                                                                                                                                                                                   |
| `--cert`                     | TLS接続のためのクライアント証明書ファイルのアドレス                                                                                                                                                                                                                                                                      |
| `--key`                      | TLS接続のためのクライアント秘密キーファイルのアドレス                                                                                                                                                                                                                                                                      |
| `--csv-delimiter`            | CSVファイル内の文字型変数の区切り記号                                                                                                                                                                                                                                                                                 | '"'                                        |
| `--csv-separator`            | CSVファイル内の各値のセパレーター。デフォルトの','を使用することはお勧めしません。'\|+\|'や他の一般的でない文字の組み合わせを使用することをお勧めします| ','                                        |
| `--csv-null-value`           | CSVファイル内のヌル値の表現                                                                                                                                                                                                                                                                                         | "\\N"                                      |
| `--csv-line-terminator`      | CSVファイルの行末の終端子。CSVファイルへのデータのエクスポート時に、このオプションで望ましい終端子を指定できます。このオプションは "\\r\\n" および "\\n" をサポートしています。デフォルト値は "\\r\\n" で、それは以前のバージョンと整合しています。bashの引用符には異なるエスケープルールがあるため、LF（改行）を終端子として指定する場合、似た構文、例えば `--csv-line-terminator $'\n'` を使用できます | "\\r\\n" |
| `--escape-backslash`         | エクスポートファイル内の特殊文字をエスケープするためにバックスラッシュ（`\`）を使用する                                                                                                                                                                                                                                                                | true                                       |
| `--output-filename-template` | [golangテンプレート](https://golang.org/pkg/text/template/#hdr-Arguments) 形式で表されたファイル名テンプレート <br/> `{{.DB}}`, `{{.Table}}`, `{{.Index}}` 引数をサポート <br/> これらの引数はそれぞれデータベース名、テーブル名、データファイルのチャンクIDを表します                                  | `{{.DB}}.{{.Table}}.{{.Index}}`            |
| `--status-addr`              | Dumplingのサービスアドレス。Prometheusがメトリクスを取得し、pprofデバッグを行うためのアドレスも含まれます                                                                                                                                                                                                                               | ":8281"                                    |
| `--tidb-mem-quota-query`     | Dumplingコマンドの単一行でエクスポートされるSQLステートメントのメモリ制限。単位はバイトです。v4.0.10以降のバージョンでは、このパラメータを設定しない場合、TiDBはデフォルトで `mem-quota-query` 設定項目の値をメモリ制限値として使用します。v4.0.10より前のバージョンでは、パラメータ値はデフォルトで 32 GB です。  | 34359738368 |
| `--params`                   | エクスポートするデータベースの接続のセッション変数を指定します。必要な形式は `"character_set_client=latin1,character_set_connection=latin1"`                                                                                                                                                           |
|  `-c` または `--compress` |  DumplingによってエクスポートされたCSVおよびSQLデータおよびテーブル構造ファイルを圧縮します。次の圧縮アルゴリズムをサポートしています：`gzip`、`snappy`、`zstd`  | "" |