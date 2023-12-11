---
title: TiDBの機能
summary: TiDBの機能概要を学ぶ。
aliases: ['/docs/dev/basic-features/','/tidb/dev/experimental-features-4.0/']
---

# TiDBの機能

このドキュメントでは、異なるTiDBのバージョンでサポートされている機能をリストしています。これには[長期サポート(LTS)バージョン](/releases/versioning.md#long-term-support-releases)と、最新のLTSバージョンの後にリリースされた[開発マイルストーンリリース(DMR)バージョン](/releases/versioning.md#development-milestone-releases)が含まれます。

TiDBの機能は[TiDB Playground](https://play.tidbcloud.com/?utm_source=docs&utm_medium=tidb_features)で試すことができます。

> **注記:**
>
> PingCAPはDMRバージョンのパッチリリースを提供していません。バグは将来のリリースで修正されます。一般的な目的では、[最新のLTSバージョン](https://docs.pingcap.com/tidb/stable)を使用することをお勧めします。
>
> 下の表での略語は以下の意味を持ちます：
>
> - Y: 機能は一般に利用可能(GA)であり、本番環境で使用することができます。ただし、DMRバージョンで機能がGAであっても、本番環境での機能利用は後のLTSバージョンで推奨されます。
> - N: 機能はサポートされていません。
> - E: 機能はまだ一般に利用可能ではありません(実験的)で、利用する際の制限を理解する必要があります。実験的機能は予告なしに変更または削除される可能性があります。構文や実装は一般利用可能になる前に変更される可能性があります。問題に遭遇した場合、GitHubで[問題](https://github.com/pingcap/tidb/issues)を報告することができます。

## データ型、関数、および演算子

| データ型、関数、および演算子 | 7.4 | 7.3 | 7.2 | 7.1 | 6.5 | 6.1 | 5.4 | 5.3 | 5.2 | 5.1 | 5.0 | 4.0 |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| [数値型](/data-type-numeric.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [日付と時間の型](/data-type-date-and-time.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [文字列型](/data-type-string.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [JSON型](/data-type-json.md) | Y | Y | Y | Y | Y | E | E | E | E | E | E | E |
| [制御フロー関数](/functions-and-operators/control-flow-functions.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [文字列関数](/functions-and-operators/string-functions.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [数値関数および演算子](/functions-and-operators/numeric-functions-and-operators.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [日付と時間の関数](/functions-and-operators/date-and-time-functions.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [ビット関数と演算子](/functions-and-operators/bit-functions-and-operators.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [キャスト関数と演算子](/functions-and-operators/cast-functions-and-operators.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [暗号化および圧縮関数](/functions-and-operators/encryption-and-compression-functions.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [情報関数](/functions-and-operators/information-functions.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [JSON関数](/functions-and-operators/json-functions.md) | Y | Y | Y | Y | Y | E | E | E | E | E | E | E |
| [集約関数](/functions-and-operators/aggregate-group-by-functions.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [ウィンドウ関数](/functions-and-operators/window-functions.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [雑多な関数](/functions-and-operators/miscellaneous-functions.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [演算子](/functions-and-operators/operators.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [文字セットと照合順序](/character-set-and-collation.md) [^1] | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [ユーザーレベルのロック](/functions-and-operators/locking-functions.md) | Y | Y | Y | Y | Y | Y | N | N | N | N | N | N |

## インデックスと制約

| インデックスと制約  | 7.4 | 7.3 | 7.2 | 7.1 | 6.5 | 6.1 | 5.4 | 5.3 | 5.2 | 5.1 | 5.0 | 4.0 |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| [式インデックス](/sql-statements/sql-statement-create-index.md#expression-index) [^2] | Y | Y | Y | Y | Y | E | E | E | E | E | E | E |
| [カラム式ストレージ(TiFlash)](/tiflash/tiflash-overview.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [OLAPシナリオでのクエリ加速のためにFastScanを使用](/tiflash/use-fastscan.md) | Y | Y | Y | Y | E | N | N | N | N | N | N | N |
| [RocksDBエンジン](/storage-engine/rocksdb-overview.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [Titanプラグイン](/storage-engine/titan-overview.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [Titan Level Merge](/storage-engine/titan-configuration.md#level-merge-experimental) | E | E | E | E | E | E | E | E | E | E | E | E |
| [バケットを使ってスキャンの並行性を向上させる](/tune-region-performance.md#use-bucket-to-increase-concurrency) | E | E | E | E | E | E | N | N | N | N | N | N |
| [目に見えないインデックス](/sql-statements/sql-statement-add-index.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | N |
| [複合`PRIMARY KEY`](/constraints.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [`CHECK`制約](/constraints.md#check) | Y | Y | Y | N | N | N | N | N | N | N | N | N |
| [一意のインデックス](/constraints.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [整数`PRIMARY KEY`のためのクラスタ化インデックス](/clustered-indexes.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [複合または非整数キーのクラスタ化インデックス](/clustered-indexes.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | N |
| [多値インデックス](/sql-statements/sql-statement-create-index.md#multi-valued-indexes) | Y | Y | Y | Y | N | N | N | N | N | N | N | N |
| [外部キー](/constraints.md#foreign-key) | E | E | E | E | N | N | N | N | N | N | N | N |
| [TiFlash遅延マテリアライゼーション](/tiflash/tiflash-late-materialization.md) | Y | Y | Y | Y | N | N | N | N | N | N | N | N |

## SQLステートメント

| SQLステートメント [^3] | 7.4 | 7.3 | 7.2 | 7.1 | 6.5 | 6.1 | 5.4 | 5.3 | 5.2 | 5.1 | 5.0 | 4.0 |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 基本的な `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `REPLACE` | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| `INSERT ON DUPLICATE KEY UPDATE` | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| `LOAD DATA INFILE` | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| `SELECT INTO OUTFILE` | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| `INNER JOIN`, LEFT\|RIGHT [OUTER] JOIN | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| `UNION`, `UNION ALL` | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [`EXCEPT` 及び `INTERSECT` オペレーター](/functions-and-operators/set-operators.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | N |
| `GROUP BY`, `ORDER BY` | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [ウィンドウ関数](/functions-and-operators/window-functions.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [共通テーブル式 (CTE)](/sql-statements/sql-statement-with.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | N | N |
| `START TRANSACTION`, `COMMIT`, `ROLLBACK` | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [`EXPLAIN`](/sql-statements/sql-statement-explain.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [`EXPLAIN ANALYZE`](/sql-statements/sql-statement-explain-analyze.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [ユーザー定義変数](/user-defined-variables.md) | E | E | E | E | E | E | E | E | E | E | E | E |
| [`BATCH [ON COLUMN] LIMIT INTEGER DELETE`](/sql-statements/sql-statement-batch.md) | Y | Y | Y | Y | Y | Y | N | N | N | N | N | N |
| [`BATCH [ON COLUMN] LIMIT INTEGER INSERT/UPDATE/REPLACE`](/sql-statements/sql-statement-batch.md) | Y | Y | Y | Y | Y | N | N | N | N | N | N | N |
| [`ALTER TABLE ... COMPACT`](/sql-statements/sql-statement-alter-table-compact.md) | Y | Y | Y | Y | Y | E | N | N | N | N | N | N |
| [テーブルロック](/sql-statements/sql-statement-lock-tables-and-unlock-tables.md) | E | E | E | E | E | E | E | E | E | E | E | E |
| [TiFlashクエリ結果のマテリアライゼーション](/tiflash/tiflash-results-materialization.md) | Y| Y | Y | Y | E | N | N | N | N | N | N | N |

## 高度なSQL機能

| 高度なSQL機能 | 7.4 | 7.3 | 7.2 | 7.1 | 6.5 | 6.1 | 5.4 | 5.3 | 5.2 | 5.1 | 5.0 | 4.0 |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| [準備済みステートメントキャッシュ](/sql-prepared-plan-cache.md) | Y | Y | Y | Y | Y | Y | Y | Y | E | E | E | E |
| [準備されていないステートメントキャッシュ](/sql-non-prepared-plan-cache.md) | Y | E | E | E | N | N | N | N | N | N | N | N |
| [SQLプラン管理 (SPM)](/sql-plan-management.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [過去の実行計画に基づいてバインディングを作成する](/sql-plan-management.md#create-a-binding-according-to-a-historical-execution-plan) | Y | Y | Y | Y | E | N | N | N | N | N | N | N |
| [コプロセッサキャッシュ](/coprocessor-cache.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | E |
| [古いデータの読取り (Stale Read)](/stale-read.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | N | N |
| [フォロワーリード](/follower-read.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [歴史的データの読取り (tidb_snapshot)](/read-historical-data.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [オプティマイザーヒント](/optimizer-hints.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [MPP実行エンジン](/explain-mpp.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | N |
| [MPP実行エンジン - 圧縮データ交換](/explain-mpp.md#mpp-version-and-exchange-data-compression) | Y | Y | Y | Y | N | N | N | N | N | N | N | N |
| [TiFlash Pipelineモデル](/tiflash/tiflash-pipeline-model.md) | Y | E | E | N | N | N | N | N | N | N | N | N |
| [TiFlashレプリカ選択戦略](/system-variables.md#tiflash_replica_read-new-in-v730) | Y | Y | N | N | N | N | N | N | N | N | N | N |
| [Index Merge](/explain-index-merge.md) | Y | Y | Y | Y | Y | Y | Y | E | E | E | E | E |
| [SQL内の配置ルール](/placement-rules-in-sql.md) | Y | Y | Y | Y | Y | Y | E | E | N | N | N | N |
| [カスケードプランナー](/system-variables.md#tidb_enable_cascades_planner) | E | E | E | E | E | E | E | E | E | E | E | E |
| [ランタイムフィルタ](/runtime-filter.md) | Y | Y | N | N | N | N | N | N | N | N | N | N |

## データ定義言語 (DDL)

| データ定義言語 (DDL) | 7.4 | 7.3 | 7.2 | 7.1 | 6.5 | 6.1 | 5.4 | 5.3 | 5.2 | 5.1 | 5.0 | 4.0 |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 基本的な `CREATE`, `DROP`, `ALTER`, `RENAME`, `TRUNCATE` | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [生成列](/generated-columns.md) | Y | Y | Y | Y | E | E | E | E | E | E | E | E |
| [ビュー](/views.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
```
| [シーケンス](/sql-statements/sql-statement-create-sequence.md)                    | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [オートインクリメント](/auto-increment.md)                                            | Y | Y | Y | Y | Y[^4] | Y | Y | Y | Y | Y | Y | Y |
| [オートランダム](/auto-random.md)                                                      | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [TTL (タイム・トゥ・リブ)](/time-to-live.md)                                     | Y | Y | Y | Y | E | N | N | N | N | N | N | N |
| [DDLアルゴリズムのアサーション](/sql-statements/sql-statement-alter-table.md)       | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| マルチスキーマ変更：カラム追加                                                       | Y | Y | Y | Y | Y | E | E | E | E | E | E | E |
| [カラムタイプ変更](/sql-statements/sql-statement-modify-column.md)                     | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | N | N |
| [一時テーブル](/temporary-tables.md)                                                  | Y | Y | Y | Y | Y | Y | Y | Y | N | N | N | N |
| 並行DDLステートメント                                                                 | Y | Y | Y | Y | Y | N | N | N | N | N | N | N |
| [`ADD INDEX`と`CREATE INDEX`の加速](/system-variables.md#tidb_ddl_enable_fast_reorg-new-in-v630) | Y | Y | Y | Y | Y | N | N | N | N | N | N | N |
| [メタデータロック](/metadata-lock.md)                                                | Y | Y | Y | Y | Y | N | N | N | N | N | N | N |
| [`FLASHBACK CLUSTER TO TIMESTAMP`](/sql-statements/sql-statement-flashback-to-timestamp.md) | Y | Y | Y | Y | Y | N | N | N | N | N | N | N |
| [DDLの一時停止](/sql-statements/sql-statement-admin-pause-ddl.md)/[再開](/sql-statements/sql-statement-admin-resume-ddl.md) | E | E | E | N | N | N | N | N | N | N | N | N |

## トランザクション

| トランザクション                                                               | 7.4 | 7.3 | 7.2 | 7.1 | 6.5 | 6.1 | 5.4 | 5.3 | 5.2 | 5.1 | 5.0 | 4.0 |
|----------------------------------------------------------------------------------|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| [非同期コミット](/system-variables.md#tidb_enable_async_commit-new-in-v50)         | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | N |
| [1PC](/system-variables.md#tidb_enable_1pc-new-in-v50)                               | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | N |
| [大規模トランザクション（10GB）](/transaction-overview.md#transaction-size-limit)  | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [悲観的トランザクション](/pessimistic-transaction.md)                              | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [楽観的トランザクション](/optimistic-transaction.md)                               | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [リピータブルリード分離（スナップショット分離）](/transaction-isolation-levels.md)    | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [リードコミット分離](/transaction-isolation-levels.md)                              | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |

## パーティショニング

| パーティショニング                                                              | 7.4 | 7.3 | 7.2 | 7.1 | 6.5 | 6.1 | 5.4 | 5.3 | 5.2 | 5.1 | 5.0 | 4.0 |
|----------------------------------------------------------------------------------|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| [レンジパーティショニング](/partitioned-table.md#range-partitioning)                 | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [ハッシュパーティショニング](/partitioned-table.md#hash-partitioning)              | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [キーパーティショニング](/partitioned-table.md#key-partitioning)                  | Y | Y | Y | Y | N | N | N | N | N | N | N | N |
| [リストパーティショニング](/partitioned-table.md#list-partitioning)                 | Y | Y | Y | Y | Y | Y | E | E | E | E | E | N |
| [リストCOLUMNSパーティショニング](/partitioned-table.md)                          | Y | Y | Y | Y | Y | Y | E | E | E | E | E | N |
| [リストとリストCOLUMNSパーティショニングテーブルのデフォルトパーティション](/partitioned-table.md#default-list-partition)  | Y | Y | N | N | N | N | N | N | N | N | N | N |
| [`EXCHANGE PARTITION`](/partitioned-table.md)                                      | Y | Y | Y | Y | Y | E | E | E | E | E | E | N |
| [`REORGANIZE PARTITION`](/partitioned-table.md#reorganize-partitions)             | Y | Y | Y | Y | N | N | N | N | N | N | N | N |
| [`COALESCE PARTITION`](/partitioned-table.md#decrease-the-number-of-partitions)    | Y | Y | Y | Y | N | N | N | N | N | N | N | N |
| [ダイナミックプルーニング](/partitioned-table.md#dynamic-pruning-mode)           | Y | Y | Y | Y | Y | Y | E | E | E | E | N | N |
| [レンジCOLUMNSパーティショニング](/partitioned-table.md#range-columns-partitioning) | Y | Y | Y | Y | Y | N | N | N | N | N | N | N |
| [レンジINTERVALパーティショニング](/partitioned-table.md#range-interval-partitioning) | Y | Y | Y | Y | E | N | N | N | N | N | N | N |
| [パーティショニングされたテーブルを非パーティショニングテーブルに変換](/partitioned-table.md#convert-a-partitioned-table-to-a-non-partitioned-table) | Y | N | N | N | N | N | N | N | N | N | N | N |
| [既存のテーブルのパーティショニング](/partitioned-table.md#partition-an-existing-table)  | Y | N | N | N | N | N | N | N | N | N | N | N |

## 統計情報

| 統計情報                                                                        | 7.4 | 7.3 | 7.2 | 7.1 | 6.5 | 6.1 | 6.0 | 5.4 | 5.3 | 5.2 | 5.1 | 5.0 |
|----------------------------------------------------------------------------------|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| [CMSketch](/statistics.md)                                                      | デフォルトで無効 | デフォルトで無効 | デフォルトで無効 | デフォルトで無効 | デフォルトで無効 | デフォルトで無効 | デフォルトで無効 | デフォルトで無効 | デフォルトで無効 | Y | Y | Y |
| [ヒストグラム](/statistics.md)                                                  | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [拡張統計情報](/extended-statistics.md)                                        | E | E | E | E | E | E | E | E | E | E | E | E |
| 統計情報フィードバック                                                          | N | N | N | N | N | 廃止 | 廃止 | 廃止 | E | E | E | E |
| [統計情報の自動更新](/statistics.md#automatic-update)                           | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
```
```
| [高速解析](/system-variables.md#tidb_enable_fast_analyze) | E | E | E | E | E | E | E | E | E | E | E | E |
| [動的プルーニング](/partitioned-table.md#dynamic-pruning-mode) | Y | Y | Y | Y | Y | Y | E | E | E | E | E | N |
| [`PREDICATE COLUMNS`の統計収集](/statistics.md#collect-statistics-on-some-columns) | E | E | E | E | E | E | E | E | N | N | N | N |
| [統計情報収集のためのメモリ割り当て制御](/statistics.md#the-memory-quota-for-collecting-statistics) | E | E | E | E | E | E | N | N | N | N | N | N |
| [約10000行のデータをランダムサンプリングして迅速に統計を構築する](/system-variables.md#tidb_enable_fast_analyze) | E | E | E | E | E | E | E | E | E | E | E | E |
| [統計情報のロック](/statistics.md#lock-statistics) | E | E | E | E | E | N | N | N | N | N | N | N |
| [ライトウェイト統計情報の初期化](/statistics.md#load-statistics) | Y | Y | Y | E | N | N | N | N | N | N | N | N |
| [統計情報収集の進捗表示](/sql-statements/sql-statement-show-analyze-status.md) | Y | Y | N | N | N | N | N | N | N | N | N | N |

## セキュリティ

| セキュリティ | 7.4 | 7.3 | 7.2 | 7.1 | 6.5 | 6.1 | 5.4 | 5.3 | 5.2 | 5.1 | 5.0 | 4.0 |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| [透過的レイヤーセキュリティ（TLS）](/enable-tls-between-clients-and-servers.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [レスト時の暗号化（TDE）](/encryption-at-rest.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [ロールベース認証（RBAC）](/role-based-access-control.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [証明書ベース認証](/certificate-authentication.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [`caching_sha2_password`認証](/system-variables.md#default_authentication_plugin) | Y | Y | Y | Y | Y | Y | Y | Y | Y | N | N | N |
| [`tidb_sm3_password`認証](/system-variables.md#default_authentication_plugin) | Y | Y | Y | Y | Y | N | N | N | N | N | N | N |
| [`tidb_auth_token`認証](/system-variables.md#default_authentication_plugin) | Y | Y | Y | Y | Y | N | N | N | N | N | N | N |
| [`authentication_ldap_sasl`認証](/system-variables.md#default_authentication_plugin) | Y | Y | Y | N | N | N | N | N | N | N | N | N |
| [`authentication_ldap_simple`認証](/system-variables.md#default_authentication_plugin) | Y | Y | Y | Y | N | N | N | N | N | N | N | N |
| [パスワード管理](/password-management.md) | Y | Y | Y | Y | Y | N | N | N | N | N | N | N |
| [MySQL互換の`GRANT`システム](/privilege-management.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [動的権限](/privilege-management.md#dynamic-privileges) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | N | N |
| [セキュリティ強化モード](/system-variables.md#tidb_enable_enhanced_security) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | N | N |
| [赤塗り処理されたログファイル](/log-redaction.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | N |

## データのインポートとエクスポート

| データのインポートとエクスポート | 7.4 | 7.3 | 7.2 | 7.1 | 6.5 | 6.1 | 5.4 | 5.3 | 5.2 | 5.1 | 5.0 | 4.0 |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| [TiDB Lightningを用いた高速インポート](/tidb-lightning/tidb-lightning-overview.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [`IMPORT INTO` ステートメントを用いた高速インポート](/sql-statements/sql-statement-import-into.md) | E | E | E | N | N | N | N | N | N | N | N | N |
| mydumperロジカルダンパー | 廃止予定 | 廃止予定 | 廃止予定 | 廃止予定 | 廃止予定 | 廃止予定 | 廃止予定 | 廃止予定 | 廃止予定 | 廃止予定 | 廃止予定 | 廃止予定 |
| [Dumplingロジカルダンパー](/dumpling-overview.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [トランザクショナル `LOAD DATA`](/sql-statements/sql-statement-load-data.md) [^5] | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | N [^6] |
| [データベースマイグレーショントールキット（DM）](/migration-overview.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [TiDB Binlog](/tidb-binlog/tidb-binlog-overview.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [チェンジデータキャプチャ（CDC）](/ticdc/ticdc-overview.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [TiCDCを通じてAmazon S3, GCS, Azure Blob Storage, NFSへのストリームデータ](/ticdc/ticdc-sink-to-cloud-storage.md) | Y | Y | Y | Y | E | N | N | N | N | N | N | N |
| [TiCDCを用いた二つのTiDBクラスタ間の双方向レプリケーションのサポート](/ticdc/ticdc-bidirectional-replication.md) | Y | Y | Y | Y | Y | N | N | N | N | N | N | N |
| [TiCDC OpenAPI v2](/ticdc/ticdc-open-api-v2.md) | Y | Y | Y | Y | N | N | N | N | N | N | N | N |

## 管理、オブザーバビリティ、そしてツール

| 管理、オブザーバビリティ、そしてツール | 7.4 | 7.3 | 7.2 | 7.1 | 6.5 | 6.1 | 5.4 | 5.3 | 5.2 | 5.1 | 5.0 | 4.0 |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| [TiDB Dashboard UI](/dashboard/dashboard-intro.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [TiDB Dashboard 連続プロファイリング](/dashboard/continuous-profiling.md) | Y | Y | Y | Y | Y | Y | E | E | N | N | N | N |
| [TiDB Dashboard Top SQL](/dashboard/top-sql.md) | Y | Y | Y | Y | Y | Y | E | N | N | N | N | N |
| [TiDB Dashboard SQL診断](/information-schema/information-schema-sql-diagnostics.md) | Y | Y | Y | Y | Y | E | E | E | E | E | E | E |
| [TiDB Dashboardクラスタ診断](/dashboard/dashboard-diagnostics-access.md) | Y | Y | Y | Y | Y | E | E | E | E | E | E | E |
```
| [TiKV-FastTune ダッシュボード](/grafana-tikv-dashboard.md#tikv-fasttune-dashboard) | E | E | E | E | E | E | E | E | E | E | E | E |
| [インフォメーション スキーマ](/information-schema/information-schema.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [メトリクス スキーマ](/metrics-schema.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [ステートメントサマリーテーブル](/statement-summary-tables.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [ステートメントサマリーテーブル - サマリー永続化](/statement-summary-tables.md#persist-statements-summary) | E | E | E | E | N | N | N | N | N | N | N | N |
| [スロークエリログ](/identify-slow-queries.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [TiUP デプロイメント](/tiup/tiup-overview.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [Kubernetes オペレーター](https://docs.pingcap.com/tidb-in-kubernetes/) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [内蔵フィジカルバックアップ](/br/backup-and-restore-use-cases.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [Global Kill](/sql-statements/sql-statement-kill.md) | Y | Y | Y | Y | Y | Y | E | E | E | E | E | E |
| [ロックビュー](/information-schema/information-schema-data-lock-waits.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | E | E | E |
| [`SHOW CONFIG`](/sql-statements/sql-statement-show-config.md) | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| [`SET CONFIG`](/dynamic-config.md) | Y | Y | Y | Y | Y | Y | E | E | E | E | E | E |
| [DM WebUI](/dm/dm-webui-guide.md) | E | E | E | E | E | E | N | N | N | N | N | N |
| [フォアグラウンド クォータ リミッター](/tikv-configuration-file.md#foreground-quota-limiter) | Y | Y | Y | Y | Y | E | N | N | N | N | N | N |
| [バックグラウンド クォータ リミッター](/tikv-configuration-file.md#background-quota-limiter) | E | E | E | E | E | N | N | N | N | N | N | N |
| [EBS ボリューム スナップショットのバックアップとリストア](https://docs.pingcap.com/tidb-in-kubernetes/v1.4/backup-to-aws-s3-by-snapshot) | Y | Y | Y | Y | Y | N | N | N | N | N | N | N |
| [PITR](/br/backup-and-restore-overview.md) | Y | Y | Y | Y | Y | N | N | N | N | N | N | N |
| [グローバル メモリ制御](/configure-memory-usage.md#configure-the-memory-usage-threshold-of-a-tidb-server-instance) | Y | Y | Y | Y | Y | N | N | N | N | N | N | N |
| [クロスクラスタ RawKV レプリケーション](/tikv-configuration-file.md#api-version-new-in-v610) | E | E | E | E | E | N | N | N | N | N | N | N |
| [グリーン GC](/system-variables.md#tidb_gc_scan_lock_mode-new-in-v50) | E | E | E | E | E | E | E | E | E | E | E | N |
| [リソース制御](/tidb-resource-control.md) | Y | Y | Y | Y | N | N | N | N | N | N | N | N |
| [制御を逃れるクエリの管理](/tidb-resource-control.md#manage-queries-that-consume-more-resources-than-expected-runaway-queries) | E | E | E | N | N | N | N | N | N | N | N | N |
| [バックグラウンドタスクの管理](/tidb-resource-control.md#manage-background-tasks) | E | N | N | N | N | N | N | N | N | N | N | N |
| [TiFlash 分散ストレージとコンピューティングアーキテクチャと S3 サポート](/tiflash/tiflash-disaggregated-and-s3.md) | Y | E | E | E | N | N | N | N | N | N | N | N |
| [分散フレームワークタスクに TiDB ノードを選択する](/system-variables.md#tidb_service_scope-new-in-v740) | E | N | N | N | N | N | N | N | N | N | N | N |

[^1]: TiDBは、latin1をutf8のサブセットとして誤って扱います。詳細については[TiDB #18955](https://github.com/pingcap/tidb/issues/18955)を参照してください。

[^2]: v6.5.0から、[`tidb_allow_function_for_expression_index`](/system-variables.md#tidb_allow_function_for_expression_index-new-in-v520)システム変数によってリストされた関数に対して作成された式インデックスはテスト済みであり、本番環境で使用できるようになりました。今後のリリースでより多くの関数がサポートされる予定です。この変数によってリストされていない関数に対する対応する式インデックスは、本番環境での使用は推奨されません。詳細は[式インデックス](/sql-statements/sql-statement-create-index.md#expression-index)を参照してください。

[^3]: サポートされているSQLステートメントの完全なリストについては、[ステートメントリファレンス](/sql-statements/sql-statement-select.md)を参照してください。

[^4]: [v6.4.0](/releases/release-6.4.0.md)から、TiDBは[高性能で大域的に単調な`AUTO_INCREMENT`カラム](/auto-increment.md#mysql-compatibility-mode)をサポートしています。

[^5]: [TiDB v7.0.0](/releases/release-7.0.0.md)から、新しいパラメータ`FIELDS DEFINED NULL BY`とS3やGCSからのデータのインポートのサポートは実験的機能です。

[^6]: TiDB v4.0において、`LOAD DATA`トランザクションは原子性を保証しません。