---
title: TiDB 3.0.18 Release Notes
---

# TiDB 3.0.18 更新履歴

リリース日: 2020年8月21日

TiDBバージョン: 3.0.18

## 改善点

+ ツール

    + TiDB Binlog

        - Pump GC構成の時間間隔形式でGoをサポート [#996](https://github.com/pingcap/tidb-binlog/pull/996)

## バグ修正

+ TiDB

    - `Hash`関数による`decimal`型の誤った処理が誤ったHashJoin結果を引き起こす問題を修正 [#19185](https://github.com/pingcap/tidb/pull/19185)
    - `Hash`関数による`set`および`enum`型の誤った処理が誤ったHashJoin結果を引き起こす問題を修正 [#19175](https://github.com/pingcap/tidb/pull/19175)
    - 悲観的ロックモードで重複キーのチェックに失敗する問題を修正 [#19236](https://github.com/pingcap/tidb/pull/19236)
    - `Apply`および`Union Scan`演算子による誤った実行結果を引き起こす問題を修正 [#19297](https://github.com/pingcap/tidb/pull/19297)
    - トランザクションで一部のキャッシュされた実行計画が誤って実行される問題を修正 [#19274](https://github.com/pingcap/tidb/pull/19274)

+ TiKV

    - GC失敗ログのレベルを`error`から`warning`に変更 [#8444](https://github.com/tikv/tikv/pull/8444)

+ ツール

    + TiDB Lightning

        - `--log-file`引数が効果を上げない問題を修正 [#345](https://github.com/pingcap/tidb-lightning/pull/345)
        - TiDB-backendを使用する際に空のバイナリ/16進数リテラルの構文エラーを修正 [#357](https://github.com/pingcap/tidb-lightning/pull/357)
        - TiDB-backendを使用する際の予期しない`switch-mode`呼び出しを修正 [#368](https://github.com/pingcap/tidb-lightning/pull/368)