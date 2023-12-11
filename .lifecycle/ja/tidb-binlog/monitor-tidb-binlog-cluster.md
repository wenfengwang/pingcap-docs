---
title: TiDBビンログモニタリング
summary: TiDBビンログクラスターの監視方法を学びます。
aliases: ['/docs/dev/tidb-binlog/monitor-tidb-binlog-cluster/','/docs/dev/reference/tidb-binlog/monitor/','/docs/dev/how-to/monitor/tidb-binlog/']
---

# TiDBビンログモニタリング

TiDBビンログを展開した後は、Grafana Web（デフォルトアドレス: <http://grafana_ip:3000>, デフォルトアカウント: admin, パスワード: admin）に移動して、PumpとDrainerの状態を確認できます。

## モニタリングメトリクス

TiDBビンログはPumpとDrainerの2つのコンポーネントで構成されています。このセクションでは、PumpとDrainerのモニタリングメトリクスを表示します。

### Pumpのモニタリングメトリクス

Pumpのモニタリングメトリクスを理解するために、次の表を確認してください:

| Pumpのモニタリングメトリクス | 説明 |
| --- | --- |
| ストレージサイズ | 総ディスク容量（容量）と利用可能なディスク容量（利用可能）を記録 |
| メタデータ | 各Pumpノードが削除できるビンログの最大TSO（`gc_tso`）と保存されたビンログの最大コミットTSO（`max_commit_tso`）を記録 |
| インスタンスごとの書き込みビンログQPS | 各Pumpノードが受信したビンログ書き込みリクエストのQPSを表示 |
| 書き込みビンログ遅延 | 各Pumpノードがビンログを書き込む際の遅延時間を記録 |
| ストレージ書き込みビンログサイズ | Pumpが書き込んだビンログデータのサイズを表示 |
| ストレージ書き込みビンログ遅延 | Pumpストレージモジュールがビンログを書き込む際の遅延時間を記録 |
| Pumpストレージタイプ別エラー | Pumpが遭遇したエラーの種類別のエラー数を記録 |
| TiKVのクエリ | PumpがTiKV経由でトランザクションステータスを問い合わせた回数 |

### Drainerのモニタリングメトリクス

Drainerのモニタリングメトリクスを理解するために、次の表を確認してください:

| Drainerのモニタリングメトリクス | 説明 |
| --- | --- |
| チェックポイントTSO | Drainerが既に下流にレプリケートしたビンログの最大TSO時間を表示します。現在の時間からビンログのタイムスタンプを引くことでラグを取得できます。ただし、タイムスタンプはマスタークラスターのPDによって割り当てられ、PDの時間によって決定されます。 |
| PumpハンドルTSO | 各PumpノードからDrainerが取得したビンログファイルの最大TSO時間を記録 |
| PumpノードIDごとのビンログ取得QPS | Drainerが各Pumpノードからビンログを取得する際のQPSを表示 |
| 95%のビンログ到達所要時間(Pump毎） | Pumpに書き込まれてからDrainerが取得するまでの遅延を記録 |
| エラー別 | エラータイプ別に数えられたDrainerが遭遇したエラー数を表示 |
| SQLクエリ時間 | Drainerが下流でSQLステートメントを実行する時間を記録 |
| Drainerイベント | "ddl", "insert", "delete", "update", "flush", "savepoint"を含むさまざまなタイプのイベントの数を表示 |
| 実行時間 | 下流の同期モジュールにビンログを書き込むのにかかる時間を記録 |
| 95%のビンログサイズ | Drainerが各Pumpノードから取得するビンログデータのサイズを表示 |
| DDLジョブ数 | Drainerが処理したDDLステートメントの数を記録 |
| キューサイズ | Drainerのワークキューサイズを記録 |

## アラートルール

このセクションでは、TiDBビンログのアラートルールを説明します。深刻度レベルに応じて、TiDBビンログのアラートルールは3つのカテゴリに分かれています（高から低に向けて）: 緊急レベル、重要レベル、警告レベル。

### 緊急レベルのアラート

緊急レベルのアラートはサービスやノードの障害によってよく引き起こされます。即座の手動介入が必要です。

#### `binlog_pump_storage_error_count`

* アラートルール:

    `changes(binlog_pump_storage_error_count[1m]) > 0`

* 説明:

    Pumpがローカルストレージにビンログデータを書き込めません。

* 対処:

    `pump_storage_error`モニタリングにエラーが存在するかどうかを確認し、Pumpのログを調べて原因を特定してください。

### 重要レベルのアラート

重要レベルのアラートでは、異常メトリクスを注意深く監視する必要があります。

#### `binlog_drainer_checkpoint_high_delay`

* アラートルール:

    `(time() - binlog_drainer_checkpoint_tso / 1000) > 3600`

* 説明:

    Drainerのレプリケーションの遅延が1時間を超えています。

* 対処:

    - Pumpからデータを取得する際に遅延が起きていないか確認してください:

        各Pumpの最新メッセージの時間を取得するために`handle tso`を確認できます。Pumpで高いレイテンシが存在するかどうかをチェックし、対応するPumpが正常に動作していることを確認してください。

    - Drainerの`event`とDrainerの`execute latency`に基づいて下流でのデータ複製が遅いかどうか確認してください:

        - もしDrainerの`execute time`が大きすぎる場合は、Drainerが展開されたマシンとターゲットデータベースが展開されたマシン間のネットワーク帯域幅とレイテンシ、およびターゲットデータベースの状態を確認してください。
        - もしDrainerの`execute time`があまり大きくなく、Drainerの`event`が小さすぎる場合は、`work count`と`batch`を追加してリトライしてください。

    - 上記の2つの解決策が機能しない場合は、[PingCAPまたはコミュニティからサポートを取得](/support.md)してください。

### 警告レベルのアラート

警告レベルのアラートは問題やエラーのリマインダーとなります。

#### `binlog_pump_write_binlog_rpc_duration_seconds_bucket`

* アラートルール:

    `histogram_quantile(0.9, rate(binlog_pump_rpc_duration_seconds_bucket{method="WriteBinlog"}[5m])) > 1`

* 説明:

    PumpがTiDBのビンログ書き込み要求を処理するのに時間がかかりすぎています。

* 対処:

    - ディスクの性能負荷を検証し、`node exported`を介してディスクのパフォーマンスモニタリングを確認してください。
    - もし`ディスクレイテンシ`と`利用率`の両方が低い場合は、[PingCAPまたはコミュニティからサポートを取得](/support.md)してください。

#### `binlog_pump_storage_write_binlog_duration_time_bucket`

* アラートルール:

    `histogram_quantile(0.9, rate(binlog_pump_storage_write_binlog_duration_time_bucket{type="batch"}[5m])) > 1`

* 説明:

    Pumpがローカルのビンログを書き込むのにかかる時間が長すぎます。

* 対処:

    Pumpのローカルディスクの状態を確認し、問題を修正してください。

#### `binlog_pump_storage_available_size_less_than_20G`

* アラートルール:

    `binlog_pump_storage_storage_size_bytes{type="available"} < 20 * 1024 * 1024 * 1024`

* 説明:

    Pumpの利用可能なディスク容量が20GB未満です。

* 対処:

    Pumpの`gc_tso`が正常かどうか確認してください。もし正常でない場合は、Pumpの`gcタイム`構成を調整するか、対応するPumpをオフラインにしてください。

#### `binlog_drainer_checkpoint_tso_no_change_for_1m`

* アラートルール:

    `changes(binlog_drainer_checkpoint_tso[1m]) < 1`

* 説明:

    Drainerの`checkpoint`が1分間更新されていません。

* 対処:

    オフラインになっていないすべてのPumpが正常に動作しているかどうかを確認してください。

#### `binlog_drainer_execute_duration_time_more_than_10s`

* アラートルール:

    `histogram_quantile(0.9, rate(binlog_drainer_execute_duration_time_bucket[1m])) > 10`

* 説明:

    Drainerが下流のTiDBにデータをレプリケートするのにかかるトランザクション時間が長すぎます。もしこの問題が発生する場合は、Drainerによるデータのレプリケーションに影響があります。

* 対処:

    - TiDBクラスターの状態を確認してください。
    - Drainerのログまたはモニタリングを確認してください。DDL操作がこの問題を引き起こすものであれば、無視しても構いません。