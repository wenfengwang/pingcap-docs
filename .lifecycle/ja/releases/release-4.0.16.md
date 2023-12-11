---
title: TiDB 4.0.16 リリースノート
---

# TiDB 4.0.16 リリースノート

リリース日: 2021年12月17日

TiDB バージョン: 4.0.16

## 互換性の変更

+ TiKV

    - v4.0.16 より前、TiDB は不正な UTF-8 文字列を Real 型に変換しようとすると、直接エラーが報告されていました。v4.0.16 より後、TiDB は文字列内の有効な UTF-8 プレフィックスに従って変換を処理します [#11466](https://github.com/tikv/tikv/issues/11466)

+ ツール

    + TiCDC

        - TiCDC がKafkaクラスターに大きすぎるメッセージを送信するのを防ぐため、Kafka Sink `max-message-bytes` のデフォルト値を1 MBに変更しました [#2962](https://github.com/pingcap/tiflow/issues/2962)
        - TiCDC がメッセージをKafkaパーティション全体により均等に配信するために、Kafka Sink `partition-num` のデフォルト値を3に変更しました [#3337](https://github.com/pingcap/tiflow/issues/3337)

## 改善点

+ TiDB

    - Grafana バージョンを7.5.7 から 7.5.11 にアップグレードしました

+ TiKV

    - バックアップ＆リストアまたは TiDB Lightning の Local-backend を使用してデータをリストアする際、SST ファイルを圧縮するために zstd アルゴリズムを採用することでディスク容量の消費を削減しました [#11469](https://github.com/tikv/tikv/issues/11469)

+ ツール

    + バックアップ＆リストア（BR）

        - 復元の堅牢性を向上させました [#27421](https://github.com/pingcap/tidb/issues/27421)

    + TiCDC

        - EtcdWorker にチクタク頻度制限を追加して、頻繁な etcd 書き込みが PD サービスに影響を与えるのを防ぎました [#3112](https://github.com/pingcap/tiflow/issues/3112)
        - TiKV リロード時のレート制御を最適化し、changefeed 初期化中の gPRC 混雑を減らしました [#3110](https://github.com/pingcap/tiflow/issues/3110)

## バグ修正

+ TiDB

    - 統計モジュールで範囲をポイントに変換する際にオーバーフローによるクエリパニックを修正しました [#23625](https://github.com/pingcap/tidb/issues/23625)
    - `ENUM` タイプデータをそのような関数のパラメータとして使用すると、制御関数（`IF` や `CASE WHEN` など）の結果が間違っていた問題を修正しました [#23114](https://github.com/pingcap/tidb/issues/23114)
    - `tidb_enable_vectorized_expression` (`on` または `off`) の値によって `GREATEST` 関数の結果が一貫しなかった問題を修正しました [#29434](https://github.com/pingcap/tidb/issues/29434)
    - 一部の場合において、プレフィックスインデックスにインデックスジョインを適用するとパニックする問題を修正しました [#24547](https://github.com/pingcap/tidb/issues/24547)
    - 一部の場合において、プランナーが `join` のために無効なプランをキャッシュする問題を修正しました [#28087](https://github.com/pingcap/tidb/issues/28087)
    - `sql_mode` が空の場合、TiDB が非NULL列に `null` を挿入できないバグを修正しました [#11648](https://github.com/pingcap/tidb/issues/11648)
    - `GREATEST` および `LEAST` 関数の結果型が間違っていた問題を修正しました [#29019](https://github.com/pingcap/tidb/issues/29019)
    - `grant` および `revoke` 操作を実行する際に発生する `privilege check fail` エラーを修正しました [#29675](https://github.com/pingcap/tidb/issues/29675)
    - `ENUM` データ型で `CASE WHEN` 関数を使用した際に発生するパニックを修正しました [#29357](https://github.com/pingcap/tidb/issues/29357)
    - ベクトル化式での `microsecond` 関数の結果が間違っていた問題を修正しました [#29244](https://github.com/pingcap/tidb/issues/29244)
    - ベクトル化式での `hour` 関数の結果が間違っていた問題を修正しました [#28643](https://github.com/pingcap/tidb/issues/28643)
    - 楽観的トランザクションの競合がトランザクションをブロックする可能性がある問題を修正しました [#11148](https://github.com/tikv/tikv/issues/11148)
    - `auto analyze` 結果からの不完全なログ情報の問題を修正しました [#29188](https://github.com/pingcap/tidb/issues/29188)
    - `SQL_MODE` が 'NO_ZERO_IN_DATE' の場合、無効なデフォルト日付を使用した際にエラーが報告されない問題を修正しました [#26766](https://github.com/pingcap/tidb/issues/26766)
    - Grafana において Coprocessor Cache パネルがメトリクスを表示しない問題を修正しました。現在、Grafana が `hits`/`miss`/`evict` の数を表示します [#26338](https://github.com/pingcap/tidb/issues/26338)
    - 同じパーティションを同時に切り詰めることが、DDL ステートメントのスタックを引き起こす問題を修正しました [#26229](https://github.com/pingcap/tidb/issues/26229)
    - `Decimal` を `String` に変換する際、長さ情報が間違っていた問題を修正しました [#29417](https://github.com/pingcap/tidb/issues/29417)
    - 複数のテーブルを `NATURAL JOIN` で結合した際にクエリ結果に余分な列が含まれる問題を修正しました [#29481](https://github.com/pingcap/tidb/issues/29481)
    - `IndexScan` がプレフィックスインデックスを使用して `TopN` を誤って `indexPlan` に押し込んでしまう問題を修正しました [#29711](https://github.com/pingcap/tidb/issues/29711)
    - `DOUBLE` タイプの自動増分列でのトランザクション再試行がデータの破損を引き起こす問題を修正しました [#29892](https://github.com/pingcap/tidb/issues/29892)

+ TiKV

    - 極端な条件で領域マージ、ConfChange、Snapshot が同時に発生するとパニックが発生する問題を修正しました [#11475](https://github.com/tikv/tikv/issues/11475)
    - 0 になる10進数の除算結果に負の符号が含まれてしまう問題を修正しました [#29586](https://github.com/pingcap/tidb/issues/29586)
    - by-instance gRPC リクエストの平均レイテンシが TiKV メトリクスにおいて正確でない問題を修正しました [#11299](https://github.com/tikv/tikv/issues/11299)
    - downstream データベースが欠落している場合に発生する TiCDC パニックの問題を修正しました [#11123](https://github.com/tikv/tikv/issues/11123)
    - チャンネルがいっぱいになると Raft 接続が切断されてしまう問題を修正しました [#11047](https://github.com/tikv/tikv/issues/11047)
    - `Int64` 型の `Max`/`Min` 関数の計算結果が誤っていた問題を修正しました。TiDB が符号付き整数かどうかを正しく識別できなかったためです [#10158](https://github.com/tikv/tikv/issues/10158)
    - Congest エラーによる CDC のスキャンリトライが頻繁に追加される問題を修正しました [#11082](https://github.com/tikv/tikv/issues/11082)

+ PD

    - TiKV ノードが削除された後に発生するパニック問題を修正しました [#4344](https://github.com/tikv/pd/issues/4344)
    - スタックしたリージョンシンカーによる遅いリーダー選出を修正しました [#3936](https://github.com/tikv/pd/issues/3936)
    - エビクトリーダースケジューラが健全でないピアを持つリージョンをスケジュールできるようにしました [#4093](https://github.com/tikv/pd/issues/4093)

+ TiFlash

    - ライブラリ `nsl` がないために TiFlash がいくつかのプラットフォームで起動しない問題を修正しました

+ Tools

    + TiDB ビンログ

        - 1 GB を超えるトランザクションを転送する際に Drainer が終了してしまうバグを修正しました [#28659](https://github.com/pingcap/tidb/issues/28659)

    + TiCDC

        - changefeed のチェックポイントラグに負の値が表示されるエラーを修正しました [#3010](https://github.com/pingcap/tiflow/issues/3010)
        - コンテナ環境における OOM を修正しました [#1798](https://github.com/pingcap/tiflow/issues/1798)
        - 複数の TiKV がクラッシュしたり強制的に再起動されたりする際に TiCDC レプリケーションが中断してしまう問題を修正しました [#3288](https://github.com/pingcap/tiflow/issues/3288)
        - DDL 処理後のメモリリーク問題を修正しました [#3174](https://github.com/pingcap/tiflow/issues/3174)
        - ErrGCTTLExceeded エラーが発生した際に changefeed が十分に速く失敗しない問題を修正しました [#3111](https://github.com/pingcap/tiflow/issues/3111)
        - アップストリーム TiDB インスタンスが予期せず終了した際に TiCDC レプリケーションタスクが終了する可能性がある問題を修正しました [#3061](https://github.com/pingcap/tiflow/issues/3061)
        - TiKVが同じRegionに重複したリクエストを送信するとTiCDCプロセスがパニックする問題を修正します [#2386](https://github.com/pingcap/tiflow/issues/2386)
        - TiCDCによって生成されるKafkaメッセージのボリュームが`max-message-size`で制約されない問題を修正します [#2962](https://github.com/pingcap/tiflow/issues/2962)
        - `force-replicate`が有効になっている場合、有効なインデックスがないパーティションテーブルが無視される可能性がある問題を修正します [#2834](https://github.com/pingcap/tiflow/issues/2834)
        - 新しいchangefeedを作成する際のメモリリークの問題を修正します [#2389](https://github.com/pingcap/tiflow/issues/2389)
        - Sinkコンポーネントがresolved tsを早めに進めることでデータの不整合を引き起こす可能性がある問題を修正します [#3503](https://github.com/pingcap/tiflow/issues/3503)
        - Stockデータをスキャンする際にTiKVがGCを実行すると失敗する可能性がある問題を修正します [#2470](https://github.com/pingcap/tiflow/issues/2470)
        - changefeedの更新コマンドがグローバルコマンドラインパラメータを認識しない問題を修正します [#2803](https://github.com/pingcap/tiflow/issues/2803)
        - changefeedの更新コマンドがグローバルコマンドラインパラメータを認識しない問題を修正します [#2803](https://github.com/pingcap/tiflow/issues/2803)