---
title: TiDB 5.0.6 リリースノート
category: リリース

# TiDB 5.0.6 リリースノート

リリース日: 2021年12月31日

TiDB バージョン: 5.0.6

## 互換性の変更

+ ツール

    + TiCDC

        - `cdc server` コマンドのエラー出力を stdout から stderr へ変更しました [#3133](https://github.com/pingcap/tiflow/issues/3133)
        - Kafka sink の `max-message-bytes` のデフォルト値を `10M` に設定しました [#3081](https://github.com/pingcap/tiflow/issues/3081)
        - TiCDC が Kafka パーティション間でメッセージをより均等に分散できるよう、Kafka Sink の `partition-num` のデフォルト値を 3 に変更しました [#3337](https://github.com/pingcap/ticdc/issues/3337)

## 改善点

+ TiDB

    - コプロセッサがロックに遭遇した際にデバッグログに影響を受ける SQL ステートメントを表示し、問題の診断に役立ちます [#27718](https://github.com/pingcap/tidb/issues/27718)

+ TiKV

    - 検証プロセスを `Apply` スレッドプールから `Import` スレッドプールに移動することで SST ファイルの挿入速度を向上させました [#11239](https://github.com/tikv/tikv/issues/11239)
    - Raft ログのガベージコレクションモジュールの性能問題を特定するため、より多くのメトリクスを追加しました [#11374](https://github.com/tikv/tikv/issues/11374)
    - Grafana ダッシュボードで一部のまれなストレージ関連メトリクスをまとめました [#11681](https://github.com/tikv/tikv/issues/11681)

+ PD

    - スケジューラの終了プロセスを高速化しました [#4146](https://github.com/tikv/pd/issues/4146)
    - `scatter-range-scheduler` スケジューラのスケジューリング結果を、空のリージョンをスケジューラにスケジュールできるようにし、スケジューラの設定を修正することで、より均等なスケジューリング結果にしました [#4497](https://github.com/tikv/pd/issues/4497)
    - リーダースケジューラが健康でないピアを持つリージョンをスケジュールできるようサポートしました [#4093](https://github.com/tikv/pd/issues/4093)

+ ツール

    + TiCDC

        - TiKV のリロード時のレート制御を最適化し、changefeed の初期化中の gRPC 混雑を減らしました [#3110](https://github.com/pingcap/ticdc/issues/3110)
        - EtcdWorker に tick 頻度制限を追加し、頻繁な etcd 書き込みが PD サービスに影響するのを防ぎました [#3112](https://github.com/pingcap/ticdc/issues/3112)
        - Kafka sink の `config.Metadata.Timeout` のデフォルト構成を追加しました [#3352](https://github.com/pingcap/tiflow/issues/3352)
        - Kafka メッセージが送信できない確率を低減するために、`max-message-bytes` のデフォルト値を `10M` に設定しました [#3081](https://github.com/pingcap/tiflow/issues/3081)
        - `no owner alert`、`mounter row`、`table sink total row`、`buffer sink total row` を含むより多くの Prometheus および Grafana 監視メトリクスとアラートを追加しました [#4054](https://github.com/pingcap/tiflow/issues/4054) [#1606](https://github.com/pingcap/tiflow/issues/1606)

    + Backup & Restore (BR)

        - PD リクエストエラーや TiKV I/O タイムアウトエラーに遭遇した場合に BR タスクをリトライするようにしました [#27787](https://github.com/pingcap/tidb/issues/27787)
        - リストアの堅牢性を向上させました [#27421](https://github.com/pingcap/tidb/issues/27421)

    + TiDB Lightning

        - 式インデックスを持つテーブルや仮想生成列に依存するインデックスにデータをインポートすることをサポートしました [#1404](https://github.com/pingcap/br/issues/1404)

## バグ修正

+ TiDB

    - 楽観的トランザクションの競合がトランザクションを相互にブロックする可能性がある問題を修正しました [#11148](https://github.com/tikv/tikv/issues/11148)
    - MPP クエリでの誤検出エラーログ `invalid cop task execution summaries length` の問題を修正しました [#1791](https://github.com/pingcap/tics/issues/1791)
    - DML および DDL ステートメントが同時に実行された際に発生する可能性のあるパニックを修正しました [#30940](https://github.com/pingcap/tidb/issues/30940)
    - `grant` および `revoke` 操作を実行してグローバルレベルの権限を付与および取り消す際に発生する `privilege check fail` エラーを修正しました [#29675](https://github.com/pingcap/tidb/issues/29675)
    - 一部のケースで `ALTER TABLE.. ADD INDEX` ステートメントを実行する際に TiDB がパニックする問題を修正しました [#27687](https://github.com/pingcap/tidb/issues/27687)
    - `enforce-mpp` 構成が v5.0.4 で適用されない問題を修正しました [#29252](https://github.com/pingcap/tidb/issues/29252)
    - `ENUM` データ型の `CASE WHEN` 関数使用時のパニックを修正しました [#29357](https://github.com/pingcap/tidb/issues/29357)
    - ベクトル化された式で `microsecond` 関数の誤った結果を修正しました [#29244](https://github.com/pingcap/tidb/issues/29244)
    - `auto analyze` 結果からの不完全なログ情報の問題を修正しました [#29188](https://github.com/pingcap/tidb/issues/29188)
    - ベクトル化式で `hour` 関数の誤った結果を修正しました [#28643](https://github.com/pingcap/tidb/issues/28643)
    - サポートされていない `cast` が TiFlash にプッシュダウンされた際に予期しない `tidb_cast to Int32 is not supported` エラーが発生する問題を修正しました [#23907](https://github.com/pingcap/tidb/issues/23907)
    - 一部のケースで MPP ノードの可用性検出が機能しない問題を修正しました [#3118](https://github.com/pingcap/tics/issues/3118)
    - `MPP task ID` の割り当て時に `DATA RACE` 問題を修正しました [#27952](https://github.com/pingcap/tidb/issues/27952)
    - 空の `dual table` を削除した後に MPP クエリで `INDEX OUT OF RANGE` エラーが発生する問題を修正しました [#28250](https://github.com/pingcap/tidb/issues/28250)
    - 無効な日付値が同時に挿入された際に TiDB がパニックする問題を修正しました [#25393](https://github.com/pingcap/tidb/issues/25393)
    - MPP モードのクエリで、JSON 型の列が `CHAR` 型の列と結合すると予期しない `column in Schema column` エラーが発生する問題を修正しました [#29401](https://github.com/pingcap/tidb/issues/29401)
    - 怠惰な存在チェックと未処理のキー最適化の不正な使用によるデータの不整合問題を修正しました [#30410](https://github.com/pingcap/tidb/issues/30410)
    - トランザクションの有無によってウィンドウ関数が異なる結果を返す問題を修正しました [#29947](https://github.com/pingcap/tidb/issues/29947)
    - `cast(integer as char) union string` を含む SQL ステートメントで予期しない `can not found column in Schema column` エラーが報告される問題を修正しました [#29513](https://github.com/pingcap/tidb/issues/29513)
    - `Decimal` を `String` にキャストする際に長さ情報が誤っている問題を修正しました [#29417](https://github.com/pingcap/tidb/issues/29417)
    - `GREATEST` 関数が `tidb_enable_vectorized_expression`（`on` または `off` に設定）の異なる値によって一貫しない結果を返す問題を修正しました [#29434](https://github.com/pingcap/tidb/issues/29434)
- いくつかのケースでプランナーが`join`の無効なプランをキャッシュする問題を修正します [#28087](https://github.com/pingcap/tidb/issues/28087)
    - SQLステートメントがいくつかのケースで`join`の結果で集約結果を評価した際に`index out of range [1] with length 1`エラーが報告される問題を修正します [#1978](https://github.com/pingcap/tics/issues/1978)

+ TiKV

    - ダウンしたTiKVノードが解決タイムスタンプを遅らせる原因になる問題を修正します [#11351](https://github.com/tikv/tikv/issues/11351)
    - Raftクライアントの実装において、バッチメッセージがあまりにも大きい問題を修正します [#9714](https://github.com/tikv/tikv/issues/9714)
    - 極限の状況下で、Regionの統合、ConfChange、Snapshotが同時に発生した際に発生するパニックの問題を修正します [#11475](https://github.com/tikv/tikv/issues/11475)
    - TiKVが逆テーブルスキャンを実行した際にメモリロックを検知できない問題を修正します [#11440](https://github.com/tikv/tikv/issues/11440)
    - 小数点以下の結果がゼロの場合に小数点以下の符号が間違っていた問題を修正します [#29586](https://github.com/pingcap/tidb/issues/29586)
    - GCタスクの蓄積がTiKVにOOM (メモリ不足)を引き起こす可能性がある問題を修正します [#11410](https://github.com/tikv/tikv/issues/11410)
    - マシン単位のgRPCリクエストの平均レイテンシーがTiKVメトリクスにおいて正確でない問題を修正します [#11299](https://github.com/tikv/tikv/issues/11299)
    - 統計スレッドの監視データによるメモリリークの問題を修正します [#11195](https://github.com/tikv/tikv/issues/11195)
    - 下流データベースが欠落している場合にTiCDCがパニックする問題を修正します [#11123](https://github.com/tikv/tikv/issues/11123)
    - キャストエラーの発生時にスキャンリトライが頻繁に追加される問題を修正します [#11082](https://github.com/tikv/tikv/issues/11082)
    - チャネルがいっぱいの場合にRaft接続が切断される問題を修正します [#11047](https://github.com/tikv/tikv/issues/11047)
    - TiDB Lightningがデータをインポートする際にファイルが存在しない場合にTiKVがパニックする問題を修正します [#10438](https://github.com/tikv/tikv/issues/10438)
    - `Max`/`Min`関数内の`Int64`型が符号付き整数であるかどうかをTiDBが正しく識別できず、`Max`/`Min`の計算結果が間違ってしまう問題を修正します [#10158](https://github.com/tikv/tikv/issues/10158)
    - TiKVのレプリカのノードがダウンする問題を修正します。これは、ノードがスナップショットを取得した後にメタデータを正確に修正できないためです [#10225](https://github.com/tikv/tikv/issues/10225)
    - バックアップスレッドプールのリーク問題を修正します [#10287](https://github.com/tikv/tikv/issues/10287)
    - 不正な文字列を浮動小数点数へのキャストする問題を修正します [#23322](https://github.com/pingcap/tidb/issues/23322)

+ PD

    - TiKVノードが削除された後に発生するパニックの問題を修正します [#4344](https://github.com/tikv/pd/issues/4344)
    - ストアがダウンしているためにオペレーターがブロックされる問題を修正します [#3353](https://github.com/tikv/pd/issues/3353)
    - 滞在しているRegion syncerによる遅いリーダー選出を修正します [#3936](https://github.com/tikv/pd/issues/3936)
    - ダウンしているノードの修復時にピアを削除する速度が制限されている問題を修正します [#4090](https://github.com/tikv/pd/issues/4090)
    - Regionのハートビートが60秒未満の場合にホットスポットキャッシュをクリアできない問題を修正します [#4390](https://github.com/tikv/pd/issues/4390)

+ TiFlash

    - プライマリキーカラムをより大きなintデータ型に変更した後の潜在的なデータ不整合問題を修正します
    - ARMなどの一部のプラットフォームでTiFlashが起動しない問題を修正します。これは`libnsl.so`ライブラリが存在しないためです
    - `ストアサイズ`メトリクスがディスク上の実際のデータサイズと一致しない問題を修正します
    - `Cannot open file`エラーによってTiFlashがクラッシュする問題を修正します
    - MPPクエリをキャンセルした際にTiFlashが時々クラッシュする問題を修正します
    - 過剰な`OR`条件によってクエリが失敗する問題を修正します
    - `where <string>`の結果が正しくない問題を修正します
    - TiFlashとTiDB/TiKVの間の`CastStringAsDecimal`の振る舞いの不一致を修正します
    - データを比較する際に`DECIMAL`データ型でオーバーフローが発生する問題を修正します

+ ツール

    + TiCDC

        - `force-replicate`が有効な場合にパーティションされた一部のテーブルが無視される問題を修正します [#2834](https://github.com/pingcap/tiflow/issues/2834)
        - 予期しないパラメータを受け取った際に`cdc cli`がユーザーパラメータを黙って切り捨て、ユーザー入力パラメータが失われる問題を修正します [#2303](https://github.com/pingcap/tiflow/issues/2303)
        - Kafkaメッセージの書き込み中にエラーが発生した際にTiCDC同期タスクが一時停止する可能性がある問題を修正します [#2978](https://github.com/pingcap/tiflow/issues/2978)
        - 一部の列のエンコード時にパニックが発生する可能性がある問題を修正します [#2758](https://github.com/pingcap/tiflow/issues/2758)
        - Kafkaが`max-message-bytes`のデフォルト値を`10M`に設定することで極端に大きなメッセージを送信する問題を修正します [#3081](https://github.com/pingcap/tiflow/issues/3081)
        - アップストリームのTiDBインスタンスが予期せず終了した際にTiCDCレプリケーションタスクが終了する問題を修正します [#3061](https://github.com/pingcap/tiflow/issues/3061)
        - TiKVが同じRegionへ重複したリクエストを送信した際にTiCDCプロセスがパニックする可能性がある問題を修正します [#2386](https://github.com/pingcap/tiflow/issues/2386)
        - 複数のTiKVがクラッシュした際や強制再起動中にTiCDCレプリケーションが中断する問題を修正します [#3288](https://github.com/pingcap/ticdc/issues/3288)
        - changefeedのチェックポイントの遅延に負の値が表示されるエラーを修正します [#3010](https://github.com/pingcap/ticdc/issues/3010)
        - MySQLシンクがデッドロックすることによって頻繁に警告が発生する問題を修正します [#2706](https://github.com/pingcap/tiflow/issues/2706)
        - AvroシンクがJSONタイプの列の解析をサポートしていない問題を修正します [#3624](https://github.com/pingcap/tiflow/issues/3624)
        - TiKV所有者が再起動した際に、TiCDCがTiKVから正しくないスキーマスナップショットを読み込む問題を修正します [#2603](https://github.com/pingcap/tiflow/issues/2603)
        - DDL処理後にメモリリークが発生する問題を修正します [#3174](https://github.com/pingcap/ticdc/issues/3174)
        - `enable-old-value`構成項目がCanalとMaxwellプロトコルで自動的に`true`に設定されないバグを修正します [#3676](https://github.com/pingcap/tiflow/issues/3676)
        - `cdc server`コマンドがRed Hat Enterprise Linuxの一部のリリース（6.8や6.9など）でタイムゾーンエラーが発生する問題を修正します [#3584](https://github.com/pingcap/tiflow/issues/3584)
        - Kafkaシンクにおける「txn_batch_size」監視メトリクスの不正確さの問題を修正します [#3431](https://github.com/pingcap/tiflow/issues/3431)
        - changefeedが存在しない場合にも`changefeed`のredisリゾルブポリシーチェックに関する警告が継続的に発生する問題を修正します [#11017](https://github.com/tikv/tikv/issues/11017)
        - 手動でetcdでタスクステータスをクリアした際にTiCDCがパニックする問題を修正します [#2980](https://github.com/pingcap/tiflow/issues/2980)
        - ErrGCTTLExceededエラーが発生した際にchangefeedが十分に速く失敗しない問題を修正します [#3111](https://github.com/pingcap/ticdc/issues/3111)
        - stockデータのスキャンが長くかかるとTiKVがGCを実行し、スキャンが失敗する問題を修正します [#2470](https://github.com/pingcap/tiflow/issues/2470)
        - コンテナ環境でのOOM問題を修正します [#1798](https://github.com/pingcap/ticdc/issues/1798)

    + バックアップ & リストア (BR)
- バックアップとリストアの平均速度が正確に計算されないバグを修正する [#1405](https://github.com/pingcap/br/issues/1405)

    + ダンプリング

        - 複合主キーまたはユニークキーを持つテーブルをダンプする際にダンプリングが非常に遅くなるバグを修正する [#29386](https://github.com/pingcap/tidb/issues/29386)