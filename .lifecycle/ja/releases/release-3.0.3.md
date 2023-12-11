---
title: TiDB 3.0.3 リリースノート
aliases: ['/docs/dev/releases/release-3.0.3/','/docs/dev/releases/3.0.3/']
---

# TiDB 3.0.3 リリースノート

リリース日: 2019年8月29日

TiDB バージョン: 3.0.3

TiDB Ansible バージョン: 3.0.3

## TiDB

+ SQL オプティマイザー
    - `opt_rule_blacklist` テーブルを追加して、`aggregation_eliminate` や `column_prune` などの論理最適化ルールを無効にする [#11658](https://github.com/pingcap/tidb/pull/11658)
    - 接頭辞インデックスや符号なしインデックス列が負の値に等しい場合、`Index Join` で誤った結果が返される可能性のある問題を修正 [#11759](https://github.com/pingcap/tidb/pull/11759)
    - `create … binding ...` の `SELECT` 文で `”` や `\` が含まれると解析エラーが発生する可能性のある問題を修正 [#11726](https://github.com/pingcap/tidb/pull/11726)
+ SQL 実行エンジン
    - Quote 関数がヌル値を処理する際に型エラーが発生する可能性がある問題を修正 [#11619](https://github.com/pingcap/tidb/pull/11619)
    - `ifnull` で `NotNullFlag` を保持した状態で Max/Min を使用すると、誤った結果が返される可能性のある問題を修正 [#11641](https://github.com/pingcap/tidb/pull/11641)
    - 文字列形式のビット型データを比較する際に発生する可能性のあるエラーを修正 [#11660](https://github.com/pingcap/tidb/pull/11660)
    - 逐次読み取りが必要なデータの並行性を減少させて、OOM の可能性を低減するための修正 [#11679](https://github.com/pingcap/tidb/pull/11679)
    - 複数のパラメーターが符号なしの場合にいくつかの組み込み関数（たとえば `if` や `coalesce`）で誤った型推論が引き起こされる可能性のある問題を修正 [#11621](https://github.com/pingcap/tidb/pull/11621)
    - `Div` 関数が符号なしの十進数型を処理する際に MySQL との非互換性を修正 [#11813](https://github.com/pingcap/tidb/pull/11813)
    - Pump/Drainer の状態を変更する SQL 文を実行する際に panic が発生する可能性のある問題を修正 [#11827](https://github.com/pingcap/tidb/pull/11827)
    - Autocommit = 1 で `begin` 文がない場合に `select ... for update` で panic が発生する可能性のある問題を修正 [#11736](https://github.com/pingcap/tidb/pull/11736)
    - `set default role` 文を実行する際に発生する許可チェックエラーを修正 [#11777](https://github.com/pingcap/tidb/pull/11777)
    - `create user` や `drop user` を実行する際に発生する許可チェックエラーを修正 [#11814](https://github.com/pingcap/tidb/pull/11814)
    - `select ... for update` 文を `PointGetExecutor` 関数に構築した際に自動的にリトライされる可能性のある問題を修正 [#11718](https://github.com/pingcap/tidb/pull/11718)
    - ウィンドウ関数がパーティションを処理する際に発生する可能性のある境界エラーを修正 [#11825](https://github.com/pingcap/tidb/pull/11825)
    - 不正な書式の引数を処理する際に `time` 関数が EOF エラーを発生させる可能性のある問題を修正 [#11893](https://github.com/pingcap/tidb/pull/11893)
    - ウィンドウ関数が渡されたパラメーターをチェックしない可能性のある問題を修正 [#11705](https://github.com/pingcap/tidb/pull/11705)
    - `Explain` で表示される計画結果が実際に実行された結果と一貫しない可能性のある問題を修正 [#11186](https://github.com/pingcap/tidb/pull/11186)