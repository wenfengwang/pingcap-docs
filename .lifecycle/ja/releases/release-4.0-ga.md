---
title: TiDB 4.0 GA リリースノート
aliases: ['/docs/dev/releases/release-4.0-ga/']
---

# TiDB 4.0 GA リリースノート

リリース日: 2020年5月28日

TiDB バージョン: 4.0.0

## 互換性の変更

* TiDB
    - 大きなトランザクションのエラーメッセージを最適化してトラブルシューティングを容易にする[#17219](https://github.com/pingcap/tidb/pull/17219)

* TiCDC
    - `Changefeed` の構造を最適化し、使いやすさを向上させるための構成ファイルを変更しました[#588](https://github.com/pingcap/tiflow/pull/588)
    - `ignore-txn-start-ts` 構成項目を追加し、トランザクションのフィルタリング中に `commit_ts` から `start_ts` への条件変更を行いました[#589](https://github.com/pingcap/tiflow/pull/589)

## 重要なバグ修正

* TiKV
    - バックアップ＆リストア（BR）を使用してバックアップを実行する際に発生する `DefaultNotFound` エラーを修正しました[#7937](https://github.com/tikv/tikv/pull/7937)
    - `ReadIndex` パッケージの順序が正しくないことによって発生するシステムパニックを修正しました[#7930](https://github.com/tikv/tikv/pull/7930)
    - TiKV を再起動した後に誤ってスナップショットファイルを削除し、システムパニックが発生する問題を修正しました[#7927](https://github.com/tikv/tikv/pull/7927)

* TiFlash
    - `Raft Admin Command` の不正確な処理ロジックによってシステムがパニック状態になる可能性があるため、発生する可能性のあるデータ損失の問題を修正しました 

## 新機能

* TiDB
    - `goroutine` のリトライコミット段階での数を制御するための `committer-concurrency` 構成項目を追加しました[#16849](https://github.com/pingcap/tidb/pull/16849)
    - `show table partition regions` 構文をサポートしました[#17294](https://github.com/pingcap/tidb/pull/17294)
    - TiDB サーバーが使用する一時ディスク領域を制限するための `tmp-storage-quota` 構成項目を追加しました[#15700](https://github.com/pingcap/tidb/pull/15700)
    - テーブルを作成および変更する際に、パーティション化されたテーブルが一意のプレフィックスインデックスを使用しているかどうかを確認するサポートを追加しました[#17213](https://github.com/pingcap/tidb/pull/17213)
    - `insert/replace into tbl_name partition`(`partition_name_list`) ステートメントをサポートしました[#17313](https://github.com/pingcap/tidb/pull/17313)
    - `Distinct` 関数を使用している場合に `collations` の値を確認するサポートを追加しました[#17240](https://github.com/pingcap/tidb/pull/17240)
    - Hash パーティションプルーニング中の `is null` フィルタ条件をサポートしました[#17310](https://github.com/pingcap/tidb/pull/17310)
    - パーティション化されたテーブルで `admin check index`、`admin cleanup index`、および `admin recover index` をサポートしました[#17392](https://github.com/pingcap/tidb/pull/17392) [#17405](https://github.com/pingcap/tidb/pull/17405) [#17317](https://github.com/pingcap/tidb/pull/17317)
    - `in` 式の範囲パーティションプルーニングをサポートしました[#17320](https://github.com/pingcap/tidb/pull/17320)

* TiFlash
    - `Learner` がデータを読み取る際に、`Lock CF` の `min commit ts` 値を使用して `TSO` に対応するデータをフィルタリングする機能をサポート
    - `TIMESTAMP` 型の値が `1970-01-01 00:00:00` よりも小さい場合、誤った計算結果を回避するために、システムが明示的にエラーを報告する機能を追加
    - ログを検索する際に正規表現でフラグを使用する機能をサポート

* TiKV
    - `ascii_bin` および `latin1_bin` エンコーディングの照合ルールをサポートしました[#7919](https://github.com/tikv/tikv/pull/7919)

* PD
    - 組み込み TiDB ダッシュボードの逆プロキシリソースプレフィックスを指定する機能をサポートしました[#2457](https://github.com/pingcap/pd/pull/2457)
    - PD クライアントリージョンのインタフェースで `pending peer` および `down peer` 情報を返す機能をサポートしました[#2443](https://github.com/pingcap/pd/pull/2443)
    - `Hotspot move leader` の移動方向、`Hotspot move peer` の移動方向、および `Hot cache read entry number` などの監視項目を追加しました[#2448](https://github.com/pingcap/pd/pull/2448)

* Tools
    + バックアップ＆リストア（BR）
        - `Sequence` および `View` のバックアップとリストアをサポートしました[#242](https://github.com/pingcap/br/pull/242)
    + TiCDC
        - `Changefeed` を作成する際に `Sink URI` の妥当性をチェックする機能をサポートしました[#561](https://github.com/pingcap/tiflow/pull/561)
        - システムの起動時に PD および TiKV のバージョンがシステム要件を満たしているかどうかをチェックする機能をサポートしました[#570](https://github.com/pingcap/tiflow/pull/570)
        - 同時スケジューリングタスク生成サイクルで複数のテーブルをスケジューリングする機能をサポートしました[#572](https://github.com/pingcap/tiflow/pull/572)
        - HTTP API でノードの役割に関する情報を追加しました[#591](https://github.com/pingcap/tiflow/pull/591)

## バグ修正

* TiDB
    - TiDB にバッチコマンドを TiFlash に送信しないようにしてメッセージの送受信時に予期しないタイムアウトの問題を修正しました[#17307](https://github.com/pingcap/tidb/pull/17307)
    - パーティションプルーニング時に符号付きと符号なし整数を誤って区別してしまう問題を修正し、パフォーマンスを向上させました[#17230](https://github.com/pingcap/tidb/pull/17230)
    - `mysql.user` テーブルとの非互換性による v3.1.1 から v4.0 へのアップグレード障害を修正しました[#17300](https://github.com/pingcap/tidb/pull/17300)
    - `update` ステートメントでパーティションの選択が誤って行われてしまう問題を修正しました[#17305](https://github.com/pingcap/tidb/pull/17305)
    - TiKV から未知のエラーメッセージを受信した際にシステムがパニック状態になってしまう問題を修正しました[#17380](https://github.com/pingcap/tidb/pull/17380)
    - `key` でパーティション化されたテーブルを作成する際の不正な処理ロジックによってシステムがパニック状態になる問題を修正しました[#17242](https://github.com/pingcap/tidb/pull/17242)
    - オプティマイザの処理ロジックが誤っていることによって誤った `Index Merge Join` プランが選択される問題を修正しました[#17365](https://github.com/pingcap/tidb/pull/17365)
    - Grafana での `SELECT` ステートメントの `duration` 監視メトリックが不正確である問題を修正しました[#16561](https://github.com/pingcap/tidb/pull/16561)
    - システムエラーが発生した際に GC ワーカーがブロックされる問題を修正しました[#16915](https://github.com/pingcap/tidb/pull/16915)
    - ブール型の列における `UNIQUE` 制約が比較時に誤った結果をもたらしてしまう問題を修正しました[#17306](https://github.com/pingcap/tidb/pull/17306)
    - `tidb_opt_agg_push_down` が有効であり、集約関数がパーティション化されたテーブルにプッシュダウンされる際にシステムがパニック状態になる問題を修正しました[#17328](https://github.com/pingcap/tidb/pull/17328)
    - 特定のケースで失敗した TiKV ノードへのアクセスが行われてしまう問題を修正しました[#17342](https://github.com/pingcap/tidb/pull/17342)
    - `tidb.toml` の `isolation-read` 構成項目が効果を持たない問題を修正しました[#17322](https://github.com/pingcap/tidb/pull/17322)
    - `hint` を使用してストリーム集約を強制する際に、誤った処理ロジックによって出力結果の順序が誤ってしまう問題を修正しました[#17347](https://github.com/pingcap/tidb/pull/17347)
    - `SQL_MODE` が異なる場合に `insert` が `DIV` を処理する挙動を修正しました[#17314](https://github.com/pingcap/tidb/pull/17314)

* TiFlash
    - 検索ログ機能の正規表現の一致動作が他のコンポーネントと一貫していない問題を修正しました
    - `Raft Compact Log Command` の遅延処理の最適化をデフォルトで無効にすることによって、ノードが大量のデータを書き込む際の過剰な再起動時間の問題を修正しました
    - 特定のシナリオで TiDB が `DROP DATABASE` ステートメントを誤って処理してしまい、システムが起動に失敗する問題を修正しました
    - `Server_info` で CPU 情報を収集する方法が他のコンポーネントと異なる問題を修正しました
    - `batch coprocessor` が有効になっている場合に `Query` ステートメントを実行した際に `Too Many Pings` エラーが報告されてしまう問題を修正しました
    - Dashboard がTiFlashが関連情報を報告しないため、正しい `deploy path` 情報を表示できない問題を修正しました

* TiKV
- BR 
    - BRを使用してバックアップを取得する際に発生する `DefaultNotFound` エラーを修正します [#7937](https://github.com/tikv/tikv/pull/7937)
    - 順序外の `ReadIndex` パケットによって発生するシステムパニックを修正します [#7930](https://github.com/tikv/tikv/pull/7930)
    - 読み込み要求のコールバック関数が呼び出されないために予期しないエラーが返される問題を修正します [#7921](https://github.com/tikv/tikv/pull/7921)
    - TiKVが再起動された際にスナップショットファイルが誤って削除されることによって発生するシステムパニックを修正します [#7927](https://github.com/tikv/tikv/pull/7927)
    - ストレージ暗号化における処理ロジックの誤りによって `master key` を回転させることができない問題を修正します [#7898](https://github.com/tikv/tikv/pull/7898)
    - ストレージ暗号化が有効な場合に、スナップショットの受信 `lock cf` ファイルが暗号化されないという問題を修正します [#7922](https://github.com/tikv/tikv/pull/7922)

* PD

    - pd-ctlを使用して `evict-leader-scheduler` または `grant-leader-scheduler` を削除する際に発生する 404 エラーを修正します [#2446](https://github.com/pingcap/pd/pull/2446)
    - TiFlash レプリカが存在する場合に `presplit` 機能が正常に動作しない可能性がある問題を修正します [#2447](https://github.com/pingcap/pd/pull/2447)

* Tools

    + Backup & Restore (BR)
        - BRがクラウドストレージからデータを復元する際にネットワークの問題によってデータの復元に失敗する問題を修正します [#298](https://github.com/pingcap/br/pull/298)
    + TiCDC
        - データ競合によって発生するシステムパニックを修正します [#565](https://github.com/pingcap/tiflow/pull/565) [#566](https://github.com/pingcap/tiflow/pull/566)
        - 不正な処理ロジックによって発生するリソースリークまたはシステムのブロックを修正します [#574](https://github.com/pingcap/tiflow/pull/574) [#586](https://github.com/pingcap/tiflow/pull/586)
        - CLIがPDに接続できないためにコマンドラインがスタックする問題を修正します [#579](https://github.com/pingcap/tiflow/pull/579)