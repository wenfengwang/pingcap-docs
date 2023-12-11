---
title: TiDB CloudとNew Relicの統合（ベータ版）
summary: TiDBクラスタをNew Relic連携で監視する方法を学びます。

# TiDB CloudとNew Relicの統合（ベータ版）

TiDB CloudはNew Relic連携（ベータ版）をサポートしています。TiDB Cloudを設定して、TiDBクラスタのメトリクスデータを[New Relic](https://newrelic.com/)に送信することができます。その後、これらのメトリクスをNew Relicのダッシュボードで直接確認できます。

## 前提条件

- TiDB CloudをNew Relicと統合するには、New Relicアカウントと[New Relic APIキー](https://one.newrelic.com/admin-portal/api-keys/home?)が必要です。New Relicは、New Relicアカウントを初めて作成した際にAPIキーを付与します。

    New Relicアカウントがない場合は、[こちら](https://newrelic.com/signup)からサインアップしてください。

- TiDB Cloudのサードパーティ統合設定を編集するには、組織の**所有者**アクセス権限またはTiDB Cloudの対象プロジェクトの**プロジェクトメンバー**アクセス権限が必要です。

## 制限

[TiDB Serverlessクラスタ](/tidb-cloud/select-cluster-tier.md#tidb-serverless)ではNew Relic連携を使用できません。

## 手順

### ステップ1. New RelicのAPIキーを統合する

1. [TiDB Cloudコンソール](https://tidbcloud.com)にログインします。
2. 左下隅の<MDSvgIcon name="icon-left-projects" />をクリックし、複数のプロジェクトを持っている場合は対象プロジェクトに切り替え、そして**プロジェクト設定**をクリックします。
3. プロジェクトの**プロジェクト設定**ページで、左のナビゲーションペインで**統合**をクリックし、次に**New Relicへの統合（BETA）**をクリックします。
4. New RelicのAPIキーを入力し、New Relicのサイトを選択します。
5. **統合をテスト**をクリックします。

    - テストに成功すると、**確認**ボタンが表示されます。
    - テストに失敗すると、エラーメッセージが表示されます。そのメッセージに従ってトラブルシューティングを行い、統合を再試行します。

6. 統合を完了するには、**確認**をクリックします。

### ステップ2. New RelicでTiDB Cloudダッシュボードを追加する

1. [New Relic](https://one.newrelic.com/)にログインします。
2. **データの追加**をクリックし、`TiDB Cloud`を検索し、次に**TiDB Cloudモニタリング**ページに移動します。または、直接[リンク](https://one.newrelic.com/marketplace?state=79bf274b-0c01-7960-c85c-3046ca96568e)をクリックしてページにアクセスできます。
3. アカウントIDを選択し、New Relicでダッシュボードを作成します。

## 事前構築されたダッシュボード

統合の**New Relic**カードの**ダッシュボード**リンクをクリックすると、TiDBクラスタの事前構築されたダッシュボードが表示されます。

## New Relicで利用可能なメトリクス

New Relicは、次のTiDBクラスタのメトリクスデータを追跡します。

| メトリック名 | メトリックタイプ | ラベル | 説明 |
| :------------| :---------- | :------| :----------------------------------------------------- |
| tidb_cloud.db_database_time| ゲージ | sql_type: Select\|Insert\|...<br/><br/>cluster_name: `<クラスタ名>`<br/><br/>instance: tidb-0\|tidb-1…<br/><br/>component: `tidb` | TiDBで実行されるすべてのSQLステートメントによって消費される合計時間（秒単位）を示します。この時間には、すべてのプロセスのCPU時間と非アイドル待機時間が含まれます。 |
| tidb_cloud.db_query_per_second| ゲージ | type: Select\|Insert\|...<br/><br/>cluster_name: `<クラスタ名>`<br/><br/>instance: tidb-0\|tidb-1…<br/><br/>component: `tidb` | すべてのTiDBインスタンスで実行されたSQLステートメントの数（1秒あたり）を示します。これは`SELECT`、`INSERT`、`UPDATE`などのステートメントごとにカウントされます。 |
| tidb_cloud.db_average_query_duration| ゲージ | sql_type: Select\|Insert\|...<br/><br/>cluster_name: `<クラスタ名>`<br/><br/>instance: tidb-0\|tidb-1…<br/><br/>component: `tidb` | クライアントのネットワークリクエストがTiDBに送信されてからTiDBが実行したリクエストをクライアントに返すまでの時間間隔を示します。 |
| tidb_cloud.db_failed_queries| ゲージ | type: executor:xxxx\|parser:xxxx\|...<br/><br/>cluster_name: `<クラスタ名>`<br/><br/>instance: tidb-0\|tidb-1…<br/><br/>component: `tidb` | 各TiDBインスタンスで発生するSQL実行エラー（構文エラーや主キーの競合など）の統計を示します。 |
| tidb_cloud.db_total_connection| ゲージ | cluster_name: `<クラスタ名>`<br/><br/>instance: tidb-0\|tidb-1…<br/><br/>component: `tidb` | TiDBサーバーの現在の接続数を示します。 |
| tidb_cloud.db_active_connections| ゲージ | cluster_name: `<クラスタ名>`<br/><br/>instance: tidb-0\|tidb-1…<br/><br/>component: `tidb` | アクティブな接続の数を示します。 |
| tidb_cloud.db_disconnections| ゲージ | result: ok\|error\|undetermined<br/><br/>cluster_name: `<クラスタ名>`<br/><br/>instance: tidb-0\|tidb-1…<br/><br/>component: `tidb` | 切断されたクライアントの数を示します。 |
| tidb_cloud.db_command_per_second| ゲージ | type: Query\|StmtPrepare\|...<br/><br/>cluster_name: `<クラスタ名>`<br/><br/>instance: tidb-0\|tidb-1…<br/><br/>component: `tidb` | TiDBによって処理されたコマンドの数（1秒あたり）を示します。これはコマンドの実行結果の成功または失敗に応じて分類されます。 |
| tidb_cloud.db_queries_using_plan_cache_ops| ゲージ | cluster_name: `<クラスタ名>`<br/><br/>instance: tidb-0\|tidb-1…<br/><br/>component: `tidb` | [Plan Cache](/sql-prepared-plan-cache.md)を使用するクエリの統計（1秒あたり）を示します。実行計画キャッシュは準備済みステートメントコマンドのみをサポートしています。 |
| tidb_cloud.db_transaction_per_second| ゲージ | txn_mode: pessimistic\|optimistic<br/><br/>type: abort\|commit\|...<br/><br/>cluster_name: `<クラスタ名>`<br/><br/>instance: tidb-0\|tidb-1…<br/><br/>component: `tidb` | 1秒あたりに実行されたトランザクションの数を示します。 |
| tidb_cloud.node_storage_used_bytes | ゲージ | cluster_name: `<クラスタ名>`<br/><br/>instance: tikv-0\|tikv-1…\|tiflash-0\|tiflash-1…<br/><br/>component: tikv\|tiflash | TiKV/TiFlashノードのディスク使用量（バイト単位）を示します。 |
| tidb_cloud.node_storage_capacity_bytes | ゲージ | cluster_name: `<クラスタ名>`<br/><br/>instance: tikv-0\|tikv-1…\|tiflash-0\|tiflash-1…<br/><br/>component: tikv\|tiflash | TiKV/TiFlashノードのディスク容量（バイト単位）を示します。 |
| tidb_cloud.node_cpu_seconds_total | カウント | cluster_name: `<クラスタ名>`<br/><br/>instance: tidb-0\|tidb-1…\|tikv-0…\|tiflash-0…<br/><br/>component: tidb\|tikv\|tiflash | TiDB/TiKV/TiFlashノードのCPU使用量を示します。 |
| tidb_cloud.node_cpu_capacity_cores | ゲージ | cluster_name: `<クラスタ名>`<br/><br/>instance: tidb-0\|tidb-1…\|tikv-0…\|tiflash-0…<br/><br/>component: tidb\|tikv\|tiflash | TiDB/TiKV/TiFlashノードのCPUコアの制限を示します。 |
| tidb_cloud.node_memory_used_bytes | ゲージ | cluster_name: `<クラスタ名>`<br/><br/>instance: tidb-0\|tidb-1…\|tikv-0…\|tiflash-0…<br/><br/>component: tidb\|tikv\|tiflash | TiDB/TiKV/TiFlashノードの使用済みメモリ（バイト単位）を示します。 |
| tidb_cloud.node_memory_capacity_bytes | ゲージ | cluster_name: `<クラスタ名>`<br/><br/>instance: tidb-0\|tidb-1…\|tikv-0…\|tiflash-0…<br/><br/>component: tidb\|tikv\|tiflash | TiDB/TiKV/TiFlashノードのメモリ容量（バイト単位）を示します。 |
