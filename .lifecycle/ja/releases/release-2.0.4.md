---
title: TiDB 2.0.4 リリースノート
aliases: ['/docs/dev/releases/release-2.0.4/','/docs/dev/releases/204/']
---

# TiDB 2.0.4 リリースノート

2018年6月15日にTiDB 2.0.4がリリースされました。TiDB 2.0.3と比較して、このリリースではシステムの互換性と安定性が大幅に向上しています。

## TiDB

- `ALTER TABLE t DROP COLUMN a CASCADE` 構文のサポート
- `tidb_snapshot` の値を TSO に設定するサポート
- モニタリング項目でのステートメントタイプの表示を精査
- クエリコスト推定の精度を最適化
- gRPC の `backoff max delay` パラメータの構成
- 構成ファイルで単一ステートメントのメモリ閾値を設定するサポート
- Optimizerのエラーのリファクタリング
- `Cast Decimal` データの副作用を修正
- 特定のシナリオでの `Merge Join` オペレータの誤った結果の問題を修正
- NullオブジェクトをStringに変換する問題を修正
- データのJSON型をJSON型にキャストする問題を修正
- `Union` + `OrderBy` 条件でMySQLと結果順序が一貫しない問題を修正
- `Union`ステートメントが`Limit/OrderBy`句をチェックする際の整合性ルールの問題を修正
- `Union All` 結果の互換性の問題を修正
- プレディケートプッシュダウンのバグを修正
- `For Update` 句を持つ`Union`ステートメントの互換性の問題を修正
- `concat_ws` 関数が誤って結果を切り捨てる問題を修正

## PD

- 設定を解除する `max-pending-peer-count` の挙動を改善し、`PendingPeer` の最大数を制限しないように変更

## TiKV

- デバッグ向けのRocksDB `PerfContext` インタフェースの追加
- `import-mode` パラメータの削除
- `tikv-ctl` 用の `region-properties` コマンドの追加
- 多くのRocksDB墓碑石が存在すると`reverse-seek` が遅い問題を修正
- `do_sub` によって引き起こされるクラッシュの問題を修正
- 多くのバージョンのデータに遭遇した場合のGCログの記録