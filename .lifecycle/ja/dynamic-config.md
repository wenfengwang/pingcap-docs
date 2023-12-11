---
title: ダイナミックな構成の動的な変更
summary: クラスタ構成を動的に変更する方法を学びます。
aliases: ['/docs/jp/dev/dynamic-config/']
---

# ダイナミックな構成の動的な変更

この文書では、クラスタ構成を動的に変更する方法について説明します。

SQLステートメントを使用して、TiDB、TiKV、およびPDを含むコンポーネントの構成を再起動せずに動的に更新できます。現在、TiDBインスタンスの構成を変更する方法は、TiKVおよびPDなどの他のコンポーネントの構成を変更する方法と異なります。

> **注記:**
>
> この機能はTiDB Self-Hostedにのみ適用され、[TiDB Cloud](https://docs.pingcap.com/tidbcloud/)では利用できません。TiDB Cloudをご利用の場合は、構成の変更について [TiDB Cloud サポート](https://docs.pingcap.com/tidbcloud/tidb-cloud-support) にお問い合わせください。

## 一般的な操作

このセクションでは、構成を動的に変更する一般的な操作について説明します。

### インスタンスの構成を表示

クラスタ内のすべてのインスタンスの構成を表示するには、`show config` ステートメントを使用します。結果は次のようになります。

{{< copyable "sql" >}}

```sql
show config;
```

```sql
+------+-----------------+-----------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Type | Instance        | Name                                                      | Value                                                                                                                                                                                                                                                                            |
+------+-----------------+-----------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| tidb | 127.0.0.1:4001  | advertise-address                                         | 127.0.0.1                                                                                                                                                                                                                                                                        |
| tidb | 127.0.0.1:4001  | binlog.binlog-socket                                      |                                                                                                                                                                                                                                                                                  |
| tidb | 127.0.0.1:4001  | binlog.enable                                             | false                                                                                                                                                                                                                                                                            |
| tidb | 127.0.0.1:4001  | binlog.ignore-error                                       | false                                                                                                                                                                                                                                                                            |
| tidb | 127.0.0.1:4001  | binlog.strategy                                           | range                                                                                                                                                                                                                                                                            |
| tidb | 127.0.0.1:4001  | binlog.write-timeout                                      | 15s                                                                                                                                                                                                                                                                              |
| tidb | 127.0.0.1:4001  | check-mb4-value-in-utf8                                   | true                                                                                                                                                                                                                                                                             |

...
```

結果をフィールドでフィルタリングすることができます。例：

{{< copyable "sql" >}}

```sql
show config where type='tidb'
show config where instance in (...)
show config where name like '%log%'
show config where type='tikv' and name='log.level'
```

### TiKV構成の動的な変更

> **注記:**
>
> - TiKV構成項目を動的に変更した後、TiKV構成ファイルが自動的に更新されますが、`tiup edit-config` を実行して対応する構成項目を変更する必要があります。さもなければ、`upgrade` および `reload` などの操作によって変更内容が上書きされます。構成項目の変更の詳細については、[TiUP を使用して構成を変更する](/maintain-tidb-using-tiup.md#modify-the-configuration)を参照してください。
> - `tiup edit-config` を実行した後は、`tiup reload` を実行する必要はありません。

`set config` ステートメントを使用すると、インスタンスのアドレスやコンポーネントタイプに応じて単一のインスタンスまたはすべてのインスタンスの構成を変更できます。

- すべてのTiKVインスタンスの構成を変更する:

> **注記:**
>
> 変数名はバッククォートで囲むことを推奨します。

{{< copyable "sql" >}}

```sql
set config tikv `split.qps-threshold`=1000;
```

- 個々のTiKVインスタンスの構成を変更する：

    {{< copyable "sql" >}}

    ```sql
    set config "127.0.0.1:20180" `split.qps-threshold`=1000;
    ```

変更が成功した場合、`Query OK` が返されます：

```sql
Query OK, 0 rows affected (0.01 sec)
```

バッチ変更中にエラーが発生した場合、警告が返されます：

{{< copyable "sql" >}}

```sql
set config tikv `log-level`='warn';
```

```sql
Query OK, 0 rows affected, 1 warning (0.04 sec)
```

{{< copyable "sql" >}}

```sql
show warnings;
```

```sql
+---------+------+---------------------------------------------------------------------------------------------------------------+
| Level   | Code | Message                                                                                                       |
+---------+------+---------------------------------------------------------------------------------------------------------------+
| Warning | 1105 | bad request to http://127.0.0.1:20180/config: fail to update, error: "config log-level can not be changed" |
+---------+------+---------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

バッチ変更は原子性を保証しません。変更は一部のインスタンスで成功し、他のインスタンスで失敗する可能性があります。`set tikv key=val` を使用してTiKVクラスタ全体の構成を変更すると、変更が一部のインスタンスで失敗することがあります。結果を確認するために `show warnings` を使用できます。

いくつかの変更が失敗した場合、対応するステートメントを再実行するか、失敗した各インスタンスを修正する必要があります。ネットワークの問題やマシンの障害によりいくつかのTiKVインスタンスにアクセスできない場合は、回復後にこれらのインスタンスを修正してください。

構成項目が正常に変更された場合、その結果は構成ファイルに永続的に保存され、以降の操作で優先されます。`limit` や `key` などのTiDB予約語と競合する構成項目の場合は、バッククォート `` ` `` を使用して囲んでください。たとえば、`` `raftstore.raft-log-gc-size-limit` `` 。

次のTiKV構成項目は動的に変更できます:

| 構成項目 | 説明 |
| :--- | :--- |
| log.level | ログレベル |
| `raftstore.raft-max-inflight-msgs` | 確認するRaftログの数。この数を超えると、Raft状態機械がログ送信を遅らせます。 |
| `raftstore.raft-log-gc-tick-interval` | Raftログの削除タスクのポーリング間隔 |
| `raftstore.raft-log-gc-threshold` | 残留するRaftログの最大許容数のソフトリミット |
| `raftstore.raft-log-gc-count-limit` | 残留するRaftログの最大許容数のハードリミット |
| `raftstore.raft-log-gc-size-limit` | 残留するRaftログの最大許容サイズのハードリミット |
| `raftstore.raft-max-size-per-msg` | 生成される単一メッセージパケットのサイズのソフトリミット |
| `raftstore.raft-entry-max-size` | 単一のRaftログの最大サイズのハードリミット |
| `raftstore.raft-entry-cache-life-time` | メモリ内のログキャッシュに許可される最大残り時間 |
| `raftstore.split-region-check-tick-interval` | Regionが分割される必要があるかどうかを確認する間隔 |
| `raftstore.region-split-check-diff` | Regionデータが分割前に超過しても良い最大値 |
| `raftstore.region-compact-check-interval` | 手動でRocksDBのコンパクションをトリガーする必要があるか確認する間隔 |
| `raftstore.region-compact-check-step` | 各ラウンドの手動コンパクションでチェックされるRegionの数 |
| `raftstore.region-compact-min-tombstones` | RocksDBコンパクションをトリガーするために必要な墓石の数 |
| `raftstore.region-compact-tombstones-percent` | RocksDBコンパクションをトリガーするために必要な墓石の割合 |
| `raftstore.pd-heartbeat-tick-interval` | PDによるRegionのハートビートをトリガーする間隔 |
| `raftstore.pd-store-heartbeat-tick-interval` | PDによるストアのハートビートをトリガーする間隔 |
| `raftstore.snap-mgr-gc-tick-interval` | 期限切れのスナップショットファイルのリサイクルをトリガーする間隔 |
| `raftstore.snap-gc-timeout` | スナップショットファイルが保存されている最長時間 |
| `raftstore.lock-cf-compact-interval` | ロック列ファミリーの手動コンパクションをトリガーする間隔 |
| `raftstore.lock-cf-compact-bytes-threshold` | ロック列ファミリーの手動コンパクションをトリガーするサイズ |
| `raftstore.messages-per-tick` | バッチあたりに処理される最大メッセージ数 |
| `raftstore.max-peer-down-duration` | ピアの最大非アクティブ期間 |
| `raftstore.max-leader-missing-duration` | ピアがリーダーを持たない期間の最大期間。この値を超えると、ピアはPDに削除されたかどうかを検証します。 |
| `raftstore.abnormal-leader-missing-duration` | ピアがリーダーを持たない正常な期間。この値を超えると、ピアは異常と見なされ、メトリクスやログに記録されます。 |
| `raftstore.peer-stale-state-check-interval` | ピアがリーダーを持たないかどうかを確認する間隔 |
| `raftstore.consistency-check-interval` | 一貫性を確認する間隔 (**NOT** 推奨されません。TiDBのガベージコレクションと互換性がないため） |
| `raftstore.raft-store-max-leader-lease` | Raftリーダーの最長信頼期間 |
| `raftstore.merge-check-tick-interval` | マージチェックの時間間隔 |
| `raftstore.cleanup-import-sst-interval` | 期限切れのSSTファイルをチェックする時間間隔 |
| `raftstore.local-read-batch-size` | 1つのバッチで処理される読み取りリクエストの最大数 |
| `raftstore.apply-yield-write-size` | Applyスレッドが各ラウンドで1つのFSM（有限状態機械）に書き込むことができる最大バイト数 |
| `raftstore.hibernate-timeout` | 開始時の休止状態に入る前の最短待機時間。この期間内はTiKVは休止状態に入らない（リリースされない）。 |
| `raftstore.apply-pool-size` | データをディスクにフラッシュするプール内のスレッド数、つまりApplyスレッドプールのサイズ |
| `raftstore.store-pool-size` | Raftを処理するプール内のスレッド数、つまりRaftstoreスレッドプールのサイズ |
| `raftstore.apply-max-batch-size` | Raft状態機械はBatchSystemによってデータ書き込みリクエストをバッチ処理します。この設定項目は、1つのバッチで実行できるRaft状態機械の最大数を指定します。 |
| `raftstore.store-max-batch-size` | Raft状態機械はBatchSystemによってログをディスクにフラッシュするリクエストをバッチ処理します。この設定項目は、1つのバッチで処理できるRaft状態機械の最大数を指定します。 |
| `raftstore.store-io-pool-size` | Raft I/Oタスクを処理するスレッド数、またStoreWriterスレッドプールのサイズ（0から非ゼロ値へ、または非ゼロ値から0への変更は避けてください） |
| `readpool.unified.max-thread-count` | 読み取りリクエストを均一に処理するスレッドプールの最大スレッド数 |
| `readpool.unified.auto-adjust-pool-size` | UnifyReadPoolスレッドプールのサイズを自動調整するかどうかを決定します |
| `coprocessor.split-region-on-table` | テーブルによってRegionを分割するかを有効にする |
| `coprocessor.batch-split-limit` | バッチでのRegion分割の閾値 |
| `coprocessor.region-max-size` | Regionの最大サイズ |
| `coprocessor.region-split-size` | 新しく分割されたRegionのサイズ |
| `coprocessor.region-max-keys` | 1つのRegionで許可される最大キー数 |
| `coprocessor.region-split-keys` | 新しく分割されたRegionのキー数 |
| `pessimistic-txn.wait-for-lock-timeout` | 悲観的トランザクションがロックを待機する最長時間 |
| `pessimistic-txn.wake-up-delay-duration` | 悲観的トランザクションが起こされるまでの期間 |
| `pessimistic-txn.pipelined` | パイプライン化された悲観的ロック処理を有効にするかどうかを決定します |
| `pessimistic-txn.in-memory` | インメモリの悲観的ロックを有効にするかどうかを決定します |
| `quota.foreground-cpu-time` | TiKV前面での読み取りと書き込みリクエストの処理に使用されるCPUリソースのソフト制限 |
| `quota.foreground-write-bandwidth` | 前景トランザクションがデータを書き込む際のバンド幅のソフト制限 |
| `quota.foreground-read-bandwidth` | 前景トランザクションとCoprocessorがデータを読む際のバンド幅のソフト制限 |
| `quota.background-cpu-time` | TiKV背景での読み取りと書き込みリクエストの処理に使用されるCPUリソースのソフト制限 |
| `quota.background-write-bandwidth` | 背景トランザクションがデータを書き込む際のバンド幅のソフト制限（まだ有効ではありません） |
| `quota.background-read-bandwidth` | 背景トランザクションとCoprocessorがデータを読む際のバンド幅のソフト制限（まだ有効ではありません） |
| `quota.enable-auto-tune` | クォータの自動チューニングを有効にするかどうか。この設定項目が有効の場合、TiKVはTiKVインスタンスの負荷に基づいて背景リクエストのためのクォータを動的に調整します。 |
| `quota.max-delay-duration` | 個々の読み取りまたは書き込みリクエストが前面で処理される前に強制的に待機する最大時間 |
| `gc.ratio-threshold` | Region GCがスキップされるしきい値（GCバージョンの数 / キーの数） |
| `gc.batch-keys` | 1つのバッチで処理されるキーの数 |
| `gc.max-write-bytes-per-sec` | 秒間にRocksDBに書き込むことができる最大バイト数 |
| `gc.enable-compaction-filter` | コンパクションフィルタを有効にするかどうか |
| `gc.compaction-filter-skip-version-check` | コンパクションフィルタのクラスターバージョンチェックをスキップするかどうか（まだリリースされていません） |
| `{db-name}.max-total-wal-size` | WALの合計最大サイズ |
| `{db-name}.max-background-jobs` | RocksDBでのバックグラウンドスレッドの数 |
| `{db-name}.max-background-flushes` | RocksDBでの最大フラッシュスレッド数 |
| `{db-name}.max-open-files` | RocksDBが開けるファイルの合計数 |
| `{db-name}.compaction-readahead-size` | コンパクション中の`readahead`のサイズ |
| `{db-name}.bytes-per-sync` | これらのファイルが非同期で書き込まれる間、OSがファイルをディスクに増分的に同期するレート |
| `{db-name}.wal-bytes-per-sync` | これらのWALファイルが書き込まれる間、OSがWALファイルをディスクに増分的に同期するレート |
| `{db-name}.writable-file-max-buffer-size` | WritableFileWriteで使用される最大バッファサイズ |
| `{db-name}.{cf-name}.block-cache-size` | ブロックのキャッシュサイズ |
| `{db-name}.{cf-name}.write-buffer-size` | メンタブルのサイズ |
| `{db-name}.{cf-name}.max-write-buffer-number` | メンタブルの最大数 |
| `{db-name}.{cf-name}.max-bytes-for-level-base` | ベースレベル（L1）の最大バイト数 |
| `{db-name}.{cf-name}.target-file-size-base` | ベースレベルでのターゲットファイルサイズ |
| `{db-name}.{cf-name}.level0-file-num-compaction-trigger` | コンパクションをトリガーするL0のファイル数の最大数 |
| `{db-name}.{cf-name}.level0-slowdown-writes-trigger` | 書き込みスタールをトリガーするL0のファイル数の最大数 |
| `{db-name}.{cf-name}.level0-stop-writes-trigger` | 完全に書き込みをブロックするL0のファイル数の最大数 |
| `{db-name}.{cf-name}.max-compaction-bytes` | コンパクションあたりにディスクに書き込まれる最大バイト数 |
| `{db-name}.{cf-name}.max-bytes-for-level-multiplier` | 各レイヤーのデフォルトの増幅倍数 |
| `{db-name}.{cf-name}.disable-auto-compactions` | 自動コンパクションの有効化または無効化 |
| `{db-name}.{cf-name}.soft-pending-compaction-bytes-limit` | ペンディングコンパクションバイトのソフト制限 |
| `{db-name}.{cf-name}.hard-pending-compaction-bytes-limit` | ペンディングコンパクションバイトのハード制限 |
| `{db-name}.{cf-name}.titan.blob-run-mode` | ブロブファイルの処理モード |
| `server.grpc-memory-pool-quota` | gRPCが使用できるメモリサイズを制限する |
| `server.max-grpc-send-msg-len` | 送信できるgRPCメッセージの最大長を設定する |
| `server.snap-io-max-bytes-per-sec` | スナップショット処理時の最大許容ディスク帯域幅を設定する |
| `server.concurrent-send-snap-limit` | 同時に送信できるスナップショットの最大数を設定する |
| `server.concurrent-recv-snap-limit` | 同時に受信できるスナップショットの最大数を設定する |
| `server.raft-msg-max-batch-size` | 1つのgRPCメッセージに含まれるRaftメッセージの最大数を設定する |
| `server.simplify-metrics` | サンプリング監視メトリクスを簡略化するかどうかを制御する |
| `storage.block-cache.capacity` | 共有ブロックキャッシュのサイズ（v4.0.3からサポートされています） |
| `storage.scheduler-worker-pool-size` | スケジューラスレッドプールのスレッド数 |
| `backup.num-threads` | バックアップスレッドの数（v4.0.3からサポートされています） |
| `split.qps-threshold` | Regionで`load-base-split`を実行する閾値。Regionの読み取りリクエストのQPSが`qps-threshold`を10秒連続で超える場合、このRegionは分割する必要があります。|
| `split.byte-threshold` | Regionで`load-base-split`を実行する閾値。Regionの読み取りリクエストのトラフィックが`byte-threshold`を10秒連続で超える場合、このRegionは分割する必要があります。 |
| `split.region-cpu-overload-threshold-ratio` | Regionで`load-base-split`を実行する閾値。Unified Read PoolのRegionのCPU使用率が`region-cpu-overload-threshold-ratio`を10秒連続で超える場合、このRegionは分割する必要があります（v6.2.0からサポートされています）。 |
| `split.split-balance-score` | `load-base-split`のパラメータで、2つの分割されたRegionの負荷ができるだけ均衡になるように保証します。値が小さいほど負荷が均衡されますが、小さすぎると分割が失敗する可能性があります。 |
| `split.split-contained-score` | `load-base-split` のパラメーター。値が小さいほど、リージョンが分割された後のクロスリージョン訪問が少なくなります。 |
| `cdc.min-ts-interval` | 解決済みTSが転送される時間間隔  |
| `cdc.old-value-cache-memory-quota` | TiCDCオールド値エントリが占有するメモリの上限 |
| `cdc.sink-memory-quota` | TiCDCデータ変更イベントが占有するメモリの上限 |
| `cdc.incremental-scan-speed-limit` | 履歴データのインクリメンタルスキャン速度の上限 |
| `cdc.incremental-scan-concurrency` | 履歴データの並行インクリメンタルスキャンタスクの最大数 |

上記の表では、`{db-name}` または `{db-name}.{cf-name}` 接頭辞を持つパラメーターは、RocksDBに関連する設定です。`db-name` のオプションの値は、`rocksdb` と `raftdb` です。

- `db-name` が `rocksdb` の場合、`cf-name` のオプションの値は `defaultcf`、`writecf`、`lockcf`、`raftcf`です。
- `db-name` が `raftdb` の場合、`cf-name` の値は `defaultcf` です。

詳しいパラメーターの説明については、「[TiKV Configuration File](/tikv-configuration-file.md)」を参照してください。

### PD設定を動的に変更する

現時点では、PDは各インスタンスごとの別々の設定をサポートしていません。すべてのPDインスタンスが同じ設定を共有します。

次の文を使用して、PDの設定を変更できます。

{{< copyable "sql" >}}

```sql
set config pd `log.level`='info';
```

変更が成功した場合、「`Query OK`」が返されます:

```sql
Query OK, 0 rows affected (0.01 sec)
```

設定項目が正常に変更されると、その結果は設定ファイルではなくetcdに永続化されます; etcdでの設定がその後の操作で優先されます。ある設定項目の名前は、TiDB予約語と競合する場合があります。これらの設定項目については、バッククォート（`` ` ``）で囲って使用してください。たとえば、「`` `schedule.leader-schedule-limit` ``」。

以下のPD設定項目は動的に変更できます:

| 設定項目 | 説明 |
| :--- | :--- |
| `log.level` | ログレベル |
| `cluster-version` | クラスタバージョン |
| `schedule.max-merge-region-size` | `Region Merge`のサイズ制限（MiB単位）を制御します |
| `schedule.max-merge-region-keys` | `Region Merge`キーの最大数を指定します |
| `schedule.patrol-region-interval` | `replicaChecker`がリージョンのヘルス状態をチェックする頻度を決定します |
| `schedule.split-merge-interval` | 同じリージョンでの分割とマージ操作の時間間隔を決定します |
| `schedule.max-snapshot-count` | 単一ストアが同時に送受信できるスナップショットの最大数を決定します |
| `schedule.max-pending-peer-count` | 単一ストア内の保留中のピアの最大数を決定します |
| `schedule.max-store-down-time` | PDが切断されたストアを回復できないと判断するダウンタイム |
| `schedule.leader-schedule-policy` | リーダースケジューリングのポリシーを決定します |
| `schedule.leader-schedule-limit` | 同時に実行されるリーダースケジューリングタスクの数 |
| `schedule.region-schedule-limit` | 同時に実行されるリージョンスケジューリングタスクの数 |
| `schedule.replica-schedule-limit` | 同時に実行されるレプリカスケジューリングタスクの数 |
| `schedule.merge-schedule-limit` | 同時に実行される`Region Merge`のスケジュールタスクの数 |
| `schedule.hot-region-schedule-limit` | 同時に実行されるホットリージョンスケジュールタスクの数 |
| `schedule.hot-region-cache-hits-threshold` | リージョンがホットスポットと見なされるしきい値 |
| `schedule.high-space-ratio` | ストアの容量が十分であるしきい値の下限比率 |
| `schedule.low-space-ratio` | ストアの容量が不十分であるしきい値の上限比率 |
| `schedule.tolerant-size-ratio` | `balance`バッファサイズを制御します |
| `schedule.enable-remove-down-replica` | `DownReplica`を自動的に削除する機能を有効にするかどうかを決定します |
| `schedule.enable-replace-offline-replica` | `OfflineReplica`を自動的にみ移行する機能を有効にするかどうかを決定します |
| `schedule.enable-make-up-replica` | レプリカを自動的に補完する機能を有効にするかどうかを決定します |
| `schedule.enable-remove-extra-replica` | 余分なレプリカを削除する機能を有効にするかどうかを決定します |
| `schedule.enable-location-replacement` | 分離レベルチェックを有効にするかどうかを決定します |
| `schedule.enable-cross-table-merge` | クロステーブルマージを有効にするかどうかを決定します |
| `schedule.enable-one-way-merge` | 1方向マージを有効にするかどうかを決定します |
| `replication.max-replicas` | 最大レプリカ数を設定します |
| `replication.location-labels` | TiKVクラスタのトポロジ情報 |
| `replication.enable-placement-rules` | プレースメントルールを有効にします |
| `replication.strictly-match-label` | ラベルチェックを有効にします |
| `pd-server.use-region-storage` | 独立したリージョンストレージを有効にします |
| `pd-server.max-gap-reset-ts` | タイムスタンプのリセット最大間隔（BR）を設定します |
| `pd-server.key-type` | クラスタキータイプを設定します |
| `pd-server.metric-storage` | クラスタメトリクスのストレージアドレスを設定します |
| `pd-server.dashboard-address` | ダッシュボードアドレスを設定します |
| `replication-mode.replication-mode` | バックアップモードを設定します |

詳しいパラメーターの説明については、「[PD Configuration File](/pd-configuration-file.md)」を参照してください。

### TiDB設定を動的に変更する

現時点では、TiDBの設定変更方法はTiKVおよびPDの設定の変更方法とは異なります。TiDB設定は [システム変数](/system-variables.md) を使用して変更できます。

以下の例は、「`tidb_slow_log_threshold`」変数を使用して、「`slow-threshold`」を動的に変更する方法を示しています。

`slow-threshold`のデフォルト値は300 msです。`tidb_slow_log_threshold`を使用して、それを200 msに設定することができます。

{{< copyable "sql" >}}

```sql
set tidb_slow_log_threshold = 200;
```

```sql
Query OK, 0 rows affected (0.00 sec)
```

{{< copyable "sql" >}}

```sql
select @@tidb_slow_log_threshold;
```

```sql
+---------------------------+
| @@tidb_slow_log_threshold |
+---------------------------+
| 200                       |
+---------------------------+
1 row in set (0.00 sec)
```

以下のTiDB設定項目は動的に変更できます:

| 設定項目 | SQL変数 | 説明 |
| :--- | :--- | :--- |
| `instance.tidb_enable_slow_log` | `tidb_enable_slow_log` | スローログを有効にするかどうか |
| `instance.tidb_slow_log_threshold` | `tidb_slow_log_threshold` | スローログのしきい値 |
| `instance.tidb_expensive_query_time_threshold` | `tidb_expensive_query_time_threshold` | 費用のかかるクエリのしきい値 |

### TiFlash設定を動的に変更する

現時点では、リクエストを実行する最大同時実行数を制御するシステム変数 [`tidb_max_tiflash_threads`](/system-variables.md#tidb_max_tiflash_threads-new-in-v610) を使用して、TiFlash設定 `max_threads` を動的に変更できます。

`tidb_max_tiflash_threads`のデフォルト値は`-1`であり、このシステム変数が無効であり、TiFlashの設定ファイルの設定に依存します。`tidb_max_tiflash_threads`を使用して`max_threads`を10に設定できます:

{{< copyable "sql" >}}

```sql
set tidb_max_tiflash_threads = 10;
```

```sql
Query OK, 0 rows affected (0.00 sec)
```

{{< copyable "sql" >}}

```sql
select @@tidb_max_tiflash_threads;
```

```sql
+----------------------------+
| @@tidb_max_tiflash_threads |
+----------------------------+
| 10                         |
+----------------------------+
1 row in set (0.00 sec)
```