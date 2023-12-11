---
title: TiKV 設定ファイル
summary: TiKV 設定ファイルについて学ぶ
aliases: ['/docs/dev/tikv-configuration-file/','/docs/dev/reference/configuration/tikv-server/configuration-file/']
---

# TiKV 設定ファイル

<!-- markdownlint-disable MD001 -->

TiKV 設定ファイルは、コマンドラインパラメータよりも多くのオプションをサポートしています。[etc/config-template.toml](https://github.com/tikv/tikv/blob/master/etc/config-template.toml) でデフォルトの設定ファイルを見つけ、これを `config.toml` にリネームすることができます。

このドキュメントでは、コマンドラインパラメータに含まれていないパラメータのみを説明しています。詳細については、[コマンドラインパラメータ](/command-line-flags-for-tikv-configuration.md) を参照してください。

> **ヒント:**
>
> 設定項目の値を調整する必要がある場合は、[設定を変更する](/maintain-tidb-using-tiup.md#modify-the-configuration) を参照してください。

## グローバル設定

### `abort-on-panic`

+ TiKV がパニックしたときに `abort()` を呼び出してプロセスを終了するかどうかを設定します。このオプションによって、TiKV がシステムにコアダンプファイルを生成するかどうかが影響を受けます。

    + この設定項目の値が `false` の場合、TiKV がパニックしたとき、`exit()` を呼び出してプロセスを終了します。
    + この設定項目の値が `true` の場合、TiKV がパニックしたとき、TiKV は `abort()` を呼び出してプロセスを終了します。このとき、TiKV は終了時にコアダンプファイルを生成する許可を取ります。コアダンプファイルを生成するには、コアダンプに関連するシステム設定を実行する必要があります（たとえば、`ulimit -c` コマンドを使用してコアダンプファイルのサイズ制限を設定し、コアダンプパスを設定します。異なるオペレーティングシステムでは、関連する設定が異なります）。コアダンプファイルがディスク容量を多く占有し、TiKV のディスク容量が不足することを防ぐためには、コアダンプ生成パスを TiKV のデータとは異なるディスクパーティションに設定することを推奨します。

+ デフォルト値: `false`

### `slow-log-file`

+ 遅いログを保存するファイル
+ この設定項目が設定されていないが、`log.file.filename` が設定されている場合、遅いログは `log.file.filename` で指定されたログファイルに出力されます。
+ `slow-log-file` と `log.file.filename` の両方が設定されていない場合、すべてのログはデフォルトで "stderr" に出力されます。
+ 両方の設定項目が設定されている場合、通常のログは `log.file.filename` で指定されたログファイルに出力され、遅いログは `slow-log-file` で設定されたログファイルに出力されます。
+ デフォルト値: `""`

### `slow-log-threshold`

+ 遅いログを出力するための閾値。処理時間がこの閾値を超える場合、遅いログが出力されます。
+ デフォルト値: `"1s"`

### `memory-usage-limit`

+ TiKV インスタンスのメモリ使用量の制限。TiKV のメモリ使用量がほぼこの閾値に達すると、内部キャッシュが解放されてメモリが解放されます。
+ ほとんどの場合、TiKV インスタンスはシステム全体の利用可能なメモリの 75% を使用するように設定されているため、この設定項目を明示的に指定する必要はありません。残りの 25% のメモリは OS のページキャッシュに予約されています。詳細は [`storage.block-cache.capacity`](#capacity) を参照してください。
+ 単一の物理マシンに複数の TiKV ノードを展開する場合でも、この設定項目を設定する必要はありません。この場合、TiKV インスタンスはメモリの `5/3 * block-cache.capacity` を使用します。
+ 異なるシステムメモリ容量のデフォルト値は次のとおりです：

    + システム=8G    ブロックキャッシュ=3.6G    メモリ使用量制限=6G   ページキャッシュ=2G
    + システム=16G   ブロックキャッシュ=7.2G    メモリ使用量制限=12G   ページキャッシュ=4G
    + システム=32G   ブロックキャッシュ=14.4G   メモリ使用量制限=24G   ページキャッシュ=8G

## log <span class="version-mark">v5.4.0 で新規追加</span>

+ ログに関連する設定項目。

+ v5.4.0 から、TiKV のログ設定項目と TiDB のログ設定項目を一貫させるために、TiKV は以前の設定項目 `log-rotation-timespan` を非推奨にし、`log-level`、`log-format`、`log-file`、`log-rotation-size` を次のものに変更しました。古い設定項目のみを設定して、その値がデフォルト値以外に設定されている場合、古い設定項目は新しい設定項目と互換性があります。新しい設定項目と古い設定項目の両方が設定されている場合、新しい設定項目が有効になります。

### `level` <span class="version-mark">v5.4.0 で新規追加</span>

+ ログレベル
+ オプション値: `"debug"`, `"info"`, `"warn"`, `"error"`, `"fatal"`
+ デフォルト値: `"info"`

### `format` <span class="version-mark">v5.4.0 で新規追加</span>

+ ログの形式
+ オプション値: `"json"`, `"text"`
+ デフォルト値: `"text"`

### `enable-timestamp` <span class="version-mark">v5.4.0 で新規追加</span>

+ ログ内のタイムスタンプを有効または無効にするかどうかを決定します
+ オプション値: `true`, `false`
+ デフォルト値: `true`

## log.file <span class="version-mark">v5.4.0 で新規追加</span>

+ ログファイルに関連する設定項目。

### `filename` <span class="version-mark">v5.4.0 で新規追加</span>

+ ログファイル。この設定項目が設定されていない場合、ログはデフォルトで "stderr" に出力されます。この設定項目が設定されている場合、ログは対応するファイルに出力されます。
+ デフォルト値: `""`

### `max-size` <span class="version-mark">v5.4.0 で新規追加</span>

+ 1 つのログファイルの最大サイズ。ファイルサイズがこの設定項目で設定された値よりも大きい場合、システムは単一のファイルを複数のファイルに自動的に分割します。
+ デフォルト値: `300`
+ 最大値: `4096`
+ 単位: MiB

### `max-days` <span class="version-mark">v5.4.0 で新規追加</span>

+ TiKV がログファイルを保持する最大日数
    + 設定項目が設定されていないか、その値がデフォルト値 `0` に設定されている場合、TiKV はログファイルをクリーンアップしません。
    + パラメータが `0` 以外の値に設定されている場合、TiKV は `max-days` の後に期限切れのログファイルをクリーンアップします。
+ デフォルト値: `0`

### `max-backups` <span class="version-mark">v5.4.0 で新規追加</span>

+ TiKV が保持するログファイルの最大数
    + 設定項目が設定されていないか、その値がデフォルト値 `0` に設定されている場合、TiKV はすべてのログファイルを保持します。
    + 設定項目が `0` 以外の値に設定されている場合、TiKV は `max-backups` で指定された古いログファイルを最大で保持します。たとえば、値が `7` に設定されている場合、TiKV は最大で 7 つの古いログファイルを保持します。
+ デフォルト値: `0`

### `pd.enable-forwarding` <span class="version-mark">v5.0.0 で新規追加</span>

+ TiKV 内の PD クライアントが、可能な場合にリーダーに対してフォロワー経由でリクエストを転送するかどうかを制御します。
+ デフォルト値: `false`
+ 環境でネットワーク分離が発生する可能性がある場合、このパラメータを有効にするとサービスの利用可能性のウィンドウを縮小できます。
+ 分離、ネットワーク障害、またはダウンタイムが正確に判断できない場合は、このメカニズムを使用すると判断ミスが発生し、可用性とパフォーマンスが低下する危険があります。ネットワーク障害が発生したことがない場合は、このパラメータを有効にすることは推奨されません。

## サーバー

+ サーバーに関連する設定項目。

### `addr`

+ リッスンする IP アドレスとリッスンポート
+ デフォルト値: `"127.0.0.1:20160"`

### `advertise-addr`

+ クライアント通信のためのリッスンアドレスを公表します
+ この設定項目が設定されていない場合、`addr` の値が使用されます。
+ デフォルト値: `""`

### `status-addr`

+ `HTTP` アドレスを介して TiKV の状態を直接レポートする設定項目

    > **警告:**
    >
    > この値が公開されると、TiKV サーバーの状態情報が漏洩する可能性があります。

+ 状態アドレスを無効にするには、値を `""` に設定します。
+ デフォルト値: `"127.0.0.1:20180"`

### `status-thread-pool-size`

+ `HTTP` API サービスのワーカースレッド数
+ デフォルト値: `1`
+ 最小値: `1`

### `grpc-compression-type`

+ gRPC メッセージの圧縮アルゴリズム
+ オプション値: `"none"`, `"deflate"`, `"gzip"`
+ デフォルト値: `"none"`

### `grpc-concurrency`

+ gRPC ワーカースレッドの数。gRPC スレッドプールのサイズを変更する場合は、[TiKV スレッドプールのパフォーマンスチューニング](/tune-tikv-thread-performance.md#performance-tuning-for-tikv-thread-pools) を参照してください。
+ デフォルト値: `5`
+ 最小値: `1`

### `grpc-concurrent-stream`

+ 一度に許可されるgRPCストリーム内の最大同時リクエスト数
+ デフォルト値: `1024`
+ 最小値: `1`

### `grpc-memory-pool-quota`

+ gRPCが使用できるメモリサイズを制限
+ デフォルト値: 制限なし
+ OOMが観察された場合はメモリを制限します。使用量を制限すると、潜在的な停止が発生する可能性があることに注意してください

### `grpc-raft-conn-num`

+ Raft通信のためのTiKVノード間リンクの最大数
+ デフォルト値: `1`
+ 最小値: `1`

### `max-grpc-send-msg-len`

+ 送信できるgRPCメッセージの最大長を設定
+ デフォルト値: `10485760`
+ 単位: バイト
+ 最大値: `2147483647`

### `grpc-stream-initial-window-size`

+ gRPCストリームのウィンドウサイズ
+ デフォルト値: `2MB`
+ 単位: KB|MB|GB
+ 最小値: `"1KB"`

### `grpc-keepalive-time`

+ gRPCが`keepalive` Pingメッセージを送信する時間間隔
+ デフォルト値: `"10s"`
+ 最小値: `"1s"`

### `grpc-keepalive-timeout`

+ gRPCストリームのタイムアウトを無効にする
+ デフォルト値: `"3s"`
+ 最小値: `"1s"`

### `concurrent-send-snap-limit`

+ 同時に送信されるスナップショットの最大数
+ デフォルト値: `32`
+ 最小値: `1`

### `concurrent-recv-snap-limit`

+ 同時に受信されるスナップショットの最大数
+ デフォルト値: `32`
+ 最小値: `1`

### `end-point-recursion-limit`

+ TiKVがCoprocessor DAG式をデコードする際に許可される再帰レベルの最大数
+ デフォルト値: `1000`
+ 最小値: `1`

### `end-point-request-max-handle-duration`

+ TiDBのプッシュダウンリクエストがTiKVに処理タスクを処理するために許可される最長の期間
+ デフォルト値: `"60s"`
+ 最小値: `"1s"`

### `snap-io-max-bytes-per-sec`

+ スナップショットの処理時の最大許容ディスク帯域幅
+ デフォルト値: `"100MB"`
+ 単位: KB|MB|GB
+ 最小値: `"1KB"`

### `enable-request-batch`

+ リクエストをバッチで処理するかどうかを決定します
+ デフォルト値: `true`

### `labels`

+ `{ zone = "us-west-1", disk = "ssd" }`など、サーバー属性を指定します。
+ デフォルト値: `{}`

### `background-thread-count`

+ バックグラウンドプールの作業スレッド数。エンドポイントスレッド、BRスレッド、分割チェックスレッド、リージョンスレッド、遅延に対して感度の低いタスクの他のスレッドを含みます。
+ デフォルト値: CPUコア数が16未満の場合、デフォルト値は `2` です。それ以上の場合は、デフォルト値は `3` です。

### `end-point-slow-log-threshold`

+ TiDBのプッシュダウンリクエストがスローログを出力するための時間閾値。処理時間がこの閾値を超えると、スローログが出力されます。
+ デフォルト値: `"1s"`
+ 最小値: `0`

### `raft-client-queue-size`

+ TiKVのRaftメッセージのキューサイズを指定します。送信されないメッセージが多すぎてバッファがいっぱいになるか、メッセージが破棄された場合は、大きな値を指定してシステムの安定性を向上させることができます。
+ デフォルト値: `8192`

### `simplify-metrics` <span class="version-mark">v6.2.0で新規追加</span>

+ 返されるモニタリングメトリクスを簡素化するかどうかを指定します。`true`に設定した場合、TiKVはいくつかのメトリクスをフィルタリングして返されるデータ量を減らします。
+ デフォルト値: `false`

### `forward-max-connections-per-address` <span class="version-mark">v5.0.0で新規追加</span>

+ サービスおよびサーバーへの転送リクエストのための接続プールのサイズを設定します。値を小さく設定すると、リクエスト遅延や負荷分散に影響が出る可能性があります。
+ デフォルト値: `4`

## readpool.unified

単一のスレッドプールに関連する設定項目。このスレッドプールは、4.0バージョン以降、元のストレージスレッドプールおよびコプロセッサスレッドプールに取って代わります。

### `min-thread-count`

+ 統一読み取りプールの最小作業スレッド数
+ デフォルト値: `1`

### `max-thread-count`

+ 統一読み取りプールまたはUnifyReadPoolスレッドプールの最大作業スレッド数。このスレッドプールのサイズを変更する際は、[TiKVスレッドプールのパフォーマンスチューニング](/tune-tikv-thread-performance.md#performance-tuning-for-tikv-thread-pools)を参照してください。
+ 値の範囲: `[min-thread-count, MAX(4, CPUクォータ * 10)]`。`MAX(4, CPUクォータ * 10)` は、`4` および `CPUクォータ * 10` のうち大きい値を取ります。
+ デフォルト値: `MAX(4, CPU * 0.8)`

> **注意:**
>
> スレッド数を増やすと、より多くのコンテキスト切り替えが発生し、パフォーマンスが低下する可能性があります。この設定項目の値を変更することは推奨されません。

### `stack-size`

+ 統一スレッドプールのスレッドのスタックサイズ
+ タイプ: 整数 + 単位
+ デフォルト値: `"10MB"`
+ 単位: KB|MB|GB
+ 最小値: `"2MB"`
+ 最大値: システムで実行された`ulimit -sH`コマンドの結果によって出力されたKバイトの数

### `max-tasks-per-worker`

+ 統一読み取りプールの単一スレッドで許可される最大タスク数。この値を超えると `Server Is Busy` が返されます。
+ デフォルト値: `2000`
+ 最小値: `2`

### `auto-adjust-pool-size` <span class="version-mark">v6.3.0で新規追加</span>

+ スレッドプールのサイズを自動的に調整するかどうかを制御します。有効にすると、TiKVの読み取りパフォーマンスが、現在のCPU使用率に基づいてUnifyReadPoolスレッドプールのサイズを自動的に調整することで最適化されます。スレッドプールの可能な範囲は `[max-thread-count, MAX(4, CPU)]` です。最大値は [`max-thread-count`](#max-thread-count)のものと同じです。
+ デフォルト値: `false`

## readpool.storage

ストレージスレッドプールに関連する設定項目。

### `use-unified-pool`

+ ストレージリクエストに統一スレッドプール（[`readpool.unified`](#readpoolunified)で構成される）を使用するかどうかを決定します。このパラメータの値が `false` の場合は、このセクション(`readpool.storage`)に他の設定がない場合、デフォルト値は `true` です。それ以外の場合は、後方互換性につき、デフォルト値は `false` です。このオプションを有効にする前に、[`readpool.unified`](#readpoolunified)で設定を変更してください。
+ デフォルト値: もしこのセクション(`readpool.storage`)に他の設定がない場合は `true`。それ以外の場合は後方互換性につき `false`。このオプションを有効にする前に、[`readpool.unified`](#readpoolunified)で設定を変更してください。

### `high-concurrency`

+ 高優先度の `read` リクエストを処理するための許容可能な同時スレッド数
+ `8` ≤ `CPU数` ≤ `16` の場合、デフォルト値は `cpu_num * 0.5`。CPU数が `8` より小さい場合は、デフォルト値は `4`。CPU数が `16` より大きい場合は、デフォルト値は `8`。
+ 最小値: `1`

### `normal-concurrency`

+ 通常の優先度の `read` リクエストを処理するための許容可能な同時スレッド数
+ `8` ≤ `CPU数` ≤ `16` の場合、デフォルト値は `cpu_num * 0.5`。CPU数が `8` より小さい場合は、デフォルト値は `4`。CPU数が `16` より大きい場合は、デフォルト値は `8`。
+ 最小値: `1`

### `low-concurrency`

+ 低優先度の `read` リクエストを処理するための許容可能な同時スレッド数
+ `8` ≤ `CPU数` ≤ `16` の場合、デフォルト値は `cpu_num * 0.5`。CPU数が `8` より小さい場合は、デフォルト値は `4`。CPU数が `16` より大きい場合は、デフォルト値は `8`。
+ 最小値: `1`

### `max-tasks-per-worker-high`

+ 高優先度スレッドプールの単一スレッドで許可される最大タスク数。この値を超えると `Server Is Busy` が返されます。
+ デフォルト値: `2000`
+ 最小値: `2`

### `max-tasks-per-worker-normal`

+ 通常優先度スレッドプールの単一スレッドで許可される最大タスク数。この値を超えると `Server Is Busy` が返されます。
+ デフォルト値: `2000`
+ 最小値: `2`

### `max-tasks-per-worker-low`

+ 低優先度スレッドプールの単一スレッドで許可される最大タスク数。この値を超えると `Server Is Busy` が返されます。
+ デフォルト値: `2000`
+ 最小値: `2`

### `stack-size`

+ ストレージ読み取りスレッドプールのスレッドのスタックサイズ
+ タイプ: 整数 + 単位
+ デフォルト値: `"10MB"`
+ 単位: KB|MB|GB
+ 最小値: `"2MB"`
+ 最大値: システムで実行される `ulimit -sH` コマンドの結果におけるKバイトの数。

## `readpool.coprocessor`

Coprocessorスレッドプールに関連する構成項目。

### `use-unified-pool`

+ コプロセッサリクエストに統一スレッドプール（[`readpool.unified`](#readpoolunified)で構成）を使用するかどうかを決定します。このパラメータの値が `false` である場合、このセクション（`readpool.coprocessor`）での残りのパラメータを使用して別々のスレッドプールが使用されます。
+ デフォルト値: このセクション（`readpool.coprocessor`）のパラメータが一つも設定されていない場合は `true` です。それ以外の場合、後方互換性のため、デフォルト値は `false` です。このパラメータを有効にする前に、[`readpool.unified`](#readpoolunified)の構成項目を調整してください。

### `high-concurrency`

+ チェックポイントなどの高優先度のコプロセッサリクエストを処理する同時スレッドの許容数
+ デフォルト値: `CPU * 0.8`
+ 最小値: `1`

### `normal-concurrency`

+ 通常優先度のコプロセッサリクエストを処理する同時スレッドの許容数
+ デフォルト値: `CPU * 0.8`
+ 最小値: `1`

### `low-concurrency`

+ テーブルスキャンなどの低優先度のコプロセッサリクエストを処理する同時スレッドの許容数
+ デフォルト値: `CPU * 0.8`
+ 最小値: `1`

### `max-tasks-per-worker-high`

+ 高優先度スレッドプール内の単一スレッドに許容されるタスク数。この数を超えると、「Server Is Busy」と返されます。
+ デフォルト値: `2000`
+ 最小値: `2`

### `max-tasks-per-worker-normal`

+ 通常優先度スレッドプール内の単一スレッドに許容されるタスク数。この数を超えると、「Server Is Busy」と返されます。
+ デフォルト値: `2000`
+ 最小値: `2`

### `max-tasks-per-worker-low`

+ 低優先度スレッドプール内の単一スレッドに許容されるタスク数。この数を超えると、「Server Is Busy」と返されます。
+ デフォルト値: `2000`
+ 最小値: `2`

### `stack-size`

+ コプロセッサスレッドプール内のスレッドのスタックサイズ
+ タイプ: 整数 + 単位
+ デフォルト値: `"10MB"`
+ 単位: KB|MB|GB
+ 最小値: `"2MB"`
+ 最大値: システムで実行される `ulimit -sH` コマンドの結果におけるKバイトの数。

## storage

ストレージに関連する構成項目。

### `data-dir`

+ RocksDBディレクトリのストレージパス
+ デフォルト値: `"./"`

### `engine` <span class="version-mark">v6.6.0で新規</span>

> **注意:**
>
> この機能は実験的です。本番環境で使用することはお勧めしません。この機能は予告なく変更または削除されるかもしれません。バグが見つかった場合は、GitHubの[issue](https://github.com/pingcap/tidb/issues)で報告できます。

+ エンジンタイプを指定します。この構成は新しいクラスタを作成する際にのみ指定でき、一度指定すると変更できません。
+ デフォルト値: `"raft-kv"`
+ 値のオプション:

    + `"raft-kv"`: TiDB v6.6.0より前のバージョンでのデフォルトエンジンタイプ。
    + `"partitioned-raft-kv"`: TiDB v6.6.0で導入された新しいストレージエンジンタイプ。

### `scheduler-concurrency`

+ キーに対する同時操作を防ぐ組み込みメモリロックメカニズム。各キーには異なるスロットのハッシュがあります。
+ デフォルト値: `52,4288`
+ 最小値: `1`

### `scheduler-worker-pool-size`

+ スケジューラスレッドプール内のスレッド数。スケジューラスレッドは主にデータ書き込み前にトランザクション整合性を確認するために使用されます。CPUコア数が `16` 以上の場合、デフォルト値は `8` です。それ以外の場合、デフォルト値は `4` です。スケジューラスレッドプールのサイズを変更する場合は、[Performance tuning for TiKV thread pools](/tune-tikv-thread-performance.md#performance-tuning-for-tikv-thread-pools)を参照してください。
+ デフォルト値: `4`
+ 値の範囲: `[1, MAX(4, CPU)]`。`MAX(4, CPU)` では、`CPU` はCPUコア数を意味します。`MAX(4, CPU)` は `4` と `CPU` の大きい方の値を取ります。

### `scheduler-pending-write-threshold`

+ 書き込みキューの最大サイズ。この値を超えると、新しい書き込みで TiKV への `Server Is Busy` エラーが返されます。
+ デフォルト値: `"100MB"`
+ 単位: MB|GB

### `enable-async-apply-prewrite`

+ Asyncコミットトランザクションが事前書き込みリクエストを適用する前にTiKVクライアントへ応答するかどうかを決定します。この構成項目を有効にした後は、適用期間が長い場合には遅延が簡単に軽減されるか、適用期間が安定していない場合には遅延の揺れを軽減できます。
+ デフォルト値: `false`

### `reserve-space`

+ TiKVが開始される際、ディスク保護としてディスクに一定のスペースを予約します。残りのディスクスペースが予約スペースより少なくなると、TiKVは一部の書き込み操作を制限します。予約スペースは2つの部分に分かれており、予約スペースの80%はディスクスペースが不足した場合に必要な余分なディスクスペースとして使用され、残りの20%は一時ファイルの保存に使用されます。スペースを回収するプロセス中に、余分なディスクスペースの使用によりストレージが枯渇すると、この一時ファイルはサービスの復元のための最終保護として機能します。
+ 一時ファイルの名前は `space_placeholder_file` で、`storage.data-dir` ディレクトリにあります。TiKVがディスクスペースが不足してオフラインになると、TiKVを再起動すると、一時ファイルは自動的に削除され、TiKVはスペースを回収しようとします。
+ 残りのスペースが不足すると、TiKVは一時ファイルを作成しません。保護の効果は予約スペースのサイズに関連しています。予約スペースのサイズは、ディスク容量の5%とこの構成値の大きい方の値との間の大きい方の値です。この構成項目の値が `"0MB"` の場合は、TiKVはこのディスク保護機能を無効にします。
+ デフォルト値: `"5GB"`
+ 単位: MB|GB

### `enable-ttl`

> **注意:**
>
> - 新しい TiKV クラスタを展開するときだけ、`enable-ttl` を `true` または `false` に設定してください。既存の TiKV クラスタでこの構成項目の値を変更しないでください。`enable-ttl` の値が異なる TiKV クラスタは異なるデータ形式を使用します。そのため、既存の TiKV クラスタでこの項目の値を変更すると、「can't enable TTL on a non-ttl」エラーが発生します。
> - `enable-ttl` は TiKV クラスタでのみ使用してください。TiDBノードがあるクラスタで(`enable-ttl` の値を `true` に設定することで、) この構成項目を使用しないでください。そうすると、データの破損やTiDBクラスタのアップグレードの失敗などの重大な問題が発生します。

+ TTLは「Time to live」の略です。このアイテムが有効になっている場合、TiKVはTTLに達したデータを自動的に削除します。TTLの値を設定するには、クライアント経由でデータを書き込む際にリクエストで指定する必要があります。TTLが指定されていない場合、TiKVは対応するデータを自動的に削除しません。
+ デフォルト値: `false`

### `ttl-check-poll-interval`

+ 物理的なスペース回収を行うためのデータのチェック間隔。データがTTLに達すると、TiKVはチェック中にその物理的なスペースを強制的に回収します。
+ デフォルト値: `"12h"`
+ 最小値: `"0s"`

### `background-error-recovery-window` <span class="version-mark">v6.1.0で新規</span>

+ TiKVがRocksDBで回復可能なバックグラウンドエラーを検出した後に回復を行うことができる最大許容時間。一部のバックグラウンドSSTファイルが損傷している場合、RocksDBは、損傷しているSSTファイルが属するPeerを特定した後、PDにハートビート経由で通知します。PDはこのPeerを削除するスケジューリング操作を行います。最終的に、損傷したSSTファイルは直接削除され、TiKVバックグラウンドは通常通り動作します。
+ 回復が完了するまで、損傷したSSTファイルは存在します。この期間中、RocksDBはデータの一部が読まれるとエラーが発生しますが、データの書き込みは継続できます。
+ この時間枠内に回復が完了しない場合、TiKVはパニック状態になります。
+ デフォルト値: 1h

### `api-version` <span class="version-mark">v6.1.0で新規</span>

+ TiKVがRawKVストアとしてサービスする際に使用するストレージ形式とインターフェースバージョン。
+ 値のオプション:
    + `1`: API V1を使用し、クライアントから渡されたデータをエンコードせず、そのままデータを保存します。v6.1.0より前のバージョンでは、デフォルトでAPI V1を使用します。
    + `2`: API V2を使用します。
    + データは、PD（TSO）からtikv-serverによって取得されるタイムスタンプを使用したMulti-Version Concurrency Control（MVCC）形式で格納されます。
    + データは、異なる使用およびAPI V2によるスコープがあり、単一クラスタ内でTiDB、Transactional KV、およびRawKVアプリケーションの共存をサポートするAPI V2があります。
    + API V2を使用する場合、同時に `storage.enable-ttl = true` を設定する必要があります。なぜなら、API V2はTTL機能をサポートしており、`enable-ttl` を明示的にオンにする必要があるからです。そうしないと、 `storage.enable-ttl` はデフォルトで `false` になるためコンフリクトが発生します。
    + API V2を有効にすると、少なくとも1つのtidb-serverインスタンスを展開して古いデータを回収する必要があります。このtidb-serverインスタンスは読み書きサービスを同時に提供できます。高可用性を確保するために、複数のtidb-serverインスタンスを展開することができます。
    + API V2にはクライアントサポートが必要です。詳細については、API V2の対応するクライアントの手順書を参照してください。
    + v6.2.0以降、RawKVのChange Data Capture（CDC）がサポートされています。[RawKV CDC](https://tikv.org/docs/latest/concepts/explore-tikv-features/cdc/cdc)を参照してください。
    + デフォルト値: `1`

    > **警告:**

    > - API V1とAPI V2はストレージ形式が異なります。TiKVがTiDBデータのみを含む場合、API V2を直接有効または無効にすることができます**ただし**、他のシナリオでは新しいクラスタを展開し、[RawKVバックアップ＆リストア](https://tikv.org/docs/latest/concepts/explore-tikv-features/backup-restore/)を使用してデータを移行する必要があります。
    > - API V2を有効にした後は、TiKVクラスタをv6.1.0よりも古いバージョンにダウングレードすることは**できません**。それを行った場合、データの破損が発生する可能性があります。

## storage.block-cache

複数のRocksDB Column Families（CF）間でのブロックキャッシュの共有に関連する構成項目。

### `capacity`

+ 共有ブロックキャッシュのサイズ。
+ デフォルト値:

    + `storage.engine="raft-kv"`の場合、デフォルト値は総システムメモリサイズの45%です。
    + `storage.engine="partitioned-raft-kv"`の場合、デフォルト値は総システムメモリサイズの30%です。

+ 単位: KB|MB|GB

## storage.flow-control

TiKVでのフローコントロールメカニズムに関連する構成項目。このメカニズムは、RocksDBの書き込みスタールメカニズムを置き換え、スケジューラーレイヤーでフローを制御し、ストールしたRaftstoreやApplyスレッドによる二次的な障害を回避します。

### `enable`

+ フローコントロールメカニズムを有効にするかどうかを決定します。有効にすると、TiKVは自動的にKvDBの書き込みスタールメカニズムとRaftDBの書き込みスタールメカニズム（メンテーブルを除く）を無効にします。
+ デフォルト値: `true`

### `memtables-threshold`

+ kvDBメンテーブルの数がこのしきい値に達すると、フローコントロールメカニズムが動作を開始します。 `enable` が `true` に設定されている場合、この構成項目は`rocksdb.(defaultcf|writecf|lockcf).max-write-buffer-number`を上書きします。
+ デフォルト値: `5`

### `l0-files-threshold`

+ kvDBのL0ファイル数がこのしきい値に達すると、フローコントロールメカニズムが動作を開始します。 `enable` が `true` に設定されている場合、この構成項目は`rocksdb.(defaultcf|writecf|lockcf).level0-slowdown-writes-trigger`を上書きします。
+ デフォルト値: `20`

### `soft-pending-compaction-bytes-limit`

+ KvDB内の保留中のコンパクションバイトがこのしきい値に達すると、フローコントロールメカニズムがいくつかの書き込みリクエストを拒否し、「ServerIsBusy」エラーを報告します。 `enable` が `true` に設定されている場合、この構成項目は`rocksdb.(defaultcf|writecf|lockcf).soft-pending-compaction-bytes-limit`を上書きします。
+ デフォルト値: `"192GB"`

### `hard-pending-compaction-bytes-limit`

+ KvDB内の保留中のコンパクションバイトがこのしきい値に達すると、フローコントロールメカニズムはすべての書き込みリクエストを拒否し、「ServerIsBusy」エラーを報告します。 `enable` が `true` に設定されている場合、この構成項目は`rocksdb.(defaultcf|writecf|lockcf).hard-pending-compaction-bytes-limit`を上書きします。
+ デフォルト値: `"1024GB"`

## storage.io-rate-limit

I/Oレート制限に関連する構成項目。

### `max-bytes-per-sec`

+ サーバーが1秒間にディスクから書き込むまたは読み込むことができる最大I/Oバイトを制限します（以下の`mode`構成項目によって決定されます）。この制限に達すると、TiKVはフォアグラウンドの操作よりもバックグラウンドの操作を制限します。この構成項目の値は、ディスクの最適なI/O帯域幅、たとえばクラウドディスクベンダーが指定した最大I/O帯域幅に設定する必要があります。この構成値をゼロに設定した場合、ディスクI/O操作は制限されません。
+ デフォルト値: `"0MB"`

### `mode`

+ I/O操作の種類が`max-bytes-per-sec`の閾値以下で制約されるかどうかを決定します。現在は書き込み専用モードのみがサポートされています。
+ 値のオプション: `"read-only"`、`"write-only"`、`"all-io"`
+ デフォルト値: `"write-only"`

## pd

### `endpoints`

+ PDのエンドポイント。複数のエンドポイントが指定されている場合、コンマで区切ってそれらを分離する必要があります。
+ デフォルト値: `["127.0.0.1:2379"]`

### `retry-interval`

+ PD接続のリトライ間隔。
+ デフォルト値: `"300ms"`

### `retry-log-every`

+ PDクライアントがエラーを観測した場合に、クライアントがエラーを報告する頻度を指定します。たとえば、値が`5`の場合、PDクライアントがエラーを観測した後、クライアントはエラーを報告せず、5回目ごとにエラーを報告します。
+ この機能を無効にするには、値を`1`に設定します。
+ デフォルト値: `10`

### `retry-max-count`

+ PD接続の初期化をリトライする最大回数。
+ リトライを無効にするには、値を`0`に設定します。リトライの制限を解除するには、値を`-1`に設定します。
+ デフォルト値: `-1`

## raftstore

Raftstoreに関連する構成項目。

### `prevote`

+ `prevote`を有効または無効にします。この機能を有効にすると、ネットワーク分断からの回復後のシステム上のジッタを減らすのに役立ちます。
+ デフォルト値: `true`

### `capacity`

+ 格納容量。これはデータを格納するために許可される最大サイズです。 `capacity`が指定されていない場合、現在のディスクの容量が優先されます。同一物理ディスク上に複数のTiKVインスタンスを展開する場合は、TiKV構成にこのパラメータを追加してください。詳細については [ハイブリッド展開の主要パラメータ](/hybrid-deployment-topology.md#key-parameters) を参照してください。
+ デフォルト値: `0`
+ 単位: KB|MB|GB

### `raftdb-path`

+ Raftライブラリのパス。デフォルトでは `storage.data-dir/raft` です。
+ デフォルト値: `""`

### `raft-base-tick-interval`

> **注意:**

> この構成項目はSQLステートメントでクエリできませんが、設定ファイルで構成できます。

+ Raftステートマシンがチックする時間間隔。
+ デフォルト値: `"1s"`
+ 最小値: `0`より大きい

### `raft-heartbeat-ticks`

> **注意:**

> この構成項目はSQLステートメントでクエリできませんが、設定ファイルで構成できます。

+ Raftハートビートを送信する経過したチックの数。これは、Raftベースティック間隔の時間間隔が `raft-base-tick-interval` * `raft-heartbeat-ticks` であることを意味します。
+ デフォルト値: `2`
+ 最小値: `0`より大きい

### `raft-election-timeout-ticks`

> **注意:**

> この構成項目はSQLステートメントでクエリできませんが、設定ファイルで構成できます。

+ Raft選挙が開始される経過したチックの数。これは、Raftグループがリーダーを欠場する場合、Raft選挙が`raft-base-tick-interval` * `raft-election-timeout-ticks` の時間間隔で開始されることを意味します。
+ デフォルト値: `10`
+ 最小値: `raft-heartbeat-ticks`

### `raft-min-election-timeout-ticks`

> **注意:**

> この構成項目はSQLステートメントでクエリできませんが、設定ファイルで構成できます。

+ Raft選挙が開始される最小のチック数。数が`0`の場合、`raft-election-timeout-ticks`の値が使用されます。このパラメータの値は、`raft-election-timeout-ticks`より大きいか等しい必要があります。
+ デフォルト値: `0`
+ 最小値: `0`

### `raft-max-election-timeout-ticks`

> **注意:**

> この構成項目はSQLステートメントでクエリできませんが、設定ファイルで構成できます。

+ Raft選挙が開始される最大のチック数。数が`0`の場合、`raft-election-timeout-ticks` * `2`の値が使用されます。
+ デフォルト値: `0`
+ 最小値: `0`
```markdown
+ この設定項目はSQLステートメントを使用してクエリすることはできませんが、設定ファイルで構成することができます。

    + 単一のメッセージパケットのサイズのソフトリミット
    + デフォルト値："1MB"
    + 最小値：`0`より大きい
    + 最大値：`3GB`
    + 単位：KB|MB|GB

### `raft-max-inflight-msgs`

> **注意:**
>
> この設定項目はSQLステートメントを使用してクエリすることはできませんが、設定ファイルで構成することができます。

    + 確認するRaftログの数。この数を超えると、Raft状態機械がログの送信を遅らせます。
    + デフォルト値: `256`
    + 最小値：`0`より大きい
    + 最大値：`16384`

### `raft-entry-max-size`

    + 単一のログの最大サイズのハードリミット
    + デフォルト値："8MB"
    + 最小値：`0`
    + 単位：MB|GB

### `raft-log-compact-sync-interval` <span class="version-mark">v5.3で新規</span>

    + 不要なRaftログをコンパクトにするための時間間隔
    + デフォルト値："2s"
    + 最小値："0s"

### `raft-log-gc-tick-interval`

    + Raftログを削除するポーリングタスクをスケジュールする時間間隔。`0`の場合、この機能は無効です。
    + デフォルト値："3s"
    + 最小値："0s"

### `raft-log-gc-threshold`

    + 残存Raftログの最大許容数のソフトリミット
    + デフォルト値: `50`
    + 最小値: `1`

### `raft-log-gc-count-limit`

    + 残存Raftログの許容数のハードリミット
    + デフォルト値: リージョンサイズの3/4に格納できるログの数 (それぞれのログのサイズを1MBとして計算)
    + 最小値：`0`

### `raft-log-gc-size-limit`

    + 残存Raftログの最大許容サイズのハードリミット
    + デフォルト値: リージョンサイズの3/4
    + 最小値：`0`より大きい

### `raft-log-reserve-max-ticks` <span class="version-mark">v5.3で新規</span>

    + この設定項目で設定されたティック数の後に、`raft-log-gc-threshold`で設定された値に到達しなくても、TiKVはそれらのログに対してガベージコレクション（GC）を実行します。
    + デフォルト値: `6`
    + 最小値: `0`より大きい

### `raft-engine-purge-interval`

    + ディスクスペースをできるだけ早く回収するために、古いTiKVログファイルを削除する間隔。Raft engineは置換可能なコンポーネントであるため、一部の実装では削除処理が必要です。
    + デフォルト値："10s"

### `raft-entry-cache-life-time`

    + メモリ内のログキャッシュに許可される最大残り有効時間
    + デフォルト値："30s"
    + 最小値：`0`

### `hibernate-regions`

    + ハイバネートリージョンを有効または無効にします。このオプションを有効にすると、長時間アイドル状態のリージョンは自動的にハイバネートされます。これにより、アイドル状態のリージョンの間でRaftリーダーとフォロワー間のハートビートメッセージによる余分なオーバーヘッドが減少します。ハイバネートされたリージョンのリーダーとフォロワー間のハートビート間隔を変更するには、`peer-stale-state-check-interval`を使用できます。
    + デフォルト値: v5.0.2およびそれ以降のバージョンでは`true`; v5.0.2より前のバージョンでは`false`

### `split-region-check-tick-interval`

    + リージョンの分割が必要かどうかを確認する間隔。`0`の場合、この機能は無効です。
    + デフォルト値："10s"
    + 最小値：`0`

### `region-split-check-diff`

    + リージョンデータが分割する前に許可される最大値
    + デフォルト値: リージョンサイズの1/16
    + 最小値: `0`

### `region-compact-check-interval`

    + 手動でRocksDBのコンパクションをトリガーする必要があるかどうかを確認する間隔。 `0`の場合、この機能は無効です。
    + デフォルト値："5m"
    + 最小値：`0`

### `region-compact-check-step`

    + 1回の手動コンパクションで確認されるリージョンの数
    + デフォルト値:

        + `storage.engine="raft-kv"`の場合、デフォルト値は`100`。
        + `storage.engine="partitioned-raft-kv"`の場合、デフォルト値は`5`。

    + 最小値：`0`

### `region-compact-min-tombstones`

    + RocksDBのコンパクションをトリガーするために必要な墓石の数
    + デフォルト値: `10000`
    + 最小値：`0`

### `region-compact-tombstones-percent`

    + RocksDBのコンパクションをトリガーするために必要な墓石の割合
    + デフォルト値: `30`
    + 最小値：`1`
    + 最大値：`100`

### `region-compact-min-redundant-rows` <span class="version-mark">v7.1.0で新規</span>

    + RocksDBコンパクションをトリガーするために必要な冗長なMVCC行の数
    + デフォルト値: `50000`
    + 最小値：`0`

### `region-compact-redundant-rows-percent` <span class="version-mark">v7.1.0で新規</span>

    + RocksDBコンパクションをトリガーするために必要な冗長なMVCC行の割合
    + デフォルト値: `20`
    + 最小値：`1`
    + 最大値：`100`

### `report-region-buckets-tick-interval` <span class="version-mark">v6.1.0で新規</span>

> **警告:**
>
> `report-region-buckets-tick-interval`はTiDB v6.1.0で導入された実験的な機能です。本番環境で使用することは推奨されません。

    + `enable-region-bucket`がtrueの場合、TiKVがPDに対してバケット情報を報告する間隔
    + デフォルト値: `"10s"`

### `pd-heartbeat-tick-interval`

    + リージョンのPDへのハートビートがトリガーされる時間間隔。 `0`の場合、この機能は無効です。
    + デフォルト値: `"1m"`
    + 最小値：`0`

### `pd-store-heartbeat-tick-interval`

    + ストアがPDへのハートビートをトリガーする時間間隔。 `0`の場合、この機能は無効です。
    + デフォルト値: `"10s"`
    + 最小値：`0`

### `snap-mgr-gc-tick-interval`

    + 期限切れのスナップショットファイルをリサイクルする間隔。 `0`の場合、この機能は無効です。
    + デフォルト値: `"1m"`
    + 最小値：`0`

### `snap-gc-timeout`

    + スナップショットファイルが保存される最長時間
    + デフォルト値: `"4h"`
    + 最小値：`0`

### `snap-generator-pool-size` <span class="version-mark">v5.4.0で新規</span>

    + `snap-generator`スレッドプールのサイズを構成する
    + 復旧シナリオでTiKV内のリージョンがより速くスナップショットを生成するために、対応するワーカーの`snap-generator`スレッドの数を増やす必要があります。この設定項目を使用して、`snap-generator`スレッドプールのサイズを増やすことができます。
    + デフォルト値: `2`
    + 最小値：`1`

### `lock-cf-compact-interval`

    + Lock Column FamilyのためにTiKVが手動でコンパクションをトリガーする時間間隔
    + デフォルト値: `"10m"`
    + 最小値：`0`

### `lock-cf-compact-bytes-threshold`

    + Lock Column FamilyのためにTiKVが手動でコンパクションをトリガーするサイズの閾値
    + デフォルト値: `"256MB"`
    + 最小値：`0`
    + 単位: MB

### `notify-capacity`

    + リージョンメッセージキューの最長長
    + デフォルト値: `40960`
    + 最小値：`0`

### `messages-per-tick`

    + バッチごとに処理されるメッセージの最大数
    + デフォルト値: `4096`
    + 最小値：`0`

### `max-peer-down-duration`

    + ピアの許容される最長の非アクティブ期間。タイムアウトしたピアは`down`とマークされ、後でPDがそれを削除しようとします。
    + デフォルト値: `"10m"`
    + 最小値: ハイバネートリージョンが有効な場合、最小値は`peer-stale-state-check-interval * 2`です。ハイバネートリージョンが無効な場合、最小値は`0`です。

### `max-leader-missing-duration`

    + Raftグループがリーダーを欠落している状態の最長期間。この値を超えると、ピアはPDに対してそのピアが削除されたかどうかを確認します。
    + デフォルト値: `"2h"`
    + 最小値: `abnormal-leader-missing-duration`より大きい

### `abnormal-leader-missing-duration`

    + Raftグループがリーダーを欠落している状態の最長期間。この値を超えると、ピアは異常と見なされ、メトリクスとログにマークされます。
    + デフォルト値: `"10m"`
    + 最小値: `peer-stale-state-check-interval`より大きい
```
### `peer-stale-state-check-interval`

+ ピアがRaftグループでリーダーが欠落している状態かどうかをチェックするための間隔です。
+ デフォルト値：`"5m"`
+ 最小値：`2 * election-timeout`より大きい

### `leader-transfer-max-log-lag`

+ Raftリーダーの転送時における許容される欠落ログの最大数です。
+ デフォルト値：`128`
+ 最小値：`10`

### `max-snapshot-file-raw-size` <span class="version-mark">v6.1.0で新規追加</span>

+ スナップショットファイルのサイズがこの設定値を超えると、このファイルは複数のファイルに分割されます。
+ デフォルト値：`100MiB`
+ 最小値：`100MiB`

### `snap-apply-batch-size`

+ インポートされたスナップショットファイルがディスクに書き込まれる際に必要なメモリキャッシュサイズです。
+ デフォルト値：`"10MB"`
+ 最小値：`0`
+ 単位：MB

### `consistency-check-interval`

> **警告:**
>
> 本番環境で整合性チェックを有効にすることは**推奨されません**。これはクラスタのパフォーマンスに影響を与え、TiDBのガベージコレクションと互換性がありません。

+ 整合性チェックがトリガーされる間隔です。 `0` はこの機能が無効であることを意味します。
+ デフォルト値：`"0s"`
+ 最小値：`0`

### `raft-store-max-leader-lease`

+ Raftリーダーの最長信頼期間です。
+ デフォルト値：`"9s"`
+ 最小値：`0`

### `right-derive-when-split`

+ リージョンが分割される際の新しいリージョンの開始キーを指定します。この構成項目が`true`に設定されている場合、開始キーは最大の分割キーになります。この構成項目が`false`に設定されている場合、開始キーは元のリージョンの開始キーになります。
+ デフォルト値：`true`

### `merge-max-log-gap`

+ `merge`操作を行う際に許容される最大の欠落ログの数です。
+ デフォルト値：`10`
+ 最小値：`raft-log-gc-count-limit`より大きい

### `merge-check-tick-interval`

+ TiKVがリージョンのマージが必要かどうかをチェックする間隔です。
+ デフォルト値：`"2s"`
+ 最小値：`0`より大きい

### `use-delete-range`

+ `rocksdb delete_range` インターフェイスからデータを削除するかどうかを決定します。
+ デフォルト値：`false`

### `cleanup-import-sst-interval`

+ 期限切れのSSTファイルがチェックされる間隔です。 `0` はこの機能が無効であることを意味します。
+ デフォルト値：`"10m"`
+ 最小値：`0`

### `local-read-batch-size`

+ 1つのバッチで処理される読み取りリクエストの最大数です。
+ デフォルト値：`1024`
+ 最小値：`0`より大きい

### `apply-yield-write-size` <span class="version-mark">v6.4.0で新規追加</span>

+ Applyスレッドが1回のポーリングで1つのFSM（有限状態機械）に書き込むことができる最大バイト数です。これはソフトリミットです。
+ デフォルト値：`"32KiB"`
+ 最小値：`0`より大きい
+ 単位：KiB|MiB|GiB

### `apply-max-batch-size`

+ Raft状態機械はBatchSystemによってデータ書き込みリクエストをバッチ処理します。この構成項目は、1つのバッチで処理できるRaft状態機械の最大数を指定します。
+ デフォルト値：`256`
+ 最小値：`0`より大きい
+ 最大値：`10240`

### `apply-pool-size`

+ データをディスクにフラッシュするスレッドプールのサイズであるApplyスレッドプールの許容可能なスレッド数です。このスレッドプールのサイズを変更する場合は、[TiKVスレッドプールのパフォーマンスチューニング](/tune-tikv-thread-performance.md#performance-tuning-for-tikv-thread-pools)を参照してください。
+ デフォルト値：`2`
+ 値の範囲：`[1, CPU * 10]`。`CPU` はCPUコア数を意味します。

### `store-max-batch-size`

+ Raft状態機械はBatchSystemによってログのフラッシュリクエストをバッチ処理します。この構成項目は、1つのバッチで処理できるRaft状態機械の最大数を指定します。
+ `hibernate-regions` が有効な場合、デフォルト値は `256` です。`hibernate-regions` が無効な場合、デフォルト値は `1024` です。
+ 最小値：`0`より大きい
+ 最大値：`10240`

### `store-pool-size`

+ Raftを処理するスレッドプールのサイズであるRaftstoreスレッドプールの許容可能なスレッド数です。このスレッドプールのサイズを変更する場合は、[TiKVスレッドプールのパフォーマンスチューニング](/tune-tikv-thread-performance.md#performance-tuning-for-tikv-thread-pools)を参照してください。
+ デフォルト値：`2`
+ 値の範囲：`[1, CPU * 10]`。`CPU` はCPUコア数を意味します。

### `store-io-pool-size` <span class="version-mark">v5.3.0で新規追加</span>

+ Raft I/Oタスクを処理するスレッドの許容可能な数であるStoreWriterスレッドプールのサイズです。このスレッドプールのサイズを変更する場合は、[TiKVスレッドプールのパフォーマンスチューニング](/tune-tikv-thread-performance.md#performance-tuning-for-tikv-thread-pools)を参照してください。
+ デフォルト値：`0`
+ 最小値：`0`

### `future-poll-size`

+ `future` を駆動するスレッドの許容可能な数です。
+ デフォルト値：`1`
+ 最小値：`0`より大きい

### `cmd-batch`

+ リクエストのバッチ処理を有効にするかどうかを制御します。有効にすると、書き込みパフォーマンスが大幅に向上します。
+ デフォルト値：`true`

### `inspect-interval`

+ 一定の間隔で、TiKVはRaftstoreコンポーネントのレイテンシーを検査します。このパラメーターは検査の間隔を指定します。もしレイテンシーがこの値を超えると、検査はタイムアウトとしてマークされます。
+ タイムアウト検査の比率に基づき、TiKVノードが遅いかどうかを判断します。
+ デフォルト値：`"500ms"`
+ 最小値：`"1ms"`

### `raft-write-size-limit` <span class="version-mark">v5.3.0で新規追加</span>

+ Raftデータをディスクに書き込むしきい値を決定します。この構成項目の値のデータサイズがこの値を超えると、データはディスクに書き込まれます。 `store-io-pool-size` の値が `0` の場合、この構成項目は効果を発揮しません。
+ デフォルト値：`1MB`
+ 最小値：`0`

### `report-min-resolved-ts-interval` <span class="version-mark">v6.0.0で新規追加</span>

+ 最小の解決されたタイムスタンプがPDリーダーに報告される間隔を決定します。この値を `0` に設定すると、報告が無効になります。
+ デフォルト値：v6.3.0以前は`"0s"`がデフォルト値です。v6.3.0以降は最小の正の値である`"1s"`がデフォルト値です。
+ 最小値：`0`
+ 単位：秒

### `evict-cache-on-memory-ratio` <span class="version-mark">v7.5.0で新規追加</span>

+ TiKVのメモリ使用量がシステムの利用可能なメモリの90%を超え、Raftエントリキャッシュによって使用されるメモリが使用済みメモリ * `evict-cache-on-memory-ratio` を超える場合、TiKVはRaftエントリキャッシュを追い出します。
+ この値を `0` に設定すると、この機能が無効になります。
+ デフォルト値：`0.1`
+ 最小値：`0`

## coprocessor

Coprocessorに関連する構成項目。

### `split-region-on-table`

+ テーブルごとにリージョンを分割するかどうかを決定します。TiDBモードでのみこの機能を使用することを推奨します。
+ デフォルト値：`false`

### `batch-split-limit`

+ バッチでリージョンを分割する閾値です。この値を増やすと、リージョンの分割が高速化されます。
+ デフォルト値：`10`
+ 最小値：`1`

### `region-max-size`

+ リージョンの最大サイズです。この値を超えると、リージョンが分割されます。
+ デフォルト値：`region-split-size / 2 * 3`
+ 単位：KiB|MiB|GiB

### `region-split-size`

+ 新しく分割されたリージョンのサイズです。この値は推定値です。
+ デフォルト値：`"96MiB"`
+ 単位：KiB|MiB|GiB

### `region-max-keys`

+ リージョン内の許容可能な最大キー数です。この値を超えると、リージョンが分割されます。
+ デフォルト値：`region-split-keys / 2 * 3`

### `region-split-keys`

+ 新しく分割されたリージョン内のキー数です。この値は推定値です。
+ デフォルト値：`960000`

### `consistency-check-method`

+ データの整合性チェックの方法を指定します。
+ MVCCデータの整合性チェックの場合、この値を `"mvcc"` に設定します。生データの整合性チェックの場合、この値を `"raw"` に設定します。
+ デフォルト値：`"mvcc"`
## コプロセッサーv2

### `coprocessor-plugin-directory`

+ コンパイルされたコプロセッサープラグインが配置されているディレクトリのパスです。このディレクトリ内のプラグインはTiKVによって自動的に読み込まれます。
+ この構成項目が設定されていない場合、コプロセッサープラグインは無効になります。
+ デフォルト値: `"./coprocessors"`

### `enable-region-bucket` <span class="version-mark">v6.1.0で新規追加</span>

+ リージョンをバケットと呼ばれる小さな範囲に分割するかどうかを決定します。これにより、スキャンの同時実行性を向上させるためのバケットとして使用されます。バケットの設計について詳しくは、[Dynamic size Region](https://github.com/tikv/rfcs/blob/master/text/0082-dynamic-size-region.md)を参照してください。
+ デフォルト値: false

> **警告:**
>
> - `enable-region-bucket`はTiDB v6.1.0で導入された実験的な機能です。本番環境で使用することはお勧めしません。
> - この構成は、`region-split-size`が`region-bucket-size`の2倍以上の場合にのみ意味があります。それ以外の場合、実際にはバケットが生成されません。
> - `region-split-size`を大きな値に調整すると、性能の低下やスケジューリングが遅くなるリスクがあります。

### `region-bucket-size` <span class="version-mark">v6.1.0で新規追加</span>

+ `enable-region-bucket`がtrueの場合のバケットのサイズです。
+ デフォルト値: v7.3.0から、デフォルト値は`96MiB`から`50MiB`に変更されました。

> **警告:**
>
> `region-bucket-size`はTiDB v6.1.0で導入された実験的な機能です。本番環境で使用することはお勧めしません。

## RocksDB

RocksDBに関連する構成項目

### `max-background-jobs`

+ RocksDB内のバックグラウンドスレッドの数です。RocksDBスレッドプールのサイズを変更する場合は、[Performance tuning for TiKV thread pools](/tune-tikv-thread-performance.md#performance-tuning-for-tikv-thread-pools)を参照してください。
+ デフォルト値:
    + CPUコア数が10の場合、デフォルト値は`9`です。
    + CPUコア数が8の場合、デフォルト値は`7`です。
    + CPUコア数が`N`の場合、デフォルト値は`max(2, min(N - 1, 9))`です。
+ 最小値: `2`

### `max-background-flushes`

+ バックグラウンドメンテーブルフラッシュジョブの最大同時実行数です。
+ デフォルト値:
    + CPUコア数が10の場合、デフォルト値は`3`です。
    + CPUコア数が8の場合、デフォルト値は`2`です。
    + CPUコア数が`N`の場合、デフォルト値は`[(max-background-jobs + 3) / 4]`です。
+ 最小値: `1`

### `max-sub-compactions`

+ RocksDBで同時に実行されるサブコンパクション操作の数です。
+ デフォルト値: `3`
+ 最小値: `1`

### `max-open-files`

+ RocksDBが開くことができるファイルの合計数です。
+ デフォルト値: `40960`
+ 最小値: `-1`

### `max-manifest-file-size`

+ RocksDBマニフェストファイルの最大サイズです。
+ デフォルト値: `"128MB"`
+ 最小値: `0`
+ 単位: B|KB|MB|GB

### `create-if-missing`

+ 自動的にDBスイッチを作成するかどうかを決定します。
+ デフォルト値: `true`

### `wal-recovery-mode`

+ WALリカバリーモードです。
+ オプション値:
    + `"tolerate-corrupted-tail-records"`: 全てのログで不完全なトレイリングデータを許容し、破棄します。
    + `"absolute-consistency"`: 破損したログが見つかった場合、リカバリーを中止します。
    + `"point-in-time"`: 破損したログが見つかるまで、順番にログを回復します。
    + `"skip-any-corrupted-records"`: ポストディザスターリカバリー。データは可能な限り回復され、破損したレコードはスキップされます。
+ デフォルト値: `"point-in-time"`

### `wal-dir`

+ WALファイルが保存されるディレクトリです。
+ デフォルト値: `"/tmp/tikv/store"`

### `wal-ttl-seconds`

+ アーカイブされたWALファイルの有効期間です。この値を超えると、システムはこれらのファイルを削除します。
+ デフォルト値: `0`
+ 最小値: `0`
+ 単位: 秒

### `wal-size-limit`

+ アーカイブされたWALファイルのサイズ制限です。この値を超えると、システムはこれらのファイルを削除します。
+ デフォルト値: `0`
+ 最小値: `0`
+ 単位: B|KB|MB|GB

### `max-total-wal-size`

+ `data-dir`内の`*.log`ファイルのサイズの合計である、RocksDB WALの最大サイズです。
+ デフォルト値:

    + `storage.engine="raft-kv"`の場合、デフォルト値は`"4GB"`です。
    + `storage.engine="partitioned-raft-kv"`の場合、デフォルト値は`1`です。

### `stats-dump-period`

+ 統計情報がログに出力される間隔です。
+ デフォルト値:

    + `storage.engine="raft-kv"`の場合、デフォルト値は`"10m"`です。
    + `storage.engine="partitioned-raft-kv"`の場合、デフォルト値は`"0"`です。

### `compaction-readahead-size`

+ RocksDBのコンパクション中にリードアヘッド機能を有効にし、リードアヘッドデータのサイズを指定します。機械ディスクを使用している場合、少なくとも2MBに設定することをお勧めします。
+ デフォルト値: `0`
+ 最小値: `0`
+ 単位: B|KB|MB|GB

### `writable-file-max-buffer-size`

+ WritableFileWriteで使用される最大バッファサイズです。
+ デフォルト値: `"1MB"`
+ 最小値: `0`
+ 単位: B|KB|MB|GB

### `use-direct-io-for-flush-and-compaction`

+ バックグラウンドフラッシュとコンパクションで`O_DIRECT`を読み書きの両方に使用するかどうかを決定します。このオプションのパフォーマンスへの影響: `O_DIRECT`を有効にすると、OSのバッファキャッシュの汚染をバイパスして防止しますが、その後のファイル読み取りでは、バッファキャッシュに再度内容を読み取る必要があります。
+ デフォルト値: `false`

### `rate-bytes-per-sec`

+ RocksDBのコンパクションレートリミッターで許可される最大レートです。
+ デフォルト値: `10GB`
+ 最小値: `0`
+ 単位: B|KB|MB|GB

### `rate-limiter-refill-period`

+ I/Oトークンが再充填される頻度を制御します。値が小さいほどI/Oバーストが減少しますが、CPUオーバーヘッドが増加します。
+ デフォルト値: `"100ms"`

### `rate-limiter-mode`

+ RocksDBのコンパクションレートリミッターモードです。
+ オプション値: `"read-only"`, `"write-only"`, `"all-io"`
+ デフォルト値: `"write-only"`

### `rate-limiter-auto-tuned` <span class="version-mark">v5.0で新規追加</span>

+ 最近のワークロードに基づいて、RocksDBのコンパクションレートリミッターの構成を自動的に最適化するかどうかを決定します。この構成が有効になっている場合、コンパクション保留バイトは通常よりもわずかに高くなります。
+ デフォルト値: `true`

### `enable-pipelined-write`

+ パイプラインライティングを有効にするかどうかを制御します。この構成が有効の場合、以前のパイプラインライティングが使用されます。この構成が無効の場合、新しいパイプラインコミットメカニズムが使用されます。
+ デフォルト値: `false`

### `bytes-per-sync`

+ ファイルが非同期で書き込まれる際にOSがディスクに増分的に同期する速度です。
+ デフォルト値: `"1MB"`
+ 最小値: `0`
+ 単位: B|KB|MB|GB

### `wal-bytes-per-sync`

+ WALファイルが書き込まれる際にOSが増分的にディスクに同期する速度です。
+ デフォルト値: `"512KB"`
+ 最小値: `0`
+ 単位: B|KB|MB|GB

### `info-log-max-size`

+ Infoログの最大サイズです。
+ デフォルト値: `"1GB"`
+ 最小値: `0`
+ 単位: B|KB|MB|GB

### `info-log-roll-time`

+ Infoログが切り捨てられる時間間隔です。値が`0s`の場合、ログは切り捨てられません。
+ デフォルト値: `"0s"`

### `info-log-keep-log-file-num`

+ 保持されるログファイルの最大数です。
+ デフォルト値: `10`
+ 最小値: `0`

### `info-log-dir`

+ ログが保存されるディレクトリです。
+ デフォルト値: `""`

### `info-log-level`

+ RocksDBのログレベルです。
+ デフォルト値: `"info"`

### `write-buffer-flush-oldest-first` <span class="version-mark">v6.6.0で新規追加</span>

> **警告:**
> この機能は実験的なものです。本番環境で使用しないことをお勧めします。この機能は予告なく変更または削除される可能性があります。バグを見つけた場合は、GitHubの[issue](https://github.com/pingcap/tidb/issues)で報告できます。

+ 指定された値以下になった場合に、現在のRocksDBの`memtable`のメモリ使用量がしきい値に達したときに使用されるフラッシュストラテジーを指定します。
+ デフォルト値： `false`
+ 値オプション：

    + `false`: データボリュームが最大の`memtable`はSSTファイルにフラッシュされます。
    + `true`: 最初の`memtable`はSSTファイルにフラッシュされます。この戦略は、明確な冷データとホットデータが存在するシナリオに適しています。

### `write-buffer-limit` <span class="version-mark">v6.6.0で新規追加</span>

> **警告:**
>
> この機能は実験的なものです。本番環境で使用しないことをお勧めします。この機能は予告なく変更または削除される可能性があります。バグを見つけた場合は、GitHubの[issue](https://github.com/pingcap/tidb/issues)で報告できます。

+ 単一のTiKV内のすべてのRocksDBインスタンスの`memtable`の合計メモリ制限を指定します。 `0` は制限なしを意味します。
+ デフォルト値：

    + `storage.engine="raft-kv"`の場合、デフォルト値は `0` で、制限なしを意味します。
    + `storage.engine="partitioned-raft-kv"`の場合、デフォルト値は総システムメモリサイズの20%です。

+ 単位: KiB|MiB|GiB

## rocksdb.titan

Titanに関連する構成項目。

### `enabled`

+ Titanの有効または無効を設定します
+ デフォルト値: `false`

### `dirname`

+ Titan Blobファイルが保存されるディレクトリ
+ デフォルト値: `"titandb"`

### `disable-gc`

+ TitanがBlobファイルに実行するGarbage Collection（GC）を無効にするかどうかを決定します
+ デフォルト値: `false`

### `max-background-gc`

+ TitanのGCスレッドの最大数
+ デフォルト値: `4`
+ 最小値: `1`

## rocksdb.defaultcf | rocksdb.writecf | rocksdb.lockcf

`rocksdb.defaultcf`、`rocksdb.writecf`、`rocksdb.lockcf`に関連する構成項目。

### `block-size`

+ RocksDBブロックのデフォルトサイズ
+ `defaultcf`と`writecf`のデフォルト値: `"32KB"`
+ `lockcf`のデフォルト値: `"16KB"`
+ 最小値: `"1KB"`
+ 単位: KB|MB|GB

### `block-cache-size`

> **警告:**
>
> v6.6.0から、この構成は非推奨です。

+ RocksDBブロックのキャッシュサイズ
+ `defaultcf`のデフォルト値: `合計マシンメモリ * 25％`
+ `writecf`のデフォルト値: `合計マシンメモリ * 15％`
+ `lockcf`のデフォルト値: `合計マシンメモリ * 2％`
+ 最小値: `0`
+ 単位: KB|MB|GB

### `disable-block-cache`

+ ブロックキャッシュの有効または無効を設定します
+ デフォルト値: `false`

### `cache-index-and-filter-blocks`

+ インデックスとフィルタのキャッシュの有効または無効を設定します
+ デフォルト値: `true`

### `pin-l0-filter-and-index-blocks`

+ レベル0のSSTファイルのインデックスとフィルタブロックをメモリにピン留めするかどうかを決定します
+ デフォルト値: `true`

### `use-bloom-filter`

+ ブルームフィルタの有効または無効を設定します
+ デフォルト値: `true`

### `optimize-filters-for-hits`

+ フィルタのヒット率を最適化するかどうかを決定します
+ `defaultcf`のデフォルト値: `true`
+ `writecf`および`lockcf`のデフォルト値: `false`

### `optimize-filters-for-memory` <span class="version-mark">v7.2.0で新規追加</span>

+ メモリ内部の断片化を最小限に抑えるBloom/Ribbonフィルタを生成するかどうかを決定します
+ この構成項目は、[`format-version`](#format-version-new-in-v620)が5以上の場合にのみ有効です。
+ デフォルト値: `false`

### `whole-key-filtering`

+ 全体のキーをブルームフィルタに入れるかどうかを決定します
+ `defaultcf`および`lockcf`のデフォルト値: `true`
+ `writecf`のデフォルト値: `false`

### `bloom-filter-bits-per-key`

+ 各キーに予約されるブルームフィルタの長さ
+ デフォルト値: `10`
+ 単位: バイト

### `block-based-bloom-filter`

+ 各ブロックがブルームフィルタを作成するかどうかを決定します
+ デフォルト値: `false`

### `ribbon-filter-above-level` <span class="version-mark">v7.2.0で新規追加</span>

+ この値以上のレベルにはRibbonフィルタを使用し、この値未満のレベルには非ブロックベースのブルームフィルタを使用するかを決定します。この設定項目が設定されている場合、[`block-based-bloom-filter`](#block-based-bloom-filter)は無視されます。
+ この構成項目は、[`format-version`](#format-version-new-in-v620)が5以上の場合にのみ有効です。
+ デフォルト値: `false`

### `read-amp-bytes-per-bit`

+ リードの増幅の統計を有効または無効にするかを決定します
+ オプションの値: `0` (無効), > `0` (有効)
+ デフォルト値: `0`

### `compression-per-level`

+ 各レベルのデフォルトの圧縮アルゴリズム
+ `defaultcf`のデフォルト値: `["no", "no", "lz4", "lz4", "lz4", "zstd", "zstd"]`
+ `writecf`のデフォルト値: `["no", "no", "lz4", "lz4", "lz4", "zstd", "zstd"]`
+ `lockcf`のデフォルト値: `["no", "no", "no", "no", "no", "no", "no"]`

### `bottommost-level-compression`

+ 底層の層の圧縮アルゴリズムを設定します。この構成項目は`compression-per-level`の設定を上書きします。
+ LSMツリーにデータが書き込まれると、RocksDBは`compression-per-level`配列で指定された最後の圧縮アルゴリズムを直接採用しません。`bottommost-level-compression`は、底層が最初から最適な圧縮アルゴリズムを使用するようにします。
+ 底層の圧縮アルゴリズムを設定したくない場合は、この構成項目の値を `disable` に設定してください。
+ デフォルト値: `"zstd"`

### `write-buffer-size`

+ Memtableのサイズ
+ `defaultcf`および`writecf`のデフォルト値: `"128MB"`
+ `lockcf`のデフォルト値:
    + `storage.engine="raft-kv"`の場合、デフォルト値は`"32MB"`です。
    + `storage.engine="partitioned-raft-kv"`の場合、デフォルト値は`"4MB"`です。
+ 最小値: `0`
+ 単位: KB|MB|GB

### `max-write-buffer-number`

+ Memtableの最大数。`storage.flow-control.enable`が`true`に設定されている場合、`storage.flow-control.memtables-threshold`がこの構成項目を上書きします。
+ デフォルト値: `5`
+ 最小値: `0`

### `min-write-buffer-number-to-merge`

+ フラッシュをトリガーするために必要なMemtableの最小数
+ デフォルト値: `1`
+ 最小値: `0`

### `max-bytes-for-level-base`

+ ベースレベル（レベル-1）の最大バイト数。一般的には、メモリテーブルのサイズの4倍が設定されます。`max-bytes-for-level-base`の制限値にレベル1のデータサイズが達すると、レベル1のSSTファイルとレベル2のSSTファイルとの重複するSSTファイルがコンパクト化されます。
+ `defaultcf`および`writecf`のデフォルト値: `"512MB"`
+ `lockcf`のデフォルト値: `"128MB"`
+ 最小値: `0`
+ 単位: KB|MB|GB
+ `max-bytes-for-level-base`の値は、不必要なコンパクションを減らすためにL0のデータボリュームにほぼ等しい値に設定することをお勧めします。例えば、圧縮方法が "no:no:lz4:lz4:lz4:lz4:lz4" の場合、 `max-bytes-for-level-base` の値は `write-buffer-size * 4` に設定することをお勧めします。なぜなら、L0とL1の両方にコンパクションが採用される場合、RocksDBログを分析してメモリテーブルから圧縮されたSSTファイルのサイズを理解する必要があるからです。例えば、ファイルサイズが32MBの場合、 `max-bytes-for-level-base` の値を128MB（`32MB * 4`）に設定することをお勧めします。

### `target-file-size-base`

+ ベースレベルのターゲットファイルのサイズ。`enable-compaction-guard`値が`true`の場合、この値は`compaction-guard-max-output-file-size`によって上書きされます。
+ デフォルト値: `"8MB"`
+ 最小値: `0`
+ 単位: KB|MB|GB

### `level0-file-num-compaction-trigger`

+ 最適に圧縮するBlobファイル内の値の最小値。指定されたサイズよりも小さい値は、LSMツリーに保存されます。
+ デフォルト値: "1KB"
+ 最小値: `0`
+ 単位: KB|MB|GB

### `blob-file-compression`

+ Blobファイルで使用する圧縮アルゴリズム
+ オプションの値: `"no"`, `"snappy"`, `"zlib"`, `"bzip2"`, `"lz4"`, `"lz4hc"`, `"zstd"`
+ デフォルト値: "lz4"

> **注意:**
>
> Snappy圧縮ファイルは[公式のSnappy形式](https://github.com/google/snappy)である必要があります。他のバリエーションのSnappy圧縮はサポートされていません。

### `blob-cache-size`

+ Blobファイルのキャッシュサイズ
+ デフォルト値: "0GB"
+ 最小値: `0`
+ 単位: KB|MB|GB

### `min-gc-batch-size`

+ 1回のGCに必要なBlobファイルの合計サイズの最小値
+ デフォルト値: "16MB"
+ 最小値: `0`
+ 単位: KB|MB|GB

### `max-gc-batch-size`

+ 1回のGCで処理が許可されているBlobファイルの合計サイズの最大値
+ デフォルト値: "64MB"
+ 最小値: `0`
+ 単位: KB|MB|GB

### `discardable-ratio`

+ Blobファイルの無効な値の割合がこの比率を超えると、GCが開始されます。Blobファイルは、その中の無効な値の割合がこの比率を超えた場合にのみ選択されます。
+ デフォルト値: `0.5`
+ 最小値: `0`
+ 最大値: `1`

### `sample-ratio`

+ Blobファイルからサンプリングする際の（Blobファイルから読み取られたデータ/全体のBlobファイル）の割合
+ デフォルト値: `0.1`
+ 最小値: `0`
+ 最大値: `1`

### `merge-small-file-threshold`

+ Blobファイルのサイズがこの値より小さい場合、それでもGCの対象となる可能性があります。この場合、`discardable-ratio`は無視されます。
+ デフォルト値: "8MB"
+ 最小値: `0`
+ 単位: KB|MB|GB

### `blob-run-mode`

+ Titanの実行モードを指定します。
+ オプションの値:
    + `normal`: 値のサイズが`min-blob-size`を超えた場合、データをBlobファイルに書き込みます。
+ `read_only`: blobファイルに新しいデータを書き込むことを拒否しますが、blobファイルから元のデータは読み取ります。
    + `fallback`: blobファイルにデータをLSMに書き戻します。
+ デフォルト値: `normal`

### `level-merge`

+ 読み取りパフォーマンスを最適化するかどうかを決定します。`level-merge`を有効にすると、書き込みの増幅が増えます。
+ デフォルト値: `false`

## raftdb

`raftdb`に関連する設定項目

### `max-background-jobs`

+ RocksDB内のバックグラウンドスレッドの数です。RocksDBスレッドプールのサイズを変更する場合は、[Performance tuning for TiKV thread pools](/tune-tikv-thread-performance.md#performance-tuning-for-tikv-thread-pools)を参照してください。
+ デフォルト値: `4`
+ 最小値: `2`

### `max-sub-compactions`

+ RocksDBで実行される並行サブコンパクション操作の数です
+ デフォルト値: `2`
+ 最小値: `1`

### `max-open-files`

+ RocksDBが開くことができるファイルの合計数です
+ デフォルト値: `40960`
+ 最小値: `-1`

### `max-manifest-file-size`

+ RocksDBマニフェストファイルの最大サイズです
+ デフォルト値: `"20MB"`
+ 最小値: `0`
+ ユニット: B|KB|MB|GB

### `create-if-missing`

+ 値が`true`の場合、データベースが不足している場合はデータベースを作成します
+ デフォルト値: `true`

### `stats-dump-period`

+ 統計情報がログに出力される間隔です
+ デフォルト値: `10m`

### `wal-dir`

+ Raft RocksDB WALファイルが格納されるディレクトリです。これはWAL用の絶対ディレクトリパスです。この設定項目を[`rocksdb.wal-dir`](#wal-dir)と同じ値に設定しないでください。
+ この設定項目が設定されていない場合、ログファイルはデータと同じディレクトリに保存されます。
+ マシンに2つのディスクがある場合、RocksDBデータとWALログを異なるディスクに保存することでパフォーマンスを向上させることができます。
+ デフォルト値: `""`

### `wal-ttl-seconds`

+ アーカイブされたWALファイルを保持する期間を指定します。値が超過すると、システムはこれらのファイルを削除します。
+ デフォルト値: `0`
+ 最小値: `0`
+ ユニット: 秒

### `wal-size-limit`

+ アーカイブされたWALファイルのサイズ制限です。値が超過すると、システムはこれらのファイルを削除します。
+ デフォルト値: `0`
+ 最小値: `0`
+ ユニット: B|KB|MB|GB

### `max-total-wal-size`

+ 総合での最大RocksDB WALサイズ
+ デフォルト値: `"4GB"`
    + `storage.engine="raft-kv"`の場合、デフォルト値は`"4GB"`です。
    + `storage.engine="partitioned-raft-kv"`の場合、デフォルト値は`1`です。

### `compaction-readahead-size`

+ RocksDBコンパクション中にリードアヘッド機能を有効にするかどうか、またリードアヘッドデータのサイズを指定します。
+ 機械ディスクを使用している場合、少なくとも値を`2MB`に設定することをお勧めします。
+ デフォルト値: `0`
+ 最小値: `0`
+ ユニット: B|KB|MB|GB

### `writable-file-max-buffer-size`

+ WritableFileWriteで使用される最大バッファサイズ
+ デフォルト値: `"1MB"`
+ 最小値: `0`
+ ユニット: B|KB|MB|GB

### `use-direct-io-for-flush-and-compaction`

+ バックグラウンドフラッシュとコンパクションで読み書きの両方に`O_DIRECT`を使用するかどうかを決定します。このオプションのパフォーマンスの影響: `O_DIRECT`を有効にすると、OSバッファキャッシュの汚染をバイパスして防ぐため、その後のファイル読み込みではバッファキャッシュへの内容の再読み込みが必要になります。
+ デフォルト値: `false`

### `enable-pipelined-write`

+ パイプライン化された書き込みを有効にするかどうかを制御します。この設定が有効な場合、以前のパイプライン書き込みが使用されます。この設定が無効になっている場合、新しいパイプライン化されたコミットメカニズムが使用されます。
+ デフォルト値: `true`

### `allow-concurrent-memtable-write`

+ 同時メンテーブル書き込みを有効にするかどうかを制御します。
+ デフォルト値: `true`

### `bytes-per-sync`

+ ファイルが非同期で書き込まれている間にOSがファイルをディスクに増分的に同期する速度です。
+ デフォルト値: `"1MB"`
+ 最小値: `0`
+ ユニット: B|KB|MB|GB

### `wal-bytes-per-sync`

+ WALファイルが書き込まれている場合に、OSがファイルをディスクに増分的に同期する速度です
+ デフォルト値: `"512KB"`
+ 最小値: `0`
+ ユニット: B|KB|MB|GB

### `info-log-max-size`

+ Infoログの最大サイズ
+ デフォルト値: `"1GB"`
+ 最小値: `0`
+ ユニット: B|KB|MB|GB

### `info-log-roll-time`

+ Infoログが切り捨てられる間隔です。値が`0s`の場合、ログは切り捨てられません。
+ デフォルト値: `"0s"` (これはログが切り捨てられないことを意味します)

### `info-log-keep-log-file-num`

+ RaftDBで保持されるInfoログファイルの最大数
+ デフォルト値: `10`
+ 最小値: `0`

### `info-log-dir`

+ Infoログが保存されるディレクトリ
+ デフォルト値: `""`

### `info-log-level`

+ RaftDBのログレベル
+ デフォルト値: `"info"`

## raft-engine

Raft Engineに関連する設定項目。

> **注意:**
>
> - Raft Engineを初めて有効にすると、TiKVはRocksDBからRaft Engineにデータを転送します。そのため、TiKVの起動には追加で数十秒かかることがあります。
> - TiDB v5.4.0のRaft Engineのデータ形式は以前のTiDBバージョンと互換性がありません。したがって、TiDBクラスタをv5.4.0から以前のバージョンにダウングレードする必要がある場合は、ダウングレード**前に**、`enable`を`false`に設定してRaft Engineを無効化し、構成が有効になるようにTiKVを再起動してください。

### `enable`

+ Raft Engineを使用してRaftログを保存するかどうかを決定します。有効にする場合、`raftdb`の設定は無視されます。
+ デフォルト値: `true`

### `dir`

+ Raftログファイルが保存されるディレクトリです。ディレクトリが存在しない場合、TiKVが起動すると作成されます。
+ この設定項目が設定されていない場合、`{data-dir}/raft-engine`が使用されます。
+ マシンに複数のディスクがある場合、Raft Engineのデータを別のディスクに保存することをお勧めします。
+ デフォルト値: `""`

### `batch-compression-threshold`

+ ログバッチの閾値サイズを指定します。この設定を超えるログバッチは圧縮されます。この設定項目を`0`に設定すると、圧縮が無効になります。
+ デフォルト値: `"8KB"`

### `bytes-per-sync`

+ バッファされた書き込みの最大累積サイズを指定します。この設定値が超過すると、バッファされた書き込みがディスクにフラッシュされます。
+ この設定項目を`0`に設定すると、インクリメンタル同期が無効になります。
+ デフォルト値: `"4MB"`

### `target-file-size`

+ ログファイルの最大サイズを指定します。ログファイルがこの値よりも大きい場合、回転されます。
+ デフォルト値: `"128MB"`

### `purge-threshold`

+ メインログキューの閾値サイズを指定します。この設定値が超過すると、メインログキューが削除されます。
+ この設定はRaft Engineのディスクスペース使用量を調整するために使用できます。
+ デフォルト値: `"10GB"`

### `recovery-mode`

+ 回復中のファイルの破損に対処する方法を決定します。
+ 値のオプション: `"absolute-consistency"`、`"tolerate-tail-corruption"`、`"tolerate-any-corruption"`
+ デフォルト値: `"tolerate-tail-corruption"`

### `recovery-read-block-size`

+ 回復中のログファイルを読み取るための最小I/Oサイズ。
+ デフォルト値: `"16KB"`
+ 最小値: `"512B"`

### `recovery-threads`

+ スキャンおよびログファイルの回復に使用されるスレッド数。
+ デフォルト値: `4`
+ 最小値: `1`

### `memory-limit`

+ Raft Engineのメモリ使用量の制限を指定します。
+ この設定値が設定されていない場合、利用可能なシステムメモリの15%が使用されます。
+ デフォルト値: `合計マシンメモリ * 15%`

### `format-version` <span class="version-mark">v6.3.0で新規</span>

> **注意:**
>
> `format-version`が`2`に設定された後、v6.3.0から以前のTiKVクラスタにダウングレードする必要がある場合は、ダウングレード**前に**次の手順を実行してください:
>
> 1. `enable`を`false`に設定してRaft Engineを無効にし、構成が有効になるようにTiKVを再起動してください。
> 2. `format-version`を`1`に設定してください。
> 3. `enable`を`true`に設定し、TiKVを再起動して構成が有効になるようにします。

+ Raft Engineのログファイルのバージョンを指定します。
+ 値オプション:
    + `1`: v6.3.0よりも前のTiKVのデフォルトログファイルバージョン。TiKV >= v6.1.0で読み取ることができます。
    + `2`: ログのリサイクルをサポート。TiKV >= v6.3.0で読み取ることができます。
+ デフォルト値:
    + `storage.engine="raft-kv"`の場合、デフォルト値は`2`です。
    + `storage.engine="partitioned-raft-kv"`の場合、デフォルト値は`5`です。

### `enable-log-recycle` <span class="version-mark">v6.3.0で新規</span>

> **注:**
>
> この構成項目は[`format-version`](#format-version-new-in-v630)が2以上の場合にのみ利用可能です。

+ Raft Engineで古いログファイルを再利用するかどうかを決定します。有効にすると、論理的に削除されたログファイルは再利用するために保持されます。これにより、書き込みワークロードの長い尾部遅延が削減されます。
+ デフォルト値: `true`

### `prefill-for-recycle` <span class="version-mark">v7.0.0で新規</span>

> **注:**
>
> この構成項目は[`enable-log-recycle`](#enable-log-recycle-new-in-v630)が`true`に設定されている場合にのみ有効です。

+ Raft Engineでログの再利用のために空のログファイルを生成するかどうかを決定します。有効にすると、Raft Engineは初期化中にログの再利用のために自動で一括の空のログファイルを用意し、初期化直後からログの再利用がすぐに有効になります。
+ デフォルト値: `false`

## security

セキュリティに関連する構成項目。

### `ca-path`

+ CAファイルのパス
+ デフォルト値: `""`

### `cert-path`

+ X.509証明書を含むPrivacy Enhanced Mail (PEM)ファイルのパス
+ デフォルト値: `""`

### `key-path`

+ X.509キーを含むPEMファイルのパス
+ デフォルト値: `""`

### `cert-allowed-cn`

+ クライアントが提示する証明書のX.509共通名のリスト。リクエストは、提示された共通名がリストのエントリのいずれかと完全に一致した場合のみ許可されます。
+ デフォルト値: `[]`。これは、デフォルトではクライアント証明書のCNチェックが無効であることを意味します。

### `redact-info-log` <span class="version-mark">v4.0.8で新規</span>

+ この構成項目はログのマスキングを有効または無効にします。構成値が`true`に設定されている場合、ログ内のすべてのユーザーデータは`?`に置き換えられます。
+ デフォルト値: `false`

## security.encryption

[データ保管時の暗号化](/encryption-at-rest.md) (TDE)に関連する構成項目。 

### `data-encryption-method`

+ データファイルの暗号化方法
+ 値オプション: "plaintext", "aes128-ctr", "aes192-ctr", "aes256-ctr"、および"sm4-ctr" (v6.3.0よりサポート)
+ "plaintext"以外の値は暗号化が有効であることを意味し、その場合はマスターキーを指定する必要があります。
+ デフォルト値: `"plaintext"`

### `data-key-rotation-period`

+ TiKVがデータ暗号化キーを回転する頻度を指定します。
+ デフォルト値: `7d`

### `enable-file-dictionary-log`

+ TiKVが暗号化メタデータを管理する際のI/Oとmutexの競合を減らす最適化を有効にします。
+ この構成パラメータを有効にする場合の潜在的な互換性の問題を避けるためには、デフォルトでこの構成パラメータを有効にすることができます。詳細については、[暗号化時の互換性 - TiKVバージョン間の互換性](/encryption-at-rest.md#compatibility-between-tikv-versions)を参照してください。
+ デフォルト値: `true`

### `master-key`

+ 暗号化が有効な場合、マスターキーを指定します。マスターキーの構成方法については、[暗号化時の互換性](/encryption-at-rest.md#configure-encryption)を参照してください。

### `previous-master-key`

+ 新しいマスターキーの回転時に古いマスターキーを指定します。構成形式は`master-key`と同じです。マスターキーの構成方法については、[暗号化時の互換性](/encryption-at-rest.md#configure-encryption)を参照してください。

## import

TiDB LightningのインポートおよびBRリストアに関連する構成項目。

### `num-threads`

+ RPCリクエストを処理するスレッド数
+ デフォルト値: `8`
+ 最小値: `1`

### `stream-channel-window`

+ Streamチャンネルのウィンドウサイズ。チャンネルがいっぱいの場合、ストリームはブロックされます。
+ デフォルト値: `128`

### `memory-use-ratio` <span class="version-mark">v6.5.0で新規</span>

+ v6.5.0から、PITRはメモリ中のバックアップログファイルに直接アクセスしてデータをリストアできます。この構成項目は、PITRがTiKVの総メモリのうちで利用可能なメモリの割合を指定します。
+ 値の範囲: [0.0, 0.5]
+ デフォルト値: `0.3`。これは、システムメモリの30%がPITRに利用可能であることを意味します。値が`0.0`の場合、PITRはローカルディレクトリにログファイルをダウンロードして実行されます。

> **注:**
>
> v6.5.0より前のバージョンでは、瞬時リストア（PITR）はローカルディレクトリにバックアップファイルをダウンロードしてデータをリストアするだけです。

## gc

ガベージコレクションに関連する構成項目。

### `batch-keys`

+ 一括でガベージコレクションを行うキーの数
+ デフォルト値: `512`

### `max-write-bytes-per-sec`

+ ガベージコレクションワーカーがRocksDBに1秒間に書き込める最大バイト数。
+ 値を`0`に設定すると制限がありません。
+ デフォルト値: `"0"`

### `enable-compaction-filter` <span class="version-mark">v5.0で新規</span>

+ コンパクションフィルタでGCを有効にするかどうかを制御します
+ デフォルト値: `true`

### `ratio-threshold`

+ GCをトリガするガベージ比率の閾値
+ デフォルト値: `1.1`

## backup

BRバックアップに関連する構成項目。

### `num-threads`

+ バックアップを処理するワーカースレッド数
+ デフォルト値: `MIN(CPU * 0.5, 8)`
+ 値の範囲: `[1, CPU]`
+ 最小値: `1`

### `batch-size`

+ 1バッチでバックアップするデータ範囲の数
+ デフォルト値: `8`

### `sst-max-size`

+ バックアップSSTファイルサイズの閾値。TiKVリージョン内のバックアップファイルサイズがこの閾値を超えると、ファイルは複数に分割され、TiKVリージョンが複数のリージョン範囲に分割された複数のファイルにバックアップされます。分割されたリージョンの各ファイルは`sst-max-size`と同じサイズ（または若干大きい）になります。
+ たとえば、リージョン[a,e)のバックアップファイルサイズが`sst-max-size`を超えると、ファイルはリージョン[a,b)、[b,c)、[c,d)、および[d,e)にバックアップされ、分割されたリージョンの各ファイルは`sst-max-size`と同じサイズ（または若干大きい）になります。
+ デフォルト値: `"144MB"`

### `enable-auto-tune` <span class="version-mark">v5.4.0で新規</span>

+ クラスタのリソース使用率が高い場合にクラスタへの影響を軽減するために、バックアップタスクで使用されるリソースを制限するかどうかを制御します。詳細については、[BR自動チューニング](/br/br-auto-tune.md)を参照してください。
+ デフォルト値: `true`

### `s3-multi-part-size` <span class="version-mark">v5.3.2で新規</span>

> **注:**
>
> この構成は、S3のレート制限によって発生するバックアップの失敗に対処するために導入されました。この問題は[バックアップデータストレージ構造の改善](/br/br-snapshot-architecture.md#structure-of-backup-files)により修正されています。そのため、この構成はv6.1.1から非推奨とされ、それ以上は推奨されません。

+ バックアップ時にS3へのマルチパートアップロード時に使用されるパートサイズ。この構成項目の値を調整して、S3へ送信されるリクエストの数を制御できます。
+ データがS3にバックアップされ、TiKVリージョンのバックアップファイルサイズがこの構成項目の値を超える場合は、[マルチパートアップロード](https://docs.aws.amazon.com/AmazonS3/latest/API/API_UploadPart.html)が自動的に有効になります。圧縮率に基づくと、96MiBのリージョンによって生成されるバックアップファイルは約10MiBから30MiBになります。
+ デフォルト値: 5MiB

## backup.hadoop

### `home`

+ HDFSシェルコマンドの場所を指定し、TiKVがシェルコマンドを見つけることを可能にします。この構成項目は環境変数`$HADOOP_HOME`と同じ効果があります。
+ デフォルト値: `""`

### `linux-user`

+ TiKVがHDFSシェルコマンドを実行するためのLinuxユーザーを指定します。
+ この構成項目が設定されていない場合、TiKVは現在のLinuxユーザーを使用します。
+ デフォルト値: `""`

## log-backup

ログバックアップに関連する構成項目。
### `enable` <span class="version-mark">v6.2.0 で新規追加</span>

+ ログバックアップを有効にするかどうかを決定します。
+ デフォルト値: `true`

### `file-size-limit` <span class="version-mark">v6.2.0 で新規追加</span>

+ 保存されるバックアップログデータのサイズ制限です。
+ デフォルト値: 256MiB
+ 注: 一般的に、`file-size-limit` の値は外部ストレージに表示されるバックアップファイルサイズよりも大きいです。これは、バックアップファイルが外部ストレージにアップロードされる前に圧縮されるためです。

### `initial-scan-pending-memory-quota` <span class="version-mark">v6.2.0 で新規追加</span>

+ ログバックアップ中に増分スキャンデータを格納するためのキャッシュのクォータです。
+ デフォルト値: `min(合計マシンメモリ * 10%, 512 MB)`

### `initial-scan-rate-limit` <span class="version-mark">v6.2.0 で新規追加</span>

+ ログバックアップ中の増分データスキャンにおけるスループットのレート制限です。
+ デフォルト値: 60（デフォルトでレート制限は60 MB/sを示します）

### `max-flush-interval` <span class="version-mark">v6.2.0 で新規追加</span>

+ バックアップデータを外部ストレージに書き込むための最大間隔です。
+ デフォルト値: 3分

### `num-threads` <span class="version-mark">v6.2.0 で新規追加</span>

+ ログバックアップで使用されるスレッドの数です。
+ デフォルト値: CPU * 0.5
+ 値の範囲: [2, 12]

### `temp-path` <span class="version-mark">v6.2.0 で新規追加</span>

+ ログファイルが外部ストレージにフラッシュされる前に書き込まれる一時的なパスです。
+ デフォルト値: `${deploy-dir}/data/log-backup-temp`

## cdc

TiCDC に関連する構成項目。

### `min-ts-interval`

+ 解決済み TS が計算および転送される間隔です。
+ デフォルト値: `"200ms"`

### `old-value-cache-memory-quota`

+ TiCDC の古い値が使用するメモリの上限です。
+ デフォルト値: `512MB`

### `sink-memory-quota`

+ TiCDC のデータ変更イベントが使用するメモリの上限です。
+ デフォルト値: `512MB`

### `incremental-scan-speed-limit`

+ 過去データを増分スキャンする最大速度です。
+ デフォルト値: `"128MB"`（秒あたり128MBを意味します）

### `incremental-scan-threads`

+ 過去データを増分スキャンするタスクに使用されるスレッドの数です。
+ デフォルト値: `4`（4つのスレッドを意味します）

### `incremental-scan-concurrency`

+ 過去データを増分スキャンするタスクの最大同時実行数です。
+ デフォルト値: `6`（最大で6つのタスクが同時に実行されることを意味します）
+ 注: 「incremental-scan-concurrency」の値は「incremental-scan-threads」より大きいか等しい必要があります。それ以外の場合、TiKV は起動時にエラーを報告します。

## resolved-ts

ステールな読み取りリクエストを処理するために Resolved TS を維持するかどうかを決定します。

### `enable`

+ すべてのリージョンのために解決済み TS を維持するかどうかを決定します。
+ デフォルト値: `true`

### `advance-ts-interval`

+ 解決済み TS が計算および転送される間隔です。
+ デフォルト値: `"20s"`

### `scan-lock-pool-size`

+ Resolved TS を初期化する際に TiKV が MVCC（multi-version concurrency control）ロックデータをスキャンするために使用するスレッド数です。
+ デフォルト値: `2`（2つのスレッドを意味します）

## pessimistic-txn

悲観的トランザクションの使用については、[TiDB 悲観的トランザクションモード](/pessimistic-transaction.md) を参照してください。

### `wait-for-lock-timeout`

- TiKV における悲観的トランザクションが他のトランザクションがロックを解放するのを待機する最長時間です。時間切れになると、TiDB にエラーが返され、TiDB はロックを追加し直します。ロックの待ち時間切れは `innodb_lock_wait_timeout` によって設定されます。
- デフォルト値: `"1s"`
- 最小値: `"1ms"`

### `wake-up-delay-duration`

- 悲観的トランザクションがロックを解放すると、ロックを待っているトランザクションの中で最小の `start_ts` を持つトランザクションのみが起こされます。他のトランザクションは `wake-up-delay-duration` 後に起こされます。
- デフォルト値: `"20ms"`

### `pipelined`

- この構成項目は、悲観的ロックの追加をパイプライン処理にする機能を有効にします。この機能を有効にすると、TiKV はデータをロックできることを検出した後、TiDB に直ちに後続リクエストを実行するよう通知し、悲観的ロックを非同期で書き込むことができます。これにより、大部分の待機時間が削減され、悲観的トランザクションのパフォーマンスが大幅に向上します。ただし、悲観的ロックの非同期書き込みが失敗する可能性が低いですがまだあります。これにより、悲観的トランザクションのコミットが失敗する可能性があります。
- デフォルト値: `true`

### `in-memory` <span class="version-mark">v6.0.0 で新規追加</span>

+ インメモリ悲観的ロック機能を有効にします。この機能を有効にすると、悲観的トランザクションはロックをディスクに書き込んだり、他のレプリカにロックを複製する代わりに、ロックをメモリに保存しようとします。これにより、悲観的トランザクションのパフォーマンスが向上します。ただし、悲観的ロックが失われ、悲観的トランザクションのコミットが失敗する可能性がまだ低いです。
+ デフォルト値: `true`
+ `pipelined` の値が `true` の場合のみ、`in-memory` が有効になります。

## quota

クォータリミッターに関連する構成項目。

### `max-delay-duration` <span class="version-mark">v6.0.0 で新規追加</span>

+ 単一の読み取りまたは書き込みリクエストが前景で処理される前に待機することが強制される最長時間です。
+ デフォルト値: `500ms`
+ 推奨設定: ほとんどの場合、デフォルト値を使用することをお勧めします。インスタンスでメモリ不足（OOM）や激しいパフォーマンスの揺れが発生した場合は、リクエストの待機時間を1秒未満にするために値を1Sに設定できます。

### 前景クォータリミッター

前景クォータリミッターに関連する構成項目。

TiKV が展開されているマシンのリソースに制限がある場合（たとえば、4v CPU と16 G メモリしかない場合）、TiKV の前景は前景が多くの読み取りおよび書き込みリクエストを処理しすぎて、バックグラウンドがリクエストの処理を支援するために使用する CPU リソースが占有される可能性があります。これは TiKV のパフォーマンスの安定性に影響を与えます。このような状況を避けるために、前景の CPU リソースの使用を制限するために前景に関連するクォータに関連する構成項目を使用できます。クォータリミッターがリクエストをトリガーすると、TiKV は CPU リソースを解放するためにしばらく待たせる強制されます。正確な待ち時間はリクエストの数に依存し、最大待ち時間は [`max-delay-duration`](#max-delay-duration-new-in-v600) の値以上にはなりません。

#### `foreground-cpu-time` <span class="version-mark">v6.0.0 で新規追加</span>

+ TiKV 前景が読み取りおよび書き込みリクエストを処理するために使用する CPU リソースの制限です。
+ デフォルト値: `0`（つまり制限なし）
+ 単位: millicpu（たとえば、`1500` は前景リクエストが 1.5v CPU を消費することを意味します）
+ 推奨設定: 4コア以上のインスタンスの場合、デフォルト値 `0` を使用します。4コアのインスタンスの場合、`1000` から `1500` の範囲に設定するとバランスがとれます。2コアのインスタンスの場合、`1200` より小さい値を保持します。

#### `foreground-write-bandwidth` <span class="version-mark">v6.0.0 で新規追加</span>

+ トランザクションがデータを書き込む際のバンド幅のソフト制限です。
+ デフォルト値: `0KB`（つまり制限なし）
+ 推奨設定: `foreground-cpu-time` の設定が書き込みバンド幅を制限するために不十分な場合を除き、ほとんどの場合はデフォルト値 `0` を使用します。その種の例外の場合、4コア以下のインスタンスで、`50MB` より小さい値を設定することをお勧めします。

#### `foreground-read-bandwidth` <span class="version-mark">v6.0.0 で新規追加</span>

+ トランザクションおよび Coprocessor がデータを読み取る際のバンド幅のソフト制限です。
+ デフォルト値: `0KB`（つまり制限なし）
+ 推奨設定: `foreground-cpu-time` の設定が読み取りバンド幅を制限するために不十分な場合を除き、ほとんどの場合はデフォルト値 `0` を使用します。その種の例外の場合、4コア以下のインスタンスで、`20MB` より小さい値を設定することをお勧めします。

### バックグラウンドクォータリミッター

バックグラウンドクォータリミッターに関連する構成項目。
```markdown
      + {T}
      + {T}
    + {T}
  + {T}
```
```markdown
      + {R}
      + {R}
    + {R}
  + {R}
```
+ CPU使用率のしきい値を制御し、リージョンがホットスポットとして識別される条件を設定します。
+ デフォルト値:
    + `region-split-size`が4GB未満の場合、`0.25`。
    + `region-split-size`が4GB以上の場合、`0.75`。

## memory <span class="version-mark">v7.5.0で新規追加</span>

### `enable-heap-profiling` <span class="version-mark">v7.5.0で新規追加</span>

+ TiKVのメモリ使用量を追跡するためにHeap Profilingを有効にするかどうかを制御します。
+ デフォルト値: `true`

### `profiling-sample-per-bytes` <span class="version-mark">v7.5.0で新規追加</span>

+ ヒーププロファイリングによって毎回サンプリングされるデータ量を指定します。最も近い2の累乗に切り上げられます。
+ デフォルト値: `512KB`