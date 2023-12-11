---
title: TiDB 5.1.4 リリースノート
---

# TiDB 5.1.4 リリースノート

リリース日: 2022年2月22日

TiDB バージョン: 5.1.4

## 互換性の変更

+ TiDB

    - システム変数 `tidb_analyze_version` のデフォルト値を[`tidbのバージョン`](/system-variables.md#tidb_analyze_version-new-in-v510) から`2` に変更しました [#31748](https://github.com/pingcap/tidb/issues/31748)
    - v5.1.4以降、TiKVが`storage.enable-ttl = true`で構成されている場合、TiDBからのリクエストは拒否されます。なぜならTiKVのTTL機能は[RaftKVモード](https://tikv.org/docs/5.1/concepts/explore-tikv-features/ttl/)のみをサポートしているからです。 [#27303](https://github.com/pingcap/tidb/issues/27303)

+ ツール

    + TiCDC

        - `max-message-bytes` のデフォルト値を10Mに設定しました [#4041](https://github.com/pingcap/tiflow/issues/4041)

## 改善点

+ TiDB

    - 範囲パーティションテーブルにおいて、組み込みの `IN` 式のパーティションプルーニングをサポートしました [#26739](https://github.com/pingcap/tidb/issues/26739)
    - `IndexJoin` が実行される際のメモリ使用量の追跡精度を向上させました [#28650](https://github.com/pingcap/tidb/issues/28650)

+ TiKV

    - procファイルシステム（procfs）をv0.12.0に更新しました  [#11702](https://github.com/tikv/tikv/issues/11702)
    - Raftクライアントにおけるエラーログの報告を改善しました [#11959](https://github.com/tikv/tikv/issues/11959)
    - SSTファイルの挿入速度を向上させるため、検証プロセスを `Import`スレッドプールから `Apply`スレッドプールに移しました  [#11239](https://github.com/tikv/tikv/issues/11239)

+ PD

    - スケジューラーの終了プロセスを高速化しました  [#4146](https://github.com/tikv/pd/issues/4146)

+ TiFlash

    - `ADDDATE()`および`DATE_ADD()`のプッシュダウンをTiFlashにサポート
    - `INET6_ATON()`および`INET6_NTOA()`のプッシュダウンをTiFlashにサポート
    - `INET_ATON()`および`INET_NTOA()`のプッシュダウンをTiFlashにサポート
    - DAGリクエストにおける式またはプランツリーの最大サポート深度を、`100` から `200` に増やしました

+ ツール

    + TiCDC

        - changefeedの再起動に対する指数バックオフメカニズムを追加しました  [#3329](https://github.com/pingcap/tiflow/issues/3329)
        - 多くのテーブルを複製する際のレプリケーション遅延を減らしました  [#3900](https://github.com/pingcap/tiflow/issues/3900)
        - 増分スキャンの残り時間を観察するためのメトリクスを追加しました  [#2985](https://github.com/pingcap/tiflow/issues/2985)
        - "EventFeed retry rate limited"ログのカウントを減らしました  [#4006](https://github.com/pingcap/tiflow/issues/4006)
        - `no owner alert`, `mounter row`, `table sink total row`, `buffer sink total row`などのPrometheusとGrafanaモニタリングメトリクスと警告を追加しました  [#4054](https://github.com/pingcap/tiflow/issues/4054) [#1606](https://github.com/pingcap/tiflow/issues/1606)
        - changefeedの初期化中にgPRCの混雑を減らすために、TiKVリロードにおけるレート制限制御を最適化しました  [#3110](https://github.com/pingcap/ticdc/issues/3110)
        - TiKVストアがダウンした際にKVクライアントがリカバリーする時間を減らしました  [#3191](https://github.com/pingcap/tiflow/issues/3191)

## バグ修正

+ TiDB

    - システム変数 `tidb_analyze_version` が `2` に設定されている場合に発生するメモリリークのバグを修正しました  [#32499](https://github.com/pingcap/tidb/issues/32499)
    - `MaxDays`および`MaxBackups`の設定が遅いログに対して効果がない問題を修正しました  [#25716](https://github.com/pingcap/tidb/issues/25716)
    - `INSERT ... SELECT ... ON DUPLICATE KEY UPDATE` ステートメントを実行する際に発生するパニックの問題を修正しました  [#28078](https://github.com/pingcap/tidb/issues/28078)
    - `ENUM`型の列を`JOIN`で使用する際に起こりうる誤った結果を修正しました  [#27831](https://github.com/pingcap/tidb/issues/27831)
    - `INDEX HASH JOIN`が`send on closed channel` エラーを返す問題を修正しました  [#31129](https://github.com/pingcap/tidb/issues/31129)
    - [`BatchCommands`](/tidb-configuration-file.md#max-batch-size) APIを使用した場合にTiDBリクエストをTiKVに送信する際に、いくつかの稀なケースでブロックする可能性がある問題を修正しました  [#32500](https://github.com/pingcap/tidb/issues/32500)
    - 楽観的トランザクションモードにおける潜在的なデータインデックスの不整合の問題を修正しました  [#30410](https://github.com/pingcap/tidb/issues/30410)
    - トランザクションの使用の有無によって、ウィンドウ関数が異なる結果を返す問題を修正しました  [#29947](https://github.com/pingcap/tidb/issues/29947)
    - `Decimal` を `String` にキャストする際の長さ情報が誤っている問題を修正しました  [#29417](https://github.com/pingcap/tidb/issues/29417)
    - `tidb_enable_vectorized_expression` ベクトル式を `off` に設定した際に `GREATEST` 関数が誤った結果を返す問題を修正しました  [#29434](https://github.com/pingcap/tidb/issues/29434)
    - オプティマイザが一部の場合に`join`用に無効なプランをキャッシュする問題を修正しました  [#28087](https://github.com/pingcap/tidb/issues/28087)
    - ベクトル式における`microsecond`および`hour`関数の誤った結果を修正しました  [#29244](https://github.com/pingcap/tidb/issues/29244) [#28643](https://github.com/pingcap/tidb/issues/28643)
    - 一部の場合に`ALTER TABLE.. ADD INDEX` ステートメントを実行した際にTiDBがパニックする問題を修正しました  [#27687](https://github.com/pingcap/tidb/issues/27687)
    - MPPノードの可用性検出が一部のケースで機能しないバグを修正しました  [#3118](https://github.com/pingcap/tics/issues/3118)
    - `MPP task ID` の割り当て時に発生する`DATA RACE`の問題を修正しました  [#27952](https://github.com/pingcap/tidb/issues/27952)
    - 空の`dual table`を削除した後にMPPクエリで`INDEX OUT OF RANGE`エラーが発生するバグを修正しました  [#28250](https://github.com/pingcap/tidb/issues/28250)
    - MPPクエリに対する`invalid cop task execution summaries length`エラーログの誤検知問題を修正しました  [#1791](https://github.com/pingcap/tics/issues/1791)
    - `SET GLOBAL tidb_skip_isolation_level_check=1`が新しいセッション設定に影響を与えない問題を修正しました  [#27897](https://github.com/pingcap/tidb/issues/27897)
    - `tiup bench` が長時間実行される際に、`microsecond`および`hour`関数の結果が誤る問題を修正しました  [#26832](https://github.com/pingcap/tidb/issues/26832)

+ TiKV

    - GCワーカーがビジーな場合にTiKVがデータの範囲を削除できないバグを修正しました (`unsafe_destroy_range`が実行できない)  [#11903](https://github.com/tikv/tikv/issues/11903)
    - ピアを削除することで、高い遅延が発生するバグを修正しました  [#10210](https://github.com/tikv/tikv/issues/10210)
    - リージョンが空の場合に`any_value`関数が誤った結果を返すバグを修正しました  [#11735](https://github.com/tikv/tikv/issues/11735)
    - 初期化されていないレプリカを削除することで、古いレプリカが再作成される可能性のあるバグを修正しました  [#10533](https://github.com/tikv/tikv/issues/10533)
    - `Prepare Merge`がトリガーされた後で新しい選挙が行われ、孤立したピアに通知されない場合に、メタデータの破損問題を修正しました  [#11526](https://github.com/tikv/tikv/issues/11526)
    - コルーチンが高速に実行される場合に、発生する可能性のあるデッドロックの問題を修正しました  [#11549](https://github.com/tikv/tikv/issues/11549)
    - プロファイリングフレームグラフを作成する際に発生する可能性のあるデッドロックおよびメモリリークの問題を修正しました  [#11108](https://github.com/tikv/tikv/issues/11108)
    - 悲観的トランザクションにおいて、再試行する場合に稀に起こるデータの不整合問題を修正しました  [#11187](https://github.com/tikv/tikv/issues/11187)
    - 設定 `resource-metering.enabled` が機能しないバグを修正 [#11235](https://github.com/tikv/tikv/issues/11235)
    - `resolved_ts` でいくつかのコルーチンが漏れてしまう問題を修正 [#10965](https://github.com/tikv/tikv/issues/10965)
    - 低い書き込みフローで誤った "GC can not work" アラートが報告される問題を修正 [#9910](https://github.com/tikv/tikv/issues/9910)
    - tikv-ctl が正しいリージョン関連情報を返せないバグを修正 [#11393](https://github.com/tikv/tikv/issues/11393)
    - TiKV ノードのダウンによって resolved タイムスタンプが遅れる問題を修正 [#11351](https://github.com/tikv/tikv/issues/11351)
    - 極端な条件下でリージョンのマージ、ConfChange、Snapshotが同時に発生するとパニックが発生する問題を修正 [#11475](https://github.com/tikv/tikv/issues/11475)
    - TiKV が逆テーブルスキャンを実行する際にメモリロックを検出できない問題を修正 [#11440](https://github.com/tikv/tikv/issues/11440)
    - 十進法の除算結果がゼロの場合に負の符号が出力される問題を修正 [#29586](https://github.com/pingcap/tidb/issues/29586)
    - 統計スレッドのモニタリングデータがメモリリークを引き起こす問題を修正 [#11195](https://github.com/tikv/tikv/issues/11195)
    - 下流データベースが欠落している場合に発生する TiCDC のパニック問題を修正 [#11123](https://github.com/tikv/tikv/issues/11123)
    - Congest エラーにより TiCDC がスキャンリトライを頻繁に追加する問題を修正 [#11082](https://github.com/tikv/tikv/issues/11082)
    - Raft クライアント実装におけるバッチメッセージが大きすぎる問題を修正 [#9714](https://github.com/tikv/tikv/issues/9714)
    - Grafana ダッシュボードの中で一部の一般的でないストレージ関連のメトリクスを閉じる [#11681](https://github.com/tikv/tikv/issues/11681)

+ PD

    - リージョンスケッターによって生成されたスケジュールがピアの数を減少させるバグを修正 [#4565](https://github.com/tikv/pd/issues/4565)
    - リージョン統計が `flow-round-by-digit` によって影響を受けない問題を修正 [#4295](https://github.com/tikv/pd/issues/4295)
    - 停滞したリージョンシンカによって引き起こされる遅いリーダー選出を修正 [#3936](https://github.com/tikv/pd/issues/3936)
    - リーダースケジューラが不健康なピアを持つリージョンをスケジュールできるようにするサポートを追加 [#4093](https://github.com/tikv/pd/issues/4093)
    - コールドスポットのデータがホットスポット統計から削除できない問題を修正 [#4390](https://github.com/tikv/pd/issues/4390)
    - TiKV ノードが削除された後に発生するパニック問題を修正 [#4344](https://github.com/tikv/pd/issues/4344)
    - スケジューリングオペレータがターゲットストアがダウンしているために素早く失敗できない問題を修正 [#3353](https://github.com/tikv/pd/issues/3353)

+ TiFlash

    - `str_to_date()` 関数がマイクロ秒をパースする際に先頭のゼロを誤って処理する問題を修正
    - メモリ制限が有効化されている場合に TiFlash がクラッシュする問題を修正
    - 入力時間が 1970-01-01 00:00:01 UTC よりも前の場合、`unix_timestamp` の動作が TiDB または MySQL のそれと一貫性がない問題を修正
    - プライマリキーカラムを拡張すると、データの整合性が損なわれる可能性のある問題を修正
    - `DECIMAL` データ型のデータ比較時にオーバーフローバグと `Can't compare` エラーの報告問題を修正
    - `substringUTF8` 関数の予期せぬ `3rd arguments of function` エラーを修正
    - TiFlash が `nsl` ライブラリのないプラットフォームで起動に失敗する問題を修正
    - データを `DECIMAL` データ型にキャストする際のオーバーフローバグを修正
    - `castStringAsReal` の動作が TiFlash と TiDB/TiKV で一貫していない問題を修正
    - TiFlash が再起動後に `EstablishMPPConnection` エラーを返す可能性のある問題を修正
    - TiFlash レプリカ数を 0 に設定した後に古いデータを回収できない問題を修正
    - `castStringAsDecimal` の動作が TiFlash と TiDB/TiKV で一貫していない問題を修正
    - `where <string>` 条件でのクエリが誤った結果を返す問題を修正
    - MPP クエリが停止された際に TiFlash がパニックを起こす可能性がある問題を修正
    - `Unexpected type of column: Nullable(Nothing)` の予期せぬエラーを修正

+ Tools

    + TiCDC

        - `batch-replace-enable` が無効になっている場合に MySQL シンクが重複した `replace` SQL ステートメントを生成するバグを修正 [#4501](https://github.com/pingcap/tiflow/issues/4501)
        - `cached region` モニタリングメトリクスが負の値を示す問題を修正 [#4300](https://github.com/pingcap/tiflow/issues/4300)
        - `min.insync.replicas` が `replication-factor` よりも小さい場合にレプリケーションが実行されない問題を修正 [#3994](https://github.com/pingcap/tiflow/issues/3994)
        - レプリケーションタスクが削除された際に発生する潜在的なパニック問題を修正 [#3128](https://github.com/pingcap/tiflow/issues/3128)
        - 不正確なチェックポイントによって引き起こされる潜在的なデータ損失の問題を修正 [#3545](https://github.com/pingcap/tiflow/issues/3545)
        - デッドロックによってレプリケーションタスクがスタックする問題を修正 [#4055](https://github.com/pingcap/tiflow/issues/4055)
        - DDL ステートメント内の特別なコメントがレプリケーションタスクを停止させる問題を修正 [#3755](https://github.com/pingcap/tiflow/issues/3755)
        - EtcdWorker がオーナーとプロセッサーを停止する可能性があるバグを修正 [#3750](https://github.com/pingcap/tiflow/issues/3750)
        - クラスターのアップグレード後に自動的に `stopped` のチェンジフィードが再開する問題を修正 [#3473](https://github.com/pingcap/tiflow/issues/3473)
        - デフォルト値がレプリケーションされない問題を修正 [#3793](https://github.com/pingcap/tiflow/issues/3793)
        - TiCDC デフォルト値のパディング例外によって引き起こされるデータの不整合性を修正 [#3918](https://github.com/pingcap/tiflow/issues/3918) [#3929](https://github.com/pingcap/tiflow/issues/3929)
        - PD リーダーがシャットダウンし新しいノードに移行する際にオーナーがスタックするバグを修正 [#3615](https://github.com/pingcap/tiflow/issues/3615)
        - 手動で etcd のタスク状態をクリアする際に TiCDC パニックが発生する問題を修正 [#2980](https://github.com/pingcap/tiflow/issues/2980)
        - RHEL リリース内のタイムゾーンの問題によりサービスを起動できない問題を修正 [#3584](https://github.com/pingcap/tiflow/issues/3584)
        - MySQL シンクのデッドロックによって過度に頻繁な警告が発生する問題を修正 [#2706](https://github.com/pingcap/tiflow/issues/2706)
        - `enable-old-value` 設定項目が自動的に Canal および Maxwell プロトコルにおいて `true` に設定されないバグを修正 [#3676](https://github.com/pingcap/tiflow/issues/3676)
        - Avro シンクが JSON タイプのカラムの解析をサポートしていない問題を修正 [#3624](https://github.com/pingcap/tiflow/issues/3624)
        - changefeed チェックポイントラグにおける負の値エラーを修正 [#3010](https://github.com/pingcap/ticdc/issues/3010)
        - コンテナ環境における OOM 問題を修正 [#1798](https://github.com/pingcap/tiflow/issues/1798)
        - 複数の TiKV がクラッシュした場合や強制再起動中に TiCDC レプリケーションが中断する問題を修正 [#3288](https://github.com/pingcap/ticdc/issues/3288)
        - DDL を処理した後のメモリリーク問題を修正 [#3174](https://github.com/pingcap/ticdc/issues/3174)
        - `ErrGCTTLExceeded` エラーが発生した際に changefeed が十分に速く失敗しない問題を修正 [#3111](https://github.com/pingcap/ticdc/issues/3111)
        - upstream TiDB インスタンスが予期せず終了した際に TiCDC レプリケーションタスクが終了する問題を修正 [#3061](https://github.com/pingcap/tiflow/issues/3061)
        - TiKV が同じリージョンに重複リクエストを送信した際に TiCDC プロセスがパニックする問題を修正 [#2386](https://github.com/pingcap/tiflow/issues/2386)
        - Kafka がデフォルト値を `10M` に設定することで過度に大きなメッセージを送信する問題を修正 [#3081](https://github.com/pingcap/tiflow/issues/3081)
```
        - TiCDC同期タスクがKafkaメッセージの書き込み中にエラーが発生した場合に一時停止する可能性の問題を修正しました[#2978](https://github.com/pingcap/tiflow/issues/2978)

    + バックアップ＆リストア（BR）

        - リストア操作が完了した後にリージョンが均等に分散されない可能性の問題を修正しました[#30425](https://github.com/pingcap/tidb/issues/30425) [#31034](https://github.com/pingcap/tidb/issues/31034)

    + TiDB バイナリログ

        - CSVファイルのサイズが約256MBで`strict-format`が`true`の場合に、DBaaSがCSVのインポート時にInvalidRangeで失敗する問題を修正しました[#27763](https://github.com/pingcap/tidb/issues/27763)

    + TiDB Lightning

        - S3ストレージパスが存在しない場合にTiDB Lightningがエラーを報告しない問題を修正しました[#28031](https://github.com/pingcap/tidb/issues/28031) [#30709](https://github.com/pingcap/tidb/issues/30709)
```