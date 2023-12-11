---
title: TiDB Lightningのデータソース
summary: TiDB Lightningでサポートされるすべてのデータソースを学ぶ。
aliases: ['/docs/dev/tidb-lightning/migrate-from-csv-using-tidb-lightning/','/docs/dev/reference/tools/tidb-lightning/csv/','/tidb/dev/migrate-from-csv-using-tidb-lightning/']
---

# TiDB Lightningのデータソース

TiDB Lightningは、CSV、SQL、およびParquetファイルなど、複数のデータソースからTiDBクラスタにデータをインポートすることができます。

TiDB Lightningでのデータソースを指定するには、以下の構成を使用します：

```toml
[mydumper]
# ローカルのデータソースディレクトリ、またはS3などの外部ストレージのURI。外部ストレージのURIの詳細については、https://docs.pingcap.com/tidb/v6.6/backup-and-restore-storages#uri-formatを参照してください。
data-source-dir = "/data/my_database"
```

TiDB Lightningが実行されている間、`data-source-dir`に一致するすべてのファイルを探します。

| ファイル | タイプ | パターン |
| --------- | -------- | ------- |
| スキーマファイル | `CREATE TABLE` DDLステートメントを含む | `${db_name}.${table_name}-schema.sql` |
| スキーマファイル | `CREATE DATABASE` DDLステートメントを含む | `${db_name}-schema-create.sql` |
| データファイル | データファイルがテーブル全体のデータを含む場合、ファイルは`${db_name}.${table_name}`という名前のテーブルにインポートされます | <code>\${db_name}.\${table_name}.\${csv\|sql\|parquet}</code> |
| データファイル | テーブルのデータが複数のデータファイルに分割されている場合、各データファイルはファイル名で番号で終わらなければなりません | <code>\${db_name}.\${table_name}.001.\${csv\|sql\|parquet}</code> |
| 圧縮ファイル | ファイルに`gzip`、`snappy`、または`zstd`などの圧縮サフィックスが含まれている場合、TiDB Lightningはインポート前にファイルを展開します。Snappy圧縮ファイルは[公式のSnappyフォーマット](https://github.com/google/snappy)である必要があります。他のバリアントのSnappy圧縮はサポートされていません。 | <code>\${db_name}.\${table_name}.\${csv\|sql\|parquet}.{compress}</code> |

TiDB Lightningは可能な限り並列処理を行います。ファイルは順次読まれる必要があるため、データ処理の並行性はファイルレベル（`region-concurrency`によって制御）です。したがって、インポートされるファイルが大きい場合、インポートのパフォーマンスは低下します。最適なパフォーマンスを得るためには、インポートされるファイルのサイズを256 MiBを超えないように制限することを推奨します。

## データベースとテーブルの名前変更

TiDB Lightningはファイル名パターンに従ってデータを対応するデータベースとテーブルにインポートします。データベースまたはテーブルの名前が変更される場合は、ファイル名を変更してからインポートするか、オンラインで名前を置換することができます。

### 一括でファイル名を変更する

Red Hat LinuxまたはRed Hat Linuxベースのディストリビューションを使用している場合、`rename`コマンドを使用して`data-source-dir`ディレクトリ内のファイル名を一括で変更できます。

例：

```shell
rename srcdb. tgtdb. *.sql
```

データベース名を変更した後は、`data-source-dir`ディレクトリから`CREATE DATABASE` DDLステートメントを含む`${db_name}-schema-create.sql`ファイルを削除することをお勧めします。テーブル名も変更する場合は、`CREATE TABLE` DDLステートメントを含む`${db_name}.${table_name}-schema.sql`ファイル内のテーブル名も変更する必要があります。

### オンラインで名前を置換するための正規表現の使用

オンラインで名前を置換するためには、`[[mydumper.files]]`内の`pattern`構成を使用してファイル名を一致させ、`schema`および`table`をお好みの名前に置換できます。詳細については、[カスタマイズファイルの一致](#match-customized-files)を参照してください。

次に、正規表現を使用してオンラインで名前を置換する例を示します。

- データファイルの`pattern`の一致ルールは、`^({schema_regrex})\.({table_regrex})\.({file_serial_regrex})\.(csv|parquet|sql)`です。
- `schema`を`'$1'`と指定すると、最初の正規表現`schema_regrex`の値が変わらないことを意味します。または`'tgtdb'`などの固定されたターゲットデータベース名と指定することもできます。
- `table`を`'$2'`と指定すると、2番目の正規表現`table_regrex`の値が変わらないことを意味します。または`'t1'`などの固定されたターゲットテーブル名と指定することもできます。
- `type`を`'$3'`と指定すると、データファイルのタイプを意味します。`type`は`"table-schema"`（`schema.sql`ファイルを表す）または`"schema-schema"`（`schema-create.sql`ファイルを表す）のいずれかを指定できます。

```toml
[mydumper]
data-source-dir = "/some-subdir/some-database/"
[[mydumper.files]]
pattern = '^(srcdb)\.(.*?)-schema-create\.sql'
schema = 'tgtdb'
type = "schema-schema"
[[mydumper.files]]
pattern = '^(srcdb)\.(.*?)-schema\.sql'
schema = 'tgtdb'
table = '$2'
type = "table-schema"
[[mydumper.files]]
pattern = '^(srcdb)\.(.*?)\.(?:[0-9]+)\.(csv|parquet|sql)'
schema = 'tgtdb'
table = '$2'
type = '$3'
```

データファイルをバックアップする際に`gzip`を使用している場合、圧縮形式を適切に構成する必要があります。データファイルの`pattern`一致ルールは、`'^({schema_regrex})\.({table_regrex})\.({file_serial_regrex})\.(csv|parquet|sql)\.(gz)'`です。`compression`を`'$4'`と指定すると圧縮されたファイル形式を表します。例：

```toml
[mydumper]
data-source-dir = "/some-subdir/some-database/"
[[mydumper.files]]
pattern = '^(srcdb)\.(.*?)-schema-create\.(sql)\.(gz)'
schema = 'tgtdb'
type = "schema-schema"
compression = '$4'
[[mydumper.files]]
pattern = '^(srcdb)\.(.*?)-schema\.(sql)\.(gz)'
schema = 'tgtdb'
table = '$2'
type = "table-schema"
compression = '$4'
[[mydumper.files]]
pattern = '^(srcdb)\.(.*?)\.(?:[0-9]+)\.(sql)\.(gz)'
schema = 'tgtdb'
table = '$2'
type = '$3'
compression = '$4'
```

## CSV

### スキーマ

CSVファイルにはスキーマがありません。TiDBにCSVファイルをインポートするには、テーブルスキーマを提供する必要があります。以下の方法のいずれかでスキーマを提供できます：

* `CREATE TABLE` DDLステートメントを含む`${db_name}.${table_name}-schema.sql`および`${db_name}-schema-create.sql`という名前のファイルを作成する。
* TiDBでテーブルスキーマを手動で作成する。

### 構成

`tidb-lightning.toml`ファイルの`[mydumper.csv]`セクションでCSV形式を構成できます。ほとんどの設定には、MySQLの`LOAD DATA`ステートメントに対応するオプションがあります。

```toml
[mydumper.csv]
# フィールドの区切り文字。1つまたは複数の文字で指定できます。デフォルトは','です。
# データにコンマが含まれる可能性がある場合、'|+|'など、一般的でない文字の組み合わせを区切り文字として使用することをお勧めします。
separator = ','
# 引用符のデリミタ。空の値は引用符を使用しないことを意味します。
delimiter = '"'
# 行の終端。1つまたは複数の文字で指定できます。空の値（デフォルト）は"\n"（LF）および"\r\n"（CRLF）の両方が行の終端です。
terminator = ''
# CSVファイルにヘッダーが含まれるかどうか。
# `header`がtrueの場合、最初の行がスキップされ、テーブルの列にマップされます。
header = true
# CSVファイルにNULL値が含まれるかどうか。
# `not-null`がtrueの場合、CSVのすべての列にNULLを解析できません。
not-null = false
# `not-null`がfalseの場合（つまり、CSVにNULLを含むことができる場合）は、この値と等しいフィールドはNULLとして扱われます。
null = '\N'
# スラッシュをエスケープ文字として解釈するかどうか。
backslash-escape = true
# `separator`を行の終端として解釈し、末尾のセパレーターをすべてトリミングするかどうか。
trim-last-separator = false
```

`separator`、`delimiter`、または`terminator`などの文字列フィールドの入力に特殊文字が含まれる場合は、バックスラッシュを使用して特殊文字をエスケープすることができます。エスケープシーケンスは*二重引用符*の文字列（`"…"`）である必要があります。たとえば、`separator = "\u001f"`はASCII文字`0X1F`を区切り文字として使用することを意味します。

バックスラッシュのエスケープを抑制するには、*単一引用符*の文字列（`'…'`）を使用できます。例えば、`terminator = '\n'`は、LF`\n`ではなく、バックスラッシュ（\）に続く文字nという2文字の文字列を行の終端として使用することを意味します。
```
詳細については、[TOML v1.0.0 specification](https://toml.io/en/v1.0.0#string) を参照してください。

#### `separator`

- フィールドの区切り記号を定義する。
- 1つまたは複数の文字である必要があり、空であってはならない。
- 一般的な値:

    * CSV（カンマ区切り値）の場合は `','`。
    * TSV（タブ区切り値）の場合は `"\t"`。
    * ASCII文字 `0x01` を使用する場合は `"\u0001"`。

- `LOAD DATA` ステートメント内の `FIELDS TERMINATED BY` オプションに対応する。

#### `delimiter`

- 引用符用のデリミタを定義する。
- `delimiter` が空の場合、すべてのフィールドは引用符でくくられない。
- 一般的な値:

    * ダブルクォーテーションでフィールドを引用符で囲む場合は `'"'`。[RFC 4180](https://tools.ietf.org/html/rfc4180) と同じ。
    * 引用符を無効にする場合は `''`。

- `LOAD DATA` ステートメント内の `FIELDS ENCLOSED BY` オプションに対応する。

#### `terminator`

- 行の終端記号を定義する。
- `terminator` が空の場合、`"\n"`（改行）および `"\r\n"`（キャリッジリターン + 改行）の両方が行の終端記号として使用される。
- `LOAD DATA` ステートメント内の `LINES TERMINATED BY` オプションに対応する。

#### `header`

- *すべて*の CSV ファイルがヘッダー行を含んでいるかどうかを定義する。
- `header` が `true` の場合、最初の行が*列名*として使用される。`header` が `false` の場合、最初の行は通常のデータ行として扱われる。

#### `not-null` および `null`

- `not-null` 設定はすべてのフィールドがノンヌラブルかどうかを制御する。
- `not-null` が `false` の場合、`null` で指定された文字列は特定の値ではなく SQL の NULL に変換される。
- 引用符の使用はフィールドが NULL であるかどうかには影響しない。

    例えば、次の CSV ファイルでは：

    ```csv
    A,B,C
    \N,"\N",
    ```

    デフォルトの設定 (`not-null = false; null = '\N'`) により、列 `A` と `B` は TiDB にインポートされた後に両方とも NULL に変換される。列 `C` は空の文字列 `''` であり NULL ではない。

#### `backslash-escape`

- フィールド内のバックスラッシュをエスケープ文字として解釈するかどうかを定義する。
- `backslash-escape` が true の場合、以下のシーケンスが認識されて変換される:

    | シーケンス | 変換される値             |
    |----------|--------------------------|
    | `\0`     | Null 文字（`U+0000`）     |
    | `\b`     | バックスペース（`U+0008`） |
    | `\n`     | 改行（`U+000A`）         |
    | `\r`     | キャリッジリターン（`U+000D`） |
    | `\t`     | タブ（`U+0009`）           |
    | `\Z`     | Windows EOF（`U+001A`）   |

    他のすべてのケース（たとえば、`\"`）では、バックスラッシュは削除され、次の文字（`"`）がフィールド内に残る。残された文字には特別な役割（たとえば区切り記号）はなく、普通の文字である。

- 引用符の使用はバックスラッシュがエスケープ文字として解釈されるかどうかには影響しない。

- `LOAD DATA` ステートメント内の `FIELDS ESCAPED BY '\'` オプションに対応する。

#### `trim-last-separator`

- `separator` を行の終端記号として扱い、末尾のすべての区切り記号をトリムするかどうかを定義する。

    例えば、次の CSV ファイルでは：

    ```csv
    A,,B,,
    ```

    - `trim-last-separator = false` の場合、これは5つのフィールド `('A', '', 'B', '', '')` の行として解釈される。
    - `trim-last-separator = true` の場合、これは3つのフィールド `('A', '', 'B')` の行として解釈される。

- このオプションは非推奨である。代わりに `terminator` オプションを使用する。

    既存の設定が次のようになっている場合：

    ```toml
    separator = ','
    trim-last-separator = true
    ```

    次のように設定を変更することをお勧めします：

    ```toml
    separator = ','
    terminator = ",\n" # 実際のファイルに応じて ",\n" または ",'\r\n" を使用する。
    ```

#### 構成できないオプション

TiDB Lightning は `LOAD DATA` ステートメントでサポートされているすべてのオプションをサポートしていません。例:

* 行プレフィックスは存在しない（`LINES STARTING BY`）。
* ヘッダーをスキップすることはできない（`IGNORE n LINES` ）し、有効な列名である必要があります。

### 厳密な形式

TiDB Lightning は、入力ファイルが一様なサイズであることが最も適しています（約256 MiB）。入力が単一の巨大な CSV ファイルの場合、TiDB Lightning はファイルを1つのスレッドでのみ処理でき、インポート速度が遅くなります。

これは最初に CSV を複数のファイルに分割することで解決できます。汎用の CSV フォーマットの場合、行がどこで開始し、終了するかをすばやく識別する方法はないため、TiDB Lightning はデフォルトで CSV ファイルを自動的に分割しません。ただし、CSV 入力が特定の制限に準拠していることが確実な場合は、`strict-format` 設定を有効にして、TiDB Lightning にファイルを並列処理するための256 MiBサイズのチャンクに分割させることができます。

```toml
[mydumper]
strict-format = true
```

厳密な CSV ファイルでは、各フィールドが単一の行を占有します。言い換えれば、以下のいずれかが真である必要があります：

* デリミタが空である。
* すべてのフィールドには終端記号自体が含まれていない。デフォルトの構成では、これはすべてのフィールドが CR（`\r`）または LF（`\n`）を含んでいないことを意味します。

CSV ファイルが厳密でない場合、しかし `strict-format` が誤って `true` に設定されている場合、複数行にわたるフィールドは2つに分割され、パースエラーが発生したり、破損したデータが静かにインポートされることがあります。

### 一般的な構成例

#### CSV

デフォルトの設定は RFC 4180 に従うCSVにすでにチューニングされています。

```toml
[mydumper.csv]
separator = ',' # データにカンマ（','）が含まれる可能性がある場合、セパレータとして '|+|' または他の一般的でない文字の組み合わせを使用することをお勧めします。
delimiter = '"'
header = true
not-null = false
null = '\N'
backslash-escape = true
```

例:

```
ID,Region,Count
1,"East",32
2,"South",\N
3,"West",10
4,"North",39
```

#### TSV

```toml
[mydumper.csv]
separator = "\t"
delimiter = ''
header = true
not-null = false
null = 'NULL'
backslash-escape = false
```

例:

```
ID    Region    Count
1     East      32
2     South     NULL
3     West      10
4     North     39
```

#### TPC-H DBGEN

```toml
[mydumper.csv]
separator = '|'
delimiter = ''
terminator = "|\n"
header = false
not-null = true
backslash-escape = false
```

例:

```
1|East|32|
2|South|0|
3|West|10|
4|North|39|
```

## SQL

TiDB Lightning が SQL ファイルを処理する場合、単一の巨大な SQL ファイルをすばやく分割することはできないため、単一のファイルからのインポート速度を上げることはできません。したがって、SQL ファイルからデータをインポートする場合は、単一の巨大な SQL ファイルを避けてください。TiDB Lightning は、入力ファイルのサイズが一様であることが最適である（約256 MiB）。

## Parquet

TiDB Lightning は現在、Amazon Aurora または Apache Hive が生成した Parquet ファイルのみをサポートしています。S3 でファイル構造を特定するには、次の構成を使用してすべてのデータファイルを一致させます：

```
[[mydumper.files]]
# Amazon Aurora パーケットファイルを解析するために必要な式
pattern = '(?i)^(?:[^/]*/)*([a-z0-9\-_]+).([a-z0-9\-_]+)/(?:[^/]*/)*(?:[a-z0-9\-_.]+\.(parquet))$'
schema = '$1'
table = '$2'
type = '$3'
```

この構成は、Aurora スナップショットによってエクスポートされたパーケットファイルを一致させる方法のみを示しています。スキーマファイルを別途エクスポートし、処理する必要があります。

`mydumper.files` の詳細については、「カスタマイズしたファイルの一致」を参照してください。

## 圧縮されたファイル

TiDB Lightning は現在、Dumpling によってエクスポートされた圧縮されたファイルまたは命名規則に従った圧縮されたファイルのみをサポートしています。現在、TiDB Lightning は次の圧縮アルゴリズムをサポートしています: `gzip`、`snappy`、`zstd`。ファイル名が命名規則に従っている場合、TiDB Lightning は追加の構成なしで圧縮アルゴリズムを自動的に識別し、ストリーミング解凍後にファイルをインポートします。

> **注意:**
>
> - TiDB Lightning は単一の大きな圧縮ファイルを同時に解凍することはできないため、圧縮ファイルのサイズはインポート速度に影響を与えます。解凍後のソースファイルのサイズが256 MiBを超えないように推奨されます。
> - TiDB Lightning は個々に圧縮されたデータファイルのみをインポートし、複数のデータファイルが含まれた単一の圧縮ファイルのインポートには対応していません。
```
- TiDB Lightningでは、`db.table.parquet.snappy`などの他の圧縮ツールを介して圧縮された `parquet` ファイルはサポートされていません。`parquet` ファイルを圧縮する場合は、`parquet` ファイルライターの圧縮形式を構成できます。
- TiDB Lightning v6.4.0以降のバージョンでは、次の圧縮されたデータファイルのみがサポートされています: `gzip`、`snappy`、および `zstd`。他の種類のファイルはエラーを引き起こします。ソースデータファイルが格納されているディレクトリにサポートされていない圧縮ファイルが存在すると、これによりタスクがエラーを報告します。このようなエラーを回避するために、インポートデータディレクトリからこれらのサポートされていないファイルを移動できます。
- Snappy圧縮ファイルは[公式のSnappyフォーマット](https://github.com/google/snappy)でなければなりません。他のバリアントのSnappy圧縮はサポートされません。

## カスタマイズされたファイルを一致させる

TiDB Lightningは、指定のファイル名パターンに従うデータファイルのみを認識します。場合によっては、データファイルが指定のパターンに従わないため、データのインポートが短時間で完了し、ファイルがインポートされないことがあります。

この問題を解決するために、`[[mydumper.files]]`を使用して、カスタマイズした表現でデータファイルを一致させることができます。

Amazon AuroraのスナップショットがS3にエクスポートされた場合を例に挙げます。Parquetファイルの完全なパスは`S3://some-bucket/some-subdir/some-database/some-database.some-table/part-00000-c5a881bb-58ff-4ee6-1111-b41ecff340a3-c000.gz.parquet`です。

通常、`data-source-dir`は`S3://some-bucket/some-subdir/some-database/`に設定され、`some-database`データベースがインポートされます。

前述のParquetファイルパスに基づいて、`(?i)^(?:[^/]*/)*([a-z0-9\-_]+).([a-z0-9\-_]+)/(?:[^/]*/)*(?:[a-z0-9\-_.]+\.(parquet))$`のような正規表現を書き、ファイルを一致させることができます。一致グループでは、`index=1`が`some-database`、`index=2`が`some-table`、そして`index=3`が`parquet`です。

正規表現と対応するインデックスに従って構成ファイルを記述し、TiDB Lightningがデフォルトの命名規則に従わないデータファイルを認識できるようにできます。たとえば:

```toml
[[mydumper.files]]
# Amazon Aurora parquetファイルを解析するために必要な表現
pattern = '(?i)^(?:[^/]*/)*([a-z0-9\-_]+).([a-z0-9\-_]+)/(?:[^/]*/)*(?:[a-z0-9\-_.]+\.(parquet))$'
schema = '$1'
table = '$2'
type = '$3'
```

- **schema**: ターゲットデータベースの名前。値は以下のいずれかです:
    - 正規表現を使用して取得したグループインデックス、たとえば`$1`。
    - インポートしたいデータベースの名前、たとえば`db1`。一致したすべてのファイルは`db1`にインポートされます。
- **table**: ターゲットテーブルの名前。値は以下のいずれかです:
    - 正規表現を使用して取得したグループインデックス、たとえば`$2`。
    - インポートしたいテーブルの名前、たとえば`table1`。一致したすべてのファイルは`table1`にインポートされます。
- **type**: ファイルの種別。`sql`、`parquet`、`csv`をサポートしています。値は以下のいずれかです:
    - 正規表現を使用して取得したグループインデックス、たとえば`$3`。
- **key**: ファイル番号、`${db_name}.${table_name}.001.csv`での`001`のようなものです。
    - 正規表現を使用して取得したグループインデックス、たとえば`$4`。

## Amazon S3からデータをインポートする

次の例は、TiDB Lightningを使用してAmazon S3からデータをインポートする方法を示しています。より詳細なパラメータ構成については、[外部ストレージサービスのURIフォーマット](/external-storage-uri.md)を参照してください。

+ ローカルに構成された権限を使用してS3データにアクセスする場合:

    ```bash
    ./tidb-lightning --tidb-port=4000 --pd-urls=127.0.0.1:2379 --backend=local --sorted-kv-dir=/tmp/sorted-kvs \
        -d 's3://my-bucket/sql-backup'
    ```

+ パス形式のリクエストを使用してS3データにアクセスする場合:

    ```bash
    ./tidb-lightning --tidb-port=4000 --pd-urls=127.0.0.1:2379 --backend=local --sorted-kv-dir=/tmp/sorted-kvs \
        -d 's3://my-bucket/sql-backup?force-path-style=true&endpoint=http://10.154.10.132:8088'
    ```

+ 特定のAWS IAMロールARNを使用してS3データにアクセスする場合:

    ```bash
    ./tidb-lightning --tidb-port=4000 --pd-urls=127.0.0.1:2379 --backend=local --sorted-kv-dir=/tmp/sorted-kvs \
        -d 's3://my-bucket/test-data?role-arn=arn:aws:iam::888888888888:role/my-role'
    ```

* AWS IAMユーザーのアクセスキーを使用してS3データにアクセスする場合:

    ```bash
    ./tidb-lightning --tidb-port=4000 --pd-urls=127.0.0.1:2379 --backend=local --sorted-kv-dir=/tmp/sorted-kvs \
        -d 's3://my-bucket/test-data?access_key={my_access_key}&secret_access_key={my_secret_access_key}'
    ```

* AWS IAMロールのアクセスキーとセッショントークンの組み合わせを使用してS3データにアクセスする場合:

    ```bash
    ./tidb-lightning --tidb-port=4000 --pd-urls=127.0.0.1:2379 --backend=local --sorted-kv-dir=/tmp/sorted-kvs \
        -d 's3://my-bucket/test-data?access_key={my_access_key}&secret_access_key={my_secret_access_key}&session-token={my_session_token}'
    ```

## その他のリソース

- [ダンプリングを使用してCSVファイルにエクスポート](/dumpling-overview.md#export-to-csv-files)
- [`LOAD DATA`](https://dev.mysql.com/doc/refman/8.0/en/load-data.html)