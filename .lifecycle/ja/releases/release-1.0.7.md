---
title: TiDB 1.0.7リリースノート
aliases: ['/docs/dev/releases/release-1.0.7/', '/docs/dev/releases/107/']
---

# TiDB 1.0.7リリースノート

2018年1月22日、TiDB 1.0.7が以下のアップデートとともにリリースされました:

## TiDB

- [`FIELD_LIST`コマンドを最適化](https://github.com/pingcap/tidb/pull/5679)
- 情報スキーマのデータ競合を修正](https://github.com/pingcap/tidb/pull/5676)
- 履歴に読み取り専用ステートメントを追加しないように修正](https://github.com/pingcap/tidb/pull/5661)
- クエリログを制御するために`session`変数を追加](https://github.com/pingcap/tidb/pull/5659)
- 統計情報のリソースリークの問題を修正](https://github.com/pingcap/tidb/pull/5657)
- ゴルーチンのリーク問題を修正](https://github.com/pingcap/tidb/pull/5624)
- HTTPステータスサーバーのためのスキーマ情報APIを追加](https://github.com/pingcap/tidb/pull/5256)
- `IndexJoin`に関する問題を修正](https://github.com/pingcap/tidb/pull/5623)
- DDLの`RunWorker`がfalseの場合の挙動を更新](https://github.com/pingcap/tidb/pull/5604)
- 統計情報のテスト結果の安定性を向上](https://github.com/pingcap/tidb/pull/5609)
- `CREATE TABLE`ステートメントの`PACK_KEYS`構文をサポート](https://github.com/pingcap/tidb/pull/5602)
- パフォーマンスを最適化するために、ヌルプッシュダウンスキーマに`row_id`列を追加](https://github.com/pingcap/tidb/pull/5447)

## PD

- 異常な状況でのスケジューリング損失の可能性を修正](https://github.com/pingcap/pd/pull/921)
- proto3との互換性の問題を修正](https://github.com/pingcap/pd/pull/919)
- ログを追加](https://github.com/pingcap/pd/pull/917)

## TiKV

- `Table Scan`をサポート](https://github.com/pingcap/tikv/pull/2657)
- `tikv-ctl`でのリモートモードをサポート](https://github.com/pingcap/tikv/pull/2377)
- tikv-ctl protoのフォーマットの互換性の問題を修正](https://github.com/pingcap/tikv/pull/2668)
- PDからのスケジューリングコマンドの損失を修正](https://github.com/pingcap/tikv/pull/2669)
- Pushメトリクスにタイムアウトを追加](https://github.com/pingcap/tikv/pull/2686)

1.0.6から1.0.7にアップグレードするには、PD -> TiKV -> TiDBの順でローリングアップグレードを行ってください。