---
title: TiDB 5.0.3 リリースノート
---

# TiDB 5.0.3 リリースノート

リリース日: 2021年7月2日

TiDB バージョン: 5.0.3

## 互換性の変更

+ TiDB

    - v4.0 クラスターを v5.0 またはそれ以降のバージョン（開発版またはv5.1）にアップグレードすると、`tidb_multi_statement_mode` 変数のデフォルト値が `WARN` から `OFF` に変更されます
    - TiDB は、MySQL 5.7のnoop変数 `innodb_default_row_format` と互換性があります。この変数を設定しても効果はありません。[#23541](https://github.com/pingcap/tidb/issues/23541)

## 機能の強化

+ ツール

    + TiCDC

        - changefeed情報とノードのヘルス情報を取得するためのHTTP APIを追加 [#1955](https://github.com/pingcap/tiflow/pull/1955)
        - kafka sinkにSASL/SCRAMサポートを追加 [#1942](https://github.com/pingcap/tiflow/pull/1942)
        - TiCDCがサーバーレベルで`--data-dir`をサポートするように変更 [#2070](https://github.com/pingcap/tiflow/pull/2070)

## 改良

+ TiDB

    - `TopN`演算子をTiFlashにプッシュダウンするサポートを追加 [#25162](https://github.com/pingcap/tidb/pull/25162)
    - 組み込み関数 `json_unquote()` をTiKVにプッシュダウンするサポートを追加 [#24415](https://github.com/pingcap/tidb/issues/24415)
    - dualテーブルからunionブランチを除去するサポートを追加 [#25614](https://github.com/pingcap/tidb/pull/25614)
    - 組み込み関数 `replace()` をTiFlashにプッシュダウンするサポートを追加 [#25565](https://github.com/pingcap/tidb/pull/25565)
    - 組み込み関数 `unix_timestamp()`、`concat()`、`year()`、`day()`、`datediff()`、`datesub()`、および`concat_ws()` をTiFlashにプッシュダウンするサポートを追加 [#25564](https://github.com/pingcap/tidb/pull/25564)
    - 集約演算子のコストファクターを最適化 [#25241](https://github.com/pingcap/tidb/pull/25241)
    - `Limit`演算子をTiFlashにプッシュダウンするサポートを追加 [#25159](https://github.com/pingcap/tidb/pull/25159)
    - 組み込み関数 `str_to_date` をTiFlashにプッシュダウンするサポートを追加 [#25148](https://github.com/pingcap/tidb/pull/25148)
    - MPP外部結合において、テーブル行数に基づいてビルドテーブルを選択するサポートを追加 [#25142](https://github.com/pingcap/tidb/pull/25142)
    - 組み込み関数 `left()`、`right()`、`abs()` をTiFlashにプッシュダウンするサポートを追加 [#25133](https://github.com/pingcap/tidb/pull/25133)
    - ブロードキャストカルテシアン結合をTiFlashにプッシュダウンするサポートを追加 [#25106](https://github.com/pingcap/tidb/pull/25106)
    - `Union All`演算子をTiFlashにプッシュダウンするサポートを追加 [#25051](https://github.com/pingcap/tidb/pull/25051)
    - 異なるTiFlashノード間でMPPクエリワークロードを平衡化するサポートを追加 [#24724](https://github.com/pingcap/tidb/pull/24724)
    - MPPクエリ実行後にキャッシュ中の古いリージョンを無効化するサポートを追加 [#24432](https://github.com/pingcap/tidb/pull/24432)
    - 組み込み関数 `str_to_date` のMySQL互換性を形式指定子 `%b/%M/%r/%T` のために改善 [#25767](https://github.com/pingcap/tidb/pull/25767)

+ TiKV

    - TiCDC sinkのメモリ消費を制限する [#10305](https://github.com/tikv/tikv/pull/10305)
    - TiCDCの旧値キャッシュにメモリに制限を追加 [#10313](https://github.com/tikv/tikv/pull/10313)

+ PD

    - TiDB Dashboardをv2021.06.15.1に更新 [#3798](https://github.com/pingcap/pd/pull/3798)

+ TiFlash

    - `STRING`タイプを`DOUBLE`タイプにキャストするサポートを追加
    - `STR_TO_DATE()`関数をサポート
    - 右外部結合の非結合データを複数スレッドで最適化
    - カルテシアン結合をサポート
    - `LEFT()`および`RIGHT()`関数をサポート
    - MPPクエリで古いリージョンを自動的に無効化するサポート
    - `ABS()`関数をサポート

+ ツール

    + TiCDC

        - gRPCの再接続ロジックを改善し、KVクライアントのスループットを増加させる [#1586](https://github.com/pingcap/tiflow/issues/1586) [#1501](https://github.com/pingcap/tiflow/issues/1501#issuecomment-820027078) [#1682](https://github.com/pingcap/tiflow/pull/1682) [#1393](https://github.com/pingcap/tiflow/issues/1393) [#1847](https://github.com/pingcap/tiflow/pull/1847) [#1905](https://github.com/pingcap/tiflow/issues/1905) [#1904](https://github.com/pingcap/tiflow/issues/1904)
        - sorter I/Oエラーをユーザーフレンドリーにする

## バグ修正

+ TiDB

    - `SET`タイプの列でマージ結合を使用すると、誤った結果が返される問題を修正 [#25669](https://github.com/pingcap/tidb/issues/25669)
    - `IN`式の引数でデータの破損問題を修正 [#25591](https://github.com/pingcap/tidb/issues/25591)
    - GCのセッションがグローバル変数に影響を受けないようにするための修正 [#24976](https://github.com/pingcap/tidb/issues/24976)
    - ウィンドウ関数クエリで`limit`を使用した場合に発生するパニックの問題を修正 [#25344](https://github.com/pingcap/tidb/issues/25344)
    - `LIMIT`を使用してパーティションテーブルをクエリすると、間違った値が返される問題を修正 [#24636](https://github.com/pingcap/tidb/issues/24636)
    - `ENUM`または`SET`タイプの列に対して`IFNULL`の影響が正しく反映されない問題を修正 [#24944](https://github.com/pingcap/tidb/issues/24944)
    - 結合サブクエリで`count`を`first_row`に変更することで発生する誤った結果を修正 [#24865](https://github.com/pingcap/tidb/issues/24865)
    - `ParallelApply`の下で`TopN`演算子を使用することで発生するクエリがハングする問題を修正 [#24930](https://github.com/pingcap/tidb/issues/24930)
    - 複数列プレフィックスインデックスを使用してSQLステートメントを実行すると、予期しないほど多くの結果が返される問題を修正 [#24356](https://github.com/pingcap/tidb/issues/24356)
    - `<=>`演算子が正しく機能しない問題を修正 [#24477](https://github.com/pingcap/tidb/issues/24477)
    - パラレル`Apply`演算子のデータ競合の問題を修正 [#23280](https://github.com/pingcap/tidb/issues/23280)
    - PartitionUnion演算子のIndexMerge結果を並べ替える際に`index out of range`エラーが発生する問題を修正 [#23919](https://github.com/pingcap/tidb/issues/23919)
    - `tidb_snapshot`変数を予期しない大きな値に設定すると、トランザクションの分離を損なう可能性がある問題を修正 [#25680](https://github.com/pingcap/tidb/issues/25680)
    - ODBC形式の定数（例:`{d '2020-01-01'}`）を式として使用できない問題を修正 [#25531](https://github.com/pingcap/tidb/issues/25531)
    - `SELECT DISTINCT`を`Batch Get`に変換すると誤った結果が生じる問題を修正 [#25320](https://github.com/pingcap/tidb/issues/25320)
    - TiFlashからTiKVへのクエリのバックオフトリガが発動しない問題を修正 [#23665](https://github.com/pingcap/tidb/issues/23665) [#24421](https://github.com/pingcap/tidb/issues/24421)
    - `only_full_group_by`を確認する際に`index-out-of-range`エラーが発生する問題を修正 [#23839](https://github.com/pingcap/tidb/issues/23839))
    - 相関サブクエリのインデックス結合の結果が誤っている問題を修正 [#25799](https://github.com/pingcap/tidb/issues/25799)

+ TiKV

    - 誤った`tikv_raftstore_hibernated_peer_state`メトリックの修正 [#10330](https://github.com/tikv/tikv/issues/10330)
- コプロセッサーの `json_unquote()` 関数の間違った引数タイプを修正する[#10176](https://github.com/tikv/tikv/issues/10176)
    - いくつかのケースで ACID を壊さないように、優雅なシャットダウン中にコールバックのクリアをスキップする[#10353](https://github.com/tikv/tikv/issues/10353) [#10307](https://github.com/tikv/tikv/issues/10307)
    - リーダーでのレプリカ読み取りにおいて読み取りインデックスが共有されるバグを修正する[#10347](https://github.com/tikv/tikv/issues/10347)
    - `DOUBLE` を `DOUBLE` に変換する誤った関数を修正する[#25200](https://github.com/pingcap/tidb/issues/25200)
+ PD
    - スケジューラーが開始された後に TTL 設定を読み込む際に発生するデータ競合問題を修正する[#3771](https://github.com/tikv/pd/issues/3771)
    - TiDB の `TIKV_REGION_PEERS` テーブルの `is_learner` フィールドが正しくないバグを修正する[#3372](https://github.com/tikv/pd/issues/3372) [#24293](https://github.com/pingcap/tidb/issues/24293)
    - あるゾーン内のすべての TiKV ノードがオフラインまたはダウンしている場合、PD が他のゾーンにレプリカをスケジュールしない問題を修正する[#3705](https://github.com/tikv/pd/issues/3705)
    - 散在リージョンスケジューラーが追加された後に PD がパニックになる可能性がある問題を修正する[#3762](https://github.com/tikv/pd/pull/3762)
+ TiFlash
    - 分割の失敗のために TiFlash が繰り返し再起動する問題を修正する
    - TiFlash がデルタデータを削除できない可能性のある問題を修正する
    - `CAST` 関数における非バイナリ文字の間違ったパディングを修正するバグを修正する
    - 複雑な `GROUP BY` 列を持つ集計クエリを処理する際の結果が正しくない問題を修正する
    - 大量の書き込みプレッシャー下で発生する TiFlash パニックの問題を修正する
    - 右側の join キーが nullable で、左側の join キーが nullable でない場合に発生する TiFlash パニックを修正する
    - `read-index` リクエストに長い時間がかかる可能性のある問題を修正する
    - 読み込み負荷が重い場合に発生するパニックの問題を修正する
    - `Date_Format` 関数が `STRING` 型の引数と `NULL` 値で呼び出された場合に発生する可能性があるパニックの問題を修正する
+ Tools
    + TiCDC
        - チェックポイントを更新する際に TiCDC オーナーが終了する問題を修正する[#1902](https://github.com/pingcap/tiflow/issues/1902)
        - MySQL シンクでエラーが発生し一時停止した後にいくつかの MySQL 接続がリークする可能性のあるバグを修正する[#1946](https://github.com/pingcap/tiflow/pull/1946)
        - `/proc/meminfo` を読み込めない際に発生する TiCDC のパニックの問題を修正する[#2024](https://github.com/pingcap/tiflow/pull/2024)
        - TiCDC のランタイムメモリ消費を削減する[#2012](https://github.com/pingcap/tiflow/pull/2012) [#1958](https://github.com/pingcap/tiflow/pull/1958)
        - 後で解決された ts の計算により TiCDC サーバーがパニックする可能性のあるバグを修正する[#1576](https://github.com/pingcap/tiflow/issues/1576)
        - プロセッサーの潜在的なデッドロックの問題を修正する[#2142](https://github.com/pingcap/tiflow/pull/2142)
    + バックアップおよびリストア（BR）
        - リストア中にすべてのシステムテーブルがフィルタリングされるバグを修正する[#1197](https://github.com/pingcap/br/issues/1197) [#1201](https://github.com/pingcap/br/issues/1201)
        - リストア中に TDE が有効になっている場合に、バックアップとリストアが "ファイルが既に存在します" エラーを報告する問題を修正する[#1179](https://github.com/pingcap/br/issues/1179)
    + TiDB Lightning
        - 特定の特殊データに対する TiDB Lightning パニックの問題を修正する[#1213](https://github.com/pingcap/br/issues/1213)
        - インポートされた大きな CSV ファイルを分割する際に報告される EOF エラーを修正する[#1133](https://github.com/pingcap/br/issues/1133)
        - `auto_increment` 列が `FLOAT` または `DOUBLE` 型のテーブルをインポートする際に過剰に大きな基本値が生成されるバグを修正する[#1186](https://github.com/pingcap/br/pull/1186)
        - Parquet ファイル内の `DECIMAL` 型データを解析できない TiDB の問題を修正する[#1277](https://github.com/pingcap/br/pull/1277)