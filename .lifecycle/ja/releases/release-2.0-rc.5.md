---
title: TiDB 2.0 RC5リリースノート
aliases: ['/docs/dev/releases/release-2.0-rc.5/','/docs/dev/releases/2rc5/']
---

# TiDB 2.0 RC5リリースノート

2018年4月17日、TiDB 2.0 RC5がリリースされました。このリリースでは、MySQL互換性、SQLの最適化、および安定性が大幅に向上しています。

## TiDB

- `Top-N`プッシュダウンルールの適用に関する問題を修正
- NULL値を含む列の行数の推定を修正
- Binary型のゼロ値を修正
- トランザクション内の`BatchGet`の問題を修正
- `Add Index`操作をロールバックする際に書き込まれたデータをクリーンアップし、消費されるスペースを削減
- `insert on duplicate key update`ステートメントを最適化してパフォーマンスを10倍向上
- `UNIX_TIMESTAMP`関数によって返される結果のタイプに関する問題を修正
- NOT NULL列を追加する際にNULL値が挿入される問題を修正
- `Show Process List`ステートメントで実行中のステートメントのメモリ使用量を表示する機能をサポート
- `Alter Table Modify Column`が極端な条件下でエラーを報告する問題を修正
- `Alter`ステートメントを使用してテーブルコメントを設定するサポートを追加

## PD

- Raft Learnerのサポートを追加
- バランスリージョンスケジューラを最適化してスケジューリングオーバーヘッドを削減
- `schedule-limit`構成のデフォルト値を調整
- IDの割り当て頻度の問題を修正
- 新しいスケジューラを追加する際の互換性の問題を修正

## TiKV

- `tikv-ctl`で `compact`で指定されたリージョンのサポートを追加
- RawKVClientでのBatch Put、Batch Get、Batch Delete、Batch Scanのサポートを追加
- 多くのスナップショットによって引き起こされるOOMの問題を修正
- Coprocessorでより詳細なエラー情報を返す
- `tikv-ctl`を通じてTiKVの`block-cache-size`を動的に変更できるようにサポート
- `importer`をさらに改善
- `ImportSST::Upload`インターフェースを簡素化
- gRPCの`keepalive`プロパティを構成
- `tikv-importer`をTiKVから独立したバイナリとして分割
- Coprocessorにおいて各`scan range`によってスキャンされた行数の統計情報を提供
- macOSシステム上のコンパイルの問題を修正
- RocksDBメトリクスの誤用の問題を修正
- Coprocessorにおける`overflow as warning`オプションをサポート