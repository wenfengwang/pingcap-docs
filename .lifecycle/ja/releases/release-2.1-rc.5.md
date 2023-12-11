---
title: TiDB 2.1 RC5 リリースノート
aliases: ['/docs/dev/releases/release-2.1-rc.5/','/docs/dev/releases/21rc5/']
---

<!-- markdownlint-disable MD032 -->

# TiDB 2.1 RC5 リリースノート

2018年11月12日、TiDB 2.1 RC5 がリリースされました。このリリースは、TiDB 2.1 RC4 と比較して、安定性、SQL オプティマイザ、統計情報、実行エンジンにおいて大幅な改善があります。

## TiDB

+ SQL オプティマイザ
    - `IndexReader` が一部のケースで誤ったハンドルを読み取る問題を修正しました [#8132](https://github.com/pingcap/tidb/pull/8132)
    - `IndexScan Prepared` ステートメントが `Plan Cache` を使用する際に発生する問題を修正しました [#8055](https://github.com/pingcap/tidb/pull/8055)
    - `Union` ステートメントの結果が不安定である問題を修正しました [#8165](https://github.com/pingcap/tidb/pull/8165)
+ SQL 実行エンジン
    - ワイドテーブルの挿入または更新における TiDB のパフォーマンスを向上させました [#8024](https://github.com/pingcap/tidb/pull/8024)
    - `Truncate` 組み込み関数で符号なし `int` フラグをサポートしました [#8068](https://github.com/pingcap/tidb/pull/8068)
    - JSON データを decimal 型に変換する際のエラーを修正しました [#8109](https://github.com/pingcap/tidb/pull/8109)
    - float 型を `Update` する際に発生するエラーを修正しました [#8170](https://github.com/pingcap/tidb/pull/8170)
+ 統計情報
    - 一部のケースでポイントクエリにおける誤った統計情報の問題を修正しました [#8035](https://github.com/pingcap/tidb/pull/8035)
    - 主キーの統計情報の選択度の推定を一部のケースで修正しました [#8149](https://github.com/pingcap/tidb/pull/8149)
    - 削除されたテーブルの統計情報が長期間クリアされない問題を修正しました [#8182](https://github.com/pingcap/tidb/pull/8182)
+ サーバー
    + ログの可読性を向上させ、ログをより良くしました
        - [#8063](https://github.com/pingcap/tidb/pull/8063)
        - [#8053](https://github.com/pingcap/tidb/pull/8053)
        - [#8224](https://github.com/pingcap/tidb/pull/8224)
    - `infoschema.profiling` のテーブルデータを取得する際に発生するエラーを修正しました [#8096](https://github.com/pingcap/tidb/pull/8096)
    - バイナリログを書き込むために、unix ソケットを pumps クライアントで置き換えました [#8098](https://github.com/pingcap/tidb/pull/8098)
    - `tidb_slow_log_threshold` 環境変数のしきい値値を追加し、スローログを動的に設定します [#8094](https://github.com/pingcap/tidb/pull/8094)
    - `tidb_query_log_max_len` 環境変数がログを動的に設定する際に、SQL ステートメントの元の長さを追加しました [#8200](https://github.com/pingcap/tidb/pull/8200)
    - `_tidb_rowid` の書き込みを許可するかどうかを制御するための `tidb_opt_write_row_id` 環境変数を追加しました [#8218](https://github.com/pingcap/tidb/pull/8218)
    - ticlient の `Scan` コマンドに上限を追加し、オーバーバウンドスキャンを回避しました [#8081](https://github.com/pingcap/tidb/pull/8081), [#8247](https://github.com/pingcap/tidb/pull/8247)
+ DDL
    - トランザクション内で DDL ステートメントを実行する際に、一部のケースでエラーが発生する問題を修正しました [#8056](https://github.com/pingcap/tidb/pull/8056)
    - パーティションテーブルで `truncate table` を実行しても効果がない問題を修正しました [#8103](https://github.com/pingcap/tidb/pull/8103)
    - キャンセル後に DDL 操作が正しくロールバックされない問題を修正しました [#8057](https://github.com/pingcap/tidb/pull/8057)
    - 次に利用可能な行 ID を返す `admin show next_row_id` コマンドを追加しました [#8268](https://github.com/pingcap/tidb/pull/8268)

## PD

+ `pd-ctl` がリージョンキーを読み取る関連した問題を修正しました
    - [#1298](https://github.com/pingcap/pd/pull/1298)
    - [#1299](https://github.com/pingcap/pd/pull/1299)
    - [#1308](https://github.com/pingcap/pd/pull/1308)
+ `regions/check` API が誤った結果を返す問題を修正しました [#1311](https://github.com/pingcap/pd/pull/1311)
+ PD が PD の参加失敗後に再起動できない問題を修正しました [#1279](https://github.com/pingcap/pd/pull/1279)
+ `watch leader` が一部のケースでイベントを失った可能性がある問題を修正しました [#1317](https://github.com/pingcap/pd/pull/1317)

## TiKV

+ `WriteConflict` のエラーメッセージを向上させました [#3750](https://github.com/tikv/tikv/pull/3750)
+ パニックマークファイルを追加しました [#3746](https://github.com/tikv/tikv/pull/3746)
+ 新しい gRPC バージョンによるセグメントフォールトの問題を避けるため、grpcio をダウングレードしました [#3650](https://github.com/tikv/tikv/pull/3650)
+ `kv_scan` インタフェースに上限を追加しました [#3749](https://github.com/tikv/tikv/pull/3749)

## ツール

- 以前のバージョンの binlog と互換性がない TiDB-Binlog クラスターをサポートしました [#8093](https://github.com/pingcap/tidb/pull/8093), [documentation](/tidb-binlog/tidb-binlog-overview.md)