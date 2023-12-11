---
title: TiDBクラスターの監視サービスをデプロイする
summary: TiDBクラスターの監視サービスをデプロイする方法について学びます。
aliases: ['/docs/dev/deploy-monitoring-services/','/docs/dev/how-to/monitor/monitor-a-cluster/','/docs/dev/monitor-a-tidb-cluster/']
---

# TiDBクラスターの監視サービスをデプロイする

このドキュメントは、TiDBの監視およびアラートサービスを手動でデプロイしたいユーザーを対象としています。

TiUPを使用してTiDBクラスターをデプロイした場合、監視およびアラートサービスは自動的にデプロイされ、手動でのデプロイは不要です。

## PrometheusおよびGrafanaをデプロイする

TiDBクラスタートポロジが以下のようであると仮定します:

| 名前   | ホストIP      | サービス     |
| :-- | :-- | :-------------- |
| Node1 | 192.168.199.113| PD1, TiDB, node_export, Prometheus, Grafana |
| Node2 | 192.168.199.114| PD2, node_export  |
| Node3 | 192.168.199.115| PD3, node_export |
| Node4 | 192.168.199.116| TiKV1, node_export |
| Node5 | 192.168.199.117| TiKV2, node_export |
| Node6 | 192.168.199.118| TiKV3, node_export |

### ステップ1: バイナリパッケージをダウンロードする

{{< copyable "shell-regular" >}}

```bash
# パッケージをダウンロードします。
wget https://download.pingcap.org/prometheus-2.27.1.linux-amd64.tar.gz
wget https://download.pingcap.org/node_exporter-v1.3.1-linux-amd64.tar.gz
wget https://download.pingcap.org/grafana-7.5.11.linux-amd64.tar.gz
```

{{< copyable "shell-regular" >}}

```bash
# パッケージを展開します。
tar -xzf prometheus-2.27.1.linux-amd64.tar.gz
tar -xzf node_exporter-v1.3.1-linux-amd64.tar.gz
tar -xzf grafana-7.5.11.linux-amd64.tar.gz
```

### ステップ2: Node1、Node2、Node3、Node4上で`node_exporter`を起動する

{{< copyable "shell-regular" >}}

```bash
cd node_exporter-v1.3.1-linux-amd64

# node_exporterサービスを起動します。
$ ./node_exporter --web.listen-address=":9100" \
    --log.level="info" &
```

### ステップ3: Node1上でPrometheusを起動する

Prometheusの設定ファイルを編集します。

{{< copyable "shell-regular" >}}

```bash
cd prometheus-2.27.1.linux-amd64 &&
vi prometheus.yml
```

```ini
...

global:
  scrape_interval:     15s  # デフォルトでは、15秒ごとにターゲットをスクレイプします。
  evaluation_interval: 15s  # デフォルトでは、15秒ごとにターゲットをスクレイプします。
  # scrape_timeoutはグローバルのデフォルト値（10秒）に設定されています。
  external_labels:
    cluster: 'test-cluster'
    monitor: "prometheus"

scrape_configs:
  - job_name: 'overwritten-nodes'
    honor_labels: true  # ジョブおよびインスタンスラベルを上書きしません。
    static_configs:
    - targets:
      - '192.168.199.113:9100'
      - '192.168.199.114:9100'
      - '192.168.199.115:9100'
      - '192.168.199.116:9100'
      - '192.168.199.117:9100'
      - '192.168.199.118:9100'

  - job_name: 'tidb'
    honor_labels: true  # ジョブおよびインスタンスラベルを上書きしません。
    static_configs:
    - targets:
      - '192.168.199.113:10080'

  - job_name: 'pd'
    honor_labels: true  # ジョブおよびインスタンスラベルを上書きしません。
    static_configs:
    - targets:
      - '192.168.199.113:2379'
      - '192.168.199.114:2379'
      - '192.168.199.115:2379'

  - job_name: 'tikv'
    honor_labels: true  # ジョブおよびインスタンスラベルを上書きしません。
    static_configs:
    - targets:
      - '192.168.199.116:20180'
      - '192.168.199.117:20180'
      - '192.168.199.118:20180'

...

```

Prometheusサービスを起動します。

```bash
$ ./prometheus \
    --config.file="./prometheus.yml" \
    --web.listen-address=":9090" \
    --web.external-url="http://192.168.199.113:9090/" \
    --web.enable-admin-api \
    --log.level="info" \
    --storage.tsdb.path="./data.metrics" \
    --storage.tsdb.retention="15d" &
```

### ステップ4: Node1上でGrafanaを起動する

Grafanaの設定ファイルを編集します。

{{< copyable "shell-regular" >}}

```ini
cd grafana-7.5.11 &&
vi conf/grafana.ini

...

[paths]
data = ./data
logs = ./data/log
plugins = ./data/plugins
[server]
http_port = 3000
domain = 192.168.199.113
[database]
[session]
[analytics]
check_for_updates = true
[security]
admin_user = admin
admin_password = admin
[snapshots]
[users]
[auth.anonymous]
[auth.basic]
[auth.ldap]
[smtp]
[emails]
[log]
mode = file
[log.console]
[log.file]
level = info
format = text
[log.syslog]
[event_publisher]
[dashboards.json]
enabled = false
path = ./data/dashboards
[metrics]
[grafana_net]
url = https://grafana.net

...

```

Grafanaサービスを起動します。

{{< copyable "shell-regular" >}}

```bash
./bin/grafana-server \
    --config="./conf/grafana.ini" &
```

## Grafanaの構成

このセクションでは、Grafanaの構成方法について説明します。

### ステップ1: Prometheusデータソースを追加

1. Grafana Webインタフェースにログインします。

    - デフォルトアドレス: [http://localhost:3000](http://localhost:3000)
    - デフォルトアカウント: admin
    - デフォルトパスワード: admin

    > **注意:**
    >
    > **パスワードの変更**ステップでは、**スキップ**を選択できます。

2. Grafanaサイドバーメニューで、**Configuration**内の**Data Source**をクリックします。

3. **Add data source**をクリックします。

4. データソース情報を指定します。

    - データソースの**名前**を指定します。
    - **Type**には**Prometheus**を選択します。
    - **URL**にはPrometheusのアドレスを指定します。
    - 必要に応じて他のフィールドを指定します。

5. 新しいデータソースを保存するには**Add**をクリックします。

### ステップ2: Grafanaダッシュボードをインポート

PDサーバー、TiKVサーバー、およびTiDBサーバー用のGrafanaダッシュボードをインポートするために、それぞれ以下の手順を実行します:

1. グラフェナロゴをクリックしてサイドバーメニューを開きます。

2. サイドバーメニューで、**Dashboards** -> **Import**をクリックして**Import Dashboard**ウィンドウを開きます。

3. **Upload .json File**をクリックしてJSONファイルをアップロードします（TiDB Grafana構成ファイルは[pingcap/tidb](https://github.com/pingcap/tidb/tree/master/pkg/metrics/grafana)から、TiKV Grafana構成ファイルは[tikv/tikv](https://github.com/tikv/tikv/tree/master/metrics/grafana)から、PD Grafana構成ファイルは[tikv/pd](https://github.com/tikv/pd/tree/master/metrics/grafana)からダウンロードします）。

    > **注意:**
    >
    > TiKV、PD、およびTiDBのダッシュボードには、それぞれ`{tikv_summary.json}`, `{tikv_details.json}`, `{tikv_trouble_shooting.json}`, `{pd.json}`, `{tidb.json}`, および `{tidb_summary.json}`という対応するJSONファイルがあります。

4. **Load**をクリックします。

5. Prometheusデータソースを選択します。

6. **Import**をクリックします。Prometheusダッシュボードがインポートされます。

## コンポーネントメトリクスの表示

トップメニューで**New dashboard**をクリックし、表示したいダッシュボードを選択します。

![view dashboard](/media/view-dashboard.png)

クラスターコンポーネントの以下のメトリクスを取得できます:

+ **TiDBサーバー:**

    - レイテンシとスループットを監視するためのクエリ処理時間
    - DDLプロセスのモニタリング
    - TiKVクライアントに関する監視
    - PDクライアントに関する監視

+ **PDサーバー:**

    - コマンドの実行回数の合計
    - 特定のコマンドの失敗回数の合計
    - コマンドの成功時の実行時間
    - コマンドの失敗時の実行時間
    - コマンドの完了および結果の戻り時間

+ **TiKVサーバー:**

    - ガベージコレクション（GC）のモニタリング
- TiKVコマンドが実行された回数の合計
- スケジューラがコマンドを実行する期間
- Raft提案コマンドの合計回数
- Raftがコマンドを実行する期間
- Raftコマンドの失敗回数の合計
- Raftがレディ状態を処理した合計回数