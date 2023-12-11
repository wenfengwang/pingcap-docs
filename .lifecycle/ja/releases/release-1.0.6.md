---
title: TiDB 1.0.6 リリースノート
aliases: ['/docs/dev/releases/release-1.0.6/','/docs/dev/releases/106/']
---

# TiDB 1.0.6 リリースノート

2018年1月8日、以下の更新内容を含むTiDB 1.0.6がリリースされました。

## TiDB

- [`Alter Table Auto_Increment`構文のサポート](https://github.com/pingcap/tidb/pull/5511)
- [コストベースの計算および統計情報の`Null Json`の問題を修正](https://github.com/pingcap/tidb/pull/5556)
- [単一のテーブルのホットスポット書き込みを避けるために暗黙の行IDをシャードするための拡張構文のサポート](https://github.com/pingcap/tidb/pull/5559)
- [潜在的なDDLの問題を修正](https://github.com/pingcap/tidb/pull/5562)
- [`curtime`、`sysdate`、`curdate`関数におけるタイムゾーン設定の考慮](https://github.com/pingcap/tidb/pull/5564)
- `GROUP_CONCAT`関数における`SEPARATOR`構文のサポート](https://github.com/pingcap/tidb/pull/5569)
- `GROUP_CONCAT`関数の間違った戻り型の問題を修正](https://github.com/pingcap/tidb/pull/5582)

## PD

- [ホットリージョンスケジューラのストア選択問題を修正](https://github.com/pingcap/pd/pull/898)

## TiKV

なし。

1.0.5から1.0.6にアップグレードするには、PD -> TiKV -> TiDBのローリングアップグレード順に従ってください。