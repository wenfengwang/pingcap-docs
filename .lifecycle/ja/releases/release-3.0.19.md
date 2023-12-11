---
title: TiDB 3.0.19リリースノート
---

# TiDB 3.0.19リリースノート

リリース日: 2020年9月25日

TiDBバージョン: 3.0.19

## 互換性の変更

+ PD

    - インポートパスを `pingcap/pd` から `tikv/pd` に変更 [#2779](https://github.com/pingcap/pd/pull/2779)
    - 著作権情報を `PingCAP, Inc` から `TiKV Project Authors` に変更 [#2777](https://github.com/pingcap/pd/pull/2777)

## 改善点

+ TiDB

    - 故障回復がクエリパフォーマンスに与える影響を軽減 [#19764](https://github.com/pingcap/tidb/pull/19764)
    - `union`演算子の同時実行数を調整するサポートを追加 [#19885](https://github.com/pingcap/tidb/pull/19885)

+ TiKV

    - `sync-log`を非調整可能な値で`true`に設定 [#8636](https://github.com/tikv/tikv/pull/8636)

+ PD

    - PD再起動のためのアラート規則を追加 [#2789](https://github.com/pingcap/pd/pull/2789)

## バグ修正

+ TiDB

    - `slow-log`ファイルが存在しない場合に発生するクエリエラーを修正 [#20050](https://github.com/pingcap/tidb/pull/20050)
    - `SHOW STATS_META`および`SHOW STATS_BUCKET`の特権チェックを追加 [#19759](https://github.com/pingcap/tidb/pull/19759)
    - 十進数型を整数型に変更することを禁止 [#19681](https://github.com/pingcap/tidb/pull/19681)
    - `ENUM`/`SET`型の列を変更する際に制約がチェックされない問題を修正 [#20045](https://github.com/pingcap/tidb/pull/20045)
    - パニック後にtidb-serverがテーブルロックを解放しないバグを修正 [#20021](https://github.com/pingcap/tidb/pull/20021)
    - `WHERE`句で`OR`演算子が正しく処理されないバグを修正 [#19901](https://github.com/pingcap/tidb/pull/19901)

+ TiKV

    - 理由フレーズが欠落したレスポンスのパース時にTiKVがパニックするバグを修正 [#8540](https://github.com/tikv/tikv/pull/8540)

+ ツール

    + TiDB Lightning

        - 厳密モードでCSVに不正なUTF文字が含まれる場合にTiDB Lightningプロセスが適切なタイミングで終了しない問題を修正 [#378](https://github.com/pingcap/tidb-lightning/pull/378)