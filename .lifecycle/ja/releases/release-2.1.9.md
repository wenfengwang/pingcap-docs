---
title: TiDB 2.1.9のリリースノート
aliases: ['/docs/dev/releases/release-2.1.9/', '/docs/dev/releases/2.1.9/']
---

# TiDB 2.1.9リリースノート

リリース日：2019年5月6日

TiDBバージョン：2.1.9

TiDB Ansibleバージョン：2.1.9

## TiDB

- 符号なし型がオーバーフローした場合の`MAKETIME`関数の互換性を修正 [#10089](https://github.com/pingcap/tidb/pull/10089)
- 定数畳み込みによるスタックオーバーフローをいくつかのケースで修正 [#10189](https://github.com/pingcap/tidb/pull/10189)
- `Update`時の権限チェックの問題を、エイリアスが存在する場合に修正 [#10157](https://github.com/pingcap/tidb/pull/10157), [#10326](https://github.com/pingcap/tidb/pull/10326)
- DistSQLにおけるメモリ使用量を追跡・制御する [#10197](https://github.com/pingcap/tidb/pull/10197)
- `utf8mb4_0900_ai_ci`として照合を指定するサポートを追加 [#10201](https://github.com/pingcap/tidb/pull/10201)
- プライマリキーが符号なし型の場合の`MAX`関数の誤った結果の問題を修正 [#10209](https://github.com/pingcap/tidb/pull/10209)
- NOT NULL列にNULL値が挿入される問題を、厳密でないSQLモードで修正 [#10254](https://github.com/pingcap/tidb/pull/10254)
- `DISTINCT`で複数の列が存在する場合の`COUNT`関数の誤った結果の問題を修正 [#10270](https://github.com/pingcap/tidb/pull/10270)
- `LOAD DATA`が異常なCSVファイルを解析した際のパニックの問題を修正 [#10269](https://github.com/pingcap/tidb/pull/10269)
- `Index Lookup Join`において外部と内部の結合キーの型が一致しない場合のオーバーフローエラーを無視 [#10244](https://github.com/pingcap/tidb/pull/10244)
- 一部の場合において、ステートメントが誤ってpoint-getと判断される問題を修正 [#10299](https://github.com/pingcap/tidb/pull/10299)
- 時間型が一部の場合においてタイムゾーンを変換しない場合の誤った結果の問題を修正 [#10345](https://github.com/pingcap/tidb/pull/10345)
- 一部の場合においてTiDB文字セットのケースが一貫していない問題を修正 [#10354](https://github.com/pingcap/tidb/pull/10354)
- オペレータによって返される行の数を制御するサポートを追加 [#9166](https://github.com/pingcap/tidb/issues/9166)
    - 選択 & 投影 [#10110](https://github.com/pingcap/tidb/pull/10110)
    - `StreamAgg` & `HashAgg` [#10133](https://github.com/pingcap/tidb/pull/10133)
    - `TableReader` & `IndexReader` & `IndexLookup` [#10169](https://github.com/pingcap/tidb/pull/10169)
- 遅いクエリログを改善
    - 類似のSQLを区別するために`SQL Digest`を追加 [#10093](https://github.com/pingcap/tidb/pull/10093)
    - 遅いクエリステートメントで使用される統計情報のバージョン情報を追加 [#10220](https://github.com/pingcap/tidb/pull/10220)
    - 遅いクエリログにおけるステートメントのメモリ消費を表示 [#10246](https://github.com/pingcap/tidb/pull/10246)
    - Coprocessor関連情報の出力形式を調整し、pt-query-digestによって解析可能に修正 [#10300](https://github.com/pingcap/tidb/pull/10300)
    - 遅いクエリステートメントにおける`#`文字の問題を修正 [#10275](https://github.com/pingcap/tidb/pull/10275)
    - 遅いクエリステートメントのメモリテーブルにいくつかの情報列を追加 [#10317](https://github.com/pingcap/tidb/pull/10317)
    - 遅いクエリログにトランザクションのコミット時間を追加 [#10310](https://github.com/pingcap/tidb/pull/10310)
    - pt-query-digestによって解析できない場合の時刻形式を修正 [#10323](https://github.com/pingcap/tidb/pull/10323)

## PD

- GetOperatorサービスのサポートを追加 [#1514](https://github.com/pingcap/pd/pull/1514)

## TiKV

- リーダーの転送時の潜在的なクオラム変更を修正 [#4604](https://github.com/tikv/tikv/pull/4604)

## ツール

- TiDB Binlog
    - プライマリキーカラムの符号なしint型のデータがマイナス数字になるため、データレプリケーションが中断される問題を修正 [#574](https://github.com/pingcap/tidb-binlog/pull/574)
    - ダウンストリームが`pb`の場合に圧縮オプションを削除し、ダウンストリーム名を`pb`から`file`に変更 [#597](https://github.com/pingcap/tidb-binlog/pull/575)
    - TiDB Binlog 2.1.7で導入されたReparoが誤った`UPDATE`ステートメントを生成するバグを修正 [#576](https://github.com/pingcap/tidb-binlog/pull/576)
- TiDB Lightning
    - パーサーによってビット型の列データが誤って解析されるバグを修正 [#164](https://github.com/pingcap/tidb-lightning/pull/164)
    - ダンプファイル内の不足する列データを、行IDまたはデフォルトの列値を使用して補完する機能を追加 [#174](https://github.com/pingcap/tidb-lightning/pull/174)
    - Importerが一部のSSTファイルのインポートに失敗するが、依然として成功のインポート結果を返すバグを修正 [#4566](https://github.com/tikv/tikv/pull/4566)
    - TiKVへのSSTファイルのアップロード時に速度制限を設定する機能をサポート [#4607](https://github.com/tikv/tikv/pull/4607)
    - CPU消費を削減するためにImporterのRocksDB SST圧縮方法を`lz4`に変更 [#4624](https://github.com/tikv/tikv/pull/4624)
- sync-diff-inspector
    - チェックポイントをサポート [#227](https://github.com/pingcap/tidb-tools/pull/227)

## TiDB Ansible

- tidb-ansibleドキュメント内のリンクをドキュメントの再構築に合わせて更新 [#740](https://github.com/pingcap/tidb-ansible/pull/740), [#741](https://github.com/pingcap/tidb-ansible/pull/741)
- `inventory.ini`ファイルで`enable_slow_query_log`パラメータを削除し、遅いクエリログをデフォルトで個別のログファイルに出力するように変更 [#742](https://github.com/pingcap/tidb-ansible/pull/742)