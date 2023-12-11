---
title: TiDB 3.0.2 リリースノート
aliases: ['/docs/dev/releases/release-3.0.2/','/docs/dev/releases/3.0.2/']
---

# TiDB 3.0.2 リリースノート

リリース日: 2019年8月7日

TiDB バージョン: 3.0.2

TiDB Ansible バージョン: 3.0.2

## TiDB

+ SQL オプティマイザ
    - クエリ内で同じテーブルが複数回出現し、論理的にクエリ結果が常に空である場合に、”スキーマ内で列を見つけられません”メッセージが報告される問題を修正 [#11247](https://github.com/pingcap/tidb/pull/11247)
    - `TIDB_INLJ` ヒントが一部のケース（例: `explain select /*+ TIDB_INLJ(t1) */ t1.b, t2.a from t t1, t t2 where t1.b = t2.a`）で正しく機能しないことにより、クエリプランが期待通りでない問題を修正 [#11362](https://github.com/pingcap/tidb/pull/11362)
    - クエリ結果の列名が一部のケース（例: `SELECT IF(1,c,c) FROM t`）で誤っている問題を修正 [#11379](https://github.com/pingcap/tidb/pull/11379)
    - 一部のクエリ（例: `SELECT 0 LIKE 'a string'`）が `LIKE` 式が一部のケースで暗黙に0に変換されることから `TRUE` を返す問題を修正 [#11411](https://github.com/pingcap/tidb/pull/11411)
    - `SHOW` 文でのサブクエリ（例: `SHOW COLUMNS FROM tbl WHERE FIELDS IN (SELECT 'a')`）をサポートする [#11459](https://github.com/pingcap/tidb/pull/11459)
    - 集約関数の関連する列が見つからずエラーが報告される問題を `outerJoinElimination` 最適化ルールが列エイリアスを正しく処理せずに修正し、最適化がより多くのクエリタイプをカバーするためにエイリアスの解析を改善 [#11377](https://github.com/pingcap/tidb/pull/11377)
    - ウィンドウ関数で構文の制限が違反された場合にエラーが報告されない問題を修正（例: `UNBOUNDED PRECEDING` がフレーム定義の最後に出現することは許可されていない） [#11543](https://github.com/pingcap/tidb/pull/11543)
    - `ERROR 3593 (HY000): You cannot use the window function FUNCTION_NAME in this context` エラーメッセージで `FUNCTION_NAME` が大文字であるために MySQL との互換性が損なわれる問題を修正 [#11535](https://github.com/pingcap/tidb/pull/11535)
    - 未実装の `IGNORE NULLS` 構文をウィンドウ関数で使用した場合にエラーが報告されない問題を修正 [#11593](https://github.com/pingcap/tidb/pull/11593)
    - オプティマイザが時間が等しい条件を正しく推定しない問題を修正 [#11512](https://github.com/pingcap/tidb/pull/11512)
    - フィードバック情報に基づいて Top-N 統計情報を更新するサポートを追加 [#11507](https://github.com/pingcap/tidb/pull/11507)
+ SQL 実行エンジン
    - `INSERT` 関数に `NULL` を含む場合に返される値が `NULL` でない問題を修正 [#11248](https://github.com/pingcap/tidb/pull/11248)
    - パーティションテーブルが `ADMIN CHECKSUM` 操作でチェックされた場合に計算結果が誤る可能性がある問題を修正 [#11266](https://github.com/pingcap/tidb/pull/11266)
    - INDEX JOIN が接頭辞インデックスを使用した場合に結果が誤る可能性がある問題を修正 [#11246](https://github.com/pingcap/tidb/pull/11246)
    - `DATE_ADD` 関数がマイクロ秒を含む日付数の差引算を行った場合に小数点以下の桁を誤って配置して結果が誤る可能性がある問題を修正 [#11288](https://github.com/pingcap/tidb/pull/11288)
    - `DATE_ADD` 関数が `INTERVAL` 内の負の数を誤って処理したことにより結果が誤る可能性がある問題を修正 [#11325](https://github.com/pingcap/tidb/pull/11325)
    - `Mod(%)`, `Multiple(*)` または `Minus(-)` によって 0 が返され、小数点以下の桁数が大きい場合に MySQL と異なる小数点以下の桁数が返される問題を修正（例: `select 0.000 % 0.11234500000000000000`） [#11251](https://github.com/pingcap/tidb/pull/11251)
    - `CONCAT` と `CONCAT_WS` 関数の結果の長さが `max_allowed_packet` を超えると `NULL` と警告が誤って返される問題を修正 [#11275](https://github.com/pingcap/tidb/pull/11275)
    - `SUBTIME` と `ADDTIME` 関数のパラメータが無効な場合に `NULL` と警告が誤って返される問題を修正 [#11337](https://github.com/pingcap/tidb/pull/11337)
    - `CONVERT_TZ` 関数のパラメータが無効な場合に `NULL` が誤って返される問題を修正 [#11359](https://github.com/pingcap/tidb/pull/11359)
    - このクエリのメモリ使用量を示すために `EXPLAIN ANALYZE` の結果に `MEMORY` 列を追加 [#11418](https://github.com/pingcap/tidb/pull/11418)
    - `EXPLAIN` の結果に `CARTESIAN` Join を追加 [#11429](https://github.com/pingcap/tidb/pull/11429)
    - 浮動小数点と倍精度浮動小数点の自動増分列の結果が誤っている問題を修正 [#11385](https://github.com/pingcap/tidb/pull/11385)
    - 擬似統計がダンプ時に一部の `nil` 情報によってパニックが発生する問題を修正 [#11460](https://github.com/pingcap/tidb/pull/11460)
    - 定数畳み込み最適化によって `SELECT ... CASE WHEN ... ELSE NULL ...` のクエリ結果が誤っている問題を修正 [#11441](https://github.com/pingcap/tidb/pull/11441)
    - `floatStrToIntStr` が `+999.9999e2` の入力を正しく解析しない問題を修正 [#11473](https://github.com/pingcap/tidb/pull/11473)
    - `DATE_ADD` と `DATE_SUB` 関数の結果がオーバーフローした場合に一部のケースで `NULL` が返されない問題を修正 [#11476](https://github.com/pingcap/tidb/pull/11476)
    - 長い文字列が整数に変換される場合に文字列に無効な文字が含まれると、変換結果が MySQL と異なる問題を修正 [#11469](https://github.com/pingcap/tidb/pull/11469)
    - `REGEXP BINARY` 関数の結果が MySQL と互換性がない問題を修正（この関数の大文字・小文字の区別） [#11504](https://github.com/pingcap/tidb/pull/11504)
    - `GRANT ROLE` 文が `CURRENT_ROLE` を受け取った場合にエラーが報告される問題を修正; `REVOKE ROLE` 文が `mysql.default_role` 権限を正しく取り消さない問題を修正 [#11356](https://github.com/pingcap/tidb/pull/11356)
    - `SELECT ADDDATE('2008-01-34', -1)` のような文を実行する際に `Incorrect datetime value` 警告情報の表示形式の問題を修正 [#11447](https://github.com/pingcap/tidb/pull/11447)
    - JSON データの浮動小数点フィールドが整数に変換される際に結果がオーバーフローした場合に `constant … overflows float` と報告されるべきところが `constant … overflows bigint` と誤って報告される問題を修正 [#11534](https://github.com/pingcap/tidb/pull/11534)
    - `DATE_ADD` 関数が `FLOAT`, `DOUBLE`, `DECIMAL` 列のパラメータを受け取った場合に異なる型変換結果が誤って返される問題を修正 [#11527](https://github.com/pingcap/tidb/pull/11527)
    - `DATE_ADD` 関数で INTERVAL 分数の符号が誤って処理されているために誤ったクエリ結果が発生する問題を修正 [#11615](https://github.com/pingcap/tidb/pull/11615)
    - `Ranger` が接頭辞インデックスを正しく処理しないことにより、インデックスルックアップに接頭辞インデックスが含まれる場合の誤ったクエリ結果を修正 [#11565](https://github.com/pingcap/tidb/pull/11565)
    - `NAME_CONST` 関数の第2パラメータが負の数である場合に `NAME_CONST` 関数が実行された際に `Incorrect arguments to NAME_CONST` メッセージが報告される問題を修正 [#11268](https://github.com/pingcap/tidb/pull/11268)
    - SQLステートメントに現在の時刻の計算が含まれ、値が複数回取得される場合に、結果がMySQLと互換性がない問題を修正し、同じSQLステートメントで現在の時刻を取得する際には同じ値を使用するようにする [#11394](https://github.com/pingcap/tidb/pull/11394)
    - `Close`が`baseExecutor`の`Close`でエラーが報告された場合、`ChildExecutor`の`Close`が呼び出されない問題を修正。この問題は、`KILL`ステートメントが影響を受けず、`ChildExecutor`が閉じられない場合に、Goroutineのリークを引き起こす可能性がある [#11576](https://github.com/pingcap/tidb/pull/11576)
+ サーバー
    - `LOAD DATA`でCSVファイル内の欠落している`TIMESTAMP`フィールドを処理する際に、自動追加される値が現在のタイムスタンプの代わりに0になる問題を修正 [#11250](https://github.com/pingcap/tidb/pull/11250)
    - `SHOW CREATE USER`ステートメントが関連する権限を正しくチェックしない問題、および`SHOW CREATE USER CURRENT_USER()`によって返される`USER`および`HOST`が間違っている可能性がある問題を修正 [#11229](https://github.com/pingcap/tidb/pull/11229)
    - JDBCで`executeBatch`が使用されている場合に、返される結果が間違っている可能性がある問題を修正 [#11290](https://github.com/pingcap/tidb/pull/11290)
    - TiKVサーバーのポートを変更する際にストリーミングクライアントのログ情報の出力を減らすようにする [#11370](https://github.com/pingcap/tidb/pull/11370)
    - ストリーミングクライアントをTiKVサーバーに再接続するロジックを最適化し、ストリーミングクライアントが長時間ブロックされないようにする [#11372](https://github.com/pingcap/tidb/pull/11372)
    - `INFORMATION_SCHEMA.TIDB_HOT_REGIONS`に`REGION_ID`を追加 [#11350](https://github.com/pingcap/tidb/pull/11350)
    - TiDB API `http://{TiDBIP}:10080/regions/hot`のPDのタイムアウトによってリージョン情報の取得が失敗することを避けるため、PD APIからのリージョン情報取得のタイムアウト期間をキャンセルする [#11383](https://github.com/pingcap/tidb/pull/11383)
    - リージョン関連のリクエストがHTTP APIでパーティション化されたテーブル関連のリージョンを返さない問題を修正 [#11466](https://github.com/pingcap/tidb/pull/11466)
    - ユーザーが悲観的なロックを手動で検証する際に、遅い操作によるロックタイムアウトの発生確率を減らすための以下の変更を行う：
        - 悲観的なロックのデフォルトのTTLを30秒から40秒に増加
        - 最大のTTLを60秒から120秒に増加
        - 最初の`LockKeys`リクエストから悲観的なロックの期間を計算する
    - TiKVクライアントの`SendRequest`関数のロジックを変更し、接続が構築できない場合に即座に別のピアに接続しようとするようにする [#11531](https://github.com/pingcap/tidb/pull/11531)
    - リージョンキャッシュを最適化し、別のストアが同じアドレスでオンラインになる際にストアが移動されたとラベル付けしてリージョンキャッシュ内のストア情報をできるだけ早く更新するようにする [#11567](https://github.com/pingcap/tidb/pull/11567)
    - `http://{TiDB_ADDRESS:TIDB_IP}/mvcc/key/{db}/{table}/{handle}`APIによって返される結果にリージョンIDを追加 [#11557](https://github.com/pingcap/tidb/pull/11557)
    - 範囲キーをエスケープしないScatter Table APIによってScatter Tableが機能しないエラーを修正 [#11298](https://github.com/pingcap/tidb/pull/11298)
    - リージョンが存在するストアをラベル付けして、このストアにアクセスできない場合にクエリパフォーマンスの低下を回避するためにリージョンキャッシュを最適化 [#11498](https://github.com/pingcap/tidb/pull/11498)
    - 同じ名前のデータベースを複数回削除した後もHTTP APIを通じてテーブルスキーマを取得できるエラーを修正 [#11585](https://github.com/pingcap/tidb/pull/11585)
+ DDL
    - 長さがゼロの非文字列カラムがインデックス化される際にエラーが発生する問題を修正 [#11214](https://github.com/pingcap/tidb/pull/11214)
    - 外部キー制約とフルテキストインデックスを持つカラムの変更を許可しない（注: TiDBは構文で外部キー制約とフルテキストインデックスをサポートしています） [#11274](https://github.com/pingcap/tidb/pull/11274)
    - `ALTER TABLE`ステートメントによって位置が変更され、カラムのデフォルト値が同時に使用されることによって、カラムのインデックスオフセットが間違っている可能性がある問題を修正 [#11346](https://github.com/pingcap/tidb/pull/11346)
    - JSONファイルの解析時に発生する2つの問題を修正：
        - `ConvertJSONToFloat`で`int64`が`uint64`の中間解析結果として使用され、精度オーバーフローエラーが発生する問題 [#11433](https://github.com/pingcap/tidb/pull/11433)
        - `ConvertJSONToInt`で`int64`が`uint64`の中間解析結果として使用され、精度オーバーフローエラーが発生する問題 [#11551](https://github.com/pingcap/tidb/pull/11551)
    - オートインクリメントカラムのインデックスを削除することを許可しない。自動インクリメントカラムが誤った結果を取得する可能性を防ぐため [#11399](https://github.com/pingcap/tidb/pull/11399)
    - 次の問題を修正 [#11492](https://github.com/pingcap/tidb/pull/11492):
        - 文字セットと`ALTER TABLE … MODIFY COLUMN`で指定された照合順序が一致しない場合に正しくエラーが報告されない問題
        - 文字セットと照合順序が`ALTER TABLE … MODIFY COLUMN`で複数回指定された場合にMySQLとの互換性がない問題
    - サブクエリのトレースの詳細を`TRACE`クエリの結果に追加 [#11458](https://github.com/pingcap/tidb/pull/11458)
    - `ADMIN CHECK TABLE`の実行パフォーマンスを最適化し、実行時間を大幅に短縮 [#11547](https://github.com/pingcap/tidb/pull/11547)
    - `SPLIT TABLE … REGIONS/INDEX`で返される結果に追加し、タイムアウト前に正常に分割されたリージョンの数を`TOTAL_SPLIT_REGION`と`SCATTER_FINISH_RATIO`に表示 [#11484](https://github.com/pingcap/tidb/pull/11484)
    - `SHOW CREATE TABLE`などのステートメントで、`ON UPDATE CURRENT_TIMESTAMP`が列属性であり、浮動小数点の精度が指定されている場合に表示される精度が不完全である問題を修正 [#11591](https://github.com/pingcap/tidb/pull/11591)
    - 仮想生成列の式に別の仮想生成列が含まれている場合に、列のインデックス結果が正しく計算されない問題を修正 [#11475](https://github.com/pingcap/tidb/pull/11475)
    - `ALTER TABLE … ADD PARTITION …`ステートメントで`VALUE LESS THAN`の後にマイナス記号を追加できない問題を修正 [#11581](https://github.com/pingcap/tidb/pull/11581)
+ モニター
    - モニタリングメトリック`TiKVTxnCmdCounter`が登録されていないため、データが収集および報告されない問題を修正 [#11316](https://github.com/pingcap/tidb/pull/11316)
    - バインド情報のモニタリングメトリック`BindUsageCounter`、`BindTotalGauge`、`BindMemoryUsage`を追加 [#11467](https://github.com/pingcap/tidb/pull/11467)

## TiKV

- Raftログが時間内に書き込まれない場合にTiKVがパニックするバグを修正 [#5160](https://github.com/tikv/tikv/pull/5160)
- TiKVがパニックした後にパニック情報がログファイルに書き込まれないバグを修正 [#5198](https://github.com/tikv/tikv/pull/5198)
- 悲観的トランザクションでInsert操作が誤って行われる可能性があるバグを修正 [#5203](https://github.com/tikv/tikv/pull/5203)
- マニュアル干渉が不要な一部のログの出力レベルをINFOに低減 [#5193](https://github.com/tikv/tikv/pull/5193)
- ストレージエンジンサイズの監視の精度を向上させる [#5200](https://github.com/tikv/tikv/pull/5200)
- tikv-ctlでのリージョンサイズの精度を向上させる [#5195](https://github.com/tikv/tikv/pull/5195)
- 悲観的なロックのデッドロック検出器のパフォーマンスを向上させる [#5192](https://github.com/tikv/tikv/pull/5192)
- タイタンストレージエンジンのGCのパフォーマンスを改善する[#5197](https://github.com/tikv/tikv/pull/5197)

## PD

- スキャッターリージョンスケジューラーが機能しないバグを修正[#1642](https://github.com/pingcap/pd/pull/1642)
- pd-ctlでマージリージョン操作を実行できないバグを修正[#1653](https://github.com/pingcap/pd/pull/1653)
- pd-ctlで墓石の削除操作を実行できないバグを修正[#1651](https://github.com/pingcap/pd/pull/1651)
- スキャンリージョン操作を実行する際、キースコープと重複するリージョンが見つからない問題を修正[#1648](https://github.com/pingcap/pd/pull/1648)
- PDにメンバーが正常に追加されることを保証する再試行メカニズムを追加[#1643](https://github.com/pingcap/pd/pull/1643)

## Tools

TiDB Binlog

- 起動時に構成項目をチェックする機能を追加し、無効な項目が見つかった場合はBinlogサービスを停止してエラーを報告する[#687](https://github.com/pingcap/tidb-binlog/pull/687)
- Drainerに特定の論理的なnode-idを指定するための構成を追加[#684](https://github.com/pingcap/tidb-binlog/pull/684)

TiDB Lightning

- 2つのチェックサムが同時に実行されている場合に、`tikv_gc_life_time`が元の値に戻らない問題を修正[#218](https://github.com/pingcap/tidb-lightning/pull/218)
- 起動時に構成項目をチェックする機能を追加し、無効な項目が見つかった場合はBinlogサービスを停止してエラーを報告する[#217](https://github.com/pingcap/tidb-lightning/pull/217)

## TiDB Ansible

- ディスクパフォーマンスモニタが秒をミリ秒として処理する単位エラーを修正[#840](https://github.com/pingcap/tidb-ansible/pull/840)
- Sparkに`log4j`構成ファイルを追加[#841](https://github.com/pingcap/tidb-ansible/pull/841)
- Binlogが有効になっていてKafkaまたはZooKeeperが構成されている場合に、Prometheus構成ファイルが誤った形式で生成される問題を修正[#844](https://github.com/pingcap/tidb-ansible/pull/844)
- 生成されたTiDB構成ファイルに`pessimistic-txn`構成パラメータが含まれない問題を修正[#850](https://github.com/pingcap/tidb-ansible/pull/850)
- TiDBダッシュボードにメトリクスを追加し、最適化[#853](https://github.com/pingcap/tidb-ansible/pull/853)
- TiDBダッシュボードの各モニタリング項目に説明を追加[#854](https://github.com/pingcap/tidb-ansible/pull/854)
- クラスタステータスをより良く表示し、問題をトラブルシューティングするためのTiDBサマリーダッシュボードを追加[#855](https://github.com/pingcap/tidb-ansible/pull/855)
- TiKVダッシュボードのAllocator Statsモニタリング項目を更新[#857](https://github.com/pingcap/tidb-ansible/pull/857)
- Node Exporterのアラート式の単位エラーを修正[#860](https://github.com/pingcap/tidb-ansible/pull/860)
- TiSpark jarパッケージをv2.1.2にアップグレード[#862](https://github.com/pingcap/tidb-ansible/pull/862)
- Ansible Task機能の説明を更新[#867](https://github.com/pingcap/tidb-ansible/pull/867)
- TiDBダッシュボードのローカルリーダーリクエストのモニタリング項目の表現を更新[#874](https://github.com/pingcap/tidb-ansible/pull/874)
- 概要ダッシュボードのTiKVメモリモニタリング項目の表現を更新し、誤って表示される問題を修正[#879](https://github.com/pingcap/tidb-ansible/pull/879)
- KafkaモードでのBinlogサポートを削除[#878](https://github.com/pingcap/tidb-ansible/pull/878)
- `rolling_update.yml`操作を実行する際にPDがリーダーの移行に失敗する問題を修正[#887](https://github.com/pingcap/tidb-ansible/pull/887)