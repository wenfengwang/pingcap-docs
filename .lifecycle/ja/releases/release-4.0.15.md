---
title: TiDB 4.0.15のリリースノート
---

# TiDB 4.0.15のリリースノート

リリース日: 2021年9月27日

TiDB バージョン: 4.0.15

## 互換性の変更

+ TiDB

    - 新しいセッションで `SHOW VARIABLES` を実行すると遅い問題を修正しました。この修正は、[#21045](https://github.com/pingcap/tidb/pull/21045) で行われた一部の変更を元に戻し、互換性の問題を引き起こす可能性があります。[#24326](https://github.com/pingcap/tidb/issues/24326)
    + 以下のバグ修正は実行結果を変更します。これにより、アップグレードの非互換性が引き起こされる可能性があります：
        - `greatest(datetime) union null` が空の文字列を返す問題を修正しました。[#26532](https://github.com/pingcap/tidb/issues/26532)
        - `having` 句が正常に機能しない可能性がある問題を修正しました。[#26496](https://github.com/pingcap/tidb/issues/26496)
        - `between` 式の周囲の照合順序が異なる場合に発生する結果の誤りを修正しました。[#27146](https://github.com/pingcap/tidb/issues/27146)
        - `extract` 関数の引数が負の期間の場合に発生する結果の誤りを修正しました。[#27236](https://github.com/pingcap/tidb/issues/27236)
        - `group_concat` 関数の列が非バイナリ照合順序を持つ場合に発生する結果の誤りを修正しました。[#27429](https://github.com/pingcap/tidb/issues/27429)
        - `Apply` 演算子を `Join` に変換する際に列情報が欠落する問題を修正しました。[#27233](https://github.com/pingcap/tidb/issues/27233)
        - 無効な文字列を `DATE` に変換する際の `count distinct` 結果が、新しい照合順序が有効になっている場合に誤っているバグを修正しました。[#27091](https://github.com/pingcap/tidb/issues/27091)

## 機能の強化

+ TiKV

    - TiCDCの構成を動的に変更する機能をサポートしました。[#10645](https://github.com/tikv/tikv/issues/10645)

## 改善点

+ TiDB

    - ヒストグラムの行数に基づいて自動的に自動分析をトリガーするようにしました。[#24237](https://github.com/pingcap/tidb/issues/24237)

+ TiKV

    - 読み取り準備と書き込み準備を別々に処理して、読み取り遅延を削減しました。[#10475](https://github.com/tikv/tikv/issues/10475)
    - TiKVコプロセッサの遅いログは、リクエスト処理に費やされた時間のみを考慮します。[#10841](https://github.com/tikv/tikv/issues/10841)
    - スローガーログがスレッドをブロックする代わりに、sloggerスレッドが過負荷でキューが一杯になったときにログを削除しました。[#10841](https://github.com/tikv/tikv/issues/10841)
    - ネットワーク帯域幅を節約するために、解決済みTSメッセージのサイズを削減しました。[#2448](https://github.com/pingcap/tiflow/issues/2448)

+ PD

    - PD間でリージョン情報を同期する際のパフォーマンスを改善しました。[#3932](https://github.com/tikv/pd/pull/3932)

+ Tools

    + バックアップ ＆ リストア（BR）

        - リストア速度を向上させるために、同時にリージョンを分割して散らします。[#1363](https://github.com/pingcap/br/pull/1363)
        - PDリクエストエラーまたはTiKV I/Oタイムアウトエラーに遭遇した場合に、BRタスクをリトライします。[#27787](https://github.com/pingcap/tidb/issues/27787)
        - リストア時に多くの小さなテーブルをリストアする際に空のリージョンを削減し、リストア後のクラスタ操作に影響を与えないようにしました。[#1374](https://github.com/pingcap/br/issues/1374)
        - テーブルの作成時に `rebase auto id` 操作を実行し、別個の `rebase auto id` DDL操作を省略してリストアを高速化しました。[#1424](https://github.com/pingcap/br/pull/1424)

    + Dumpling

        - `SHOW TABLE STATUS` のフィルタリング効率を向上させるために、スキップされたデータベースを取得する前にフィルタリングしました。[#337](https://github.com/pingcap/dumpling/pull/337)
        - `SHOW TABLE STATUS` が一部のMySQLバージョンで正しく機能しないため、エクスポートするテーブルの情報を取得するために `SHOW FULL TABLES` を使用しました。[#322](https://github.com/pingcap/dumpling/issues/322)
        - `START TRANSACTION ... WITH CONSISTENT SNAPSHOT` または `SHOW CREATE TABLE` 構文をサポートしていない、MySQL互換のデータベースのバックアップをサポートしました。[#309](https://github.com/pingcap/dumpling/issues/309)
        - ダンプが失敗したときの混乱を避けるために、Dumplingの警告ログを改善しました。[#340](https://github.com/pingcap/dumpling/pull/340)

    + TiDB Lightning

        - 式インデックスまたは仮想生成列に依存するインデックスを持つテーブルにデータをインポートする機能をサポートしました。[#1404](https://github.com/pingcap/br/issues/1404)

    + TiCDC

        - 使用性を改善するために、常にTiKV内部から古い値を取得するようにしました。[#2397](https://github.com/pingcap/tiflow/pull/2397)
        - テーブルのリージョンがTiKVノードからすべて転送されたときに、より少ないゴルーチンを使用するようにworkerpoolを最適化しました。[#2284](https://github.com/pingcap/tiflow/issues/2284)
        - 並列度が高い場合にゴルーチンを減らすようにworkerpoolを最適化しました。[#2211](https://github.com/pingcap/tiflow/issues/2211)
        - 他のチェンジフィードに影響を与えないように、DDLステートメントを非同期で実行するようにしました。[#2295](https://github.com/pingcap/tiflow/issues/2295)
        - グローバルgRPC接続プールを追加し、KVクライアント間でgRPC接続を共有するようにしました。[#2531](https://github.com/pingcap/tiflow/pull/2531)
        - 回復不可能なDMLエラーに対して速攻で失敗するようにしました。[#1724](https://github.com/pingcap/tiflow/issues/1724)
        - Unified Sorterがデータを並べ替えるためにメモリを使用している場合のメモリ管理を最適化しました。[#2553](https://github.com/pingcap/tiflow/issues/2553)
        - DDL実行のためのPrometheusメトリクスを追加しました。[#2595](https://github.com/pingcap/tiflow/issues/2595) [#2669](https://github.com/pingcap/tiflow/issues/2669)
        - メジャーまたはマイナーバージョンをまたがるTiCDCクラスタの操作を禁止しました。[#2601](https://github.com/pingcap/tiflow/pull/2601)
        - `file sorter` を削除しました。[#2325](https://github.com/pingcap/tiflow/pull/2325)
        - チェンジフィードが削除されたときにチェンジフィードメトリクスをクリーンアップし、プロセッサが終了したときにプロセッサメトリクスをクリーンアップしました。[#2156](https://github.com/pingcap/tiflow/issues/2156)
        - リージョンが初期化された後のロック解決アルゴリズムを最適化しました。[#2188](https://github.com/pingcap/tiflow/issues/2188)

## バグ修正

+ TiDB

    - レンジの構築中に、バイナリリテラルの照合が誤って設定されるバグを修正しました。[#23672](https://github.com/pingcap/tidb/issues/23672)

    - `GROUP BY` と `UNION` を含むクエリで発生する "index out of range" エラーを修正しました。[#26553](https://github.com/pingcap/tidb/pull/26553)
    - TiDBがリクエストを送信できない可能性がある問題を解消しました。[#23676](https://github.com/pingcap/tidb/issues/23676) [#24648](https://github.com/pingcap/tidb/issues/24648)
    - 未公開の `/debug/sub-optimal-plan` HTTP APIを削除しました。[#27264](https://github.com/pingcap/tidb/pull/27264)
    - `case when` 式の誤った文字セットと照合を修正しました。[#26662](https://github.com/pingcap/tidb/issues/26662)

+ TiKV

    - TDEがデータのリストア中に有効になっているときに、BRが "file already exists" エラーを報告する問題を修正しました。[#1179](https://github.com/pingcap/br/issues/1179)
    - 破損したスナップショットファイルによって引き起こされるディスクフル問題を修正しました。[#10813](https://github.com/tikv/tikv/issues/10813)
    - TiKVが頻繁にPDクライアントとの再接続を行う問題を修正しました。[#9690](https://github.com/tikv/tikv/issues/9690)
    - 暗号化ファイル辞書から古いファイル情報を確認するようにしました。[#9115](https://github.com/tikv/tikv/issues/9115)

+ PD

    - PDがダウンしているリージョンを適切に修正しない問題を修正しました。[#4077](https://github.com/tikv/pd/issues/4077)
    - TiKVのスケーリング中にPDがパニックになる可能性があるバグを修正しました。[#3868](https://github.com/tikv/pd/issues/3868)

+ TiFlash
- 複数のディスクにTiFlashが展開された場合に発生するデータの整合性の潜在的な問題を修正します
    - `CONSTANT`、`<`、`<=`、`>`、`>=`、または`COLUMN`などのフィルタを含むクエリが不正確な結果を発生させるバグを修正します
    - メトリクスでのストアサイズが重い書き込み時に不正確となる問題を修正します
    - 複数のディスクに展開されたTiFlashがデータをリストアできない可能性のあるバグを修正します
    - 長時間実行した後にTiFlashがデルタデータをガベージコレクションできない可能性のある問題を修正します

+ ツール

    + バックアップ・リストア（BR）

        - バックアップおよびリストアの平均速度が不正確に計算されるバグを修正します [#1405](https://github.com/pingcap/br/issues/1405)

    + TiCDC

        - 統合テストでDDL Jobの重複が発生した際に発生する`ErrSchemaStorageTableMiss`エラーを修正します [#2422](https://github.com/pingcap/tiflow/issues/2422)
        - `ErrGCTTLExceeded`エラーが発生した場合にチェンジフィードを削除できないバグを修正します [#2391](https://github.com/pingcap/tiflow/issues/2391)
        - `capture list`コマンドの出力に古いキャプチャが表示される問題を修正します [#2388](https://github.com/pingcap/tiflow/issues/2388)
        - TiCDCプロセッサのデッドロック問題を修正します [#2017](https://github.com/pingcap/tiflow/pull/2017)
        - テーブルが再スケジュールされる際に複数のプロセッサが同じテーブルにデータを書き込む可能性があるため発生するデータ整合性の問題を修正します [#2230](https://github.com/pingcap/tiflow/issues/2230)
        - メタデータ管理で`EtcdWorker`スナップショットアイソレーションが違反されるバグを修正します [#2557](https://github.com/pingcap/tiflow/pull/2557)
        - DDLシンクエラーによりチェンジフィードを停止できない問題を修正します [#2552](https://github.com/pingcap/tiflow/issues/2552)
        - TiCDC Open Protocolの問題: TiCDCがトランザクションに変更がない場合に空の値を出力する問題を修正します [#2612](https://github.com/pingcap/tiflow/issues/2612)
        - 符号なしの`TINYINT`型でTiCDCがパニックを起こすバグを修正します [#2648](https://github.com/pingcap/tiflow/issues/2648)
        - TiCDCがあまりにも多くのリージョンをキャプチャすると発生するOOMを避けるためにgRPCウィンドウサイズを減らします [#2202](https://github.com/pingcap/tiflow/issues/2202)
        - TiCDCがあまりにも多くのリージョンをキャプチャすると発生するOOMを修正します [#2673](https://github.com/pingcap/tiflow/issues/2673)
        - `mysql.TypeString, mysql.TypeVarString, mysql.TypeVarchar`などのデータ型をJSONにエンコードする際に発生するプロセスのパニックを修正します [#2758](https://github.com/pingcap/tiflow/issues/2758)
        - 新しいチェンジフィードを作成する際に発生するメモリリークのバグを修正します [#2389](https://github.com/pingcap/tiflow/issues/2389)
        - チェンジフィードがスキーマ変更の終了TSで開始した場合にDDL処理が失敗するバグを修正します [#2603](https://github.com/pingcap/tiflow/issues/2603)
        - オーナーがDDLステートメントを実行中にクラッシュした際に発生する潜在的なDDLのロストの問題を修正します [#1260](https://github.com/pingcap/tiflow/issues/1260)
        - `SinkManager`のマップへの並行アクセスをセキュリティで制御する問題を修正します [#2298](https://github.com/pingcap/tiflow/pull/2298)