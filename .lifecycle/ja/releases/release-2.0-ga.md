---
title: TiDB 2.0 リリースノート
aliases: ['/docs/dev/releases/release-2.0-ga/','/docs/dev/releases/2.0ga/']
---

# TiDB 2.0 リリースノート

2018年4月27日、TiDB 2.0 GA がリリースされました！ TiDB 1.0 と比較して、このリリースはMySQL互換性、SQLオプティマイザー、エグゼキューター、安定性において大幅な改善があります。

## TiDB

- SQLオプティマイザー
    - よりコンパクトなデータ構造を使用して統計情報のメモリ使用量を削減
    - tidb-serverプロセスの起動時に統計情報の読み込みを高速化
    - 統計情報を動的に更新するサポート [実験的]
    - コストモデルを最適化してより正確なクエリコスト評価を提供
    - `Count-Min Sketch`を使用してポイントクエリのコストをより正確に推定
    - インデックスの最大/最小関数に対するインデックス利用をサポート
    - `ストリーム集計`演算子を使用して、`GROUP BY` 句が空の場合に性能を向上
    - `MAX/MIN` 関数に対してインデックスを使用するサポート
    - 相関サブクエリの処理アルゴリズムを最適化して、より多くの種類の相関サブクエリをサポートし、`Left Outer Join` に変換
    - `IndexLookupJoin` を拡張して、インデックスプレフィックスの一致に使用
- SQL実行エンジン
    - チャンクアーキテクチャを使用してすべての演算子をリファクタリングし、解析クエリの実行性能を向上し、メモリ使用量を削減。TPC-Hベンチマーク結果で大幅な改善があります。
    - ストリーミング集計演算子のプッシュダウンをサポート
    - `Insert Into Ignore` ステートメントを最適化して、パフォーマンスを10倍以上向上
    - `Insert On Duplicate Key Update` ステートメントを最適化して、パフォーマンスを10倍以上向上
    - `Load Data` を最適化して、パフォーマンスを10倍以上向上
    - TiKVにさらにデータ型と関数をプッシュダウン
    - 物理演算子のメモリ使用量を計算し、メモリ使用量がしきい値を超えた場合に構成ファイルとシステム変数で処理動作を指定するサポート
    - 単一のSQLステートメントによるメモリ使用量の制限をサポートして、OOMのリスクを削減
    - CRUD操作で暗黙のRowIDを使用するサポート
    - ポイントクエリのパフォーマンスを向上
- サーバー
    - プロキシプロトコルをサポート
    - 監視メトリクスを追加し、ログをリファイン
    - 構成ファイルの検証をサポート
    - HTTP APIを介してTiDBパラメータの情報を取得するサポート
    - バッチモードのロックを解決してガベージコレクションを高速化
    - マルチスレッドのガベージコレクションをサポート
    - TLSをサポート
- 互換性
    - より多くのMySQL構文をサポート
    - 構成ファイルで `lower_case_table_names` システム変数を変更して、OGGデータレプリケーションツールをサポート
    - Navicat管理ツールとの互換性を向上
    - `Information_Schema` でテーブル作成時間を表示するサポート
    - 一部の関数/式の返り値がMySQLと異なる問題を修正
    - JDBCとの互換性を向上
    - より多くのSQLモードをサポート
- DDL
    - `Add Index` 操作を最適化して、一部シナリオで実行速度を大幅に向上
    - `Add Index` 操作に低い優先度を割り当てて、オンラインビジネスへの影響を軽減
    - `Admin Show DDL Jobs` でDDLジョブの詳細ステータス情報を出力
    - `Admin Show DDL Job Queries JobID` を使用して、現在実行中のDDLジョブの元のステートメントをクエリするサポート
    - `Admin Recover Index` を使用してインデックスデータの復旧をサポート
    - `Alter` ステートメントを使用してテーブルオプションを変更するサポート

## PD

- `Region Merge` をサポートしてデータを削除した後の空のリージョンをマージ [実験的]
- `Raft Learner` をサポート [実験的]
- スケジューラを最適化
    - スケジューラを異なるリージョンサイズに適応させる
    - TiKVの停止時のデータ復元の優先度と速度を向上
    - TiKVノードの削除時のデータ転送を高速化
    - ディスクの空き容量が不足してTiKVノードがいっぱいにならないようスケジューリングポリシーを最適化
    - balance-leaderスケジューラのスケジューリング効率を向上
    - balance-regionスケジューラのスケジューリングオーバーヘッドを削減
    - hot-regionスケジューラの実行効率を最適化
- 操作インターフェースと構成
    - TLSをサポート
    - PDリーダーを優先設定するサポート
    - ラベルに基づいてスケジューリングポリシーを構成するサポート
    - 特定のラベルでストアを構成してRaftリーダーのスケジュールを行わないようにするサポート
    - 単一のリージョンにホットスポットを処理するために手動でリージョンを分割するサポート
    - 特定のリージョンを散らすことで、一部のケースでリージョンの分布を調整するサポート
    - 構成パラメータのチェックルールを追加し、構成項目の妥当性チェックを向上
- デバッグインターフェース
    - `Drop Region` デバッグインターフェースを追加
    - 各PDのヘルスステータスを列挙するインターフェースを追加
- 統計
    - 異常なリージョンに関する統計情報を追加
    - リージョンの分離レベルに関する統計情報を追加
    - スケジューリング関連のメトリクスを追加
- パフォーマンス
    - PDリーダーとetcdリーダーを同じノードに配置して書き込みパフォーマンスを向上
    - リージョンのスナップショットが完了するとすぐにPDに通知してバランスを速める
    - RocksDBのフラッシングによるパフォーマンスの揺らぎを解決
    - データ削除後のスペース回収メカニズムを最適化
    - サーバー起動時のガベージクリーニングを高速化
    - `DeleteFilesInRanges` を使用してレプリカの移行時のI/Oオーバーヘッドを削減

## TiKV

- 機能
    - 重要な構成を不正な修正から保護
    - `Region Merge` をサポート [実験的]
    - `Raw DeleteRange` APIを追加
    - `GetMetric` APIを追加
    - `Raw Batch Put`、`Raw Batch Get`、`Raw Batch Delete`、`Raw Batch Scan` を追加
    - RawKV APIのためのColumn Familyオプションを追加し、特定のColumn Familyで操作を実行するサポート
    - Coprocessorでのストリーミングとストリーミング集計をサポート
    - Coprocessorのリクエストタイムアウトを設定するサポート
    - リージョンハートビートでタイムスタンプを伴う
    - `block-cache-size` などの一部のRocksDBパラメータをオンラインで変更するサポート
    - Coprocessorが警告やエラーに遭遇した場合の動作を構成するサポート
    - データインポートの過程での書き込み増加を削減するようにデータインポートモードで起動するサポート
    - リージョンを手動で半分に分割するサポート
    - データ復旧ツール `tikv-ctl` を改善
    - Coprocessorでロックを返すための統計情報を返すサポート
    - `ImportSST` APIを使用してSSTファイルをインポートするサポート [実験的]
    - TiKV Importerバイナリを追加し、TiDB Lightningと統合してデータを迅速にインポートするサポート [実験的]
- パフォーマンス
    - `ReadPool` を使用して読み込みパフォーマンスを最適化し、`raw_get/get/batch_get` を30%向上
    - メトリクスのパフォーマンスを向上
    - Raftスナップショットプロセスが完了した場合はすぐにPDに通知してバランスを速める
    - RocksDBのフラッシングによるパフォーマンスの揺らぎを解決
    - データ削除後のスペース回収メカニズムを最適化
    - サーバー起動時のI/Oオーバーヘッドを削減
    - gRPCコールがPDリーダーを切り替えた場合に返されない問題を修正
    - スナップショットによる一時的なスペース使用量を制限
    - 長時間リーダーを選出できないリージョンを報告
    - コンパクションイベントに応じてリージョンサイズ情報をタイムリーに更新
    - リクエストタイムアウトを避けるためにスキャンロックのサイズを制限
    - OOMを避けるためにスナップショット受信時のメモリ使用量を制限
    - CIテストの速度を向上
    - スナップショットが多すぎることによるOOMの問題を修正
    - gRPCの `keepalive` を構成
    - リージョン数の増加によるOOMの問題を修正

## TiSpark

TiSparkは独自のバージョン番号を使用します。現在のTiSparkバージョンは1.0 GAです。TiSpark 1.0のコンポーネントは、Apache Sparkを使用してTiDBデータの分散コンピューティングを提供します。

- TiKVからデータを読み取るためのgRPC通信フレームワークを提供
- TiKVコンポーネントデータと通信プロトコルのエンコードとデコードを提供
- プッシュダウン計算を提供します:
    - 集約プッシュダウン
    - プレディケートプッシュダウン
    - TopNプッシュダウン
    - Limitプッシュダウン
- インデックス関連のサポートを提供
    - プレディケートをリージョンキー範囲または二次インデックスに変換
    - `Index Only` クエリを最適化
    - リージョンごとにインデックススキャンをテーブルスキャンに自動変更
- コストベースの最適化を提供
    - 統計情報をサポート
    - インデックスの選択
    - ブロードキャストテーブルコストを推定
- 複数のSparkインターフェースのサポートを提供
    - Spark Shellをサポート
    - ThriftServer/JDBCをサポート
    - Spark-SQLインタラクションをサポート
    - PySpark Shellをサポート
    - SparkRをサポート