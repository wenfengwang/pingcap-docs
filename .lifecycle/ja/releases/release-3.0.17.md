---
title: TiDB 3.0.17 リリースノート
---

# TiDB 3.0.17 リリースノート

リリース日: 2020年8月3日

TiDB バージョン: 3.0.17

## 改善点

+ TiDB

    - `query-feedback-limit` 設定項目のデフォルト値を1024から512に減少し、統計フィードバックメカニズムを改善し、クラスタへの影響を緩和しました [#18770](https://github.com/pingcap/tidb/pull/18770)
    - 1つのリクエストに対するバッチ分割数を制限するようにしました [#18694](https://github.com/pingcap/tidb/pull/18694)
    - TiDB クラスタで多くの過去のDDLジョブがある場合、`/tiflash/replica` HTTP API の実行を高速化しました [#18386](https://github.com/pingcap/tidb/pull/18386)
    - インデックスの等しい条件に対する行数の見積もりを改善しました [#17609](https://github.com/pingcap/tidb/pull/17609)
    - `kill tidb conn_id` の実行を高速化しました [#18506](https://github.com/pingcap/tidb/pull/18506)

+ TiKV

    - リージョンの休止を遅らせる`hibernate-timeout` 設定を追加し、ローリングアップデートのパフォーマンスを改善しました [#8207](https://github.com/tikv/tikv/pull/8207)

+ ツール

    + TiDB Lightning

        - `[black-white-list]` は新しい、より理解しやすいフィルタ形式に置き換えられました [#332](https://github.com/pingcap/tidb-lightning/pull/332)

## バグ修正

+ TiDB

    - `IndexHashJoin` または `IndexMergeJoin` を含むクエリがパニックに遭遇した場合、空のセットではなく実際のエラーメッセージを返すようにしました [#18498](https://github.com/pingcap/tidb/pull/18498)
    - `SELECT a FROM t HAVING t.a` のようなSQLステートメントでの不明な列のエラーを修正しました [#18432](https://github.com/pingcap/tidb/pull/18432)
    - テーブルにプライマリキーがない場合や、テーブルにすでに整数型のプライマリキーがある場合は、テーブルにプライマリキーを追加することを禁止しました [#18342](https://github.com/pingcap/tidb/pull/18342)
    - `EXPLAIN FORMAT="dot" FOR CONNECTION` を実行すると、空のセットを返すようにしました [#17157](https://github.com/pingcap/tidb/pull/17157)
    - `STR_TO_DATE` での '%r', '%h' フォーマットトークンの処理を修正しました [#18725](https://github.com/pingcap/tidb/pull/18725)

+ TiKV

    - リージョンのマージ中に古いデータが読み込まれる可能性があるバグを修正しました [#8111](https://github.com/tikv/tikv/pull/8111)
    - スケジューリングプロセス中のメモリリークの問題を修正しました [#8355](https://github.com/tikv/tikv/pull/8355)

+ ツール

    + TiDB Lightning

        - `log-file` フラグが無視される問題を修正しました [#345](https://github.com/pingcap/tidb-lightning/pull/345)