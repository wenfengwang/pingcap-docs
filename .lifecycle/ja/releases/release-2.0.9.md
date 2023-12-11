---
title: TiDB 2.0.9のリリースノート
aliases: ['/docs/dev/releases/release-2.0.9/','/docs/dev/releases/209/']
---

# TiDB 2.0.9のリリースノート

2018年11月19日、TiDB 2.0.9がリリースされました。TiDB 2.0.8と比較して、このリリースではシステムの互換性と安定性に大きな改善があります。

## TiDB

- 空の統計ヒストグラムによって引き起こされた問題を修正しました [#7927](https://github.com/pingcap/tidb/pull/7927)
- 一部のケースでの `UNION ALL` ステートメントのパニック問題を修正しました [#7942](https://github.com/pingcap/tidb/pull/7942)
- 誤ったDDLジョブによって引き起こされたスタックオーバーフローの問題を修正しました [#7959](https://github.com/pingcap/tidb/pull/7959)
- `Commit` 操作のためのスローログを追加しました [#7983](https://github.com/pingcap/tidb/pull/7983)
- 大きすぎる `Limit` 値によって引き起こされたパニックの問題を修正しました [#8004](https://github.com/pingcap/tidb/pull/8004)
- `USING` 句で `utf8mb4` 文字セットを指定するサポートを追加しました [#8048](https://github.com/pingcap/tidb/pull/8048)
- `TRUNCATE` 組み込み関数が符号なし整数型のパラメータをサポートするようにしました [#8069](https://github.com/pingcap/tidb/pull/8069)
- 統計モジュールのプライマリキーの選択性推定の問題を一部のケースで修正しました [#8150](https://github.com/pingcap/tidb/pull/8150)
- `_tidb_rowid` が書き込み可能であるかどうかを制御するための `Session` 変数を追加しました [#8126](https://github.com/pingcap/tidb/pull/8126)
- 一部のケースでの `PhysicalProjection` のパニック問題を修正しました [#8154](https://github.com/pingcap/tidb/pull/8154)
- 一部のケースでの `Union` ステートメントの不安定な結果を修正しました [#8168](https://github.com/pingcap/tidb/pull/8168)
- 非 `Insert` ステートメントにおいて `NULL` が `values` によって返されない問題を修正しました [#8179](https://github.com/pingcap/tidb/pull/8179)
- 一部のケースで統計モジュールが古いデータをクリアできない問題を修正しました [#8184](https://github.com/pingcap/tidb/pull/8184)
- トランザクションの最大実行時間を構成可能なオプションにしました [#8209](https://github.com/pingcap/tidb/pull/8209)
- 一部のケースでの `expression rewriter` の誤った比較アルゴリズムを修正しました [#8288](https://github.com/pingcap/tidb/pull/8288)
- `UNION ORDER BY` ステートメントによって生成される余分な列を除外しました [#8307](https://github.com/pingcap/tidb/pull/8307)
- `admin show next_row_id` ステートメントをサポートしました [#8274](https://github.com/pingcap/tidb/pull/8274)
- `Show Create Table` ステートメントにおける特殊文字のエスケープの問題を修正しました [#8321](https://github.com/pingcap/tidb/pull/8321)
- 一部のケースでの `UNION` ステートメントにおける予期しないエラーを修正しました [#8318](https://github.com/pingcap/tidb/pull/8318)
- DDLジョブのキャンセルによって一部のケースでスキーマのロールバックが行われない問題を修正しました [#8312](https://github.com/pingcap/tidb/pull/8312)
- `tidb_max_chunk_size` をグローバル変数に変更しました [#8333](https://github.com/pingcap/tidb/pull/8333)
- ticlientの `Scan` コマンドに `end-key` 制限を追加し、オーバーバウンドスキャンを避けます [#8309](https://github.com/pingcap/tidb/pull/8309) [#8310](https://github.com/pingcap/tidb/pull/8310)

## PD

- etcdの起動に失敗した場合、PDサーバーがスタックする問題を修正しました [#1267](https://github.com/pingcap/pd/pull/1267)
- `pd-ctl` がリージョンキーを読み取る関連する問題を修正しました [#1298](https://github.com/pingcap/pd/pull/1298) [#1299](https://github.com/pingcap/pd/pull/1299) [#1308](https://github.com/pingcap/pd/pull/1308)
- `regions/check` APIが誤った結果を返す問題を修正しました [#1311](https://github.com/pingcap/pd/pull/1311)
- PDがPDの参加に失敗した後に再起動できない問題を修正しました [#1279](https://github.com/pingcap/pd/pull/1279)

## TiKV

- `kv_scan` インタフェースに `end-key` 制限を追加しました [#3749](https://github.com/tikv/tikv/pull/3749)
- `max-tasks-xxx` 構成を廃止し、`max-tasks-per-worker-xxx` を追加しました [#3093](https://github.com/tikv/tikv/pull/3093)
- RocksDBにおける `CompactFiles` の問題を修正しました [#3789](https://github.com/tikv/tikv/pull/3789)