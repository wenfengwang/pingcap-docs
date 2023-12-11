---
title: TiDBのスキーマ
summary: TiDBのシステムテーブルについて学ぶ
aliases: ['/docs/dev/system-tables/system-table-overview/','/docs/dev/reference/system-databases/mysql/','/tidb/dev/system-table-overview/']
---

# `mysql`スキーマ

`mysql`スキーマにはTiDBのシステムテーブルが含まれています。デザインはMySQLの`mysql`スキーマに類似しており、`mysql.user`などのテーブルは直接編集できます。また、MySQLの拡張機能としていくつかのテーブルが含まれています。

## 権限システムテーブル

これらのシステムテーブルにはユーザーアカウントおよびその権限に関する権限情報が含まれています：

- `user`: ユーザーアカウント、グローバル権限、およびその他の非権限列
- `db`: データベースレベルの権限
- `tables_priv`: テーブルレベルの権限
- `columns_priv`: カラムレベルの権限
- `password_history`: パスワード変更履歴
- `default_roles`: ユーザーのデフォルトのロール
- `global_grants`: ダイナミック権限
- `global_priv`: 証明書に基づく認証情報
- `role_edges`: ロール間の関係

## サーバーサイドのヘルプシステムテーブル

現在、`help_topic`はNULLです。

## 統計情報システムテーブル

- `stats_buckets`: 統計情報のバケット
- `stats_histograms`: 統計情報のヒストグラム
- `stats_top_n`: 統計情報のTopN
- `stats_meta`: テーブルのメタ情報、例えば総行数や更新された行数など
- `stats_extended`: 拡張統計情報、例えば列間の順序相関など
- `stats_feedback`: 統計情報のクエリフィードバック
- `stats_fm_sketch`: 統計情報のヒストグラムのFMSketch分布
- `analyze_options`: 各テーブルのデフォルトの`analyze`オプション
- `column_stats_usage`: カラム統計情報の使用状況
- `schema_index_usage`: インデックスの使用状況
- `analyze_jobs`: 過去7日間の統計情報収集タスクと履歴タスクレコード

## 実行計画関連のシステムテーブル

- `bind_info`: 実行計画のバインディング情報
- `capture_plan_baselines_blacklist`: 実行計画の自動バインディングのブロックリスト

## GCワーカーシステムテーブル

> **注意:**
>
> GCワーカーシステムテーブルはTiDB Self-Hostedにのみ適用され、[TiDB Cloud](https://docs.pingcap.com/tidbcloud/)では利用できません。

- `gc_delete_range`: 削除されるべきKV範囲
- `gc_delete_range_done`: 削除されたKV範囲

## キャッシュされたテーブルに関連するシステムテーブル

- `table_cache_meta`はキャッシュされたテーブルのメタデータを格納します。

## TTLに関連するシステムテーブル

> **注意:**

> TTLに関連するシステムテーブルは[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスタでは利用できません。

* `tidb_ttl_table_status`: すべてのTTLテーブルの以前に実行されたTTLジョブおよび現在実行中のTTLジョブ
* `tidb_ttl_task`: 現在実行中のTTLサブタスク
* `tidb_ttl_job_history`: 過去90日間のTTLタスクの実行履歴

## メタデータロックに関連するシステムテーブル

* `tidb_mdl_view`：メタデータロックのビュー。現在ブロックされているDDLステートメントに関する情報を表示できます
* `tidb_mdl_info`：TiDB内部で使用され、ノード間でメタデータロックを同期するための情報を表示します

## その他のシステムテーブル

<CustomContent platform="tidb">

> **注意:**
>
> `tidb`、`expr_pushdown_blacklist`、`opt_rule_blacklist`、`table_cache_meta`、`tidb_import_jobs`、`tidb_timers`システムテーブルはTiDB Self-Hostedにのみ適用され、[TiDB Cloud](https://docs.pingcap.com/tidbcloud/)では利用できません。

- `GLOBAL_VARIABLES`: グローバルシステム変数テーブル
- `tidb`: TiDBが`bootstrap`を実行する際のバージョン情報を記録する
- `expr_pushdown_blacklist`: 式プッシュダウンのブロックリスト
- `opt_rule_blacklist`: 論理最適化ルールのブロックリスト
- `table_cache_meta`: キャッシュされたテーブルのメタデータ
- `tidb_import_jobs`: [`IMPORT INTO`](/sql-statements/sql-statement-import-into.md)のジョブ情報
- `tidb_timers`: 内部タイマーのメタデータ

</CustomContent>

<CustomContent platform="tidb-cloud">

- `GLOBAL_VARIABLES`: グローバルシステム変数テーブル

</CustomContent>