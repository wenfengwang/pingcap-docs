---
title: モニタリングサーバーの設定のカスタマイズ
summary: TiUPによって管理されるモニタリングサーバーの構成のカスタマイズ方法について学びます

# モニタリングサーバーの設定のカスタマイズ

TiUPを使用してTiDBクラスターを展開すると、Prometheus、Grafana、Alertmanagerなどのモニタリングサーバーも展開されます。また、このクラスターをスケールアウトする場合、TiUPは新しいノードを監視対象に含めます。

上記のモニタリングサーバーの設定をカスタマイズするには、TiDBクラスターのtopology.yamlファイルに関連する構成項目を追加する以下の手順に従うことができます。

> **注意:**
>
> - モニタリングサーバーの構成ファイルを直接変更しないでください。後続のTiUP操作（展開、スケールアウト、スケールイン、および再読み込み）によってこれらの変更は上書きされます。
>
> - モニタリングサーバーがTiUPによって展開および管理されていない場合、このドキュメントを参照せずにモニタリングサーバーの構成ファイルを直接変更できます。
>
> - この機能はTiUP v1.9.0以降でサポートされています。したがって、この機能を使用する前にTiUPのバージョンを確認してください。

## Prometheusの設定のカスタマイズ

現在、TiUPではPrometheusルールおよびスクレイプ構成ファイルのカスタマイズがサポートされています。

### Prometheusルールの設定のカスタマイズ

1. ルール構成ファイルをカスタマイズし、TiUPが配置されているマシンのディレクトリに配置します。

2. topology.yamlファイルで、`rule_dir`をカスタマイズされたルール構成ファイルのディレクトリに設定します。

    次は、topology.yamlファイルのmonitoring_serversの構成例です:

    ```
    # Server configs are used to specify the configuration of Prometheus Server.
    monitoring_servers:
      # The ip address of the Monitoring Server.
    - host: 127.0.0.1
      rule_dir: /home/tidb/prometheus_rule   # TiUPマシン上のPrometheusルールディレクトリ
    ```

前述の構成が完了すると、TiUPはカスタマイズされたルール設定を`rule_dir`（例： `/home/tidb/prometheus_rule`）から読み込み、Prometheusサーバーに送信してデフォルトのルール設定を置き換えます。

### Prometheusスクレイプ構成のカスタマイズ

1. TiDBクラスターのtopology.yamlファイルを開きます。

2. `monitoring_servers`構成に`additional_scrape_conf`フィールドを追加します。

    次は、topology.yamlファイルのmonitoring_serversの構成例です:

    ```
    monitoring_servers:
    - host: xxxxxxx
    ssh_port: 22
    port: 9090
    deploy_dir: /tidb-deploy/prometheus-9090
    data_dir: /tidb-data/prometheus-9090
    log_dir: /tidb-deploy/prometheus-9090/log
    external_alertmanagers: []
    arch: amd64
    os: linux
    additional_scrape_conf:
      metric_relabel_configs:
        - source_labels: [__name__]
            separator: ;
            regex: tikv_thread_nonvoluntary_context_switches|tikv_thread_voluntary_context_switches|tikv_threads_io_bytes_total
            action: drop
        - source_labels: [__name__,name]
            separator: ;
            regex: tikv_thread_cpu_seconds_total;(tokio|rocksdb).+
            action: drop
    ```

前述の構成が完了すると、TiUPはTiDBクラスターを展開、スケールアウト、スケールイン、または再読み込みする場合、`additional_scrape_conf`フィールドをPrometheus構成ファイルの対応するパラメータに追加します。

## Grafanaの設定のカスタマイズ

現在、TiUPではGrafanaダッシュボードおよびその他の設定のカスタマイズがサポートされています。

### Grafanaダッシュボードのカスタマイズ

1. Grafanaダッシュボードの構成ファイルをカスタマイズし、TiUPが配置されているマシンのディレクトリに配置します。

2. topology.yamlファイルで、`dashboard_dir`をカスタマイズされたダッシュボード構成ファイルのディレクトリに設定します。

    次は、topology.yamlファイルのgrafana_serversの構成例です:

    ```
    # Server configs are used to specify the configuration of Grafana Servers.
    grafana_servers:
      # The ip address of the Grafana Server.
     - host: 127.0.0.1
     dashboard_dir: /home/tidb/dashboards   # TiUPマシン上のGrafanaダッシュボードディレクトリ
    ```

前述の構成が完了すると、TiUPはカスタマイズされたダッシュボード設定を`dashboard_dir`（例： `/home/tidb/dashboards`）から読み込み、Grafanaサーバーに関連の構成を送信してデフォルトのダッシュボード設定を置き換えます。

### その他のGrafana設定のカスタマイズ

1. TiDBクラスターのtopology.yamlファイルを開きます。

2. `grafana_servers`構成に他の構成項目を追加します。

    次は、topology.yamlファイルの`[log.file] level`および`smtp`フィールドの構成例です:

    ```
    # Server configs are used to specify the configuration of Grafana Servers.
    grafana_servers:
    # The ip address of the Grafana Server.
    - host: 127.0.0.1
        config:
        log.file.level: warning
        smtp.enabled: true
        smtp.host: {IP}:{port}
        smtp.user: example@pingcap.com
        smtp.password: {password}
        smtp.skip_verify: true
    ```

前述の構成が完了すると、TiUPはTiDBクラスターを展開、スケールアウト、スケールイン、または再読み込みする場合、`config`フィールドをGrafana構成ファイル`grafana.ini`に追加します。

## Alertmanagerの設定のカスタマイズ

現在、TiUPではAlertmanagerのリスニングアドレスのカスタマイズがサポートされています。

TiUPによって展開されたAlertmanagerはデフォルトで`alertmanager_servers.host`をリスンします。プロキシを使用すると、Alertmanagerにアクセスできません。この問題を解決するために、`topology.yaml`に`listen_host`を追加してリスンアドレスを指定できます。推奨値は0.0.0.0です。

次は、`listen_host`フィールドを0.0.0.0に設定する例です:

```
alertmanager_servers:
  # The ip address of the Alertmanager Server.
  - host: 172.16.7.147
    listen_host: 0.0.0.0
    # SSH port of the server.
    ssh_port: 22
```

前述の構成が完了すると、TiUPはTiDBクラスターを展開、スケールアウト、スケールイン、または再読み込みする場合、Alertmanagerの起動パラメーター`--web.listen-address`に`listen_host`フィールドを追加します。