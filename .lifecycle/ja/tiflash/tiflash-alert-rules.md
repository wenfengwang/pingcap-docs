---
title: TiFlashのアラートルール
summary: TiFlashクラスターのアラートルールを学ぶ
aliases: ['/docs/dev/tiflash/tiflash-alert-rules/','/docs/dev/reference/tiflash/alert-rules/']
---

# TiFlashのアラートルール

このドキュメントでは、TiFlashクラスターのアラートルールについて紹介します。

## `TiFlash_schema_error`

- アラートルール:

    `increase(tiflash_schema_apply_count{type="failed"}[15m]) > 0`

- 説明:

    スキーマ適用エラーが発生した場合、アラートがトリガーされます。

- 解決策:

    エラーは間違ったロジックによるものかもしれません。PingCAPかコミュニティから[サポートを受ける](/support.md)ことができます。

## `TiFlash_schema_apply_duration`

- アラートルール:

    `histogram_quantile(0.99, sum(rate(tiflash_schema_apply_duration_seconds_bucket[1m])) BY (le, instance)) > 20`

- 説明:

    適用期間が20秒を超える確率が99％を超える場合、アラートがトリガーされます。

- 解決策:

    TiFlashストレージエンジンの内部問題による可能性があります。PingCAPかコミュニティから[サポートを受ける](/support.md)ことができます。

## `TiFlash_raft_read_index_duration`

- アラートルール:

    `histogram_quantile(0.99, sum(rate(tiflash_raft_read_index_duration_seconds_bucket[1m])) BY (le, instance)) > 3`

- 説明:

    読み取りインデックスの期間が3秒を超える確率が99％を超える場合、アラートがトリガーされます。

    > **注意:**
    >
    > `read index` とは、TiKVリーダーに送信されるkvprotoリクエストです。TiKVリージョンのリトライ、ビジーストア、またはネットワークの問題が`read index`の長いリクエスト時間を引き起こす可能性があります。

- 解決策:

    頻繁なリトライは、TiKVクラスターの頻繁な分割や移行による可能性があります。リトライの理由を特定するためにTiKVクラスターの状態をチェックできます。

## `TiFlash_raft_wait_index_duration`

- アラートルール:

    `histogram_quantile(0.99, sum(rate(tiflash_raft_wait_index_duration_seconds_bucket[1m])) BY (le, instance)) > 2`

- 説明:

    TiFlashでのリージョンRaftインデックスの待機時間が2秒を超える確率が99％を超える場合、アラートがトリガーされます。

- 解決策:

    TiKVとプロキシ間の通信エラーが原因である可能性があります。PingCAPかコミュニティから[サポートを受ける](/support.md)ことができます。