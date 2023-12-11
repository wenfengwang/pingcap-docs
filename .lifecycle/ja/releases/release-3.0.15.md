---
title: TiDB 3.0.15 リリースノート
aliases: ['/docs/dev/releases/release-3.0.15/']
---

# TiDB 3.0.15 リリースノート

リリース日: 2020年6月5日

TiDB バージョン: 3.0.15

## 新機能

+ TiDB

    - パーティションテーブルでのクエリでプランキャッシュ機能の使用を禁止 [#16759](https://github.com/pingcap/tidb/pull/16759)
    - パーティションテーブルに対する`admin recover index`および`admin check index`ステートメントのサポート [#17315](https://github.com/pingcap/tidb/pull/17315) [#17390](https://github.com/pingcap/tidb/pull/17390)
    - Rangeパーティションテーブルの`in`条件のパーティションプルーニングのサポート [#17318](https://github.com/pingcap/tidb/pull/17318)
    - `SHOW CREATE TABLE`の出力を最適化し、パーティション名に引用符を追加 [#16315](https://github.com/pingcap/tidb/pull/16315)
    - `GROUP_CONCAT`関数での`ORDER BY`句のサポート [#16988](https://github.com/pingcap/tidb/pull/16988)
    - `CMSketch`統計のメモリ割り当てメカニズムを最適化し、ガベージコレクション（GC）が性能に与える影響を軽減 [#17543](https://github.com/pingcap/tidb/pull/17543)

+ PD

    - PDがリーダーの数に基づいてスケジューリングを行うポリシーの追加 [#2479](https://github.com/pingcap/pd/pull/2479)

## バグ修正

+ TiDB

    - `Hash`集約関数で`enum`および`set`タイプのデータをコピーする際にディープコピーを使用し、正確性の問題を修正 [#16890](https://github.com/pingcap/tidb/pull/16890)
    - `PointGet`が整数のオーバーフローの処理ロジックの誤りにより、不正確な結果を返す問題を修正 [#16753](https://github.com/pingcap/tidb/pull/16753)
    - クエリ述語で`CHAR()`関数が使用される際の処理ロジックの誤りにより、不正確な結果が返される問題を修正 [#16557](https://github.com/pingcap/tidb/pull/16557)
    - `IsTrue`および`IsFalse`関数のストレージ層と計算層での結果の不一致問題を修正 [#16627](https://github.com/pingcap/tidb/pull/16627)
    - `case when`などいくつかの式での不正確な`NotNull`フラグの修正 [#16993](https://github.com/pingcap/tidb/pull/16993)
    - 一部のシナリオでオプティマイザが`TableDual`の物理計画を見つけられない問題を修正 [#17014](https://github.com/pingcap/tidb/pull/17014)
    - Hashパーティションテーブルでパーティション選択の構文が正しく機能しない問題を修正 [#17051](https://github.com/pingcap/tidb/pull/17051)
    - 浮動小数点数がXOR演算を行った際のTiDBとMySQLの結果の不一致を修正 [#16976](https://github.com/pingcap/tidb/pull/16976)
    - 準備済みの状態でDDLステートメントを実行した際に発生するエラーを修正 [#17415](https://github.com/pingcap/tidb/pull/17415)
    - ID配布機のバッチサイズの計算処理ロジックの不正確な処理を修正 [#17548](https://github.com/pingcap/tidb/pull/17548)
    - 時間が高コストしきい値を超えた際に`MAX_EXEC_TIME` SQLヒントが効果を発揮しない問題を修正 [#17534](https://github.com/pingcap/tidb/pull/17534)

+ TiKV

    - 長期実行後にメモリデフラグメンテーションが効果的でない問題を修正 [#7790](https://github.com/tikv/tikv/pull/7790)
    - TiKVが誤って再起動された後にスナップショットファイルが誤って削除されることによるパニックの問題を修正 [#7925](https://github.com/tikv/tikv/pull/7925)
    - メッセージパッケージが大きすぎて発生したgRPC切断の問題を修正 [#7822](https://github.com/tikv/tikv/pull/7822)