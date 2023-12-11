---
title: TiDB 2.1.12 リリースノート
aliases: ['/docs/dev/releases/release-2.1.12/','/docs/dev/releases/2.1.12/']
---

# TiDB 2.1.12 リリースノート

リリース日: 2019年6月13日

TiDB バージョン: 2.1.12

TiDB Ansible バージョン: 2.1.12

## TiDB

- インデックスクエリフィードバック時のデータ型不一致による問題を修正 [#10755](https://github.com/pingcap/tidb/pull/10755)
- 文字セットの変更により blob 列がテキスト列に変更される問題を修正 [#10745](https://github.com/pingcap/tidb/pull/10745)
- トランザクションにおける `GRANT` 操作が誤って "Duplicate Entry" を報告する問題を修正 [#10739](https://github.com/pingcap/tidb/pull/10739)
- MySQL との互換性を向上
    - `DAYNAME` 関数 [#10732](https://github.com/pingcap/tidb/pull/10732)
    - `MONTHNAME` 関数 [#10733](https://github.com/pingcap/tidb/pull/10733)
    - `EXTRACT` 関数における `MONTH` 処理時の `0` 値のサポート [#10702](https://github.com/pingcap/tidb/pull/10702)
    - `DECIMAL` 型を `TIMESTAMP` または `DATETIME` に変換可能に [#10734](https://github.com/pingcap/tidb/pull/10734)
- テーブル文字セット変更時の列文字セットの変更 [#10714](https://github.com/pingcap/tidb/pull/10714)
- 浮動小数点から浮動小数点への変換時のオーバーフロー問題を修正 [#10730](https://github.com/pingcap/tidb/pull/10730)
- TiDB および TiKV による gRPC によるメッセージサイズの不一致による "grpc: received message larger than max" エラーが報告される問題を修正 [#10710](https://github.com/pingcap/tidb/pull/10710)
- `ORDER BY` における NULL のフィルタリング不具合を修正 [#10488](https://github.com/pingcap/tidb/pull/10488)
- 複数ノードが存在する場合に `UUID` 関数によって重複する可能性のある値を修正 [#10711](https://github.com/pingcap/tidb/pull/10711)
- `CAST(-num as datetime)` による返り値を `error` から NULL に変更 [#10703](https://github.com/pingcap/tidb/pull/10703)
- 符号なしヒストグラムが符号付き範囲に遭遇する問題を修正 [#10695](https://github.com/pingcap/tidb/pull/10695)
- 統計フィードバックが bigint unsigned 主キーに遭遇した際にデータ読み取りに対して誤ってエラーが報告される問題を修正 [#10307](https://github.com/pingcap/tidb/pull/10307)
- パーティションされたテーブルの `Show Create Table` 結果が一部の場合に正しく表示されない問題を修正 [#10690](https://github.com/pingcap/tidb/pull/10690)
- 相関副問い合わせにおける `GROUP_CONCAT` 集約関数の計算結果が一部の場合に正しくない問題を修正 [#10670](https://github.com/pingcap/tidb/pull/10670)
- スロークエリログを解析する際にメモリテーブルが遅いクエリの解析結果を誤って表示する問題を修正 [#10776](https://github.com/pingcap/tidb/pull/10776)

## PD

- 極端な条件下で etcd リーダー選出がブロックされる問題を修正 [#1576](https://github.com/pingcap/pd/pull/1576)

## TiKV

- 極端な状況下でリーダー転送プロセス中にリージョンが利用できなくなる問題を修正 [#4799](https://github.com/tikv/tikv/pull/4734)
- 異常なシャットダウン時に TiKV がデータを失う問題を修正 [#4850](https://github.com/tikv/tikv/pull/4850)