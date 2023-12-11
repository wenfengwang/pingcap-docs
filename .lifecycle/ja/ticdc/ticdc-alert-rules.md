---
title: TiCDCアラートルール
summary: TiCDCのアラートルールとその対処方法について学びます。
---
# TiCDCアラートルール

このドキュメントではTiCDCのアラートルールとそれに対応する解決策について説明します。重要度の降順に、深刻度レベルは次のとおりです: **Critical**、**Warning**。

## Critical alerts

このセクションでは重大なアラートとその解決策について紹介します。

### `cdc_checkpoint_high_delay`

重大なアラートでは、異常な監視メトリクスに注意を払う必要があります。

- アラートルール：

    (time() - ticdc_owner_checkpoint_ts / 1000) > 600

- 説明：

    レプリケーションタスクの遅延が10分以上になっています。

- 解決策：

    [TiCDC レプリケーションの障害を処理する](/ticdc/troubleshoot-ticdc.md#how-do-i-handle-replication-interruptions)を参照してください。

### `cdc_resolvedts_high_delay`

- アラートルール：

    (time() - ticdc_owner_resolved_ts / 1000) > 300

- 説明：

     レプリケーションタスクのResolved TSが5分以上遅延しています。

- 解決策：

    [TiCDC レプリケーションの障害を処理する](/ticdc/troubleshoot-ticdc.md#how-do-i-handle-replication-interruptions)を参照してください。

### `ticdc_processor_exit_with_error_count`

- アラートルール：

    `changes(ticdc_processor_exit_with_error_count[1m]) > 0`

- 説明：

    レプリケーションタスクでエラーが報告され、終了しています。

- 解決策：

    [TiCDC レプリケーションの障害を処理する](/ticdc/troubleshoot-ticdc.md#how-do-i-handle-replication-interruptions)を参照してください。

## Warning alerts

警告アラートは問題やエラーのリマインダーです。

### `cdc_multiple_owners`

- アラートルール：

    `sum(rate(ticdc_owner_ownership_counter[30s])) >= 2`

- 説明：

    TiCDCクラスタに複数の所有者がいます。

- 解決策：

    TiCDCログを収集して原因を特定してください。

### `cdc_sink_flush_duration_time_more_than_10s`

- アラートルール：

    `histogram_quantile(0.9, rate(ticdc_sink_txn_worker_flush_duration[1m])) > 10`

- 説明：

    レプリケーションタスクが10秒以上かかって下流データベースにデータを書き込んでいます。

- 解決策：

    下流データベースに問題があるかどうかを確認してください。

### `cdc_processor_checkpoint_tso_no_change_for_1m`

- アラートルール：

    `changes(ticdc_processor_checkpoint_ts[1m]) < 1`

- 説明：

    レプリケーションタスクが1分以上進んでいません。

- 解決策：

    [TiCDC レプリケーションの障害を処理する](/ticdc/troubleshoot-ticdc.md#how-do-i-handle-replication-interruptions)を参照してください。

### `ticdc_puller_entry_sorter_sort_bucket`

- アラートルール：

    `histogram_quantile(0.9, rate(ticdc_puller_entry_sorter_sort_bucket{}[1m])) > 1`

- 説明：

    TiCDCプラーヤーエントリーソーターの遅延が高すぎます。

- 解決策：

    TiCDCログを収集して原因を特定してください。

### `ticdc_puller_entry_sorter_merge_bucket`

- アラートルール：

    `histogram_quantile(0.9, rate(ticdc_puller_entry_sorter_merge_bucket{}[1m])) > 1`

- 説明：

    TiCDCプラーヤーエントリーソーターマージの遅延が高すぎます。

- 解決策：

    TiCDCログを収集して原因を特定してください。

### `tikv_cdc_min_resolved_ts_no_change_for_1m`

- アラートルール：

    `changes(tikv_cdc_min_resolved_ts[1m]) < 1 and ON (instance) tikv_cdc_region_resolve_status{status="resolved"} > 0`

- 説明：

    TiKV CDCの最小Resolved TSが1分以上進んでいません。

- 解決策：

    TiKVログを収集して原因を特定してください。

### `tikv_cdc_scan_duration_seconds_more_than_10min`

- アラートルール：

    `histogram_quantile(0.9, rate(tikv_cdc_scan_duration_seconds_bucket{}[1m])) > 600`

- 説明：

    TiKV CDCモジュールが増分レプリケーションを10分以上スキャンしています。

- 解決策：

    TiCDCの監視メトリクスとTiKVログを収集して原因を特定してください。

### `ticdc_sink_mysql_execution_error`

- アラートルール：

    `changes(ticdc_sink_mysql_execution_error[1m]) > 0`

- 説明：

    レプリケーションタスクが下流のMySQLにデータを書き込む際にエラーが発生しています。

- 解決策：

    多くの可能性が考えられます。[TiCDCのトラブルシューティング](/ticdc/troubleshoot-ticdc.md)を参照してください。

### `ticdc_memory_abnormal`

- アラートルール：

    `go_memstats_heap_alloc_bytes{job="ticdc"} > 1e+10`

- 説明：

    TiCDCのヒープメモリ使用量が10 GiBを超えています。

- 解決策：

    TiCDCログを収集して原因を特定してください。