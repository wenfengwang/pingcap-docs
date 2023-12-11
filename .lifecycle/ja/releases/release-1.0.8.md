---
title: TiDB 1.0.8 リリースノート
aliases: ['/docs/dev/releases/release-1.0.8/','/docs/dev/releases/108/']
---

# TiDB 1.0.8 リリースノート

2018年2月11日、TiDB 1.0.8 が以下のアップデートを含んでリリースされました。

## TiDB

- [いくつかのシナリオにおける `Outer Join` 結果の問題を修正](https://github.com/pingcap/tidb/pull/5712)
- `InsertIntoIgnore` ステートメントのパフォーマンスを最適化](https://github.com/pingcap/tidb/pull/5738)
- `ShardRowID` オプションの問題を修正](https://github.com/pingcap/tidb/pull/5751)
- 1 つのトランザクション内の DML ステートメント数に制限を追加（設定可能で、デフォルト値は 5000）](https://github.com/pingcap/tidb/pull/5754)
- `Prepare` ステートメントによって返される Table/Column 別名の問題を修正](https://github.com/pingcap/tidb/pull/5776)
- 統計デルタの更新に関する問題を修正](https://github.com/pingcap/tidb/pull/5787)
- `Drop Column` ステートメントにおける panic エラーを修正](https://github.com/pingcap/tidb/pull/5805)
- `Add Column After` ステートメントの実行時に発生する DML の問題を修正](https://github.com/pingcap/tidb/pull/5818)
- GC プロセスの安定性を向上させ、GC エラーが発生したリージョンを無視するように改善](https://github.com/pingcap/tidb/pull/5815)
- GC プロセスを高速化するために GC を同時に実行する](https://github.com/pingcap/tidb/pull/5850)
- `CREATE INDEX` ステートメントの構文サポートを提供](https://github.com/pingcap/tidb/pull/5853)

## PD

- リージョンのハートビートのロック過熱を軽減](https://github.com/pingcap/pd/pull/932)
- ホットリージョンスケジューラが間違ったリーダーを選択する問題を修正](https://github.com/pingcap/pd/pull/939)

## TiKV

- 旧データをクリアし、TiKV の起動速度を向上させるために `DeleteFilesInRanges` を使用](https://github.com/pingcap/tikv/pull/2740)
- Coprocessor sum で `Decimal` を使用](https://github.com/pingcap/tikv/pull/2754)
- 受信したスナップショットのメタデータを強制的に同期化し、その安全性を確保](https://github.com/pingcap/tikv/pull/2758)

1.0.7 から 1.0.8 にアップグレードするには、PD -> TiKV -> TiDB のローリングアップグレード順に従ってください。