---
title: TiDB 3.0.7リリースノート
aliases: ['/docs/dev/releases/release-3.0.7/','/docs/dev/releases/3.0.7/']
---

# TiDB 3.0.7リリースノート

リリース日: 2019年12月4日

TiDBバージョン: 3.0.7

TiDB Ansibleバージョン: 3.0.7

## TiDB

- TiDBサーバーのローカル時間がPDのタイムスタンプよりも遅れているため、ロックTTLの値が大きすぎる問題を修正 [#13868](https://github.com/pingcap/tidb/pull/13868)
- `gotime.Local`を使用して文字列から日付を解析した後にタイムゾーンが正しくない問題を修正 [#13793](https://github.com/pingcap/tidb/pull/13793)
- `builtinIntervalRealSig`の実装において`binSearch`関数がエラーを返さないために結果が正しくない可能性がある問題を修正 [#13767](https://github.com/pingcap/tidb/pull/13767)
- 整数が符号なし浮動小数点数または10進数型に変換される際に精度が失われる問題があるためデータが正しくない問題を修正 [#13755](https://github.com/pingcap/tidb/pull/13755)
- Natural Outer JoinおよびOuter Joinで`USING`句が使用された際に`not null`フラグが適切にリセットされないために結果が正しくない問題を修正 [#13739](https://github.com/pingcap/tidb/pull/13739)
- 統計が更新される際にデータ競合が発生するため統計が正確でない問題を修正 [#13687](https://github.com/pingcap/tidb/pull/13687)

## TiKV

- デッドロックマネージャが有効なリージョン内にあることを確認するためにデッドロック検出器が有効なリージョンのみを観察するように修正 [#6110](https://github.com/tikv/tikv/pull/6110)
- 潜在的なメモリリーク問題を修正 [#6128](https://github.com/tikv/tikv/pull/6128)