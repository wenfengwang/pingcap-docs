---
title: TiDB 2.0.11 リリースノート
aliases: ['/docs/dev/releases/release-2.0.11/', '/docs/dev/releases/2.0.11/']
---

# TiDB 2.0.11 リリースノート

2019年1月3日に、TiDB 2.0.11 がリリースされました。対応する TiDB Ansible 2.0.11 もリリースされています。このリリースは、TiDB 2.0.10 に比べてシステムの互換性と安定性が大幅に向上しています。

## TiDB

- PD が異常な状態にあるときにエラーが適切に処理されない問題を修正しました [#8764](https://github.com/pingcap/tidb/pull/8764)
- TiDB でのテーブルの `Rename` 操作が MySQL と互換性がない問題を修正しました [#8809](https://github.com/pingcap/tidb/pull/8809)
- `ADMIN CHECK TABLE` 操作を実行中に `ADD INDEX` 文を実行する際に誤ってエラーメッセージが報告される問題を修正しました [#8750](https://github.com/pingcap/tidb/pull/8750)
- 特定のケースでプレフィックスインデックスの範囲が正しくない問題を修正しました [#8877](https://github.com/pingcap/tidb/pull/8877)
- 特定のケースで列が追加された場合の `UPDATE` 文のパニック問題を修正しました [#8904](https://github.com/pingcap/tidb/pull/8904)

## TiKV

- Region マージに関する2つの問題を修正しました [#4003](https://github.com/tikv/tikv/pull/4003), [#4004](https://github.com/tikv/tikv/pull/4004)