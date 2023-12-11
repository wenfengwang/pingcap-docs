---
title: TiDB 3.1.2 リリースノート
aliases: ['/docs/dev/releases/release-3.1.2/']
---

# TiDB 3.1.2 リリースノート

リリース日: 2020年6月4日

TiDB バージョン: 3.1.2

## バグ修正

+ TiKV

    - S3とGCSを使用したバックアップとリストア時のエラーハンドリングの問題を修正 [#7965](https://github.com/tikv/tikv/pull/7965)
    - リストア中に発生する `DefaultNotFound` エラーを修正 [#7838](https://github.com/tikv/tikv/pull/7938)

+ ツール

    - バックアップとリストア（BR）

        - ネットワークが不安定な場合に自動的にリトライし、S3とGCSストレージの安定性を向上 [#314](https://github.com/pingcap/br/pull/314) [#7965](https://github.com/tikv/tikv/pull/7965)
        - 小さなテーブルをリストアする際にリージョンのリーダーが見つからないために発生するリストアの失敗を修正 [#303](https://github.com/pingcap/br/pull/303)
        - テーブルの行IDが `2^(63)` を超える場合のリストア時のデータ損失の問題を修正 [#323](https://github.com/pingcap/br/pull/323)
        - 空のデータベースとテーブルをリストアできない問題を修正 [#318](https://github.com/pingcap/br/pull/318)
        - S3ストレージを対象とする際に、サーバーサイド暗号化（SSE）にAWS KMSを使用することをサポート [#261](https://github.com/pingcap/br/pull/261)