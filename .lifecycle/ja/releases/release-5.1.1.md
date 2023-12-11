---
title: TiDB 5.1.1 リリースノート
---

# TiDB 5.1.1 リリースノート

リリース日: 2021年7月30日

TiDB バージョン: 5.1.1

## 互換性の変更

+ TiDB

    - TiDBクラスタのv4.0からv5.1へのアップグレードでは、 `tidb_multi_statement_mode` のデフォルト値は `OFF` になりました。代わりにクライアントライブラリのマルチステートメント機能を使用することをお勧めします。 詳細については、[ `tidb_multi_statement_mode`のドキュメント](/system-variables.md#tidb_multi_statement_mode-new-in-v4011)を参照してください。[#25751](https://github.com/pingcap/tidb/pull/25751)
    - `tidb_stmt_summary_max_stmt_count` 変数のデフォルト値を `200` から `3000` に変更しました [#25874](https://github.com/pingcap/tidb/pull/25874)
    - `table_storage_stats` テーブルにアクセスするには `SUPER` 特権が必要になりました [#26352](https://github.com/pingcap/tidb/pull/26352)
    - 他のユーザーの権限を表示するには、`mysql.user` に対する `SELECT` 特権が `information_schema.user_privileges` テーブルにアクセスするために必要になりました [#26311](https://github.com/pingcap/tidb/pull/26311)
    - `information_schema.cluster_hardware` テーブルにアクセスするには `CONFIG` 特権が必要になりました [#26297](https://github.com/pingcap/tidb/pull/26297)
    - `information_schema.cluster_info` テーブルにアクセスするには `PROCESS` 特権が必要になりました [#26297](https://github.com/pingcap/tidb/pull/26297)
    - `information_schema.cluster_load` テーブルにアクセスするには `PROCESS` 特権が必要になりました [#26297](https://github.com/pingcap/tidb/pull/26297)
    - `information_schema.cluster_systeminfo` テーブルにアクセスするには `PROCESS` 特権が必要になりました [#26297](https://github.com/pingcap/tidb/pull/26297)
    - `information_schema.cluster_log` テーブルにアクセスするには `PROCESS` 特権が必要になりました [#26297](https://github.com/pingcap/tidb/pull/26297)
    - `information_schema.cluster_config` テーブルにアクセスするには `CONFIG` 特権が必要になりました [#26150](https://github.com/pingcap/tidb/pull/26150)

## 機能の強化

+ TiDB Dashboard

    - OIDC SSOをサポートしました。OIDC互換SSOサービス（OktaやAuth0など）を設定することで、ユーザーはSQLパスワードを入力せずにTiDB Dashboardにログインできます。[#3883](https://github.com/tikv/pd/pull/3883)

+ TiFlash

    - DAGリクエストで `HAVING()` 関数をサポートしました

## 改善点

+ TiDB

    - Stale Read機能の一般提供（GA）を発表しました
    - データの挿入を高速化するために `paramMarker` への割り当てを避けるようにしました [#26076](https://github.com/pingcap/tidb/pull/26076)
    - クエリ結果をより安定させるために安定結果モードをサポートしました [#25995](https://github.com/pingcap/tidb/pull/25995)
    - 組み込み関数 `json_unquote()` を TiKV にプッシュダウンするサポートを追加しました [#26265](https://github.com/pingcap/tidb/pull/26265)
    - MPPクエリのリトライをサポートしました [#26480](https://github.com/pingcap/tidb/pull/26480)
    - `UPDATE` 読み取り用にインデックスキーの `point get` または `batch point get` で `LOCK` レコードを `PUT` レコードに変更しました [#26225](https://github.com/pingcap/tidb/pull/26225)
    - 古いクエリからのビューの作成を禁止しました [#26200](https://github.com/pingcap/tidb/pull/26200)
    - MPPモードでの `COUNT(DISTINCT)` 集約関数の完全なプッシュダウンをサポートしました [#26194](https://github.com/pingcap/tidb/pull/26194)
    - MPPクエリを開始する前にTiFlashの利用可能性を確認するようにしました [#26192](https://github.com/pingcap/tidb/pull/26192)
    - 読み取りタイムスタンプを将来の時刻に設定することを許可しないようにしました [#25763](https://github.com/pingcap/tidb/pull/25763)
    - `EXPLAIN` 文で集計関数をプッシュダウンできない場合にログ警告を出力するようにしました [#25737](https://github.com/pingcap/tidb/pull/25737)
    - クラスタの `statements_summary_evicted` テーブルを追加して、クラスタのエビクト数情報を記録するようにしました [#25587](https://github.com/pingcap/tidb/pull/25587)
    - 組み込み関数 `str_to_date` のMySQL互換性を向上し、フォーマット指定子 `%b/%M/%r/%T` に対応しました [#25768](https://github.com/pingcap/tidb/pull/25768)

+ TiKV

    - 可能な限りprewriteリクエストを冪等性にすることで、未確定のエラーの発生率を減らすようにしました [#10586](https://github.com/tikv/tikv/pull/10586)
    - 多数の期限切れコマンドを処理する際のスタックオーバーフローのリスクを防ぐようにしました [#10502](https://github.com/tikv/tikv/pull/10502)
    - ステールリードリクエストの `start_ts` を使用して `max_ts` を更新しないことで、コミットリクエストの過剰なリトライを回避しました [#10451](https://github.com/tikv/tikv/pull/10451)
    - 読み取り待ちと書き込み待ちを別々に処理することで読み取り待ちの遅延を減らしました [#10592](https://github.com/tikv/tikv/pull/10592)
    - I/Oレート制限が有効な場合のデータインポート速度への影響を減らすようにしました [#10390](https://github.com/tikv/tikv/pr/10390)
    - Raft gRPC接続間の負荷分散を改善しました [#10495](https://github.com/tikv/tikv/pr/10495)

+ ツール

    + TiCDC

        - `file sorter` を削除しました [#2327](https://github.com/pingcap/tiflow/pull/2327)
        - PDエンドポイントが証明書を欠いている場合に返されるエラーメッセージを改善しました [#1973](https://github.com/pingcap/tiflow/issues/1973)

    + TiDB Lightning

        - スキーマの復元のためのリトライメカニズムを追加しました [#1294](https://github.com/pingcap/br/pull/1294)

    + Dumpling

        - 上流がTiDB v3.xクラスターである場合は常に `_tidb_rowid` を使用してテーブルを分割し、TiDBのメモリ使用量を削減するのに役立ちます [#295](https://github.com/pingcap/dumpling/issues/295)
        - データベースメタデータへのアクセス頻度を減らし、Dumplingのパフォーマンスと安定性を向上させるために改善しました [#315](https://github.com/pingcap/dumpling/pull/315)

## バグ修正

+ TiDB

    - `tidb_enable_amend_pessimistic_txn=on` を使用して列の型を変更すると発生するデータ損失の問題を修正しました [#26203](https://github.com/pingcap/tidb/issues/26203)
    - SQLモードで `last_day` 関数の動作が互換性のない問題を修正しました [#26001](https://github.com/pingcap/tidb/pull/26001)
    - ウィンドウ関数の上に `LIMIT` がある場合に発生する可能性があるパニックの問題を修正しました [#25344](https://github.com/pingcap/tidb/issues/25344)
    - 悲観的なトランザクションのコミットが書き込み競合を引き起こす可能性がある問題を修正しました [#25964](https://github.com/pingcap/tidb/issues/25964)
    - 相関サブクエリでのインデックス結合の結果が間違っている問題を修正しました [#25799](https://github.com/pingcap/tidb/issues/25799)
    - 正常にコミットされた楽観的トランザクションがコミットエラーを報告する可能性があるバグを修正しました [#10468](https://github.com/tikv/tikv/issues/10468)
    - `SET` 型のカラムでマージ結合を使用した場合に不正な結果が返される問題を修正しました [#25669](https://github.com/pingcap/tidb/issues/25669)
    - 悲観的トランザクションでインデックスキーが繰り返しコミットされる可能性があるバグを修正しました [#26359](https://github.com/pingcap/tidb/issues/26359)
    - オプティマイザがパーティションを特定する際に整数オーバーフローのリスクを修正しました [#26227](https://github.com/pingcap/tidb/issues/26227)
    - `DATE` をタイムスタンプにキャストする際に無効な値が書き込まれる可能性がある問題を修正しました [#26292](https://github.com/pingcap/tidb/issues/26292)
    - Grafana上でCoprocessor Cacheメトリクスが表示されない問題を修正しました [#26338](https://github.com/pingcap/tidb/issues/26338)
    - テレメトリによって発生するうるさいログの問題を修正しました [#25760](https://github.com/pingcap/tidb/issues/25760) [#25785](https://github.com/pingcap/tidb/issues/25785)
    - プレフィックスインデックスのクエリ範囲に関するバグを修正しました [#26029](https://github.com/pingcap/tidb/issues/26029)
- 同一のパーティションを同時に刈り取る際にDDLの実行が停止する問題を修正しました[#26229](https://github.com/pingcap/tidb/issues/26229)
    - 重複する`ENUM`アイテムの問題を修正しました[#25955](https://github.com/pingcap/tidb/issues/25955)
    - CTEイテレーターが正しくクローズされないバグを修正しました[#26112](https://github.com/pingcap/tidb/issues/26112)
    - `LOAD DATA`ステートメントが非UTF8データを異常に取り込む可能性がある問題を修正しました[#25979](https://github.com/pingcap/tidb/issues/25979)
    - 符号なし整数列でウィンドウ関数を使用する際に発生するパニックの問題を修正しました[#25956](https://github.com/pingcap/tidb/issues/25956)
    - 非同期コミットロックの解決時にTiDBがパニックする可能性がある問題を修正しました[#25778](https://github.com/pingcap/tidb/issues/25778)
    - 期限切れの読み込みが`PREPARE`ステートメントと完全に互換性がない問題を修正しました[#25800](https://github.com/pingcap/tidb/pull/25800)
    - ODBCスタイルの定数（たとえば、`{d '2020-01-01'}`）を式として使用できない問題を修正しました[#25531](https://github.com/pingcap/tidb/issues/25531)
    - TiDBを単独で実行する際に発生するエラーを修正しました[#25555](https://github.com/pingcap/tidb/pull/25555)

+ TiKV

    - 特定のプラットフォームで期間の計算がパニックする可能性がある問題を修正しました[#10569](https://github.com/tikv/tikv/pull/10569)
    - `Load Base Split`が`batch_get_command`のエンコードされていないキーを誤って使用する問題を修正しました[#10542](https://github.com/tikv/tikv/issues/10542)
    - `resolved-ts.advance-ts-interval`構成を動的に変更しても即座に有効にならない問題を修正しました[#10426](https://github.com/tikv/tikv/issues/10426)
    - 4つ以上のレプリカがある場合の稀なケースでフォロワーメタデータが破損する問題を修正しました[#10225](https://github.com/tikv/tikv/issues/10225)
    - 暗号化が有効な場合にスナップショットを2回構築するとパニックする問題を修正しました[#9786](https://github.com/tikv/tikv/issues/9786) [#10407](https://github.com/tikv/tikv/issues/10407)
    - `tikv_raftstore_hibernated_peer_state`メトリックが間違っている問題を修正しました[#10330](https://github.com/tikv/tikv/issues/10330)
    - コプロセッサーで`json_unquote()`関数の引数の型が間違っていた問題を修正しました[#10176](https://github.com/tikv/tikv/issues/10176)
    - 悲観的なトランザクションでインデックスキーが繰り返しコミットされる可能性があるバグを修正しました[#10468](https://github.com/tikv/tikv/issues/10468#issuecomment-869491061)
    - リーダーが移行された直後に`ReadIndex`リクエストが古い結果を返す問題を修正しました[#9351](https://github.com/tikv/tikv/issues/9351)

+ PD

    - 複数のスケジューラが同時に実行されると競合が発生して期待されるスケジューリングが生成されない問題を修正しました[#3807](https://github.com/tikv/pd/issues/3807) [#3778](https://github.com/tikv/pd/issues/3778)
    - スケジューラが既に削除されているにも関わらずスケジューラが再度表示される問題を修正しました[#2572](https://github.com/tikv/pd/issues/2572)

+ TiFlash

    - テーブルスキャンタスクを実行する際に発生する潜在的なパニックの問題を修正しました
    - DAQリクエストを処理する際にTiFlashが`duplicated region`のエラーを発生させるバグを修正しました
    - 読み込み負荷が重い場合に発生する可能性のあるパニックの問題を修正しました
    - `DateFormat`関数を実行する際に発生する潜在的なパニックの問題を修正しました
    - MPPタスクを実行する際に発生する潜在的なメモリリークの問題を修正しました
    - 集約関数`COUNT`または`COUNT DISTINCT`を実行する際に予期しない結果が発生する問題を修正しました
    - 複数のディスクに展開された場合にTiFlashがデータを復元できない可能性のあるバグを修正しました
    - TiDBダッシュボードがTiFlashのディスク情報を正しく表示できない問題を修正しました
    - `SharedQueryBlockInputStream`の解体時に発生する潜在的なパニックの問題を修正しました
    - `MPPTask`の解体時に発生する潜在的なパニックの問題を修正しました
    - スナップショットを介したデータ同期後にデータの不整合が発生する可能性のある問題を修正しました

+ ツール

    + TiCDC

        - 新しい照合機能のサポートを修正しました[#2301](https://github.com/pingcap/tiflow/issues/2301)
        - 実行時に共有マップへの同期されていないアクセスがパニックを引き起こす可能性がある問題を修正しました[#2300](https://github.com/pingcap/tiflow/pull/2300)
        - オーナーがDDLステートメントを実行中にクラッシュした場合に発生するDDLの損失問題を修正しました[#2290](https://github.com/pingcap/tiflow/pull/2290)
        - TiDBで未然にロックを解決しようとする問題を修正しました[#2188](https://github.com/pingcap/tiflow/issues/2188)
        - TiCDCノードがテーブル移行直後に即座に終了されてしまうとデータが失われる可能性のあるバグを修正しました[#2033](https://github.com/pingcap/tiflow/pull/2033)
        - `--sort-dir`および`--start-ts`の`changefeed update`の処理ロジックを修正しました[#1921](https://github.com/pingcap/tiflow/pull/1921)

    + バックアップ＆リストア（BR）

        - リストアするデータのサイズが誤って計算される問題を修正しました[#1270](https://github.com/pingcap/br/issues/1270)
        - cdclogからのリストア時に発生するDDLイベントの不足問題を修正しました[#870](https://github.com/pingcap/br/issues/870)

    + TiDB Lightning

        - TiDBがParquetファイルの`DECIMAL`型データを解析できない問題を修正しました[#1275](https://github.com/pingcap/br/pull/1275)
        - キー間隔を計算する際に整数オーバーフローが発生する問題を修正しました[#1291](https://github.com/pingcap/br/issues/1291) [#1290](https://github.com/pingcap/br/issues/1290)