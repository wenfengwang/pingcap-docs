---
title: TiDB 4.0.3 リリースノート
---

# TiDB 4.0.3 リリースノート

リリース日: 2020年7月24日

TiDB バージョン: 4.0.3

## 新機能

+ TiDB ダッシュボード

    - 詳細な TiDB ダッシュボードバージョン情報を表示する[#679](https://github.com/pingcap-incubator/tidb-dashboard/pull/679)
    - サポートされていないブラウザや古いブラウザに対するブラウザ互換性注意を表示する[#654](https://github.com/pingcap-incubator/tidb-dashboard/pull/654)
    - **SQL ステートメント** ページでの検索をサポートする[#658](https://github.com/pingcap-incubator/tidb-dashboard/pull/658)

+ TiFlash

    - TiFlash プロキシでのファイル暗号化の実装

+ ツール

    + バックアップ & リストア（BR）

        - zstd、lz4、またはsnappyを使用してバックアップファイルを圧縮するサポート[#404](https://github.com/pingcap/br/pull/404)

    + TiCDC

        - MQ sink-uriで`kafka-client-id`を設定するサポート[#706](https://github.com/pingcap/tiflow/pull/706)
        - `changefeed`の構成をオフラインで更新するサポート[#699](https://github.com/pingcap/tiflow/pull/699)
        - カスタマイズされた`changefeed`名の設定をサポートする [#727](https://github.com/pingcap/tiflow/pull/727)
        - TLSおよびMySQL SSL接続をサポートする[#347](https://github.com/pingcap/tiflow/pull/347)
        - Avroフォーマットでの変更の出力をサポートする[#753](https://github.com/pingcap/tiflow/pull/753)
        - Apache Pulsar sinkをサポートする[#751](https://github.com/pingcap/tiflow/pull/751)

    + Dumpling

        - 専用のCSV区切り記号と区切り記号をサポートする[#116](https://github.com/pingcap/dumpling/pull/116)
        - 出力ファイル名の形式を指定するサポート[#122](https://github.com/pingcap/dumpling/pull/122)

## 改善

+ TiDB

    - SQLクエリのロギング時にデータを規制するための`tidb_log_desensitization`グローバル変数を追加する[#18581](https://github.com/pingcap/tidb/pull/18581)
    - `tidb_allow_batch_cop`をデフォルトで有効にする[#18552](https://github.com/pingcap/tidb/pull/18552)
    - クエリのキャンセルを高速化する[#18505](https://github.com/pingcap/tidb/pull/18505)
    - `tidb_decode_plan`の結果にヘッダーを追加する[#18501](https://github.com/pingcap/tidb/pull/18501)
    - 構成ファイルの以前のバージョンとの互換性を確認する構成チェッカーを追加する[#18046](https://github.com/pingcap/tidb/pull/18046)
    - デフォルトで実行情報を収集するようにする[#18518](https://github.com/pingcap/tidb/pull/18518)
    - `tiflash_tables`および`tiflash_segments`システムテーブルを追加する[#18536](https://github.com/pingcap/tidb/pull/18536)
    - `AUTO RANDOM`を実験的な機能から外し、一般的な利用可能に発表する。改善点および互換性の変更は以下の通り：
        - 設定ファイルで`experimental.allow-auto-random`を非推奨にする。この項目の構成に関係なく、常に列上で`AUTO RANDOM`機能を定義できる。[#18613](https://github.com/pingcap/tidb/pull/18613) [#18623](https://github.com/pingcap/tidb/pull/18623)
        - 明示的な`AUTO RANDOM`列への書き込みを制御するための`tidb_allow_auto_random_explicit_insert`セッション変数を追加する。デフォルト値は`false`である。これは、列への明示的な書き込みによって予期せぬ`AUTO_RANDOM_BASE`の更新が発生するのを避けるためである。[#18508](https://github.com/pingcap/tidb/pull/18508)
        - `AUTO_RANDOM`は`BIGINT`および`UNSIGNED BIGINT`列でのみ定義できるようにし、最大のシャードビット数を`15`に制限する。これにより格納可能なスペースの過度な消費を防ぐ。[#18538](https://github.com/pingcap/tidb/pull/18538)
        - `BIGINT`列で`AUTO_RANDOM`属性を定義し、プライマリキーに負の値を挿入する場合、`AUTO_RANDOM_BASE`の更新がトリガーされないようにする。[#17987](https://github.com/pingcap/tidb/pull/17987)
        - `UNSIGNED BIGINT`列で`AUTO_RANDOM`属性を定義し、整数の最上位ビットをID割り当てに使用することで、割り当て可能なスペースを増やす。[#18404](https://github.com/pingcap/tidb/pull/18404)
        - `SHOW CREATE TABLE`の結果で`AUTO_RANDOM`属性を更新することをサポートする[#18316](https://github.com/pingcap/tidb/pull/18316)

+ TiKV

    - バックアップスレッドプールのサイズを制御するための新しい`backup.num-threads`構成を導入する[#8199](https://github.com/tikv/tikv/pull/8199)
    - スナップショットを受信する際にストアハートビートを送信しないようにする[#8136](https://github.com/tikv/tikv/pull/8136)
    - 共有ブロックキャッシュの容量を動的に変更することをサポートする[#8232](https://github.com/tikv/tikv/pull/8232)

+ PD

    - JSON形式のログをサポートする[#2565](https://github.com/pingcap/pd/pull/2565)

+ TiDB ダッシュボード

    - 寒冷な論理範囲のためのKey Visualizerのバケットマージを改善する[#674](https://github.com/pingcap-incubator/tidb-dashboard/pull/674)
    - 一貫性のために`disable-telemetry`の構成項目を`enable-telemetry`に名前変更する[#684](https://github.com/pingcap-incubator/tidb-dashboard/pull/684)
    - ページを切り替える際に進行状況バーを表示する[#661](https://github.com/pingcap-incubator/tidb-dashboard/pull/661)
    - スペース区切りがある場合のログ検索と同じ動作をするように、遅いログ検索を確実にする[#682](https://github.com/pingcap-incubator/tidb-dashboard/pull/682)

+ TiFlash

    - Grafanaの**DDLジョブ**パネルの単位を`分ごとの操作`に変更する
    - **TiFlash-Proxy**についての詳細なメトリクスを表示するための新しいダッシュボードを追加する
    - TiFlashプロキシでIOPSを削減する

+ ツール

    + TiCDC

        - メトリクスでのテーブルIDをテーブル名で置き換える[#695](https://github.com/pingcap/tiflow/pull/695)

    + バックアップ & リストア（BR）

        - JSONログの出力をサポートする[#336](https://github.com/pingcap/br/issues/336)
        - 実行時にpprofを有効にするサポートを追加する[#372](https://github.com/pingcap/br/pull/372)
        - リストア時にDDLを並行して送信することでDDLの実行を高速化する[#377](https://github.com/pingcap/br/pull/377)

    + TiDB Lightning

        - より新しく理解しやすいフィルタ形式である`black-white-list`を非推奨にする[#332](https://github.com/pingcap/tidb-lightning/pull/332)

## バグ修正

+ TiDB

    - 実行中にエラーが発生した場合、`IndexHashJoin`の空のセットではなくエラーを返すようにする[#18586](https://github.com/pingcap/tidb/pull/18586)
    - gRPC transportReaderが壊れた場合の再発するパニックを修正する[#18562](https://github.com/pingcap/tidb/pull/18562)
    - オフラインストアでのロックスキャンが不足しているためデータの不完全性が生じる可能性があるGreen GCの問題を修正する[#18550](https://github.com/pingcap/tidb/pull/18550)
    - TiFlashエンジンを使用して読み取り専用でないステートメントを処理しないようにする[#18534](https://github.com/pingcap/tidb/pull/18534)
    - クエリ接続がパニックを起こした際に実際のエラーメッセージを返すように修正する[#18500](https://github.com/pingcap/tidb/pull/18500)
    - `ADMIN REPAIR TABLE`が実行されてもテーブルのメタデータがTiDBノードで再読み込まれない問題を修正する[#18323](https://github.com/pingcap/tidb/pull/18323)
    - 別のトランザクションで書き込まれた後に削除されたプライマリキーのロックが別のトランザクションによって解決されることによって発生したデータの一貫性の問題を修正する[#18291](https://github.com/pingcap/tidb/pull/18291)
    - スピリングディスクの動作を改善する[#18288](https://github.com/pingcap/tidb/pull/18288)
    - `REPLACE INTO`ステートメントが生成された列を含むテーブルで実行された際に報告されるエラーを修正する[#17907](https://github.com/pingcap/tidb/pull/17907)
    - `IndexHashJoin`および`IndexMergeJoin`ワーカーがパニックした場合にOOMエラーを返すようにする[#18527](https://github.com/pingcap/tidb/pull/18527)
    - 特定のケースで `Index Join` による実行が整合しない結果を返す可能性のあるバグを修正します。この問題は、`Index Join` で使用されるインデックスに整数主キーが含まれている場合に発生します [#18565](https://github.com/pingcap/tidb/pull/18565)
    - クラスターで新しい照合順序が有効になっている場合に、トランザクションで新しい照合順序を持つ列が更新されても、ユニークインデックスを通じてそのデータを読み取ることができない問題を修正します [#18703](https://github.com/pingcap/tidb/pull/18703)

+ TiKV

    - マージ中に読み取りがステールデータを取得する可能性のある問題を修正します [#8113](https://github.com/tikv/tikv/pull/8113)
    - 集約が TiKV にプッシュされた場合、`min`/`max` 関数で照合順序が機能しない問題を修正します [#8108](https://github.com/tikv/tikv/pull/8108)

+ PD

    - サーバーがクラッシュした場合に、TSO ストリームの作成がしばらくブロックされる可能性のある問題を修正します [#2648](https://github.com/pingcap/pd/pull/2648)
    - `getSchedulers` がデータ競合を引き起こす可能性のある問題を修正します [#2638](https://github.com/pingcap/pd/pull/2638)
    - スケジューラーの削除がデッドロックを引き起こす可能性のある問題を修正します [#2637](https://github.com/pingcap/pd/pull/2637)
    - `balance-leader-scheduler` が有効になっている場合に、配置ルールが考慮されないバグを修正します [#2636](https://github.com/pingcap/pd/pull/2636)
    - 時々、`safepoint` サービスが適切に設定されない可能性がある問題を修正し、それによって BR と dumpling が失敗する可能性があります [#2635](https://github.com/pingcap/pd/pull/2635)
    - `hot region scheduler` でターゲットストアが誤って選択される問題を修正します [#2627](https://github.com/pingcap/pd/pull/2627)
    - PD リーダーが切り替わると TSO リクエストが長時間かかる可能性がある問題を修正します [#2622](https://github.com/pingcap/pd/pull/2622)
    - リーダーの変更後にステールスケジューラーの問題を修正します [#2608](https://github.com/pingcap/pd/pull/2608)
    - 配置ルールが有効な場合に、時々リージョンのレプリカを最適な場所に調整できない問題を修正します [#2605](https://github.com/pingcap/pd/pull/2605)
    - ストアの展開パスが展開ディレクトリの変更に応じて更新されない問題を修正します [#2600](https://github.com/pingcap/pd/pull/2600)
    - `store limit` がゼロに変更されないようにします [#2588](https://github.com/pingcap/pd/pull/2588)

+ TiDB Dashboard

    - TiDB のスケーリングアウト時に TiDB 接続エラーを修正します [#689](https://github.com/pingcap-incubator/tidb-dashboard/pull/689)
    - ログ検索ページに TiFlash インスタンスが表示されない問題を修正します [#680](https://github.com/pingcap-incubator/tidb-dashboard/pull/680)
    - 概要ページを更新してもメトリック選択がリセットされる問題を修正します [#663](https://github.com/pingcap-incubator/tidb-dashboard/pull/663)
    - 一部の TLS シナリオでの接続問題を修正します [#660](https://github.com/pingcap-incubator/tidb-dashboard/pull/660)
    - 一部のケースで言語ドロップダウンボックスが正しく表示されない問題を修正します [#677](https://github.com/pingcap-incubator/tidb-dashboard/pull/677)

+ TiFlash

    - プライマリキー列の名前変更後に TiFlash がクラッシュする問題を修正します
    - 同時に `Learner Read` と `Remove Region` を実行した場合にデッドロックが発生する問題を修正します

+ Tools

    + TiCDC

        - 一部のケースで TiCDC がメモリをリークする問題を修正します [#704](https://github.com/pingcap/tiflow/pull/704)
        - 引用符で囲まれていないテーブル名が SQL 構文エラーを引き起こす問題を修正します [#676](https://github.com/pingcap/tiflow/pull/676)
        - `p.stop` が呼び出された後にプロセッサが完全に終了しない問題を修正します [#693](https://github.com/pingcap/tiflow/pull/693)

    + バックアップ＆リストア (BR)

        - バックアップ時間が負になる可能性がある問題を修正します [#405](https://github.com/pingcap/br/pull/405)

    + Dumpling

        - `--r` が指定された場合に `NULL` 値が省略される問題を修正します [#119](https://github.com/pingcap/dumpling/pull/119)
        - ダンプするテーブルに対してテーブルのフラッシュが機能しない場合があるバグを修正します [#117](https://github.com/pingcap/dumpling/pull/117)

    + TiDB Lightning

        - `--log-file` が効果を発揮しない問題を修正します [#345](https://github.com/pingcap/tidb-lightning/pull/345)

    + TiDB Binlog

        - TiDB Binlog が下流にデータをレプリケートする際に TLS が有効になっている場合に、チェックポイントを更新するために使用されるデータベースドライバーで TLS が有効になっていないために Drainer を起動できない問題を修正します [#988](https://github.com/pingcap/tidb-binlog/pull/988)