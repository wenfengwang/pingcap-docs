---
title: TiDB 5.0.2 更新ノート
---

# TiDB 5.0.2 更新ノート

リリース日: 2021年6月10日

TiDB バージョン: 5.0.2

## 互換性の変更

+ Tools

    + TiCDC

        - `cdc cli changefeed` コマンドで `--sort-dir` を非推奨にしました。代わりにユーザーは `cdc server` コマンドで `--sort-dir` を設定できます。[#1795](https://github.com/pingcap/tiflow/pull/1795)

## 新機能

+ TiKV

    - デフォルトで Hibernate リージョン機能を有効にしました [#10266](https://github.com/tikv/tikv/pull/10266)

## 改善点

+ TiDB

    - キャッシュされた統計情報が最新である場合は、高いCPU使用率を避けるために`mysql.stats_histograms` テーブルを頻繁に読み込まないようにしました [#24317](https://github.com/pingcap/tidb/pull/24317)

+ TiKV

    - BR がS3互換ストレージをバーチャルホストアドレッシングモードを使用してサポートするようにしました [#10243](https://github.com/tikv/tikv/pull/10243)
    - TiCDCのスキャン速度に対する逆流圧力をサポートするようにしました [#10151](https://github.com/tikv/tikv/pull/10151)
    - TiCDCの初期スキャンのメモリ使用量を削減しました [#10133](https://github.com/tikv/tikv/pull/10133)
    - TiCDCの悲観的トランザクションでの Old Value 機能のキャッシュヒット率を向上させました [#10089](https://github.com/tikv/tikv/pull/10089)
    - ホットスポット書き込みがある場合にリージョンサイズの成長が分割速度を超える問題を緩和するために、リージョンをより均等に分割するようにしました [#9785](https://github.com/tikv/tikv/issues/9785)

+ TiFlash

    - DDLジョブとデータ読み込みが互いにブロックしないようにテーブルロックを最適化しました
    - `INTEGER`または `REAL` 型を `REAL` 型にキャストすることをサポートしました

+ Tools

    + TiCDC

        - テーブルメモリ消費量の監視メトリクスを追加しました [#1885](https://github.com/pingcap/tiflow/pull/1885)
        - ソーティングステージ中のメモリとCPU使用量を最適化しました [#1863](https://github.com/pingcap/tiflow/pull/1863)
        - ユーザーを混乱させる可能性のあるいくつかの無用なログ情報を削除しました [#1759](https://github.com/pingcap/tiflow/pull/1759)

    + Backup & Restore (BR)

        - 曖昧なエラーメッセージを明確にしました [#1132](https://github.com/pingcap/br/pull/1132)
        - バックアップのクラスターバージョンをチェックするようにサポートしました [#1091](https://github.com/pingcap/br/pull/1091)
        - `mysql` スキーマのシステムテーブルのバックアップとリストアをサポートしました [#1143](https://github.com/pingcap/br/pull/1143) [#1078](https://github.com/pingcap/br/pull/1078)

    + Dumpling

        - バックアップ操作が失敗した場合にエラーが出力されない問題を修正しました [#280](https://github.com/pingcap/dumpling/pull/280)

## バグ修正

+ TiDB

    - 一部の場合におけるプレフィックスインデックスとインデックス結合の使用によるパニックの問題を修正しました [#24547](https://github.com/pingcap/tidb/issues/24547) [#24716](https://github.com/pingcap/tidb/issues/24716) [#24717](https://github.com/pingcap/tidb/issues/24717)
    - トランザクション内で `point get` ステートメントが誤って `prepared plan cache` を使用する問題を修正しました [#24741](https://github.com/pingcap/tidb/issues/24741)
    - 照合が `ascii_bin` または `latin1_bin` の場合に、間違ったプレフィックスインデックス値を書き込む問題を修正しました [#24569](https://github.com/pingcap/tidb/issues/24569)
    - 進行中のトランザクションがGCワーカーによって中断される可能性がある問題を修正しました [#24591](https://github.com/pingcap/tidb/issues/24591)
    - `new-collation` が有効にされているが `new-row-format` が無効にされている場合に、`point query` がクラスタリングインデックスで誤った結果を取得する可能性があるバグを修正しました [#24541](https://github.com/pingcap/tidb/issues/24541)
    - シャッフルハッシュ結合のためのパーティションキーの変換を再構成しました [#24490](https://github.com/pingcap/tidb/pull/24490)
    - `HAVING` 句を含むクエリのプランをビルドする際に発生するパニックの問題を修正しました [#24045](https://github.com/pingcap/tidb/issues/24045)
    - 列の刈り込みの改善により、`Apply` および `Join` 演算子の結果が誤る問題を修正しました [#23887](https://github.com/pingcap/tidb/issues/23887)
    - 非同期コミットからフォールバックしたプライマリロックが解決できないバグを修正しました [#24384](https://github.com/pingcap/tidb/issues/24384)
    - 統計のGC問題によって、重複したfm-sketchレコードが発生する可能性がある問題を修正しました [#24357](https://github.com/pingcap/tidb/pull/24357)
    - 悲観的ロックが `ErrKeyExists` エラーを受信した際に不必要な悲観的ロールバックを避けるようにしました [#23799](https://github.com/pingcap/tidb/issues/23799)
    - numericリテラルが `ANSI_QUOTES` を含む `sql_mode` に認識されない問題を修正しました [#24429](https://github.com/pingcap/tidb/issues/24429)
    - `INSERT INTO table PARTITION (<partitions>) ... ON DUPLICATE KEY UPDATE` のようなステートメントが、リストされていないパーティションからデータを読み取らないようにしました [#24746](https://github.com/pingcap/tidb/issues/24746)
    - `GROUP BY` および `UNION` を含むSQLステートメントが `index out of range` エラーをポテンシャルに引き起こす問題を修正しました [#24281](https://github.com/pingcap/tidb/issues/24281)
    - `CONCAT` 関数が照合を誤って処理する問題を修正しました [#24296](https://github.com/pingcap/tidb/issues/24296)
    - `collation_server` グローバル変数が新しいセッションで効果を持たない問題を修正しました [#24156](https://github.com/pingcap/tidb/pull/24156)

+ TiKV

    - 古い値を読み込むことによって引き起こされるTiCDCのOOM問題を修正しました [#9996](https://github.com/tikv/tikv/issues/9996) [#9981](https://github.com/tikv/tikv/issues/9981)
    - クラスタリングプライマリキーカラムのセカンダリインデックスにおける空の値の問題を修正しました：照合が `latin1_bin` の場合 [#24548](https://github.com/pingcap/tidb/issues/24548)
    - パニックが発生した際にTiKVがコアダンプファイルを生成できるように`abort-on-panic`構成を追加しました。ユーザーは引き続き環境を正しく設定してコアダンプを有効にする必要があります [#10216](https://github.com/tikv/tikv/pull/10216)
    - TiKVがビジーでない状態の場合に発生する`point get` クエリのパフォーマンスの回帰問題を修正しました [#10046](https://github.com/tikv/tikv/issues/10046)

+ PD

    - 多くのストアが存在する場合にPDリーダー再選が遅い問題を修正しました [#3697](https://github.com/tikv/pd/issues/3697)
    - 存在しないストアからevict leaderスケジューラを削除しようとした際に発生するパニック問題を修正しました [#3660](https://github.com/tikv/pd/issues/3660)
    - オフラインペアがマージされた後に統計情報が更新されない問題を修正しました [#3611](https://github.com/tikv/pd/issues/3611)

+ TiFlash

    - 共有デルタインデックスのクローン中に不正な結果が発生する問題を修正しました
    - TiFlashが不完全なデータで再起動に失敗する可能性がある問題を修正しました
    - 古いdmファイルが自動的に削除されない問題を修正しました
    - Compaction Filter機能が有効になっている際に発生する可能性のあるパニックを修正しました
    - `ExchangeSender` が重複したデータを送信する問題を修正しました
    - asyncコミットからのフォールバックされたプライマリーロックがTiFlashによって解決されない問題を修正しました
    - `TIMEZONE`タイプのキャスト結果が`TIMESTAMP`タイプを含む場合に、誤った結果が返される問題を修正しました
    - セグメント分割中に発生するTiFlashのパニック問題を修正しました
    - 非ルートMPPタスクの実行情報が正確でない問題を修正しました

+ Tools

    + TiCDC

        - Avro出力においてタイムゾーン情報が失われる問題を修正しました [#1712](https://github.com/pingcap/tiflow/pull/1712)
        - Unified Sorterにおけるステール一時ファイルのクリーンアップをサポートし、`sort-dir` ディレクトリの共有を禁止する問題を修正しました [#1742](https://github.com/pingcap/tiflow/pull/1742)
        - 多くのステールリージョンが存在する際に発生するKVクライアントのデッドロックバグを修正しました [#1599](https://github.com/pingcap/tiflow/issues/1599)
        - `--cert-allowed-cn` フラグの誤ったヘルプ情報を修正する [#1697](https://github.com/pingcap/tiflow/pull/1697)
        - MySQL にデータをレプリケートする際に `explicit_defaults_for_timestamp` のアップデートを元に戻す `[SUPER` 権限が必要です] [#1750](https://github.com/pingcap/tiflow/pull/1750)
        - メモリオーバーフローのリスクを減らすために、シンクフローコントロールをサポートする [#1840](https://github.com/pingcap/tiflow/pull/1840)
        - テーブルを移動する際にレプリケーションタスクが停止する可能性があるバグを修正する [#1828](https://github.com/pingcap/tiflow/pull/1828)
        - TiKV GC セーフポイントが TiCDC changefeed チェックポイントの停滞によりブロックされる問題を修正する [#1759](https://github.com/pingcap/tiflow/pull/1759)

    + バックアップ & リストア (BR)

        - ログのリストア中に `DELETE` イベントが失われる問題を修正する [#1063](https://github.com/pingcap/br/issues/1063)
        - BR が TiKV に対して多くの無駄な RPC リクエストを送信するバグを修正する [#1037](https://github.com/pingcap/br/pull/1037)
        - バックアップ操作が失敗した場合にエラーが出力されない問題を修正する [#1043](https://github.com/pingcap/br/pull/1043)

    + TiDB Lightning

        - KV データ生成時に発生する TiDB Lightning パニックの問題を修正する [#1127](https://github.com/pingcap/br/pull/1127)
        - オートコミットが無効な場合に TiDB  バックエンドモードの TiDB Lightning がデータをロードできない問題を修正する [#1104](https://github.com/pingcap/br/issues/1104)
        - データインポート時に raft エントリ制限を超えた合計キー長でバッチ分割された領域が失敗するバグを修正する [#969](https://github.com/pingcap/br/issues/969)