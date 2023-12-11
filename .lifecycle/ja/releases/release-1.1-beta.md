---
title: TiDB 1.1 ベータリリースノート
aliases: ['/docs/dev/releases/release-1.1-beta/','/docs/dev/releases/11beta/']
---

# TiDB 1.1 ベータリリースノート

2018年2月24日、TiDB 1.1 Betaがリリースされました。このリリースでは、MySQLとの互換性、SQLの最適化、安定性、およびパフォーマンスが大幅に向上しています。

## TiDB

- より多くの監視メトリクスを追加し、ログを精練化
- より多くのMySQL構文と互換性を持つ
- `information_schema`でテーブル作成時刻を表示する機能をサポート
- `MaxOneRow`演算子を含むクエリを最適化
- Joinによって生成された中間結果セットのサイズを構成して、Joinが使用するメモリをさらに削減
- 現在のTiDB構成を出力するための`tidb_config`セッション変数を追加
- `Union`および`Index Join`演算子でのパニック問題を修正
- 一部シナリオでの`Sort Merge Join`演算子の誤った結果の問題を修正
- 追加中のインデックスを表示する`Show Index`ステートメントが追加中のインデックスを表示する問題を修正
- `Drop Stats`ステートメントの失敗を修正
- SQLエンジンのクエリパフォーマンスを最適化し、Sysbench Select/OLTPのテスト結果を10%改善
- 新しい実行エンジンを使用して、最適化プロセスの副問い合わせの計算速度を向上させることで、TPC-HやTPC-DSなどのテストにおいて、TiDB 1.0と比較してTiDB 1.1 Betaが大幅な改善を遂げた

## PD

- リージョンのデバッグインターフェースを追加
- PDリーダーの優先度設定をサポート
- 特定のラベルを持つストアをRaftリーダーのスケジューリング対象としないように設定するサポートを追加
- 各PDのヘルスステータスを列挙するインターフェースを追加
- より多くのメトリクスを追加
- PDリーダーとetcdリーダーをできるだけ同じノードに維持
- TiKVがダウンした際のデータの優先度と速度を向上
- `data-dir`構成項目の有効性チェックを強化
- リージョンハートビートのパフォーマンスを最適化
- ホットスポットのスケジューリングがラベルの制約に違反する問題を修正
- その他の安定性の問題を修正

## TiKV

- 潜在的なGC問題を回避するために、オフセット+リミットを使用してロックを走査
- GCの速度を向上させるためにロックをバッチで解決するサポートを追加
- GCの速度を向上させるためにGC並行性をサポート
- より正確なPDスケジューリングのために、RocksDBコンパクションリスナーを使用してリージョンサイズを更新
- より速くTiKVを起動するために、`DeleteFilesInRanges`を使用して古いデータをバッチで削除
- 持続ファイルがあまり多くのスペースを取らないように、Raftスナップショットの最大サイズを構成
- `tikv-ctl`でのより多くのリカバリ操作をサポート
- 順序付けられたフロー集約操作の最適化
- メトリクスの改善およびバグの修正