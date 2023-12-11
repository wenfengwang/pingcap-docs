---
title: TiDB 3.0.8のリリースノート
aliases: ['/docs/dev/releases/release-3.0.8/','/docs/dev/releases/3.0.8/']
---

# TiDB 3.0.8のリリースノート

リリース日: 2019年12月31日

TiDBバージョン: 3.0.8

TiDB Ansibleバージョン: 3.0.8

## TiDB

+ SQLオプティマイザ
    - キャッシュの更新がタイミングに遅れることによって誤ったSQLバインディング計画が発生する問題を修正 [#13891](https://github.com/pingcap/tidb/pull/13891)
    - SQLステートメントにシンボルリストが含まれる場合にSQLバインディングが無効になる可能性がある問題を修正 [#14004](https://github.com/pingcap/tidb/pull/14004)
    - SQLステートメントが`;`で終了する場合にSQLバインディングを作成または削除できない問題を修正 [#14113](https://github.com/pingcap/tidb/pull/14113)
    - `PhysicalUnionScan`演算子が誤った統計情報を設定するために誤ったSQLクエリプランが選択される可能性がある問題を修正 [#14133](https://github.com/pingcap/tidb/pull/14133)
    - `minAutoAnalyzeRatio`の制限を削除し、`autoAnalyze`をよりタイムリーに行うための改善 [#14015](https://github.com/pingcap/tidb/pull/14015)
+ SQL実行エンジン
    - `INSERT/REPLACE/UPDATE ... SET ... = DEFAULT`構文がエラーを報告する場合や、`DEFAULT`式の使用が仮想生成された列と組み合わされる場合にエラーを報告する問題を修正 [#14160](https://github.com/pingcap/tidb/pull/14160)
    - 文字列を浮動小数点に変換する際に`INSERT`ステートメントがエラーを報告する場合がある問題を修正 [#14011](https://github.com/pingcap/tidb/pull/14011)
    - `HashAgg`実行ユニットの並行値が誤って初期化されたため、集約の効率が低下することがある問題を修正 [#13811](https://github.com/pingcap/tidb/pull/13811)
    - 括弧内に`group by item`が存在する場合にエラーが報告される問題を修正 [#13658](https://github.com/pingcap/tidb/pull/13658)
    - `OUTER JOIN`の実行に関して、TiDBが`group by item`を誤って計算するためにエラーを報告する問題を修正 [#14014](https://github.com/pingcap/tidb/pull/14014)
    - 範囲を超えるデータが範囲パーティションテーブルに書き込まれた場合に不正確なエラーメッセージが表示される問題を修正 [#14107](https://github.com/pingcap/tidb/pull/14107)
    - `PadCharToFullLength`が特定のケースで予期しないクエリ結果を破棄する可能性があるため、[PR #10124](https://github.com/pingcap/tidb/pull/10124)を取り消し、`PadCharToFullLength`の効果をキャンセルする修正を実行 [#14157](https://github.com/pingcap/tidb/pull/14157)
    - `ExplainExec`での予期しない`close()`呼び出しによって`EXPLAIN ANALYZE`ステートメントの実行時にgoroutineリークが発生する問題を修正 [#14226](https://github.com/pingcap/tidb/pull/14226)
+ DDL
    - `change column`/`modify column`のエラーメッセージ出力を最適化し、わかりやすくするための改善 [#13796](https://github.com/pingcap/tidb/pull/13796)
    - パーティションテーブルのリージョンを分割する`SPLIT PARTITION TABLE`構文のサポートを追加 [#13929](https://github.com/pingcap/tidb/pull/13929)
    - インデックスの長さが3072バイトを超えており、インデックスの作成時に長さが誤ってチェックされるためエラーが報告されない問題を修正 [#13779](https://github.com/pingcap/tidb/pull/13779)
    - パーティションテーブルにインデックスを追加する際に時間がかかりすぎるため`GCライフタイムがトランザクションの期間よりも短い`エラーメッセージが報告される可能性がある問題を修正 [#14132](https://github.com/pingcap/tidb/pull/14132)
    - `DROP COLUMN`/`MODIFY COLUMN`/`CHANGE COLUMN`を実行する際に外部キーがチェックされないため、`SELECT * FROM information_schema.KEY_COLUMN_USAGE`を実行するとパニックが発生する問題を修正 [#14105](https://github.com/pingcap/tidb/pull/14105)
+ サーバ
    - ステートメントサマリの改善:
        - SQLメトリクスフィールドを大量に追加し、より詳細にSQLステートメントを分析するための改善 [#14151](https://github.com/pingcap/tidb/pull/14151), [#14168](https://github.com/pingcap/tidb/pull/14168)
        - `stmt-summary.refresh-interval`パラメータを追加し、`events_statements_summary_by_digest`テーブルの古いデータを`events_statements_summary_by_digest_history`テーブルに移動するかどうかを制御する（デフォルトの間隔: 30分） [#14161](https://github.com/pingcap/tidb/pull/14161)
        - 古いデータを`events_statements_summary_by_digest`から`events_statements_summary_by_digest_history`テーブルに保存する`events_statements_summary_by_digest_history`テーブルを追加 [#14166](https://github.com/pingcap/tidb/pull/14166)
    - RBAC関連の内部SQLステートメントを実行した際、binlogが誤って出力される問題を修正 [#13890](https://github.com/pingcap/tidb/pull/13890)
    - TiDBサーババージョンを変更する機能を制御するための`server-version`設定項目を追加 [#13906](https://github.com/pingcap/tidb/pull/13906)
    - TiDB binlogの書き込みを回復するためのHTTPインターフェースの使用を可能にする機能を追加 [#13892](https://github.com/pingcap/tidb/pull/13892)
    - `GRANT roles TO user`に必要な権限を`GrantPriv`から`ROLE_ADMIN`または`SUPER`に変更し、MySQLの挙動と一貫性を保つための改善 [#13932](https://github.com/pingcap/tidb/pull/13932)
    - `GRANT`ステートメントがデータベース名を指定しない場合、TiDBの現在のデータベースを使用する挙動を修正し、`No database selected`エラーを報告するように変更し、MySQLの挙動との互換性を保つための改善 [#13784](https://github.com/pingcap/tidb/pull/13784)
    - `REVOKE`ステートメントの実行権限を`SuperPriv`から、対応するスキーマの権限がユーザにある場合にのみ`REVOKE`が実行可能であるように変更し、MySQLの挙動と一貫性を保つための改善 [#13306](https://github.com/pingcap/tidb/pull/13306)
    - `GRANT ALL`構文に`WITH GRANT OPTION`が含まれない場合に`GrantPriv`が誤って対象ユーザに付与される問題を修正 [#13943](https://github.com/pingcap/tidb/pull/13943)
    - `LoadDataInfo`が`addRecord`を呼び出せない場合に`LOAD DATA`ステートメントの誤った動作についてエラーメッセージが原因を含まない問題を修正 [#13980](https://github.com/pingcap/tidb/pull/13980)
    - 複数のSQLステートメントが同じ`StartTime`を共有するために誤った遅延クエリ情報が出力される問題を修正 [#13898](https://github.com/pingcap/tidb/pull/13898)
    - `batchClient`が大きなトランザクションを処理する際にメモリがリークする可能性がある問題を修正 [#14032](https://github.com/pingcap/tidb/pull/14032)
    - `system_time_zone`が常に`CST`と表示される問題を修正し、現在のTiDBの`system_time_zone`が`mysql.tidb`テーブルの`systemTZ`から取得されるように変更 [#14086](https://github.com/pingcap/tidb/pull/14086)
    - `GRANT ALL`構文がユーザに対してすべての権限を付与しない問題を修正 [#14092](https://github.com/pingcap/tidb/pull/14092)
    - `Priv_create_user`権限が`CREATE ROLE`および`DROP ROLE`に対して無効な問題を修正 [#14088](https://github.com/pingcap/tidb/pull/14088)
    - `ErrInvalidFieldSize`のエラーコードを`1105(Unknow Error)`から`3013`に変更 [#13737](https://github.com/pingcap/tidb/pull/13737)
    - TiDBサーバを停止するための`SHUTDOWN`コマンドを追加し、`ShutdownPriv`権限を追加 [#14104](https://github.com/pingcap/tidb/pull/14104)
    - `DROP ROLE`ステートメントの原子性の問題を修正し、TiDBがステートメントの実行に失敗した際に予期せず一部のロールが削除されるのを回避するための改善 [#14130](https://github.com/pingcap/tidb/pull/14130)
    - `SHOW VARIABLE`の結果で、`tidb_enable_window_function`がTiDBのバージョンを3.0にアップグレードした際に誤って`1`が表示される問題を修正し、誤った結果が`0`に置換されるように変更 [#14131](https://github.com/pingcap/tidb/pull/14131)
    - `gcworker`がTiKVノードがオフラインになった場合に、`gcworker`が継続的にリトライを行うためにgoroutineがリークする可能性のある問題を修正しました[#14106](https://github.com/pingcap/tidb/pull/14106)
    - スロークエリログにおいてbinlogの`Prewrite`時間を記録して、問題の追跡の利便性を向上させるために調整しました[#14138](https://github.com/pingcap/tidb/pull/14138)
    - `tidb_enable_table_partition`変数が`GLOBAL SCOPE`をサポートするようにしました[#14091](https://github.com/pingcap/tidb/pull/14091)
    - 新たな特権が追加された場合に、新たに追加された特権が対応するユーザーに正しく付与されないためにユーザーの特権が抜けているか誤って追加されている可能性のある問題を修正しました[#14178](https://github.com/pingcap/tidb/pull/14178)
    - TiKVサーバーが切断された場合に`rpcClient`が閉じられないために`CheckStreamTimeoutLoop` goroutineがリークする可能性のある問題を修正しました[#14227](https://github.com/pingcap/tidb/pull/14227)
    - 証明書ベースの認証をサポートしました([ユーザードキュメント](/certificate-authentication.md))[#13955](https://github.com/pingcap/tidb/pull/13955)
+ トランザクション
    - 新しいクラスターが作成された場合に`tidb_txn_mode`変数のデフォルト値を`""`から`"pessimistic"`に更新しました[#14171](https://github.com/pingcap/tidb/pull/14171)
    - 悲観的トランザクションの待機時間が長すぎる問題を修正しました。単一のステートメントのためのロック待ち時間がトランザクションがリトライされた場合にリセットされていないためです[#13990](https://github.com/pingcap/tidb/pull/13990)
    - 悲観的なトランザクションモードのために、未変更データがロック解除されるために誤ったデータが読み取られる可能性のある問題を修正しました[#14050](https://github.com/pingcap/tidb/pull/14050)
    - prewriteがmocktikvにおいて実行される際にトランザクションタイプが区別されていないために繰り返し挿入値の制限がチェックされる問題を修正しました[#14175](https://github.com/pingcap/tidb/pull/14175)
    - `session.TxnState`が`Invalid`の場合にトランザクションが正しく処理されていないためにパニックが発生する問題を修正しました[#13988](https://github.com/pingcap/tidb/pull/13988)
    - mocktikvにおいて`ErrConfclit`構造体が`ConflictCommitTS`を含んでいない問題を修正しました[#14080](https://github.com/pingcap/tidb/pull/14080)
    - ロックが解決された後にTiDBがロックのタイムアウトを正しくチェックしていないためにトランザクションがブロックされる問題を修正しました[#14083](https://github.com/pingcap/tidb/pull/14083)
+ モニター
    - `LockKeys`において`pessimistic_lock_keys_duration`のモニタリング項目を追加しました[#14194](https://github.com/pingcap/tidb/pull/14194)

## TiKV

+ Coprocessor
    - Coprocessorにおいてエラーが発生した際に、出力ログのレベルを`error`から`warn`に変更しました[#6051](https://github.com/tikv/tikv/pull/6051)
    - 統計サンプリングデータの更新動作を直接行うから削除して挿入するように変更し、tidb-serverの更新動作と一貫性を保つために調整しました[#6069](https://github.com/tikv/tikv/pull/6096)
+ Raftstore
    - `peerfsm`に対して`destroy`メッセージを繰り返し送信し、`peerfsm`が複数回破棄されることによって発生するパニックを修正しました[#6297](https://github.com/tikv/tikv/pull/6297)
    - テーブルごとにリージョンを分割する機能のデフォルト値を`true`から`false`に変更して、デフォルトでテーブルごとにリージョンを分割する機能を無効にしました[#6253](https://github.com/tikv/tikv/pull/6253)
+ Engine
    - 極端な条件下でRocksDBイテレータのエラーが正しく処理されないために空のデータが返される可能性のある問題を修正しました[#6326](https://github.com/tikv/tikv/pull/6326)
+ トランザクション
    - TiKVが悲観的なロックを誤ってクリーンアップしているためにデータの書き込みに失敗し、GCがブロックされる問題を修正しました[#6354](https://github.com/tikv/tikv/pull/6354)
    - ロックの競合が深刻なシナリオにおいて性能を向上させるために、悲観的なロックの待機メカニズムを最適化しました[#6296](https://github.com/tikv/tikv/pull/6296)
+ デフォルト値を`tikv_alloc/default`から`jemalloc`に更新しました[#6206](https://github.com/tikv/tikv/pull/6206)

## PD

- クライアント
    - `context`を使用してクライアントを作成し、新しいクライアントを作成する際のタイムアウト期間を設定することをサポートしました[#1994](https://github.com/pingcap/pd/pull/1994)
    - `KeepAlive`接続を作成することをサポートしました[#2035](https://github.com/pingcap/pd/pull/2035)
- `/api/v1/regions`APIのパフォーマンスを最適化しました[#1986](https://github.com/pingcap/pd/pull/1986)
- `tombstone`状態のストアを削除する際にパニックが発生する可能性がある問題を修正しました[#2038](https://github.com/pingcap/pd/pull/2038)
- ディスクからリージョン情報を読み込む際に、重複したリージョンが誤って削除される可能性のある問題を修正しました[#2011](https://github.com/pingcap/pd/issues/2011), [#2040](https://github.com/pingcap/pd/pull/2040)
- etcdをv3.4.0からv3.4.3にアップグレードしました（アップグレード後にpd-recoverを使用してのみetcdをダウングレードできることに注意してください）[#2058](https://github.com/pingcap/pd/pull/2058)

## Tools

+ TiDB Binlog
    - PumpがDDLがコミットされたbinlogを受信しない問題を修正しました[#853](https://github.com/pingcap/tidb-binlog/pull/853)

## TiDB Ansible

- 簡略化された構成項目を元に戻しました[#1053](https://github.com/pingcap/tidb-ansible/pull/1053)
- ローリングアップデートを実行する際にTiDBバージョンをチェックするロジックを最適化しました[#1056](https://github.com/pingcap/tidb-ansible/pull/1056)
- TiSparkをv2.1.8にアップグレードしました[#1061](https://github.com/pingcap/tidb-ansible/pull/1061)
- GrafanaにおいてPDのロールモニタリング項目が誤って表示される問題を修正しました[#1065](https://github.com/pingcap/tidb-ansible/pull/1065)
- GrafanaにおいてTiKV Detailページでの`Thread Voluntary Context Switches`と`Thread Nonvoluntary Context Switches`のモニタリング項目を最適化しました[#1071](https://github.com/pingcap/tidb-ansible/pull/1071)