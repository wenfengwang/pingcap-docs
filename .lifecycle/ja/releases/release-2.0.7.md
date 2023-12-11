---
title: TiDB 2.0.7リリースノート
aliases: ['/docs/dev/releases/release-2.0.7/','/docs/dev/releases/207/']
---

# TiDB 2.0.7リリースノート

2018年9月7日、TiDB 2.0.7がリリースされました。TiDB 2.0.6と比較して、このリリースではシステムの互換性と安定性が大幅に向上しています。

## TiDB

- 新機能
    - `information_schema`の`PROCESSLIST`テーブルを追加しました [#7286](https://github.com/pingcap/tidb/pull/7286)
- 改善
    - SQL文の実行に関する詳細情報をより多く収集し、その情報を`SLOW QUERY`ログに出力するように改善しました [#7364](https://github.com/pingcap/tidb/pull/7364)
    - `SHOW CREATE TABLE`でパーティション情報を削除しました [#7388](https://github.com/pingcap/tidb/pull/7388)
    - `ANALYZE`文の実行効率を向上させるために、RC分離レベルと低優先度に設定しました [#7500](https://github.com/pingcap/tidb/pull/7500)
    - ユニークインデックスの追加を高速化しました [#7562](https://github.com/pingcap/tidb/pull/7562)
    - DDLの並行性を制御するオプションを追加しました [#7563](https://github.com/pingcap/tidb/pull/7563)
- バグ修正
    - 主キーが整数のテーブルで`USE INDEX(PRIMARY)`が使用できない問題を修正しました [#7298](https://github.com/pingcap/tidb/pull/7298)
    - `Merge Join`および`Index Join`が内部行が`NULL`の場合に正しい結果を出力しない問題を修正しました [#7301](https://github.com/pingcap/tidb/pull/7301)
    - チャンクサイズが小さすぎると`Join`が誤った結果を出力する問題を修正しました [#7315](https://github.com/pingcap/tidb/pull/7315)
    - `range column`を含むテーブルを作成するステートメントによって引き起こされるパニックの問題を修正しました [#7379](https://github.com/pingcap/tidb/pull/7379)
    - `admin check table`が誤って時間型列のエラーを報告する問題を修正しました [#7457](https://github.com/pingcap/tidb/pull/7457)
    - `current_timestamp`のデフォルト値を持つデータを`=`条件を使用してクエリできない問題を修正しました [#7467](https://github.com/pingcap/tidb/pull/7467)
    - `ComStmtSendLongData`コマンドで挿入されたゼロ長のパラメータが誤ってNULLに解析される問題を修正しました [#7508](https://github.com/pingcap/tidb/pull/7508)
    - 特定のシナリオで`auto analyze`が繰り返し実行される問題を修正しました [#7556](https://github.com/pingcap/tidb/pull/7556)
    - ニューライン文字で終わる単一行コメントをパーサーが解析できない問題を修正しました [#7635](https://github.com/pingcap/tidb/pull/7635)

## TiKV

- 改善
    - 空のクラスタではデフォルトで`dynamic-level-bytes`パラメータを開き、スペースの増幅を削減しました
- バグ修正
    - リージョンのマージ後にリージョンの`approximate size`および`approximate keys count`を更新します