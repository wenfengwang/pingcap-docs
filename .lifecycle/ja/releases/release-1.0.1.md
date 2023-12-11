---
title: TiDB 1.0.1 リリースノート
aliases: ['/docs/dev/releases/release-1.0.1/', '/docs/dev/releases/101/']
---

# TiDB 1.0.1 リリースノート

2017年11月1日にTiDB 1.0.1がリリースされ、以下の更新が行われました：

## TiDB

- DDLジョブのキャンセルをサポート
- `IN`式の最適化
- `SHOW`ステートメントの結果型を修正
- 遅いクエリを別のログファイルに記録するサポート
- バグの修正

## TiKV

- 書き込みバイトでのフロー制御をサポート
- Raftの割り当てを削減
- コプロセッサのスタックサイズを10MBに増やす
- コプロセッサから無用なログを削除