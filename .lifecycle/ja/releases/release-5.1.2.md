---
title: TiDB 5.1.2 リリースノート
---

# TiDB 5.1.2 リリースノート

リリース日: 2021年9月27日

TiDB バージョン: 5.1.2

## 互換性の変更

+ TiDB

    + 以下のバグ修正は実行結果を変更する可能性があり、アップグレードの非互換性を引き起こす可能性があります:

        - `greatest(datetime) union null` が空の文字列を返す問題を修正しました [#26532](https://github.com/pingcap/tidb/issues/26532)
        - `having` 句が正しく機能しない可能性がある問題を修正しました [#26496](https://github.com/pingcap/tidb/issues/26496)
        - `between` 式周辺の照合が異なる場合に誤った実行結果が発生する問題を修正しました [#27146](https://github.com/pingcap/tidb/issues/27146)
        - `group_concat` 関数の列が非バイナリの照合を持つ場合に誤った実行結果が発生する問題を修正しました [#27429](https://github.com/pingcap/tidb/issues/27429)
        - 新しい照合が有効になっている場合、複数の列で `count(distinct)` 式を使用すると間違った結果が返る問題を修正しました [#27091](https://github.com/pingcap/tidb/issues/27091)
        - `extract` 関数の引数が負の期間の場合に発生する結果の誤りを修正しました [#27236](https://github.com/pingcap/tidb/issues/27236)
        - `SQL_MODE` が 'STRICT_TRANS_TABLES' の場合に無効な日付を挿入してもエラーが報告されない問題を修正しました [#26762](https://github.com/pingcap/tidb/issues/26762)
        - `SQL_MODE` が 'NO_ZERO_IN_DATE' の場合に無効なデフォルト日付を使用してもエラーが報告されない問題を修正しました [#26766](https://github.com/pingcap/tidb/issues/26766)

+ ツール

    + TiCDC

        - 互換性のあるバージョンを `5.1.0-alpha` から `5.2.0-alpha` に設定しました [#2659](https://github.com/pingcap/tiflow/pull/2659)

## 改善点

+ TiDB

    - ヒストグラムの行数による自動解析をトリガーし、このトリガーアクションの精度を向上させるように設定しました  [#24237](https://github.com/pingcap/tidb/issues/24237)

+ TiKV

    - TiCDC 構成を動的に変更する機能をサポートしました [#10645](https://github.com/tikv/tikv/issues/10645)
    - ネットワーク帯域幅を保存するために、Resolved TS メッセージのサイズを削減しました [#2448](https://github.com/pingcap/tiflow/issues/2448)
    - 個々のストアが報告するハートビートメッセージ内のピアの統計数を制限しました [#10621](https://github.com/tikv/tikv/pull/10621)

+ PD

    - 空のリージョンをスケジュール可能にし、scatter range スケジューラーで別個の許容設定を使用するように設定しました [#4117](https://github.com/tikv/pd/pull/4117)
    - PD 間でのリージョン情報の同期性能を向上させました [#3933](https://github.com/tikv/pd/pull/3933)
    - 生成されたオペレータに基づいてストアの再試行制限を動的に調整する機能をサポートしました [#3744](https://github.com/tikv/pd/issues/3744)

+ TiFlash

    - `DATE()` 関数をサポートしました
    - インスタンスごとの書き込みスループット用の Grafana パネルを追加しました
    - `leader-read` プロセスのパフォーマンスを最適化しました
    - MPP タスクのキャンセル処理を加速しました

+ ツール

    + TiCDC

        - Unified Sorter がデータをソートする際のメモリ管理を最適化しました [#2553](https://github.com/pingcap/tiflow/issues/2553)
        - コンカレンシーが高い場合により少ないゴールーチンを使用するために workerpool を最適化しました  [#2211](https://github.com/pingcap/tiflow/issues/2211)
        - テーブルのリージョンが TiKV ノードから移動する際のゴールーチン使用量を削減しました [#2284](https://github.com/pingcap/tiflow/issues/2284)
        - グローバル gRPC コネクションプールを追加し、KV クライアント間で gRPC 接続を共有するようにしました [#2534](https://github.com/pingcap/tiflow/pull/2534)
        - メジャーおよびマイナーバージョン間での TiCDC クラスターの操作を禁止しました [#2599](https://github.com/pingcap/tiflow/pull/2599)

    + Dumpling

        - `START TRANSACTION ... WITH CONSISTENT SNAPSHOT` および `SHOW CREATE TABLE` をサポートしていない MySQL 互換データベースのバックアップをサポートしました [#309](https://github.com/pingcap/dumpling/issues/309)

## バグ修正

+ TiDB

    - ハッシュ列が `ENUM` 型でハッシュ結合の誤った結果が発生する可能性がある場合の潜在的な誤った結果を修正しました [#27893](https://github.com/pingcap/tidb/issues/27893)
    - アイドルコネクションの再利用が一部の特殊なケースでリクエストの送信をブロックする可能性があるバッチクライアントのバグを修正しました [#27678](https://github.com/pingcap/tidb/pull/27678)
    - `FLOAT64` 型のオーバーフローチェックが MySQL と異なる場合がある問題を修正しました [#23897](https://github.com/pingcap/tidb/issues/23897)
    - `pd is timeout` エラーが返るべきところで TiDB が `unknow` エラーを返す問題を修正しました [#26147](https://github.com/pingcap/tidb/issues/26147)
    - `case when` 式の文字セットと照合を修正しました [#26662](https://github.com/pingcap/tidb/issues/26662)
    - MPP クエリのための `can not found column in Schema column` エラーを防ぐための潜在的なバグを修正しました [#28148](https://github.com/pingcap/tidb/pull/28148)
    - TiFlash がシャットダウンしているときに TiDB がパニックを引き起こす可能性があるバグを修正しました [#28096](https://github.com/pingcap/tidb/issues/28096)
    - `enum like 'x%'` を使用した場合の誤った範囲を修正しました [#27130](https://github.com/pingcap/tidb/issues/27130)
    - IndexLookupJoin と組み合わせて使用される Common Table Expression (CTE) のデッドロック問題を修正しました [#27410](https://github.com/pingcap/tidb/issues/27410)
    - `INFORMATION_SCHEMA.DEADLOCKS` テーブルに再試行可能なデッドロックが誤って記録されるバグを修正しました [#27400](https://github.com/pingcap/tidb/issues/27400)
    - パーティション化されたテーブルの `TABLESAMPLE` クエリ結果が期待どおりに並べ替えられない問題を修正しました [#27349](https://github.com/pingcap/tidb/issues/27349)
    - 未使用の `/debug/sub-optimal-plan` HTTP API を削除しました  [#27265](https://github.com/pingcap/tidb/pull/27265)
    - ハッシュパーティションテーブルが符号なしのデータを処理する際にクエリが誤った結果を返す可能性があるバグを修正しました [#26569](https://github.com/pingcap/tidb/issues/26569)
    - `NO_UNSIGNED_SUBTRACTION` が設定されている場合にパーティションの作成に失敗するバグを修正しました [#26765](https://github.com/pingcap/tidb/issues/26765)
    - `Apply` が `Join` に変換されたときに `distinct` フラグが欠落する問題を修正しました [#26958](https://github.com/pingcap/tidb/issues/26958)
    - 新たに回復した TiFlash ノードでクエリをブロックするのを避けるためにブロックの期間を設定しました [#26897](https://github.com/pingcap/tidb/pull/26897)
    - CTE が複数回参照される場合に発生する可能性があるバグを修正しました [#26212](https://github.com/pingcap/tidb/issues/26212)
    - MergeJoin を使用したときの CTE バグを修正しました [#25474](https://github.com/pingcap/tidb/issues/25474)
    - 通常のテーブルがパーティション化されたテーブルに結合した場合、`SELECT FOR UPDATE` ステートメントがデータを正しくロックしないバグを修正しました [#26251](https://github.com/pingcap/tidb/issues/26251)
    - 通常のテーブルがパーティション化されたテーブルに結合した場合、`SELECT FOR UPDATE` ステートメントがエラーを返すバグを修正しました [#26250](https://github.com/pingcap/tidb/issues/26250)
    - `PointGet` がロックの解決に lite バージョンを使用しない問題を修正しました [#26562](https://github.com/pingcap/tidb/pull/26562)

+ TiKV

    - TiKV が v3.x から後のバージョンにアップグレードされた後に発生するパニック問題を修正しました [#10902](https://github.com/tikv/tikv/issues/10902)
    - 破損したスナップショットファイルによってディスクフルの問題が発生する可能性があるバグを修正しました [#10813](https://github.com/tikv/tikv/issues/10813)
    - TiKV コプロセッサのスローログがリクエスト処理に費やされた時間のみを考慮するようにしました [#10841](https://github.com/tikv/tikv/issues/10841)
    - スロガースレッドが過負荷になりキューが満杯になったときにスレッドをブロックする代わりにログを破棄するようにしました [#10841](https://github.com/tikv/tikv/issues/10841)
    - Coprocessorリクエストの処理がタイムアウトしたときに発生するパニックの問題を修正 [#10852](https://github.com/tikv/tikv/issues/10852)
    - Titanを有効にした状態からの事前5.0バージョンからのアップグレード時に発生するTiKVのパニック問題を修正 [#10842](https://github.com/tikv/tikv/pull/10842)
    - TiKVの新しいバージョンをv5.0.xにダウングレードできない問題を修正 [#10842](https://github.com/tikv/tikv/pull/10842)
    - TiKVがデータをRocksDBに取り込む前にファイルを削除する可能性のある問題を修正 [#10438](https://github.com/tikv/tikv/issues/10438)
    - 左の悲観的なロックによって引き起こされる解析の失敗問題を修正 [#26404](https://github.com/pingcap/tidb/issues/26404)

+ PD

    - PDがダウンしたペアを適時に修正しない問題を修正 [#4077](https://github.com/tikv/pd/issues/4077)
    - デフォルトの配置ルールのレプリカ数が `replication.max-replicas` が更新された後も一定のままである問題を修正 [#3886](https://github.com/tikv/pd/issues/3886)
    - TiKVのスケーリングアウト時にPDがパニックする可能性のあるバグを修正 [#3868](https://github.com/tikv/pd/issues/3868)
    - クラスタにevict leaderスケジューラがある場合にHot Regionスケジューラが機能しないバグを修正 [#3697](https://github.com/tikv/pd/issues/3697)

+ TiFlash

    - TiFlashがMPP接続を確立できないときに予期しない結果の問題を修正
    - TiFlashが複数のディスクに展開された場合にデータの不整合の潜在的な問題を修正
    - TiFlashサーバが負荷が高い状態でMPPクエリが誤った結果を取得するバグを修正
    - MPPクエリが永遠にハングする可能性のあるバグを修正
    - ストアの初期化とDDLを同時に実行する際に発生するパニックの問題を修正
    - `CONSTANT`、`<`、`<=`、`>`、`>=`、または `COLUMN` のフィルタが含まれるクエリで誤った結果が取得されるバグを修正
    - `Snapshot`が複数のDDL操作と同時に適用されるときに発生する潜在的なパニックの問題を修正
    - メトリクスのストアサイズが重い書き込み時に不正確である問題を修正
    - TiFlashが長時間実行した後にデルタデータをガベージコレクションできない可能性のある問題を修正
    - 新しい照合が有効になっている場合に誤った結果が発生する問題を修正
    - ロックの解決時に発生する潜在的なパニックの問題を修正
    - メトリクスが間違った値を表示する可能性のあるバグを修正

+ Tools

    + Backup & Restore (BR)

        - データバックアップと復元中の平均スピードが正確でない問題を修正 [#1405](https://github.com/pingcap/br/issues/1405)

    + Dumpling

        - 一部のMySQLバージョン（8.0.3および8.0.23）で `show table status` が誤った結果を返すとDumplingが保留状態になる問題を修正 [#322](https://github.com/pingcap/dumpling/issues/322)
        - デフォルトの `sort-engine` オプションで4.0.xクラスタとのCLI互換性の問題を修正 [#2373](https://github.com/pingcap/tiflow/issues/2373)

    + TiCDC

        - 文字列型の値が `string` または `[]byte` である場合にパニックを引き起こす可能性のあるJSONエンコーディングのバグを修正 [#2758](https://github.com/pingcap/tiflow/issues/2758)
        - OOMを回避するためにgRPCウィンドウサイズを縮小する [#2673](https://github.com/pingcap/tiflow/issues/2673)
        - 高いメモリ圧力下でのgRPC `keepalive` エラーを修正するバグを修正 [#2202](https://github.com/pingcap/tiflow/issues/2202)
        - 符号なし `tinyint` がTiCDCをパニックさせるバグを修正 [#2648](https://github.com/pingcap/tiflow/issues/2648)
        - 1つのトランザクションで変更がない場合に空の値が出力されないようにTiCDC Open Protocolにおける空の値の問題を修正 [#2612](https://github.com/pingcap/tiflow/issues/2612)
        - マニュアル再起動中のDDL処理のバグを修正 [#2603](https://github.com/pingcap/tiflow/issues/2603)
        - `EtcdWorker`がメタデータを管理する際にスナップショット分離が誤って違反される可能性のある問題を修正 [#2559](https://github.com/pingcap/tiflow/pull/2559)
        - TiCDCがテーブルを再スケジューリングしている際に複数のプロセッサが同じテーブルにデータを書き込む可能性のあるバグを修正 [#2230](https://github.com/pingcap/tiflow/issues/2230)
        - changefeedが `ErrSchemaStorageTableMiss` エラーを取得したときに予期しないリセットが発生する可能性のあるバグを修正 [#2422](https://github.com/pingcap/tiflow/issues/2422)
        - changefeedが `ErrGCTTLExceeded` エラーを取得したときにchangefeedを削除できないバグを修正 [#2391](https://github.com/pingcap/tiflow/issues/2391)
        - 大きなテーブルをcdclogに同期できない問題を修正 [#1259](https://github.com/pingcap/tiflow/issues/1259) [#2424](https://github.com/pingcap/tiflow/issues/2424)