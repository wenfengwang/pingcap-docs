---
title: TiDB 2.0.6 リリースノート
aliases: ['/docs/dev/releases/release-2.0.6/','/docs/dev/releases/206/']
---

# TiDB 2.0.6 リリースノート

2018年8月6日、TiDB 2.0.6 がリリースされました。TiDB 2.0.5 と比較して、このリリースではシステムの互換性と安定性に大幅な改善がされています。

## TiDB

- 改善
    - "システム変数の設定"のログを短くしてディスク容量を節約するようにしました [#7031](https://github.com/pingcap/tidb/pull/7031)
    - `ADD INDEX`の実行中に遅い操作を記録し、トラブルシューティングを容易にするためのログに記録するようにしました [#7083](https://github.com/pingcap/tidb/pull/7083)
    - 統計情報を更新する際のトランザクションの競合を減らすようにしました [#7138](https://github.com/pingcap/tidb/pull/7138)
    - 評価待ちの値が統計情報の範囲を超えた場合の行数推定の精度を改善するようにしました [#7185](https://github.com/pingcap/tidb/pull/7185)
    - 実行効率を改善するために `Index Join` の外部テーブルに推定行数が小さいテーブルを選択するようにしました [#7277](https://github.com/pingcap/tidb/pull/7277)
    - `ANALYZE TABLE`の実行中に発生したパニックの回復メカニズムを追加し、統計情報収集プロセスの異常な振る舞いによるtidb-serverの利用不能を回避するようにしました [#7228](https://github.com/pingcap/tidb/pull/7228)
    - `RPAD`/`LPAD`の結果が `max_allowed_packet` システム変数の値を超えた場合、NULLと対応する警告を返すようにしました。これによりMySQLとの互換性が担保されます [#7244](https://github.com/pingcap/tidb/pull/7244)
    - `PREPARE`文のプレースホルダー数の上限を65535に設定するようにしました。これによりMySQLとの互換性が担保されます [#7250](https://github.com/pingcap/tidb/pull/7250)
- バグ修正
    - 特定のケースで `DROP USER` 文がMySQLの動作と互換性がない問題を修正しました [#7014](https://github.com/pingcap/tidb/pull/7014)
    - `INSERT`/`LOAD DATA`のような文が`tidb_batch_insert`を開いた後、OOMに遭遇する問題を修正しました [#7092](https://github.com/pingcap/tidb/pull/7092)
    - テーブルのデータが更新し続けると、統計情報が自動的にアップデートされない問題を修正しました [#7093](https://github.com/pingcap/tidb/pull/7093)
    - ファイアウォールが非アクティブなgRPC接続を中断させる問題を修正しました [#7099](https://github.com/pingcap/tidb/pull/7099)
    - 特定のシナリオでプレフィックスインデックスが誤った結果を返す問題を修正しました [#7126](https://github.com/pingcap/tidb/pull/7126)
    - 特定のシナリオで古い統計情報によってパニックが発生する問題を修正しました [#7155](https://github.com/pingcap/tidb/pull/7155)
    - `ADD INDEX`操作の後、1つのインデックスデータが欠落する問題を特定のシナリオで修正しました [#7156](https://github.com/pingcap/tidb/pull/7156)
    - ユニークインデックスを使用して `NULL` 値をクエリする際に誤った結果が返される問題を特定のシナリオで修正しました [#7172](https://github.com/pingcap/tidb/pull/7172)
    - 特定のシナリオで `DECIMAL` の乗算結果の誤ったコードが返される問題を修正しました [#7212](https://github.com/pingcap/tidb/pull/7212)
    - 特定のシナリオで `DECIMAL` のモジュロ演算の誤った結果が返される問題を修正しました [#7245](https://github.com/pingcap/tidb/pull/7245)
    - トランザクション内の `UPDATE`/`DELETE`文が特定の特殊な文のシーケンスの下で誤った結果を返す問題を修正しました [#7219](https://github.com/pingcap/tidb/pull/7219)
    - 特定のシナリオで実行計画の構築中に `UNION ALL`/`UPDATE`文においてパニックが発生する問題を修正しました [#7225](https://github.com/pingcap/tidb/pull/7225)
    - 特定のシナリオでプレフィックスインデックスの範囲が誤って計算される問題を修正しました [#7231](https://github.com/pingcap/tidb/pull/7231)
    - 特定のシナリオで `LOAD DATA`文がバイナリログに書き込むに失敗する問題を修正しました [#7242](https://github.com/pingcap/tidb/pull/7242)
    - `ADD INDEX`の実行中に `SHOW CREATE TABLE`の誤った結果が返る問題を特定のシナリオで修正しました [#7243](https://github.com/pingcap/tidb/pull/7243)
    - 特定のシナリオで `Index Join`がタイムスタンプを初期化しないことによってパニックが発生する問題を修正しました [#7246](https://github.com/pingcap/tidb/pull/7246)
    - 誤ってタイムゾーンをセッションで使用する `ADMIN CHECK TABLE`が誤った警告を出す問題を修正しました [#7258](https://github.com/pingcap/tidb/pull/7258)
    - 特定のシナリオで `ADMIN CLEANUP INDEX`がインデックスをクリーンアップしない問題を修正しました [#7265](https://github.com/pingcap/tidb/pull/7265)
    - Read Committed分離レベルを無効にしました [#7282](https://github.com/pingcap/tidb/pull/7282)

## TiKV

- 改善
    - 偽の競合を減らすために、スケジューラのデフォルトスロットを拡張しました
    - 極端な競合がある場合のReadパフォーマンスを向上させるため、連続するロールバックトランザクションの記録を減らしました
    - 長時間実行される状況において不必要なディスク使用量を減らすために、RocksDBログファイルのサイズと数を制限しました
- バグ修正
    - 文字列から10進数にデータ型を変換する際のクラッシュ問題を修正しました