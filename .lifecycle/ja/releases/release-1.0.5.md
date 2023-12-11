---
title: TiDB 1.0.5 リリースノート
aliases: ['/docs/dev/releases/release-1.0.5/','/docs/dev/releases/105/']
---

# TiDB 1.0.5 リリースノート

2017年12月26日、TiDB 1.0.5 が以下のアップデートとともにリリースされました：

## TiDB

- [`Show Create Table` ステートメントに現在のAuto_Increment IDの最大値を追加しました。](https://github.com/pingcap/tidb/pull/5489)
- 潜在的なゴルーチンリークを修正しました。
    - [5486](https://github.com/pingcap/tidb/pull/5486)
- 遅いクエリを別のファイルに出力する機能をサポートしました。
    - [5484](https://github.com/pingcap/tidb/pull/5484)
- 新しいセッションを作成する際にTiKVから`TimeZone`変数をロードします。
    - [5479](https://github.com/pingcap/tidb/pull/5479)
- スキーマステートチェックをサポートし、`Show Create Table`および`Analyze`ステートメントがパブリックなテーブル/インデックスのみ処理する機能をサポートしました。
    - [5474](https://github.com/pingcap/tidb/pull/5474)
- `set transaction read only`が`tx_read_only`変数に影響するようにしました。
    - [5491](https://github.com/pingcap/tidb/pull/5491)
- ロールバック時に増分統計データをクリーンアップするようにしました。
    - [5391](https://github.com/pingcap/tidb/pull/5391)
- `Show Create Table` ステートメントで欠落していたインデックス長の問題を修正しました。
    - [5421](https://github.com/pingcap/tidb/pull/5421)

## PD

- 指定の状況下でリーダーのバランスが停止する問題を修正しました。
    - [869](https://github.com/pingcap/pd/pull/869)
    - [874](https://github.com/pingcap/pd/pull/874)
- ブートストラップ中に潜在的なパニックを修正しました。
    - [889](https://github.com/pingcap/pd/pull/889)

## TiKV

- [`get_cpuid`](https://github.com/pingcap/tikv/pull/2611) 関数を使用してCPU IDを取得するのが遅い問題を修正しました。
- スペース収集状況を改善するために`dynamic-level-bytes`パラメータをサポートしました。
    - [2605](https://github.com/pingcap/tikv/pull/2605)

1.0.4 から 1.0.5 にアップグレードするには、PD -> TiKV -> TiDB のローリングアップグレードの順序に従ってください。