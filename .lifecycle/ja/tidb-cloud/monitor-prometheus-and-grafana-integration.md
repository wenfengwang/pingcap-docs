---
title: PrometheusとGrafanaをTiDB Cloudと統合する（ベータ版）
summary: PrometheusとGrafanaの連携によりTiDBクラスタを監視する方法について学びます。

# PrometheusとGrafanaをTiDB Cloudと統合する（ベータ版）

TiDB Cloudは[Prometheus](https://prometheus.io/)のAPIエンドポイント（ベータ版）を提供しています。Prometheusサービスを保有している場合は、そのエンドポイントからTiDB Cloudの主要なメトリクスを簡単に監視できます。

このドキュメントでは、Prometheusサービスを構成してTiDB Cloudの主要メトリクスを読み取る方法、およびそれらのメトリクスを[Grafana](https://grafana.com/)を使用して表示する方法について説明します。

## 前提条件

- TiDB CloudをPrometheusと連携させるには、自己ホスト型または管理済みのPrometheusサービスが必要です。

- TiDB Cloudのサードパーティの統合設定を編集するには、組織のオーナーアクセスまたはTiDB Cloudの対象プロジェクトへのプロジェクトメンバーアクセスが必要です。

## 制限

- [TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスタでは、PrometheusとGrafanaの統合は使用できません。

- クラスタのステータスが **CREATING**, **RESTORING**, **PAUSED**, または **RESUMING** の場合は、PrometheusとGrafanaの統合は利用できません。

## 手順

### ステップ1. Prometheus用のscrape_configファイルを取得する

TiDB CloudのメトリクスをPrometheusサービスで読み取る前に、まずTiDB Cloudでscrape_config YAMLファイルを生成する必要があります。scrape_configファイルには、Prometheusサービスが現在のプロジェクト内の任意のデータベースクラスタを監視するための一意のベアラトークンが含まれています。

Prometheus用のscrape_configファイルを取得するには、次の手順を実行します：

1. [TiDB Cloudコンソール](https://tidbcloud.com)にログインします。
2. 左下の <MDSvgIcon name="icon-left-projects" /> をクリックし、複数のプロジェクトを持っている場合は対象のプロジェクトに切り替えてから **プロジェクト設定** をクリックします。
3. プロジェクトの**プロジェクト設定**ページで、左のナビゲーションペインで **Integrations** をクリックし、次に **Integration to Prometheus (BETA)** をクリックします。
4. **ファイルを追加** をクリックして現在のプロジェクトのscrape_configファイルを生成し表示します。

5. 生成されたscrape_configファイルの内容を後で使用するためにコピーします。

    > **注意：**
    >
    > セキュリティ上の理由から、TiDB Cloudは新しく生成されたscrape_configファイルを一度のみ表示します。ファイルウィンドウを閉じる前に内容をコピーすることを忘れないようにしてください。そうしないと、TiDB Cloudのscrape_configファイルを削除して新しいファイルを生成する必要があります。scrape_configファイルを削除するには、ファイルを選択して **...** をクリックし、次に **削除** をクリックします。

### ステップ2. Prometheusと統合する

1. Prometheusサービスで指定された監視ディレクトリ内で、Prometheus構成ファイルを検索します。

    たとえば、`/etc/prometheus/prometheus.yml`です。

2. Prometheus構成ファイル内で、`scrape_configs`セクションを検索し、そのセクションにTiDB Cloudから取得したscrape_configファイルの内容をコピーします。

3. Prometheusサービスで、**Status** > **Targets** をチェックして、新しいscrape_configファイルが読み取られたことを確認します。読み取られていない場合は、Prometheusサービスを再起動する必要がある場合があります。

### ステップ3. Grafana GUIダッシュボードを使用してメトリクスを視覚化する

PrometheusサービスがTiDB Cloudからメトリクスを読み取っている後、次の手順でグラフィナGUIダッシュボードを使用してメトリクスを視覚化できます：

1. TiDB CloudのGrafanaダッシュボードJSONを[こちら](https://github.com/pingcap/docs/blob/master/tidb-cloud/monitor-prometheus-and-grafana-integration-grafana-dashboard-UI.json)からダウンロードします。
2. [このJSONを独自のGrafana GUIにインポート](https://grafana.com/docs/grafana/v8.5/dashboards/export-import/#import-dashboard)してメトリクスを視覚化します。
3. （オプション）パネルを追加または削除したり、データソースを変更したり、表示オプションを変更して、必要に応じてダッシュボードをカスタマイズします。

Grafanaの使用方法についての詳細は、[Grafanaドキュメント](https://grafana.com/docs/grafana/latest/getting-started/getting-started-prometheus/)を参照してください。

## scrape_configの定期的な更新のベストプラクティス

データのセキュリティを向上させるために、定期的なscrape_configファイルベアラトークンの更新が一般的なベストプラクティスです。

1. [ステップ1](#step-1-get-a-scrape_config-file-for-prometheus)に従って、Prometheus用の新しいscrape_configファイルを作成します。
2. 新しいファイルの内容をPrometheus構成ファイルに追加します。
3. PrometheusサービスがTiDB Cloudから引き続き読み取り可能であることを確認した後、古いscrape_configファイルの内容をPrometheus構成ファイルから削除します。
4. プロジェクトの**Integration**ページで、関連する古いscrape_configファイルを削除して、他のユーザーがTiDB Cloud Prometheusエンドポイントから読み取るのをブロックします。

## Prometheusで利用可能なメトリクス

Prometheusは、TiDBクラスタの以下のメトリクスデータを追跡します。

| メトリック名 |  メトリックの種類  | ラベル | 説明 |
|:--- |:--- |:--- |:--- |
| tidbcloud_db_queries_total | カウント | sql_type: `Select\|Insert\|...`<br/>cluster_name: `<クラスタ名>`<br/>instance: `tidb-0\|tidb-1…`<br/>component: `tidb` | 実行されたステートメントの総数 |
| tidbcloud_db_failed_queries_total | カウント | type: `planner:xxx\|executor:2345\|...`<br/>cluster_name: `<クラスタ名>`<br/>instance: `tidb-0\|tidb-1…`<br/>component: `tidb` | 実行エラーの総数 |
| tidbcloud_db_connections | ゲージ | クラスタ名: `<クラスタ名>`<br/>instance: `tidb-0\|tidb-1…`<br/>component: `tidb` | TiDBサーバの現在の接続数 |
| tidbcloud_db_query_duration_seconds | ヒストグラム | sql_type: `Select\|Insert\|...`<br/>cluster_name: `<クラスタ名>`<br/>instance: `tidb-0\|tidb-1…`<br/>component: `tidb` | ステートメントの実行時間のヒストグラム |
| tidbcloud_changefeed_latency | ゲージ | changefeed_id | changefeedの上流と下流間のデータ複製の遅延 |
| tidbcloud_changefeed_replica_rows | ゲージ | changefeed_id | changefeedが下流に書き込む複製行数（秒単位） |
| tidbcloud_node_storage_used_bytes | ゲージ | cluster_name: `<クラスタ名>`<br/>instance: `tikv-0\|tikv-1…\|tiflash-0\|tiflash-1…`<br/>component: `tikv\|tiflash` | TiKV/TiFlashノードのディスク使用量（バイト単位） |
| tidbcloud_node_storage_capacity_bytes | ゲージ | cluster_name: `<クラスタ名>`<br/>instance: `tikv-0\|tikv-1…\|tiflash-0\|tiflash-1…`<br/>component: `tikv\|tiflash` | TiKV/TiFlashノードのディスク容量（バイト単位） |
| tidbcloud_node_cpu_seconds_total | カウント | cluster_name: `<クラスタ名>`<br/>instance: `tidb-0\|tidb-1…\|tikv-0…\|tiflash-0…`<br/>component: `tidb\|tikv\|tiflash` | TiDB/TiKV/TiFlashノードのCPU使用量 |
| tidbcloud_node_cpu_capacity_cores | ゲージ | cluster_name: `<クラスタ名>`<br/>instance: `tidb-0\|tidb-1…\|tikv-0…\|tiflash-0…`<br/>component: `tidb\|tikv\|tiflash` | TiDB/TiKV/TiFlashノードのCPUリミットコア |
| tidbcloud_node_memory_used_bytes | ゲージ | cluster_name: `<クラスタ名>`<br/>instance: `tidb-0\|tidb-1…\|tikv-0…\|tiflash-0…`<br/>component: `tidb\|tikv\|tiflash` | TiDB/TiKV/TiFlashノードの使用メモリバイト数 |
| tidbcloud_node_memory_capacity_bytes | ゲージ | cluster_name: `<クラスタ名>`<br/>instance: `tidb-0\|tidb-1…\|tikv-0…\|tiflash-0…`<br/>component: `tidb\|tikv\|tiflash` | TiDB/TiKV/TiFlashノードのメモリ容量バイト数 |

## FAQ

- 同じメトリックがGrafanaとTiDB Cloudコンソールで同時に異なる値を持つ理由は？

    GrafanaとTiDB Cloudでは集計計算ロジックが異なるため、表示される集約値が異なる可能性があります。より詳細なメトリクス値を取得するには、Grafanaの **mini step** 設定を調整できます。