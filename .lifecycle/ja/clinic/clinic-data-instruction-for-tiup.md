---
title: PingCAP Clinicの診断データ
summary: TiUPを使用して展開されたTiDBおよびDMクラスターからPingCAP Clinicの診断サービスが収集できる診断データについて学びます。

# PingCAP Clinicの診断データ

この文書では、TiUPを使用して展開されたTiDBおよびDMクラスターからPingCAP Clinicの診断サービス（PingCAP Clinic）が収集できる診断データのタイプについて説明します。また、データの種類に応じてコマンドに必要なパラメータを追加することができるよう、データ収集に対応するパラメータをリストします。[Diagクライアント（Diag）を使用してデータを収集する](/clinic/clinic-user-guide-for-tiup.md)場合、収集するデータのタイプに応じて必要なパラメータをコマンドに追加できます。

PingCAP Clinicが収集した診断データは、クラスターの問題のトラブルシューティングに**のみ**使用されます。

クラウドに展開された診断サービス、Clinic Serverはデータの保存場所に応じて2つの独立したサービスを提供します。

- [国際ユーザー向けClinic Server](https://clinic.pingcap.com): 国際ユーザー向けに収集されたデータをClinic Serverにアップロードする場合、データはPingCAPがAWS USリージョンで展開したAmazon S3サービスに保存されます。PingCAPは厳格なデータアクセスポリシーを適用し、認可されたテクニカルサポートのみがデータにアクセスできます。
- [中国本土ユーザー向けClinic Server](https://clinic.pingcap.com.cn): 中国本土ユーザー向けに収集されたデータをClinic Serverにアップロードする場合、データはPingCAPが中国（北京）リージョンで展開したAmazon S3サービスに保存されます。PingCAPは厳格なデータアクセスポリシーを適用し、認可されたテクニカルサポートのみがデータにアクセスできます。

## TiDBクラスター

このセクションでは、TiUPを使用して展開されたTiDBクラスターから[Diag](https://github.com/pingcap/diag)が収集できる診断データのタイプについてリストします。

### TiDBクラスター情報

| データタイプ | エクスポートファイル | PingCAP Clinicによるデータ収集のためのパラメータ |
| :------ | :------ |:-------- |
| クラスターIDを含むクラスターの基本情報 | `cluster.json` | デフォルトで実行ごとにデータが収集されます。 |
| クラスターの詳細情報 | `meta.yaml` | デフォルトで実行ごとにデータが収集されます。 |

### TiDB診断データ

| データタイプ | エクスポートファイル | PingCAP Clinicによるデータ収集のためのパラメータ |
| :------ | :------ |:-------- |
| ログ | `tidb.log` | `--include=log` |
| エラーログ | `tidb_stderr.log` | `--include=log` |
| 遅いクエリのログ | `tidb_slow_query.log` | `--include=log` |
| 構成ファイル | `tidb.toml` | `--include=config` |
| リアルタイム構成 | `config.json` | `--include=config` |

### TiKV診断データ

| データタイプ | エクスポートファイル | PingCAP Clinicによるデータ収集のためのパラメータ |
| :------ | :------ |:-------- |
| ログ | `tikv.log` | `--include=log` |
| エラーログ | `tikv_stderr.log` | `--include=log` |
| 構成ファイル | `tikv.toml` | `--include=config` |
| リアルタイム構成 | `config.json` | `--include=config` |

### PD診断データ

| データタイプ | エクスポートファイル | PingCAP Clinicによるデータ収集のためのパラメータ |
| :------ | :------ |:-------- |
| ログ | `pd.log` | `--include=log` |
| エラーログ | `pd_stderr.log` | `--include=log` |
| 構成ファイル | `pd.toml` | `--include=config` |
| リアルタイム構成 | `config.json` | `--include=config` |
| `tiup ctl:v<CLUSTER_VERSION> pd -u http://${pd IP}:${PORT} store`コマンドの出力 | `store.json` | `--include=config` |
| `tiup ctl:v<CLUSTER_VERSION> pd -u http://${pd IP}:${PORT} config placement-rules show`コマンドの出力 | `placement-rule.json` | `--include=config` |

### TiFlash診断データ

| データタイプ | エクスポートファイル | PingCAP Clinicによるデータ収集のためのパラメータ |
| :------ | :------ |:-------- |
| ログ | `tiflash.log` | `--include=log` |
| エラーログ | `tiflash_stderr.log` | `--include=log` |
| 構成ファイル |  `tiflash-learner.toml`, `tiflash-preprocessed.toml`, `tiflash.toml` | `--include=config` |
| リアルタイム構成 | `config.json` | `--include=config` |

### TiCDC診断データ

| データタイプ | エクスポートファイル | PingCAP Clinicによるデータ収集のためのパラメータ |
| :------ | :------ |:-------- |
| ログ | `ticdc.log` | `--include=log`|
| エラーログ | `ticdc_stderr.log` | `--include=log` |
| 構成ファイル | `ticdc.toml` | `--include=config` |
| デバッグデータ | `info.txt`, `status.txt`, `changefeeds.txt`, `captures.txt`, `processors.txt` | `--include=debug`（Diagはデフォルトでこのデータタイプを収集しません） |

### Prometheus監視データ

| データタイプ | エクスポートファイル | PingCAP Clinicによるデータ収集のためのパラメータ |
| :------ | :------ |:-------- |
| すべてのメトリクスデータ | `{metric_name}.json` | `--include=monitor` |
| すべてのアラートデータ | `alerts.json` | `--include=monitor` |

### TiDBシステム変数

| データタイプ | エクスポートファイル | PingCAP Clinicによるデータ収集のためのパラメータ |
| :------ | :------ |:-------- |
| TiDBシステム変数 | `mysql.tidb.csv` | `--include=db_vars`（Diagはデフォルトでこのデータタイプを収集しません。このデータタイプを収集する必要がある場合は、データベース認証情報が必要です） |
| | `global_variables.csv` | `--include=db_vars`（Diagはデフォルトでこのデータタイプを収集しません） |

### クラスターノードのシステム情報

| データタイプ | エクスポートファイル | PingCAP Clinicによるデータ収集のためのパラメータ |
| :------ | :------ |:-------- |
| カーネルログ | `dmesg.log` | `--include=system` |
| システムおよびハードウェアの基本情報 | `insight.json` | `--include=system` |
| `/etc/security/limits.conf`の内容 | `limits.conf` | `--include=system` |
| カーネルパラメータのリスト | `sysctl.conf` | `--include=system` |
| `ss`コマンドの出力であるソケットシステム情報 | `ss.txt` | `--include=system` |

## DMクラスター

このセクションでは、TiUPを使用して展開されたDMクラスターからDiagが収集できる診断データのタイプについてリストします。

### DMクラスター情報

| データタイプ | エクスポートファイル | PingCAP Clinicによるデータ収集のためのパラメータ |
| :------ | :------ |:-------- |
| クラスターの基本情報を含むクラスターの基本情報 | `cluster.json`| デフォルトで実行ごとにデータが収集されます。 |
| クラスターの詳細情報 | `meta.yaml` | デフォルトで実行ごとにデータが収集されます。 |

### dm-master診断データ

| データタイプ | エクスポートファイル | PingCAP Clinicによるデータ収集のためのパラメータ |
| :------ | :------ |:-------- |
| ログ | `m-master.log` | `--include=log` |
| エラーログ | `dm-master_stderr.log` | `--include=log` |
| 構成ファイル | `dm-master.toml` | `--include=config` |

### dm-worker診断データ

| データタイプ | エクスポートファイル | PingCAP Clinicによるデータ収集のためのパラメータ |
| :------ | :------ |:-------- |
| ログ | `dm-worker.log` | `--include=log`|
| エラーログ | `dm-worker_stderr.log` | `--include=log` |
| 構成ファイル | `dm-work.toml` | `--include=config` |

### Prometheus監視データ

| データタイプ | エクスポートファイル | PingCAP Clinicによるデータ収集のためのパラメータ |
| :------ | :------ |:-------- |
| すべてのメトリクスデータ | `{metric_name}.json` | `--include=monitor` |
| すべてのアラートデータ | `alerts.json` | `--include=monitor` |

### クラスターノードのシステム情報

| データタイプ | エクスポートファイル | PingCAP Clinicによるデータ収集のためのパラメータ |
| :------ | :------ |:-------- |
| カーネルログ | `dmesg.log` | `--include=system` |
| システムおよびハードウェアの基本情報 | `insight.json` | `--include=system` |
| `/etc/security/limits.conf`システムの内容 | `limits.conf` | `--include=system` |
| カーネルパラメータのリスト | `sysctl.conf` | `--include=system` |
| `ss`コマンドの出力であるソケットシステム情報 | `ss.txt` | `--include=system` |