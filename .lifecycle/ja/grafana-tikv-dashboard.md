---
title: TiKVの主要な監視メトリクス
summary: Grafana TiKVダッシュボードに表示される主要なメトリクスについて学びます。
aliases: ['/docs/dev/grafana-tikv-dashboard/', '/docs/dev/reference/key-monitoring-metrics/tikv-dashboard/']
---

# TiKVの主要な監視メトリクス

TiUPを使用してTiDBクラスタを展開する場合、監視システム（Prometheus/Grafana）が同時に展開されます。詳細については、[監視フレームワークの概要](/tidb-monitoring-framework.md)を参照してください。

Grafanaダッシュボードは、概要、PD、TiDB、TiKV、Node_exporter、パフォーマンスの概要を含む一連のサブダッシュボードに分割されています。診断に役立つ多くのメトリクスがあります。

## TiKV-Details ダッシュボード

**TiKV-Details** ダッシュボードでは、主要なメトリクスが表示されるため、TiKVのコンポーネントのステータスの概要が把握できます。[パフォーマンスマップ](https://asktug.com/_/tidb-performance-map/#/)によれば、クラスタのステータスが予想どおりかどうかを確認できます。

このセクションでは、**TiKV-Details** ダッシュボード上のこれらの主要なメトリクスについて詳細な説明が提供されます。

### クラスタ

- ストアサイズ：TiKVインスタンスごとのストレージサイズ
- 利用可能サイズ：TiKVインスタンスごとの利用可能容量
- 容量サイズ：TiKVインスタンスごとの容量サイズ
- CPU：TiKVインスタンスごとのCPU利用率
- メモリ：TiKVインスタンスごとのメモリ使用量
- I/O利用率：TiKVインスタンスごとのI/O利用率
- MBps：各TiKVインスタンスでの読み取りと書き込みの合計バイト数
- QPS：各TiKVインスタンスでのコマンドごとのQPS
- Errps：gRPCメッセージの失敗率
- leader：TiKVインスタンスごとのリーダー数
- リージョン：TiKVインスタンスごとのリージョン数
- アップタイム：直近の再起動以降のTiKVの稼働時間

![TiKVダッシュボード - クラスタメトリクス](/media/tikv-dashboard-cluster.png)

### エラー

- クリティカルエラー：クリティカルエラーの数
- サーバーがビジー：TiKVインスタンスが一時的に利用できなくなる出来事の発生を示します（書き込みスタールやチャネルフルなど）。通常は `0` であるべきです。
- サーバーレポートエラー：サーバーによって報告されたエラーメッセージの数。通常は `0` であるべきです。
- Raftstoreエラー：各TiKVインスタンスごとのRaftstoreエラーの数
- スケジューラエラー：各TiKVインスタンスごとのスケジューラエラーの数
- Coprocessorエラー：各TiKVインスタンスごとのCoprocessorエラーの数
- gRPCメッセージエラー：各TiKVインスタンスごとのgRPCメッセージエラーの数
- リーダードロップ：TiKVインスタンスごとのドロップされたリーダーの数
- リーダーミッシング：TiKVインスタンスごとの欠落しているリーダーの数
- ログ複製拒否：各TiKVインスタンスでのメモリが不足しているために拒否されたログ追加メッセージの数

![TiKVダッシュボード - エラーメトリクス](/media/tikv-dashboard-errors-v610.png)

### サーバー

- CFサイズ：各カラムファミリーのサイズ
- ストアサイズ：TiKVインスタンスごとのストレージサイズ
- チャネルフル：TiKVインスタンスごとのチャネルフルエラー数。通常は `0` であるべきです。
- アクティブな書き込みリーダー：各TiKVインスタンスで書き込まれているリーダーの数
- 近似リージョンサイズ：近似リージョンサイズ
- 近似リージョンサイズヒストグラム：各近似リージョンサイズのヒストグラム
- リージョン平均書き込みキー：TiKVインスタンスごとのリージョンへの平均書き込みキー数
- リージョン平均書き込みバイト：TiKVインスタンスごとのリージョンへの平均書き込みバイト数

![TiKVダッシュボード - サーバーメトリクス](/media/tikv-dashboard-server.png)

### gRPC

- gRPCメッセージ数：メッセージタイプごとのgRPCメッセージのレート
- gRPCメッセージ失敗：失敗したgRPCメッセージのレート
- 99% gRPCメッセージ期間：gRPCメッセージの期間（P99）の平均
- 平均gRPCメッセージ期間：平均のgRPCメッセージの実行時間
- gRPCバッチサイズ：TiDBとTiKV間のgRPCメッセージのバッチサイズ
- Raftメッセージバッチサイズ：TiKVインスタンス間のRaftメッセージのバッチサイズ
- gRPCリクエストソースQPS：gRPCリクエストソースのQPS
- gRPCリクエストソース期間：gRPCリクエストソースの実行時間
- gRPCリクエストソースリソースグループQPS：リソースグループごとのgRPCリクエストソースのQPS

### スレッドCPU

- RaftストアCPU：`raftstore`スレッドのCPU利用率。通常は、CPU利用率は 80% * `raftstore.store-pool-size` よりも少ないはずです。
- Async apply CPU：`async apply`スレッドのCPU利用率。通常は、CPU利用率は 90% * `raftstore.apply-pool-size` よりも少ないはずです。
- ストアライターCPU：非同期IOスレッドのCPU利用率。通常は、CPU利用率は 90% * `raftstore.store-io-pool-size` よりも少ないはずです。
- gRPCポーリングCPU：`gRPC`スレッドのCPU利用率。通常は、CPU利用率は 80% * `server.grpc-concurrency` よりも少ないはずです。
- スケジューラワーカーCPU：`スケジューラワーカー`スレッドのCPU利用率。通常は、CPU利用率は 90% * `storage.scheduler-worker-pool-size` よりも少ないはずです。
- ストレージReadPool CPU：`ストレージリードプール`スレッドのCPU利用率
- 統一リードプールCPU：`統一リードプール`スレッドのCPU利用率
- RocksDB CPU：RocksDBスレッドのCPU利用率
- Coprocessor CPU：`coprocessor`スレッドのCPU利用率
- GCワーカーCPU：`GCワーカー`スレッドのCPU利用率
- バックグラウンドワーカーCPU：`バックグラウンドワーカー`スレッドのCPU利用率
- Import CPU：`import`スレッドのCPU利用率
- Backup Worker CPU：`バックアップ`スレッドのCPU利用率
- CDC Worker CPU：`CDCワーカー`スレッドのCPU利用率
- CDC endpoint CPU：`CDCエンドポイント`スレッドのCPU利用率
- Raftlog fetch worker CPU：非同期RaftログフェッチャーワーカーのCPU利用率
- TSO Worker CPU：`TSOワーカー`スレッドのCPU利用率

### PD

- PDリクエスト：TiKVからPDに送信されるレート
- PDリクエスト期間（平均）：TiKVからPDに送信されるリクエストの平均処理時間
- PDハートビート：TiKVからPDへ送信されるハートビートメッセージのレート
- PDバリデートピア：TiKVからPDへTiKVピアを検証するために送信されるメッセージのレート

### Raft IO

- ログ適用期間：Raftがログを適用するために消費された時間
- サーバーごとのログ適用期間：各TiKVインスタンスごとにRaftがログを適用するために消費された時間
- ログ追加期間：Raftがログを追加するために消費された時間
- サーバーごとのログ追加期間：各TiKVインスタンスごとにRaftがログを追加するために消費された時間
- コミットログ期間：Raftがログをコミットするために消費された時間
- サーバーごとのコミットログ期間：各TiKVインスタンスごとにRaftがログをコミットするために消費された時間

![TiKVダッシュボード - Raft IOメトリクス](/media/tikv-dashboard-raftio.png)

### Raftプロセス

- レディハンドル数：タイプごとの1秒あたりの処理されたレディ操作の数
    - count：1秒あたりの処理されたレディ操作の数
    - has_ready_region：1秒あたりのレディがあるリージョンの数
    - pending_region：レディがあるかどうかを確認するためにチェックされているリージョンの操作。このメトリックは v3.0.0 以降非推奨です
    - message：1秒あたりに含まれるレディ操作のメッセージの数
    - append：1秒あたりに含まれるRaftログエントリの数
    - commit：1秒あたりに含まれるコミットされたRaftログエントリの数
    - snapshot：1秒あたりに含まれるスナップショットの数
- 0.99 Raftストアイベント期間：Raftストアイベントの時間（P99）
- プロセスの準備期間：Raftのプロセスが準備されるために消費された時間
- サーバーごとの準備期間：各TiKVインスタンスごとのRaft内のピアプロセスが準備するために消費された時間。2秒未満であるべきです（P99.99）。
- Raftストアイベントの最大期間：最も遅いRaftストアイベントの時間
- レプリカ読み取りロックチェック期間：レプリカ読み取りの処理時にロックを確認するために消費された時間
- ピアメッセージ長分布：各TiKVインスタンスの各リージョンで処理されるメッセージの数。メッセージが多いほど、ピアがより忙しいです。

![TiKVダッシュボード - Raftプロセスメトリクス](/media/tikv-dashboard-raft-process.png)

### Raftメッセージ

- サーバーごとの送信されたメッセージ：各TiKVインスタンスが送信したRaftメッセージの数（1秒あたり）
- サーバーごとのフラッシュされたメッセージ：各TiKVインスタンスでRaftクライアントによってフラッシュされたRaftメッセージの数（1秒あたり）
- サーバーごとの受信されたメッセージ：各TiKVインスタンスが受信したRaftメッセージの数（1秒あたり）
- メッセージ：1秒あたりのメッセージタイプごとのRaftメッセージの数
- 投票：Raft内で送信された投票メッセージの数（1秒あたり）
- Raftドロップメッセージ：タイプごとのRaftメッセージのドロップ数（1秒あたり）

![TiKVダッシュボード - Raftメッセージのメトリクス](/media/tikv-dashboard-raft-message.png)

### Raft提案

- レディごとの提案のヒストグラム：提案のバッチごとに含まれる提案の数のヒストグラムを適用する際の各レディ操作ごとに測定
- Raftの読み書き提案：タイプごとの秒間提案数
- サーバごとのRaft読み込み提案：TiKVインスタンスごとに行われる読み込み提案数
- サーバごとのRaft書き込み提案：TiKVインスタンスごとに行われる書き込み提案数
- 提案待ち時間：各提案の待ち時間のヒストグラム
- サーバごとの提案待ち時間：TiKVインスタンスごとに行われる各提案の待ち時間のヒストグラム
- 適用待ち時間：各提案の適用時間のヒストグラム
- サーバごとの適用待ち時間：TiKVインスタンスごとに行われる各提案の適用時間のヒストグラム
- Raftログの速度：ピアごとのログ提案の平均速度

![TiKVダッシュボード - Raft提案のメトリクス](/media/tikv-dashboard-raft-propose.png)

### Raft管理者

- 管理者提案：秒ごとの管理者提案の数
- 管理者適用：秒ごとの処理された適用コマンドの数
- スプリットチェック：秒ごとのRaftstoreのスプリットチェックコマンドの数
- 99.99%のスプリットチェック時間：スプリットチェックコマンドの実行時間（P99.99）

![TiKVダッシュボード - Raft管理者のメトリクス](/media/tikv-dashboard-raft-admin.png)

### ローカルリーダー

- ローカルリーダーリクエスト：ローカルリードスレッドからの総リクエスト数および拒否された数

![TiKVダッシュボード - ローカルリーダーのメトリクス](/media/tikv-dashboard-local-reader.png)

### 統合リードプール

- レベルごとの使用時間：統合リードプールの各レベルごとの使用時間。レベル0は小さいクエリを意味します。
- レベル0の確率：統合リードプールのレベル0タスクの比率
- 実行中のタスク：統合リードプール内で同時に実行されているタスクの数

### ストレージ

- ストレージコマンド総数：秒ごとのタイプごとの受信コマンド数
- 非同期リクエストエラー：秒ごとのエンジンの非同期リクエストエラー数
- 非同期スナップショット処理時間：非同期スナップショットリクエストの処理時間。`.99`で`1s`以下である必要があります。
- 非同期書き込み処理時間：非同期書き込みリクエストの処理時間。`.99`で`1s`以下である必要があります。

![TiKVダッシュボード - ストレージのメトリクス](/media/tikv-dashboard-storage.png)

### スケジューラ

- 各ステージごとのスケジューラのコマンド数：秒ごとの各ステージのコマンド数。短時間の間に多くのエラーがあってはなりません。
- スケジューラの書き込みバイト数：各TiKVインスタンスで処理されたコマンドによって書き込まれた合計バイト数
- スケジューラの優先コマンド数：秒ごとの異なる優先コマンドの数
- スケジューラの保留コマンド数：TiKVインスタンスごとの秒ごとの保留コマンドの数

![TiKVダッシュボード - スケジューラのメトリクス](/media/tikv-dashboard-scheduler.png)

### スケジューラ - コミット

- コミットコマンドを実行する際の各ステージごとのコマンド数：秒ごとの各ステージのコマンド数。短時間の間に多くのエラーがあってはなりません。
- コミットコマンドの実行時間：コミットコマンドの実行にかかる時間。`1s`以下である必要があります。
- スケジューラのラッチ待機時間：コミットコマンドの実行時にラッチによって生じる待機時間。`1s`以下である必要があります。
- 読み取られたキーの数：コミットコマンドによって読み取られたキーの数
- 書き込まれたキーの数：コミットコマンドによって書き込まれたキーの数
- スキャンの詳細：コミットコマンドの実行時に各CFのスキャンの詳細キー
- スキャンの詳細 [lock]：コミットコマンドの実行時のロックCFのスキャンの詳細キー
- スキャンの詳細 [write]：コミットコマンドの実行時の書き込みCFのスキャンの詳細キー
- スキャンの詳細 [default]：コミットコマンドの実行時のデフォルトCFのスキャンの詳細キー

![TiKVダッシュボード - スケジューラのコミットメトリクス](/media/tikv-dashboard-scheduler-commit.png)

### スケジューラ - pessimistic_rollback

- `pessimistic_rollback`コマンドを実行する際の各ステージごとのコマンド数：秒ごとの各ステージのコマンド数。短時間の間に多くのエラーがあってはなりません。
- `pessimistic_rollback`コマンドの実行時間：`pessimistic_rollback`コマンドの実行にかかる時間。`1s`以下である必要があります。
- スケジューラのラッチ待機時間：`pessimistic_rollback`コマンドの実行時にラッチによって生じる待機時間。`1s`以下である必要があります。
- 読み取られたキーの数：`pessimistic_rollback`コマンドによって読み取られたキーの数
- 書き込まれたキーの数：`pessimistic_rollback`コマンドによって書き込まれたキーの数
- スキャンの詳細：`pessimistic_rollback`コマンドの実行時に各CFのスキャンの詳細キー
- スキャンの詳細 [lock]：`pessimistic_rollback`コマンドの実行時のロックCFのスキャンの詳細キー
- スキャンの詳細 [write]：`pessimistic_rollback`コマンドの実行時の書き込みCFのスキャンの詳細キー
- スキャンの詳細 [default]：`pessimistic_rollback`コマンドの実行時のデフォルトCFのスキャンの詳細キー

### スケジューラ - prewrite

- prewriteコマンドを実行する際の各ステージごとのコマンド数：秒ごとの各ステージのコマンド数。短時間の間に多くのエラーがあってはなりません。
- prewriteコマンドの実行時間：prewriteコマンドの実行にかかる時間。`1s`以下である必要があります。
- スケジューラのラッチ待機時間：prewriteコマンドの実行時にラッチによって生じる待機時間。`1s`以下である必要があります。
- 読み取られたキーの数：prewriteコマンドによって読み取られたキーの数
- 書き込まれたキーの数：prewriteコマンドによって書き込まれたキーの数
- スキャンの詳細：prewriteコマンドの実行時に各CFのスキャンの詳細キー
- スキャンの詳細 [lock]：prewriteコマンドの実行時のロックCFのスキャンの詳細キー
- スキャンの詳細 [write]：prewriteコマンドの実行時の書き込みCFのスキャンの詳細キー
- スキャンの詳細 [default]：prewriteコマンドの実行時のデフォルトCFのスキャンの詳細キー

### スケジューラ - rollback

- rollbackコマンドを実行する際の各ステージごとのコマンド数：秒ごとの各ステージのコマンド数。短時間の間に多くのエラーがあってはなりません。
- rollbackコマンドの実行時間：rollbackコマンドの実行にかかる時間。`1s`以下である必要があります。
- スケジューラのラッチ待機時間：rollbackコマンドの実行時にラッチによって生じる待機時間。`1s`以下である必要があります。
- 読み取られたキーの数：rollbackコマンドによって読み取られたキーの数
- 書き込まれたキーの数：rollbackコマンドによって書き込まれたキーの数
- スキャンの詳細：rollbackコマンドの実行時に各CFのスキャンの詳細キー
- スキャンの詳細 [lock]：rollbackコマンドの実行時のロックCFのスキャンの詳細キー
- スキャンの詳細 [write]：rollbackコマンドの実行時の書き込みCFのスキャンの詳細キー
- スキャンの詳細 [default]：rollbackコマンドの実行時のデフォルトCFのスキャンの詳細キー

### GC

- GCタスク数：gc_workerが処理したGCタスクの数
- GCタスクの実行時間：GCタスクの実行にかかる時間
- TiDB GC秒：GCの所要時間
- TiDB GCワーカーアクション数：TiDB GCワーカーアクションの数
- ResolveLocksの進捗：GCの第1フェーズ（Resolve Locks）の進捗
- TiKV Auto GCの進捗：GCの第2フェーズの進捗
- GC速度：GCによって削除されるキーの数（秒ごと）
- TiKV Auto GCセーフポイント：TiKV GCのセーフポイントの値。セーフポイントは現在のGCタイムスタンプです。
- GCの寿命：TiDB GCの寿命
- GCの間隔：TiDB GCの間隔
- コンパクションフィルターでのGC：書き込みCFのコンパクションフィルターでのフィルタリングされたバージョンの数

### スナップショット

- スナップショットメッセージの送信速度：Raftスナップショットメッセージの送信速度
- 99%のスナップショット処理時間：スナップショットの処理時間（P99）
- スナップショットの状態数：状態ごとのスナップショット数
- 99.99%のスナップショットのサイズ：スナップショットのサイズ（P99.99）
- 99.99%のスナップショットのKV数：スナップショット内のKVの数（P99.99）

### タスク

- ワーカーが処理するタスク数：ワーカーが処理するタスク数（秒ごと）
- ワーカーの保留タスク数：ワーカーの現在の保留および実行中のタスク数。通常の場合は`1000`未満である必要があります。
- FuturePoolが処理するタスク数：FuturePoolが処理するタスク数（秒ごと）
- FuturePoolの保留タスク数：FuturePoolの現在の保留および実行中のタスク数（秒ごと）

### コプロセッサの概要

- リクエストの所要時間：コプロセッサリクエストを受け取って処理を完了するまでの所要時間の合計
- 合計リクエスト数：タイプごとの1秒あたりのリクエスト数
- 処理時間：コプロセッサリクエストの処理に実際にかかる時間のヒストグラム（1分ごと）
- 合計リクエストエラー数：1秒あたりのコプロセッサのリクエストエラー数。短時間の間に多くのエラーがあってはなりません。
- 合計KVカーソル操作数：タイプごとの1秒あたりのKVカーソル操作の総数（`select`、`index`、`analyze_table`、`analyze_index`、`checksum_table`、`checksum_index`など）
- KVカーソル操作：タイプごとの1秒あたりのKVカーソル操作のヒストグラム
- 合計RocksDBパフォーマンス統計情報：RocksDBパフォーマンスの統計情報
- 総応答サイズ：Coprocessor応答の総サイズ

### Coprocessor 詳細

- ハンドル時間：実際に処理されるcoprocessorリクエストの経過時間のヒストグラム（分単位）
- ストアによる95%ハンドル時間：TiKVインスタンスごとのcoprocessorリクエストの処理に要する時間（P95）
- 待機時間：coprocessorリクエストの処理を待っている時間。`10秒`未満である必要があります（P99.99）。
- ストアによる95%待機時間：TiKVインスタンスごとのcoprocessorリクエストの処理を待っている時間（P95）
- 総DAGリクエスト：秒あたりの総DAGリクエスト数
- 総DAGエグゼキューター：秒あたりの総DAGエグゼキューター数
- 総Ops詳細（テーブルスキャン）：Coprocessorでselectスキャンを実行する際の秒あたりのRocksDB内部操作数
- 総Ops詳細（インデックススキャン）：Coprocessorでインデックススキャンを実行する際の秒あたりのRocksDB内部操作数
- CFごとの総Ops詳細（テーブルスキャン）：Coprocessorでselectスキャンを実行する際の各CFごとの秒あたりのRocksDB内部操作数
- CFごとの総Ops詳細（インデックススキャン）：Coprocessorでインデックススキャンを実行する際の各CFごとの秒あたりのRocksDB内部操作数

### スレッド

- スレッド状態：TiKVスレッドの状態
- スレッドI/O：各TiKVスレッドのI/Oトラフィック
- スレッド任意のコンテキストスイッチ：TiKVスレッドの任意のコンテキストスイッチの数
- スレッド非任意のコンテキストスイッチ：TiKVスレッドの非任意のコンテキストスイッチの数

### RocksDB - kv/raft

- Get operations：秒あたりのget操作の数
- Get duration：get操作の実行に要する時間
- Seek operations：秒あたりのseek操作の数
- Seek duration：seek操作の実行に要する時間
- Write operations：秒あたりの書き込み操作の数
- Write duration：書き込み操作の実行に要する時間
- WAL sync operations：秒あたりのWAL同期操作の数
- Write WAL duration：WALの書き込みに要する時間
- WAL sync duration：WAL同期操作の実行に要する時間
- Compaction operations：秒あたりのコンパクションおよびフラッシュ操作の数
- Compaction duration：コンパクションおよびフラッシュ操作の実行に要する時間
- SST read duration：SSTファイルの読み込みに要する時間
- Write stall duration：書き込みスタールの時間。通常の場合は`0`である必要があります。
- Memtable size：各列ファミリのメムテーブルサイズ
- Memtable hit：メムテーブルのヒット率
- Block cache size：ブロックキャッシュサイズ。共有ブロックキャッシュが無効の場合は、列ファミリごとに分割されます。
- Block cache hit：ブロックキャッシュのヒット率
- Block cache flow：各種別ごとのブロックキャッシュ操作のフロー率
- Block cache operations：各種別ごとのブロックキャッシュ操作数
- Keys flow：キーの操作のフロー率
- Total keys：各列ファミリのキー数
- Read flow：読み取り操作のフロー率
- Bytes / Read：1読み取り操作当たりのバイト数
- Write flow：書き込み操作のフロー率
- Bytes / Write：1書き込み操作当たりのバイト数
- Compaction flow：コンパクション操作のフロー率
- Compaction pending bytes：コンパクション対象の保留バイト数
- Read amplification：TiKVインスタンスあたりのリード増幅
- Compression ratio：各レベルの圧縮比
- Number of snapshots：TiKVインスタンスあたりのスナップショット数
- Oldest snapshots duration：最も古いリリースされていないスナップショットの生存期間
- Number files at each level：各レベルごとの異なる列ファミリのSSTファイル数
- Ingest SST duration seconds：SSTファイルのインジェストに要する時間
- Stall conditions changed of each CF：各列ファミリのスタール条件の変更

### Raft Engine

- Operations
    - 書き込み：Raft Engineの秒あたりの書き込み操作数
    - read_entry：Raft Engineの秒あたりのraftログ読み込み操作数
    - read_message：Raft Engineの秒あたりのraftメタデータ読み込み操作数
- Write duration：Raft Engineの書き込み操作に要する時間。この時間は、これらのデータの書き込みに関わるディスクI/Oの遅延時間の合計に近いです。
- Flow
    - 書き込み：Raft Engineの書き込みトラフィック
    - rewrite append：書き直し追加ログのトラフィック
    - rewrite rewrite：書き直し記述のトラフィック
- Write Duration Breakdown (99%)
    - WAL：Raft Engine WALの書き込みの遅延時間
    - 待機：書き込み前の待機時間
    - apply：メモリへのデータ適用に要する時間
- Bytes/Written：Raft Engineによる1回ごとの書き込みごとのバイト数
- WAL Duration Breakdown (P99%)：Raft Engine WALの書き込みの各段階で要する時間
- File Count
    - append：Raft Engineによるデータ追加に使用されるファイル数
    - rewrite：Raft Engineによるデータ書き直しに使用されるファイル数（書き直しはRocksDBのコンパクションに類似）
- Entry Count
    - rewrite：Raft Engineによって書き直されたエントリ数
    - append：Raft Engineによって追加されたエントリ数

### Titan - 全般

- Blob file count：Titan blobファイルの数
- Blob file size：Titan blobファイルの合計サイズ
- Live blob size：有効なblobレコードの合計サイズ
- Blob cache hit：Titanブロックキャッシュのヒット率
- Iter touched blob file count：単一のイテレーターに関与するblobファイルの数
- Blob file discardable ratio distribution：blobファイルのレコードが失敗する割合分布
- Blob key size：Titan blobキーのサイズ
- Blob value size：Titan blob値のサイズ
- Blob get operations：Titan blobでのget操作の回数
- Blob get duration：Titan blobでのget操作の実行に要する時間
- Blob iter operations：Titan blobでのiter操作の実行に要する時間
- Blob seek duration：Titan blobでのseek操作の実行に要する時間
- Blob next duration：Titan blobでのnext操作の実行に要する時間
- Blob prev duration：Titan blobでのprev操作の実行に要する時間
- Blob keys flow：Titan blobキーの操作のフロー率
- Blob bytes flow：Titan blobキーのバイトのフロー率
- Blob file read duration：Titan blobファイルの読み込みに要する時間
- Blob file write duration：Titan blobファイルの書き込みに要する時間
- Blob file sync operations：blobファイルの同期操作の回数
- Blob file sync duration：blobファイルの同期に要する時間
- Blob GC action：Titan GCアクションの回数
- Blob GC duration：Titan GCの実行に要する時間
- Blob GC keys flow：Titan GCによって読み取られたキーと書き込まれたキーのフロー率
- Blob GC bytes flow：Titan GCによって読み取られたバイトと書き込まれたバイトのフロー率
- Blob GC input file size：Titan GCの入力ファイルサイズ
- Blob GC output file size：Titan GCの出力ファイルサイズ
- Blob GC file count：Titan GCに関与するblobファイルの数

### 悲観的ロック

- ロックマネージャースレッドCPU：ロックマネージャースレッドのCPU利用率
- ロックマネージャーハンドルタスク：ロックマネージャが処理したタスク数
- 待機者の寿命：ロックが解放されるのを待つトランザクションの待機時間
- ウェイトテーブル：ウェイトテーブルの状態情報で、ロックの数とロックを待っているトランザクションの数を含みます
- デッドロック検出時間：デッドロックの検出に要する時間
- 検出エラー：デッドロックを検出する際に遭遇したエラーの数。デッドロックの数を含みます
- デッドロック検出リーダー：デッドロック検出リーダーが位置するノードの情報
- 総悲観的ロックメモリサイズ：インメモリの悲観的ロックが占有するメモリサイズ
- インメモリ悲観的ロック結果：メモリへの悲観的ロックのみの保存結果。「full」は、メモリ制限を超えたために悲観的ロックがメモリに保存されなかった回数を意味します。

### Resolved-TS

- Resolved-TSワーカーCPU：resolved-tsワーカースレッドのCPU利用率
- Advance-TSワーカーCPU：advance-tsワーカースレッドのCPU利用率
- Scan lockワーカーCPU：scan lockワーカースレッドのCPU利用率
- Resolved-tsの最大ギャップ：このTiKVのすべてのアクティブなリージョンのresolved-tsと現在の時刻との最大時差
- Safe-tsの最大ギャップ：このTiKVのすべてのアクティブなリージョンのsafe-tsと現在の時刻との最大時差
- 最小のResolved TSリージョン：resolved-tsが最小のRegionのID
- 最小のSafe TSリージョン：safe-tsが最小のRegionのID
- リーダーのチェック時間：リーダーリクエストの処理に要した時間の分布。リクエストの送信からリーダーへの応答を受け取るまでの時間を含みます
- リージョンリーダーにおけるresolved-tsの最大ギャップ：このTiKVのすべてのアクティブなリージョンのresolved-tsと現在の時刻との最大時差。リージョンリーダー向けのみです
- 最小のリーダーリゾルブTSリージョン：resolved-tsが最小のRegionのID。リージョンリーダー向けのみです
- ロックヒープサイズ：resolved-tsモジュールでロックを追跡するヒープのサイズ

### メモリ

- Allocato r統計：メモリアロケータの統計情報

### バックアップ

- バックアップCPU：バックアップスレッドのCPU利用率
- レンジサイズ：バックアップレンジサイズのヒストグラム
- バックアップの所要時間：バックアップに費やされる時間
- バックアップフロー：バックアップの合計バイト数
- ディスクスループット：インスタンスごとのディスクスループット
- バックアップ範囲の所要時間：範囲をバックアップするための所要時間
- バックアップエラー：バックアップ中に発生したエラーの数

### 暗号化

- 暗号化データキー：暗号化されたデータキーの総数
- 暗号化されたファイル：暗号化されたファイルの数
- 暗号化の初期化：暗号化が有効かどうかを示す。`1` は有効を意味する。
- 暗号化メタファイルのサイズ：暗号化メタファイルのサイズ
- データの暗号化/復号時間：データの暗号化/復号にかかる時間のヒストグラム
- 暗号化メタデータの読み書き所要時間：暗号化メタファイルの読み書きにかかる時間

### 共通パラメータの説明

#### gRPCメッセージタイプ

1. トランザクショナルAPI：

    - kv_get: `ts` で指定されたデータの最新バージョンを取得するコマンド
    - kv_scan: データの範囲をスキャンするコマンド
    - kv_prewrite: 2段階コミットの最初の段階で確定するデータを事前書き込みするコマンド
    - kv_pessimistic_lock: 他のトランザクションがこのキーを修正するのを防ぐためにキーに悲観的ロックを追加するコマンド
    - kv_pessimistic_rollback: キーの悲観的ロックを削除するコマンド
    - kv_txn_heart_beat: 悲観トランザクションや大規模なトランザクションのために `lock_ttl` を更新して、それらがロールバックされないようにするコマンド
    - kv_check_txn_status: トランザクションの状態をチェックするコマンド
    - kv_commit: 事前書き込みコマンドで書かれたデータを確定するコマンド
    - kv_cleanup: v4.0では非推奨となったトランザクションをロールバックするコマンド
    - kv_batch_get: 一度にバッチキーの値を取得するコマンド。`kv_get` と同様
    - kv_batch_rollback: 複数の事前書きトランザクションのバッチロールバックのコマンド
    - kv_scan_lock: 期限切れのトランザクションをクリーンアップするために、`max_version` よりも前のバージョン番号のすべてのロックをスキャンするコマンド
    - kv_resolve_lock: トランザクションロックをコミットまたはロールバックするコマンド。トランザクションの状態に応じて
    - kv_gc: GCのコマンド
    - kv_delete_range: TiKV からデータの範囲を削除するコマンド

2. Raw API：

    - raw_get: キーの値を取得するコマンド
        - raw_batch_get: バッチキーの値を取得するコマンド
        - raw_scan: データの範囲をスキャンするコマンド
        - raw_batch_scan: 複数の連続したデータ範囲をスキャンするコマンド
        - raw_put: キー/値のペアを書き込むコマンド
        - raw_batch_put: キー/値のペアのバッチを書き込むコマンド
        - raw_delete: キー/値のペアを削除するコマンド
        - raw_batch_delete: キー/値のペアのバッチを削除するコマンド
        - raw_delete_range: データの範囲を削除するコマンド

## TiKV-FastTune ダッシュボード

TiKV のパフォーマンスに関する問題（QPSの揺れ、レイテンシの揺れ、およびレイテンシの増加傾向など）が発生した場合は、**TiKV-FastTune** ダッシュボードを確認できます。このダッシュボードには、クラスター内の書き込みワークロードが中程度または大きい場合に特に診断に役立つ一連のパネルが含まれています。

書き込み関連のパフォーマンス問題が発生した場合は、まず TiDB 関連のダッシュボードをチェックできます。問題がストレージ側にある場合は、**TiKV-FastTune** ページを開き、その上のすべてのパネルを閲覧およびチェックできます。

**TiKV-FastTune** ダッシュボードでは、パフォーマンス問題の可能性のある原因を示すタイトルを見ることができます。提示された原因が正しいかどうかを確認するには、ページ上のグラフをチェックしてください。

グラフの左Y軸はストレージ側の書き込みRPC QPSを表し、右Y軸のグラフが逆さまに描かれています。左のグラフの形が右のグラフと一致する場合、提示された原因が正しいです。

詳細なメトリクスと説明については、[ユーザーマニュアル](https://docs.google.com/presentation/d/1aeBF2VCKf7eo4-3TMyP7oPzFWIih6UBA53UI8YQASCQ/edit#slide=id.gab6b984c2a_1_352)を参照してください。