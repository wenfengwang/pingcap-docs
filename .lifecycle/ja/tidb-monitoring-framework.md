---
title: TiDB モニタリングフレームワークの概要
summary: Prometheus と Grafana を使用して TiDB モニタリングフレームワークを構築します。
aliases: ['/docs/dev/tidb-monitoring-framework/', '/docs/dev/how-to/monitor/overview/']
---

# TiDB モニタリングフレームワークの概要

TiDB モニタリングフレームワークは、2つのオープンソースプロジェクト、Prometheus と Grafana を採用しています。TiDB は、[Prometheus](https://prometheus.io) を使用してモニタリングとパフォーマンスメトリクスを保存し、 [Grafana](https://grafana.com/grafana) を使用してこれらのメトリクスを視覚化しています。

## TiDB における Prometheus について

Prometheus は時系列データベースとして、多次元のデータモデルと柔軟なクエリ言語を持っています。最も人気のあるオープンソースプロジェクトの1つとして、Prometheus は多くの企業や組織に採用され、非常にアクティブなコミュニティを持っています。PingCAP は、TiDB、TiKV、PD におけるモニタリングとアラートのために Prometheus をアクティブに開発・採用しています。

Prometheus には複数のコンポーネントが含まれています。現在、TiDB は以下のコンポーネントを使用しています:

- Prometheus サーバー: 時系列データをスクレイプして保存
- アプリケーション内で必要なメトリクスをカスタマイズするためのクライアントライブラリ
- アラートメカニズムのための Alertmanager

次の図は、その構造を示しています:

![diagram](/media/prometheus-in-tidb.png)

## TiDB における Grafana について

Grafana は、メトリクスの分析と可視化のためのオープンソースプロジェクトです。TiDB は、以下のように Grafana を使用してパフォーマンスメトリクスを表示しています:

![Grafana monitored_groups](/media/grafana-monitored-groups.png)

- {TiDB_Cluster_name}-Backup-Restore: バックアップとリストアに関連するモニタリングメトリクス。
- {TiDB_Cluster_name}-Binlog: TiDB バイナリログに関連するモニタリングメトリクス。
- {TiDB_Cluster_name}-Blackbox_exporter: ネットワークプローブに関連するモニタリングメトリクス。
- {TiDB_Cluster_name}-Disk-Performance: ディスクパフォーマンスに関連するモニタリングメトリクス。
- {TiDB_Cluster_name}-Kafka-Overview: Kafka に関連するモニタリングメトリクス。
- {TiDB_Cluster_name}-Lightning: TiDB Lightning に関連するモニタリングメトリクス。
- {TiDB_Cluster_name}-Node_exporter: オペレーティングシステムに関連するモニタリングメトリクス。
- {TiDB_Cluster_name}-Overview: 重要なコンポーネントに関連するモニタリング概要。
- {TiDB_Cluster_name}-PD: PD サーバーに関連するモニタリングメトリクス。
- {TiDB_Cluster_name}-Performance-Read: 読み取りパフォーマンスに関連するモニタリングメトリクス。
- {TiDB_Cluster_name}-Performance-Write: 書き込みパフォーマンスに関連するモニタリングメトリクス。
- {TiDB_Cluster_name}-TiDB: TiDB サーバーに関連する詳細なモニタリングメトリクス。
- {TiDB_Cluster_name}-TiDB-Summary: TiDB に関連するモニタリング概要。
- {TiDB_Cluster_name}-TiFlash-Proxy-Summary: TiFlash へのデータ複製に使用されるプロキシサーバーに関連するモニタリング概要。
- {TiDB_Cluster_name}-TiFlash-Summary: TiFlash に関連するモニタリング概要。
- {TiDB_Cluster_name}-TiKV-Details: TiKV サーバーに関連する詳細なモニタリングメトリクス。
- {TiDB_Cluster_name}-TiKV-Summary: TiKV サーバーに関連するモニタリング概要。
- {TiDB_Cluster_name}-TiKV-Trouble-Shooting: TiKV エラー診断に関連するモニタリングメトリクス。
- {TiDB_Cluster_name}-TiCDC: TiCDC に関連する詳細なモニタリングメトリクス。

各グループには複数のモニタリングメトリクスのパネルラベルがあり、各パネルには複数のモニタリングメトリクスの詳細情報が含まれています。たとえば、「**Overview**」モニタリンググループには5つのパネルラベルがあり、それぞれがモニタリングパネルに対応しています。次の UI を参照してください:

![Grafana Overview](/media/grafana-monitor-overview.png)