---
title: brコマンドラインマニュアル
summary: brコマンドラインツールの説明、オプション、使用方法について学びます。

# brコマンドラインマニュアル

このドキュメントでは、`br`コマンドの定義、構成、共通オプション、`br`コマンドを使用してスナップショットのバックアップとリストア、およびログのバックアップと時間軸リカバリ（PITR）を実行する方法について説明します。

## `br`コマンドラインの説明

`br`コマンドはサブコマンド、オプション、およびパラメータで構成されています。サブコマンドは、`-`や`--`を含まない文字です。オプションは、`-`または`--`から始まる文字です。パラメータは、直後に続く文字であり、サブコマンドまたはオプションに渡されます。

以下は完全な`br`コマンドです：

```shell
br backup full --pd "${PD_IP}:2379" \
--storage "s3://backup-data/snapshot-202209081330/"
```

前述のコマンドの説明は次の通りです：

* `backup`: `br`のサブコマンドです。
* `full`: `br backup`のサブコマンドです。
* `-s`(または`--storage`): バックアップファイルが保存されているパスを指定するオプションです。`"s3://backup-data/snapshot-202209081330/"`は`-s`のパラメータです。
* `--pd`: PDサービスアドレスを指定するオプションです。`"${PD_IP}:2379"`は`--pd`のパラメータです。

### コマンドとサブコマンド

`br`コマンドには複数の階層のサブコマンドが含まれています。現在、`br`コマンドラインツールには以下のサブコマンドがあります：

* `br backup`: TiDBクラスターのデータをバックアップするために使用されます。
* `br log`: ログのバックアップタスクを開始および管理するために使用されます。
* `br restore`: TiDBクラスターのバックアップデータを復元するために使用されます。

`br backup`および`br restore`には以下のサブコマンドが含まれます：

* `full`: クラスターデータ全体をバックアップまたは復元するために使用されます。
* `db`: クラスターの指定されたデータベースをバックアップまたは復元するために使用されます。
* `table`: クラスターの指定されたデータベース内の単一のテーブルをバックアップまたは復元するために使用されます。

### 共通オプション

* `--pd`: PDサービスアドレスを指定します。例：`"${PD_IP}:2379"`。
* `-s`(または`--storage`): バックアップファイルが保存されているパスを指定します。Amazon S3、Google Cloud Storage（GCS）、Azure Blob Storage、NFSをサポートしています。詳細は、[外部ストレージサービスのURIフォーマット](/external-storage-uri.md)を参照してください。
* `--ca`: PEM形式の信頼できるCA証明書へのパスを指定します。
* `--cert`: PEM形式のSSL証明書へのパスを指定します。
* `--key`: PEM形式のSSL証明書キーへのパスを指定します。
* `--status-addr`: `br`がPrometheusに統計情報を提供するリッスンアドレスを指定します。
* `--concurrency`: バックアップまたはリストア中の並列タスク数。

## フルバックアップのコマンド

クラスターデータをバックアップするには、`br backup`コマンドを実行します。バックアップ操作の範囲を指定するために`full`または`table`サブコマンドを追加できます：クラスタ全体（`full`）または単一のテーブル（`table`）。

- [TiDBクラスタースナップショットのバックアップ](/br/br-snapshot-manual.md#back-up-cluster-snapshots)
- [データベースのバックアップ](/br/br-snapshot-manual.md#back-up-a-database)
- [テーブルのバックアップ](/br/br-snapshot-manual.md#back-up-a-table)
- [テーブルフィルタで複数のテーブルをバックアップ](/br/br-snapshot-manual.md#back-up-multiple-tables-with-table-filter)
- [スナップショットの暗号化](/br/backup-and-restore-storages.md#server-side-encryption)

## ログバックアップのコマンド

ログのバックアップタスクを開始および管理するには、`br log`コマンドを実行します。

- [ログバックアップタスクを開始](/br/br-pitr-manual.md#start-a-backup-task)
- [バックアップステータスのクエリ](/br/br-pitr-manual.md#query-the-backup-status)
- [ログバックアップタスクの一時停止および再開](/br/br-pitr-manual.md#pause-and-resume-a-backup-task)
- [ログバックアップタスクの停止および再開](/br/br-pitr-manual.md#stop-and-restart-a-backup-task)
- [バックアップデータのクリーンアップ](/br/br-pitr-manual.md#clean-up-backup-data)
- [バックアップメタデータの表示](/br/br-pitr-manual.md#view-the-backup-metadata)

## バックアップデータの復元コマンド

クラスターデータを復元するには、`br restore`コマンドを実行します。リストアの範囲を指定するために`full`、`db`、または`table`サブコマンドを追加できます：クラスタ全体（`full`）、単一のデータベース（`db`）、または単一のテーブル（`table`）。

- [時間軸リカバリ](/br/br-pitr-manual.md#restore-to-a-specified-point-in-time-pitr)
- [TiDBクラスタースナップショットのリストア](/br/br-snapshot-manual.md#restore-cluster-snapshots)
- [データベースのリストア](/br/br-snapshot-manual.md#restore-a-database)
- [テーブルのリストア](/br/br-snapshot-manual.md#restore-a-table)
- [テーブルフィルタで複数のテーブルをリストア](/br/br-snapshot-manual.md#restore-multiple-tables-with-table-filter)
- [暗号化されたスナップショットのリストア](/br/br-snapshot-manual.md#restore-encrypted-snapshots)
