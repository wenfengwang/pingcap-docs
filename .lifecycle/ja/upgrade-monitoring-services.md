---
title: クラスターモニタリングサービスのアップグレード
summary: TiDBクラスターのPrometheus、Grafana、Alertmanagerモニタリングサービスをアップグレードする方法を学びます。

# TiDBクラスターモニタリングサービスのアップグレード

TiDBクラスターをデプロイすると、TiUPは自動的にクラスターのモニタリングサービス（Prometheus、Grafana、Alertmanagerなど）をデプロイします。このクラスターをスケールアウトする場合、TiUPはスケーリング中に新しく追加されたノードのモニタリング構成も自動的に追加します。TiUPによって自動的にデプロイされるモニタリングサービスは通常、これらのサードパーティのモニタリングサービスの最新バージョンではありません。最新バージョンを使用するには、このドキュメントに従ってモニタリングサービスをアップグレードできます。

クラスターを管理する際に、TiUPはモニタリングサービスの構成を上書きするため、モニタリングサービスの構成ファイルを直接置き換えてアップグレードすると、クラスター上での任意のTiUP操作（`deploy`、`scale-out`、`scale-in`、`reload`など）がアップグレードを上書きし、エラーが発生する可能性があります。Prometheus、Grafana、Alertmanagerをアップグレードするには、構成ファイルを直接置き換えるのではなく、このドキュメントの手順に従ってください。

> **注意：**
>
> - もしモニタリングサービスを[TiUPを使用せずに手動でデプロイ](/deploy-monitoring-services.md)した場合は、このドキュメントを参照せずに直接アップグレードできます。
> - TiDBがより新しいバージョンのモニタリングサービスとの互換性が検証されていないため、アップグレード後に一部の機能が期待通りに動作しない可能性があります。問題がある場合は、GitHubの[issue](https://github.com/pingcap/tidb/issues)を作成してください。
> - このドキュメントのアップグレード手順は、TiUPバージョン1.9.0以降に適用されます。そのため、アップグレード前にTiUPのバージョンを確認してください。
> - TiUPを使用してTiDBクラスターをアップグレードする場合、TiUPはモニタリングサービスをデフォルトバージョンに再デプロイします。したがって、TiDBをアップグレードした後は、モニタリングサービスのアップグレードをもう一度実行する必要があります。

## Prometheusのアップグレード

TiDBとの互換性を向上させるためには、TiDBインストールパッケージに提供されているPrometheusのインストールパッケージを使用することをお勧めします。TiDBインストールパッケージ内のPrometheusのバージョンは固定されています。より新しいPrometheusバージョンを使用したい場合は、[Prometheusリリースノート](https://github.com/prometheus/prometheus/releases)を参照して各バージョンの新機能を確認し、本番環境に適したバージョンを選択できます。また、適切なバージョンについてはPingCAPの技術スタッフに相談することも可能です。

次のアップグレード手順では、Prometheusのウェブサイトから必要なバージョンのPrometheusインストールパッケージをダウンロードし、それをTiUPが使用できるようにする必要があります。

### Step 1. Prometheusの新しいインストールパッケージをPrometheusのウェブサイトからダウンロード

[Prometheusダウンロードページ](https://prometheus.io/download/)から新しいインストールパッケージをダウンロードし、それを解凍します。

### Step 2. TiDBが提供するPrometheusのインストールパッケージをダウンロード

1. [TiDBダウンロードページ](https://www.pingcap.com/download/)からTiDBの**Server Package**をダウンロードし、それを解凍します。
2. 解凍されたファイル内で、`prometheus-v{version}-linux-amd64.tar.gz`を見つけ、それを解凍します。

    ```bash
    tar -xzf prometheus-v{version}-linux-amd64.tar.gz
    ```

### Step 3. TiUPが使用できる新しいPrometheusパッケージを作成

1. [Step 1](#step-1-download-a-new-prometheus-installation-package-from-the-prometheus-website)で解凍されたファイルをコピーし、[Step 2](#step-2-download-the-prometheus-installation-package-provided-by-tidb)で解凍された`./prometheus-v{version}-linux-amd64/prometheus`ディレクトリ内のファイルを置き換えます。
2. `./prometheus-v{version}-linux-amd64`ディレクトリを再圧縮し、新しい圧縮パッケージを`prometheus-v{new-version}.tar.gz`と名前を付けます。`{new-version}`は必要に応じて指定できます。

    ```bash
    cd prometheus-v{version}-linux-amd64.tar.gz
    tar -zcvf ../prometheus-v{new-version}.tar.gz ./
    ```

### Step 4. 新しく作成したPrometheusパッケージを使用してPrometheusをアップグレード

以下のコマンドを実行してPrometheusをアップグレードします：

```bash
tiup cluster patch <cluster-name> prometheus-v{new-version}.tar.gz -R prometheus
```

アップグレード後に、Prometheusサーバーのホームページ（通常は`http://<Prometheus-server-host-name>:9090`）に移動し、トップナビゲーションメニューの**Status**をクリックしてから**Runtime & Build Information**ページを開き、Prometheusのバージョンを確認し、アップグレードが成功したかどうかを確認できます。

## Grafanaのアップグレード

TiDBとの互換性を向上させるためには、TiDBインストールパッケージに提供されているGrafanaのインストールパッケージを使用することをお勧めします。TiDBインストールパッケージ内のGrafanaのバージョンは固定されています。より新しいGrafanaバージョンを使用したい場合は、[Grafanaリリースノート](https://grafana.com/docs/grafana/latest/whatsnew/)を参照して各バージョンの新機能を確認し、本番環境に適したバージョンを選択できます。また、適切なバージョンについてはPingCAPの技術スタッフに相談することも可能です。

次のアップグレード手順では、必要なバージョンのGrafanaインストールパッケージをGrafanaのウェブサイトからダウンロードし、それをTiUPが使用できるようにする必要があります。

### Step 1. Grafanaの新しいインストールパッケージをGrafanaのウェブサイトからダウンロード

1. [Grafanaダウンロードページ](https://grafana.com/grafana/download?pg=get&plcmt=selfmanaged-box1-cta1)から新しいインストールパッケージをダウンロードします。必要に応じて`OSS`または`Enterprise`エディションを選択できます。
2. ダウンロードしたパッケージを解凍します。

### Step 2. TiDBが提供するGrafanaのインストールパッケージをダウンロード

1. [TiDBダウンロードページ](https://www.pingcap.com/download)からTiDBの**Server Package**をダウンロードし、それを解凍します。
2. 解凍されたファイル内で、`grafana-v{version}-linux-amd64.tar.gz`を見つけ、それを解凍します。

    ```bash
    tar -xzf grafana-v{version}-linux-amd64.tar.gz
    ```

### Step 3. TiUPが使用できる新しいGrafanaパッケージを作成

1. [Step 1](#step-1-download-a-new-grafana-installation-package-from-the-grafana-website)で解凍されたファイルをコピーし、[Step 2](#step-2-download-the-grafana-installation-package-provided-by-tidb)で解凍された`./grafana-v{version}-linux-amd64/`ディレクトリ内のファイルを置き換えます。
2. `./grafana-v{version}-linux-amd64`ディレクトリを再圧縮し、新しい圧縮パッケージを`grafana-v{new-version}.tar.gz`と名前を付けます。`{new-version}`は必要に応じて指定できます。

    ```bash
    cd grafana-v{version}-linux-amd64.tar.gz
    tar -zcvf ../grafana-v{new-version}.tar.gz ./
    ```

### Step 4. 新しく作成したGrafanaパッケージを使用してGrafanaをアップグレード

以下のコマンドを実行してGrafanaをアップグレードします：

```bash
tiup cluster patch <cluster-name> grafana-v{new-version}.tar.gz -R grafana

```

アップグレード後に、Grafanaサーバーのホームページに移動（通常は`http://<Grafana-server-host-name>:3000`）、ページでGrafanaのバージョンを確認してアップグレードが成功したかどうかを確認できます。

## Alertmanagerのアップグレード

TiDBインストールパッケージ内のAlertmanagerパッケージは、Prometheusのウェブサイトから直接提供されています。そのため、Alertmanagerをアップグレードする際は、新しいバージョンのAlertmanagerをPrometheusのウェブサイトからダウンロードしてインストールするだけで十分です。

### Step 1. Prometheusのウェブサイトから新しいAlertmanagerインストールパッケージをダウンロード

[Prometheusダウンロードページ](https://prometheus.io/download/#alertmanager)から`alertmanager`のインストールパッケージをダウンロードします。

### Step 2. ダウンロードしたインストールパッケージを使用してAlertmanagerをアップグレード

以下のコマンドを実行してAlertmanagerをアップグレードします：

```bash
tiup cluster patch <cluster-name> alertmanager-v{new-version}-linux-amd64.tar.gz -R alertmanager
```

アップグレード後に、Alertmanagerサーバーのホームページに移動（通常は`http://<Alertmanager-server-host-name>:9093`）、トップナビゲーションメニューで**Status**をクリックし、Alertmanagerのバージョンを確認してアップグレードが成功したかどうかを確認できます。