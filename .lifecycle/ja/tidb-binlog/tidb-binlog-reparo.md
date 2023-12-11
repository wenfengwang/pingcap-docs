---
title: Reparoユーザーガイド
summary: Reparoの使用方法を学ぶ。
aliases: ['/docs/dev/tidb-binlog/tidb-binlog-reparo/','/docs/dev/reference/tidb-binlog/reparo/']
---

# Reparoユーザーガイド

ReparoはTiDB Binlogツールであり、増分データを回復するために使用されます。増分データをバックアップするには、TiDB BinlogのDrainerを使用して、バイナリ形式のデータをファイルに出力します。増分データを復元するには、Reparoを使用してファイル内のバイナリデータを解析し、TiDB/MySQLでバイナリログを適用します。

Reparoインストールパッケージ（`reparo`）はTiDB Toolkitに含まれています。TiDB Toolkitをダウンロードするには、[TiDBツールのダウンロード](/download-ecosystem-tools.md)を参照してください。

## Reparoの使用方法

### コマンドラインパラメータの説明

```
Reparoの使用法:
-L string
    ログの出力情報レベル
    値: "debug"/"info"/"warn"/"error"/"fatal"（デフォルトは"info"）
-V バージョンを表示する。
-c int
    レプリケーションプロセスの下流での同時実行数(`16`がデフォルト)。より高い値はレプリケーションのスループットを向上させます。
-config string
    設定ファイルのパス
    設定ファイルが指定された場合、Reparoはこのファイル内の設定データを読み込みます。
    コマンドラインパラメータ内に設定データが存在する場合、Reparoは設定ファイル内の設定データを上書きします。
-data-dir string
    Drainerがprotobuf形式で出力するバイナリログファイルのストレージディレクトリ（デフォルトは"data.drainer")
-dest-type string
    下流サービスのタイプ
    値: "print"/"mysql"（デフォルトは"print"）
    "print"に設定されると、データは解析され標準出力に表示されますが、SQLステートメントは実行されません。
    "mysql"に設定されると、設定ファイル内で"host"、"port"、"user"、"password"情報を構成する必要があります。
-log-file string
    ログファイルのパス
-log-rotate string
    ログファイルの切り替え頻度
    値: "hour"/"day"
-start-datetime string
    回復を開始する時点の時刻を指定します。
    フォーマット: "2006-01-02 15:04:05"
    未設定の場合、回復プロセスは最も早いバイナリログファイルから開始します。
-stop-datetime string
    回復プロセスの終了時点の時刻を指定します。
    フォーマット: "2006-01-02 15:04:05"
    未設定の場合、回復プロセスは最後のバイナリログファイルで終了します。
-safe-mode bool
    セーフモードを有効にするかどうかを指定します。有効にすると、繰り返しレプリケーションがサポートされます。
-txn-batch int
    下流データベースに出力されるトランザクション内のSQLステートメントの数（デフォルトは`20`）。
```

### 設定ファイルの説明

```toml
# Drainerが出力するprotobuf形式のバイナリログファイルのストレージディレクトリ
data-dir = "./data.drainer"

# ログの出力情報レベル
# 値: "debug"/"info"/"warn"/"error"/"fatal"（デフォルトは"info"）
log-level = "info"

# `start-datetime`と`stop-datetime`を使用して、回復するバイナリログファイルの時間範囲を指定します。
# フォーマット: "2006-01-02 15:04:05"
# start-datetime = ""
# stop-datetime = ""

# それぞれ`start-datetime`と`stop-datetime`に対応します。
# `start-datetime`と`stop-datetime`を設定すると、`start-tso`と`stop-tso`を設定する必要はありません。
# フルリカバリを行うか増分リカバリを再開する場合、それぞれstart-tsoをtso+1またはstop-tso+1に設定してください。
# start-tso = 0
# stop-tso = 0

# 下流サービスのタイプ
# 値: "print"/"mysql"（デフォルトは"print"）
# "print"に設定すると、データは解析され標準出力に表示されますが、SQLステートメントは実行されません。
# "mysql"に設定すると、設定情報の[dest-db]内で`host`、`port`、`user`、`password`情報を指定する必要があります。
dest-type = "mysql"

# 下流データベースに出力されるトランザクション内のSQLステートメントの数（デフォルトは`20`）。
txn-batch = 20

# レプリケーションプロセスの下流での同時実行数（デフォルトは`16`）。より高い値はレプリケーションのスループットを向上させます。
worker-count = 16

# セーフモードの設定
# 値: "true"/"false"（デフォルトは"false"）
# "true"に設定すると、Reparoは`UPDATE`ステートメントを`DELETE`ステートメントと`REPLACE`ステートメントに分割します。
safe-mode = false

# `replicate-do-db`と`replicate-do-table`は回復するデータベースとテーブルを指定します。
# `replicate-do-db`は`replicate-do-table`よりも優先されます。
# 構成には正規表現が使用できます。正規表現は"~"で始まる必要があります。
# `replicate-do-db`と`replicate-do-table`の構成方法は、Drainerの`replicate-do-db`および`replicate-do-table`と同じです。
# replicate-do-db = ["~^b.*","s1"]
# [[replicate-do-table]]
# db-name ="test"
# tbl-name = "log"
# [[replicate-do-table]]
# db-name ="test"
# tbl-name = "~^a.*"

# `dest-type`が`mysql`に設定されている場合、`dest-db`を構成する必要があります。
[dest-db]
host = "127.0.0.1"
port = 3309
user = "root"
password = ""
```

### 開始の例

```
./reparo -config reparo.toml
```

> **注意:**
>
> * `data-dir`はDrainerが出力するバイナリログファイルのディレクトリを指定します。
> * `start-datatime`と`start-tso`はともに回復開始時点の時刻を指定しますが、時刻の形式が異なります。未設定の場合、回復プロセスはデフォルトで最も早いバイナリログファイルから開始します。
> * `stop-datetime`と`stop-tso`はともに回復終了時点の時刻を指定しますが、時刻の形式が異なります。未設定の場合、回復プロセスはデフォルトで最後のバイナリログファイルで終了します。
> * `dest-type`は宛先タイプを指定します。その値は"mysql"および"print"です。
>
>     * `mysql`に設定されている場合、データはMySQLまたはMySQLプロトコルを使用するTiDBに回復できます。その場合、設定情報の`[dest-db]`内でデータベースの情報を指定する必要があります。
>     * `print`に設定されている場合、バイナリログ情報のみが表示されます。通常はデバッグとバイナリログ情報のチェックに使用されます。その場合、`[dest-db]`を指定する必要はありません。
>
> * `replicate-do-db`は回復対象のデータベースを指定します。未設定の場合、全てのデータベースが回復対象になります。
> * `replicate-do-table`は回復対象のテーブルを指定します。未設定の場合、全てのテーブルが回復対象になります。