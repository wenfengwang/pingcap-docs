---
title: TiDB 5.2.1リリースノート
---

# TiDB 5.2.1リリースノート

リリース日: 2021年9月9日

TiDBのバージョン: 5.2.1

## バグ修正

+ TiDB

    - 集計演算子をパーティション化されたテーブルにプッシュダウンする際にスキーマのカラムが浅いコピーされたことにより発生する実行中のエラーを修正しました。誤った実行計画によりエラーが発生します。[#27797](https://github.com/pingcap/tidb/issues/27797) [#26554](https://github.com/pingcap/tidb/issues/26554)

+ TiKV

    - リージョンの移行中にRaftstoreデッドロックによってTiKVが利用できなくなる問題を修正しました。問題の回避策としてスケジューリングを無効にして利用できないTiKVを再起動する必要があります。[#10909](https://github.com/tikv/tikv/issues/10909)