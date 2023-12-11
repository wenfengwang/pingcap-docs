---
title: TiDB CloudとDatadogの統合（ベータ版）
summary: TiDBクラスタをDatadogの統合でモニタリングする方法を学ぶ。

# TiDB CloudとDatadogの統合（ベータ版）

TiDB CloudはDatadogの統合（ベータ版）をサポートしています。TiDBクラスタに関するメトリクスデータを[Datadog](https://www.datadoghq.com/)に送信するようにTiDB Cloudを構成できます。その後、これらのメトリクスをDatadogのダッシュボードで直接表示できます。

## 前提条件

- TiDB CloudをDatadogと統合するためには、Datadogのアカウントと[Datadog APIキー](https://app.datadoghq.com/organization-settings/api-keys)が必要です。Datadogのアカウントを最初に作成すると、DatadogからAPIキーが付与されます。

    Datadogのアカウントを持っていない場合は、[https://app.datadoghq.com/signup](https://app.datadoghq.com/signup)でサインアップしてください。

- TiDB Cloudのサードパーティ統合設定を編集するには、組織の`Organization Owner`アクセスまたはTiDB Cloudのターゲットプロジェクトへの`Project Member`アクセスが必要です。

## 制限事項

- [TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスタではDatadogの統合を使用できません。

- クラスタのステータスが**CREATING**、**RESTORING**、**PAUSED**、または**RESUMING**の場合、Datadogの統合は利用できません。

## 手順

### ステップ1. DatadogのAPIキーと統合する

1. [TiDB Cloudコンソール](https://tidbcloud.com)にログインします。
2. 左下隅の<MDSvgIcon name="icon-left-projects" />をクリックし、複数のプロジェクトを持っている場合は対象プロジェクトに切り替えてから**Project Settings**をクリックします。
3. プロジェクトの**Project Settings**ページで、左のナビゲーションペインで**Integrations**をクリックし、次に**Integration to Datadog (BETA)**をクリックします。
4. DatadogのAPIキーを入力し、Datadogのサイトを選択します。
5. **Test Integration**をクリックします。

    - テストに成功すると、**Confirm**ボタンが表示されます。
    - テストに失敗すると、エラーメッセージが表示されます。メッセージに従ってトラブルシューティングし、統合を再試行してください。

6. 統合を完了するには**Confirm**をクリックします。

### ステップ2. DatadogでTiDB Cloud統合をインストールする

1. [Datadog](https://app.datadoghq.com)にログインします。
2. Datadogで**TiDB Cloud Integration**ページ(<https://app.datadoghq.com/account/settings#integrations/tidb-cloud>)に移動します。
3. **Configuration**タブで**Install Integration**をクリックします。その結果、[**TiDBCloud Cluster Overview**](https://app.datadoghq.com/dash/integration/30586/tidbcloud-cluster-overview)ダッシュボードが[**Dashboard List**](https://app.datadoghq.com/dashboard/lists)に表示されます。

## 事前に構築されたダッシュボード

統合の**Datadog**カードの**Dashboard**リンクをクリックします。TiDBクラスタの事前に構築されたダッシュボードが表示されます。

## Datadogが利用可能なメトリクス

Datadogは以下のTiDBクラスタに関するメトリクスデータをトラッキングします。

| メトリック名        | メトリックの種類 | ラベル | 説明 |
| :----------------- | :---------------- | :----- | :------------------------------------------------- |
| tidb_cloud.db_database_time | ゲージ値 | sql_type: Select\|Insert\|...<br/>cluster_name: `<クラスタ名>`<br/>instance: tidb-0\|tidb-1…<br/>component: `tidb` | TiDBで実行されるすべてのSQLステートメントによって消費される秒ごとの合計時間を示し、すべてのプロセスのCPU時間とアイドルでない待ち時間を含んでいます。 |
| tidb_cloud.db_query_per_second | ゲージ値 | type: Select\|Insert\|...<br/>cluster_name: `<クラスタ名>`<br/>instance: tidb-0\|tidb-1…<br/>component: `tidb` | すべてのTiDBインスタンスで実行される秒ごとのSQLステートメントの数を示し、SELECT、INSERT、UPDATEなどのステートメントに従ってカウントされます。 |
| tidb_cloud.db_average_query_duration | ゲージ値 | sql_type: Select\|Insert\|...<br/>cluster_name: `<クラスタ名>`<br/>instance: tidb-0\|tidb-1…<br/>component: `tidb` | クライアントのネットワークリクエストがTiDBに送信されてからTiDBがそれを実行した後にクライアントに返されるまでの期間を示します。 |
| tidb_cloud.db_failed_queries | ゲージ値 | type: executor:xxxx\|parser:xxxx\|...<br/>cluster_name: `<クラスタ名>`<br/>instance: tidb-0\|tidb-1…<br/>component: `tidb` | 各TiDBインスタンスで発生するSQL実行エラー（構文エラーや主キーの衝突など）の統計を示します。 |
| tidb_cloud.db_total_connection | ゲージ値 | cluster_name: `<クラスタ名>`<br/>instance: tidb-0\|tidb-1…<br/>component: `tidb` | TiDBサーバの現在の接続数を示します。 |
| tidb_cloud.db_active_connections | ゲージ値 | cluster_name: `<クラスタ名>`<br/>instance: tidb-0\|tidb-1…<br/>component: `tidb` | アクティブな接続の数を示します。 |
| tidb_cloud.db_disconnections | ゲージ値 | result: ok\|error\|undetermined<br/>cluster_name: `<クラスタ名>`<br/>instance: tidb-0\|tidb-1…<br/>component: `tidb` | 切断されたクライアントの数を示します。 |
| tidb_cloud.db_command_per_second | ゲージ値 | type: Query\|StmtPrepare\|...<br/>cluster_name: `<クラスタ名>`<br/>instance: tidb-0\|tidb-1…<br/>component: `tidb` | TiDBによって秒ごとに処理されるコマンドの数を示し、コマンドの実行結果の成功または失敗に応じて分類されます。 |
| tidb_cloud.db_queries_using_plan_cache_ops | ゲージ値 | cluster_name: `<クラスタ名>`<br/>instance: tidb-0\|tidb-1…<br/>component: `tidb` | [Plan Cache](/sql-prepared-plan-cache.md)を使用するクエリの統計を示します。実行計画キャッシュは準備されたステートメントコマンドのみをサポートしています。 |
| tidb_cloud.db_transaction_per_second | ゲージ値 | txn_mode: pessimistic\|optimistic<br/>type: abort\|commit\|...<br/>cluster_name: `<クラスタ名>`<br/>instance: tidb-0\|tidb-1…<br/>component: `tidb` | 秒ごとに実行されるトランザクションの数を示します。 |
| tidb_cloud.node_storage_used_bytes | ゲージ値 | cluster_name: `<クラスタ名>`<br/>instance: tikv-0\|tikv-1…\|tiflash-0\|tiflash-1…<br/>component: tikv\|tiflash | TiKV/TiFlashノードのディスク使用量（バイト単位）を示します。 |
| tidb_cloud.node_storage_capacity_bytes | ゲージ値 | cluster_name: `<クラスタ名>`<br/>instance: tikv-0\|tikv-1…\|tiflash-0\|tiflash-1…<br/>component: tikv\|tiflash | TiKV/TiFlashノードのディスク容量（バイト単位）を示します。 |
| tidb_cloud.node_cpu_seconds_total | カウント | cluster_name: `<クラスタ名>`<br/>instance: tidb-0\|tidb-1…\|tikv-0…\|tiflash-0…<br/>component: tidb\|tikv\|tiflash | TiDB/TiKV/TiFlashノードのCPU使用量を示します。 |
| tidb_cloud.node_cpu_capacity_cores | ゲージ値 | cluster_name: `<クラスタ名>`<br/>instance: tidb-0\|tidb-1…\|tikv-0…\|tiflash-0…<br/>component: tidb\|tikv\|tiflash | TiDB/TiKV/TiFlashノードのCPUコアの制限を示します。 |
| tidb_cloud.node_memory_used_bytes | ゲージ値 | cluster_name: `<クラスタ名>`<br/>instance: tidb-0\|tidb-1…\|tikv-0…\|tiflash-0…<br/>component: tidb\|tikv\|tiflash | TiDB/TiKV/TiFlashノードの使用済みメモリ（バイト単位）を示します。 |
| tidb_cloud.node_memory_capacity_bytes | ゲージ値 | cluster_name: `<クラスタ名>`<br/>instance: tidb-0\|tidb-1…\|tikv-0…\|tiflash-0…<br/>component: tidb\|tikv\|tiflash | TiDB/TiKV/TiFlashノードのメモリ容量（バイト単位）を示します。 |
