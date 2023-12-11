---
title: TiDB 3.0.13 リリースノート
aliases: ['/docs/dev/releases/release-3.0.13/','/docs/dev/releases/3.0.13/']
---

# TiDB 3.0.13 リリースノート

リリース日: 2020年4月22日

TiDB バージョン: 3.0.13

## バグ修正

+ TiDB

    - `MemBuffer` の未チェックの問題を修正しました。`INSERT ... ON DUPLICATE KEY UPDATE` ステートメントがトランザクション内で複数行の重複データを挿入する必要がある場合に誤って実行される可能性がある問題を修正しました。[#16690](https://github.com/pingcap/tidb/pull/16690)

+ TiKV

    - `Region Merge` が繰り返し実行されると、システムがスタックし、サービスが利用できなくなる可能性がある問題を修正しました。[#7612](https://github.com/tikv/tikv/pull/7612)