---
title: TiDB 3.0 GAリリースノート
aliases: ['/docs/dev/releases/release-3.0-ga/','/docs/dev/releases/3.0-ga/']
---

# TiDB 3.0 GAリリースノート

リリース日: 2019年6月28日

TiDBバージョン: 3.0.0

TiDB Ansibleバージョン: 3.0.0

## 概要

2019年6月28日に、TiDB 3.0 GAがリリースされました。対応するTiDB Ansibleバージョンは3.0.0です。TiDB 2.1と比較して、このリリースは以下の点で大幅に改善されています。

- 安定性。 TiDB 3.0は、150+ノードおよび300+ TBのストレージを持つ大規模クラスタにおいて長期間の安定性を示しました。
- 利便性。 TiDB 3.0は、標準化された遅いクエリログ、よく開発されたログファイル仕様、および`EXPLAIN ANALYZE`およびSQLトレースなどの新機能を含む、利便性の向上が多岐にわたります。
- パフォーマンス。 TiDB 3.0のパフォーマンスは、TPC-CベンチマークにおいてTiDB 2.1の約4.5倍、Sysbenchベンチマークにおいて1.5倍以上に向上しています。Viewsのサポートにより、TPC-H 50G Q15は正常に実行できるようになりました。

## TiDB

+ 新機能
    - Window Functionsのサポート；MySQL 8.0のすべてのWindow Functionsに互換性があり、`NTILE`、`LEAD`、`LAG`、`PERCENT_RANK`、`NTH_VALUE`、`CUME_DIST`、`FIRST_VALUE`、`LAST_VALUE`、`RANK`、`DENSE_RANK`、`ROW_NUMBER`を含みます。
    - Viewsのサポート（**実験的**）
    - テーブルパーティションの改善
        - Range Partitionのサポート
        - Hash Partitionのサポート
    - IPホワイトリスト（**Enterprise**）や監査ログ（**Enterprise**）などのプラグインをサポートするプラグインフレームワークの追加
    - クエリの安定性を確保するためのSQL Plan Management機能のサポート（**実験的**）
+ SQLオプティマイザ
    - `NOT EXISTS`サブクエリを最適化し、`Anti Semi Join`に変換してパフォーマンスを向上させる
    - `Outer Join`における定数の伝播を最適化し、`Outer Join`除去の最適化ルールを追加して無効な計算を減らし、パフォーマンスを向上させる
    - `IN`サブクエリを最適化し、集計後に`Inner Join`を実行してパフォーマンスを向上させる
    - より多くのシナリオに適応するように`Index Join`を最適化
    - Range Partitionのパーティションプルーニングの最適化ルールを改善
    - `_tidb_rowid`のクエリロジックを最適化し、全表スキャンを回避してパフォーマンスを向上させる
    - フィルタに関連する列がある場合、複合インデックスのアクセス条件からプレフィックス列をより一致させ、パフォーマンスを向上させる
    - 列間の順序相関を使用してコスト見積もりの精度を向上させる
    - 貪欲法と動的プログラミングアルゴリズムに基づく`Join Order`を最適化し、複数のテーブルの結合操作を高速化する
    - Skyline Pruningをサポートし、統計情報への過度な依存を防ぐためのいくつかのルールを追加し、クエリの安定性を向上させる
    - NULL値を持つ単一列インデックスの行数見積もりの精度を向上させる
    - 全地域でランダムにサンプリングしてフル表スキャンを回避し、統計情報の収集によるパフォーマンスを向上させる`FAST ANALYZE`をサポート
    - 単調増加のインデックス列での増分分析操作をサポートし、統計情報の収集によるパフォーマンスを向上させる
    - `DO`ステートメントでのサブクエリの使用をサポート
    - トランザクションでの`Index Join`をサポート
    - パラメータのないDDLステートメントをサポートするために`prepare`/`execute`を最適化
    - `stats-lease`変数の値が0の場合に統計情報を自動的にロードするシステムの動作を変更
    - 過去の統計情報のエクスポートをサポート
    - ヒストグラムの`dump`/`load`の関連付けをサポート
+ SQL実行エンジン
    - `EXECUTE`はユーザー変数を出力し、`COMMIT`は遅いクエリログを出力してトラブルシューティングを容易にするようにログ出力を最適化
    - SQLチューニングの利便性を向上させるために`EXPLAIN ANALYZE`機能をサポート
    - 次の行IDを取得するための`admin show next_row_id`コマンドをサポート
    - `JSON_QUOTE`、`JSON_ARRAY_APPEND`、`JSON_MERGE_PRESERVE`、`BENCHMARK`、`COALESCE`、`NAME_CONST`の6つの組み込み関数を追加
    - クエリコンテキストに基づいてダイナミックに調整されるようにチャンクサイズの制御ロジックを最適化し、SQLの実行時間とリソース消費を削減
    - `TableReader`、`IndexReader`、および`IndexLookupReader`のメモリ使用量を追跡および制御するためのサポート
    - 空の`ON`条件をサポートするためにMerge Join演算子を最適化
    - 多くの列を含む単一のテーブルの書き込みパフォーマンスを最適化
    - 逆順でデータをスキャンすることで、`admin show ddl jobs`のパフォーマンスを向上させる
    - ホットスポットの問題を和らげるためにテーブルRegionを手動で分割するための`split table region`ステートメントを追加
    - ホットスポットの問題を和らげるためにインデックスRegionを手動で分割するための`split index region`ステートメントを追加
    - Coprocessorに式をプッシュダウンすることを禁止するブロックリストを追加
    - 構成された実行時間またはメモリの制限を超えた場合、`Expensive Query`ログでSQLクエリを出力するように最適化
+ DDL
    - `utf8`から`utf8mb4`への文字セットのマイグレーションをサポート
    - デフォルトの文字セットを`utf8`から`utf8mb4`に変更
    - データベースの文字セットと照合順序を変更するための`alter schema`ステートメントを追加
    - ALTERアルゴリズム`INPLACE`/`INSTANT`をサポート
    - `SHOW CREATE VIEW`をサポート
    - `SHOW CREATE USER`をサポート
    - 誤って削除されたテーブルを素早く復旧する機能をサポート
    - ADD INDEXの同時実行数を動的に調整する機能をサポート
    - `CREATE TABLE`ステートメントを使用してテーブルを作成する際に、書き込み後の多くの書き込みによって引き起こされるホットスポットの問題を和らげるための`pre_split_regions`オプションを追加
    - SQLステートメントを使用して指定されたテーブルのインデックスおよび範囲でのRegionを分割し、ホットスポットの問題を和らげるための機能を追加
    - DDLタスクのリトライ回数を制限するための`ddl_error_count_limit`グローバル変数を追加
    - `AUTO_INCREMENT`属性を持つ列に`SHARD_ROW_ID_BITS`を使用して行IDを分散させる機能を追加
    - 無効なDDLメタデータの寿命を最適化し、TiDBクラスタをアップグレードした後にDDL操作の正常な実行を高速に復旧させる
+ トランザクション
    - 悲観的トランザクションモードのサポート（**実験的**）
    - より多くのシナリオに適応するためにトランザクション処理ロジックを最適化：
        - 自動コミットされないトランザクションは自動的に再試行されないことを意味する`tidb_disable_txn_auto_retry`のデフォルト値を`on`に変更
        - 1つのトランザクションを複数のトランザクションに分割して並行して実行するための`tidb_batch_commit`システム変数を追加
        - 一貫性の要求が比較的低いシナリオでのパフォーマンスを向上させるために`tidb_low_resolution_tso`システム変数を追加
        - 分離レベルが`SERIALIZABLE`に設定された場合にエラーを報告するかどうかを制御するための`tidb_skip_isolation_level_check`変数を追加
        - `tidb_disable_txn_auto_retry`システム変数を修正して再試行可能なエラーで機能するようにする
+ 権限管理
    - `ANALYZE`、`USE`、`SET GLOBAL`、および`SHOW PROCESSLIST`ステートメントに対する権限チェックを実行
    - ロールベースのアクセス制御（RBAC）のサポート（**実験的**）
+ サーバ
    - 遅いクエリログを最適化：
        - ログ形式を再構築
        - ログコンテンツを最適化
        - メモリーテーブルの`INFORMATION_SCHEMA.SLOW_QUERY`ステートメントや`ADMIN SHOW SLOW`ステートメントを使用して遅いクエリログをクエリするためのログクエリメソッドを最適化
    - ツールによる収集および分析を容易にする再構築されたログシステムに統一されたログ形式仕様の開発
    - TiDB Binlogサービスを管理するためのSQLステートメントの使用をサポートし、TiDB Binlogの有効化、メンテナンス、および送信戦略の実行を可能にする
    - `unix_socket`を使用してデータベースに接続することをサポート
    - SQLステートメントの`Trace`をサポート
    - トラブルシューティングを容易にするために、`/debug/zip` HTTPインターフェースを介してTiDBインスタンスの情報を取得する機能をサポート
    - トラブルシューティングを容易にするためにモニタリングアイテムを最適化：
        - 統計に基づいて推定されたデータ容量と実際のデータ容量の違いを監視する`high_error_rate_feedback_total`モニタリングアイテムを追加
        - データベースの次元でQPSを監視するアイテムを追加
    - DDL所有者のみが初期化を実行できるようにシステムの初期化プロセスを最適化。これにより初期化またはアップグレードの起動時間が短縮される
    - リソースが適切に解放されることを保証するために`kill query`の実行ロジックを最適化
    - 構成ファイルの有効性を確認するための起動オプション`config-check`を追加
    - 内部エラーのリトライのバックオフ時間を制御するための`tidb_back_off_weight`システム変数を追加
    - 最大アイドル接続数を制御するための`wait_timeout`および`interactive_timeout`システム変数を追加
    - TiKVの接続確立時間を短縮するために、接続プールを追加
+ 互換性
    - `ALLOW_INVALID_DATES` SQLモードのサポート
    - MySQL 320 Handshakeプロトコルのサポート
    - 無符号BIGINTカラムを自動増加カラムとして表現するマニフェストのサポート
    - `SHOW CREATE DATABASE IF NOT EXISTS` 構文のサポート
    - CSVファイルの`load data`の障害耐性の最適化
    - フィルタリング条件にユーザー変数が含まれる場合、ウィンドウ関数をシミュレートするためのMySQLの振る舞いとの互換性を向上させるために、プレディケートの押し込み操作を放棄

## PD

+ シングルノードからクラスターを再作成するサポート
+ 大規模クラスターのストレージボトルネックを解決するために、etcdからgo-leveldbストレージエンジンに地域メタデータを移行
+ API
    - `remove-tombstone` APIの追加でTombstoneストアをクリア
    - `ScanRegions` APIの追加で地域情報をバッチでクエリ
    - `GetOperator` APIの追加で実行中のオペレータをクエリ
    - `GetStores` APIのパフォーマンスの最適化
+ 設定
    - 設定項目のエラーを回避するために、構成チェックロジックを最適化
    - リージョンマージの方向を制御するための `enable-two-way-merge` の追加
    - ホットリージョンのスケジューリング率を制御するための `hot-region-schedule-limit` の追加
    - 連続的に複数の閾値をヒットした場合にホットスポットを識別するための `hot-region-cache-hits-threshold` の追加
    - 1分あたりに許可される最大のバランスリージョンオペレータ数を制御するための `store-balance-rate` 設定項目の追加
+ スケジューラの最適化
    - 各ストアのオペレータ速度を別々に制御するためのストア制限メカニズムの追加
    - 異なるスケジューラ間でのリソース競合を最適化するための `waitingOperator` キューのサポート
    - アクティブにスケジューリング操作をTiKVに送信するためのスケジュールレート制限のサポート。これにより単一ノード上での同時スケジューリングタスクの数を制限することで、スケジューリング率が向上
    - `Region Scatter`のスケジューリングを制限メカニズムに拘束されないように最適化
    - ホットスポットのスケジューリングの助けとなる `shuffle-hot-region` スケジューラの追加
+ シミュレータ
    - データインポートシナリオのためのシミュレータの追加
    - ストアの異なるハートビート間隔のサポート
+ その他
    - 不整合なログ出力フォーマット、prevoteでのリーダー選出の失敗、リースデッドロックの解決のためにetcdのアップグレード
    - 道具による収集と分析を容易にするために、再構築されたログシステムで統一されたログフォーマット仕様の開発
    - スケジューリングパラメータ、クラスタラベル情報、PDがTSOリクエストを処理するために消費される時間、ストアID、アドレス情報を含む監視メトリクスの追加

## TiKV

+ 分散GCのサポートと並行ロック解決の改良によるGC性能の向上
+ 逆 `raw_scan` と `raw_batch_scan` のサポート
+ 単一ノード内の拡張性、並行性キャパシティ、リソース使用率を向上させるためのマルチスレッドRaftstoreとマルチスレッドApplyのサポート。同じ圧力下での性能が70%向上
+ Raftメッセージのバッチ受信および送信のサポートにより、ライト集中型シナリオのTPS向上7%
+ 書き込みスタールを避けるために、スナップショットを適用する前にRocksDBレベル0ファイルをチェック
+ 値のサイズが1KiBを超えるシナリオでの書き込み性能向上と一部程度の書き込み増加を軽減するキーバリュープラグイン、Titanの導入
+ 悲観的トランザクションモードのサポート（**実験的**）
+ HTTP経由での監視情報の取得のサポート
+ `Insert` のセマンティクスを変更し、Keyが存在しない場合にのみPrewriteが成功するように
+ 設定情報およびキーの境界を示すパフォーマンスメトリクスの追加
+ メモリアロケーションおよび`Iterator Key Bound Option`のメモリ割り当てとコピーを削減するためのメモリ管理の最適化
+ 異なるカラムファミリ間での`block cache`共有サポート
+ `batch commands`からのコンテキストスイッチオーバーヘッドの削減
+ `txn scheduler`の削除
+ `read index`および`GC worker`に関連する監視項目の追加
+ RaftStore
    - RaftStoreからのCPU消費を最適化するためのヒベル化地域のサポート（**実験的**）
    - ローカルリーダースレッドの削除
+ Coprocessor
    - ベクトル演算子の実装、ベクトル式を用いた計算、およびパフォーマンス向上のためのベクトル集約を実装するために、計算フレームワークを再構築
    - TiDBで`EXPLAIN ANALYZE`文のためにオペレータ実行ステータスを提供
    - コンテキストスイッチコストを削減するために、`work-stealing`スレッドプールモデルにスイッチ

## ツール

+ TiDB Lightning
    - データテーブルのリダイレクトレプリケーションのサポート
    - CSVファイルのインポートのサポート
    - SQLからKVペアへの変換のためのパフォーマンスの向上
    - パフォーマンスの向上のためのシングルテーブルのバッチインポートのサポート
    - 大きなテーブルのデータおよびインデックスの別々のインポートのサポートでTiKV-importerのパフォーマンスの向上
    - 新しいファイルでカラムデータが欠落している場合、`row_id`またはデフォルトのカラム値を使用して欠落しているカラムを補うサポート
    - SSTファイルをTiKVにアップロードする際に、`TIKV-importer`で速度制限を設定するサポート
+ TiDB Binlog
    - Drainerでの`advertise-addr`構成の追加により、コンテナ環境でのブリッジモードのサポート
    - Pumpでの`GetMvccByEncodeKey`関数の追加により、トランザクションステータスのクエリを高速化するサポート
    - ネットワークリソース消費を削減するために、コンポーネント間の通信データの圧縮のサポート
    - Kafkaからbinlogを読み込み、データをMySQLにレプリケートするためのArbiterツールの追加
    - レパロによるレプリケーションが不要なファイルの除外をサポート
    - 生成されたカラムのレプリケーションのサポート
    - 異なるSQLモードを使用してDDLクエリを解析するサポートするための`syncer.sql-mode`構成項目の追加
    - レプリケーション対象外のテーブルをフィルタリングするための`syncer.ignore-table`構成項目の追加
+ sync-diff-inspector
    - チェックポイントサポートで計算されたチェックサムによりデータの一貫性をチェックするための`only-use-checksum`構成項目の追加
    - TiDB統計情報および複数のカラムを使用してチャンクを分割するための機能の追加により、検証状態を記録し、最後に保存されたポイントから検証を継続

## TiDB Ansible

- 次のモニタリングコンポーネントを安定版にアップグレード
    - PrometheusのV2.2.1からV2.8.1
    - PushgatewayのV0.4.0からV0.7.0
    - Node_exporterのV0.15.2からV0.17.0
    - AlertmanagerのV0.14.0からV0.17.0
    - GrafanaのV4.6.3からV6.1.6
    - AnsibleのV2.5.14からV2.7.11
- クラスターステータスを便利に表示するためのTiKVサマリモニタリングダッシュボードの追加
- 重複項目の除去とトラブルシューティングを容易にするためのTiKVトラブルシューティングモニタリングダッシュボードの追加
- デバッグとトラブルシューティングを容易にするためのTiKV詳細モニタリングダッシュボードの追加
- バージョンの一貫性を並行してチェックするための同時チェックの追加により、更新パフォーマンスを向上
- TiDB Lightningのデプロイおよび操作のサポート
- テーブルごとのリーダー分布を表示するために`table-regions.py`スクリプトの最適化
- TiDBモニタリングの最適化およびSQLカテゴリに関連する遅延に関連する監視項目の追加
- CentOS 7.0+およびRed Hat 7.0+のみをサポートするようにオペレーティングシステムバージョンの制限を変更
- クラスターの最大QPSを予測する監視項目の追加（デフォルトでは非表示）