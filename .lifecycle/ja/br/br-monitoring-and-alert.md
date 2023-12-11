---
title: バックアップとリストアの監視とアラート
summary: バックアップとリストア機能の監視とアラートについて学びます。

# バックアップとリストアの監視とアラート

このドキュメントでは、バックアップとリストア機能の監視とアラートについて、監視コンポーネントの展開方法、監視メトリクス、一般的なアラートについて説明します。

## ログバックアップの監視

ログバックアップは、[Prometheus](https://prometheus.io/)を使用して監視メトリクスを収集できます。現在、すべての監視メトリクスはTiKVに組み込まれています。

### 監視構成

- TiUPを使用して展開されたクラスターの場合、Prometheusは自動的に監視メトリクスを収集します。
- 手動で展開されたクラスターの場合、[TiDBクラスターの監視展開](/deploy-monitoring-services.md)の手順に従って、Prometheus設定ファイルの`srape_configs`セクションにTiKV関連のジョブを追加してください。

### Grafana構成

- TiUPを使用して展開されたクラスターの場合、[Grafana](https://grafana.com/)ダッシュボードには、時間指定のリカバリ（PITR）パネルが含まれています。TiKV-Detailsダッシュボードの**Backup Log**パネルがPITRパネルです。
- 手動で展開されたクラスターの場合、[Grafanaダッシュボードのインポート](/deploy-monitoring-services.md#step-2-import-a-grafana-dashboard)を参照し、[tikv_details](https://github.com/tikv/tikv/blob/master/metrics/grafana/tikv_details.json) JSONファイルをGrafanaにアップロードしてください。その後、TiKV-Detailsダッシュボードで**Backup Log**パネルを見つけてください。

### 監視メトリクス

| メトリクス                                                  | タイプ    |  説明
|-------------------------------------------------------|-----------|---------------------------------------------------------------------------------------------------------------------------------------------------------|
| **tikv_log_backup_interal_actor_acting_duration_sec** | ヒストグラム| すべての内部メッセージとイベントの処理時間。<br/>`message :: TaskType`                                                                                                            |
| **tikv_log_backup_initial_scan_reason**               | カウンタ   | 初回スキャンがトリガーされる理由の統計。主な理由はリーダーの切り替えまたはリージョンのバージョンの変更。<br/> `reason :: {"leader-changed", "region-changed", "retry"}`                                           |
| **tikv_log_backup_event_handle_duration_sec**         | ヒストグラム| KVイベントの処理時間。 `tikv_log_backup_on_event_duration_seconds`と比較して、このメトリクスには内部変換の処理時間も含まれます。<br/> `stage :: {"to_stream_event", "save_to_temp_file"}` |
| **tikv_log_backup_handle_kv_batch**                   | ヒストグラム| Raftstoreによって送信されたKVペアのバッチのサイズのリージョンレベルの統計。                                                                                                    |
| **tikv_log_backup_initial_scan_disk_read**            | カウンタ   | 初回スキャン中にディスクから読み取られたデータのサイズ。Linuxでは、この情報はprocfsから取得され、実際にブロックデバイスから読み取られたデータのサイズです。このメトリクスには`initial-scan-rate-limit`構成項目が適用されます。                                   |
| **tikv_log_backup_incremental_scan_bytes**            | ヒストグラム| 初回スキャン中に実際に生成されたKVペアのサイズ。圧縮と読み取り増幅のため、この値は`tikv_log_backup_initial_scan_disk_read`の値と異なる場合があります。                                                                     |
| **tikv_log_backup_skip_kv_count**                     | カウンタ   | バックアップ時にスキップされたRaftイベントの数。                                                                                                                   |
| **tikv_log_backup_errors**                            | カウンタ   | バックアップ中に再試行または無視できるエラー。 <br/>`type :: ErrorType`                                                                                                       |
| **tikv_log_backup_fatal_errors**                      | カウンタ   | バックアップ中に再試行や無視できないエラー。このタイプのエラーが発生すると、ログバックアップは一時停止されます。<br/>`type :: ErrorType`                                                                                   |
| **tikv_log_backup_heap_memory**                       | ゲージ    | ログバックアップ中に初回スキャンで検出された未処理のイベントによって占有されたメモリ量。                                                                                                                         |
| **tikv_log_backup_on_event_duration_seconds**         | ヒストグラム| KVイベントを一時ファイルに保存する時間。 <br/>`stage :: {"write_to_tempfile", "syscall_write"}`                                                                        |
| **tikv_log_backup_store_checkpoint_ts**               | ゲージ    | 不要になったストアレベルのチェックポイントTS。現在のストアによって登録されたGC安全ポイントに近いです。<br/>`task :: string`                                                                    |
| **tidb_log_backup_last_checkpoint**                    | ゲージ    | グローバルチェックポイントTS。これは、バックアップが行われたログデータの時点を示します。<br/>`task :: string` |
| **tikv_log_backup_flush_duration_sec**                | ヒストグラム| ローカルの一時ファイルを外部ストレージに移動する時間。 <br/>`stage :: {"generate_metadata", "save_files", "clear_temp_files"}`                                                                |
| **tikv_log_backup_flush_file_size**                   | ヒストグラム| バックアップ中に生成されたファイルのサイズの統計。                                                                                                                                          |
| **tikv_log_backup_initial_scan_duration_sec**         | ヒストグラム| 初回スキャンの全体的な処理時間の統計。                                                                                                                                           |
| **tikv_log_backup_skip_retry_observe**                | カウンタ   | ログバックアップ中に無視できるエラーの統計、または再試行がスキップされる理由の統計。 <br/>`reason :: {"region-absent", "not-leader", "stale-command"}`                                                   |
| **tikv_log_backup_initial_scan_operations**           | カウンタ   | 初回スキャン中のRocksDB関連操作の統計。 <br/>`cf :: {"default", "write", "lock"}, op :: RocksDBOP`                                                                       |
| **tikv_log_backup_enabled**                           | カウンタ   | ログバックアップの有効化。値が`0`より大きい場合、バックアップが有効になります。                                                                                                                                |
| **tikv_log_backup_observed_region**                   | ゲージ    | 監視されているリージョンの数。                                                                                                                                        |
| **tikv_log_backup_task_status**                       | ゲージ    | ログバックアップタスクのステータス。 `0`は動作中を示します。 `1`は一時停止を示します。 `2`はエラーを示します。<br/>`task :: string`                                                                                                |
| **tikv_log_backup_pending_initial_scan**              | ゲージ    | 保留中の初回スキャンの統計。 <br/>`stage :: {"queuing", "executing"}`                                                                                                     |

### ログバックアップアラート

#### アラート構成

現在、PITRには組み込みのアラートアイテムがありません。このセクションでは、PITRでアラートアイテムを構成する方法といくつかの推奨アイテムについて紹介します。

PITRでアラートアイテムを構成する手順は次の通りです。

1. Prometheusが存在するノードでアラートルールの構成ファイル（たとえば`pitr.rules.yml`）を作成します。ファイルには、[Prometheusドキュメント](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/)、以下の推奨アラートアイテム、および構成サンプルに従ってアラートルールを記入します。
2. Prometheus構成ファイルの`rule_files`フィールドに、アラートルールファイルのパスを追加します。
3. Prometheusプロセスに`SIGHUP`シグナルを送信（`kill -HUP pid`）するか、Prometheusを起動する際に`--web.enable-lifecycle`パラメータを追加してHTTP `POST`リクエストを`http://prometheus-addr/-/reload`に送信します（HTTPリクエストを送信する前に、Prometheusの起動時に`--web.enable-lifecycle`パラメータを追加してください）。

推奨アラートアイテムは以下の通りです:

#### LogBackupRunningRPOMoreThan10m

- アラートアイテム: `max(time() - tidb_log_backup_last_checkpoint / 262144000) by (task) / 60 > 10 and max(tidb_log_backup_last_checkpoint) by (task) > 0 and max(tikv_log_backup_task_status) by (task) == 0`
- アラートレベル: 警告
- 説明: ログデータが10分以上ストレージに保存されていません。このアラートアイテムはリマインダーです。ほとんどの場合、ログバックアップに影響はありません。

このアラートアイテムの構成サンプルは次のとおりです:

```yaml
groups:
- name: PiTR
  rules:
  - alert: LogBackupRunningRPOMoreThan10m
    expr: max(time() - tidb_log_backup_last_checkpoint / 262144000) by (task) / 60 > 10 and max(tidb_log_backup_last_checkpoint) by (task) > 0 and max(tikv_log_backup_task_status) by (task) == 0
    labels:
      severity: warning
    annotations:
      summary: RPO of log backup is high
      message: RPO of the log backup task {{ $labels.task }} is more than 10m
```

#### LogBackupRunningRPOMoreThan30m

- アラートアイテム: `max(time() - tidb_log_backup_last_checkpoint / 262144000) by (task) / 60 > 30 and max(tidb_log_backup_last_checkpoint) by (task) > 0 and max(tikv_log_backup_task_status) by (task) == 0`
- アラートレベル: クリティカル
- 説明: ログデータが30分以上ストレージに保存されていません。このアラートは通常、異常を示します。原因を見つけるためにTiKVログを確認できます。

#### LogBackupPausingMoreThan2h

- アラートアイテム: `max(time() - tidb_log_backup_last_checkpoint / 262144000) by (task) / 3600 > 2 and max(tidb_log_backup_last_checkpoint) by (task) > 0 and max(tikv_log_backup_task_status) by (task) == 1`
- アラートレベル: 警告
- 説明：ログバックアップタスクが2時間以上一時停止しています。このアラートはリマインダーであり、「br log resume」をできるだけ早く実行することが期待されています。

#### LogBackupPausingMoreThan12h

- アラート項目：`max(time() - tidb_log_backup_last_checkpoint / 262144000) by (task) / 3600 > 12 and max(tidb_log_backup_last_checkpoint) by (task) > 0 and max(tikv_log_backup_task_status) by (task) == 1`
- アラートレベル：重大
- 説明：ログバックアップタスクが12時間以上一時停止しています。「br log resume」をできるだけ早く実行してタスクを再開してください。長時間一時停止したログタスクはデータ損失のリスクがあります。

#### LogBackupFailed

- アラート項目：`max(tikv_log_backup_task_status) by (task) == 2 and max(tidb_log_backup_last_checkpoint) by (task) > 0`
- アラートレベル：重大
- 説明：ログバックアップタスクが失敗しました。「br log status」を実行して失敗理由を確認する必要があります。必要に応じて、TiKVログをさらにチェックする必要があります。

#### LogBackupGCSafePointExceedsCheckpoint

- アラート項目：`min(tidb_log_backup_last_checkpoint) by (instance) - max(tikv_gcworker_autogc_safe_point) by (instance) < 0`
- アラートレベル：重大
- 説明：バックアップ前にデータがガベージコレクションされました。これは、一部のデータが失われ、サービスに影響を及ぼす可能性が非常に高いことを意味します。