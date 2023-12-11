---
title: TiDB 3.0.4 リリースノート
aliases: ['/docs/dev/releases/release-3.0.4/','/docs/dev/releases/3.0.4/']
---

# TiDB 3.0.4 リリースノート

リリース日: 2019年10月8日

TiDB バージョン: 3.0.4

TiDB Ansible バージョン: 3.0.4

- 新機能
    - `performance_schema.events_statements_summary_by_digest` システムテーブルを追加し、SQL レベルでのパフォーマンスの問題をトラブルシューティングできるようにする
    - TiDB の `SHOW TABLE REGIONS` 構文に `WHERE` 句を追加
    - Reparo に `worker-count` および `txn-batch` 構成アイテムを追加して、回復速度を制御する
- 改善点
    - Region の分割コマンドと空の分割コマンドをサポートし、分割のパフォーマンスを改善するために TiKV でバッチ Region の分割をサポート
    - TiKV の RocksDB にダブルリンクドリストをサポートし、逆スキャンのパフォーマンスを向上させる
    - TiDB Ansible に `iosnoop` および `funcslower` の 2 つのパフォーマンスツールを追加し、クラスタの状態をよりよく診断できるようにする
    - TiDB の遅いクエリログの出力を最適化し、冗長なフィールドを削除することで、改善
- 動作の変更
    - `txn-local-latches.enable` のデフォルト値を `false` に更新し、TiDB でローカルトランザクションの競合をチェックする既定の動作を無効にする
    - TiDB でグローバルスコープの `tidb_txn_mode` システム変数を追加し、悲観的ロックの使用を許可する; TiDB は引き続きデフォルトで楽観的ロックを採用することに注意
    - TiDB 遅いクエリログの `Index_ids` フィールドを `Index_names` に置き換えて、遅いクエリログの使いやすさを向上させる
    - TiDB 構成ファイルに `split-region-max-num` パラメータを追加して、`SPLIT TABLE` 構文で許可される Region の最大数を変更する
    - SQL 実行がメモリ制限を超えた場合、リンクを切断せずに `Out Of Memory Quota` エラーを返す
    - 誤操作を避けるため、TiDB で列の `AUTO_INCREMENT` 属性を削除することを許可しない; この属性を削除するには、`tidb_allow_remove_auto_inc` システム変数を変更する
- 修正された問題
    - コメントされていない TiDB 固有の構文 `PRE_SPLIT_REGIONS` が、下流データベースでデータレプリケーション中にエラーを引き起こす可能性がある問題を修正
    - カーソルを使用して `PREPARE` + `EXECUTE` の結果を取得する際、TiDB で遅いクエリログが不正確である問題を修正
    - 隣接する小さな Region をマージできない問題を PD で修正
    - アイドル状態のクラスタでのファイル記述子リークが、TiKV プロセスが長時間実行された場合にプロセスが異常終了する可能性がある問題を TiKV で修正
- 貢献者

    このリリースに貢献したコミュニティからの以下の貢献者に感謝します:

    - [sduzh](https://github.com/sduzh)
    - [lizhenda](https://github.com/lizhenda)

## TiDB

- SQL オプティマイザー
    - フィードバックによって分割された無効なクエリ範囲が生成される可能性がある問題を修正 [#12170](https://github.com/pingcap/tidb/pull/12170)
    - `SHOW STATS_BUCKETS` ステートメントの返されたエラーを 16 進数で表示し、無効なキーを含む場合にエラーを返すのではなく、エラーにする問題を修正 [#12094](https://github.com/pingcap/tidb/pull/12094)
    - `SLEEP` 関数を含むクエリにおいて、列抽出によって無効な `sleep(1)` が発生する問題を修正 [#11953](https://github.com/pingcap/tidb/pull/11953)
    - クエリが列の数だけを対象とする場合に、インデックススキャンを使用して IO を低減させるように修正 [#12112](https://github.com/pingcap/tidb/pull/12112)
    - `use index()` でインデックスが指定されていない場合に、どのインデックスも使用しないように修正し、MySQL と互換性があるようにする [#12100](https://github.com/pingcap/tidb/pull/12100)
    - `TopN` レコードの数を厳密に制限して、`ANALYZE` ステートメントがトランザクションのサイズ制限を超えるために失敗する問題を修正 [#11914](https://github.com/pingcap/tidb/pull/11914)
    - `Update` ステートメントに含まれるサブクエリを変換する際のエラーを修正 [#12483](https://github.com/pingcap/tidb/pull/12483)
    - `select ... limit ... offset ...` ステートメントの実行パフォーマンスを最適化し、`Limit` 演算子を `IndexLookUpReader` の実行ロジックに押し下げることで改善 [#12378](https://github.com/pingcap/tidb/pull/12378)
- SQL 実行エンジン
    - `PREPARED` ステートメントが誤って実行された場合に、ログに SQL ステートメントを出力するように修正 [#12191](https://github.com/pingcap/tidb/pull/12191)
    - `UNIX_TIMESTAMP` 関数を使用してパーティショニングを実装する際にパーティショニングをプルーニングするようにサポート [#12169](https://github.com/pingcap/tidb/pull/12169)
    - `AUTO_INCREMENT` が誤って `MAX int64` および `MAX uint64` を割り当てるとエラーが報告されない問題を修正 [#12162](https://github.com/pingcap/tidb/pull/12162)
    - `SHOW TABLE … REGIONS` および `SHOW TABLE .. INDEX … REGIONS` 構文に `WHERE` 句を追加 [#12123](https://github.com/pingcap/tidb/pull/12123)
    - SQL 実行がメモリ制限を超えた場合、リンクを切断せずに `Out Of Memory Quota` エラーを返すように修正 [#12127](https://github.com/pingcap/tidb/pull/12127)
    - `JSON_UNQUOTE` 関数が JSON テキストを処理する際に不正な結果を返す問題を修正 [#11955](https://github.com/pingcap/tidb/pull/11955)
    - `AUTO_INCREMENT` 列に最初の行で値を割り当てる際に、`LAST INSERT ID` が正しくない問題を修正 (例: `insert into t (pk, c) values (1, 2), (NULL, 3)`) [#12002](https://github.com/pingcap/tidb/pull/12002)
    - `PREPARE` ステートメントの `GROUPBY` 解析ルールが不正確である問題を修正 [#12351](https://github.com/pingcap/tidb/pull/12351)
    - ポイントクエリにおける権限チェックが不正確である問題を修正 [#12340](https://github.com/pingcap/tidb/pull/12340)
    - `PREPARE` ステートメントの `sql_type` による期間が監視レコードで表示されない問題を修正 [#12331](https://github.com/pingcap/tidb/pull/12331)
    - ポイントクエリでテーブルのエイリアスを使用することをサポート (例: `select * from t tmp where a = "aa"`) [#12282](https://github.com/pingcap/tidb/pull/12282)
    - 負の値を符号なしとして扱わずに負の数値を BIT 型の列に挿入する際にエラーが発生する問題を修正 [#12423](https://github.com/pingcap/tidb/pull/12423)
    - 時間の丸め処理が不正確である問題を修正 (例: `2019-09-11 11:17:47.999999666` は `2019-09-11 11:17:48` に丸めるべきである) [#12258](https://github.com/pingcap/tidb/pull/12258)
    - 式のブロックリストの使用を洗練させる (例: `<` は `It` と同等) [#11975](https://github.com/pingcap/tidb/pull/11975)
    - 存在しない関数エラーのメッセージにデータベースプレフィックスを追加 (例: `[expression:1305]FUNCTION test.std_samp does not exist`) [#12111](https://github.com/pingcap/tidb/pull/12111)
- サーバ
    - 遅いクエリログで直前のステートメントを出力するために `Prev_stmt` フィールドを追加 [#12180](https://github.com/pingcap/tidb/pull/12180)
    - 冗長なフィールドを削除することで、遅いクエリログの出力を最適化 [#12144](https://github.com/pingcap/tidb/pull/12144)
    - ローカルトランザクションの競合をチェックする既定の動作を無効にするために、`txn-local-latches.enable` のデフォルト値を `false` に更新 [#12095](https://github.com/pingcap/tidb/pull/12095)
    - `Index_ids` フィールドを `Index_names` に置き換えて、遅いクエリログの使いやすさを向上させる [#12061](https://github.com/pingcap/tidb/pull/12061)
    - グローバルスコープで `tidb_txn_mode` システム変数を追加し、悲観的ロックを使用することを許可 [#12049](https://github.com/pingcap/tidb/pull/12049)
    - コミット段階の 2PC での Backoff 情報を記録するために、遅いクエリログに `Backoff` フィールドを追加 [#12335](https://github.com/pingcap/tidb/pull/12335)
- `PREPARE` + `EXECUTE`でcursorを使用した場合に、遅いクエリログが正しくない問題を修正します（たとえば、`PREPARE stmt1FROM SELECT * FROM t WHERE a > ?; EXECUTE stmt1 USING @variable`）[#123092](https://github.com/pingcap/tidb/pull/12392)
    - `tidb_enable_stmt_summary`をサポートします。この機能が有効になると、TiDBはSQLステートメントをカウントし、システムテーブル`performance_schema.events_statements_summary_by_digest`を使用してクエリできます[#12308](https://github.com/pingcap/tidb/pull/12308)
    - tikv-clientの一部のログレベルを調整します（たとえば、`batchRecvLoop fails`のログレベルを`ERROR`から`INFO`に変更）[#12383](https://github.com/pingcap/tidb/pull/12383)
- DDL
    - `tidb_allow_remove_auto_inc`変数を追加します。カラムの`AUTO INCREMENT`属性の削除はデフォルトで無効になっています[#12145](https://github.com/pingcap/tidb/pull/12145)
    - コメントされていないTiDB固有の構文`PRE_SPLIT_REGIONS`が、データレプリケーション中にダウンストリームデータベースでエラーを引き起こす問題を修正します[#12120](https://github.com/pingcap/tidb/pull/12120)
    - 設定ファイルに`split-region-max-num`変数を追加し、最大許容のリージョン数を調整可能にします[#12097](https://github.com/pingcap/tidb/pull/12079)
    - リージョンを複数のリージョンに分割する機能をサポートし、リージョンの分散中のタイムアウト問題を修正します[#12343](https://github.com/pingcap/tidb/pull/12343)
    - `AUTO_INCREMENT`カラムを含むインデックスが二つのインデックスによって参照されている場合に`drop index`ステートメントが失敗する問題を修正します[#12344](https://github.com/pingcap/tidb/pull/12344)
- モニタ
    - `tikvclient`でgRPC接続エラーの数をカウントする`connection_transient_failure_count`モニタリングメトリクスを追加します[#12093](https://github.com/pingcap/tidb/pull/12093)

## TiKV

- Raftstore
    - 空のリージョンでキーの数を正確にカウントしない問題を修正します[#5414](https://github.com/tikv/tikv/pull/5414)
    - リバーススキャンのパフォーマンスを向上させるためにRocksDB用のダブルリンクリストをサポートします[#5368](https://github.com/tikv/tikv/pull/5368)
    - バッチリージョン分割コマンドと空の分割コマンドをサポートし、分割パフォーマンスを向上させます[#5470](https://github.com/tikv/tikv/pull/5470)
- サーバ
    - `-V`コマンドの出力形式が2.Xの形式と一致しない問題を修正します[#5501](https://github.com/tikv/tikv/pull/5501)
    - 3.0ブランチでTitanを最新バージョンにアップグレードします[#5517](https://github.com/tikv/tikv/pull/5517)
    - grpcioをv0.4.5にアップグレードします[#5523](https://github.com/tikv/tikv/pull/5523)
    - gRPCのコアダンプの問題を修正し、OOMを回避するために共有メモリをサポートします[#5524](https://github.com/tikv/tikv/pull/5524)
    - アイドルクラスターでのファイルディスクリプタのリークが長時間実行されたときにTiKVプロセスが異常終了する問題を修正します[#5567](https://github.com/tikv/tikv/pull/5567)
- ストレージ
    - `txn_heart_beat` APIをサポートして、TiDBの悲観的ロックをできる限りMySQLと一致するようにします[#5507](https://github.com/tikv/tikv/pull/5507)
    - 特定の状況でのポイントクエリのパフォーマンスが低い問題を修正します[#5495](https://github.com/tikv/tikv/pull/5495) [#5463](https://github.com/tikv/tikv/pull/5463)

## PD

- 隣接する小さなリージョンをマージできない問題を修正します[#1726](https://github.com/pingcap/pd/pull/1726)
- `pd-ctl`でのTLS有効化パラメータが無効である問題を修正します[#1738](https://github.com/pingcap/pd/pull/1738)
- PDオペレータが誤って削除されるスレッドセーフの問題を修正します[#1734](https://github.com/pingcap/pd/pull/1734)
- リージョンシンカーにTLSをサポートします[#1739](https://github.com/pingcap/pd/pull/1739)

## ツール

- TiDB Binlog
    - Reparoでの`worker-count`および`txn-batch`構成項目を追加し、リカバリ速度を制御します[#746](https://github.com/pingcap/tidb-binlog/pull/746)
    - Drainerのメモリ使用量を最適化し、同時実行の効率を向上させます[#737](https://github.com/pingcap/tidb-binlog/pull/737)
- TiDB Lightning
    - チェックポイントからのデータ再インポートがTiDB Lightningをパニックさせる可能性がある問題を修正します[#237](https://github.com/pingcap/tidb-lightning/pull/237)
    - `AUTO_INCREMENT`のアルゴリズムを最適化し、`AUTO_INCREMENT`カラムのオーバーフローのリスクを減らします[#227](https://github.com/pingcap/tidb-lightning/pull/227)

## TiDB Ansible

- TiSparkをv2.2.0にアップグレードします[#926](https://github.com/pingcap/tidb-ansible/pull/926)
- TiDB構成項目`pessimistic_txn`のデフォルト値を`true`に更新します[#933](https://github.com/pingcap/tidb-ansible/pull/933)
- `node_exporter`にシステムレベルのモニタリングメトリクスを追加します[#938](https://github.com/pingcap/tidb-ansible/pull/938)
- クラスターの状態をよりよく診断するためにTiDB Ansibleに`iosnoop`および`funcslower`の2つのパフォーマンスツールを追加します[#946](https://github.com/pingcap/tidb-ansible/pull/946)
- パスワード有効期限切れなどの状況での待機時間が長い問題に対処するために、`raw`モジュールを`shell`モジュールに置換します[#949](https://github.com/pingcap/tidb-ansible/pull/949)
- TiDB構成項目`txn_local_latches`のデフォルト値を`false`に更新します
- Grafanaダッシュボードのモニタリングメトリクスと警告ルールを最適化します[#962](https://github.com/pingcap/tidb-ansible/pull/962) [#963](https://github.com/pingcap/tidb-ansible/pull/963) [#969](https://github.com/pingcap/tidb-ansible/pull/963)
- 展開とアップグレード前に設定ファイルをチェックします[#934](https://github.com/pingcap/tidb-ansible/pull/934) [#972](https://github.com/pingcap/tidb-ansible/pull/972)