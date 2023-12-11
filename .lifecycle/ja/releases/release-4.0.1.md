---
title: TiDB 4.0.1リリースノート
aliases: ['/docs/dev/releases/release-4.0.1/']
---

# TiDB 4.0.1リリースノート

リリース日: 2020年6月12日

TiDBバージョン: 4.0.1

## 新機能

+ TiKV

    - `--advertise-status-addr`スタートフラグを追加し、ステータスアドレスを広告する[#8046](https://github.com/tikv/tikv/pull/8046)

+ PD

    - 組み込みTiDBダッシュボード用の内部プロキシをサポート[#2511](https://github.com/pingcap/pd/pull/2511)
    - PDクライアントのカスタムタイムアウトの設定をサポート[#2509](https://github.com/pingcap/pd/pull/2509)

+ TiFlash

    - TiDB新しい照合フレームワークをサポート
    - TiFlashへの`If`/`BitAnd/BitOr`/`BitXor/BitNot`/`Json_length`関数の押し下げをサポート
    - TiFlashにおける大規模トランザクションのResolve Lockロジックをサポート

+ Tools

    - バックアップ＆リストア（BR）

        - BRを開始する際にバージョンチェックを追加し、BRとTiDBクラスタが互換性のない問題を回避[#311](https://github.com/pingcap/br/pull/311)

## バグ修正

+ TiKV

    - 起動ログにおける`use-unified-pool`構成が誤って出力される問題を修正[#7946](https://github.com/tikv/tikv/pull/7946)
    - tikv-ctlが相対パスをサポートしていない問題を修正[#7963](https://github.com/tikv/tikv/pull/7963)
    - Point Selectsの監視メトリクスが不正確である問題を修正[#8033](https://github.com/tikv/tikv/pull/8033)
    - ネットワーク隔離が解消された後も、ピアが破棄されない可能性がある問題を修正[#8006](https://github.com/tikv/tikv/pull/8006)
    - 読み取りインデックスのリクエストが古いコミットインデックスになる可能性がある問題を修正[#8043](https://github.com/tikv/tikv/pull/8043)
    - S3およびGCSストレージを使用したバックアップとリストアの信頼性を向上させる[#7917](https://github.com/tikv/tikv/pull/7917)

+ PD

    - 特定の状況下でPlacement Rulesの誤設定を防止する[#2516](https://github.com/pingcap/pd/pull/2516)
    - Placement Ruleの削除がパニックを引き起こす可能性がある問題を修正[#2515](https://github.com/pingcap/pd/pull/2515)
    - 使用されているサイズがゼロの場合、ストア情報を取得できない問題を修正[#2474](https://github.com/pingcap/pd/pull/2474)

+ TiFlash

    - TiFlashにおける`bit`型列のデフォルト値が誤って解析される問題を修正
    - TiFlashにおける一部のタイムゾーンにおける`1970-01-01 00:00:00 UTC`の誤計算を修正