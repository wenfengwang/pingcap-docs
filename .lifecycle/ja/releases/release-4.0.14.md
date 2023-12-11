# TiDB 4.0.14 リリースノート

リリース日: 2021年7月27日

TiDB バージョン: 4.0.14

## 互換性の変更

+ TiDB

    - `tidb_multi_statement_mode` のデフォルト値を v4.0 で `WARN` から `OFF` へ変更しました。お客様のクライアントライブラリのマルチステートメント機能を使用することをお勧めいたします。詳細については、[ `tidb_multi_statement_mode` のドキュメント](/system-variables.md#tidb_multi_statement_mode-new-in-v4011)をご覧ください。[#25749](https://github.com/pingcap/tidb/pull/25749)
    - 2つのセキュリティ脆弱性を解決するために、Grafana ダッシュボードを v6.1.16 から v7.5.7 にアップグレードしました。詳細については[Grafanaブログ投稿](https://grafana.com/blog/2020/06/03/grafana-6.7.4-and-7.0.2-released-with-important-security-fix/)をご覧ください。
    - `tidb_stmt_summary_max_stmt_count` のデフォルト値を `200` から `3000` に変更しました[#25872](https://github.com/pingcap/tidb/pull/25872)

+ TiKV

    - リージョンマージプロセスを高速化するために、`merge-check-tick-interval` のデフォルト値を `10` から `2` に変更しました[#9676](https://github.com/tikv/tikv/pull/9676)

## 機能強化

+ TiKV

    - PD の心拍数が遅い問題を特定するのに役立つ、保留中の PD ハートビートの数を監視するメトリクス `pending` を追加しました[#10008](https://github.com/tikv/tikv/pull/10008)
    - BR が S3 互換ストレージをサポートするために、バーチャルホストアドレッシングモードの使用をサポートしました[#10242](https://github.com/tikv/tikv/pull/10242)

+ TiDB ダッシュボード

    - OIDC SSO をサポートしました。OIDC 互換の SSO サービス（Okta および Auth0 など）を設定することで、ユーザーはSQLパスワードを入力せずにTiDB ダッシュボードにログインできます。[#960](https://github.com/pingcap/tidb-dashboard/pull/960)
    - 一部の一般的な TiDB および PD 内部 API をコールするための代替手段である**デバッグAPI** UI を追加しました[#927](https://github.com/pingcap/tidb-dashboard/pull/927)

## 改善点

+ TiDB

    - `UPDATE` 読み取りの際に、インデックスキーの `point get` または `batch point get` に対して `LOCK` レコードを `PUT` レコードに変更しました[#26223](https://github.com/pingcap/tidb/pull/26223)
    - MySQL システム変数 `init_connect` およびその関連機能をサポートしました[#26031](https://github.com/pingcap/tidb/pull/26031)
    - クエリ結果を安定させるために安定結果モードをサポートしました[#26003](https://github.com/pingcap/tidb/pull/26003)
    - 組み込み関数 `json_unquote()` を TiKV にプッシュダウンするサポートを追加しました[#25721](https://github.com/pingcap/tidb/pull/25721)
    - SQLプラン管理（SPM）を文字セットに影響されないようにしました[#23295](https://github.com/pingcap/tidb/pull/23295)

+ TiKV

    - クライアントが正しくシャットダウン状態をチェックできるように、ステータスサーバーを先にシャットダウンするようにしました[#10504](https://github.com/tikv/tikv/pull/10504)
    - ステールペアに常に応答し、これらのペアがより速くクリアされるようにしました[#10400](https://github.com/tikv/tikv/pull/10400)
    - TiCDC シンクのメモリ消費を制限しました[#10147](https://github.com/tikv/tikv/pull/10147)
    - リージョンが大きすぎる場合、分割プロセスを高速化するために均等分割を使用するようにしました[#10275](https://github.com/tikv/tikv/pull/10275)

+ PD

    - 同時に実行される複数のスケジューラー間の競合を減らしました[#3858](https://github.com/pingcap/pd/pull/3858) [#3854](https://github.com/tikv/pd/pull/3854)

+ TiDB ダッシュボード

    - TiDB ダッシュボードを v2021.07.17.1 に更新しました[#3882](https://github.com/pingcap/pd/pull/3882)
    - 現在のセッションを読み取り専用セッションとして共有し、それ以降の変更を回避するためのサポートを追加しました[#960](https://github.com/pingcap/tidb-dashboard/pull/960)

+ ツール

    + バックアップ＆リストア（BR）

        - 小さなバックアップファイルをマージすることでリストアを高速化しました[#655](https://github.com/pingcap/br/pull/655)

    + Dumpling

        - 上流がTiDB v3.xクラスターの場合、`_tidb_rowid`を使用して常にテーブルを分割するようにし、TiDBのメモリ使用量を削減することをサポートしました[#306](https://github.com/pingcap/dumpling/pull/306)

    + TiCDC

        - PD エンドポイントが証明書を取り込んでいない場合に返されるエラーメッセージを改善しました[#1973](https://github.com/pingcap/tiflow/issues/1973)
        - ソーター I/O エラーをユーザーフレンドリーにしました[#1976](https://github.com/pingcap/tiflow/pull/1976)
        - KV クライアントにリージョンインクリメンタルスキャンの並行性制限を追加し、TiKVの圧力を軽減しました[#1926](https://github.com/pingcap/tiflow/pull/1926)
        - テーブルメモリ消費に関するメトリクスを追加しました[#1884](https://github.com/pingcap/tiflow/pull/1884)
        - `capture-session-ttl` を TiCDC サーバー設定に追加しました[#2169](https://github.com/pingcap/tiflow/pull/2169)

## バグ修正

+ TiDB

    - サブクエリを`WHERE`句が`false`に評価されるときに`SELECT`結果がMySQLと互換性がない問題を修正しました[#24865](https://github.com/pingcap/tidb/issues/24865)
    - 引数が`ENUM`または`SET`型の場合に発生する`ifnull`関数の計算エラーを修正しました[#24944](https://github.com/pingcap/tidb/issues/24944)
    - 一部のケースでの誤った集約剪定を修正しました[#25202](https://github.com/pingcap/tidb/issues/25202)
    - 列が`SET`型である場合に発生するマージ結合演算の不正確な結果を修正しました[#25669](https://github.com/pingcap/tidb/issues/25669)
    - TiDB がカテシャン結合の誤った結果を返す問題を修正しました[#25591](https://github.com/pingcap/tidb/issues/25591)
    - `SELECT ... FOR UPDATE` がパーティションテーブルを使用し、その結果ジョイン操作でパニックが発生する問題を修正しました[#20028](https://github.com/pingcap/tidb/issues/20028)
    - キャッシュされた`prepared`プランが`point get`に対して誤って使用される問題を修正しました[#24741](https://github.com/pingcap/tidb/issues/24741)
    - `LOAD DATA` ステートメントが非UTF-8データを異常にインポートする問題を修正しました[#25979](https://github.com/pingcap/tidb/issues/25979)
    - HTTP APIを介して統計にアクセスした際に発生する潜在的なメモリリーク問題を修正しました[#24650](https://github.com/pingcap/tidb/pull/24650)
    - `ALTER USER` ステートメントを実行する際に発生するセキュリティ問題を修正しました[#25225](https://github.com/pingcap/tidb/issues/25225)
    - `TIKV_REGION_PEERS`テーブルが`DOWN`ステータスを適切に処理できないバグを修正しました[#24879](https://github.com/pingcap/tidb/issues/24879)
    - `DateTime`を解析する際に無効な文字列が切り捨てられない問題を修正しました[#22231](https://github.com/pingcap/tidb/issues/22231)
    - カラムのタイプが`YEAR`である場合に、`select into outfile`ステートメントが結果を持たない可能性がある問題を修正しました[#22159](https://github.com/pingcap/tidb/issues/22159)
    - `NULL`が`UNION`サブクエリにある場合にクエリ結果が誤る可能性がある問題を修正しました[#26532](https://github.com/pingcap/tidb/issues/26532)
    - 実行中にプロジェクション演算子がいくつかの状況でパニックを引き起こす可能性がある問題を修正しました[#26534](https://github.com/pingcap/tidb/pull/26534)

+ TiKV

    - 特定のプラットフォームで期間計算がパニックする可能性がある問題を修正しました[#related-issue](https://github.com/rust-lang/rust/issues/86470#issuecomment-877557654)
    - `DOUBLE`を`DOUBLE`にキャストする誤った関数を修正しました[#25200](https://github.com/pingcap/tidb/issues/25200)
    - 非同期ロガーを使用する場合にパニックログが失われる可能性がある問題を修正しました[#8998](https://github.com/tikv/tikv/issues/8998)
- 暗号化が有効になっている場合に、スナップショットを2回構築するとパニックが発生する問題を修正します [#9786](https://github.com/tikv/tikv/issues/9786) [#10407](https://github.com/tikv/tikv/issues/10407)
    - コプロセッサの`json_unquote()`関数の引数の型が間違っている問題を修正します [#10176](https://github.com/tikv/tikv/issues/10176)
    - シャットダウン中の疑わしい警告とRaftstoreからの非決定的な応答の問題を修正します [#10353](https://github.com/tikv/tikv/issues/10353) [#10307](https://github.com/tikv/tikv/issues/10307)
    - バックアップスレッドがリークする問題を修正します [#10287](https://github.com/tikv/tikv/issues/10287)
    - リージョンの分割が遅れてリージョンのマージが進行中の場合にパニックが発生し、メタデータが破損する可能性がある問題を修正します [#8456](https://github.com/tikv/tikv/issues/8456) [#8783](https://github.com/tikv/tikv/issues/8783)
    - リージョンの心拍がTiKVの一部の状況で大規模なリージョンの分割を妨げる問題を修正します [#10111](https://github.com/tikv/tikv/issues/10111)
    - TiKVとTiDBのCM Sketchのフォーマットの不一致に起因する不正確な統計情報の問題を修正します [#25638](https://github.com/pingcap/tidb/issues/25638)
    - `apply wait duration`メトリックの不正確な統計情報の問題を修正します [#9893](https://github.com/tikv/tikv/issues/9893)
    - Titanで`delete_files_in_range`を使用した後に"Missing Blob"エラーが発生する問題を修正します [#10232](https://github.com/tikv/tikv/pull/10232)

+ PD

    - 削除操作を実行した後にスケジューラが再度表示される可能性があるバグを修正します [#2572](https://github.com/tikv/pd/issues/2572)
    - 一時的な構成が読み込まれる前にスケジューラが開始されてデータ競合が発生する可能性がある問題を修正します [#3771](https://github.com/tikv/pd/issues/3771)
    - リージョンの拡散操作中にPDのパニックが発生する可能性がある問題を修正します [#3761](https://github.com/pingcap/pd/pull/3761)
    - 一部のオペレータの優先度が正しく設定されていない問題を修正します [#3703](https://github.com/pingcap/pd/pull/3703)
    - 存在しないストアから`evict-leader`スケジューラを削除する際にPDのパニックが発生する可能性がある問題を修正します [#3660](https://github.com/tikv/pd/issues/3660)
    - 多数のストアがある場合にPDリーダーの再選出が遅い問題を修正します [#3697](https://github.com/tikv/pd/issues/3697)

+ TiDB Dashboard

    - **Profiling** UIがすべてのTiDBインスタンスをプロファイリングできない問題を修正します [#944](https://github.com/pingcap/tidb-dashboard/pull/944)
    - **Statements** UIが "Plan Count"を表示しない問題を修正します [#939](https://github.com/pingcap/tidb-dashboard/pull/939)
    - クラスターのアップグレード後に**Slow Query** UIが "unknown field"エラーを表示する可能性がある問題を修正します [#902](https://github.com/pingcap/tidb-dashboard/issues/902)

+ TiFlash

    - DAGリクエストのコンパイル時に発生する潜在的なパニックの問題を修正します
    - 読み込み負荷が重い場合に発生するパニックの問題を修正します
    - カラムストレージの分割の失敗によりTiFlashが繰り返し再起動する問題を修正します
    - TiFlashが共有デルタインデックスを同時にクローニングできない可能性があるバグを修正します
    - 不完全なデータの場合にTiFlashが再起動できないバグを修正します
    - 旧式のdmファイルが自動的に削除されない問題を修正します
    - 特定の引数で`SUBSTRING`関数を実行した際に発生するパニックの問題を修正します
    - `INTEGER`型を`TIME`型にキャストした際の不正確な結果の問題を修正します

+ Tools

    + Backup & Restore (BR)

        - `mysql`スキーマからのデータ復元に失敗する可能性がある問題を修正します [#1142](https://github.com/pingcap/br/pull/1142)

    + TiDB Lightning

        - Parquetファイル内の`DECIMAL`型のデータを解析できない問題を修正します [#1276](https://github.com/pingcap/br/pull/1276)
        - 大きなCSVファイルを読み込む際にTiDB LightningがEOFエラーを報告する問題を修正します [#1133](https://github.com/pingcap/br/issues/1133)
        - `FLOAT`または`DOUBLE`型の`auto_increment`列を持つテーブルをインポートする際に過剰に大きな基準値が生成されるバグを修正します [#1185](https://github.com/pingcap/br/pull/1185)
        - 4 GBを超えるKVデータを生成した際にTiDB Lightningが発生するパニックの問題を修正します [#1128](https://github.com/pingcap/br/pull/1128)

    + Dumpling

        - Dumplingを使用してデータをS3ストレージにエクスポートする際、`s3:ListBucket`権限がバケット全体ではなくデータソースの接頭辞のみで必要となります。[#898](https://github.com/pingcap/br/issues/898)

    + TiCDC

        - 新しいテーブルパーティションを追加した後に余分なパーティションが配信される問題を修正します [#2205](https://github.com/pingcap/tiflow/pull/2205)
        - TiCDCが`/proc/meminfo`を読み取れない際に発生するパニックの問題を修正します [#2023](https://github.com/pingcap/tiflow/pull/2023)
        - TiCDCのランタイムメモリ使用量を削減します [#2011](https://github.com/pingcap/tiflow/pull/2011) [#1957](https://github.com/pingcap/tiflow/pull/1957)
        - MySQL接続がエラーを起こし一時停止した後に一部のMySQL接続がリークする可能性があるバグを修正します [#1945](https://github.com/pingcap/tiflow/pull/1945)
        - 開始TSが現在のTSからGC TTLを差し引いた値よりも小さい場合にTiCDC changefeedを作成できない問題を修正します [#1839](https://github.com/pingcap/tiflow/issues/1839)
        - sort heap内のメモリ`malloc`を削減し、過剰なCPU負荷を回避するように修正します [#1853](https://github.com/pingcap/tiflow/issues/1853)
        - テーブルを移動する際に複製タスクが停止する可能性があるバグを修正します [#1827](https://github.com/pingcap/tiflow/pull/1827)