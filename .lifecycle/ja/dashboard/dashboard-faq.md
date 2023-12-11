---
title: TiDBダッシュボードFAQ
summary: TiDBダッシュボードに関するよくある質問（FAQ）とその回答をまとめたトゥビメント
aliases: ['/docs/dev/dashboard/dashboard-faq/']
---

# TiDBダッシュボードFAQ

このドキュメントは、TiDBダッシュボードに関するよくある質問（FAQ）とその回答をまとめたものです。問題の特定ができず、また指示に従っても解消しない場合は、[PingCAPのサポート](/support.md) またはコミュニティからサポートを受けてください。

## アクセスに関連するFAQ

### ファイアウォールまたはリバースプロキシが構成されていると、TiDBダッシュボード以外の内部アドレスにリダイレクトされる

クラスターに複数のPlacement Driver（PD）インスタンスが展開されている場合、PDインスタンスのうちの1つだけが実際にTiDBダッシュボードサービスを実行します。この1つではない他のPDインスタンスにアクセスすると、Webブラウザが別のアドレスにリダイレクトします。ファイアウォールまたはリバースプロキシがTiDBダッシュボードにアクセスするために正しく構成されていないと、ダッシュボードを訪れるとファイアウォールまたはリバースプロキシによって保護された内部アドレスにリダイレクトされる場合があります。

- 複数のPDインスタンスを使用したTiDBダッシュボードのデプロイに関する詳細は、[TiDBダッシュボードの複数PDインスタンスの展開](/dashboard/dashboard-ops-deploy.md)を参照してください。
- 正しくリバースプロキシを構成する方法については、[リバースプロキシを介してTiDBダッシュボードを使用する](/dashboard/dashboard-ops-reverse-proxy.md)を参照してください。
- ファイアウォールを正しく構成する方法については、[TiDBダッシュボードのセキュリティを確保する](/dashboard/dashboard-ops-security.md)を参照してください。

### ダブルネットワーク インターフェイス カード（NIC）でTiDBダッシュボードが展開された場合、他のNICを使用してTiDBダッシュボードにアクセスできない

セキュリティ上の理由から、PD上のTiDBダッシュボードは展開時に指定されたIPアドレスのみ（つまり、`0.0.0.0`ではなく）でのみ監視します。したがって、ホストに複数のNICがインストールされている場合、他のNICを使用してTiDBダッシュボードにアクセスすることはできません。

`tiup cluster`または`tiup playground`コマンドを使用してTiDBを展開した場合、現在この問題を解決することはできません。他のNICに安全にTiDBダッシュボードを公開するには、リバースプロキシを使用することをお勧めします。詳細については、[リバースプロキシを介してTiDBダッシュボードを利用する](/dashboard/dashboard-ops-reverse-proxy.md)を参照してください。

## UIに関連するFAQ

### 概要ページの**QPS**および**レイテンシ**セクションで`prometheus_not_found`エラーが表示される

**概要**ページの**QPS**および**レイテンシ**セクションでは、Prometheusが展開されたクラスターが必要です。そうでない場合、エラーが表示されます。この問題は、クラスターにPrometheusインスタンスを展開することで解決できます。

Prometheusインスタンスが展開されているにもかかわらず、この問題が発生する場合、デプロイツールが古い（TiUPまたはTiDB Operator）、そしてツールが自動的にメトリクスアドレスを報告しないため、TiDBダッシュボードがメトリクスを問い合わせることができません。デプロイツールを最新バージョンにアップグレードしてもう一度お試しください。

デプロイツールがTiUPの場合、次の手順を実行してこの問題を解決できます。他のデプロイツールについては、それらのツールの対応するドキュメントを参照してください。

1. TiUPとTiUP Clusterをアップグレードします：

    {{< copyable "shell-regular" >}}

    ```bash
    tiup update --self
    tiup update cluster --force
    ```

2. アップグレード後、Prometheusインスタンスが展開された新しいクラスターでは、メトリクスが通常に表示されます。

3. アップグレード後の既存のクラスターでは、このクラスターを再起動してメトリクスアドレスを報告できます。`CLUSTER_NAME`を実際のクラスター名に置き換えてください：

    {{< copyable "shell-regular" >}}

    ```bash
    tiup cluster start CLUSTER_NAME
    ```

   クラスターが起動されている場合でも、このコマンドを実行します。このコマンドはクラスターの通常のアプリケーションに影響しませんが、メトリクスアドレスを更新して報告するため、TiDBダッシュボードで監視メトリクスを通常に表示できます。

### **遅いクエリ**ページで`invalid connection`エラーが表示される

TiDBの実験的な機能として、準備済みプランキャッシュ機能が有効になっている可能性があります。有効になっている場合、特定のTiDBバージョンで準備されたプランキャッシュは正常に機能しない場合があり、これがTiDBダッシュボード（および他のアプリケーション）でこの問題を引き起こす可能性があります。システム変数[`tidb_enable_prepared_plan_cache = OFF`](/system-variables.md#tidb_enable_prepared_plan_cache-new-in-v610)を設定することで、準備済みプランキャッシュを無効にできます。

### **遅いクエリ**ページで`required component NgMonitoring is not started`エラーが表示される

NgMonitoringは、TiDBクラスターのv5.4.0以降のバージョンで構築されたTiDBダッシュボードの機能（**Continuous Profiling**および**Top SQL**）をサポートするための高度なモニタリングコンポーネントです。NgMonitoringは、新しいバージョンのTiUPでクラスターをデプロイまたはアップグレードすると自動的に展開されます。TiDB Operatorを使用して展開されたクラスターの場合、[Continuous Profilingを有効にする](https://docs.pingcap.com/tidb-in-kubernetes/dev/access-dashboard/#enable-continuous-profiling)を参照して、NgMonitoringを手動で展開できます。

Webページに`required component NgMonitoring is not started` と表示される場合は、次の手順でデプロイの問題をトラブルシューティングできます：

<details>
  <summary>TiUPを使用してデプロイされたクラスター</summary>

ステップ1. バージョンを確認します

1. TiUPクラスターバージョンを確認します。NgMonitoringは、TiUPがv1.9.0以降の場合にのみ展開されます。

    {{< copyable "shell-regular" >}}

    ```shell
    tiup cluster --version
    ```

    コマンドの出力にTiUPのバージョンが表示されます。例：

    ```
    tiup version 1.9.0 tiup
    Go Version: go1.17.2
    Git Ref: v1.9.0
    ```

2. TiUPクラスターバージョンがv1.9.0より前の場合、TiUPとTiUPクラスターを最新バージョンにアップグレードします：

    {{< copyable "shell-regular" >}}

    ```shell
    tiup update --all
    ```

ステップ2. TiUPを使用して制御マシンにng_port構成項目を追加し、その後Prometheusを再読み込みします。

1. 編集モードでクラスター構成ファイルを開きます：

    {{< copyable "shell-regular" >}}

    ```shell
    tiup cluster edit-config ${cluster-name}
    ```

2. `monitoring_servers`の下に`ng_port:12020`パラメータを追加します：

    ```
    monitoring_servers:
    - host: 172.16.6.6
      ng_port: 12020
    ```

3. Prometheusを再読み込みします：

    {{< copyable "shell-regular" >}}

    ```shell
    tiup cluster reload ${cluster-name} --role prometheus
    ```

上記の手順を実行してもエラーメッセージが表示される場合は、[PingCAPのサポート](/support.md) またはコミュニティからサポートを受けてください。

</details>

<details>
  <summary>TiDB Operatorを使用してデプロイされたクラスター</summary>

TiDB Operatorドキュメント内の[Continuous Profilingを有効にする](https://docs.pingcap.com/tidb-in-kubernetes/dev/access-dashboard/#enable-continuous-profiling)セクションの手順に従って、NgMonitoringコンポーネントを展開します。

</details>

<details>
  <summary>TiUP Playgroundで開始されたクラスター</summary>

クラスターを開始すると、TiUP Playground（v1.8.0以上）は自動的にNgMonitoringコンポーネントを開始します。TiUP Playgroundを最新バージョンに更新するには、次のコマンドを実行します：

{{< copyable "shell-regular" >}}

```shell
tiup update --self
tiup update playground
```

</details>

### **遅いクエリ**ページで`unknown field`エラーが表示される

クラスターのアップグレード後に**遅いクエリ**ページで`unknown field`エラーが表示される場合、そのエラーはTiDBダッシュボードサーバーフィールド（更新される可能性がある）とユーザープリファレンスフィールド（ブラウザキャッシュ中のフィールド）の違いによる互換性の問題に関連しています。この問題は修正されています。クラスターがv5.0.3またはv4.0.14より前の場合は、ブラウザキャッシュをクリアするために次の手順を実行します：

1. TiDBダッシュボードページを開きます。

2. Developer Toolsを開きます。異なるブラウザには異なるDeveloper Toolsを開く方法があります。**Menuバー**をクリックした後：

    - Firefox: **Menu** > **Web Developer** > **Toggle Tools**、または**Tools** > **Web Developer** > **Toggle Tools**。
    - Chrome: **More tools** > **Developer tools**。
    - Safari: **Develop** > **Show Web Inspector**。 **Develop**メニューが表示されない場合は、**Safari** > **Preferences** > **Advanced**に移動し、メニューバー内の**Show Develop**メニューをチェックします。

    以下の例では、Chromeを使用しています。

    ![ChromeメインメニューからDevToolsを開く](/media/dashboard/dashboard-faq-devtools.png)

3. **Application**パネルを選択し、**Local Storage**メニューを展開し、**TiDBダッシュボードページドメイン**を選択します。**Clear All**ボタンをクリックします。

    ![Local Storageをクリア](/media/dashboard/dashboard-faq-devtools-application.png)