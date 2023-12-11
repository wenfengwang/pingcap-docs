---
title: TiDB 2.1 GAリリースノート
aliases: ['/docs/dev/releases/release-2.1-ga/','/docs/dev/releases/2.1ga/']
---

# TiDB 2.1 GAリリースノート

2018年11月30日、TiDB 2.1 GAがリリースされました。このリリースでのアップデート内容を以下に示します。TiDB 2.0と比較して、このリリースでは安定性、パフォーマンス、互換性、および使いやすさに大幅な改善が加えられています。

## TiDB

+ SQLオプティマイザ

    - `Index Join`の選択範囲を最適化し、実行パフォーマンスを向上させる

    - `Index Join`のアウターテーブルの選択を最適化し、行数見積もりの値が小さいテーブルをアウターテーブルとして使用する

    - Join Hintの`TIDB_SMJ`を最適化し、適切なインデックスが利用できなくてもMerge Joinを使用できるようにする

    - Join Hintの`TIDB_INLJ`を最適化し、Joinするインナーテーブルを指定する

    - 相関サブクエリを最適化し、Filterをプッシュダウンし、インデックスの選択範囲を拡張して、いくつかのクエリの効率を桁違いに向上させる

    - `UPDATE`および`DELETE`文でIndex HintとJoin Hintを使用するサポート

    - `ABS`/`CEIL`/`FLOOR`/`IS TRUE`/`IS FALSE`のような関数をプッシュダウンするサポート

    - `IF`および`IFNULL`組み込み関数のための定数折りたたみアルゴリズムを最適化

    - `EXPLAIN`文の出力を最適化し、演算子間の関係を示す階層構造を使用

+ SQL実行エンジン

    - すべての集約関数をリファクタリングし、`Stream`および`Hash`集約演算子の実行効率を向上させる

    - 並列`Hash Aggregate`演算子を実装し、一部のシナリオで計算パフォーマンスを350%向上させる

    - 並列`Project`演算子を実装し、一部のシナリオでパフォーマンスを74%向上させる

    - `Hash Join`のインナーテーブルとアウターテーブルのデータを同時に読み取り、実行パフォーマンスを向上させる

    - `REPLACE INTO`文の実行スピードを最適化し、パフォーマンスをほぼ10倍向上させる

    - 日付型のメモリ使用量を最適化し、メモリ使用量を50%減少させる

    - ポイント選択のパフォーマンスを最適化し、Sysbenchのポイント選択効率を60%向上させる

    - ワイドテーブルの挿入や更新時にTiDBのパフォーマンスを20倍向上させる

    - 設定ファイルで単一のステートメントのメモリ上限を設定するサポート

    - Hash Joinの実行を最適化し、Joinの種類がInner JoinまたはSemi Joinであり、インナーテーブルが空である場合、アウターテーブルからデータを読み取らずに結果を返す

    - `EXPLAIN ANALYZE`文を使用して実行時間や各演算子の返された行数など、実行時統計を確認するサポート

+ 統計情報

    - 特定の時間帯だけ自動的にANALYZE統計情報を有効にするサポート

    - クエリのフィードバックに基づいて表の統計情報を自動的に更新するサポート

    - `ANALYZE TABLE WITH BUCKETS`文を使用してヒストグラムのバケット数を構成するサポート

    - ヒストグラムを使用した等価クエリと範囲クエリの混合クエリのRow Count推定アルゴリズムを最適化

+ 式

    + 次の組み込み関数をサポート:

        - `json_contains`

        - `json_contains_path`

        - `encode/decode`

+ サーバ

    - ローカルで競合するトランザクションをtidb-serverインスタンス内でキューイングすることで、競合するトランザクションのパフォーマンスを最適化するサポート

    - サーバサイドカーソルのサポート

    + [HTTP API](https://github.com/pingcap/tidb/blob/master/docs/tidb_http_api.md)の追加

        - TiKVクラスタ内のテーブルリージョンの分布を散らす

        - `general log`をオープンするかどうかを制御する

        - ログレベルをオンラインで変更するサポート

        - TiDBクラスタ情報の確認

    - `auto_analyze_ratio`システム変数を追加して、Analyzeの比率を制御する

    - `tidb_retry_limit`システム変数を追加して、トランザクションの自動リトライ回数を制御する

    - `tidb_disable_txn_auto_retry`システム変数を追加して、トランザクションの自動リトライを制御する

    - `admin show slow`文を使用して遅いクエリを取得するサポート

    - `tidb_slow_log_threshold`環境変数を追加して、遅いログのしきい値を自動的に設定する

    - `tidb_query_log_max_len`環境変数を追加して、ログで切り捨てるSQLステートメントの長さを動的に設定する

+ DDL

    - `ADD INDEX`文や他の文の並列実行をサポートして、時間のかかるAdd index操作が他の操作をブロックしないようにする

    - `ADD INDEX`の実行スピードを最適化し、一部のシナリオで大幅に向上させる

    - `tidb_is_ddl_owner()`文をサポートして、TiDBが`DDL Owner`かどうかを判断するのを容易にする

    - `ALTER TABLE FORCE`構文をサポート

    - `ALTER TABLE RENAME KEY TO`構文をサポート

    - `admin show ddl jobs`の出力情報にテーブル名とデータベース名を追加

    - `ddl/owner/resign` HTTPインターフェースを使用してDDLオーナーを解放し、新しいDDLオーナーの選出を開始するサポート

+ 互換性

    - より多くのMySQL構文をサポート

    - `BIT`集約関数が`ALL`パラメータをサポート

    - `SHOW PRIVILEGES`文をサポート

    - `LOAD DATA`文での`CHARACTER SET`構文をサポート

    - `CREATE USER`文での`IDENTIFIED WITH`構文をサポート

    - `LOAD DATA IGNORE LINES`文をサポート

    - `Show ProcessList`文がより正確な情報を返す

## プレースメントドライバ (PD)

+ 可用性の最適化

    - バージョン管理メカニズムを導入し、クラスタの互換性を保ったままローリングアップデートをサポート

    - ネットワーク隔離が解除された後のリーダーの再選出を回避するために、PDノード間で`Raft PreVote`を有効にする

    - マシンの障害によるデータの利用不能リスクを低減するために、`raft learner`をデフォルトで有効にする

    - TSOの割り当てはもはやシステムクロックが後ろに戻った影響を受けない

    - メタデータによるオーバーヘッドを減らすために、`Region merge`機能をサポート

+ スケジューラの最適化

    - ダウンストアの処理を最適化してレプリカの補充を高速化する

    - トラフィック統計情報が揺れた場合にその適応性を向上させるためにホットスポットスケジューラを最適化

    - Coordinatorのスタートを最適化してPDの再起動による不必要なスケジューリングを減らす

    - Balance Schedulerが小さなリージョンを頻繁にスケジュールする問題を最適化

    - リージョンマージを行う際にリージョン内の行数を考慮するように最適化

    - スケジュールポリシーを制御するためのコマンドを追加

    - スケジューリングシナリオをシミュレートするためのPDシミュレータを改良

+ APIおよび操作ツール

    - `GetPrevRegion`インターフェースを追加して`TiDB逆スキャン`機能をサポート

    - `BatchSplitRegion`インターフェースを追加してTiKVリージョンの分割を高速化

    - 分散GCをサポートするための`GCSafePoint`インターフェースを追加

    - 分散GCをサポートするための`GetAllStores`インターフェースを追加

    + pd-ctlで以下をサポート:

        - リージョン分割のための統計の使用


        - [calling `jq` to format the JSON output](/pd-control.ja#jq-formatted-json-output-usage)

        - [checking the Region information of the specified store](/pd-control.ja#region-store-store_id)

        - [checking topN Region list sorted by versions](/pd-control.ja#region-topconfver-limit)

        - [checking topN Region list sorted by size](/pd-control.ja#region-topsize-limit)

        - [more precise TSO encoding](/pd-control.ja#tso)

    - [pd-recover](/pd-recover.ja) doesn't need to provide the `max-replica` parameter

+ Metrics

    - Add related metrics for `Filter`

    - Add metrics about etcd Raft state machine

+ Performance

    - Optimize the performance of Region heartbeat to reduce the memory overhead brought by heartbeats

    - Optimize the Region tree performance

    - Optimize the performance of computing hotspot statistics

## TiKV

+ Coprocessor

    - Add more built-in functions

    - [Add Coprocessor `ReadPool` to improve the concurrency in processing the requests](https://github.com/tikv/rfcs/blob/master/text/0010-read-pool.ja)

    - Fix the time function parsing issue and the time zone related issues

    - Optimize the memory usage for pushdown aggregation computing

+ Transaction

    - Optimize the read logic and memory usage of MVCC to improve the performance of the scan operation and the performance of full table scan is 1 time better than that in TiDB 2.0

    - Fold the continuous Rollback records to ensure the read performance

    - [Add the `UnsafeDestroyRange` API to support to collecting space for the dropping table/index](https://github.com/tikv/rfcs/blob/master/text/0002-unsafe-destroy-range.ja)

    - Separate the GC module to reduce the impact on write

    - Add the `upper bound` support in the `kv_scan` command

+ Raftstore

    - Improve the snapshot writing process to avoid RocksDB stall

    - [Add the `LocalReader` thread to process read requests and reduce the delay for read requests](https://github.com/tikv/rfcs/pull/17.ja)

    - [Support `BatchSplit` to avoid large Region brought by large amounts of write](https://github.com/tikv/rfcs/pull/6.ja)

    - Support `Region Split` according to statistics to reduce the I/O overhead

    - Support `Region Split` according to the number of keys to improve the concurrency of index scan

    - Improve the Raft message process to avoid unnecessary delay brought by `Region Split`

    - Enable the `PreVote` feature by default to reduce the impact of network isolation on services

+ Storage Engine

    - Fix the `CompactFiles`bug in RocksDB and reduce the impact on importing data using Lightning

    - Upgrade RocksDB to v5.15 to fix the possible issue of snapshot file corruption

    - Improve `IngestExternalFile` to avoid the issue that flush could block write

+ tikv-ctl

    - [Add the `ldb` command to diagnose RocksDB related issues](https://tikv.org/docs/3.0/reference/tools/tikv-ctl/#ldb-command.ja)

    - The `compact` command supports specifying whether to compact data in the bottommost level

## Tools

- Fast full import of large amounts of data: [TiDB Lightning](/tidb-lightning/tidb-lightning-overview.ja)

- Support new [TiDB Binlog](/tidb-binlog/tidb-binlog-overview.ja)

## Upgrade caveat

- TiDB 2.1 does not support downgrading to v2.0.x or earlier due to the adoption of the new storage engine

+ Parallel DDL is enabled in TiDB 2.1, so the clusters with TiDB version earlier than 2.0.1 cannot upgrade to 2.1 using rolling update. You can choose either of the following two options:

    - Stop the cluster and upgrade to 2.1 directly
    - Roll update to 2.0.1 or later 2.0.x versions, and then roll update to the 2.1 version

- If you upgrade from TiDB 2.0.6 or earlier to TiDB 2.1, check if there is any ongoing DDL operation, especially the time consuming `Add Index` operation, because the DDL operations slow down the upgrading process. If there is ongoing DDL operation, wait for the DDL operation finishes and then roll update.