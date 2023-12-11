---
title: TiDB 7.3.0 リリースノート
summary: TiDB 7.3.0 での新機能、互換性の変更、改善、およびバグ修正について学びましょう。
---

# TiDB 7.3.0 リリースノート

リリース日: 2023年8月14日

TiDB バージョン: 7.3.0

クイックアクセス: [クイックスタート](https://docs.pingcap.com/tidb/v7.3/quick-start-with-tidb) | [インストールパッケージ](https://www.pingcap.com/download/?version=v7.3.0#version-list)

7.3.0 には次の主要な機能が追加されています。また、7.3.0 には、TiDB サーバーおよび TiFlash のクエリの安定性に関する一連の改善（[機能の詳細](#feature-details) セクションで説明）も含まれています。これらの改善はより雑多な性質のものであり、ユーザーには関係ないため、以下の表には含まれていません。

<table>
<thead>
  <tr>
    <th>カテゴリ</th>
    <th>機能</th>
    <th>説明</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>スケーラビリティとパフォーマンス</td>
    <td>TiDB Lightning が <a href="https://docs.pingcap.com/tidb/v7.3/partitioned-raft-kv">Partitioned Raft KV</a> をサポート（実験的）</td>
    <td>TiDB Lightning は今後のアーキテクチャの一部として、Partitioned Raft KV アーキテクチャをサポートします。</td>
  </tr>
  <tr>
    <td rowspan="2">信頼性と可用性</td>
    <td><a href="https://docs.pingcap.com/tidb/v7.3/tidb-lightning-physical-import-mode-usage#conflict-detection">データインポート時の自動競合検出と解決を追加</a></td>
    <td>新しい競合検出のバージョンをサポートし、競合データを自動的に処理します。</td>
  </tr>
  <tr>
    <td><a href="https://docs.pingcap.com/tidb/v7.3/tidb-resource-control#query-watch-parameters">予期せぬ巨大なクエリの手動管理</a>（実験的）</td>
    <td>新しいリソースグループの監視リストを使用して、クエリを効果的に管理し、優先順位を付けたり、終了させたりできます。</td>
  </tr>
  <tr>
    <td>SQL</td>
    <td><a href="https://docs.pingcap.com/tidb/v7.3/optimizer-hints">クエリプランナーによるより多くのオプティマイザーヒントの追加によるクエリ安定性の向上</a></td>
    <td>追加されたヒント： <code>NO_INDEX_JOIN()</code>, <code>NO_MERGE_JOIN()</code>, <code>NO_INDEX_MERGE_JOIN()</code>, <code>NO_HASH_JOIN()</code>, <code>NO_INDEX_HASH_JOIN()</code>
    </td>
  </tr>
  <tr>
    <td>DB 操作と可観測性</td>
    <td><a href="https://docs.pingcap.com/tidb/v7.3/sql-statement-show-analyze-status">統計収集タスクの進行状況を表示</a></td>
    <td><code>SHOW ANALYZE STATUS</code> 文または <code>mysql.analyze_jobs</code> システムテーブルを使用して、<code>ANALYZE</code> タスクの進行状況を表示サポートします。</td>
  </tr>
</tbody>
</table>

## 機能の詳細

### パフォーマンス

* TiFlash がリプリカの選択戦略をサポート [#44106](https://github.com/pingcap/tidb/issues/44106) @[XuHuaiyu](https://github.com/XuHuaiyu)

    v7.3.0 より前は、TiFlash はデータのスキャンおよび MPP 計算のためにすべてのノードからリプリカを使用していました。v7.3.0 からは、TiFlash はリプリカの選択戦略を導入し、[`tiflash_replica_read`](/system-variables.md#tiflash_replica_read-new-in-v730) システム変数を使用して構成できるようになりました。この戦略は、ノードの[ゾーン属性](/schedule-replicas-by-topology-labels.md#optional-configure-labels-for-tidb)に基づいて特定のリプリカを選択し、データスキャンおよび MPP 計算のために特定のノードをスケジュールすることをサポートします。

    複数のデータセンターに展開され、各データセンターに完全な TiFlash データリプリカがあるクラスターの場合、この戦略を構成して現在のデータセンターからの TiFlash リプリカのみを選択できます。これにより、データスキャンおよび MPP 計算を現在のデータセンターの TiFlash ノードのみで実行でき、データセンター間の過剰なネットワークデータ転送を回避できます。

    詳細については、[ドキュメント](/system-variables.md#tiflash_replica_read-new-in-v730) を参照してください。

* TiFlash がノード内でのランタイムフィルタをサポート [#40220](https://github.com/pingcap/tidb/issues/40220) @[elsa0520](https://github.com/elsa0520)

    ランタイムフィルタは、クエリプランニングフェーズ中に生成される**動的述語**です。表の結合プロセスでは、これらの動的述語は結合条件を満たさない行を効果的にフィルタリングし、スキャン時間とネットワークのオーバーヘッドを減らし、表の結合の効率を向上させます。v7.3.0 より、TiFlash はノード内でのランタイムフィルタをサポートし、分析クエリ全体のパフォーマンスを向上させます。TPC-DS ワークロードでは、パフォーマンスが10%から50%向上することがあります。

    この機能は v7.3.0 ではデフォルトで無効になっています。この機能を有効にするには、システム変数 [`tidb_runtime_filter_mode`](/system-variables.md#tidb_runtime_filter_mode-new-in-v720) を `LOCAL` に設定します。

    詳細については、[ドキュメント](/runtime-filter.md) を参照してください。

* TiFlash が共通テーブル式（CTE）の実行をサポート（実験的） [#43333](https://github.com/pingcap/tidb/issues/43333) @[winoros](https://github.com/winoros)

    v7.3.0 より前は、TiFlash の MPP エンジンはデフォルトで CTE を含むクエリを実行できませんでした。MPP フレームワーク内で最適な実行パフォーマンスを達成するには、システム変数 [`tidb_opt_force_inline_cte`](/system-variables.md#tidb_opt_force_inline_cte-new-in-v630) を使用して CTE をインライン化する必要があります。

    v7.3.0 より、TiFlash の MPP エンジンは、CTE をインライン化せずに実行することをサポートし、MPP フレームワーク内での最適なクエリ実行を可能にします。TPC-DS ベンチマークテストでは、CTE を含むクエリの全体的な実行速度が20%向上することが示されています。

    この機能は実験的であり、デフォルトでは無効になっています。この機能はシステム変数 [`tidb_opt_enable_mpp_shared_cte_execution`](/system-variables.md#tidb_opt_enable_mpp_shared_cte_execution-new-in-v720) で制御されます。

### 信頼性

* 新しいオプティマイザーヒントを追加 [#45520](https://github.com/pingcap/tidb/issues/45520) @[qw4990](https://github.com/qw4990)

    v7.3.0 では、TiDB はテーブル間の結合方法を制御するための複数の新しいオプティマイザーヒントを導入しました：

    - [`NO_MERGE_JOIN()`](/optimizer-hints.md#no_merge_joint1_name--tl_name-) マージ結合以外の結合方法を選択します。
    - [`NO_INDEX_JOIN()`](/optimizer-hints.md#no_index_joint1_name--tl_name-) インデックスネストループ結合以外の結合方法を選択します。
    - [`NO_INDEX_MERGE_JOIN()`](/optimizer-hints.md#no_index_merge_joint1_name--tl_name-) インデックスネストループマージ結合以外の結合方法を選択します。
    - [`NO_HASH_JOIN()`](/optimizer-hints.md#no_hash_joint1_name--tl_name-) ハッシュ結合以外の結合方法を選択します。
    - [`NO_INDEX_HASH_JOIN()`](/optimizer-hints.md#no_index_hash_joint1_name--tl_name-) [インデックスネストループハッシュ結合](/optimizer-hints.md#inl_hash_join)以外の結合方法を選択します。

  詳細については、[ドキュメント](/optimizer-hints.md) を参照してください。

* 予期されるよりも多くのリソースを使用するクエリを手動でマークする（実験的） [#43691](https://github.com/pingcap/tidb/issues/43691) @[Connor1996](https://github.com/Connor1996) @[CabinfeverB](https://github.com/CabinfeverB)
```yaml
    v7.2.0では、TiDBは予想以上のリソースを使用するクエリ（暴走クエリ）を自動的に低ランク化またはキャンセルすることで自動的に管理します。実際の運用では、ルールだけではすべてのケースをカバーすることはできません。そのため、TiDB v7.3.0では、暴走クエリを手動でマークする機能が導入されています。新しい[`QUERY WATCH`](/sql-statements/sql-statement-query-watch.md)コマンドを使用すると、SQLテキスト、SQL Digest、または実行計画に基づいて暴走クエリをマークできます。そして、マークされた暴走クエリは低ランク化またはキャンセルされます。

    この機能により、データベースの突然のパフォーマンスの問題に効果的に介入できます。クエリによるパフォーマンスの問題が原因である場合、ルート原因が特定される前に、この機能を使用することで全体のパフォーマンスに対する影響を迅速に緩和し、システムサービスの品質を向上させることができます。

    詳細については、[ドキュメント](/tidb-resource-control.md#query-watch-parameters)を参照してください。

### SQL

* リストおよびリストCOLUMNSパーティションテーブルでのデフォルトパーティションのサポート [#20679](https://github.com/pingcap/tidb/issues/20679) @[mjonss](https://github.com/mjonss) @[bb7133](https://github.com/bb7133)

    v7.3.0以前では、`INSERT`文を使用してリストまたはリストCOLUMNSパーティションテーブルにデータを挿入する際、データはテーブルの指定されたパーティション条件を満たす必要がありました。挿入するデータがこれらの条件のいずれにも適合しない場合、文の実行が失敗するか、非適合のデータが無視されました。

   v7.3.0からは、リストおよびリストCOLUMNSパーティションテーブルでデフォルトパーティションがサポートされます。デフォルトパーティションが作成されると、挿入するデータがいずれのパーティション条件も満たさない場合、デフォルトパーティションに書き込まれます。この機能により、リストおよびリストCOLUMNSパーティショニングの使用が向上し、`INSERT`文の実行失敗やパーティション条件を満たさないデータによるデータの無視を回避することができます。

    なお、この機能はMySQL構文に対するTiDBの拡張です。デフォルトパーティションを持つパーティションテーブルのデータは、直接MySQLにレプリケートすることはできません。

    詳細については、[ドキュメント](/partitioned-table.md#list-partitioning)を参照してください。

### 可観測性

* 統計情報の収集進捗の表示 [#44033](https://github.com/pingcap/tidb/issues/44033) @[hawkingrei](https://github.com/hawkingrei)

    大規模なテーブルの統計情報の収集にはしばしば時間がかかります。以前のバージョンでは、統計情報の収集の進捗を表示することができず、完了時刻を予測することができませんでした。TiDB v7.3.0では、統計情報の収集進捗を表示する機能が導入されました。システムテーブル `mysql.analyze_jobs` または `SHOW ANALYZE STATUS`を使用すると、全体の作業量、現在の進捗、および各サブタスクの予想完了時刻を表示できます。大規模なデータのインポートやSQLパフォーマンスの最適化などのシナリオでは、この機能によって全体のタスク進捗を把握し、ユーザーエクスペリエンスを向上させることができます。

    詳細については、[ドキュメント](/sql-statements/sql-statement-show-analyze-status.md)を参照してください。

* プランリプレイヤーが履歴統計情報のエクスポートをサポート [#45038](https://github.com/pingcap/tidb/issues/45038) @[time-and-fate](https://github.com/time-and-fate)

    v7.3.0から、新しく追加された [`dump with stats as of timestamp`](/sql-plan-replayer.md)句を使用して、プランリプレイヤーを使用して特定の時点でのSQL関連オブジェクトの統計情報をエクスポートできます。実行計画の問題の診断時に、履歴統計情報を正確に取得することで、問題が発生した時点での実行計画がどのように生成されたかをより正確に分析することができます。これにより、問題の原因を特定でき、実行計画の問題の診断効率が大幅に向上します。

    詳細については、[ドキュメント](/sql-plan-replayer.md)を参照してください。

### データ移行

* TiDB Lightningが新しいバージョンの競合データ検出および処理戦略を導入 [#41629](https://github.com/pingcap/tidb/issues/41629) @[lance6716](https://github.com/lance6716)

    以前のバージョンでは、TiDB Lightningは論理インポートモードおよび物理インポートモードそれぞれに異なる競合検出および処理方法を使用しており、構成が複雑でユーザーが理解しにくかったです。さらに、物理インポートモードでは `replace`または `ignore`戦略を使用して競合を処理することができませんでした。v7.3.0からは、TiDB Lightningは論理インポートモードおよび物理インポートモードの両方に統一された競合検出および処理戦略を導入します。競合したデータが発生した場合、エラーを報告する（`error`）、置換する（`replace`）または無視する（`ignore`）ことが選択できます。競合レコードの数を制限することもできます。たとえば、一定数量の競合レコードの処理後にタスクが中断および終了するなどです。さらに、システムはトラブルシューティングのために競合データを記録することができます。

    多くの競合を含むデータをインポートする場合、パフォーマンスの向上のために新しいバージョンの競合検出および処理戦略を使用することをお勧めします。ラボ環境では、新しいバージョンの戦略を使用した場合、競合検出および処理のパフォーマンスが、古いバージョンよりも3倍速く向上することがあります。このパフォーマンス値は参考用にのみです。実際のパフォーマンスは、構成、テーブル構造、および競合データの割合に応じて異なる可能性があります。なお、新しいバージョンと古いバージョンの競合戦略を同時に使用することはできません。古い競合検出および処理戦略は将来的には非推奨となります。

    詳細については、[ドキュメント](/tidb-lightning/tidb-lightning-physical-import-mode-usage.md#conflict-detection)を参照してください。

* TiDB Lightningがパーティション化されたRaft-KVをサポート（実験的） [#14916](https://github.com/tikv/tikv/issues/14916) @[GMHDBJD](https://github.com/GMHDBJD)

    TiDB Lightningは現在、パーティション化されたRaft-KVをサポートしています。この機能により、TiDB Lightningのデータインポートのパフォーマンスが向上します。

* TiDB Lightningが診断ログをより多く出力してトラブルシューティングを強化するための新しいパラメータ `enable-diagnose-log` を導入 [#45497](https://github.com/pingcap/tidb/issues/45497) @[D3Hunter](https://github.com/D3Hunter)

    この機能はデフォルトでは無効であり、TiDB Lightningは `lightning/main` を含むログのみを出力します。有効にした場合、TiDB Lightningは `client-go` や `tidb` などのすべてのパッケージのログを出力し、`client-go` や `tidb` に関連する問題を診断するのに役立ちます。

    詳細については、[ドキュメント](/tidb-lightning/tidb-lightning-configuration.md#tidb-lightning-global)を参照してください。

## 互換性の変更

> **注意:**
>
> このセクションでは、v7.2.0から現在のバージョン（v7.3.0）にアップグレードする際に把握する必要のある互換性の変更情報を提供します。v7.1.0またはそれ以前のバージョンから現在のバージョンにアップグレードする場合は、中間バージョンで導入された互換性の変更を確認する必要がある場合があります。

### 動作の変更

* バックアップ＆リストア（BR）

    - BRは完全なデータ復元を実行する前に空クラスタチェックを追加します。デフォルトでは、空でないクラスタにデータを復元することは許可されていません。復元を強制する場合は、`--filter`オプションを使用して対応するテーブル名を指定してデータを復元できます。

* TiDB Lightning

    - `tikv-importer.on-duplicate` は非推奨とされ、[`conflict.strategy`](/tidb-lightning/tidb-lightning-configuration.md#tidb-lightning-task) に置き換えられました。
    - マイグレーションタスクを停止させるまでにTiDB Lightningが許容できる非致命的エラーの最大数を制御する `max-error` パラメータは、インポートデータの競合を制限しなくなりました。今後は、競合レコードの最大数を制御する [`conflict.threshold`](/tidb-lightning/tidb-lightning-configuration.md#tidb-lightning-task) パラメータが制御します。

* TiCDC

    - Kafka SinkがAvroプロトコルを使用する場合、`force-replicate`パラメータが`true`に設定されている場合、TiCDCはチェンジフィードを作成する際にエラーを報告します。
    - `delete-only-output-handle-key-columns` と `force-replicate` パラメータの非互換性により、両方のパラメータが有効になっている場合、TiCDCはチェンジフィードの作成時にエラーを報告します。
    - 出力プロトコルがOpen Protocolの場合、`UPDATE`イベントは変更された列のみを出力します。

### システム変数

| 変数名 | 変更タイプ | 説明 |
|--------|------------------------------|------|
| [`tidb_opt_enable_mpp_shared_cte_execution`](/system-variables.md#tidb_opt_enable_mpp_shared_cte_execution-new-in-v720) | 変更 | このシステム変数はv7.3.0以降に効果を発揮します。これにより、非再帰Common Table Expressions（CTE）をTiFlash MPPで実行するかどうかを制御できます。 |
| [`tidb_lock_unchanged_keys`](/system-variables.md#tidb_lock_unchanged_keys-new-in-v711-and-v730) | 新規追加 | この変数は、特定のシナリオでトランザクションで変更されないが関与しているキーをロックするかどうかを制御します。 |
| [`tidb_opt_enable_non_eval_scalar_subquery`](/system-variables.md#tidb_opt_enable_non_eval_scalar_subquery-new-in-v730) | 新規追加 | `EXPLAIN`文で最適化段階で展開できる定数サブクエリの実行を無効にするかどうかを制御します。 |
| [`tidb_skip_missing_partition_stats`](/system-variables.md#tidb_skip_missing_partition_stats-new-in-v730) | 追加 | この変数は、パーティション統計が欠落している場合にGlobalStatsの生成を制御します。 |
| [`tiflash_replica_read`](/system-variables.md#tiflash_replica_read-new-in-v730) | 追加 | クエリがTiFlashエンジンを必要とする場合のTiFlashレプリカの選択戦略を制御します。 |

### 設定ファイルパラメータ

| 設定ファイル | 設定パラメータ | 変更タイプ | 説明 |
| -------- | -------- | -------- | -------- |
| TiDB | [`enable-32bits-connection-id`](/tidb-configuration-file.md#enable-32bits-connection-id-new-in-v730) | 追加 | 32ビット接続ID機能を有効にするかどうかを制御します。 |
| TiDB | [`in-mem-slow-query-recent-num`](/tidb-configuration-file.md#in-mem-slow-query-recent-num-new-in-v730) | 追加 | メモリにキャッシュされる最近使用された遅いクエリの数を制御します。 |
| TiDB | [`in-mem-slow-query-topn-num`](/tidb-configuration-file.md#in-mem-slow-query-topn-num-new-in-v730) | 追加 | メモリにキャッシュされる最も遅いクエリの数を制御します。 |
| TiKV | [`coprocessor.region-bucket-size`](/tikv-configuration-file.md#region-bucket-size-new-in-v610) | 修正済み | デフォルト値を `96MiB` から `50MiB` に変更します。 |
| TiKV | [`raft-engine.format-version`](/tikv-configuration-file.md#format-version-new-in-v630) | 修正済み | Partitioned Raft KV (`storage.engine="partitioned-raft-kv"`) を使用する場合、Ribbonフィルタが使用されます。そのため、TiKVはデフォルト値を `2` から `5` に変更します。 |
| TiKV | [`raftdb.max-total-wal-size`](/tikv-configuration-file.md#max-total-wal-size-1) | 修正済み | Partitioned Raft KV (`storage.engine="partitioned-raft-kv"`) を使用する場合、TiKVはWALの書き込みをスキップします。そのため、TiKVはデフォルト値を `"4GB"` から `1` に変更し、WALを無効にします。 |
| TiKV | [<code>rocksdb.\[defaultcf\|writecf\|lockcf\].compaction-guard-min-output-file-size</code>](/tikv-configuration-file.md#compaction-guard-min-output-file-size) | 修正済み | デフォルト値を `"1MB"` から `"8MB"` に変更します。大容量データの書き込み中にコンパクション速度が遅れる問題を解決するためです。 |
| TiKV | [<code>rocksdb.\[defaultcf\|writecf\|lockcf\].format-version</code>](/tikv-configuration-file.md#format-version-new-in-v620) | 修正済み | Partitioned Raft KV (`storage.engine="partitioned-raft-kv"`) を使用する場合、Ribbonフィルタが使用されます。そのため、TiKVはデフォルト値を `2` から `5` に変更します。 |
| TiKV | [`rocksdb.lockcf.write-buffer-size`](/tikv-configuration-file.md#write-buffer-size) | 修正済み | Partitioned Raft KV (`storage.engine="partitioned-raft-kv"`) を使用する場合、lockcf上のコンパクションを高速化するため、TiKVはデフォルト値を `"32MB"` から `"4MB"` に変更します。 |
| TiKV | [`rocksdb.max-total-wal-size`](/tikv-configuration-file.md#max-total-wal-size) | 修正済み | Partitioned Raft KV (`storage.engine="partitioned-raft-kv"`) を使用する場合、TiKVはWALの書き込みをスキップします。そのため、TiKVはデフォルト値を `"4GB"` から `1` に変更し、WALを無効にします。 |
| TiKV | [`rocksdb.stats-dump-period`](/tikv-configuration-file.md#stats-dump-period) | 修正済み | Partitioned Raft KV (`storage.engine="partitioned-raft-kv"`) を使用する場合、冗長なログ出力を無効にするため、デフォルト値を `"10m"` から `"0"` に変更します。 |
| TiKV | [`rocksdb.write-buffer-limit`](/tikv-configuration-file.md#write-buffer-limit-new-in-v660) | 修正済み | `storage.engine="raft-kv"` の場合、メムテーブルのメモリオーバーヘッドを減らすため、TiKVはデフォルト値をマシンのメモリの25% から `0` に変更します（無制限）。また、Partitioned Raft KV (`storage.engine="partitioned-raft-kv"`) を使用する場合、デフォルト値をマシンのメモリの25% から20%に変更します。 |
| TiKV | [`storage.block-cache.capacity`](/tikv-configuration-file.md#capacity) | 修正済み | Partitioned Raft KV (`storage.engine="partitioned-raft-kv"`) を使用する場合、メムテーブルのメモリオーバーヘッドを補償するため、TiKVはデフォルト値を総システムメモリの45% から30%に変更します。 |
| TiFlash | [`storage.format_version`](/tiflash/tiflash-configuration.md) | 修正済み | 新しいDTFileフォーマット `format_version = 5` を導入し、小さいファイルをマージして物理ファイルの数を減らします。なお、このフォーマットは実験的なものであり、デフォルトでは有効になっていません。 |
| TiDB Lightning  | `tikv-importer.incremental-import` | 削除済み | TiDB Lightning並列インポートパラメータ。これは誤って増分インポートパラメータと間違えられやすいため、「tikv-importer.parallel-import」という新しい名前に変更されました。古いパラメータ名を渡すと自動的に新しいものに変換されます。 |
| TiDB Lightning  | `tikv-importer.on-duplicate` | 廃止予定 | 論理インポートモードで競合するレコードを挿入しようとした際のアクションを制御します。v7.3.0以降、このパラメータには[`conflict.strategy`](/tidb-lightning/tidb-lightning-configuration.md#tidb-lightning-task)が代わります。 |
| TiDB Lightning | [`conflict.max-record-rows`](/tidb-lightning/tidb-lightning-configuration.md) | 追加 | 競合するデータを処理する新しい戦略バージョン。`conflict_records`テーブルの最大行数を制御します。デフォルト値は100です。 |
| TiDB Lightning  | [`conflict.strategy`](/tidb-lightning/tidb-lightning-configuration.md) | 追加 | 競合するデータを処理する新しい戦略バージョン。以下のオプションを含みます: ""（TiDB Lightningは競合データを検出して処理しません）、`error`（インポート中にプライマリキーまたは一意キーの競合が検出されると、インポートが終了されエラーが報告されます）、`replace`（競合するプライマリキーまたは一意キーのデータが発生した場合、新しいデータが保持され古いデータが上書きされます）、`ignore`（競合するプライマリキーまたは一意キーのデータが発生した場合、古いデータが保持され新しいデータが無視されます）。デフォルト値は "" で、つまりTiDB Lightningは競合データを検出して処理しません。 |
| TiDB Lightning  | [`conflict.threshold`](/tidb-lightning/tidb-lightning-configuration.md) | 追加 | 競合するデータの上限を制御します。`conflict.strategy="error"` の場合、デフォルト値は `0` です。`conflict.strategy="replace"` または `conflict.strategy="ignore"` の場合にはmaxintと設定できます。 |
| TiDB Lightning  | [`enable-diagnose-logs`](/tidb-lightning/tidb-lightning-configuration.md) | 追加 | 診断ログを有効にするかどうかを制御します。デフォルト値は `false` で、つまりインポートに関連するログのみが出力されます。他の依存コンポーネントのログは出力されません。 `true` に設定すると、インポートプロセスと他の依存コンポーネントのログが出力され、GRPCデバッグが有効になり、診断に使用できます。 |
|TiDB Lightning  | [`tikv-importer.parallel-import`](/tidb-lightning/tidb-lightning-configuration.md) | 追加 | TiDB Lightning並列インポートパラメータ。誤解されやすい増分インポートパラメータ `tikv-importer.incremental-import` の代わりになります。 |
|BR  | `azblob.encryption-scope` | 追加 | BRはAzure Blob Storageの暗号化スコープをサポートします。 |
|BR  | `azblob.encryption-key` | 追加 | BRはAzure Blob Storageの暗号化キーをサポートします。 |
| TiCDC | [`large-message-handle-option`](/ticdc/ticdc-sink-to-kafka.md#handle-messages-that-exceed-the-kafka-topic-limit) | 追加 | デフォルトでは空です。つまり、メッセージサイズがKafkaトピックの制限を超えるとチェンジフィードが失敗します。この設定を`"handle-key-only"`に設定すると、メッセージがサイズ制限を超える場合、ハンドルキーのみが送信されてメッセージサイズが減少します。減少したメッセージも制限を超える場合は、チェンジフィードが失敗します。 |
| TiCDC | [`sink.csv.binary-encoding-method`](/ticdc/ticdc-changefeed-config.md#changefeed-configuration-parameters) | 追加 | バイナリデータのエンコーディング方法。`'base64'` または `'hex'` に設定できます。デフォルト値は `'base64'` です。 |

### システムテーブル

- 内部タイマーのメタデータを格納する新しいシステムテーブル `mysql.tidb_timers` を追加しました。

## 廃止された機能

* TiDB
- [`Fast Analyze`](/system-variables.md#tidb_enable_fast_analyze) 機能（実験的）による統計情報は v7.5.0 で非推奨になります。
    - 統計情報を取得するための [増分コレクション](https://docs.pingcap.com/tidb/v7.3/statistics#incremental-collection) 機能は v7.5.0 で非推奨になります。

## 改善点

+ TiDB

    - 新しいシステム変数 [`tidb_opt_enable_non_eval_scalar_subquery`](/system-variables.md#tidb_opt_enable_non_eval_scalar_subquery-new-in-v730) を導入し、`EXPLAIN` 文が最適化フェーズ中にサブクエリを事前に実行するかどうかを制御します [#22076](https://github.com/pingcap/tidb/issues/22076) @[winoros](https://github.com/winoros)
    - [グローバルキル](/tidb-configuration-file.md#enable-global-kill-new-in-v610) が有効になっている場合、<kbd>Control+C</kbd> を押して現在のセッションを終了できます [#8854](https://github.com/pingcap/tidb/issues/8854) @[pingyu](https://github.com/pingyu)
    - `IS_FREE_LOCK()` と `IS_USED_LOCK()` のロック関数をサポートします [#44493](https://github.com/pingcap/tidb/issues/44493) @[dveeden](https://github.com/dveeden)
    - ディスクからダンプされたチャンクの読み取りパフォーマンスを最適化します [#45125](https://github.com/pingcap/tidb/issues/45125) @[YangKeao](https://github.com/YangKeao)
    - インデックス結合の内部テーブルの過大評価問題を、Optimizer Fix Controls を使用することで最適化します [#44855](https://github.com/pingcap/tidb/issues/44855) @[time-and-fate](https://github.com/time-and-fate)

+ TiKV

    - `安全な-ts` の最大ギャップと `安全な-ts` の最小リージョンメトリクスを追加し、`tikv-ctl get-region-read-progress` コマンドを導入して、解決された-ts と安全な-ts の状態をよりよく観察および診断します [#15082](https://github.com/tikv/tikv/issues/15082) @[ekexium](https://github.com/ekexium)

+ PD

    - Swagger サーバーが無効になっている場合、Swagger API をデフォルトでブロックするサポートを追加します [#6786](https://github.com/tikv/pd/issues/6786) @[bufferflies](https://github.com/bufferflies)
    - etcd の高可用性を改善します [#6554](https://github.com/tikv/pd/issues/6554) [#6442](https://github.com/tikv/pd/issues/6442) @[lhy1024](https://github.com/lhy1024)
    - `GetRegions` リクエストのメモリ消費を削減します [#6835](https://github.com/tikv/pd/issues/6835) @[lhy1024](https://github.com/lhy1024)

+ TiFlash

    - 新しい DTFile フォーマットバージョン [`storage.format_version = 5`](/tiflash/tiflash-configuration.md) をサポートし、物理ファイルの数を減らします（実験的） [#7595](https://github.com/pingcap/tiflash/issues/7595) @[hongyunyan](https://github.com/hongyunyan)

+ Tools

    + バックアップ＆リストア（BR）

        - BR を使用してデータを Azure Blob Storage にバックアップする際、サーバーサイド暗号化のために暗号化スコープまたは暗号化キーを指定できます [#45025](https://github.com/pingcap/tidb/issues/45025) @[Leavrth](https://github.com/Leavrth)

    + TiCDC

        - `UPDATE` イベントを送信する際に、更新された列の値のみを含めるように Open Protocol のメッセージサイズを最適化します [#9336](https://github.com/pingcap/tiflow/issues/9336) @[3AceShowHand](https://github.com/3AceShowHand)
        - ストレージシンクは、HEX形式のデータに対応するための16進エンコーディングをサポートし、AWS DMSのフォーマット仕様と互換性があります [#9373](https://github.com/pingcap/tiflow/issues/9373) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - メッセージが大きすぎる場合にのみ、ハンドルキーのデータを送信するように Kafka Sink をサポートし、メッセージのサイズを削減します [#9382](https://github.com/pingcap/tiflow/issues/9382) @[3AceShowHand](https://github.com/3AceShowHand)

## バグ修正

+ TiDB

    - MySQL Cursor Fetch プロトコルを使用すると、結果セットのメモリ消費が `tidb_mem_quota_query` 制限を超え、TiDB OOM を引き起こす問題を修正します。修正後、TiDB は自動的に結果セットをディスクに書き込んでメモリを解放します [#43233](https://github.com/pingcap/tidb/issues/43233) @[YangKeao](https://github.com/YangKeao)
    - データ競合によって引き起こされる TiDB パニック問題を修正します [#45561](https://github.com/pingcap/tidb/issues/45561) @[genliqi](https://github.com/gengliqi)
    - `indexMerge` を持つクエリがキャンセルされた際に発生するハングアップ問題を修正します [#45279](https://github.com/pingcap/tidb/issues/45279) @[xzhangxian1008](https://github.com/xzhangxian1008)
    - `tidb_enable_parallel_apply` が有効になっている場合、MPPモードでのクエリ結果が正しくない問題を修正します [#45299](https://github.com/pingcap/tidb/issues/45299) @[windtalker](https://github.com/windtalker)
    - `PD` 時間に急な変化がある場合に、`resolve lock` がハングアップする可能性がある問題を修正します [#44822](https://github.com/pingcap/tidb/issues/44822) @[zyguan](https://github.com/zyguan)
    - GC Resolve Locks ステップで一部の悲観的ロックを見逃す可能性がある問題を修正します [#45134](https://github.com/pingcap/tidb/issues/45134) @[MyonKeminta](https://github.com/MyonKeminta)
    - ダイナミックプルーニングモードで `ORDER BY` を持つクエリが正しい結果を返さない問題を修正します [#45007](https://github.com/pingcap/tidb/issues/45007) @[Defined2014](https://github.com/Defined2014)
    - 同じ列に `DEFAULT` 列値が指定されている場合に `AUTO_INCREMENT` を指定できる問題を修正します [#45136](https://github.com/pingcap/tidb/issues/45136) @[Defined2014](https://github.com/Defined2014)
    - 一部の場合に、`INFORMATION_SCHEMA.TIKV_REGION_STATUS` システムテーブルの問い合わせ結果が正しくない問題を修正します [#45531](https://github.com/pingcap/tidb/issues/45531) @[Defined2014](https://github.com/Defined2014)
    - 一部の場合に不正確なエラーメッセージが発生する問題を修正します [#42273](https://github.com/pingcap/tidb/issues/42273) @[jiyfhust](https://github.com/jiyfhust)
    - パーティションテーブルでのグローバルインデックスがトランケートされた際にクリアされない問題を修正します [#42435](https://github.com/pingcap/tidb/issues/42435) @[L-maple](https://github.com/L-maple)
    - 1つの TiDB ノードでの障害後、他の TiDB ノードが TTL タスクを引き継がない問題を修正します [#45022](https://github.com/pingcap/tidb/issues/45022) @[lcwangchao](https://github.com/lcwangchao)
    - TTL 実行中にメモリリークが発生する問題を修正します [#45510](https://github.com/pingcap/tidb/issues/45510) @[lcwangchao](https://github.com/lcwangchao)
    - パーティションされたテーブルにデータを挿入する際の不正確なエラーメッセージの問題を修正します [#44966](https://github.com/pingcap/tidb/issues/44966) @[lilinghai](https://github.com/lilinghai)
    - `INFORMATION_SCHEMA.TIFLASH_REPLICA` テーブルの読み取り許可の問題を修正します [#7795](https://github.com/pingcap/tiflash/issues/7795) @[Lloyd-Pottiger](https://github.com/Lloyd-Pottiger)
    - 一部のケースで `tidb_enable_dist_task` が有効になっている場合に、インデックスの作成が停滞する問題を修正します [#44440](https://github.com/pingcap/tidb/issues/44440) @[tangenta](https://github.com/tangenta)
    - `AUTO_ID_CACHE=1` を使用してテーブルをリストアした際に `duplicate entry` エラーが発生する問題を修正します [#44716](https://github.com/pingcap/tidb/issues/44716) @[tiancaiamao](https://github.com/tiancaiamao)
- `TRUNCATE TABLE`の実行時間が`ADMIN SHOW DDL JOBS`で示されるタスク実行時間と一貫しない問題を修正します [#44785](https://github.com/pingcap/tidb/issues/44785) @[tangenta](https://github.com/tangenta)
    - TiDBのアップグレード中にメタデータの読み込みに1つのDDLリースよりも長い時間がかかるとTiDBのアップグレードが停止してしまう問題を修正します [#45176](https://github.com/pingcap/tidb/issues/45176) @[zimulala](https://github.com/zimulala)
    - `SELECT CAST(n AS CHAR)`ステートメントで`n`が負の数の場合、そのクエリ結果が正しくない問題を修正します [#44786](https://github.com/pingcap/tidb/issues/44786) @[xhebox](https://github.com/xhebox)
    - `tidb_opt_agg_push_down`が有効になっている場合に、クエリが正しくない結果を返す可能性がある問題を修正します [#44795](https://github.com/pingcap/tidb/issues/44795) @[AilinKid](https://github.com/AilinKid)
    - `current_date()`を使用するクエリがプランキャッシュを使用すると誤った結果が発生する問題を修正します [#45086](https://github.com/pingcap/tidb/issues/45086) @[qw4990](https://github.com/qw4990)

+ TiKV

    - GC中のデータの読み取りがTiKVを停止させる可能性があるまれなケースでパニックを引き起こす問題を修正します [#15109](https://github.com/tikv/tikv/issues/15109) @[MyonKeminta](https://github.com/MyonKeminta)

+ PD

    - PDの再起動が`default`リソースグループの再初期化を引き起こす可能性がある問題を修正します [#6787](https://github.com/tikv/pd/issues/6787) @[glorv](https://github.com/glorv)
    - etcdが既に起動しているがクライアントがまだそれに接続していない場合に、クライアントを呼び出すとPDがパニックを引き起こす可能性がある問題を修正します [#6860](https://github.com/tikv/pd/issues/6860) @[HuSharp](https://github.com/HuSharp)
    - Regionの`health-check`の出力がRegion IDをクエリで取得したRegion情報と一貫していない問題を修正します [#6560](https://github.com/tikv/pd/issues/6560) @[JmPotato](https://github.com/JmPotato)
    - `unsafe recovery`中の失敗したLearnerピアが`auto-detect`モードで無視される問題を修正します [#6690](https://github.com/tikv/pd/issues/6690) @[v01dstar](https://github.com/v01dstar)
    - プレースメントルールがルールに合致しないTiFlashのLearnerを選択する問題を修正します [#6662](https://github.com/tikv/pd/issues/6662) @[rleungx](https://github.com/rleungx)
    - ルールチェッカーが選択したピアが不健康な場合に削除できない問題を修正します [#6559](https://github.com/tikv/pd/issues/6559) @[nolouch](https://github.com/nolouch)

+ TiFlash

    - ロックが発生するため、TiFlashがパーティションされたテーブルを正常に複製できない問題を修正します [#7758](https://github.com/pingcap/tiflash/issues/7758) @[hongyunyan](https://github.com/hongyunyan)
    - `INFORMATION_SCHEMA.TIFLASH_REPLICA`システムテーブルにアクセス権限のないユーザが含まれる問題を修正します [#7795](https://github.com/pingcap/tiflash/issues/7795) @[Lloyd-Pottiger](https://github.com/Lloyd-Pottiger)
    - 同じMPPタスク内の複数のHashAgg演算子がある場合、MPPタスクのコンパイルに非常に長い時間がかかり、クエリのパフォーマンスが著しく低下する可能性がある問題を修正します [#7810](https://github.com/pingcap/tiflash/issues/7810) @[SeaRise](https://github.com/SeaRise)

+ Tools

    + TiCDC

        - `changefeed`が一時的にPDに利用できないために失敗する問題を修正します [#9294](https://github.com/pingcap/tiflow/issues/9294) @[asddongmen](https://github.com/asddongmen)
        - 一部のTiCDCノードがネットワークから切断された場合に発生する可能性があるデータ不整合問題を修正します [#9344](https://github.com/pingcap/tiflow/issues/9344) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - Kafka Sinkでエラーが発生した場合に、changefeedの進行が無期限でブロックされる可能性がある問題を修正します [#9309](https://github.com/pingcap/tiflow/issues/9309) @[hicqu](https://github.com/hicqu)
        - TiCDCノードの状態が変化した際に発生するパニック問題を修正します [#9354](https://github.com/pingcap/tiflow/issues/9354) @[sdojjy](https://github.com/sdojjy)
        - デフォルトの`ENUM`値のエンコードエラーを修正します [#9259](https://github.com/pingcap/tiflow/issues/9259) @[3AceShowHand](https://github.com/3AceShowHand)

    + TiDB Lightning

        - TiDB Lightningのインポート完了後にチェックサムを実行するとSSLエラーが発生する問題を修正します [#45462](https://github.com/pingcap/tidb/issues/45462) @[D3Hunter](https://github.com/D3Hunter)
        - 論理インポートモードでインポート中に下流のテーブルを削除するとTiDB Lightningのメタデータがタイムリーに更新されない問題を修正します [#44614](https://github.com/pingcap/tidb/issues/44614) @[dsdashun](https://github.com/dsdashun)

## 貢献者

TiDBコミュニティの以下の貢献者に感謝します:

- [charleszheng44](https://github.com/charleszheng44)
- [dhysum](https://github.com/dhysum)
- [haiyux](https://github.com/haiyux)
- [Jiang-Hua](https://github.com/Jiang-Hua)
- [Jille](https://github.com/Jille)
- [jiyfhust](https://github.com/jiyfhust)
- [krishnaduttPanchagnula](https://github.com/krishnaduttPanchagnula)
- [L-maple](https://github.com/L-maple)
- [pingandb](https://github.com/pingandb)
- [testwill](https://github.com/testwill)
- [tisonkun](https://github.com/tisonkun)
- [xuyifangreeneyes](https://github.com/xuyifangreeneyes)
- [yumchina](https://github.com/yumchina)