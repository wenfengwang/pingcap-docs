---
title: TiDB 3.0 Betaのリリースノート
aliases: ['/docs/dev/releases/release-3.0-beta/','/docs/dev/releases/3.0beta/']
---

# TiDB 3.0 Betaのリリースノート

2019年1月19日、TiDB 3.0 Betaがリリースされました。対応するTiDB Ansible 3.0 Betaもリリースされています。TiDB 3.0 Betaは、TiDB 2.1をベースに安定性、SQLオプティマイザ、統計、実行エンジンの改善に重点を置いています。

## TiDB

+ 新機能
    - ビューのサポート
    - ウィンドウ関数のサポート
    - 範囲パーティショニングのサポート
    - ハッシュパーティショニングのサポート
+ SQLオプティマイザ
    - `AggregationElimination`の最適化ルールを再サポート [#7676](https://github.com/pingcap/tidb/pull/7676)
    - `NOT EXISTS`サブクエリを最適化し、Anti Semi Joinに変換する [#7842](https://github.com/pingcap/tidb/pull/7842)
    - 新しいCascadesオプティマイザをサポートするための`tidb_enable_cascades_planner`変数を追加。現在、Cascadesオプティマイザは完全に実装されておらず、デフォルトでオフになっています [#7879](https://github.com/pingcap/tidb/pull/7879)
    - トランザクションでのIndex Joinのサポートを追加 [#7877](https://github.com/pingcap/tidb/pull/7877)
    - アウタージョインの定数の伝搬を最適化し、Join結果のアウターテーブルに関連するフィルタリング条件をアウタージョインを通じてアウターテーブルにプッシュダウンできるようにし、アウタージョインの無駄な計算を減らし、実行パフォーマンスを向上させる [#7794](https://github.com/pingcap/tidb/pull/7794)
    - プロジェクション削除の最適化ルールを、Aggregation Eliminationの位置の後に調整し、冗長な`Project`演算子を回避するために [#7909](https://github.com/pingcap/tidb/pull/7909)
    - `IFNULL`関数を最適化し、入力パラメーターが非NULL属性を持つ場合はこの関数を除外するように [#7924](https://github.com/pingcap/tidb/pull/7924)
    - `_tidb_rowid`構築クエリでのRangeのサポートを追加し、フルテーブルスキャンを回避し、クラスターの負荷を軽減するためのクエリ構築をサポート [#8047](https://github.com/pingcap/tidb/pull/8047)
    - `IN`サブクエリを最適化して、集約後にInner Joinを行い、この最適化ルールを有効にするかどうかを制御するための`tidb_opt_insubq_to_join_and_agg`変数を追加し、デフォルトでオープンにする [#7531](https://github.com/pingcap/tidb/pull/7531)
    - `DO`ステートメントでサブクエリの使用をサポート [#8343](https://github.com/pingcap/tidb/pull/8343)
    - アウタージョインの削除の最適化ルールを追加して、不要なテーブルスキャンとJoin演算を減らし、実行パフォーマンスを向上させる [#8021](https://github.com/pingcap/tidb/pull/8021)
    - `TIDB_INLJ`オプティマイザのヒント動作を変更し、オプティマイザはヒントで指定されたテーブルをIndex Joinの内部テーブルとして使用するようになる [#8243](https://github.com/pingcap/tidb/pull/8243)
    - 実行計画キャッシュの`Prepare`ステートメントが有効になる場合に`PointGet`を広範囲に使用できるようにし、実行計画キャッシュの`Prepare`ステートメントが有効になる場合に使用できるようにする [#8108](https://github.com/pingcap/tidb/pull/8108)
    - 複数のテーブルを結合する場合の結合順序選択を最適化するための貪欲な`Join Reorder`アルゴリズムを導入する [#8394](https://github.com/pingcap/tidb/pull/8394)
    - ビューのサポートを追加 [#8757](https://github.com/pingcap/tidb/pull/8757)
    - ウィンドウ関数のサポートを追加 [#8630](https://github.com/pingcap/tidb/pull/8630)
    - `TIDB_INLJ`が有効にならない場合にクライアントに警告を返し、使いやすさを向上させる [#9037](https://github.com/pingcap/tidb/pull/9037)
    - フィルタリング条件とテーブル統計に基づいてフィルタリングされたデータの統計の推論をサポート [#7921](https://github.com/pingcap/tidb/pull/7921)
    - Range PartitionのPartition Pruningの最適化ルールを改善する [#8885](https://github.com/pingcap/tidb/pull/8885)
+ SQLエグゼキュータ
    - 空の`ON`条件をサポートするために`Merge Join`演算子を最適化する [#9037](https://github.com/pingcap/tidb/pull/9037)
    - `EXECUTE`ステートメントの実行時に使用されるユーザー変数を記録し、出力するログを最適化する [#7684](https://github.com/pingcap/tidb/pull/7684)
    - `COMMIT`ステートメントの遅いクエリ情報を出力するためにログを最適化する [#7951](https://github.com/pingcap/tidb/pull/7951)
    - SQLチューニングプロセスを容易にするために`EXPLAIN ANALYZE`機能をサポートする [#7827](https://github.com/pingcap/tidb/pull/7827)
    - 多くの列を持つワイドテーブルの書き込みパフォーマンスを最適化する [#7935](https://github.com/pingcap/tidb/pull/7935)
    - `admin show next_row_id`をサポートする [#8242](https://github.com/pingcap/tidb/pull/8242)
    - 実行エンジンで使用される初期Chunkのサイズを制御するための`tidb_init_chunk_size`変数を追加する [#8480](https://github.com/pingcap/tidb/pull/8480)
    - `shard_row_id_bits`を改善し、auto-increment IDを相互チェックする [#8936](https://github.com/pingcap/tidb/pull/8936)
+ `Prepare`ステートメント
    - 異なるユーザ変数が入力された場合にクエリプランが正しいことを保証するために、サブクエリを含む`Prepare`ステートメントのクエリプランをクエリプランキャッシュに追加するのを禁止する [#8064](https://github.com/pingcap/tidb/pull/8064)
    - 確定的でない関数が含まれる場合にプランがキャッシュされることを保証するためにクエリプランキャッシュを最適化する [#8105](https://github.com/pingcap/tidb/pull/8105)
    - `DELETE`/`UPDATE`/`INSERT`のクエリプランがキャッシュされることを保証するためにクエリプランキャッシュを最適化する [#8107](https://github.com/pingcap/tidb/pull/8107)
    - `DEALLOCATE`ステートメントの実行時に対応するプランを削除するためにクエリプランキャッシュを最適化する [#8332](https://github.com/pingcap/tidb/pull/8332)
    - メモリ使用量を制限してTiDB OOM問題を回避するために、クエリプランキャッシュを最適化する [#8339](https://github.com/pingcap/tidb/pull/8339)
    - `ORDER BY`/`GROUP BY`/`LIMIT`句で`?`プレースホルダーを使用できるように`Prepare`ステートメントを最適化する [#8206](https://github.com/pingcap/tidb/pull/8206)
+ 権限管理
    - `ANALYZE`ステートメントの権限チェックを追加する [#8486](https://github.com/pingcap/tidb/pull/8486)
    - `USE`ステートメントの権限チェックを追加する [#8418](https://github.com/pingcap/tidb/pull/8418)
    - `SET GLOBAL`ステートメントの権限チェックを追加する [#8837](https://github.com/pingcap/tidb/pull/8837)
    - `SHOW PROCESSLIST`ステートメントの権限チェックを追加する [#7858](https://github.com/pingcap/tidb/pull/7858)
+ サーバー
    - `Trace`機能をサポートする [#9029](https://github.com/pingcap/tidb/pull/9029)
    - プラグインフレームワークをサポートする [#8788](https://github.com/pingcap/tidb/pull/8788)
    - `unix_socket`とTCPを同時に使用してデータベースに接続するサポートを追加する [#8836](https://github.com/pingcap/tidb/pull/8836)
    - `interactive_timeout`システム変数をサポートする [#8573](https://github.com/pingcap/tidb/pull/8573)
    - `wait_timeout`システム変数をサポートする [#8346](https://github.com/pingcap/tidb/pull/8346)
    - `tidb_batch_commit`変数を使用して、ステートメントの数に基づいてトランザクションを複数のトランザクションに分割することをサポートする [#8293](https://github.com/pingcap/tidb/pull/8293)
    - 遅いクエリを確認するために`ADMIN SHOW SLOW`ステートメントをサポートする [#7785](https://github.com/pingcap/tidb/pull/7785)
+ 互換性
    - `ALLOW_INVALID_DATES` SQLモードをサポートする [#9027](https://github.com/pingcap/tidb/pull/9027)
    - CSVファイルの`LoadData`のフォルトトレランスを向上する[#9005](https://github.com/pingcap/tidb/pull/9005)
    - MySQL 320ハンドシェイクプロトコルをサポートする [#8812](https://github.com/pingcap/tidb/pull/8812)

- `bigint` カラムを符号なしとして使用するオートインクリメントカラムをサポートする [#8181](https://github.com/pingcap/tidb/pull/8181)
- `SHOW CREATE DATABASE IF NOT EXISTS` 構文をサポートする [#8926](https://github.com/pingcap/tidb/pull/8926)
- フィルタリング条件にユーザ変数を含む場合は、述語のプッシュダウン操作を廃止し、MySQLのユーザ変数を使用してウィンドウ関数の振る舞いを模倣する互換性を向上させる [#8412](https://github.com/pingcap/tidb/pull/8412)
+ DDL
    - 誤って削除されたテーブルを素早く回復する機能をサポートする [#7937](https://github.com/pingcap/tidb/pull/7937)
    - `ADD INDEX` の並行数を動的に調整する機能をサポートする [#8295](https://github.com/pingcap/tidb/pull/8295)
    - テーブルやカラムの文字セットを `utf8`/`utf8mb4` に変更する機能をサポートする [#8037](https://github.com/pingcap/tidb/pull/8037)
    - デフォルトの文字セットを `utf8` から `utf8mb4` に変更する機能をサポートする [#7965](https://github.com/pingcap/tidb/pull/7965)
    - Range Partition をサポートする[#8011](https://github.com/pingcap/tidb/pull/8011)

## ツール

+ TiDB Lightning
    - SQL ステートメントを KV ペアに劇的に変換する処理を高速化する機能をサポートする [#110](https://github.com/pingcap/tidb-lightning/pull/110)
    - シングルテーブルのバッチインポートをサポートし、インポートのパフォーマンスと安定性を向上させる機能をサポートする [#113](https://github.com/pingcap/tidb-lightning/pull/113)

## PD

- `RegionStorage` を追加して、リージョンのメタデータを別途保存する機能を追加する [#1237](https://github.com/pingcap/pd/pull/1237)
- ホットなリージョンをシャッフルするスケジューラを追加する [#1361](https://github.com/pingcap/pd/pull/1361)
- スケジューリングパラメータに関連するメトリクスを追加する [#1406](https://github.com/pingcap/pd/pull/1406)
- クラスターラベルに関連するメトリクスを追加する [#1402](https://github.com/pingcap/pd/pull/1402)
- データのインポートシミュレータを追加する [#1263](https://github.com/pingcap/pd/pull/1263)
- リーダー選出に関する `Watch` の問題を修正する [#1396](https://github.com/pingcap/pd/pull/1396)

## TiKV

- 分散GCをサポートする [#3179](https://github.com/tikv/tikv/pull/3179)
- スナップショットの適用前にRocksDB Level 0ファイルをチェックしてWrite Stallを回避する機能を追加する [#3606](https://github.com/tikv/tikv/pull/3606)
- 逆向きの `raw_scan` と `raw_batch_scan` をサポートする [#3742](https://github.com/tikv/tikv/pull/3724)
- モニタリング情報を取得するためにHTTPを使用する機能をサポートする [#3855](https://github.com/tikv/tikv/pull/3855)
- DST のサポートを改善する [#3786](https://github.com/tikv/tikv/pull/3786)
- Raft メッセージの一括受信と送信をサポートする [#3931](https://github.com/tikv/tikv/pull/3913)
- 新しいストレージエンジン Titan を導入する [#3985](https://github.com/tikv/tikv/pull/3985)
- gRPC を v1.17.2 にアップグレードする [#4023](https://github.com/tikv/tikv/pull/4023)
- クライアントリクエストの一括受信と返信をサポートする [#4043](https://github.com/tikv/tikv/pull/4043)
- マルチスレッド Apply をサポートする [#4044](https://github.com/tikv/tikv/pull/4044)
- マルチスレッド Raftstore をサポートする [#4066](https://github.com/tikv/tikv/pull/4066)