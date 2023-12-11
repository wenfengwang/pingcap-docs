---
title: TiDB 4.0 RC.2 リリースノート
aliases: ['/docs/dev/releases/release-4.0.0-rc.2/']
---

# TiDB 4.0 RC.2 リリースノート

リリース日: 2020年5月15日

TiDB バージョン: 4.0.0-rc.2

## 互換性の変更

+ TiDB

    - TiDB Binlogが有効な場合、単一トランザクション（100MB）のサイズ制限を削除しました。トランザクションのサイズ制限は現在、10GBです。ただし、TiDB Binlogが有効で、ダウンストリームがKafkaの場合は、Kafkaのメッセージサイズ制限に応じて `txn-total-size-limit` パラメータを構成してください [#16941](https://github.com/pingcap/tidb/pull/16941)
    - クエリに `CLUSTER_LOG` テーブルで指定されていない場合、既定の時間範囲をクエリする動作をエラーとして返し、指定された時間範囲を要求するように変更しました [#17003](https://github.com/pingcap/tidb/pull/17003)
    - `CREATE TABLE` ステートメントを使用してパーティションテーブルを作成する際に、未サポートの `sub-partition` または `linear hash` オプションが指定されている場合、オプションを無視して通常のテーブルが作成されるように変更しました [#17197](https://github.com/pingcap/tidb/pull/17197)

+ TiKV

    - 暗号化関連の構成をセキュリティ関連の構成に移動し、TiKV構成ファイルの`[encryption]`を`[security.encryption]`に変更しました [#7810](https://github.com/tikv/tikv/pull/7810)

+ ツール

    - TiDB Lightning

        - データのインポート時のデフォルトのSQLモードを `ONLY_FULL_GROUP_BY,NO_AUTO_CREATE_USER` に変更して互換性を向上させました [#316](https://github.com/pingcap/tidb-lightning/pull/316)
        - tidb-backend モードでPDまたはTiKVポートにアクセスすることを禁止しました [#312](https://github.com/pingcap/tidb-lightning/pull/312)
        - TiDB Lightningの起動時にログ情報をtmpファイルに出力し、tmpファイルのパスを表示するように変更しました [#313](https://github.com/pingcap/tidb-lightning/pull/313)

## 重要なバグ修正

+ TiDB

    - `WHERE` 句に等価条件が1つしかない場合に間違ったパーティションが選択される問題を修正しました [#17054](https://github.com/pingcap/tidb/pull/17054)
    - `WHERE` 句に文字列列しか含まれていない場合の不正な結果を修正しました [#16660](https://github.com/pingcap/tidb/pull/16660)
    - `DELETE` 操作後のトランザクションで `PointGet` クエリを実行した際に発生するパニックの問題を修正しました [#16991](https://github.com/pingcap/tidb/pull/16991)
    - エラーが発生した場合にGCワーカーがデッドロックに遭遇する可能性がある問題を修正しました [#16915](https://github.com/pingcap/tidb/pull/16915)
    - TiKVの応答が遅れているがダウンしていない場合に不要なRegionMissリトライを回避しました [#16956](https://github.com/pingcap/tidb/pull/16956)
    - MySQLプロトコルのハンドシェイクフェーズのクライアントのログレベルを `DEBUG` に変更し、ログ出力に干渉する問題を解決しました [#16881](https://github.com/pingcap/tidb/pull/16881)
    - `TRUNCATE` 操作後にテーブルで定義された `PRE_SPLIT_REGIONS` 情報に従ってRegionが事前分割されない問題を修正しました [#16776](https://github.com/pingcap/tidb/pull/16776)
    - 2段階コミットの第2フェーズ中にTiKVが利用できない場合にリトライによるgoroutineの急増の性能低下を修正しました [#16876](https://github.com/pingcap/tidb/pull/16876)
    - 式をプッシュダウンできない場合にステートメントの実行のパニックの問題を修正しました [#16869](https://github.com/pingcap/tidb/pull/16869)
    - パーティションテーブルでのIndexMerge操作の実行結果が間違っていた問題を修正しました [#17124](https://github.com/pingcap/tidb/pull/17124)
    - `wide_table`の性能低下を引き起こしていたMemory Trackersのmutex競合を修正しました [#17234](https://github.com/pingcap/tidb/pull/17234)

+ TiFlash

    - データベースまたはテーブルの名前に特殊文字が含まれる場合に、システムが正常に起動できない問題を修正しました

## 新機能

+ TiDB

    - `BACKUP` および `RESTORE` コマンドのサポートを追加し、データのバックアップと復元を行います [#16960](https://github.com/pingcap/tidb/pull/16960)
    - 単一のリージョンでデータ容量を確認し、容量がしきい値を超えた場合にコミット前にリージョンを事前分割する機能をサポートしました [#16959](https://github.com/pingcap/tidb/pull/16959)
    - `Session` スコープで `LAST_PLAN_FROM_CACHE` 変数を追加し、最後に実行されたステートメントがプランキャッシュにヒットしたかどうかを示します [#16830](https://github.com/pingcap/tidb/pull/16830)
    - 遅いログと `SLOW_LOG` テーブルで `Cop_time` 情報を記録する機能をサポートしました [#16904](https://github.com/pingcap/tidb/pull/16904)
    - Go Runtimeのメモリ状態を監視するための追加のメトリクスをGrafanaに追加しました [#16928](https://github.com/pingcap/tidb/pull/16928)
    - `forUpdateTS` および `Read Consistency` 隔離レベル情報を一般ログに出力する機能をサポートしました [#16946](https://github.com/pingcap/tidb/pull/16946)
    - TiKV Regionのロック解除リクエストの重複を折りたたむ機能をサポートしました [#16925](https://github.com/pingcap/tidb/pull/16925)
    - `SET CONFIG` ステートメントを使用してPD/TiKVノードの構成を変更する機能をサポートしました [#16853](https://github.com/pingcap/tidb/pull/16853)
    - `CREATE TABLE` ステートメントで `auto_random` オプションをサポートしました [#16813](https://github.com/pingcap/tidb/pull/16813)
    - DistSQLリクエストにTaskIDを割り当て、TiKVがリクエストをより良くスケジュールして処理できるようにサポートしました [#17155](https://github.com/pingcap/tidb/pull/17155)
    - MySQLクライアントにログインした後にTiDBサーバーのバージョン情報を表示する機能をサポートしました [#17187](https://github.com/pingcap/tidb/pull/17187)
    - `GROUP_CONCAT` 関数の `ORDER BY` 句をサポートしました [#16990](https://github.com/pingcap/tidb/pull/16990)
    - 遅いログで `Plan_from_cache` 情報を表示することで、ステートメントがプランキャッシュにヒットしたかどうかを示す機能をサポートしました [#17121](https://github.com/pingcap/tidb/pull/17121)
    - TiDB DashboardにTiFlashの複数ディスク展開の容量情報を表示する機能を追加しました
    - DashboardでSQLステートメントを使用してTiFlashログをクエリする機能を追加しました

+ TiKV

    - 暗号化ストレージが有効な場合にtikv-ctlで暗号化デバッグをサポートし、暗号化ストレージが有効な場合にクラスタを操作および管理するためにtikv-ctlを使用できるようにしました [#7698](https://github.com/tikv/tikv/pull/7698)
    - スナップショットでロックカラムファミリを暗号化する機能をサポートしました [#7712](https://github.com/tikv/tikv/pull/7712)
    - Raftstoreレイテンシの要約のためにGrafanaダッシュボードでヒートマップを使用し、ジッタの問題をよりよく診断するようにしました [#7717](https://github.com/tikv/tikv/pull/7717)
    - gRPCメッセージのサイズの上限を設定する機能をサポートしました [#7824](https://github.com/tikv/tikv/pull/7824)
    - 暗号化関連の監視メトリクスをGrafanaダッシュボードに追加しました [#7827](https://github.com/tikv/tikv/pull/7827)
    - Application-Layer Protocol Negotiation (ALPN) をサポートしました [#7825](https://github.com/tikv/tikv/pull/7825)
    - Titanに関するさらなる統計情報を追加しました [#7818](https://github.com/tikv/tikv/pull/7818)
    - クライアントが提供するタスクIDを使用して統一読み取りプールでタスクの優先度が他のタスクによって低下しないようにする機能をサポートしました [#7814](https://github.com/tikv/tikv/pull/7814)
    - `batch insert` リクエストのパフォーマンスを改善しました [#7718](https://github.com/tikv/tikv/pull/7718)

+ PD

    - ノードをオフラインにする際のピアを削除する速度制限を削除しました [#2372](https://github.com/pingcap/pd/pull/2372)

+ TiFlash

    - Grafanaの **Read Index** のCountグラフの名前を **Ops** に変更しました
    - システム負荷が低いときにファイルディスクリプタを開くデータを最適化して、システムリソース消費を削減しました
    - データストレージ容量を制限するための容量関連の構成パラメータを追加しました

+ ツール
- TiDB Lightning

    - `tidb-lightning-ctl` に `fetch-mode` サブコマンドを追加して、TiKV クラスタモードを表示する [#287](https://github.com/pingcap/tidb-lightning/pull/287)

- TiCDC

    - `cdc cli`（changefeed）を使用してレプリケーションタスクの管理をサポートする [#546](https://github.com/pingcap/tiflow/pull/546)

- Backup & Restore (BR)

    - バックアップ中に GC 時間を自動的に調整するサポートを追加する [#257](https://github.com/pingcap/br/pull/257)
    - データをリストアする際に PD パラメータを調整して、リストアの高速化をサポートする [#198](https://github.com/pingcap/br/pull/198)

## バグ修正

+ TiDB

    - 複数の演算子で式の実行にベクトル化を使用するかどうかを判断するロジックを改善する [#16383](https://github.com/pingcap/tidb/pull/16383)
    - `IndexMerge` ヒントがデータベース名を正しくチェックしない問題を修正する [#16932](https://github.com/pingcap/tidb/pull/16932)
    - シーケンスオブジェクトを切り捨てることを禁止する [#17037](https://github.com/pingcap/tidb/pull/17037)
    - `INSERT`/`UPDATE`/`ANALYZE`/`DELETE` ステートメントをシーケンスオブジェクトで実行できる問題を修正する [#16957](https://github.com/pingcap/tidb/pull/16957)
    - 起動フェーズで内部 SQL ステートメントが正しく内部クエリとしてマークされない問題を修正する [#17062](https://github.com/pingcap/tidb/pull/17062)
    - TiFlash でサポートされているが TiKV でサポートされていないフィルタ条件が `IndexLookupJoin` オペレータにプッシュダウンされるとエラーが発生する問題を修正する [#17036](https://github.com/pingcap/tidb/pull/17036)
    - 照合が有効になってから発生する可能性がある `LIKE` 式の並行実行問題を修正する [#16997](https://github.com/pingcap/tidb/pull/16997)
    - 照合が有効になってから `LIKE` 関数が正しく `Range` クエリインデックスを構築できない問題を修正する [#16783](https://github.com/pingcap/tidb/pull/16783)
    - `Plan Cache` ステートメントがトリガされた後に `@@LAST_PLAN_FROM_CACHE` を実行すると誤った値が返される問題を修正する [#16831](https://github.com/pingcap/tidb/pull/16831)
    - `IndexMerge` の `TableFilter` が `IndexLookupJoin` 向けの候補パスを計算する際にインデックス上で欠落する問題を修正する [#16947](https://github.com/pingcap/tidb/pull/16947)
    - `MergeJoin` ヒントを使用し、`TableDual` オペレータが存在する場合に物理クエリプランが生成できない問題を修正する [#17016](https://github.com/pingcap/tidb/pull/17016)
    - Statement Summary テーブルの `Stmt_Type` 列の値の大文字と小文字が正しくない問題を修正する [#17018](https://github.com/pingcap/tidb/pull/17018)
    - 異なるユーザーが同じ `tmp-storage-path` を使用するとサービスを起動できない `Permission Denied` エラーが報告される問題を修正する [#16996](https://github.com/pingcap/tidb/pull/16996)
    - 複数の入力列によって結果タイプが決まる式（`CASE WHEN` など）で `NotNullFlag` 結果タイプが誤って設定される問題を修正する [#16995](https://github.com/pingcap/tidb/pull/16995)
    - dirty store が存在する場合に green GC が未解決のロックを残す可能性がある問題を修正する [#16949](https://github.com/pingcap/tidb/pull/16949)
    - 単一のキーに複数の異なるロックが存在する場合に green GC が未解決のロックを残す可能性がある問題を修正する [#16948](https://github.com/pingcap/tidb/pull/16948)
    - サブクエリが親クエリの列を参照すると `INSERT VALUE` ステートメントで誤った値が挿入される問題を修正する [#16952](https://github.com/pingcap/tidb/pull/16952)
    - `Float` 値に `AND` オペレータを使用した場合に誤った結果が得られる問題を修正する [#16666](https://github.com/pingcap/tidb/pull/16666)
    - 費用対効果の高いログでの `WAIT_TIME` フィールドの情報が間違っている問題を修正する [#16907](https://github.com/pingcap/tidb/pull/16907)
    - 悲観的トランザクションモードで `SELECT FOR UPDATE` ステートメントをスロー ログに記録できない問題を修正する [#16897](https://github.com/pingcap/tidb/pull/16897)
    - `Enum` や `Set` 型の列に `SELECT DISTINCT` を実行した場合に誤った結果が得られる問題を修正する [#16892](https://github.com/pingcap/tidb/pull/16892)
    - `SHOW CREATE TABLE` ステートメントで `auto_random_base` の表示が間違っている問題を修正する [#16864](https://github.com/pingcap/tidb/pull/16864)
    - `WHERE` 句の `string_value` の値が正しくない問題を修正する [#16559](https://github.com/pingcap/tidb/pull/16559)
    - `GROUP BY` ウィンドウ関数のエラーメッセージが MySQL のそれと一貫していない問題を修正する [#16165](https://github.com/pingcap/tidb/pull/16165)
    - 大文字が含まれるデータベース名を含む場合に `FLASH TABLE` ステートメントの実行に失敗する問題を修正する [#17167](https://github.com/pingcap/tidb/pull/17167)
    - Projection 実行プログラムの不正確なメモリトレースの問題を修正する [#17118](https://github.com/pingcap/tidb/pull/17118)
    - 異なるタイムゾーンで `SLOW_QUERY` テーブルの時間フィルタリングが正しくない問題を修正する [#17164](https://github.com/pingcap/tidb/pull/17164)
    - 仮想生成列を使用した場合に `IndexMerge` でパニックが発生する問題を修正する [#17126](https://github.com/pingcap/tidb/pull/17126)
    - `INSTR` および `LOCATE` 関数の大文字と小文字の問題を修正する [#17068](https://github.com/pingcap/tidb/pull/17068)
    - `tidb_allow_batch_cop` 設定が有効になってから `tikv server timeout` エラーが頻繁に報告される問題を修正する [#17161](https://github.com/pingcap/tidb/pull/17161)
    - Float 型で `XOR` 演算を実行すると MySQL 8.0 と一貫性のない結果が得られる問題を修正する [#16978](https://github.com/pingcap/tidb/pull/16978)
    - サポートされていない `ALTER TABLE REORGANIZE PARTITION` ステートメントを実行した場合にエラーが報告される問題を修正する [#17178](https://github.com/pingcap/tidb/pull/17178)
    - `EXPLAIN FORMAT="dot"  FOR CONNECTION ID` でサポートされていないプランが遭遇されるとエラーが報告される問題を修正する [#17160](https://github.com/pingcap/tidb/pull/17160)
    - Statement Summary システム変数を設定する際に値が検証されない問題を修正する [#17129](https://github.com/pingcap/tidb/pull/17129)
    - プランキャッシュが有効になっている場合に `UNSIGNED BIGINT` 主キーでオーバーフロー値を使用してクエリを実行するとエラーが報告される問題を修正する [#17120](https://github.com/pingcap/tidb/pull/17120)
    - Grafana の **TiDB Summary** ダッシュボードでマシン インスタンスおよびリクエストタイプによる QPS の表示が正しくない問題を修正する [#17105](https://github.com/pingcap/tidb/pull/17105)

+ TiKV

    - リストア後に多くの空のリージョンが生成される問題を修正する [#7632](https://github.com/tikv/tikv/pull/7632)
    - Raftstore が順序外れの read index レスポンスを受信したときに発生するパニックの問題を修正する [#7370](https://github.com/tikv/tikv/pull/7370)
    - 統一スレッドプールが有効になっているときに無効なストレージまたはコプロセッサー読み取りプール構成が拒否されない問題を修正する [#7513](https://github.com/tikv/tikv/pull/7513)
    - TiKV サーバーがシャットダウンされたときに `join` 操作がパニックを起こす問題を修正する [#7713](https://github.com/tikv/tikv/pull/7713)
    - 診断 API を使用して TiKV スロー ログを検索すると結果が返されない問題を修正する [#7776](https://github.com/tikv/tikv/pull/7776)
    - TiKV ノードが長時間実行されていると著しいメモリ断片化が発生する問題を修正する [#7556](https://github.com/tikv/tikv/pull/7556)
- 無効な日付が格納されている場合にSQLステートメントの実行に失敗する問題を修正する[#7268](https://github.com/tikv/tikv/pull/7268)
- バックアップデータをGCSから復元できない問題を修正する[#7739](https://github.com/tikv/tikv/pull/7739)
- 暗号化時にKMSキーIDが検証されない問題を修正する[#7719](https://github.com/tikv/tikv/pull/7719)
- 異なるアーキテクチャのコンパイラのCoprocessorの基盤的な正確さの問題を修正する[#7714](https://github.com/tikv/tikv/pull/7714) [#7730](https://github.com/tikv/tikv/pull/7730)
- 暗号化が有効になっている場合に`スナップショットインジェスション`エラーを修正する[#7815](https://github.com/tikv/tikv/pull/7815)
- 設定ファイルを書き換える際の`無効なデバイス間リンク`エラーを修正する[#7817](https://github.com/tikv/tikv/pull/7817)
- 空のファイルへの設定ファイルの書き込みで誤ったtomlフォーマットの問題を修正する[#7817](https://github.com/tikv/tikv/pull/7817)
- Raftstoreの破損したピアがリクエストを処理し続ける問題を修正する[#7836](https://github.com/tikv/tikv/pull/7836)

+ PD

    - pd-ctlで`region key`コマンドを使用した際に発生する`404`問題を修正する[#2399](https://github.com/pingcap/pd/pull/2399)
    - GrafanaダッシュボードからTSOおよびID割り当ての監視メトリクスが見つからない問題を修正する[#2405](https://github.com/pingcap/pd/pull/2405)
    - pd-recoverがDockerイメージに含まれていない問題を修正する[#2406](https://github.com/pingcap/pd/pull/2406)
    - TiDBダッシュボードがPD情報を正しく表示しない可能性がある問題を修正するために、データディレクトリのパスを絶対パスに解析する[#2420](https://github.com/pingcap/pd/pull/2420)
    - pd-ctlで`scheduler config shuffle-region-scheduler`コマンドを使用した際にデフォルトの出力がない問題を修正する[#2416](https://github.com/pingcap/pd/pull/2416)

+ TiFlash

    - 特定のシナリオで使用容量の誤った情報が報告される問題を修正する

+ Tools

    - TiDB Binlog

        - 下流がKafkaの場合に`mediumint`タイプのデータが処理されない問題を修正する[#962](https://github.com/pingcap/tidb-binlog/pull/962)
        - DDLでデータベース名がキーワードになっている場合にreparoがDDLステートメントを解析できない問題を修正する[#961](https://github.com/pingcap/tidb-binlog/pull/961)

    - TiCDC

        - `TZ`環境変数が設定されていない場合に誤ったタイムゾーンを使用する問題を修正する[#512](https://github.com/pingcap/tiflow/pull/512)
        - サーバーがエラーを適切に処理していないために、オーナーがリソースをクリーンアップしない問題を修正する[#528](https://github.com/pingcap/tiflow/pull/528)
        - TiKVに再接続する際にTiCDCが立ち往生する可能性がある問題を修正する[#531](https://github.com/pingcap/tiflow/pull/531)
        - テーブルスキーマを初期化する際のメモリ使用量を最適化する[#534](https://github.com/pingcap/tiflow/pull/534)
        - レプリケーション遅延を減らすためにレプリケーションステータスの変更を監視し、準リアルタイムの更新を実行するために`watch`モードを使用する[#481](https://github.com/pingcap/tiflow/pull/481)

    + Backup & Restore (BR)

        - BRが`auto_random`属性を持つテーブルを復元した後に、データの挿入が`重複エントリ`エラーを引き起こす問題を修正する[#241](https://github.com/pingcap/br/issues/241)