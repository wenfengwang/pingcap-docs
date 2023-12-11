---
title: TiDB 4.0.11リリースノート
---

# TiDB 4.0.11リリースノート

リリース日: 2021年2月26日

TiDBバージョン: 4.0.11

## 新機能

+ TiDB

    - `utf8_unicode_ci`および`utf8mb4_unicode_ci`照合をサポート [#22558](https://github.com/pingcap/tidb/pull/22558)

+ TiKV

    - `utf8mb4_unicode_ci`照合をサポート [#9577](https://github.com/tikv/tikv/pull/9577)
    - `cast_year_as_time`照合をサポート [#9299](https://github.com/tikv/tikv/pull/9299)

+ TiFlash

    - Coprocessorリクエストを実行するためにCoprocessorスレッドプールを追加し、いくつかのケースでのメモリ不足（OOM）を避ける。デフォルト値は`NumOfPhysicalCores * 2`の`cop_pool_size`および`batch_cop_pool_size`構成項目を追加

## 改善

+ TiDB

    - アウタージョインから簡素化されたインナージョインを並べ替えるように [#22402](https://github.com/pingcap/tidb/pull/22402)
    - Grafanaダッシュボードで複数のクラスタをサポート [#22534](https://github.com/pingcap/tidb/pull/22534)
    - 複数の文の問題のためのワークアラウンドを追加 [#22468](https://github.com/pingcap/tidb/pull/22468)
    - 遅いクエリのメトリクスを`internal`および`general`に分けるように追加 [#22405](https://github.com/pingcap/tidb/pull/22405)
    - `utf8_unicode_ci`および`utf8mb4_unicode_ci`照合のためのインターフェースを追加 [#22099](https://github.com/pingcap/tidb/pull/22099)

+ TiKV

    - DBaaSのためのサーバ情報のメトリクスを追加 [#9591](https://github.com/tikv/tikv/pull/9591)
    - Grafanaダッシュボードで複数のクラスタをサポート [#9572](https://github.com/tikv/tikv/pull/9572)
    - RocksDBメトリクスをTiDBに報告するように追加 [#9316](https://github.com/tikv/tikv/pull/9316)
    - Coprocessorタスクの一時停止時間を記録するように追加 [#9277](https://github.com/tikv/tikv/pull/9277)
    - Load Base Splitのキーカウントおよびキーサイズの閾値を追加 [#9354](https://github.com/tikv/tikv/pull/9354)
    - データのインポート前にファイルの存在を確認するように修正 [#9544](https://github.com/tikv/tikv/pull/9544)
    - Fast Tuneパネルを改善 [#9180](https://github.com/tikv/tikv/pull/9180)

+ PD

    - Grafanaダッシュボードで複数のクラスタをサポート [#3398](https://github.com/pingcap/pd/pull/3398)

+ TiFlash

    - `date_format`関数のパフォーマンスを最適化
    - Ingest SSTのメモリ消費を最適化
    - バッチCoprocessorの再試行ロジックを最適化し、リージョンエラーの発生確率を低減

+ Tools

    + TiCDC

        - `capture`メタデータにバージョン情報を追加し、`changefeed`メタデータに`changefeed`のCLIバージョンを追加 [#1342](https://github.com/pingcap/tiflow/pull/1342)

    + TiDB Lightning

        - インポートのパフォーマンスを向上するためにテーブルを並列で作成するように修正 [#502](https://github.com/pingcap/tidb-lightning/pull/502)
        - エンジンの総サイズがリージョンサイズよりも小さい場合は、インポートのパフォーマンスを向上するためにリージョンを分割しないように修正 [#524](https://github.com/pingcap/tidb-lightning/pull/524)
        - インポートの進行状況バーを追加し、復元進行状況の精度を最適化 [#506](https://github.com/pingcap/tidb-lightning/pull/506)

## バグ修正

+ TiDB

    - `unicode_ci`定数の異常な伝播の問題を修正 [#22614](https://github.com/pingcap/tidb/pull/22614)
    - 誤った照合および強制変換の原因となり得る問題を修正 [#22602](https://github.com/pingcap/tidb/pull/22602)
    - 誤った照合結果の原因となり得る問題を修正 [#22599](https://github.com/pingcap/tidb/pull/22599)
    - 異なる照合のための定数の代替の問題を修正 [#22582](https://github.com/pingcap/tidb/pull/22582)
    - `like`関数が照合を使用する場合に誤った結果を返す可能性のあるバグを修正 [#22531](https://github.com/pingcap/tidb/pull/22531)
    - `least`および`greatest`関数での`duration`型の推論の誤りを修正 [#22580](https://github.com/pingcap/tidb/pull/22580)
    - `like`関数が、パターン文字列がユニコード文字列である場合に誤った結果を返す可能性のあるバグを修正 [#22529](https://github.com/pingcap/tidb/pull/22529)
    - `@@tidb_snapshot`変数が設定されている場合にスナップショットデータを取得しない点取得クエリがスナップショットデータを取得しない問題を修正 [#22527](https://github.com/pingcap/tidb/pull/22527)
    - 結合からヒントを生成する際に発生する潜在的なパニックを修正 [#22518](https://github.com/pingcap/tidb/pull/22518)
    - `tidb_rowid`列に値を挿入する際に発生する`index out of range`エラーを修正 [#22359](https://github.com/pingcap/tidb/pull/22359)
    - キャッシュされたプランが誤って使用されるバグを修正 [#22353](https://github.com/pingcap/tidb/pull/22353)
    - バイナリ/char文字列の長さが大きすぎる場合に`WEIGHT_STRING`関数でランタイムパニックが発生する問題を修正 [#22332](https://github.com/pingcap/tidb/pull/22332)
    - 生成された列の数が無効である場合に生成された列を使用することを禁止するように修正 [#22174](https://github.com/pingcap/tidb/pull/22174)
    - 実行計画を構築する前にプロセス情報を正しく設定するように修正 [#22148](https://github.com/pingcap/tidb/pull/22148)
    - `IndexLookUp`のランタイム統計の不正確な問題を修正 [#22136](https://github.com/pingcap/tidb/pull/22136)
    - クラスタがコンテナで展開されている場合にメモリ使用状況の情報のキャッシュを追加 [#22116](https://github.com/pingcap/tidb/pull/22116)
    - デコードプランのエラーの問題を修正 [#22022](https://github.com/pingcap/tidb/pull/22022)
    - 無効なウィンドウ仕様を使用している場合にエラーを報告するように修正 [#21976](https://github.com/pingcap/tidb/pull/21976)
    - `PREPARE`文が`EXECUTE`、`DEALLOCATE`、または`PREPARE`とネストされた場合にエラーを報告するように修正 [#21972](https://github.com/pingcap/tidb/pull/21972)
    - `INSERT IGNORE`文が存在しないパーティションに使用されている場合にエラーを報告するように修正 [#21971](https://github.com/pingcap/tidb/pull/21971)
    - `EXPLAIN`の結果と遅いログのエンコーディングを統一するように修正 [#21964](https://github.com/pingcap/tidb/pull/21964)
    - 集約演算子を使用する際の結合内の不明な列の問題を修正 [#21957](https://github.com/pingcap/tidb/pull/21957)
    - `ceiling`関数での間違った型推論の問題を修正 [#21936](https://github.com/pingcap/tidb/pull/21936)
    - `Double`型列がその小数を無視する問題を修正 [#21916](https://github.com/pingcap/tidb/pull/21916)
    - サブクエリで関連付けられた集約が計算される問題を修正 [#21877](https://github.com/pingcap/tidb/pull/21877)
    - キー長 >= 65536のJSONオブジェクトのエラーを報告するように修正 [#21870](https://github.com/pingcap/tidb/pull/21870)
    - `dyname`関数がMySQLと互換性がない問題を修正 [#21850](https://github.com/pingcap/tidb/pull/21850)
- 入力データが長すぎる場合に `to_base64` 関数が `NULL` を返す問題を修正 [#21813](https://github.com/pingcap/tidb/pull/21813)
    - サブクエリで複数のフィールドを比較する際の失敗を修正 [#21808](https://github.com/pingcap/tidb/pull/21808)
    - JSON の float 型を比較する際に発生する問題を修正 [#21785](https://github.com/pingcap/tidb/pull/21785)
    - JSON オブジェクトの型を比較する際に発生する問題を修正 [#21718](https://github.com/pingcap/tidb/pull/21718)
    - `cast` 関数の coercibility 値が誤って設定される問題を修正 [#21714](https://github.com/pingcap/tidb/pull/21714)
    - `IF` 関数を使用する際の予期しないパニックを修正 [#21711](https://github.com/pingcap/tidb/pull/21711)
    - JSON 検索から返される `NULL` 結果が MySQL と互換性がない問題を修正 [#21700](https://github.com/pingcap/tidb/pull/21700)
    - `ORDER BY` および `HAVING` を使用して `only_full_group_by` モードをチェックする際に発生する問題を修正 [#21697](https://github.com/pingcap/tidb/pull/21697)
    - `Day` および `Time` の単位が MySQL と互換性がない問題を修正 [#21676](https://github.com/pingcap/tidb/pull/21676)
    - `LEAD` および `LAG` のデフォルト値がフィールドタイプに適応できない問題を修正 [#21665](https://github.com/pingcap/tidb/pull/21665)
    - `LOAD DATA` ステートメントでデータをベーステーブルにのみロードできるようにするためのチェックを実行 [#21638](https://github.com/pingcap/tidb/pull/21638)
    - `addtime` および `subtime` 関数が無効な引数を処理する際に発生する問題を修正 [#21635](https://github.com/pingcap/tidb/pull/21635)
    - 近似値に対する丸めるルールを "最も近い偶数に丸める" に変更 [#21628](https://github.com/pingcap/tidb/pull/21628)
    - `WEEK()` が `@@GLOBAL.default_week_format` を明示的に読み込むまで認識しない問題を修正 [#21623](https://github.com/pingcap/tidb/pull/21623)

+ TiKV

    - TiKV が `PROST=1` でビルドに失敗する問題を修正 [#9604](https://github.com/tikv/tikv/pull/9604)
    - 不一致のメモリ診断を修正 [#9589](https://github.com/tikv/tikv/pull/9589)
    - 部分的な RawKV リストア範囲の終了キーが包括的になる問題を修正 [#9583](https://github.com/tikv/tikv/pull/9583)
    - TiCDC の増分スキャン中にロールバックされたトランザクションの古い値を読み込む際に発生する TiKV パニックの問題を修正 [#9569](https://github.com/tikv/tikv/pull/9569)
    - 異なる設定で changefeed が1つのリージョンに接続した場合の古い値の構成の不具合を修正 [#9565](https://github.com/tikv/tikv/pull/9565)
    - ネットワークインタフェースに MAC アドレスがないマシンで TiKV クラスタを実行している場合に発生するクラッシュの問題を修正（v4.0.9 で導入） [#9516](https://github.com/tikv/tikv/pull/9516)
    - 巨大なリージョンをバックアップする際の TiKV OOM の問題を修正 [#9448](https://github.com/tikv/tikv/pull/9448)
    - `region-split-check-diff` をカスタマイズできない問題を修正 [#9530](https://github.com/tikv/tikv/pull/9530)
    - システム時刻が戻ったときに TiKV がパニックする問題を修正 [#9542](https://github.com/tikv/tikv/pull/9542)

+ PD

    - メンバーの健康メトリクスが誤って表示される問題を修正 [#3368](https://github.com/pingcap/pd/pull/3368)
    - 依然としてピアを持つ墓石ストアを削除することを禁止する機能を追加 [#3352](https://github.com/pingcap/pd/pull/3352)
    - ストア制限を永続化できない問題を修正 [#3403](https://github.com/pingcap/pd/pull/3403)
    - スキャッタ範囲スケジューラの制限を修正 [#3401](https://github.com/pingcap/pd/pull/3401)

+ TiFlash

    - decimal 型の `min`/`max` 結果が間違っている不具合を修正
    - データを読み込む際に TiFlash がクラッシュする可能性がある不具合を修正
    - DDL 操作後に書き込まれた一部のデータがデータ圧縮後に失われる問題を修正
    - Coprocessor で decimal 定数を誤って処理する問題を修正
    - リーダー読み取りプロセス中に発生する潜在的なクラッシュを修正
    - TiDB と TiFlash 間の `0` または `NULL` による除算の一貫性の不一致を修正

+ Tools

    + TiCDC

        - `ErrTaskStatusNotExists` および `capture` セッションのクローズが同時に発生した際に TiCDC サービスが予期せず終了する可能性があるバグを修正 [#1240](https://github.com/pingcap/tiflow/pull/1240)
        - `changefeed` が他の `changefeed` に影響を受ける古い値スイッチの問題を修正 [#1347](https://github.com/pingcap/tiflow/pull/1347)
        - 無効な `sort-engine` パラメータで新しい `changefeed` を処理する際に TiCDC サービスが停止する可能性があるバグを修正 [#1309](https://github.com/pingcap/tiflow/pull/1309)
        - オーナーノード以外でデバッグ情報を取得する際に発生するパニックの問題を修正 [#1349](https://github.com/pingcap/tiflow/pull/1349)
        - テーブルの追加または削除時に `ticdc_processor_num_of_tables` および `ticdc_processor_table_resolved_ts` メトリクスが適切に更新されない問題を修正 [#1351](https://github.com/pingcap/tiflow/pull/1351)
        - プロセッサがクラッシュした場合に潜在的なデータ損失が発生する可能性がある問題を修正 [#1363](https://github.com/pingcap/tiflow/pull/1363)
        - テーブルの移行中にオーナーが異常終了する可能性があるバグを修正 [#1352](https://github.com/pingcap/tiflow/pull/1352)
        - サービス GC セーフポイントが失われた場合に TiCDC がタイムリーに終了しないバグを修正 [#1367](https://github.com/pingcap/tiflow/pull/1367)
        - KV クライアントがイベントフィードの作成をスキップする可能性があるバグを修正 [#1336](https://github.com/pingcap/tiflow/pull/1336)
        - トランザクションの下流への複製時にトランザクションの原子性が壊れる可能性があるバグを修正 [#1375](https://github.com/pingcap/tiflow/pull/1375)

    + Backup & Restore (BR)

        - BR がバックアップをリストアした後に大きなリージョンを生成する原因となる問題を修正 [#702](https://github.com/pingcap/br/pull/702)
        - BR がテーブルに Auto ID がない場合でもテーブルの Auto ID をリストアする問題を修正 [#720](https://github.com/pingcap/br/pull/720)

    + TiDB Lightning

        - TiDB バックエンドを使用している時に `column count mismatch` が発生する可能性があるバグを修正 [#535](https://github.com/pingcap/tidb-lightning/pull/535)
        - ソースファイルの列数とターゲットテーブルの列数が一致しない場合に TiDB バックエンドが予期せずパニックする問題を修正 [#528](https://github.com/pingcap/tidb-lightning/pull/528)
        - TiDB Lightning のデータインポート中に TiKV が予期せずパニックする問題を修正 [#554](https://github.com/pingcap/tidb-lightning/pull/554)