---
title: TiDB 2.1 RC2 のリリースノート
aliases: ['/docs/dev/releases/release-2.1-rc.2/', '/docs/dev/releases/21rc2/']
---

# TiDB 2.1 RC2 のリリースノート

2018年9月14日、TiDB 2.1 RC2 がリリースされました。このリリースでは、TiDB 2.1 RC1 と比較して、安定性、SQL オプティマイザ、統計情報、実行エンジンの大幅な改善が行われています。

## TiDB

* SQL オプティマイザ
    * 次世代プランナーの提案 [#7543](https://github.com/pingcap/tidb/pull/7543)
    * 定数伝播の最適化ルールの改善 [#7276](https://github.com/pingcap/tidb/pull/7276)
    * 計算ロジックの強化により `Range` が複数の `IN` または `EQUAL` 条件を同時に処理できるように [#7577](https://github.com/pingcap/tidb/pull/7577)
    * `Range` が空の場合に `TableScan` の推定結果が不正確である問題を修正 [#7583](https://github.com/pingcap/tidb/pull/7583)
    * `UPDATE` ステートメントに `PointGet` 演算子をサポート [#7586](https://github.com/pingcap/tidb/pull/7586)
    * 特定の条件下で `FirstRow` 集約関数を実行する際のパニック問題を修正 [#7624](https://github.com/pingcap/tidb/pull/7624)
* SQL 実行エンジン
    * `HashJoin` 演算子でエラーが発生した際の潜在的な `DataRace` 問題を修正 [#7554](https://github.com/pingcap/tidb/pull/7554)
    * `HashJoin` 演算子が内部テーブルを読み込み、同時にハッシュテーブルを構築するように修正 [#7544](https://github.com/pingcap/tidb/pull/7544)
    * ハッシュ集約演算子のパフォーマンスを最適化 [#7541](https://github.com/pingcap/tidb/pull/7541)
    * 結合演算子のパフォーマンスを最適化 [#7493](https://github.com/pingcap/tidb/pull/7493), [#7433](https://github.com/pingcap/tidb/pull/7433)
    * 結合順序が変更された場合に `UPDATE JOIN` の結果が不正確である問題を修正 [#7571](https://github.com/pingcap/tidb/pull/7571)
    * Chunk の反復子のパフォーマンスを向上 [#7585](https://github.com/pingcap/tidb/pull/7585)
* 統計情報
    * 自動アナライズが統計情報を繰り返し分析する問題を修正 [#7550](https://github.com/pingcap/tidb/pull/7550)
    * 統計情報の変更がない場合に発生する統計情報更新エラーを修正 [#7530](https://github.com/pingcap/tidb/pull/7530)
    * `Analyze` リクエストの構築時に RC 分離レベルと低い優先度を使用するように修正 [#7496](https://github.com/pingcap/tidb/pull/7496)
    * 特定の時間帯での統計情報自動解析をサポートするように修正 [#7570](https://github.com/pingcap/tidb/pull/7570)
    * 統計情報のログ記録時のパニック問題を修正 [#7588](https://github.com/pingcap/tidb/pull/7588)
    * `ANALYZE TABLE WITH BUCKETS` ステートメントを使用してヒストグラムのバケット数を構成するようにサポート [#7619](https://github.com/pingcap/tidb/pull/7619)
    * 空のヒストグラムを更新する際のパニック問題を修正 [#7640](https://github.com/pingcap/tidb/pull/7640)
    * 統計情報を使用して `information_schema.tables.data_length` を更新するように修正 [#7657](https://github.com/pingcap/tidb/pull/7657)
* サーバー
    * Trace 関連の依存関係を追加 [#7532](https://github.com/pingcap/tidb/pull/7532)
    * Golang の `mutex profile` 機能を有効化 [#7512](https://github.com/pingcap/tidb/pull/7512)
    * `Admin` ステートメントに `Super_priv` 権限が必要なように修正 [#7486](https://github.com/pingcap/tidb/pull/7486)
    * 重要なシステムテーブルを `Drop` することをユーザーに禁止するように修正 [#7471](https://github.com/pingcap/tidb/pull/7471)
    * `juju/errors` から `pkg/errors` に切り替えるように修正 [#7151](https://github.com/pingcap/tidb/pull/7151)
    * SQL トレースの機能プロトタイプを完成させるように修正 [#7016](https://github.com/pingcap/tidb/pull/7016)
    * ゴルーチンプールを削除するように修正 [#7564](https://github.com/pingcap/tidb/pull/7564)
    * `USER1` シグナルを使用してゴルーチン情報を表示するようにサポートするように修正 [#7587](https://github.com/pingcap/tidb/pull/7587)
    * TiDB の起動時に内部 SQL を高い優先度で設定するように修正 [#7616](https://github.com/pingcap/tidb/pull/7616)
    * モニタリングメトリクスで内部 SQL とユーザーSQLを異なるラベルでフィルタリングするように修正 [#7631](https://github.com/pingcap/tidb/pull/7631)
    * 直近1週間のトップ30の遅いクエリをTiDBサーバに保存するように修正 [#7646](https://github.com/pingcap/tidb/pull/7646)
    * TiDB クラスタのグローバルシステムタイムゾーンを設定する提案を提出するように修正 [#7656](https://github.com/pingcap/tidb/pull/7656)
    * "GC life time is shorter than transaction duration" のエラーメッセージを改善するように修正 [#7658](https://github.com/pingcap/tidb/pull/7658)
    * TiDB クラスタの起動時にグローバルシステムタイムゾーンを設定するように修正 [#7638](https://github.com/pingcap/tidb/pull/7638)
* 互換性
    * `Year` 型の符号なしフラグを追加するように修正 [#7542](https://github.com/pingcap/tidb/pull/7542)
    * `Prepare`/`Execute` モードで `Year` 型の結果長を設定する問題を修正 [#7525](https://github.com/pingcap/tidb/pull/7525)
    * `Prepare`/`Execute` モードでゼロタイムスタンプを挿入する問題を修正 [#7506](https://github.com/pingcap/tidb/pull/7506)
    * 整数除算のエラー処理問題を修正 [#7492](https://github.com/pingcap/tidb/pull/7492)
    * `ComStmtSendLongData` の処理時の互換性問題を修正 [#7485](https://github.com/pingcap/tidb/pull/7485)
    * 文字列を整数に変換する際のエラー処理問題を修正 [#7483](https://github.com/pingcap/tidb/pull/7483)
    * `information_schema.columns_in_table` テーブルの値の精度を最適化 [#7463](https://github.com/pingcap/tidb/pull/7463)
    * MariaDBクライアントを使用してデータの文字列型を書き込むか更新する際の互換性問題を修正 [#7573](https://github.com/pingcap/tidb/pull/7573)
    * 戻り値の別名の互換性問題を修正 [#7600](https://github.com/pingcap/tidb/pull/7600)
    * `information_schema.COLUMNS` テーブルで float 型の `NUMERIC_SCALE` 値が不正確である問題を修正 [#7602](https://github.com/pingcap/tidb/pull/7602)
    * 単一行コメントが空の場合にParserがエラーを報告する問題を修正 [#7612](https://github.com/pingcap/tidb/pull/7612)
* 式
    * `insert` 関数内部で `max_allowed_packet` の値をチェックするように修正 [#7528](https://github.com/pingcap/tidb/pull/7528)
    * 組み込み関数 `json_contains` をサポートするように修正 [#7443](https://github.com/pingcap/tidb/pull/7443)
    * 組み込み関数 `json_contains_path` をサポートするように修正 [#7596](https://github.com/pingcap/tidb/pull/7596)
    * 組み込み関数 `encode/decode` をサポートするように修正 [#7622](https://github.com/pingcap/tidb/pull/7622)
    * 一部の時間関連関数が特定の場合にMySQLの挙動と互換性がない問題を修正 [#7636](https://github.com/pingcap/tidb/pull/7636)
    * 文字列内のデータの時刻型の解析の互換性問題を修正 [#7654](https://github.com/pingcap/tidb/pull/7654)
    * `DateTime` データのデフォルト値を計算する際にタイムゾーンを考慮しない問題を修正 [#7655](https://github.com/pingcap/tidb/pull/7655)
* DML
    * `InsertOnDuplicateUpdate` ステートメントで正しい `last_insert_id` を設定するように修正 [#7534](https://github.com/pingcap/tidb/pull/7534)
    * `auto_increment_id` カウンターの更新ケースを削減する[#7515](https://github.com/pingcap/tidb/pull/7515)
    * `Duplicate Key`のエラーメッセージを最適化する[#7495](https://github.com/pingcap/tidb/pull/7495)
    * `insert...select...on duplicate key update`の問題を修正する[#7406](https://github.com/pingcap/tidb/pull/7406)
    * `LOAD DATA IGNORE LINES`ステートメントをサポートする[#7576](https://github.com/pingcap/tidb/pull/7576)
* DDL
    * モニターにDDLジョブタイプと現在のスキーマバージョン情報を追加する[#7472](https://github.com/pingcap/tidb/pull/7472)
    * `Admin Restore Table`機能の設計を完了する[#7383](https://github.com/pingcap/tidb/pull/7383)
    * `Bit`型のデフォルト値が128を超える問題を修正する[#7249](https://github.com/pingcap/tidb/pull/7249)
    * `Bit`型のデフォルト値が`NULL`であっても使用できない問題を修正する[#7604](https://github.com/pingcap/tidb/pull/7604)
    * DDLキューでの`CREATE TABLE/DATABASE`のチェック間隔を短縮する[#7608](https://github.com/pingcap/tidb/pull/7608)
    * `ddl/owner/resign` HTTPインターフェースを使用してDDLオーナーを解放し、新しいオーナーの選出を開始する[#7649](https://github.com/pingcap/tidb/pull/7649)
* TiKV Goクライアント
    * `Seek`操作が`Key`のみを取得する問題をサポートする[#7419](https://github.com/pingcap/tidb/pull/7419)
* [テーブルパーティション](https://github.com/pingcap/tidb/projects/6)（実験）
    * `Bigint`型をパーティションキーとして使用できない問題を修正する[#7520](https://github.com/pingcap/tidb/pull/7520)
    * パーティションテーブルにインデックスを追加する際に問題が発生した場合のロールバック操作をサポートする[#7437](https://github.com/pingcap/tidb/pull/7437)

## PD

* 機能
    * `GetAllStores`インターフェースをサポートする[#1228](https://github.com/pingcap/pd/pull/1228)
    * スケジューリング推定の統計情報をシミュレータに追加する[#1218](https://github.com/pingcap/pd/pull/1218)
* 改善
    * ダウンストアの処理プロセスを最適化し、できるだけ早くレプリカを補完する[#1222](https://github.com/pingcap/pd/pull/1222)
    * コーディネータの起動を最適化し、PDを再起動したことによる不必要なスケジューリングを削減する[#1225](https://github.com/pingcap/pd/pull/1225)
    * ハートビートに伴うオーバーヘッドを削減するためにメモリ使用量を最適化する[#1195](https://github.com/pingcap/pd/pull/1195)
    * エラー処理を最適化し、ログ情報を改善する[#1227](https://github.com/pingcap/pd/pull/1227)
    * pd-ctlで特定のストアのリージョン情報のクエリをサポートする[#1231](https://github.com/pingcap/pd/pull/1231)
    * pd-ctlでバージョン比較に基づいたトップNリージョン情報のクエリをサポートする[#1233](https://github.com/pingcap/pd/pull/1233)
    * pd-ctlでより正確なTSOデコードをサポートする[#1242](https://github.com/pingcap/pd/pull/1242)
* バグ修正
    * pd-ctlが`hot store`コマンドを誤って終了させる問題を修正する[#1244](https://github.com/pingcap/pd/pull/1244)

## TiKV

* パフォーマンス
    * 統計推定に基づいてリージョンを分割し、I/Oコストを削減する[#3511](https://github.com/tikv/tikv/pull/3511)
    * トランザクションスケジューラでのクローンを削減する[#3530](https://github.com/tikv/tikv/pull/3530)
* 改善
    * 大量の組み込み関数に対するプッシュダウンサポートを追加する
    * 特定のシナリオでのリーダースケジューリングの失敗問題を修正するために`leader-transfer-max-log-lag`構成を追加する[#3507](https://github.com/tikv/tikv/pull/3507)
    * `tikv-importer`によって同時にオープンされるエンジンの数を制限するために`max-open-engines`構成を追加する[#3496](https://github.com/tikv/tikv/pull/3496)
    * `snapshot apply`に対するガベージデータのクリーンアップ速度を制限し、影響を軽減する[#3547](https://github.com/tikv/tikv/pull/3547)
    * 重要なRaftメッセージのコミットメッセージをブロードキャストして不要な遅延を回避する[#3592](https://github.com/tikv/tikv/pull/3592)
* バグ修正
    * 新しく分割されたリージョンの`PreVote`メッセージを破棄することによるリーダー選出問題を修正する[#3557](https://github.com/tikv/tikv/pull/3557)
    * リージョンのマージ後に関連するフォロワーの統計を修正する[#3573](https://github.com/tikv/tikv/pull/3573)
    * ローカルリーダーが過去のリージョン情報を使用する問題を修正する[#3565](https://github.com/tikv/tikv/pull/3565)