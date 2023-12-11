---
title: TiDB 5.2.2 リリースノート
---

# TiDB 5.2.2 リリースノート

リリース日: 2021年10月29日

TiDB バージョン: 5.2.2

## 改善点

+ TiDB

    - コプロセッサがロックに遭遇したときに、デバッグログに影響を受けるSQLステートメントを表示するようにしました。これにより問題の診断が容易になります [#27718](https://github.com/pingcap/tidb/issues/27718)
    - SQL論理層でバックアップとリストアデータのサイズを表示する機能をサポートしました [#27247](https://github.com/pingcap/tidb/issues/27247)

+ TiKV

    - L0フローコントロールのアルゴリズムを簡素化しました [#10879](https://github.com/tikv/tikv/issues/10879)
    - Raftクライアントモジュールのエラーログ報告を改善しました [#10983](https://github.com/tikv/tikv/pull/10983)
    - スレッドのログ出力を最適化し、パフォーマンスのボトルネックになるのを防ぎます [#10841](https://github.com/tikv/tikv/issues/10841)
    - 書き込みクエリの統計種類を追加しました [#10507](https://github.com/tikv/tikv/issues/10507)

+ PD

    - ホットスポットスケジューラのQPS次元に書き込みクエリのさらなる種類を追加しました [#3869](https://github.com/tikv/pd/issues/3869)
    - バランスリージョンスケジューラのリトライ制限を動的に調整する機能をサポートし、スケジューラのパフォーマンスを向上させました [#3744](https://github.com/tikv/pd/issues/3744)
    - TiDBダッシュボードをv2021.10.08.1に更新しました [#4070](https://github.com/tikv/pd/pull/4070)
    - 不健全なピアを持つリージョンをスケジュール可能にするように、リーダースケジューラをサポートしました [#4093](https://github.com/tikv/pd/issues/4093)
    - スケジューラの終了処理を高速化しました [#4146](https://github.com/tikv/pd/issues/4146)

+ ツール

    + TiCDC

        - `MaxMessageBytes`のKafkaシンク構成項目のデフォルト値を64MBから1MBに減らし、大きなメッセージがKafkaブローカーに拒否される問題を修正しました [#3104](https://github.com/pingcap/tiflow/pull/3104)
        - レプリケーションパイプラインのメモリ使用量を削減しました [#2553](https://github.com/pingcap/tiflow/issues/2553) [#3037](https://github.com/pingcap/tiflow/pull/3037) [#2726](https://github.com/pingcap/tiflow/pull/2726)
        - 同期リンク、メモリGC、およびストックデータスキャンプロセスの監視アイテムとアラートルールを最適化し、可観測性を向上させました [#2735](https://github.com/pingcap/tiflow/pull/2735) [#1606](https://github.com/pingcap/tiflow/issues/1606) [#3000](https://github.com/pingcap/tiflow/pull/3000) [#2985](https://github.com/pingcap/tiflow/issues/2985) [#2156](https://github.com/pingcap/tiflow/issues/2156)
        - 同期タスクのステータスが正常な場合、歴史的なエラーメッセージを表示しないようにし、ユーザーを誤解させないようにしました [#2242](https://github.com/pingcap/tiflow/issues/2242)

## バグ修正

+ TiDB

    - プランキャッシュが符号なしフラグの変更を検出できない問題を修正しました [#28254](https://github.com/pingcap/tidb/issues/28254)
    - パーティション関数が範囲外の場合に誤ったパーティションプルーニングを修正しました [#28233](https://github.com/pingcap/tidb/issues/28233)
    - 一部の場合で`join`のプランを無効にキャッシュする問題を修正しました [#28087](https://github.com/pingcap/tidb/issues/28087)
    - ハッシュ列の種類がenumの場合の誤ったインデックスハッシュ結合を修正しました [#27893](https://github.com/pingcap/tidb/issues/27893)
    - アイドルコネクションの再利用が一部の稀なケースでリクエスト送信をブロックする可能性があるバッチクライアントのバグを修正しました [#27688](https://github.com/pingcap/tidb/pull/27688)
    - ターゲットクラスターでチェックサムを実行できない場合にTiDB Lightningのパニック発生問題を修正しました [#27686](https://github.com/pingcap/tidb/pull/27686)
    - 一部の場合で`date_add`および`date_sub`関数の誤った結果を修正しました [#27232](https://github.com/pingcap/tidb/issues/27232)
    - ベクトル化式で`hour`関数の誤った結果を修正しました [#28643](https://github.com/pingcap/tidb/issues/28643)
    - MySQL 5.1またはそれ以前のクライアントバージョンに接続する際の認証の問題を修正しました  [#27855](https://github.com/pingcap/tidb/issues/27855)
    - 新しいインデックスが追加されると`auto analyze`が指定した時間外にトリガーされる可能性がある問題を修正しました [#28698](https://github.com/pingcap/tidb/issues/28698)
    - 任意のセッション変数を設定すると`tidb_snapshot`が無効になるバグを修正しました [#28683](https://github.com/pingcap/tidb/pull/28683)
    - 多くの欠落ピアリージョンを持つクラスターに対してBRが機能しないバグを修正しました [#27534](https://github.com/pingcap/tidb/issues/27534)
    - サポートされていない`cast`がTiFlashに押し下げられると`tidb_cast to Int32 is not supported`のような予期しないエラーが発生する問題を修正しました [#23907](https://github.com/pingcap/tidb/issues/23907)
    - `%s value is out of range in '%s'`エラーメッセージで`DECIMAL overflow`が欠落する問題を修正しました  [#27964](https://github.com/pingcap/tidb/issues/27964)
    - MPPノードの可用性検出が一部の端末で機能しない問題を修正しました [#3118](https://github.com/pingcap/tics/issues/3118)
    - `MPP task ID`を割り当てる際の`DATA RACE`問題を修正しました [#27952](https://github.com/pingcap/tidb/issues/27952)
    - 空の`dual table`を削除した後のMPPクエリの`INDEX OUT OF RANGE`エラーを修正しました [#28250](https://github.com/pingcap/tidb/issues/28250)
    - MPPクエリに対する`invalid cop task execution summaries length`の偽のエラーログの問題を修正しました [#1791](https://github.com/pingcap/tics/issues/1791)
    - MPPクエリに対する`cannot found column in Schema column`のエラーログの問題を修正しました [#28149](https://github.com/pingcap/tidb/pull/28149)
    - TiFlashのシャットダウン時にTiDBがパニックする可能性がある問題を修正しました [#28096](https://github.com/pingcap/tidb/issues/28096)
    - 安全でない3DES（Triple Data Encryption Algorithm）ベースのTLS暗号スイートをサポートしないようにしました [#27859](https://github.com/pingcap/tidb/pull/27859)
    - プレチェック中にオフラインTiKVノードに接続することでインポートが失敗する問題を修正しました [#27826](https://github.com/pingcap/tidb/pull/27826)
    - 多くのファイルをテーブルにインポートする際にプレチェックに余分な時間がかかる問題を修正しました [#27605](https://github.com/pingcap/tidb/issues/27605)
    - 式の再書き込みによって`between`が間違った照合を推論する問題を修正しました [#27146](https://github.com/pingcap/tidb/issues/27146)
    - `group_concat`関数が照合を考慮しない問題を修正しました [#27429](https://github.com/pingcap/tidb/issues/27429)
    - `extract`関数の引数が負の期間の場合に発生する結果が間違っていた問題を修正しました [#27236](https://github.com/pingcap/tidb/issues/27236)
    - `NO_UNSIGNED_SUBTRACTION`が設定されている場合にパーティションの作成に失敗する問題を修正しました [#26765](https://github.com/pingcap/tidb/issues/26765)
    - カラムプルーニングおよび集計プッシュダウンで副作用を持つ式を回避しました [#27106](https://github.com/pingcap/tidb/issues/27106)
    - 不要なgRPCログを削除しました [#24190](https://github.com/pingcap/tidb/issues/24190)
    - 有効な10進数の長さを制限して、桁数関連の問題を修正しました [#3091](https://github.com/pingcap/tics/issues/3091)
    - `plus`式でオーバーフローをチェックする誤った方法を修正しました [#26977](https://github.com/pingcap/tidb/issues/26977)
    - `new collation`データを持つテーブルから統計情報をダンプする際の`data too long`エラーの問題を修正しました [#27024](https://github.com/pingcap/tidb/issues/27024)
    - リトライされたトランザクションのステートメントが`TIDB_TRX`に含まれない問題を修正しました [#28670](https://github.com/pingcap/tidb/pull/28670)

+ TiKV
    - CDCがCongestエラーによってスキャン再試行を頻繁に追加する問題を修正 [#11082](https://github.com/tikv/tikv/issues/11082)
    - チャネルがいっぱいの場合、raft接続が切断される問題を修正 [#11047](https://github.com/tikv/tikv/issues/11047)
    - Raftクライアント実装においてバッチメッセージが大きすぎる問題を修正 [#9714](https://github.com/tikv/tikv/issues/9714)
    - `resolved_ts`において一部のコルーチンが漏れる問題を修正 [#10965](https://github.com/tikv/tikv/issues/10965)
    - 応答のサイズが4 GiBを超える場合に、coprocessorでパニックが発生する問題を修正 [#9012](https://github.com/tikv/tikv/issues/9012)
    - スナップショットガベージコレクション（GC）がスナップショットファイルのGCを逃す問題を修正 [#10813](https://github.com/tikv/tikv/issues/10813)
    - Coprocessorリクエストの処理中にタイムアウトによるパニックが発生する問題を修正 [#10852](https://github.com/tikv/tikv/issues/10852)

+ PD

    - 設定されたピア数を超えるため、PDが誤ってデータとペンディング状態のピアを削除する問題を修正 [#4045](https://github.com/tikv/pd/issues/4045)
    - PDがダウンしているピアを適切なタイミングで修復しない問題を修正 [#4077](https://github.com/tikv/pd/issues/4077)
    - スキャッタ範囲スケジューラが空のリージョンをスケジュールできない問題を修正 [#4118](https://github.com/tikv/pd/pull/4118)
    - キーマネージャが高いCPUを消費する問題を修正 [#4071](https://github.com/tikv/pd/issues/4071)
    - Hotリージョンスケジューラの設定を行う際に発生するデータレース問題を修正 [#4159](https://github.com/tikv/pd/issues/4159)
    - スタックされたリージョンシンカによって遅延したリーダー選出を修正 [#3936](https://github.com/tikv/pd/issues/3936)

+ TiFlash

    - ライブラリ `nsl` の不在によってTiFlashの起動に失敗する問題を修正

+ Tools

    + TiCDC
        - アップストリームのTiDBインスタンスが予期せず終了した際に、TiCDCレプリケーションタスクが終了する場合がある問題を修正 [#3061](https://github.com/pingcap/tiflow/issues/3061)
        - TiKVが同一リージョンに対して重複リクエストを送信した時に、TiCDCプロセスがパニックする可能性がある問題を修正 [#2386](https://github.com/pingcap/tiflow/issues/2386)
- 下流のTiDB/MySQLの可用性を検証する際に不要なCPU消費を修正 [#3073](https://github.com/pingcap/tiflow/issues/3073)
        - TiCDCによって生成されたKafkaメッセージのボリュームが`max-message-size`に制約されない問題を修正 [#2962](https://github.com/pingcap/tiflow/issues/2962)
        - Kafkaメッセージの書き込み中にエラーが発生した際にTiCDC同期タスクが一時停止する可能性がある問題を修正 [#2978](https://github.com/pingcap/tiflow/issues/2978)
        - 有効なインデックスを持たないパーティション化されたテーブルが`force-replicate`で無視される可能性がある問題を修正 [#2834](https://github.com/pingcap/tiflow/issues/2834)
        - ストックデータのスキャンが長時間かかり、TiKVがGCを実行する際に失敗する可能性がある問題を修正 [#2470](https://github.com/pingcap/tiflow/issues/2470)
        - 一部の列をOpen Protocol形式にエンコードする際に、パニックが発生する可能性がある問題を修正 [#2758](https://github.com/pingcap/tiflow/issues/2758)
        - 一部の列をAvro形式にエンコードする際に、パニックが発生する可能性がある問題を修正 [#2648](https://github.com/pingcap/tiflow/issues/2648)

    + TiDB Binlog

        - 多くのテーブルがフィルタリングされる場合に、特定の負荷下でチェックポイントが更新できない問題を修正 [#1075](https://github.com/pingcap/tidb-binlog/pull/1075)