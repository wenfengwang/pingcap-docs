---
title: TiDB 2.0.10 リリースノート
aliases: ['/docs/dev/releases/release-2.0.10/','/docs/dev/releases/2.0.10/']
---

# TiDB 2.0.10 リリースノート

2018年12月18日、TiDB 2.0.10 がリリースされました。対応する TiDB Ansible 2.0.10 もリリースされました。このリリースは、TiDB 2.0.9 と比較して、システムの互換性と安定性に大きな改善があります。

## TiDB

- DDL ジョブをキャンセルすることによって引き起こされる可能性のある問題を修正しました [#8513](https://github.com/pingcap/tidb/pull/8513)
- `ORDER BY` および `UNION` 句でテーブル名を含む列をクォートできない問題を修正しました [#8514](https://github.com/pingcap/tidb/pull/8514)
- `UNCOMPRESS` 関数が誤った入力長を判断しない問題を修正しました [#8607](https://github.com/pingcap/tidb/pull/8607)
- TiDB をアップグレードする際に発生する `ANSI_QUOTES SQL_MODE` に遭遇した問題を修正しました [#8575](https://github.com/pingcap/tidb/pull/8575)
- 一部のケースで `select` が誤った結果を返す問題を修正しました [#8570](https://github.com/pingcap/tidb/pull/8570)
- TiDB が終了シグナルを受信した際に終了できない可能性のある問題を修正しました [#8501](https://github.com/pingcap/tidb/pull/8501)
- 一部のケースで `IndexLookUpJoin` が誤った結果を返す問題を修正しました [#8508](https://github.com/pingcap/tidb/pull/8508)
- `GetVar` または `SetVar` を含むフィルタをプッシュダウンするのを避けるようにしました [#8454](https://github.com/pingcap/tidb/pull/8454)
- 一部のケースで `UNION` 句の結果の長さが正しくない問題を修正しました [#8491](https://github.com/pingcap/tidb/pull/8491)
- `PREPARE FROM @var_name` の問題を修正しました [#8488](https://github.com/pingcap/tidb/pull/8488)
- 一部のケースで統計情報のダンプ中にパニックが発生する問題を修正しました [#8464](https://github.com/pingcap/tidb/pull/8464)
- 一部のケースでポイントクエリの統計推定の問題を修正しました [#8493](https://github.com/pingcap/tidb/pull/8493)
- デフォルトの `enum` 値が文字列で返される場合にパニックが発生する問題を修正しました [#8476](https://github.com/pingcap/tidb/pull/8476)
- ワイドテーブルのシナリオで過剰なメモリ消費が発生する問題を修正しました [#8467](https://github.com/pingcap/tidb/pull/8467)
- Parser が mod 演算子を誤ってフォーマットする場合に遭遇する問題を修正しました [#8431](https://github.com/pingcap/tidb/pull/8431)
- 外部キー制約を追加することによって発生する可能性のあるパニックの問題を修正しました [#8421](https://github.com/pingcap/tidb/pull/8421), [#8410](https://github.com/pingcap/tidb/pull/8410)
- `YEAR` 列のタイプがゼロ値を誤って変換する問題を修正しました [#8396](https://github.com/pingcap/tidb/pull/8396)
- `VALUES` 関数の引数が列でない場合に発生する可能性のあるパニックの問題を修正しました [#8404](https://github.com/pingcap/tidb/pull/8404)
- サブクエリを含むステートメントのためのプランキャッシュを無効化しました [#8395](https://github.com/pingcap/tidb/pull/8395)

## PD

- デッドロックによって RaftCluster が停止できなくなる可能性のある問題を修正しました [#1370](https://github.com/pingcap/pd/pull/1370)

## TiKV

- 新しく作成されたピアへのリーダーの移行を避け、可能な遅延を最適化するための不要なリージョンのハートビートを修正しました [#3929](https://github.com/tikv/tikv/pull/3929)
- 冗長なリージョンのハートビートを修正しました [#3930](https://github.com/tikv/tikv/pull/3930)