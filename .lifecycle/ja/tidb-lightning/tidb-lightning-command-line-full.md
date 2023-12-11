---
title: TiDB Lightningのコマンドラインフラグ
summary: コマンドラインフラグを使用してTiDB Lightningを構成する方法を学びます。

# TiDB Lightningのコマンドラインフラグ

TiDB Lightningは、構成ファイルまたはコマンドラインを使用して構成することができます。このドキュメントでは、TiDB Lightningのコマンドラインフラグについて説明します。

## コマンドラインフラグ

### `tidb-lightning`

`tidb-lightning`を使用して、以下のパラメータを構成できます。

| パラメータ | 説明 | 対応する構成項目 |
| :---- | :---- | :---- |
| `--config <file>` | ファイルからグローバル構成を読み込みます。このパラメータが指定されていない場合、TiDB Lightningはデフォルトの構成を使用します。 | |
| `-V` | プログラムのバージョンを表示します。 | |
| `-d <directory>` | データファイルのローカルディレクトリまたは[外部ストレージURI](/external-storage-uri.md)。 | `mydumper.data-source-dir` |
| `-L <level>` | ログレベル：`debug`、`info`、`warn`、`error`、`fatal`。デフォルトは`info`です。 | `lightning.level` |
| `-f <rule>` | [テーブルフィルタルール](/table-filter.md)。複数回指定できます。 | `mydumper.filter` |
| `--backend <backend>` | インポートモードを選択します。`local`は[物理的なインポートモード](/tidb-lightning/tidb-lightning-physical-import-mode.md)を参照し、`tidb`は[論理的なインポートモード](/tidb-lightning/tidb-lightning-logical-import-mode.md)を参照します。 | `tikv-importer.backend` |
| `--log-file <file>` | ログファイルのパス。デフォルトでは `/tmp/lightning.log.{timestamp}` です。`-`を設定すると、ログファイルが標準出力に出力されます。 | `lightning.log-file` |
| `--status-addr <ip:port>` | TiDB Lightningサーバーのリスニングアドレス | `lightning.status-port` |
| `--pd-urls <host:port>` | PDエンドポイントアドレス | `tidb.pd-addr` |
| `--tidb-host <host>` | TiDBサーバーホスト | `tidb.host` |
| `--tidb-port <port>` | TiDBサーバーポート（デフォルト=4000） | `tidb.port` |
| `--tidb-status <port>` | TiDBステータスポート（デフォルト=10080） | `tidb.status-port` |
| `--tidb-user <user>` | TiDBに接続するユーザー名 | `tidb.user` |
| `--tidb-password <password>` | TiDBに接続するためのパスワード。パスワードは平文またはBase64でエンコードされていることができます。 | `tidb.password` |
| `--enable-checkpoint <bool>` | チェックポイントを有効にするかどうか（デフォルト=true） | `checkpoint.enable` |
| `--analyze <level>` | インポート後にテーブルを分析します。利用可能な値は "required"、"optional"（デフォルト値）、"off" です。 | `post-restore.analyze` |
| `--checksum <level>` | インポート後にチェックサムを比較します。利用可能な値は "required"（デフォルト値）、"optional"、"off" です。 | `post-restore.checksum` |
| `--check-requirements <bool>` | タスクを開始する前にクラスターバージョンの互換性をチェックし、実行中にTiKVに10%以上の空き容量があるかどうかをチェックします（デフォルト=true）。 | `lightning.check-requirements` |
| `--ca <file>` | TLS接続のためのCA証明書のパス | `security.ca-path` |
| `--cert <file>` | TLS接続のための証明書のパス | `security.cert-path` |
| `--key <file>` | TLS接続のための秘密キーのパス | `security.key-path` |
| `--server-mode` | サーバーモードでTiDB Lightningを起動します | `lightning.server-mode` |

コマンドラインパラメータと構成ファイルで対応する設定の両方を指定した場合、コマンドラインパラメータが優先されます。例えば、`./tidb-lightning -L debug --config cfg.toml` を実行すると、常にログレベルが "debug" に設定されます。

## `tidb-lightning-ctl`

`tidb-lightning`のすべてのパラメータは `tidb-lightning-ctl`にも適用されます。さらに、`tidb-lightning-ctl`を使用して以下のパラメータも構成できます。

| パラメータ | 説明 |
|:----|:----|
| `--compact` | 完全な圧縮を実行します。 |
| `--switch-mode <mode>` | 各TiKVストアを指定されたモード（通常またはインポート）に切り替えます。 |
| `--fetch-mode` | 各TiKVストアの現在のモードを表示します。 |
| `--import-engine <uuid>` | TiKVインポータから閉じたエンジンファイルをTiKVクラスタにインポートします。 |
| `--cleanup-engine <uuid>` | TiKVインポータからエンジンファイルを削除します。 |
| `--checkpoint-dump <folder>` | 現在のチェックポイントをCSV形式でフォルダにダンプします。 |
| `--checkpoint-error-destroy <table_name>` | チェックポイントを削除します。エラーが発生した場合、テーブルを削除します。 |
| `--checkpoint-error-ignore <table_name>` | 指定されたテーブルに関連するチェックポイントで記録されたエラーを無視します。 |
| `--checkpoint-remove <table_name>` | 指定されたテーブルのチェックポイントを無条件で削除します。 |

`<table_name>` は、 `` `db`.`tbl` ``（バッククォートを含む）という形式で修飾されたテーブル名、または `all` というキーワードである必要があります。
