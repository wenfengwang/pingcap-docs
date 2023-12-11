---
title：TiDB 4.0.8リリースノート
---

# TiDB 4.0.8リリースノート

リリース日：2020年10月30日

TiDBバージョン：4.0.8

## 新機能

+ TiDB

    - 新しい集約関数`APPROX_PERCENTILE`をサポート [#20197](https://github.com/pingcap/tidb/pull/20197)

+ TiFlash

    - `CAST`機能をプッシュダウンするサポート

+ ツール

    + TiCDC

        - スナップショットレベルの一貫性のあるレプリケーションをサポート [#932](https://github.com/pingcap/tiflow/pull/932)

## 改善

+ TiDB

    - `Selectivity()`の貪欲な検索手順で低選択性インデックスを優先する [#20154](https://github.com/pingcap/tidb/pull/20154)
    - Coprocessorランタイム統計情報により多くのRPCランタイム情報を記録する [#19264](https://github.com/pingcap/tidb/pull/19264)
    - クエリパフォーマンスを向上させるために遅いログのパースを高速化する [#20556](https://github.com/pingcap/tidb/pull/20556)
    - SQLオプティマイザが潜在的な新しいプランを検証する際にタイムアウト実行プランを待機し、より多くのデバッグ情報を記録する [#20530](https://github.com/pingcap/tidb/pull/20530)
    - 遅いログと遅いクエリ結果に実行リトライ時間を追加する [#20495](https://github.com/pingcap/tidb/pull/20495) [#20494](https://github.com/pingcap/tidb/pull/20494)
    - `table_storage_stats`システムテーブルを追加する [#20431](https://github.com/pingcap/tidb/pull/20431)
    - `INSERT`/`UPDATE`/`REPLACE`ステートメントのためのRPCランタイム統計情報を追加する [#20430](https://github.com/pingcap/tidb/pull/20430)
    - `EXPLAIN FOR CONNECTION`の結果にオペレータ情報を追加する [#20384](https://github.com/pingcap/tidb/pull/20384)
    - クライアント接続/切断アクティビティ用にTiDBエラーログを`DEBUG`レベルに調整する [#20321](https://github.com/pingcap/tidb/pull/20321)
    - Coprocessor Cacheの監視メトリクスを追加する [#20293](https://github.com/pingcap/tidb/pull/20293)
    - 悲観的ロックキーのランタイム情報を追加する [#20199](https://github.com/pingcap/tidb/pull/20199)
    - ランタイム情報と `trace`スパンに時間消費情報の2つの追加セクションを追加する [#20187](https://github.com/pingcap/tidb/pull/20187)
    - 遅いログにトランザクションコミットのランタイム情報を追加する [#20185](https://github.com/pingcap/tidb/pull/20185)
    - インデックスマージジョインを無効にする [#20599](https://github.com/pingcap/tidb/pull/20599)
    - ISO 8601およびタイムゾーンのサポートを時間文字列リテラルに追加する [#20670](https://github.com/pingcap/tidb/pull/20670)

+ TiKV

    - パフォーマンス診断を支援する**Fast-Tune**パネルページを追加する [#8804](https://github.com/tikv/tikv/pull/8804)
    - ユーザーデータをログから取り除く`security.redact-info-log`構成項目を追加する [#8746](https://github.com/tikv/tikv/pull/8746)
    - エラーコードのメタファイルをリフォーマットする [#8877](https://github.com/tikv/tikv/pull/8877)
    - `pessimistic-txn.pipelined`構成を動的に変更可能にする [#8853](https://github.com/tikv/tikv/pull/8853)
    - メモリプロファイリング機能をデフォルトで有効にする [#8801](https://github.com/tikv/tikv/pull/8801)

+ PD

    - エラーのメタファイルを生成する [#3090](https://github.com/pingcap/pd/pull/3090)
    - オペレータに追加情報を追加する [#3009](https://github.com/pingcap/pd/pull/3009)

+ TiFlash

    - Raftログの監視メトリクスを追加する
    - `cop`タスクのメモリ使用量の監視メトリクスを追加する
    - データが削除された場合に`min`/`max`インデックスをより正確にする
    - 小規模なデータボリュームの場合のクエリパフォーマンスを向上する
    - 標準エラーコードをサポートする`errors.toml`ファイルを追加する

+ ツール

    + バックアップとリストア（BR）

        - `split`と`ingest`をパイプライン化してリストアプロセスを高速化する [#427](https://github.com/pingcap/br/pull/427)
        - PDスケジューラを手動でリストアするサポートを追加する [#530](https://github.com/pingcap/br/pull/530)
        - `remove`スケジューラではなく`pause`スケジューラを使用するサポートを追加する [#551](https://github.com/pingcap/br/pull/551)

    + TiCDC

        - 定期的にMySQLシンクで統計情報を出力する [#1023](https://github.com/pingcap/tiflow/pull/1023)

    + ダンプリング

        - S3ストレージにデータを直接ダンプするサポートを追加する [#155](https://github.com/pingcap/dumpling/pull/155)
        - ビューのダンプをサポートする [#158](https://github.com/pingcap/dumpling/pull/158)
        - 生成された列のみを含むテーブルのダンプをサポートする [#166](https://github.com/pingcap/dumpling/pull/166)

    + TiDB Lightning

        - マルチバイトCSV区切り文字とセパレータをサポートする [#406](https://github.com/pingcap/tidb-lightning/pull/406)
        - 一部のPDスケジューラを無効にすることでリストアプロセスを高速化するサポートを追加する [#408](https://github.com/pingcap/tidb-lightning/pull/408)
        - v4.0クラスタでGCエラーを回避するためにGC-TTL APIを使用するサポートを追加する [#396](https://github.com/pingcap/tidb-lightning/pull/396)

## バグ修正

+ TiDB

    - パーティションされたテーブルを使用する際に予期せぬパニックが発生する問題を修正する [#20565](https://github.com/pingcap/tidb/pull/20565)
    - インデックスマージジョインを使用して外部結合をフィルタリングする際に誤った結果が修正する [#20427](https://github.com/pingcap/tidb/pull/20427)
    - データを`BIT`タイプに変換する際に、データが長すぎる場合`NULL`値が返される問題を修正する [#20363](https://github.com/pingcap/tidb/pull/20363)
    - `BIT`タイプカラムの破損したデフォルト値を修正する [#20340](https://github.com/pingcap/tidb/pull/20340)
    - `BIT`タイプを`INT64`タイプに変換する際にオーバーフローエラーが発生する可能性がある問題を修正する [#20312](https://github.com/pingcap/tidb/pull/20312)
    - ハイブリッドタイプカラムのプロパゲート列最適化の可能性がある誤った結果を修正する [#20297](https://github.com/pingcap/tidb/pull/20297)
    - プランキャッシュから古いプランを格納する際に予期しないパニックが発生する問題を修正する [#20246](https://github.com/pingcap/tidb/pull/20246)
    - `FROM_UNIXTIME`と`UNION ALL`を一緒に使用した場合に誤って結果が切り捨てられる問題を修正する [#20240](https://github.com/pingcap/tidb/pull/20240)
    - `Enum`タイプの値を`Float`タイプに変換した時に誤った結果が返される可能性がある問題を修正する [#20235](https://github.com/pingcap/tidb/pull/20235)
    - `RegionStore.accessStore`の可能性があるパニックを修正する [#20210](https://github.com/pingcap/tidb/pull/20210)
    - `BatchPointGet`で最大符号なし整数をソートする際に誤った結果が返される問題を修正する [#20205](https://github.com/pingcap/tidb/pull/20205)
    - `Enum`と`Set`の適合性が誤っている問題を修正する [#20364](https://github.com/pingcap/tidb/pull/20364)
    - 曖昧な`YEAR`変換の問題を修正する [#20292](https://github.com/pingcap/tidb/pull/20292)
    - **KVデュレーション**パネルに`store0`を含む場合に発生する誤って報告された結果の問題を修正する [#20260](https://github.com/pingcap/tidb/pull/20260)
    - `Float`タイプデータが`out of range`エラーに関係なく誤って挿入されるバグを修正する [#20252](https://github.com/pingcap/tidb/pull/20252)
    - 生成された列が誤った`NULL`値を処理しないバグを修正する [#20216](https://github.com/pingcap/tidb/pull/20216)
    - `YEAR`タイプのデータが範囲外の場合の正確でないエラー情報を修正 [#20170](https://github.com/pingcap/tidb/pull/20170)
    - 悲観的トランザクションのリトライ中に発生するかもしれない予期しない `invalid auto-id` エラーを修正 [#20134](https://github.com/pingcap/tidb/pull/20134)
    - `ALTER TABLE`を使用して `Enum`/`Set` タイプを変更する際に制約がチェックされない問題を修正する [#20046](https://github.com/pingcap/tidb/pull/20046)
    - 複数の演算子を使用して並行して実行する場合に記録される `cop` タスクの実行時情報が誤っている問題を修正する [#19947](https://github.com/pingcap/tidb/pull/19947)
    - 読み取り専用システム変数をセッション変数として明示的に選択できない問題を修正する [#19944](https://github.com/pingcap/tidb/pull/19944)
    - 重複した `ORDER BY` 条件がサブ最適な実行計画を引き起こす可能性がある問題を修正する [#20333](https://github.com/pingcap/tidb/pull/20333)
    - 生成されたメトリクスプロファイルがフォントサイズが最大許容値を超える場合に失敗する可能性がある問題を修正する [#20637](https://github.com/pingcap/tidb/pull/20637)

+ TiKV

    - 暗号化における mutex の競合が pd-worker がハートビートを遅延処理する原因となるバグを修正する [#8869](https://github.com/tikv/tikv/pull/8869)
    - メモリプロファイルが誤って生成される問題を修正する [#8790](https://github.com/tikv/tikv/pull/8790)
    - ストレージクラスが指定された場合に GCS でデータベースのバックアップに失敗する問題を修正する [#8763](https://github.com/tikv/tikv/pull/8763)
    - リージョンが再起動されるか新しく分割された場合に learner がリーダーを見つけられないバグを修正する [#8864](https://github.com/tikv/tikv/pull/8864)

+ PD

    - TiDB ダッシュボードの Key Visualizer が原因で PD が特定の状況でパニックを起こす可能性のあるバグを修正する [#3096](https://github.com/pingcap/pd/pull/3096)
    - PD ストアが10分以上ダウンした場合に PD がパニックを起こす可能性のあるバグを修正する [#3069](https://github.com/pingcap/pd/pull/3069)

+ TiFlash

    - ログメッセージ中の誤ったタイムスタンプの問題を修正する
    - マルチディスク TiFlash デプロイメント中に誤った容量が TiFlash レプリカの作成に失敗する問題を修正する
    - TiFlash が再起動後に破損したデータファイルに関するエラーを発生させる可能性のあるバグを修正する
    - TiFlash がクラッシュ後にディスクに破損したファイルを残す可能性のあるバグを修正する
    - プロキシが最新の Raft リース情報に追いつけない場合に learner 読み込み中にインデックス待ちが長時間かかる可能性のあるバグを修正する
    - プロキシが古い Raft ログをリプレイする際に、過剰な Region 状態情報を key-value エンジンに書き込む可能性のあるバグを修正する

+ Tools

    + バックアップとリストア (BR)

        - 復元中に `send on closed channel` パニックを修正する [#559](https://github.com/pingcap/br/pull/559)

    + TiCDC

        - GC セーフポイントの更新に失敗したことによる予期しない終了を修正する [#979](https://github.com/pingcap/tiflow/pull/979)
        - 不正な mod revision キャッシュによって予期しないタスク状態がフラッシュされる問題を修正する [#1017](https://github.com/pingcap/tiflow/pull/1017)
        - 予期しない空の Maxwell メッセージを修正する [#978](https://github.com/pingcap/tiflow/pull/978)

    + TiDB Lightning

        - 誤った列情報の問題を修正する [#420](https://github.com/pingcap/tidb-lightning/pull/420)
        - ローカルモードでリージョン情報の取得を再試行する際に発生する無限ループを修正する [#418](https://github.com/pingcap/tidb-lightning/pull/418)