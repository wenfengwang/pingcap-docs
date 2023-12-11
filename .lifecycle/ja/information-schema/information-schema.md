---
title: インフォメーション・スキーマ
summary: TiDBはシステムメタデータを表示するためのANSI標準のinformation_schemaを実装しています。
aliases: ['/docs/dev/system-tables/system-table-information-schema/','/docs/dev/reference/system-databases/information-schema/','/tidb/dev/system-table-information-schema/']
---

# インフォメーション・スキーマ

インフォメーション・スキーマは、システムメタデータを表示するためのANSI標準の方法を提供します。 TiDBは、MySQL互換性のために含まれているテーブルに加えて、複数のカスタム `INFORMATION_SCHEMA` テーブルを提供します。

多くの `INFORMATION_SCHEMA` テーブルには、対応する `SHOW` コマンドがあります。 `INFORMATION_SCHEMA` をクエリする利点は、テーブル間での結合が可能であることです。

## MySQL互換性のためのテーブル

<CustomContent platform="tidb">

| テーブル名 | 説明 |
|--------------------|----------------------|
| [`CHARACTER_SETS`](/information-schema/information-schema-character-sets.md) | サーバーがサポートしている文字セットのリストを提供します。 |
| [`CHECK_CONSTRAINTS`](/information-schema/information-schema-check-constraints.md) | テーブルの[`CHECK`制約](/constraints.md#check)についての情報を提供します。 |
| [`COLLATIONS`](/information-schema/information-schema-collations.md) | サーバーがサポートしている照合順序のリストを提供します。 |
| [`COLLATION_CHARACTER_SET_APPLICABILITY`](/information-schema/information-schema-collation-character-set-applicability.md) | どの照合順序がどの文字セットに適用されるかを説明します。 |
| [`COLUMNS`](/information-schema/information-schema-columns.md) | すべてのテーブルの列のリストを提供します。 |
| `COLUMN_PRIVILEGES` | TiDBでは実装されていません。ゼロ行を返します。 |
| `COLUMN_STATISTICS` | TiDBでは実装されていません。ゼロ行を返します。 |
| [`ENGINES`](/information-schema/information-schema-engines.md) | サポートされているストレージエンジンのリストを提供します。 |
| `EVENTS` | TiDBでは実装されていません。ゼロ行を返します。 |
| `FILES` | TiDBでは実装されていません。ゼロ行を返します。 |
| `GLOBAL_STATUS` | TiDBでは実装されていません。ゼロ行を返します。 |
| `GLOBAL_VARIABLES` | TiDBでは実装されていません。ゼロ行を返します。 |
| [`KEY_COLUMN_USAGE`](/information-schema/information-schema-key-column-usage.md) | 列のキー制約（プライマリキー制約など）についての情報を記述します。 |
| `OPTIMIZER_TRACE` | TiDBでは実装されていません。ゼロ行を返します。 |
| `PARAMETERS` | TiDBでは実装されていません。ゼロ行を返します。 |
| [`PARTITIONS`](/information-schema/information-schema-partitions.md) | テーブルのパーティションの一覧を提供します。 |
| `PLUGINS` | TiDBでは実装されていません。ゼロ行を返します。 |
| [`PROCESSLIST`](/information-schema/information-schema-processlist.md) | コマンド `SHOW PROCESSLIST` に類似した情報を提供します。 |
| `PROFILING` | TiDBでは実装されていません。ゼロ行を返します。 |
| `REFERENTIAL_CONSTRAINTS` | `FOREIGN KEY` 制約に関する情報を提供します。 |
| `ROUTINES` | TiDBでは実装されていません。ゼロ行を返します。 |
| [`SCHEMATA`](/information-schema/information-schema-schemata.md) | `SHOW DATABASES` に類似した情報を提供します。 |
| `SCHEMA_PRIVILEGES` | TiDBでは実装されていません。ゼロ行を返します。 |
| `SESSION_STATUS` | TiDBでは実装されていません。ゼロ行を返します。 |
| [`SESSION_VARIABLES`](/information-schema/information-schema-session-variables.md) | コマンド `SHOW SESSION VARIABLES` に類似した機能を提供します。 |
| [`STATISTICS`](/information-schema/information-schema-statistics.md) | テーブルのインデックスに関する情報を提供します。 |
| [`TABLES`](/information-schema/information-schema-tables.md) | 現在のユーザーが表示可能なテーブルの一覧を提供します。`SHOW TABLES` と類似しています。 |
| `TABLESPACES` | TiDBでは実装されていません。ゼロ行を返します。 |
| [`TABLE_CONSTRAINTS`](/information-schema/information-schema-table-constraints.md) | プライマリキー、一意なインデックス、外部キーに関する情報を提供します。 |
| `TABLE_PRIVILEGES` | TiDBでは実装されていません。ゼロ行を返します。 |
| `TRIGGERS` | TiDBでは実装されていません。ゼロ行を返します。 |
| [`USER_ATTRIBUTES`](/information-schema/information-schema-user-attributes.md) | ユーザーコメントやユーザー属性に関する情報をまとめます。 |
| [`USER_PRIVILEGES`](/information-schema/information-schema-user-privileges.md) | 現在のユーザーに関連付けられた特権をまとめます。 |
| [`VARIABLES_INFO`](/information-schema/information-schema-variables-info.md) | TiDBシステム変数に関する情報を提供します。 |
| [`VIEWS`](/information-schema/information-schema-views.md) | 現在のユーザーが表示可能なビューの一覧を提供します。`SHOW FULL TABLES WHERE table_type = 'VIEW'` を実行したのと同様です。 |

</CustomContent>

<CustomContent platform="tidb-cloud">

| テーブル名 | 説明 |
|--------------------|----------------------|
| [`CHARACTER_SETS`](/information-schema/information-schema-character-sets.md) | サーバーがサポートしている文字セットのリストを提供します。 |
| [`CHECK_CONSTRAINTS`](/information-schema/information-schema-check-constraints.md) | テーブルの[`CHECK`制約](/constraints.md#check)についての情報を提供します。 |
| [`COLLATIONS`](/information-schema/information-schema-collations.md) | サーバーがサポートしている照合順序のリストを提供します。 |
| [`COLLATION_CHARACTER_SET_APPLICABILITY`](/information-schema/information-schema-collation-character-set-applicability.md) | どの照合順序がどの文字セットに適用されるかを説明します。 |
| [`COLUMNS`](/information-schema/information-schema-columns.md) | すべてのテーブルの列のリストを提供します。 |
| `COLUMN_PRIVILEGES` | TiDBでは実装されていません。ゼロ行を返します。 |
| `COLUMN_STATISTICS` | TiDBでは実装されていません。ゼロ行を返します。 |
| [`ENGINES`](/information-schema/information-schema-engines.md) | サポートされているストレージエンジンのリストを提供します。 |
| `EVENTS` | TiDBでは実装されていません。ゼロ行を返します。 |
| `FILES` | TiDBでは実装されていません。ゼロ行を返します。 |
| `GLOBAL_STATUS` | TiDBでは実装されていません。ゼロ行を返します。 |
| `GLOBAL_VARIABLES` | TiDBでは実装されていません。ゼロ行を返します。 |
| [`KEY_COLUMN_USAGE`](/information-schema/information-schema-key-column-usage.md) | 列のキー制約（プライマリキー制約など）についての情報を記述します。 |
| `OPTIMIZER_TRACE` | TiDBでは実装されていません。ゼロ行を返します。 |
| `PARAMETERS` | TiDBでは実装されていません。ゼロ行を返します。 |
| [`PARTITIONS`](/information-schema/information-schema-partitions.md) | テーブルのパーティションの一覧を提供します。 |
| `PLUGINS` | TiDBでは実装されていません。ゼロ行を返します。 |
| [`PROCESSLIST`](/information-schema/information-schema-processlist.md) | コマンド `SHOW PROCESSLIST` に類似した情報を提供します。 |
| `PROFILING` | TiDBでは実装されていません。ゼロ行を返します。 |
| `REFERENTIAL_CONSTRAINTS` | `FOREIGN KEY` 制約に関する情報を提供します。 |
| `ROUTINES` | TiDBでは実装されていません。ゼロ行を返します。 |
| [`SCHEMATA`](/information-schema/information-schema-schemata.md) | `SHOW DATABASES` に類似した情報を提供します。 |
| `SCHEMA_PRIVILEGES` | TiDBでは実装されていません。ゼロ行を返します。 |
| `SESSION_STATUS` | TiDBでは実装されていません。ゼロ行を返します。 |
| [`SESSION_VARIABLES`](/information-schema/information-schema-session-variables.md) | コマンド `SHOW SESSION VARIABLES` に類似した機能を提供します。 |
| [`STATISTICS`](/information-schema/information-schema-statistics.md) | テーブルのインデックスに関する情報を提供します。 |
| [`TABLES`](/information-schema/information-schema-tables.md) | 現在のユーザーが表示可能なテーブルの一覧を提供します。`SHOW TABLES` と類似しています。 |
| `TABLESPACES` | TiDBでは実装されていません。ゼロ行を返します。 |
| [`TABLE_CONSTRAINTS`](/information-schema/information-schema-table-constraints.md) | プライマリキー、一意なインデックス、外部キーに関する情報を提供します。 |
| `TABLE_PRIVILEGES` | TiDBでは実装されていません。ゼロ行を返します。 |
| `TRIGGERS` | TiDBでは実装されていません。ゼロ行を返します。 |
| [`USER_ATTRIBUTES`](/information-schema/information-schema-user-attributes.md) | ユーザーコメントやユーザー属性に関する情報をまとめます。 |
| [`USER_PRIVILEGES`](/information-schema/information-schema-user-privileges.md) | 現在のユーザーに関連付けられた特権をまとめます。 |
| [`VARIABLES_INFO`](/information-schema/information-schema-variables-info.md) | TiDBシステム変数に関する情報を提供します。 |
| [`VIEWS`](/information-schema/information-schema-views.md) | 現在のユーザーが表示可能なビューの一覧を提供します。`SHOW FULL TABLES WHERE table_type = 'VIEW'` を実行したのと同様です。 |

</CustomContent>

## TiDBの拡張テーブル

<CustomContent platform="tidb">

> **注：**
>
> 以下のテーブルの一部は、TiDB Self-Hostedでのみサポートされ、TiDB Cloudではサポートされていません。TiDB Cloudでサポートされていないテーブルの完全なリストについては、[システムテーブル](https://docs.pingcap.com/tidbcloud/limited-sql-features#system-tables)を参照してください。

| テーブル名                                                                              | 説明 |
|-----------------------------------------------------------------------------------------|-------------|
| [`ANALYZE_STATUS`](/information-schema/information-schema-analyze-status.md)            | 統計情報を収集するタスクに関する情報を提供します。 |
| [`CLIENT_ERRORS_SUMMARY_BY_HOST`](/information-schema/client-errors-summary-by-host.md)  | クライアントリクエストによって生成され、クライアントに返されたエラーや警告の概要を提供します。 |
| [`CLIENT_ERRORS_SUMMARY_BY_USER`](/information-schema/client-errors-summary-by-user.md)  | クライアントによって生成されたエラーや警告の概要を提供します。 |
| [`CLIENT_ERRORS_SUMMARY_GLOBAL`](/information-schema/client-errors-summary-global.md)   | クライアントによって生成されたエラーや警告の概要を提供します。 |
| [`CLUSTER_CONFIG`](/information-schema/information-schema-cluster-config.md)            | TiDBクラスタ全体の構成設定の詳細を提供します。このテーブルはTiDB Cloudには適用されません。 |
| `CLUSTER_DEADLOCKS` | `DEADLOCKS`テーブルのクラスタレベルのビューを提供します。 |
| [`CLUSTER_HARDWARE`](/information-schema/information-schema-cluster-hardware.md)            | 各TiDBコンポーネントで発見された基礎物理ハードウェアの詳細を提供します。このテーブルはTiDB Cloudには適用されません。 |
| [`CLUSTER_INFO`](/information-schema/information-schema-cluster-info.md)                | 現在のクラスタトポロジの詳細を提供します。このテーブルは[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスタでは利用できません。 |
| [`CLUSTER_LOAD`](/information-schema/information-schema-cluster-load.md)                | クラスタ内のTiDBサーバの現在の負荷情報を提供します。このテーブルはTiDB Cloudには適用されません。 |
| [`CLUSTER_LOG`](/information-schema/information-schema-cluster-log.md)                  | TiDBクラスタ全体のログを提供します。このテーブルはTiDB Cloudには適用されません。 |
| `CLUSTER_MEMORY_USAGE`                                                                  | `MEMORY_USAGE`テーブルのクラスタレベルのビューを提供します。 |
| `CLUSTER_MEMORY_USAGE_OPS_HISTORY`                                                      | `MEMORY_USAGE_OPS_HISTORY`テーブルのクラスタレベルのビューを提供します。 |
| `CLUSTER_PROCESSLIST`                                                                   | `PROCESSLIST`テーブルのクラスタレベルのビューを提供します。 |
| `CLUSTER_SLOW_QUERY`                                                                    | `SLOW_QUERY`テーブルのクラスタレベルのビューを提供します。このテーブルは[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスタでは利用できません。 |
| `CLUSTER_STATEMENTS_SUMMARY`                                                            | `STATEMENTS_SUMMARY`テーブルのクラスタレベルのビューを提供します。このテーブルは[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスタでは利用できません。 |
| `CLUSTER_STATEMENTS_SUMMARY_HISTORY`                                                    | `STATEMENTS_SUMMARY_HISTORY`テーブルのクラスタレベルのビューを提供します。このテーブルは[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスタでは利用できません。 |
| `CLUSTER_TIDB_TRX` | `TIDB_TRX`テーブルのクラスタレベルのビューを提供します。 |
| [`CLUSTER_SYSTEMINFO`](/information-schema/information-schema-cluster-systeminfo.md)    | クラスタ内のサーバのカーネルパラメータ構成に関する詳細を提供します。このテーブルはTiDB Cloudには適用されません。 |
| [`DATA_LOCK_WAITS`](/information-schema/information-schema-data-lock-waits.md) | TiKVサーバ上のロック待ち情報を提供します。 |
| [`DDL_JOBS`](/information-schema/information-schema-ddl-jobs.md)                        | `ADMIN SHOW DDL JOBS`と類似した出力を提供します。 |
| [`DEADLOCKS`](/information-schema/information-schema-deadlocks.md) | 最近発生したいくつかのデッドロックエラーの情報を提供します。 |
| [`INSPECTION_RESULT`](/information-schema/information-schema-inspection-result.md)      | 内部の診断チェックをトリガーします。このテーブルはTiDB Cloudには適用されません。 |
| [`INSPECTION_RULES`](/information-schema/information-schema-inspection-rules.md)        | 実行される内部の診断チェックのリストを提供します。このテーブルはTiDB Cloudには適用されません。 |
| [`INSPECTION_SUMMARY`](/information-schema/information-schema-inspection-summary.md)    | 重要なモニタリングメトリクスの要約レポートを提供します。このテーブルはTiDB Cloudには適用されません。 |
| [`MEMORY_USAGE`](/information-schema/information-schema-memory-usage.md)                | 現在のTiDBインスタンスのメモリ使用状況を提供します。 |
| [`MEMORY_USAGE_OPS_HISTORY`](/information-schema/information-schema-memory-usage-ops-history.md)    | メモリ関連の操作の履歴および現在のTiDBインスタンスの実行基準を提供します。 |
| [`METRICS_SUMMARY`](/information-schema/information-schema-metrics-summary.md)          | Prometheusから抽出されたメトリクスの要約を提供します。このテーブルはTiDB Cloudには適用されません。 |
| `METRICS_SUMMARY_BY_LABEL`                                                              | `METRICS_SUMMARY`テーブルを参照してください。このテーブルはTiDB Cloudには適用されません。 |
| [`METRICS_TABLES`](/information-schema/information-schema-metrics-tables.md)            | `METRICS_SCHEMA`内のテーブルのPromQL定義を提供します。このテーブルはTiDB Cloudには適用されません。 |
| [`PLACEMENT_POLICIES`](/information-schema/information-schema-placement-policies.md)    | すべての配置ポリシーに関する情報を提供します。このテーブルは[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスタでは利用できません。 |
| [`SEQUENCES`](/information-schema/information-schema-sequences.md)                      | TiDBのシーケンスの実装はMariaDBに基づいています。 |
| [`SLOW_QUERY`](/information-schema/information-schema-slow-query.md)                    | 現在のTiDBサーバ上の遅いクエリに関する情報を提供します。このテーブルは[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスタでは利用できません。 |
| [`STATEMENTS_SUMMARY`](/statement-summary-tables.md)                                    | MySQLのperformance_schemaステートメントサマリと類似しています。このテーブルは[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスタでは利用できません。 |
| [`STATEMENTS_SUMMARY_HISTORY`](/statement-summary-tables.md)                            | MySQLのperformance_schemaステートメントサマリ履歴と類似しています。このテーブルは[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスタでは利用できません。 |
| [`TABLE_STORAGE_STATS`](/information-schema/information-schema-table-storage-stats.md)  | ストレージ内のテーブルサイズの詳細を提供します。 |
| [`TIDB_HOT_REGIONS`](/information-schema/information-schema-tidb-hot-regions.md)        | ホットなリージョンに関する統計情報を提供します。 |
| [`TIDB_HOT_REGIONS_HISTORY`](/information-schema/information-schema-tidb-hot-regions-history.md) | ホットなリージョンに関する履歴統計情報を提供します。 |
| [`TIDB_INDEXES`](/information-schema/information-schema-tidb-indexes.md)                | TiDBテーブルのインデックス情報を提供します。 |
| [`TIDB_SERVERS_INFO`](/information-schema/information-schema-tidb-servers-info.md)      | TiDBサーバ（つまり、tidb-serverコンポーネント）のリストを提供します。 |
| [`TIDB_TRX`](/information-schema/information-schema-tidb-trx.md) | TiDBノードで実行されているトランザクションの情報を提供します。 |
| [`TIFLASH_REPLICA`](/information-schema/information-schema-tiflash-replica.md)          | TiFlashレプリカに関する詳細を提供します。 |
| [`TIKV_REGION_PEERS`](/information-schema/information-schema-tikv-region-peers.md)      | リージョンが格納されている場所に関する詳細を提供します。 |
| [`TIKV_REGION_STATUS`](/information-schema/information-schema-tikv-region-status.md)    | リージョンに関する統計情報を提供します。 |
| [`TIKV_STORE_STATUS`](/information-schema/information-schema-tikv-store-status.md)      | TiKVサーバに関する基本情報を提供します。 |

</CustomContent>

<CustomContent platform="tidb-cloud">

| テーブル名                                                                              | 説明 |
|-----------------------------------------------------------------------------------------|-------------|
| [`ANALYZE_STATUS`](/information-schema/information-schema-analyze-status.md)            | 統計情報を収集するタスクに関する情報を提供します。 |
| [`CLIENT_ERRORS_SUMMARY_BY_HOST`](/information-schema/client-errors-summary-by-host.md)  | クライアントリクエストによって生成され、クライアントに返されたエラーや警告の概要を提供します。 |
| [`CLIENT_ERRORS_SUMMARY_BY_USER`](/information-schema/client-errors-summary-by-user.md)  | クライアントによって生成されたエラーや警告の概要を提供します。 |
| [`CLIENT_ERRORS_SUMMARY_GLOBAL`](/information-schema/client-errors-summary-global.md)   | クライアントによって生成されたエラーや警告の概要を提供します。 |
| [`CLUSTER_CONFIG`](https://docs.pingcap.com/tidb/stable/information-schema-cluster-config)            | TiDBクラスタ全体の構成設定の詳細を提供します。このテーブルはTiDB Cloudには適用されません。 |
| `CLUSTER_DEADLOCKS` | `DEADLOCKS`テーブルのクラスタレベルのビューを提供します。 |
| [`CLUSTER_HARDWARE`](https://docs.pingcap.com/tidb/stable/information-schema-cluster-hardware)            | 各TiDBコンポーネントで検出された基礎物理ハードウェアの詳細を提供します。この表はTiDB Cloudには適用されません。 |
| [`CLUSTER_INFO`](/information-schema/information-schema-cluster-info.md)                | 現在のクラスタトポロジの詳細を提供します。この表は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスタで利用できません。 |
| [`CLUSTER_LOAD`](https://docs.pingcap.com/tidb/stable/information-schema-cluster-load)                | クラスタ内のTiDBサーバの現在の負荷情報を提供します。この表はTiDB Cloudには適用されません。 |
| [`CLUSTER_LOG`](https://docs.pingcap.com/tidb/stable/information-schema-cluster-log)                  | TiDBクラスタ全体のログを提供します。この表はTiDB Cloudには適用されません。 |
| `CLUSTER_MEMORY_USAGE`                                                                  | `MEMORY_USAGE`テーブルのクラスタレベルのビューを提供します。この表はTiDB Cloudには適用されません。 |
| `CLUSTER_MEMORY_USAGE_OPS_HISTORY`                                                      | `MEMORY_USAGE_OPS_HISTORY`テーブルのクラスタレベルのビューを提供します。この表はTiDB Cloudには適用されません。 |
| `CLUSTER_PROCESSLIST`                                                                   | `PROCESSLIST`テーブルのクラスタレベルのビューを提供します。 |
| `CLUSTER_SLOW_QUERY`                                                                    | `SLOW_QUERY`テーブルのクラスタレベルのビューを提供します。この表は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスタで利用できません。 |
| `CLUSTER_STATEMENTS_SUMMARY`                                                            | `STATEMENTS_SUMMARY`テーブルのクラスタレベルのビューを提供します。この表は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスタで利用できません。 |
| `CLUSTER_STATEMENTS_SUMMARY_HISTORY`                                                    | `STATEMENTS_SUMMARY_HISTORY`テーブルのクラスタレベルのビューを提供します。この表は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスタで利用できません。 |
| `CLUSTER_TIDB_TRX` | `TIDB_TRX`テーブルのクラスタレベルのビューを提供します。 |
| [`CLUSTER_SYSTEMINFO`](https://docs.pingcap.com/tidb/stable/information-schema-cluster-systeminfo)    | クラスタ内のサーバのカーネルパラメーター構成の詳細を提供します。この表はTiDB Cloudには適用されません。 |
| [`DATA_LOCK_WAITS`](/information-schema/information-schema-data-lock-waits.md) | TiKVサーバでのロック待ち情報を提供します。 |
| [`DDL_JOBS`](/information-schema/information-schema-ddl-jobs.md)                        | `ADMIN SHOW DDL JOBS`と同様の出力を提供します。 |
| [`DEADLOCKS`](/information-schema/information-schema-deadlocks.md) | 最近発生した複数のデッドロックエラーの情報を提供します。 |
| [`INSPECTION_RESULT`](https://docs.pingcap.com/tidb/stable/information-schema-inspection-result)      | 内部診断チェックをトリガーします。この表はTiDB Cloudには適用されません。 |
| [`INSPECTION_RULES`](https://docs.pingcap.com/tidb/stable/information-schema-inspection-rules)        | 実行された内部診断チェックの一覧を提供します。この表はTiDB Cloudには適用されません。 |
| [`INSPECTION_SUMMARY`](https://docs.pingcap.com/tidb/stable/information-schema-inspection-summary)    | 重要なモニタリングメトリクスの要約レポートを提供します。この表はTiDB Cloudには適用されません。 |
| [`MEMORY_USAGE`](/information-schema/information-schema-memory-usage.md)                | 現在のTiDBインスタンスのメモリ使用状況。 |
| [`MEMORY_USAGE_OPS_HISTORY`](/information-schema/information-schema-memory-usage-ops-history.md)    | メモリ関連操作の履歴および現在のTiDBインスタンスの実行基準を提供します。 |
| [`METRICS_SUMMARY`](https://docs.pingcap.com/tidb/stable/information-schema-metrics-summary)          | Prometheusから抽出されたメトリクスの要約を提供します。この表はTiDB Cloudには適用されません。 |
| `METRICS_SUMMARY_BY_LABEL`                                                              | `METRICS_SUMMARY`テーブルを参照してください。この表はTiDB Cloudには適用されません。 |
| [`METRICS_TABLES`](https://docs.pingcap.com/tidb/stable/information-schema-metrics-tables)            | `METRICS_SCHEMA`内のテーブルのPromQL定義を提供します。この表はTiDB Cloudには適用されません。 |
| [`PLACEMENT_POLICIES`](https://docs.pingcap.com/tidb/stable/information-schema-placement-policies)    | すべての配置ポリシーに関する情報を提供します。この表は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスタで利用できません。 |
| [`SEQUENCES`](/information-schema/information-schema-sequences.md)                      | TiDBでのシーケンスの実装はMariaDBに基づいています。 |
| [`SLOW_QUERY`](/information-schema/information-schema-slow-query.md)                    | 現在のTiDBサーバでの遅いクエリに関する情報を提供します。この表は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスタで利用できません。 |
| [`STATEMENTS_SUMMARY`](/statement-summary-tables.md)                                    | MySQLのperformance_schemaステートメントサマリと類似しています。この表は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスタで利用できません。 |
| [`STATEMENTS_SUMMARY_HISTORY`](/statement-summary-tables.md)                            | MySQLのperformance_schemaステートメントサマリ履歴と類似しています。この表は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスタで利用できません。|
| [`TABLE_STORAGE_STATS`](/information-schema/information-schema-table-storage-stats.md)  | ストレージ内のテーブルサイズの詳細を提供します。 |
| [`TIDB_HOT_REGIONS`](https://docs.pingcap.com/tidb/stable/information-schema-tidb-hot-regions)        | ホットなリージョンに関する統計情報を提供します。この表はTiDB Cloudには適用されません。 |
| [`TIDB_HOT_REGIONS_HISTORY`](/information-schema/information-schema-tidb-hot-regions-history.md) | ホットなリージョンに関する履歴統計情報を提供します。 |
| [`TIDB_INDEXES`](/information-schema/information-schema-tidb-indexes.md)                | TiDBテーブルに関するインデックス情報を提供します。 |
| [`TIDB_SERVERS_INFO`](/information-schema/information-schema-tidb-servers-info.md)      | TiDBサーバ（つまり、tidb-serverコンポーネント）のリストを提供します。 |
| [`TIDB_TRX`](/information-schema/information-schema-tidb-trx.md) | TiDBノードで実行されているトランザクションの情報を提供します。 |
| [`TIFLASH_REPLICA`](/information-schema/information-schema-tiflash-replica.md)          | TiFlashレプリカに関する詳細を提供します。 |
| [`TIKV_REGION_PEERS`](/information-schema/information-schema-tikv-region-peers.md)      | 領域が保存されている場所に関する詳細を提供します。 |
| [`TIKV_REGION_STATUS`](/information-schema/information-schema-tikv-region-status.md)    | 領域に関する統計情報を提供します。 |
| [`TIKV_STORE_STATUS`](/information-schema/information-schema-tikv-store-status.md)      | TiKVサーバに関する基本情報を提供します。 |