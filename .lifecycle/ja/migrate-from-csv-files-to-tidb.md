---
title: TiDB へのCSVファイルからのデータ移行
summary: CSVファイルからTiDBへのデータ移行方法について学びます。
aliases: ['/docs/dev/tidb-lightning/migrate-from-csv-using-tidb-lightning/','/docs/dev/reference/tools/tidb-lightning/csv/','/tidb/dev/migrate-from-csv-using-tidb-lightning']
---

# TiDB へのCSVファイルからのデータ移行

このドキュメントでは、CSVファイルからTiDBへのデータ移行方法について説明します。

TiDB Lightning は、CSVファイルやタブ区切り値（TSV）などのデリミタ形式からデータを読み取ることができます。その他のフラットファイルデータソースについても、このドキュメントを参照してTiDB にデータを移行できます。

## 前提条件

- [TiDB Lightning をインストールする](/migration-tools.md)。
- [TiDB Lightning に必要なターゲットデータベースの権限を取得する](/tidb-lightning/tidb-lightning-requirements.md#privileges-of-the-target-database)。

## ステップ1. CSVファイルの準備

すべてのCSVファイルを同じディレクトリに配置します。TiDB Lightning にすべてのCSVファイルを認識させる必要がある場合は、ファイル名は以下の要件を満たす必要があります：

- CSVファイルが1つのテーブルのデータを含む場合は、ファイル名を`${db_name}.${table_name}.csv`とします。
- 1つのテーブルのデータが複数のCSVファイルに分かれている場合は、これらのCSVファイルに数値のサフィックスを追加します。例：`${db_name}.${table_name}.003.csv`。 数値のサフィックスは連続している必要はなく、昇順である必要があります。また、すべてのサフィックスが同じ長さになるように数値の前に余分なゼロを追加する必要があります。

## ステップ2. ターゲットテーブルスキーマの作成

CSVファイルにはスキーマ情報が含まれていないため、CSVファイルからTiDBにデータをインポートする前に、ターゲットテーブルスキーマを作成する必要があります。以下の2つの方法のいずれかでターゲットテーブルスキーマを作成することができます：

* **方法1**：TiDB Lightning を使用してターゲットテーブルスキーマを作成する。

    必要なDDLステートメントを含むSQLファイルを作成します:

    - `${db_name}-schema-create.sql`ファイルに`CREATE DATABASE`ステートメントを追加します。
    - `${db_name}.${table_name}-schema.sql`ファイルに`CREATE TABLE`ステートメントを追加します。

* **方法2**：手動でターゲットテーブルスキーマを作成する。

## ステップ3. 構成ファイルの作成

以下の内容で`tidb-lightning.toml`ファイルを作成します：

```toml
[lightning]
# ログ
level = "info"
file = "tidb-lightning.log"

[tikv-importer]
# "local": デフォルトバックエンド。大容量のデータ（1 TiB以上）をインポートする場合は、ローカルバックエンドを推奨します。インポート中、ターゲットのTiDBクラスタはどんなサービスも提供できません。
# "tidb"： データが1 TiB未満の場合は "tidb" バックエンドを推奨します。インポート中、ターゲットのTiDBクラスタは通常サービスを提供できます。
# インポートモードの詳細については、<https://docs.pingcap.com/tidb/stable/tidb-lightning-overview#tidb-lightning-architecture>を参照してください。
backend = "local"
# ソートされたキー値ファイルの一時保存ディレクトリを設定します。ディレクトリは空であり、ストレージ容量がインポート対象データのサイズよりも大きい必要があります。インポートのパフォーマンス向上のためには、`data-source-dir`と異なるディレクトリを使用し、I/Oを排他的に使用できるフラッシュストレージを使用することを推奨します。
sorted-kv-dir = "/mnt/ssd/sorted-kv-dir"

[mydumper]
# データソースのディレクトリ。
data-source-dir = "${data-path}" # ローカルパスまたはS3パス。例：'s3://my-bucket/sql-backup'。

# CSV形式の設定。
[mydumper.csv]
# CSVファイルのフィールド区切り文字。空であってはなりません。ソースファイルに、バイナリ、BLOB、またはビットなどの文字列と数値以外のフィールドが含まれている場合は、","などのシンプルな区切り文字を使用せず、代わりに"|+|"などの一般的でない文字の組み合わせを使用することをお勧めします。
separator = ','
# 区切り文字。ゼロ個以上の文字を指定できます。
delimiter = '"'
# CSVファイルにテーブルヘッダが含まれているかどうかを設定します。
# この項目がtrueに設定されている場合、TiDB Lightning はCSVファイルの最初の行を使用してフィールドの関係を解析します。
header = true
# CSVファイルにNULLが含まれているかどうかを設定します。
# この項目がtrueに設定されている場合、CSVファイルの任意の列はNULLとして解析することができません。
not-null = false
# `not-null`がfalseに設定されている場合（CSVにNULLが含まれている場合），
# 以下の値がNULLとして解析されます。
null = '\N'
# 文字列内のバックスラッシュ ('\') をエスケープ文字として扱うかどうかを設定します。
backslash-escape = true
# 各行の最後の区切り文字をトリムするかどうかを設定します。
trim-last-separator = false

[tidb]
# ターゲットクラスタ。
host = ${host}            # 例：172.16.32.1
port = ${port}            # 例：4000
user = "${user_name}"     # 例： "root"
password = "${password}"  # 例： "rootroot"
status-port = ${status-port} # インポート中、TiDB Lightning はTiDB ステータスポートからテーブルスキーマ情報を取得する必要があります。例：10080
pd-addr = "${ip}:${port}" # PDクラスタのアドレス。例：172.16.31.3:2379。TiDB Lightning はPDからいくつかの情報を取得します。backend = "local"の場合、status-portとpd-addrを正しく指定する必要があります。そうでない場合、インポートは異常終了します。
```

構成ファイルの詳細については、[TiDB Lightning 構成](/tidb-lightning/tidb-lightning-configuration.md)を参照してください。

## ステップ4. インポートパフォーマンスの調整（オプション）

CSVファイルから256 MiB程度の一様なサイズでデータをインポートする場合、TiDB Lightning は最良のパフォーマンスで動作します。ただし、単一の大きなCSVファイルからデータをインポートする場合、TiDB Lightning はデフォルトで1つのスレッドしか使用できず、インポート速度が遅くなる可能性があります。

インポート速度を向上させるには、大きなCSVファイルを複数の小さなファイルに分割することができます。一般的な形式のCSVファイルの場合、TiDB Lightning はデフォルトでは全体のファイルを読み取る前に各行の開始位置と終了位置を素早く特定することができないため、CSVファイルを自動的に分割しません。ただし、インポートするCSVファイルが特定の形式要件を満たしている場合、`strict-format` モードを有効にすることができます。このモードでは、TiDB Lightning は単一の大きなCSVファイルをおおよそ256 MiBの複数のファイルに自動的に分割し、並行処理を行います。

> **メモ：**
>
> CSVファイルがstrictフォーマットでない場合でも、誤って `strict-format` モードが `true` に設定されている場合、複数行にわたるフィールドが2つのフィールドに分割されます。これによりパースに失敗し、TiDB Lightning はエラーを報告せずに壊れたデータをインポートする可能性があります。

strictフォーマットのCSVファイルでは、各フィールドが1行を占有します。以下の要件を満たす必要があります：

- 区切り文字が空である。
- 各フィールドに CR（`\r`）または LF（`\n`）が含まれていない。

CSVファイルが上記の要件を満たす場合、以下のようにして `strict-format` モードを有効にすることでインポートを高速化することができます：
```toml
[mydumper]
strict-format = true
```

## ステップ5. データのインポート

インポートを開始するには、`tidb-lightning` を実行します。コマンドラインでプログラムを起動すると、プロセスは予期せず終了する場合がありますが、`nohup` や `screen` ツールを使用してプログラムを実行することをお勧めします。例：

{{< copyable "shell-regular" >}}

```shell
nohup tiup tidb-lightning -config tidb-lightning.toml > nohup.out 2>&1 &
```

インポートが開始されると、以下のいずれかの方法でインポートの進捗状況を確認することができます：

- ログでキーワード `progress` を `grep` します。進捗はデフォルトで5分ごとに更新されます。
- [監視ダッシュボード](/tidb-lightning/monitor-tidb-lightning.md)で進捗状況を確認します。
- [TiDB Lightning ウェブインターフェイス](/tidb-lightning/tidb-lightning-web-interface.md)で進捗状況を確認します。

TiDB Lightning がインポートを完了したら、プログラムは自動的に終了します。`tidb-lightning.log` に最後の行に `the whole procedure completed` という記述があるかどうかを確認します。あれば、インポートは成功です。なければ、インポートがエラーに遭遇したことになります。エラーメッセージの指示に従ってエラーを修正します。

> **メモ：**
>
> インポートが成功したかどうかに関係なく、ログの最後の行に `tidb lightning exit` という記述が表示されます。これは、TiDB Lightning が正常に終了したことを意味しますが、インポートが成功したことを必ずしも意味しません。

インポートに失敗する場合は、トラブルシューティングのために [TiDB Lightning FAQ](/tidb-lightning/tidb-lightning-faq.md) を参照してください。

## その他のファイル形式

データソースが他の形式の場合は、データソース名を `.csv` で終了させ、`tidb-lightning.toml` 構成ファイルの `[mydumper.csv]` セクションで対応する変更を行う必要があります。一般的な形式の例:

**TSV:**

```toml
# 形式の例
# ID    Region    Count```
```toml
# 1     East      32
# 2     South     NULL
# 3     West      10
# 4     North     39

# フォーマット設定
[mydumper.csv]
separator = "\t"
delimiter = ''
header = true
not-null = false
null = 'NULL'
backslash-escape = false
trim-last-separator = false
```

**TPC-H DBGEN:**

```toml
# フォーマット例
# 1|East|32|
# 2|South|0|
# 3|West|10|
# 4|North|39|

# フォーマット設定
[mydumper.csv]
separator = '|'
delimiter = ''
header = false
not-null = true
backslash-escape = false
trim-last-separator = true
```

## 次は何ですか

- [CSVサポートと制限事項](/tidb-lightning/tidb-lightning-data-source.md#csv).
```