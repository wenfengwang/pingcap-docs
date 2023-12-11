---
title: Overview Page
summary: TiDB Dashboardの概要ページを学びます。
aliases: ['/docs/dev/dashboard/dashboard-overview/']
---

# 概要ページ

このページでは、TiDBクラスタ全体の概要が表示されます。以下の情報が含まれます：

- クラスタ全体の1秒当たりのクエリ数（QPS）。
- クラスタ全体のクエリ遅延時間。
- 直近の期間において最も長い実行時間を累積したSQLステートメント。
- 最近の期間において閾値を超えた遅いクエリ。
- 各インスタンスのノード数とステータス。
- モニターおよびアラートメッセージ。

## ページへのアクセス

TiDB Dashboardにログインした後、概要ページにはデフォルトで入ります。または、左側のナビゲーションメニューから**概要**をクリックしてこのページに入ることができます：

![概要ページへの入り方](/media/dashboard/dashboard-overview-access-v650.png)

## QPS

このエリアでは、直近1時間におけるクラスタ全体の成功および失敗したクエリ数が表示されます：

![QPS](/media/dashboard/dashboard-overview-qps.png)

> **ノート：**
>
> この機能は、Prometheusモニタリングコンポーネントがデプロイされているクラスタでのみ利用可能です。Prometheusがデプロイされていない場合、エラーが表示されます。

## 遅延時間

このエリアでは、クラスタ全体のクエリの99.9%、99%、および90%の遅延時間が表示されます：

![遅延時間](/media/dashboard/dashboard-overview-latency.png)

> **ノート：**
>
> この機能は、Prometheusモニタリングコンポーネントがデプロイされているクラスタでのみ利用可能です。Prometheusがデプロイされていない場合、エラーが表示されます。

## 上位SQLステートメント

このエリアでは、直近の期間にクラスタ全体で最も長い実行時間を累積した10種類のSQLステートメントが表示されます。異なるクエリパラメータを持つが同じ構造のSQLステートメントは、同じSQLタイプに分類されて同じ行に表示されます：

![上位SQL](/media/dashboard/dashboard-overview-top-statements.png)

このエリアに表示される情報は、より詳細な[SQLステートメントページ](/dashboard/dashboard-statement-list.md)と一致しています。**上位SQLステートメント**の見出しをクリックして、完全なリストを表示することができます。このテーブルの列の詳細については、[SQLステートメントページ](/dashboard/dashboard-statement-list.md)を参照してください。

> **ノート：**
>
> この機能は、SQLステートメント機能が有効になっているクラスタでのみ利用可能です。

## 直近の遅いクエリ

デフォルトでは、このエリアには直近30分間におけるクラスタ全体での最新の10件の遅いクエリが表示されます：

![直近の遅いクエリ](/media/dashboard/dashboard-overview-slow-query.png)

デフォルトでは、300ミリ秒よりも長く実行されるSQLクエリは遅いクエリとしてカウントされ、テーブルに表示されます。この閾値は、[tidb_slow_log_threshold](/system-variables.md#tidb_slow_log_threshold)変数または[instance.tidb_slow_log_threshold](/tidb-configuration-file.md#tidb_slow_log_threshold) TiDBパラメータを変更することで調整できます。

このエリアに表示される内容は、より詳細な[遅いクエリページ](/dashboard/dashboard-slow-query.md)と一致しています。**直近の遅いクエリ**の見出しをクリックして、完全なリストを表示することができます。このテーブルの列の詳細については、[遅いクエリページ](/dashboard/dashboard-slow-query.md)を参照してください。

> **ノート：**
>
> この機能は、遅いクエリログが有効になっているクラスタでのみ利用可能です。デフォルトでは、TiUPを使用して展開されたクラスタで遅いクエリログが有効になっています。

## インスタンス

このエリアでは、TiDB、TiKV、PD、およびTiFlashの合計インスタンス数と異常なインスタンス数がクラスタ全体でまとめられています：

![インスタンス](/media/dashboard/dashboard-overview-instances.png)

前述の画像で示したステータスは次の通りです：

- Up: インスタンスは正常に稼働しています（オフラインストレージインスタンスを含む）。
- Down: インスタンスは異常に稼働しており、ネットワークの切断やプロセスのクラッシュなどが発生しています。

**インスタンス**の見出しをクリックして、各インスタンスの詳細な稼働状況が表示される[クラスタ情報ページ](/dashboard/dashboard-cluster-info.md)に入ることができます。

## モニターおよびアラート

このエリアでは、詳細なモニターおよびアラートを表示するためのリンクが提供されます：

![モニターおよびアラート](/media/dashboard/dashboard-overview-monitor.png)

- **メトリクスを表示**：このリンクをクリックすると、クラスタの詳細な監視情報を表示できるGrafanaダッシュボードに移動します。Grafanaダッシュボードの各監視メトリクスの詳細については、[監視メトリクス](/grafana-overview-dashboard.md)を参照してください。
- **アラートを表示**：このリンクをクリックすると、クラスタの詳細なアラート情報を表示できるAlertManagerページに移動します。クラスタ内にアラートが存在する場合、その数はリンクテキストに直接表示されます。
- **診断を実行**：このリンクをクリックすると、より詳細な[クラスタ診断ページ](/dashboard/dashboard-diagnostics-access.md)に移動します。

> **ノート：**
>
> **メトリクスを表示**リンクは、Grafanaノードが展開されているクラスタでのみ利用可能です。**アラートを表示**リンクは、AlertManagerノードが展開されているクラスタでのみ利用可能です。