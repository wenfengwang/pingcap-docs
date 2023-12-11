---
title: TiDB 4.0.2 リリースノート
aliases: ['/docs/dev/releases/release-4.0.2/']
---

# TiDB 4.0.2 リリースノート

リリース日: 2020年7月1日

TiDB バージョン: 4.0.2

## 互換性の変更

+ TiDB

    - 遅いクエリログとステートメントサマリーテーブルから機密情報を削除 [#18130](https://github.com/pingcap/tidb/pull/18130)
    - シーケンスキャッシュで負の値を禁止 [#18103](https://github.com/pingcap/tidb/pull/18103)
    - `CLUSTER_INFO`テーブルから墓石のあるTiKVとTiFlashストアを削除 [#17953](https://github.com/pingcap/tidb/pull/17953)
    - 診断ルールを`current-load`から`node-check`に変更 [#17660](https://github.com/pingcap/tidb/pull/17660)

+ PD

    - `store-limit`を永続化し、`store-balance-rate`を削除 [#2557](https://github.com/pingcap/pd/pull/2557)

## 新しい変更

- デフォルトで、TiDBとTiDBダッシュボードは製品の改善方法を理解するために、PingCAPと使用状況の詳細を共有します [#18180](https://github.com/pingcap/tidb/pull/18180)。共有される内容と共有方法の詳細については、[Telemetry](/telemetry.md)を参照してください。

## 新機能

+ TiDB

    - `INSERT`ステートメントでの`MEMORY_QUOTA()`ヒントをサポート [#18101](https://github.com/pingcap/tidb/pull/18101)
    - `TLS`証明書の`SAN`フィールドに基づく認証をサポート [#17698](https://github.com/pingcap/tidb/pull/17698)
    - `REGEXP()`関数の照合をサポート [#17581](https://github.com/pingcap/tidb/pull/17581)
    - `sql_select_limit`セッションとグローバル変数をサポート [#17604](https://github.com/pingcap/tidb/pull/17604)
    - 新たに追加されたパーティションのRegion分割をデフォルトでサポート [#17665](https://github.com/pingcap/tidb/pull/17665)
    - `TiFlash Coprocessor`に`IF()`/`BITXOR()`/`BITNEG()`/`JSON_LENGTH()`関数のプッシュをサポート [#17651](https://github.com/pingcap/tidb/pull/17651) [#17592](https://github.com/pingcap/tidb/pull/17592)
    - 近似結果の`COUNT(DISTINCT)`を計算する新しい集約関数`APPROX_COUNT_DISTINCT()`をサポート [#18120](https://github.com/pingcap/tidb/pull/18120)
    - TiFlashでの照合とTiFlashへの関連する照合関数のプッシュをサポート [#17705](https://github.com/pingcap/tidb/pull/17705)
    - `INFORMATION_SCHEMA.INSPECTION_RESULT`テーブルに`STATUS_ADDRESS`列を追加し、サーバーのステータスアドレスを示す [#17695](https://github.com/pingcap/tidb/pull/17695)
    - `MYSQL.BIND_INFO`テーブルに`SOURCE`列を追加し、結合の作成方法を示す  [#17587](https://github.com/pingcap/tidb/pull/17587)
    - SQLステートメントのプランキャッシュ使用状況を示す`PERFORMANCE_SCHEMA.EVENTS_STATEMENTS_SUMMARY_BY_DIGEST`テーブルに`PLAN_IN_CACHE`および`PLAN_CACHE_HITS`列を追加 [#17493](https://github.com/pingcap/tidb/pull/17493)
    - 各演算子の実行情報を収集し、その情報を遅いクエリログに記録するかどうかを制御するための`enable-collect-execution-info`構成項目と`tidb_enable_collect_execution_info`セッション変数を追加 [#18073](https://github.com/pingcap/tidb/pull/18073) [#18072](https://github.com/pingcap/tidb/pull/18072)
    - 遅いクエリログのクエリを非表示化するかどうかを制御するための`tidb_slow_log_masking`グローバル変数を追加  [#17694](https://github.com/pingcap/tidb/pull/17694)
    - `INFORMATION_SCHEMA.INSPECTION_RESULT`テーブルに`storage.block-cache.capacity` TiKV設定項目の診断ルールを追加 [#17671](https://github.com/pingcap/tidb/pull/17671)
    - データのバックアップとリストアを行うための`BACKUP`および`RESTORE` SQLステートメントを追加 [#15274](https://github.com/pingcap/tidb/pull/15274)

+ TiKV

    - `TiKV Control`で`encryption-meta`コマンドをサポート [#8103](https://github.com/tikv/tikv/pull/8103)
    - `RocksDB::WriteImpl`のためのパフォーマンスコンテキストメトリクスを追加 [#7991](https://github.com/tikv/tikv/pull/7991)

+ PD

    - リーダーピアを削除しようとする操作が失敗した場合に、直ちにオペレータを失敗させるサポートを追加 [#2551](https://github.com/pingcap/pd/pull/2551)
    - TiFlashストアに適したデフォルトのストア制限を設定するサポートを追加 [#2559](https://github.com/pingcap/pd/pull/2559)

+ TiFlash

    - コプロセッサに新しい集約関数`APPROX_COUNT_DISTINCT`をサポート
    - デフォルトで`rough set filter`機能を有効化
    - TiFlashをARMアーキテクチャで実行できるようにサポートを追加
    - コプロセッサで`JSON_LENGTH`関数をプッシュダウンするサポートを追加

+ ツール

    - TiCDC

        - サブタスクを新しい`capture`に移行するサポートを追加 [#665](https://github.com/pingcap/tiflow/pull/665)
        - TiCDC GC TTLを削除するための`cli`コマンドを追加 [#652](https://github.com/pingcap/tiflow/pull/652)
        - MQシンクでのcanalプロトコルをサポート [#649](https://github.com/pingcap/tiflow/pull/649)

## 改善

+ TiDB

    - CM-Sketchがあまりにも多くのメモリを消費するときに生じるGolangメモリ割り当てによるクエリ遅延を削減 [#17545](https://github.com/pingcap/tidb/pull/17545)
    - TiKVサーバーが障害回復プロセスであるときに、クラスターのQPS回復期間を短縮 [#17681](https://github.com/pingcap/tidb/pull/17681)
    - パーティションテーブルのTiKV/TiFlash Coprocessorに集約関数をプッシュするサポートを追加 [#17655](https://github.com/pingcap/tidb/pull/17655)
    - インデックスの等しい条件の行数推定の精度を向上 [#17611](https://github.com/pingcap/tidb/pull/17611)

+ TiKV

    - PDクライアントのパニックログを改善 [#8093](https://github.com/tikv/tikv/pull/8093)
    - `process_cpu_seconds_total`および`process_start_time_seconds`モニタリングメトリクスを再追加 [#8029](https://github.com/tikv/tikv/pull/8029)

+ TiFlash

    - 古いバージョンからのアップグレード時の後方互換性を向上 [#786](https://github.com/pingcap/tics/pull/786)
    - デルタインデックスのメモリ消費を削減 [#787](https://github.com/pingcap/tics/pull/787)
    - より効率的な更新アルゴリズムを使用するようにデルタインデックスを改善 [#794](https://github.com/pingcap/tics/pull/794)

+ ツール

    - バックアップとリストア（BR）

        - リストアプロセスをパイプライン処理することでパフォーマンスを向上 [#266](https://github.com/pingcap/br/pull/266)

## バグ修正

+ TiDB

    - `tidb_isolation_read_engines`が変更された後のプランキャッシュからの誤った実行計画の問題を修正 [#17570](https://github.com/pingcap/tidb/pull/17570)
    - `EXPLAIN FOR CONNECTION`ステートメントを実行する際に発生する場合があるランタイムエラーを修正 [#18124](https://github.com/pingcap/tidb/pull/18124)
    - `last_plan_from_cache`セッション変数の一部の場合に誤った結果を修正 [#18111](https://github.com/pingcap/tidb/pull/18111)
    - `UNIX_TIMESTAMP()`関数をプランキャッシュから実行する際に発生するランタイムエラーを修正 [#18002](https://github.com/pingcap/tidb/pull/18002) [#17673](https://github.com/pingcap/tidb/pull/17673)
    - `HashJoin`エグゼキューターの子が`NULL`カラムを返す場合に発生するランタイムエラーを修正 [#17937](https://github.com/pingcap/tidb/pull/17937)
    - `DROP DATABASE`ステートメントと同一のデータベース内で他のDDLステートメントを並行して実行すると発生するランタイムエラーを修正 [#17659](https://github.com/pingcap/tidb/pull/17659)
    - ユーザ変数に対する`COERCIBILITY()`関数の誤った結果を修正 [#17890](https://github.com/pingcap/tidb/pull/17890)
- `IndexMergeJoin` executorの偶発的なスタックの問題を修正する[#18091](https://github.com/pingcap/tidb/pull/18091) 
- メモリクォータとクエリキャンセル時の`IndexMergeJoin` executorのハング問題を修正する[#17654](https://github.com/pingcap/tidb/pull/17654) 
- `Insert`および`Replace` executorのメモリ使用量の過剰なカウントを修正する[#18062](https://github.com/pingcap/tidb/pull/18062) 
- 同一データベース内で`DROP DATABASE`および`DROP TABLE`が同時に実行された場合にTiFlashストレージへのデータレプリケーションが停止する問題を修正する[#17901](https://github.com/pingcap/tidb/pull/17901) 
- TiDBとオブジェクトストレージサービス間の`BACKUP`/`RESTORE`の失敗を修正する[#17844](https://github.com/pingcap/tidb/pull/17844) 
- アクセスが拒否された際の権限チェック失敗の誤ったエラーメッセージを修正する[#17724](https://github.com/pingcap/tidb/pull/17724) 
- `DELETE`/`UPDATE`文から生成されたクエリフィードバックを破棄する[#17843](https://github.com/pingcap/tidb/pull/17843) 
- `AUTO_RANDOM`プロパティのないテーブルの`AUTO_RANDOM_BASE`の変更を禁止する[#17828](https://github.com/pingcap/tidb/pull/17828) 
- テーブルが`ALTER TABLE ... RENAME`によってデータベース間で移動された際に`AUTO_RANDOM`列が間違った結果を割り当てる問題を修正する[#18243](https://github.com/pingcap/tidb/pull/18243) 
- `tidb_isolation_read_engines`の値を`tidb`なしで設定する際に一部のシステムテーブルにアクセスできない問題を修正する[#17719](https://github.com/pingcap/tidb/pull/17719) 
- 大きな整数および浮動小数点数のJSON比較の不正確な結果を修正する[#17717](https://github.com/pingcap/tidb/pull/17717) 
- `COUNT()`関数の誤った10進数プロパティを修正する [#17704](https://github.com/pingcap/tidb/pull/17704) 
- 入力のタイプがバイナリ文字列の場合に`HEX()`関数の不正確な結果を修正する[#17620](https://github.com/pingcap/tidb/pull/17620) 
- フィルタ条件なしで`INFORMATION_SCHEMA.INSPECTION_SUMMARY`テーブルをクエリした際に空の結果が返される問題を修正する[#17697](https://github.com/pingcap/tidb/pull/17697) 
- `ALTER USER`文で使用されるハッシュ化されたパスワードが予期しないものになる問題を修正する[#17646](https://github.com/pingcap/tidb/pull/17646) 
- `ENUM`および`SET`の値に対する照合のサポートを追加する[#17701](https://github.com/pingcap/tidb/pull/17701) 
- テーブルのpre-splitting Regionsのタイムアウトメカニズムがテーブル作成時に機能しない問題を修正する[#17619](https://github.com/pingcap/tidb/pull/17619) 
- DDLジョブが再試行された際にスキーマが予期せず更新され、DDLジョブのアトミシティが損なわれる可能性がある問題を修正する[#17608](https://github.com/pingcap/tidb/pull/17608) 
- 引数に列が含まれる場合の`FIELD()`関数の不正確な結果を修正する[#17562](https://github.com/pingcap/tidb/pull/17562) 
- `max_execution_time`ヒントが時折機能しない問題を修正する[#17536](https://github.com/pingcap/tidb/pull/17536) 
- `EXPLAIN ANALYZE`の結果に冗長に並列情報が出力される問題を修正する[#17350](https://github.com/pingcap/tidb/pull/17350) 
- `STR_TO_DATE`関数の`%h`の非互換な動作を修正する[#17498](https://github.com/pingcap/tidb/pull/17498) 
- `tidb_replica_read`が`follower`に設定されている際にリーダーとフォロワー/ラーナー間にネットワーク分断がある場合にフォロワー/ラーナーがリトライし続ける問題を修正する[#17443](https://github.com/pingcap/tidb/pull/17443) 
- TiDBが一部のケースでPDフォロワーに多くのpingを送信する問題を修正する[#17947](https://github.com/pingcap/tidb/pull/17947) 
- 古いバージョンの範囲テーブルがTiDB v4.0で読み込めない問題を修正する[#17983](https://github.com/pingcap/tidb/pull/17983) 
- 複数のRegion要求が同時に失敗した場合に発生するSQLステートメントのタイムアウト問題を`Backoffer`を各Regionに割り当てることで修正する[#17585](https://github.com/pingcap/tidb/pull/17585) 
- `DateTime`区切り記号の解析時のMySQL非互換な動作を修正する[#17501](https://github.com/pingcap/tidb/pull/17501) 
- TiFlashサーバーへのTiKVリクエストが時折送信される問題を修正する[#18105](https://github.com/pingcap/tidb/pull/18105) 
- 1つのトランザクションで書き込まれ、削除された主キーのロックが他のトランザクションで解決されることによるデータの不整合問題を修正する[#18250](https://github.com/pingcap/tidb/pull/18250) 

+ TiKV

    - ステータスサーバーのメモリセーフティの問題を修正する[#8101](https://github.com/tikv/tikv/pull/8101) 
    - JSON数値の比較における精度の失われた問題を修正する[#8087](https://github.com/tikv/tikv/pull/8087) 
    - 誤ったクエリスローログを修正する[#8050](https://github.com/tikv/tikv/pull/8050) 
    - 複数のマージプロセス中にストアが孤立した場合にピアを削除できない問題を修正する[#8048](https://github.com/tikv/tikv/pull/8048) 
    - `tikv-ctl recover-mvcc`が無効な悲観的ロックを削除しない問題を修正する[#8047](https://github.com/tikv/tikv/pull/8047) 
    - 一部のTitanヒストグラムメトリクスが欠落している問題を修正する[#7997](https://github.com/tikv/tikv/pull/7997) 
    - TiKVがTiCDCに対して`重複エラー`を返す問題を修正する[#7887](https://github.com/tikv/tikv/pull/7887) 

+ PD

    - `pd-server.dashboard-address`構成項目の正確性を確認する[#2517](https://github.com/pingcap/pd/pull/2517) 
    - `store-limit-mode`を`auto`に設定した際のPDのパニック問題を修正する[#2544](https://github.com/pingcap/pd/pull/2544) 
    - 一部のケースでホットスポットを識別できない問題を修正する[#2463](https://github.com/pingcap/pd/pull/2463) 
    - プレースメントルールが一部のケースでストアを`tombstone`に変更できなくなる問題を修正する[#2546](https://github.com/pingcap/pd/pull/2546) 
    - 一部のケースで以前のバージョンからのアップグレード時にPDのパニック問題を修正する[#2564](https://github.com/pingcap/pd/pull/2564) 

+ TiFlash

    - `region not found`エラーが発生した際にプロキシがパニックする問題を修正する 
    - `drop table`で発生したI/O例外がTiFlashスキーマの同期失敗を導く問題を修正する