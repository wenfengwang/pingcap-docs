---
title: サードパーティメトリクス統合（ベータ版）
summary: サードパーティメトリクス統合の使用方法について学びます。

# サードパーティメトリクス統合（ベータ版）

TiDB Cloudをサードパーティメトリクスサービスと統合して、TiDB Cloudのアラートを受信し、メトリクスサービスを使用してTiDBクラスタのパフォーマンスメトリクスを表示することができます。現在、サードパーティメトリクス統合はベータ版です。

## 必要なアクセス権

サードパーティ統合設定を編集するには、組織の「組織所有者」ロールか、対象プロジェクトの「プロジェクト所有者」ロールにいる必要があります。

## サードパーティ統合の表示または変更

1. [TiDB Cloudコンソール](https://tidbcloud.com)にログインします。
2. 左下隅にある<MDSvgIcon name="icon-left-projects" />をクリックし、複数のプロジェクトがある場合は、対象のプロジェクトに切り替えてから**プロジェクト設定**をクリックします。
3. プロジェクトの**プロジェクト設定**ページで、左のナビゲーションペインで**統合**をクリックします。

利用可能なサードパーティ統合が表示されます。

## 制限事項

- [TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless) クラスタでは、サードパーティメトリクス統合はサポートされていません。

- クラスタのステータスが「作成中」、「復元中」、「一時停止中」、「再開中」の場合、サードパーティメトリクス統合は利用できません。

## 利用可能な統合

### Datadog 統合（ベータ版）

Datadog 統合を使用すると、TiDB Cloudを構成して、TiDBクラスタに関するメトリックデータを[Datadog](https://www.datadoghq.com/)に送信し、Datadogのダッシュボードでこれらのメトリクスを表示することができます。

詳細な統合手順とDatadogがトラッキングするメトリクスのリストについては、[TiDB CloudをDatadogと統合する](/tidb-cloud/monitor-datadog-integration.md)を参照してください。

### Prometheus と Grafana 統合（ベータ版）

Prometheus と Grafana 統合を使用すると、TiDB CloudからPrometheusのscrape_configファイルを取得し、そのファイルの内容を使用してPrometheusを構成できます。Grafanaのダッシュボードでこれらのメトリクスを表示できます。

詳細な統合手順とPrometheusがトラッキングするメトリクスのリストについては、[TiDB CloudをPrometheusとGrafanaと統合する](/tidb-cloud/monitor-prometheus-and-grafana-integration.md)を参照してください。

### New Relic 統合（ベータ版）

New Relic 統合を使用すると、TiDB Cloudを構成して、TiDBクラスタに関するメトリックデータを[New Relic](https://newrelic.com/)に送信し、New Relicのダッシュボードでこれらのメトリクスを表示することができます。

詳細な統合手順とNew Relicがトラッキングするメトリクスのリストについては、[TiDB CloudをNew Relicと統合する](/tidb-cloud/monitor-new-relic-integration.md)を参照してください。