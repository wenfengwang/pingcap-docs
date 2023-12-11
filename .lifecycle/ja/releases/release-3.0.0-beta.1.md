---
title: TiDB 3.0.0 Beta.1 リリースノート
aliases: ['/docs/dev/releases/release-3.0.0-beta.1/','/docs/dev/releases/3.0.0-beta.1/']
---

# TiDB 3.0.0 Beta.1 リリースノート

リリース日: 2019年3月26日

TiDB バージョン: 3.0.0-beta.1

TiDB Ansible バージョン: 3.0.0-beta.1

## 概要

2019年3月26日に、TiDB 3.0.0 Beta.1 がリリースされました。対応する TiDB Ansible バージョンは 3.0.0 Beta.1 です。このリリースは、TiDB 3.0.0 Beta と比較して、安定性、使いやすさ、機能、SQL オプティマイザ、統計情報、および実行エンジンが大幅に向上しています。

## TiDB

+ SQL オプティマイザ
    - `Sort Merge Join` を使用して、直積を計算するサポート [#9037](https://github.com/pingcap/tidb/pull/9037)
    - 統計情報に過度に依存する実行計画を防ぐための、一部のルールを使用した Skyline Pruning のサポート [#9337](https://github.com/pingcap/tidb/pull/9337)
    + ウィンドウ関数のサポート
        - `NTILE` [#9682](https://github.com/pingcap/tidb/pull/9682)
        - `LEAD` および `LAG` [#9672](https://github.com/pingcap/tidb/pull/9672)
        - `PERCENT_RANK` [#9671](https://github.com/pingcap/tidb/pull/9671)
        - `NTH_VALUE` [#9596](https://github.com/pingcap/tidb/pull/9596)
        - `CUME_DIST` [#9619](https://github.com/pingcap/tidb/pull/9619)
        - `FIRST_VALUE` および `LAST_VALUE` [#9560](https://github.com/pingcap/tidb/pull/9560)
        - `RANK` および `DENSE_RANK` [#9500](https://github.com/pingcap/tidb/pull/9500)
        - `RANGE FRAMED` [#9450](https://github.com/pingcap/tidb/pull/9450)
        - `ROW FRAMED` [#9358](https://github.com/pingcap/tidb/pull/9358)
        - `ROW NUMBER` [#9098](https://github.com/pingcap/tidb/pull/9098)
    - 列間の順序相関を示す統計情報のタイプを追加 [#9315](https://github.com/pingcap/tidb/pull/9315)
+ SQL 実行エンジン
    + 組込み関数の追加
        - `JSON_QUOTE` [#7832](https://github.com/pingcap/tidb/pull/7832)
        - `JSON_ARRAY_APPEND` [#9609](https://github.com/pingcap/tidb/pull/9609)
        - `JSON_MERGE_PRESERVE` [#8931](https://github.com/pingcap/tidb/pull/8931)
        - `BENCHMARK` [#9252](https://github.com/pingcap/tidb/pull/9252)
        - `COALESCE` [#9087](https://github.com/pingcap/tidb/pull/9087)
        - `NAME_CONST` [#9261](https://github.com/pingcap/tidb/pull/9261)
    - クエリーの文脈に基づいて Chunk サイズを最適化し、SQL ステートメントの実行時間とクラスタのリソース消費を削減するためのサポート [#6489](https://github.com/pingcap/tidb/issues/6489)
+ 特権管理
    - `SET ROLE` および `CURRENT_ROLE` のサポート [#9581](https://github.com/pingcap/tidb/pull/9581)
    - `DROP ROLE` のサポート [#9616](https://github.com/pingcap/tidb/pull/9616)
    - `CREATE ROLE` のサポート [#9461](https://github.com/pingcap/tidb/pull/9461)
+ サーバー
    - `/debug/zip` HTTP インターフェースを追加し、現在の TiDB インスタンスの情報を取得するためのサポート [#9651](https://github.com/pingcap/tidb/pull/9651)
    - Pump または Drainer のステータスをチェックするための `show pump status` および `show drainer status` SQL ステートメントをサポート [#9456](https://github.com/pingcap/tidb/pull/9456)
    - SQL ステートメントを使用して Pump または Drainer のステータスを変更するためのサポート [#9789](https://github.com/pingcap/tidb/pull/9789)
    - 遅いSQL ステートメントを簡単にトラッキングするために、SQL テキストに HASH フィンガープリントを追加するサポート [#9662](https://github.com/pingcap/tidb/pull/9662)
    - バイナリログの有効化状態を制御するための `log_bin` システム変数（デフォルトは "0"）を追加し、現在は状態のみサポート [#9343](https://github.com/pingcap/tidb/pull/9343)
    - 設定ファイルを使用してバイナリログの送信戦略を管理するためのサポート [#9864](https://github.com/pingcap/tidb/pull/9864)
    - `INFORMATION_SCHEMA.SLOW_QUERY` メモリーテーブルを使用して遅いクエリーを問い合わせるサポート [#9290](https://github.com/pingcap/tidb/pull/9290)
    - TiDB で表示される MySQL バージョンを 5.7.10 から 5.7.25 に変更 [#9553](https://github.com/pingcap/tidb/pull/9553)
    - ツールによる簡単な収集および解析のため、[ログフォーマット](https://github.com/tikv/rfcs/blob/master/text/0018-unified-log-format.md)を統一
    - 統計情報に基づく実際のデータボリュームと推定データボリュームの差を記録するための `high_error_rate_feedback_total` モニタリングアイテムを追加 [#9209](https://github.com/pingcap/tidb/pull/9209)
    - データベースディメンションで QPS モニタリングアイテムを追加し、設定項目を使用して有効にするサポート [#9151](https://github.com/pingcap/tidb/pull/9151)
+ DDL
    - DDL タスクのリトライ回数を制限するための `ddl_error_count_limit` グローバル変数（デフォルトは "512"）を追加（この数が限界を超えると、DDL タスクがキャンセルされます） [#9295](https://github.com/pingcap/tidb/pull/9295)
    - `ALTER ALGORITHM` `INPLACE`/`INSTANT` のサポート [#8811](https://github.com/pingcap/tidb/pull/8811)
    - `SHOW CREATE VIEW` ステートメントのサポート [#9309](https://github.com/pingcap/tidb/pull/9309)
    - `SHOW CREATE USER` ステートメントのサポート [#9240](https://github.com/pingcap/tidb/pull/9240)

## PD

+ 統一された[ログフォーマット](https://github.com/tikv/rfcs/blob/master/text/0018-unified-log-format.md)をツールによる簡単な収集と解析のために統一
+ シミュレータ
    - 異なるストアで異なるハートビート間隔をサポート [#1418](https://github.com/pingcap/pd/pull/1418)
    - データのインポートに関するケースを追加 [#1263](https://github.com/pingcap/pd/pull/1263)
+ ホットスポットのスケジューリングを設定可能にするための追加 [#1412](https://github.com/pingcap/pd/pull/1412)
+ 監視アイテムのディメンションとしてストアアドレスを追加し、以前のストアIDを置き換えるためのサポート [#1429](https://github.com/pingcap/pd/pull/1429)
+ Region 検査サイクルを高速化するための、`GetStores` のオーバーヘッドの最適化 [#1410](https://github.com/pingcap/pd/pull/1410)
+ Tombstone ストアを削除するためのインターフェースを追加 [#1472](https://github.com/pingcap/pd/pull/1472)

## TiKV

+ Coprocessor 計算実行フレームワークを最適化し、TableScan セクションを実装し、Single TableScan パフォーマンスを5%〜30%改善しました
    - `BatchRows` 行と `BatchColumn` 列の定義を実装 [#3660](https://github.com/tikv/tikv/pull/3660)
    - エンコードおよびデコードされたデータにアクセスするための`VectorLike` の実装をサポートするための `BatchExecutor` の定義を実装 [#4242](https://github.com/tikv/tikv/pull/4242)
    - 式ツリーを逆ポーランド記法に変換する実装 [#4329](https://github.com/tikv/tikv/pull/4329)
    - `BatchTableScanExecutor` ベクトル化オペレータの実装を加速させるためのサポート [#4351](https://github.com/tikv/tikv/pull/4351)
+ 統一された[ログフォーマット](https://github.com/tikv/rfcs/blob/master/text/0018-unified-log-format.md)をツールによる簡単な収集と解析のために統一
+ Local Readerを使用してRaw Readインターフェイスで読み込むサポートを追加 [#4222](https://github.com/tikv/tikv/pull/4222)
+ 構成情報に関するメトリクスを追加 [#4206](https://github.com/tikv/tikv/pull/4206)
+ キーの上限を超えることに関するメトリクスを追加 [#4255](https://github.com/tikv/tikv/pull/4255)
+ キーの上限を超えるエラーが発生した場合のパニックまたはエラーを制御するオプションを追加 [#4254](https://github.com/tikv/tikv/pull/4254)
+ `INSERT`操作のサポートを追加し、キーが存在しない場合のみprewriteが成功し、`Batch Get`を取り除く[#4085](https://github.com/tikv/tikv/pull/4085)
+ バッチシステムでより公平なバッチ戦略を使用する[#4200](https://github.com/tikv/tikv/pull/4200)
+ tikv-ctlでRawスキャンをサポート[#3825](https://github.com/tikv/tikv/pull/3825)

## ツール

+ TiDB Binlog
    - Kafkaからbinlogを読み込み、データをMySQLにレプリケートするArbiterツールを追加
    - レプリケートする必要のないファイルをフィルタリングするサポートを追加
    - 生成列のレプリケートをサポート
+ Lightning
    - TiKVの定期的なLevel-1コンパクションを無効にするサポートを追加し、TiKVクラスターバージョンが2.1.4以降の場合、ImportモードでLevel-1コンパクションが自動的に実行される[#119](https://github.com/pingcap/tidb-lightning/pull/119), [#4199](https://github.com/tikv/tikv/pull/4199)
    - インポートエンジンの数を制限し、Importerディスクスペースの過剰使用を回避するために`table_concurrency`構成項目を追加（デフォルトは"16"）
    - メモリ使用量を削減するために、中間状態のSSTをディスクに保存するサポートを追加[#4369](https://github.com/tikv/tikv/pull/4369)
    - TiKV-Importerのインポートパフォーマンスを最適化し、大きなテーブルのデータとインデックスの別々のインポートをサポート[#132](https://github.com/pingcap/tidb-lightning/pull/132)
    - CSVファイルのインポートをサポート[#111](https://github.com/pingcap/tidb-lightning/pull/111)
+ データレプリケーション比較ツール（sync-diff-inspector）
    - TiDB統計を使用して比較するチャンクを分割するサポートを追加[#197](https://github.com/pingcap/tidb-tools/pull/197)
    - 複数の列を使用して比較するチャンクを分割するサポートを追加[#197](https://github.com/pingcap/tidb-tools/pull/197)