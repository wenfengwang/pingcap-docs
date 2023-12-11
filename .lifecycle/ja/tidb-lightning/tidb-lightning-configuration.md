---
title: TiDBライトニング構成
summary: TiDBライトニングのCLIの使用法とサンプル構成について学びます。
aliases: ['/docs/dev/tidb-lightning/tidb-lightning-configuration/','/docs/dev/reference/tools/tidb-lightning/config/']
---

# TiDBライトニング構成

このドキュメントでは、グローバル構成とタスク構成のサンプルを提供し、コマンドラインパラメータの使用方法を説明しています。

## 構成ファイル

TiDBライトニングには「グローバル」と「タスク」の2つの構成クラスがあり、互換性のある構造を持っています。サーバーモードが有効になっていない場合（デフォルト）、TiDBライトニングは1つのタスクのみを実行し、グローバル構成とタスク構成には同じ構成ファイルが使用されます。

### TiDBライトニング（グローバル）

```toml
### tidb-lightning global configuration

[lightning]
# Webインタフェースの表示、Prometheusメトリクスの取得、デバッグデータの公開、importタスクの送信（サーバーモードで）のためのHTTPポートです。0に設定するとポートは無効になります。
status-addr = ':8289'

# サーバーモード。デフォルトはfalseで、これはコマンドを実行した直後にimportタスクが開始されます。
# この値をtrueに設定すると、コマンドを実行した後、TiDBライトニングはWebインタフェースでimportタスクを送信するまで待機します。
# 詳細については、「TiDBライトニングWebインタフェース」セクションを参照してください。
server-mode = false

# ロギング
level = "info"
file = "tidb-lightning.log"
max-size = 128 # MB
max-days = 28
max-backups = 14

# 診断ログを有効にするかどうかを制御します。デフォルト値はfalseで、つまりimportに関連するログのみが出力され、その他の依存コンポーネントのログは出力されません。
# trueに設定すると、importプロセスとその他の依存コンポーネントのログが出力され、GRPCデバッグが有効になり、診断に使用できます。
# このパラメータはv7.3.0で導入されました。
enable-diagnose-logs = false
```

### TiDBライトニング（タスク）

```toml
### tidb-lightning task configuration

[lightning]
# クラスターがタスクを開始する前に最小要件を満たしているかどうかを確認し、実行中にTiKVに十分な空き容量があるかどうかを確認します。
#check-requirements = true

# 同時に開かれるエンジンの最大数です。
# 各テーブルは1つの「インデックスエンジン」（索引を保存する）と複数の「データエンジン」（行データを保存する）に分割されます。これらの設定は、各タイプのエンジンに対する最大同時数を制御します。通常、以下の2つのデフォルト値を使用できます。
index-concurrency = 2
table-concurrency = 6

# データの同時数です。デフォルトでは論理CPUコア数に設定されます。他のコンポーネントと一緒に展開する場合は、CPU使用率を制限するために論理CPUコアのサイズの75%に設定できます。
# region-concurrency =

# 最大I/O同時性です。過剰なI/O同時性はディスクの内部バッファが頻繁にリフレッシュされるため、I/Oの遅延が増加し、キャッシュミスが発生して読み取り速度が遅くなります。ストレージメディアによっては、最適なパフォーマンスのためにこの値を調整する必要があるかもしれません。
io-concurrency = 5

# TiDBライトニングを停止する前に許容する非致命的エラーの最大数です。
# 非致命的エラーは、いくつかの行に局在しており、これらの行を無視することでimportプロセスを継続できます。
# これをNに設定すると、（N+1）番目のエラーが発生した時点でTiDBライトニングはできるだけ早く停止します。
# スキップされた行は、ターゲットTiDB内の「タスク情報」スキーマ内のテーブルに挿入されます。以下に設定できるデフォルト値は「MaxInt64」バイトで、つまり、9223372036854775807バイトです。
max-error = 0
# task-info-schema-nameは、TiDBライトニングの実行結果を保存するスキーマまたはデータベースの名前です。
# エラー記録を無効にするには、これを空の文字列に設定します。
# task-info-schema-name = 'lightning_task_info'

# 並行インポートモードでは、ターゲットクラスター内の各TiDBライトニングインスタンスのメタ情報を保存するスキーマの名前です。
# デフォルト値は「lightning_metadata」です。
# このパラメータは並行インポートが有効になっている場合にのみ構成してください。
# **注意:**
# - このパラメータに設定する値は、同じ並行インポートに参加する各TiDBライトニングインスタンスで同じでなければならないため、インポートされたデータの正確性が保証されません。
# - 並行インポートモードが有効な場合は、インポートに使用されるユーザー（tidb.user構成用）が、この構成に対応するデータベースを作成およびアクセスするための権限を持っていることを確認してください。
# - TiDBライトニングは、インポートが完了した後にこのスキーマを削除します。したがって、既存のスキーマ名をこのパラメータで構成しないでください。
meta-schema-name = "lightning_metadata"

[security]
# クラスター内のTLS接続の証明書とキーを指定します。
# CAの公開証明書。無効にするには空にしてください。
# ca-path = "/path/to/ca.pem"
# このサービスの公開証明書。
# cert-path = "/path/to/lightning.pem"
# このサービスの秘密鍵。
# key-path = "/path/to/lightning.key"

[checkpoint]
# チェックポイントを有効にするかどうかを指定します。
# データをインポートする間、TiDBライトニングはどのテーブルがインポートされたかを記録します。そのため、TiDBライトニングまたは他のコンポーネントがクラッシュしても、ゼロからではなく既知の健全な状態から開始できます。
enable = true
# チェックポイントを格納するスキーマ（データベース名）です。
schema = "tidb_lightning_checkpoint"
# チェックポイントを格納する場所です。
#  - file: ローカルファイルとして格納。
#  - mysql: リモートのMySQL互換データベースに格納
driver = "file"
# チェックポイントの格納場所を示すデータソース名（DSN）です。
# 「file」ドライバの場合、DSNはパスです。パスが指定されていない場合、TiDBライトニングはデフォルトで「/tmp/CHECKPOINT_SCHEMA.pb」を使用します。
# 「mysql」ドライバの場合、DSNは「USER:PASS@tcp(HOST:PORT)/」形式のURLです。
# URLが指定されていない場合、[tidb]セクションからTiDBサーバーがチェックポイントを格納するために使用されます。ターゲットTiDBクラスターの負荷を軽減するために異なるMySQL互換データベースサーバーを指定する必要があります。
# dsn = "/tmp/tidb_lightning_checkpoint.pb"
# データがすべてインポートされた後にチェックポイントを保持するかどうかを指定します。falseの場合、チェックポイントは削除されます。
# チェックポイントを保持するとデバッグが容易になりますが、データソースに関するメタデータが漏れます。
# keep-after-success = false

[conflict]
# v7.3.0からは競合データを処理するための新しいバージョンの戦略が導入されます。デフォルト値は""です。
# - "": TiDBライトニングは競合データを検出または処理しません。ソースファイルに競合するプライマリやユニークキーレコードが含まれている場合、後続のステップはエラーを報告します。
# - "error": インポートされたデータ内で競合するプライマリやユニークキーレコードを検出した場合、TiDBライトニングはインポートを中止してエラーを報告します。
# - "replace": 競合するプライマリやユニークキーレコードに遭遇した場合、新しいデータを保持し、古いデータを上書きします。
# - "ignore": 競合するプライマリやユニークキーレコードに遭遇した場合、TiDBライトニングは古いデータを保持し、新しいデータを無視します。
# 新しいバージョンの戦略はtikv-importer.duplicate-resolution（競合検出の旧バージョン）とは一緒に使用できません。
strategy = ""
# 戦略が「replace」または「ignore」の場合に処理できる競合データの上限を制御します。これは「replace」または「ignore」の場合のみ設定できます。デフォルト値は9223372036854775807で、ほぼすべてのエラーが許容されます。
# threshold = 9223372036854775807
# conflict_recordsテーブル内の最大レコード数を制御します。デフォルト値は100です。戦略が「ignore」の場合、無視された競合レコードが記録されます。戦略が「replace」の場合、上書きされた競合レコードが記録されます。ただし、論理インポートモードでは競合レコードは記録できません。
# max-record-rows = 100

[tikv-importer]
# 「local」: デフォルトで使用される物理インポートモード。1TiBを超える大規模データセットのインポートに適用されますが、インポート中は下流のTiDBがサービスを提供できません。
# 「tidb」: 論理インポートモード。1TiB未満の小規模データセットのインポートにこのモードを使用できます。インポート中、下流のTiDBがサービスを提供できます。
# backend = "local"
# 複数のTiDBライトニングインスタンスが（物理インポートモードで）1つ以上のターゲットテーブルにデータを並行してインポートするかどうかを指定します。
# デフォルト値は`false`です。
# 並行インポートモードを使用する場合は、パラメータを`true`に設定する必要がありますが、前提条件としてターゲットテーブルにデータが存在しないこと、つまりすべてのデータがTiDBライトニングによってのみインポートできることです。
# このパラメータは、ターゲットテーブルが空であるシナリオにのみ使用されます。
# parallel-import = false

# 物理インポートモードで重複するレコード（一意キーの競合）を検出して解決するかどうかを指定します。
# 次の解決アルゴリズムがサポートされています:
#  - none: 重複レコードを検出しません。2つのアルゴリズムの中で最高のパフォーマンスです。
```yaml
      + 私は専門の技術翻訳者であり、英語と日本語の両方で堪能です。
      + 私は専門の技術翻訳者であり、英語と日本語の両方で堪能です。
    + 私は専門の技術翻訳者であり、英語と日本語の両方で堪能です。
  + 私は専門の技術翻訳者であり、英語と日本語の両方で堪能です。
```
```yaml
strict-format = false

max-region-size = "256MiB"

filter = ['*.*', '!mysql.*', '!sys.*', '!INFORMATION_SCHEMA.*', '!PERFORMANCE_SCHEMA.*', '!METRICS_SCHEMA.*', '!INSPECTION_SCHEMA.*']

[mydumper.csv]
separator = ','
delimiter = '"'
terminator = ''
header = true
header-schema-match = true
not-null = false
null = '\N'
backslash-escape = true
trim-last-separator = false

[tidb]
host = "172.16.31.1"
port = 4000
user = "root"
password = ""
status-port = 10080
pd-addr = "172.16.31.4:2379"
log-level = "error"
build-stats-concurrency = 20
distsql-scan-concurrency = 15
index-serial-scan-concurrency = 20
checksum-table-concurrency = 2
sql-mode = "ONLY_FULL_GROUP_BY,NO_ENGINE_SUBSTITUTION"
max-allowed-packet = 67_108_864
tls = ""

[post-restore]
checksum = "required"
checksum-via-sql = "false"
analyze = "optional"

[cron]
switch-mode = "5m"
log-progress = "5m"
```
# check-disk-quota = "60s"
```

## コマンドラインパラメータ

### `tidb-lightning`の使用法

| パラメータ | 説明 | 対応する設定 |
|:----|:----|:----|
| --config *file* | *file* からグローバル設定を読み込みます。指定されていない場合、デフォルトの設定が使用されます。 | |
| -V | プログラムのバージョンを表示します | |
| -d *directory* | データのダンプ元のディレクトリまたは[外部ストレージURI](/external-storage-uri.md) | `mydumper.data-source-dir` |
| -L *level* | ログレベル: debug, info, warn, error, fatal (デフォルト = info) | `lightning.log-level` |
| -f *rule* | [テーブルフィルタールール](/table-filter.md) (複数回指定可能) | `mydumper.filter` |
| --backend *[backend](/tidb-lightning/tidb-lightning-overview.md)* | インポートモードを選択します。 `local` は物理インポートモードを、 `tidb` は論理インポートモードを指します。 | `local` |
| --log-file *file* | ログファイルのパス。デフォルトでは、`/tmp/lightning.log.{timestamp}` です。 '-' に設定すると、ログファイルは標準出力に出力されます。 | `lightning.log-file` |
| --status-addr *ip:port* | TiDB Lightningサーバーのリスニングアドレス | `lightning.status-port` |
| --pd-urls *host:port* | PDエンドポイントアドレス | `tidb.pd-addr` |
| --tidb-host *host* | TiDBサーバーホスト | `tidb.host` |
| --tidb-port *port* | TiDBサーバーポート (デフォルト = 4000) | `tidb.port` |
| --tidb-status *port* | TiDBステータスポート (デフォルト = 10080) | `tidb.status-port` |
| --tidb-user *user* | TiDBに接続するユーザー名 | `tidb.user` |
| --tidb-password *password* | TiDBに接続するためのパスワード。パスワードは平文またはBase64でエンコードされている必要があります。 | `tidb.password` |
| --enable-checkpoint *bool* | チェックポイントを有効にするかどうか (デフォルト = true) | `checkpoint.enable` |
| --analyze *level* | インポート後にテーブルを分析します。使用可能な値は "required"、"optional" (デフォルト値)、"off" です | `post-restore.analyze` |
| --checksum *level* | インポート後にチェックサムを比較します。使用可能な値は "required" (デフォルト値)、"optional"、"off" です | `post-restore.checksum` |
| --check-requirements *bool* | タスクを開始する前にクラスターバージョンの互換性をチェックし、実行中にTiKVに10%以上の空き容量があるかどうかを確認します。 (デフォルト = true) | `lightning.check-requirements` |
| --ca *file* | TLS接続のCA証明書パス | `security.ca-path` |
| --cert *file* | TLS接続の証明書パス | `security.cert-path` |
| --key *file* | TLS接続の秘密鍵パス | `security.key-path` |
| --server-mode | TiDB Lightningをサーバーモードで起動します | `lightning.server-mode` |

コマンドラインパラメータと構成ファイル内の対応する設定が両方提供されている場合、コマンドラインパラメータが使用されます。たとえば、`./tidb-lightning -L debug --config cfg.toml` を実行すると、常にログレベルが "debug" に設定されますが、`cfg.toml` の内容に関係なく。

## `tidb-lightning-ctl`の使用法

このツールは、次のパラメータのいずれかを指定することで、さまざまなアクションを実行できます:

| パラメータ | 説明 |
|:----|:----|
| --compact | フルコンパクションを実行します |
| --switch-mode *mode* | 各TiKVストアを指定のモードに切り替えます: normal、import |
| --fetch-mode | 各TiKVストアの現在のモードを表示します |
| --import-engine *uuid* | TiKVインポータから閉じたエンジンファイルをTiKVクラスターにインポートします |
| --cleanup-engine *uuid* | TiKVインポータからエンジンファイルを削除します |
| --checkpoint-dump *folder* | 現在のチェックポイントをCSV形式で指定のフォルダにダンプします |
| --checkpoint-error-destroy *tablename* | エラーが発生した場合、チェックポイントを削除し、テーブルを削除します |
| --checkpoint-error-ignore *tablename* | チェックポイントに記録された任意のエラーを無視します（指定されたテーブルに関連するエラー） |
| --checkpoint-remove *tablename* | テーブルのチェックポイントを無条件で削除します |

*tablename* は、`` `db`.`tbl` ``（バッククォートを含む）の形式で修飾されたテーブル名、または「all」というキーワードである必要があります。

また、上記セクションで説明されている `tidb-lightning` のすべてのパラメータは、 `tidb-lightning-ctl` でも有効です。