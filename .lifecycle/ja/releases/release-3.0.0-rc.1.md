---
title: TiDB 3.0.0-rc.1 リリースノート
aliases: ['/docs/dev/releases/release-3.0.0-rc.1/','/docs/dev/releases/3.0.0-rc.1/']
---

# TiDB 3.0.0-rc.1 リリースノート

リリース日: 2019年5月10日

TiDB バージョン: 3.0.0-rc.1

TiDB Ansible バージョン: 3.0.0-rc.1

## 概要

2019年5月10日に、TiDB 3.0.0-rc.1がリリースされました。これに対応するTiDB Ansibleバージョンは3.0.0-rc.1です。このリリースでは、TiDB 3.0.0-beta.1と比較して、安定性、使いやすさ、機能、SQLオプティマイザ、統計情報、実行エンジンが大幅に改善されています。

## TiDB

+ SQLオプティマイザー
    - カラム間の順序相関を使用してコスト推定の精度を向上させる; ヒューリスティック・パラメータ`tidb_opt_correlation_exp_factor`を導入し、相関が推定に直接使用できないシナリオにおいてインデックススキャンを好むように制御する。 [#9839](https://github.com/pingcap/tidb/pull/9839)
    - 複合インデックスのアクセス条件を抽出する際、関連カラムがフィルターに存在する場合、インデックスのより多くの接頭辞カラムを一致させる。 [#10053](https://github.com/pingcap/tidb/pull/10053)
    - 結合に参加するテーブルの数が`tidb_opt_join_reorder_threshold`の値より少ない場合、動的計画法アルゴリズムを使用して結合操作の実行順序を指定する。 [#8816](https://github.com/pingcap/tidb/pull/8816)
    - 複合インデックスをアクセス条件として使用する際、インデックスジョインを構築する内部テーブルのインデックスのより多くの接頭辞カラムを一致させる。 [#8471](https://github.com/pingcap/tidb/pull/8471)
    - NULL値を持つ単一カラムインデックスの行数推定の精度を向上させる。 [#9474](https://github.com/pingcap/tidb/pull/9474)
    - 論理最適化フェーズで、`GROUP_CONCAT`を特別に処理して不正確な実行を防止する。 [#9967](https://github.com/pingcap/tidb/pull/9967)
    - フィルターが定数である場合、そのフィルターを結合演算子の子ノードに適切にプッシュダウンする。 [#9848](https://github.com/pingcap/tidb/pull/9848)
    - MySQLとの非互換性を防ぐため、論理最適化フェーズで`RAND()`などの特定の関数を列の剪定時に特別に処理する。[#10064](https://github.com/pingcap/tidb/pull/10064)
    - 変数`tidb_enable_fast_analyze`によって制御される、リージョンのサンプリングにより統計情報収集を高速化する`FAST ANALYZE`をサポートする。[#10258](https://github.com/pingcap/tidb/pull/10258)
    - SQLプラン管理をサポートし、SELECTステートメントのバインド実行プランを現在ベータ版で提供する。本番環境での使用は推奨されません。 [#10284](https://github.com/pingcap/tidb/pull/10284)

+ 実行エンジン
    - `TableReader`、`IndexReader`、`IndexLookupReader`の3つのオペレーターでメモリ使用量の追跡と制御をサポートする。 [#10003](https://github.com/pingcap/tidb/pull/10003)
    - コプロセッサータスクに関する詳細な情報（コプロセッサー内のタスク数、実行/待機時間の平均/最長/90%など）をスローログに表示する機能をサポートする。最も実行時間が長いまたは待機時間が長いTiKVのアドレスも表示する。 [#10165](https://github.com/pingcap/tidb/pull/10165)
    - プレースホルダーのない準備されたDDLステートメントをサポートする。 [#10144](https://github.com/pingcap/tidb/pull/10144)

+ サーバー
    - TiDB起動時にDDL所有者のみがブートストラップを実行できるようにする。 [#10029](https://github.com/pingcap/tidb/pull/10029)
    - トランザクション分離レベルをSERIALIZABLEに設定するとTiDBがエラーを報告するのを防ぐために変数`tidb_skip_isolation_level_check`を追加する。 [#10065](https://github.com/pingcap/tidb/pull/10065)
    - スローログ内で暗黙的コミット時間とSQL実行時間をマージする。 
        - SQLロール（RBAC権限管理）をサポートする
        - `SHOW GRANT`をサポートする。 [#10016](https://github.com/pingcap/tidb/pull/10016)
        - `SET DEFAULT ROLE`をサポートする。 [#9949](https://github.com/pingcap/tidb/pull/9949)
    - `GRANT ROLE`をサポートする。 [#9721](https://github.com/pingcap/tidb/pull/9721)
    - TiDBを終了させる`whitelist`プラグインからの`ConnectionEvent`エラーを修正する。 [#9889](https://github.com/pingcap/tidb/pull/9889)
    - 誤って読み取り専用ステートメントをトランザクション履歴に追加する問題を修正する。 [#9723](https://github.com/pingcap/tidb/pull/9723)
    - SQLの実行を停止しリソースを迅速に解放するように`kill`ステートメントを改善する。 [#9844](https://github.com/pingcap/tidb/pull/9844)
    - 設定ファイルの有効性をチェックするための起動オプション`config-check`を追加する。 [#9855](https://github.com/pingcap/tidb/pull/9855)
    - 厳密SQLモードが無効になっている場合、NULLフィールドの挿入の有効性チェックを修正する。 [#10161](https://github.com/pingcap/tidb/pull/10161)

+ DDL
    - `CREATE TABLE`ステートメントに`pre_split_regions`オプションを追加する。このオプションは、テーブル作成後の多くの書き込みによるホットスポットを避けるために、テーブルリージョンの事前分割をサポートする。 [#10138](https://github.com/pingcap/tidb/pull/10138)
    - 一部のDDLステートメントの実行パフォーマンスを最適化する。 [#10170](https://github.com/pingcap/tidb/pull/10170)
    - `FULLTEXT KEY`に対して全文検索インデックスがサポートされていないことを警告する。 [#9821](https://github.com/pingcap/tidb/pull/9821)
    - 古いバージョンのTiDBにおけるUTF8およびUTF8MB4文字セットの互換性の問題を修正する。 [#9820](https://github.com/pingcap/tidb/pull/9820)
    - `shard_row_id_bits`でのテーブルの潜在的なバグを修正する。 [#9868](https://github.com/pingcap/tidb/pull/9868)
    - テーブル文字セットが変更された後にカラム文字セットが変更されないバグを修正する。 [#9790](https://github.com/pingcap/tidb/pull/9790)
    - `BINARY`/`BIT`を列のデフォルト値として使用する場合の`SHOW COLUMN`での潜在的なバグを修正する。 [#9897](https://github.com/pingcap/tidb/pull/9897)
    - `SHOW FULL COLUMNS`ステートメントでの`CHARSET`/`COLLATION`の表示における互換性の問題を修正する。 [#10007](https://github.com/pingcap/tidb/pull/10007)
    - `SHOW COLLATIONS`ステートメントにおいてTiDBがサポートする照合順序のみがリストされる問題を修正する。 [#10186](https://github.com/pingcap/tidb/pull/10186)

## PD

+ ETCDのアップグレード [#1452](https://github.com/pingcap/pd/pull/1452)
    - etcdとPDサーバーのログ形式を統一する
    - PreVoteによるLeaderの選出に失敗する問題を修正する
    - ブロック後続リクエストを避けるために失敗する「提案」と「読み取り」リクエストを迅速に破棄する機能をサポートする
    - リースのデッドロック問題を修正する
+ ホットストアがキーの誤った統計を生成する問題を修正する。 [#1487](https://github.com/pingcap/pd/pull/1487)
+ 単一のPDノードからPDクラスターを強制的に再構築する機能をサポートする。 [#1485](https://github.com/pingcap/pd/pull/1485)
+ `regionScatterer`が無効な`OperatorStep`を生成する可能性のある問題を修正する。 [#1482](https://github.com/pingcap/pd/pull/1482)
+ `MergeRegion`オペレーターのタイムアウトが短すぎる問題を修正する。 [#1495](https://github.com/pingcap/pd/pull/1495)
+ ホットリージョンのスケジューリングに高い優先度を付与する機能をサポートする。 [#1492](https://github.com/pingcap/pd/pull/1492)
+ PDサーバーサイドでTSOリクエストの処理時間を記録するメトリクスを追加する。 [#1502](https://github.com/pingcap/pd/pull/1502)
+ ストアに関連するメトリクスに対応するストアIDとアドレスを追加する。 [#1506](https://github.com/pingcap/pd/pull/1506)
+ `GetOperator`サービスをサポートする。 [#1477](https://github.com/pingcap/pd/pull/1477)
+ [PingCAP/pd #1521](https://github.com/pingcap/pd/pull/1521)の問題として、Heartbeatストリームにエラーを送信できない問題をストアが見つからないために修正しました。

## TiKV

+ Engine
    - 誤った統計情報を読み取りトラフィックにもたらす可能性のある問題を修正しました [#4436](https://github.com/tikv/tikv/pull/4436)
    - 範囲を削除する際にprefix extractorがパニックを起こす可能性のある問題を修正しました [#4503](https://github.com/tikv/tikv/pull/4503)
    - `Iterator Key Bound Option`のためのメモリ管理を最適化して、メモリの割り当てやコピーを減らしました [#4537](https://github.com/tikv/tikv/pull/4537)
    - learnerログのギャップを考慮しないことがパニックを引き起こす可能性がある問題を修正しました [#4559](https://github.com/tikv/tikv/pull/4559)
    - 異なる`column families`の間で`block cache`を共有するサポートを追加しました [#4612](https://github.com/tikv/tikv/pull/4612)

+ Server
    - `batch commands`のコンテキストスイッチのオーバーヘッドを削減しました [#4473](https://github.com/tikv/tikv/pull/4473)
    - シークイテレータステータスの有効性を確認します [#4470](https://github.com/tikv/tikv/pull/4470)

+ RaftStore
    - 設定可能な`properties index distance`をサポートしました [#4517](https://github.com/tikv/tikv/pull/4517)

+ Coprocessor
    - バッチインデックススキャンエグゼキュータを追加しました [#4419](https://github.com/tikv/tikv/pull/4419)
    - ベクトル化された評価フレームワークを追加しました [#4322](https://github.com/tikv/tikv/pull/4322)
    - バッチエグゼキュータのための実行サマリーフレームワークを追加しました [#4433](https://github.com/tikv/tikv/pull/4433)
    - 評価パニックを引き起こす可能性のある無効な列オフセットを回避するために、RPN式の構築時に最大列をチェックします [#4481](https://github.com/tikv/tikv/pull/4481)
    - `BatchLimitExecutor`を追加しました [#4469](https://github.com/tikv/tikv/pull/4469)
    - コンテキストスイッチを減らすために、ReadPoolで元の`futures-cpupool`を`tokio-threadpool`に置き換えました [#4486](https://github.com/tikv/tikv/pull/4486)
    - バッチ集計フレームワークを追加しました [#4533](https://github.com/tikv/tikv/pull/4533)
    - `BatchSelectionExecutor`を追加しました [#4562](https://github.com/tikv/tikv/pull/4562)
    - バッチ集計関数 `AVG`を追加しました [#4570](https://github.com/tikv/tikv/pull/4570)
    - RPN関数 `LogicalAnd`を追加しました [#4575](https://github.com/tikv/tikv/pull/4575)

+ Misc
    - メモリ割り当てプログラムとして`tcmalloc`をサポートしました [#4370](https://github.com/tikv/tikv/pull/4370)

## Tools

+ TiDB Binlog
    - 符号なし整数型のプライマリキーカラムのバイナリログデータが負の場合に、レプリケーションの中断問題を修正しました [#573](https://github.com/pingcap/tidb-binlog/pull/573)
    - Downstreamが`pb`の場合、圧縮オプションを提供せず、Downstream名を`pb`から`file`に変更しました [#559](https://github.com/pingcap/tidb-binlog/pull/559)
    - Pumpでローカルストレージに非同期フラッシュを許可する`storage.sync-log`構成項目を追加しました [#509](https://github.com/pingcap/tidb-binlog/pull/509)
    - PumpとDrainer間の通信のトラフィック圧縮をサポートしました [#495](https://github.com/pingcap/tidb-binlog/pull/495)
    - Drainerに`syncer.sql-mode`構成項目を追加して、異なるsql-modeでDDLクエリを解析するサポートを追加しました [#511](https://github.com/pingcap/tidb-binlog/pull/511)
    - レプリケーションを必要としないテーブルをフィルタリングするための`syncer.ignore-table`構成項目を追加しました [#520](https://github.com/pingcap/tidb-binlog/pull/520)

+ Lightning
    - ダンプファイルで欠落している列データを埋めるために、行IDまたはデフォルト列値を使用しました [#170](https://github.com/pingcap/tidb-lightning/pull/170)
    - ImporterでSSTの一部がインポートに失敗しても、インポート成功が返されるバグを修正しました [#4566](https://github.com/tikv/tikv/pull/4566)
    - TiKVへSSTをアップロードする際のImporterで速度制限をサポートしました [#4412](https://github.com/tikv/tikv/pull/4412)
    - 大規模なテーブルのChecksumとAnalyzeによるクラスタへの影響を減らし、ChecksumとAnalyzeの成功率を向上させるために、サイズでテーブルをインポートするサポートを追加しました [#156](https://github.com/pingcap/tidb-lightning/pull/156)
    - KVエンコーダーからの余分なパースオーバーヘッドをTiDBのtypes.Datumとしてデータソースファイルを直接解析することで、LightningのSQLエンコーディングパフォーマンスを50%改善しました [#145](https://github.com/pingcap/tidb-lightning/pull/145)
    - ログフォーマットを[Unified Log Format](https://github.com/tikv/rfcs/blob/master/text/0018-unified-log-format.md)に変更しました [#162](https://github.com/pingcap/tidb-lightning/pull/162)
    - 構成ファイルがない場合に使用するコマンドラインオプションをいくつか追加しました [#157](https://github.com/pingcap/tidb-lightning/pull/157)

+ sync-diff-inspector
    - チェックサムを計算してデータの整合性を確認するための`only-use-checksum`構成項目を追加しました [#215](https://github.com/pingcap/tidb-tools/pull/215)
    - 状態を記録して、最後に保存されたポイントから検証を継続するためのチェックポイントをサポートしました [#224](https://github.com/pingcap/tidb-tools/pull/224)

## TiDB Ansible

+ TiKV監視パネルをサポートし、Ansible、Grafana、Prometheusのバージョンを更新しました [#727](https://github.com/pingcap/tidb-ansible/pull/727)
    - クラスターステータスを表示するためのサマリーダッシュボード
    - トラブルシューティングのためのトラブルシューティングダッシュボード
    - 開発者が問題を分析するための詳細ダッシュボード
+ TiDB BinlogのKafkaバージョンのダウンロードに失敗する原因を修正しました [#730](https://github.com/pingcap/tidb-ansible/pull/730)
+ 対応するオペレーティングシステムのバージョン制約をCentOS 7.0+およびそれ以降、およびRed Hat 7.0およびそれ以降に変更しました [#733](https://github.com/pingcap/tidb-ansible/pull/733)
+ ローリングアップデート時のバージョン検出モードをマルチコンカレントに変更しました [#736](https://github.com/pingcap/tidb-ansible/pull/736)
+ README内のドキュメントリンクを更新しました [#740](https://github.com/pingcap/tidb-ansible/pull/740)
+ 冗長なTiKV監視メトリクスを削除し、トラブルシューティングのための新しいメトリクスを追加しました [#735](https://github.com/pingcap/tidb-ansible/pull/735)
+ リーダーの分布を表す`table-regions.py`スクリプトを最適化しました [#739](https://github.com/pingcap/tidb-ansible/pull/739)
+ Drainerのための構成ファイルを更新しました [#745](https://github.com/pingcap/tidb-ansible/pull/745)
+ SQLカテゴリごとの遅延を表示する新しいパネルでTiDB監視を最適化しました [#747](https://github.com/pingcap/tidb-ansible/pull/747)
+ Lightningの構成ファイルを更新し、`tidb_lightning_ctl`スクリプトを追加しました [#1e946f8](https://github.com/pingcap/tidb-ansible/commit/1e946f89908e8fd6ef84128c6da3064ddfccf6a8)