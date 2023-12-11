---
title: TiDB 2.0.8 更新情報
aliases: ['/docs/dev/releases/release-2.0.8/','/docs/dev/releases/208/']
---

# TiDB 2.0.8 更新情報

2018年10月16日、TiDB 2.0.8 がリリースされました。TiDB 2.0.7 と比較して、このリリースではシステムの互換性と安定性が大幅に向上しています。

## TiDB

+ 改良
    - 更新ステートメントが対応するAUTO-INCREMENTカラムを変更しない場合、AUTO-IDの増加速度を遅くする [#7846](https://github.com/pingcap/tidb/pull/7846)

+ バグ修正
    - Quickly create a new etcd session to recover the service when the PD leader goes down [#7810](https://github.com/pingcap/tidb/pull/7810)
    - デフォルトの`DateTime`型のデフォルト値の計算時にタイムゾーンが考慮されない問題を修正 [#7672](https://github.com/pingcap/tidb/pull/7672)
    - `duplicate key update`が特定の条件下で値を誤って挿入する問題を修正 [#7685](https://github.com/pingcap/tidb/pull/7685)
    - `UnionScan`の述語条件がプッシュダウンされない問題を修正 [#7726](https://github.com/pingcap/tidb/pull/7726)
    - `TIMESTAMP`インデックスを追加する際に、タイムゾーンが正しく処理されない問題を修正 [#7812](https://github.com/pingcap/tidb/pull/7812)
    - 特定の条件下で統計モジュールによるメモリリーク問題を修正 [#7864](https://github.com/pingcap/tidb/pull/7864)
    - 異常条件下で`ANALYZE`の結果を取得できない問題を修正 [#7871](https://github.com/pingcap/tidb/pull/7871)
    -  `SYSDATE`関数を折りたたまず、正しい結果が返されるように修正 [#7894](https://github.com/pingcap/tidb/pull/7894)
    - 特定の条件下で`substring_index`のパニック問題を修正 [#7896](https://github.com/pingcap/tidb/pull/7896)
    - 特定の条件下で`OUTER JOIN`が誤って`INNER JOIN`に変換される問題を修正 [#7899](https://github.com/pingcap/tidb/pull/7899)

## TiKV

+ バグ修正
    - ノードがダウンするとRaftstoreの`EntryCache`によって消費されるメモリが増加し続ける問題を修正 [#3529](https://github.com/tikv/tikv/pull/3529)