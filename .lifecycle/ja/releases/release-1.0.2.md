---
title: TiDB 1.0.2 リリースノート
aliases: ['/docs/dev/releases/release-1.0.2/','/docs/dev/releases/102/']
---

# TiDB 1.0.2 リリースノート

2017年11月13日、TiDB 1.0.2 が次のアップデートと共にリリースされました:

## TiDB

- インデックスポイントクエリのコスト見積もりを最適化
- `Alter Table Add Column (ColumnDef ColumnPosition)` 構文をサポート
- `where` 条件が矛盾しているクエリを最適化
- `Add Index` 操作を最適化して進行状況を正確にし、繰り返し操作を削減
- `Index Look Join` 演算子を最適化して、小規模データのクエリ速度を加速
- プレフィックスインデックスの判定に関する問題を修正

## プレイスメントドライバー (PD)

- 例外的な状況下でのスケジューリングの安定性を向上

## TiKV

- テーブルの分割をサポートし、1つのリージョンが複数のテーブルのデータを含まないようにする
- キーの長さを4KBを超えないように制限
- より正確な読み取りトラフィック統計
- 共同処理スタックに深い保護を実装
- `LIKE`の動作と`do_div_mod`のバグを修正