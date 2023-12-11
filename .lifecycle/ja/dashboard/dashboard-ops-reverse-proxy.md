---
title: リバースプロキシを使用してTiDB Dashboardを利用する
aliases: ['/docs/dev/dashboard/dashboard-ops-reverse-proxy/']
---

# リバースプロキシを使用してTiDB Dashboardを利用する

内部ネットワークから外部にTiDB Dashboardサービスを安全に公開するために、リバースプロキシを使用できます。

## 手順

### ステップ1: 実際のTiDB Dashboardアドレスを取得する

クラスタに複数のPDインスタンスが展開されている場合、PDインスタンスのうちの1つだけが実際にTiDB Dashboardを実行します。そのため、リバースプロキシのアップストリームが正しいアドレスを指すようにする必要があります。このメカニズムの詳細については、[複数のPDインスタンスを展開する](/dashboard/dashboard-ops-deploy.md#deployment-with-multiple-pd-instances)を参照してください。

TiUPツールを使用して展開する場合は、次のコマンドを実行して実際のTiDB Dashboardアドレスを取得します（`CLUSTER_NAME`をクラスタ名に置き換えます）：

{{< copyable "shell-regular" >}}

```shell
tiup cluster display CLUSTER_NAME --dashboard
```

出力されるのは実際のTiDB Dashboardアドレスです。サンプルを以下に示します：

```bash
http://192.168.0.123:2379/dashboard/
```

> **注意:**
>
> この機能は、`tiup cluster`展開ツールの後のバージョン（v1.0.3以降）でのみ利用可能です。
>
> <details>
> <summary>TiUP Clusterをアップグレードする</summary>
>
> ```bash
> tiup update --self
> tiup update cluster --force
> ```
>
> </details>

### ステップ2: リバースプロキシを設定する

<details>
<summary> <strong>HAProxyを使用する</strong> </summary>

[HAProxy](https://www.haproxy.org/)をリバースプロキシとして使用する場合、次の手順を実行します：

1. `8033`ポート（例）でTiDB Dashboardのリバースプロキシを使用します。HAProxy構成ファイルに次の構成を追加します：

    {{< copyable "" >}}

    ```haproxy
    frontend tidb_dashboard_front
      bind *:8033
      use_backend tidb_dashboard_back if { path /dashboard } or { path_beg /dashboard/ }

    backend tidb_dashboard_back
      mode http
      server tidb_dashboard 192.168.0.123:2379
    ```

    [ステップ1](#step-1-get-the-actual-tidb-dashboard-address)で取得したTiDB Dashboardの実際のアドレスのIPとポート（`192.168.0.123:2379`）を置き換えます。

    > **警告:**
    >
    > サービスが**このパス内だけ**にあることを確実にするために、`use_backend`ディレクティブの`if`部分を保持する必要があります。さもなければ、セキュリティリスクが発生する可能性があります。[TiDB Dashboardをセキュアにする](/dashboard/dashboard-ops-security.md)を参照してください。

2. 構成が有効になるようにHAProxyを再起動します。

3. リバースプロキシが有効かどうかをテストします：HAProxyが設置されているマシンの`8033`ポートで`/dashboard/`アドレスにアクセスして（`http://example.com:8033/dashboard/`など）、TiDB Dashboardにアクセスします。

</details>

<details>
<summary> <strong>NGINXを使用する</strong> </summary>

[NGINX](https://nginx.org/)をリバースプロキシとして使用する場合、次の手順を実行します：

1. `8033`ポート（例）でTiDB Dashboardのリバースプロキシを使用します。NGINX構成ファイルに次の構成を追加します：

    {{< copyable "" >}}

    ```nginx
    server {
        listen 8033;
        location /dashboard/ {
        proxy_pass http://192.168.0.123:2379/dashboard/;
        }
    }
    ```

    [ステップ1](#step-1-get-the-actual-tidb-dashboard-address)で取得したTiDB Dashboardの実際のアドレスを`http://192.168.0.123:2379/dashboard/`で置き換えます。

    > **警告:**
    >
    > `proxy_pass`ディレクティブの`/dashboard/`パスを保持して、このパス内のサービスのみがリバースプロキシされるようにします。さもなければ、セキュリティリスクが発生します。[TiDB Dashboardをセキュアにする](/dashboard/dashboard-ops-security.md)を参照してください。

2. 構成が有効になるようにNGINXを再読み込みします。

    {{< copyable "shell-regular" >}}

    ```shell
    sudo nginx -s reload
    ```

3. リバースプロキシが有効かどうかをテストします：NGINXが設置されているマシンの`8033`ポートで`/dashboard/`アドレスにアクセスして（`http://example.com:8033/dashboard/`など）、TiDB Dashboardにアクセスします。

</details>

## パスプレフィックスをカスタマイズする

TiDB Dashboardはデフォルトで`/dashboard/`パスでサービスを提供しますが、リバースプロキシを使用している場合も同様です（`http://example.com:8033/dashboard/`など）。リバースプロキシを使用してTiDB Dashboardサービスをデフォルトでないパスで提供する場合（`http://example.com:8033/foo/`または`http://example.com:8033/`など）、次の手順を実行します。

### ステップ1: PD設定を修正してTiDB Dashboardサービスのパスプレフィックスを指定する

PD構成の`[dashboard]`カテゴリ内の`public-path-prefix`構成項目を修正して、TiDB Dashboardサービスのパスプレフィックスを指定します。この項目を変更した後は、変更が有効になるようにPDインスタンスを再起動します。

例えば、クラスタがTiUPを使用して展開されており、サービスを`http://example.com:8033/foo/`で実行したい場合は、次の構成を指定できます：

{{< copyable "" >}}

```yaml
server_configs:
  pd:
    dashboard.public-path-prefix: /foo
```

<details>
<summary> <strong>TiUPを使用して新しいクラスタを展開する場合の構成の修正</strong> </summary>

新しいクラスタを展開する場合は、`topology.yaml` TiUPトポロジファイルに上記の構成を追加し、クラスタを展開できます。詳細な手順については、[TiUPデプロイメントドキュメント](/production-deployment-using-tiup.md#step-3-initialize-cluster-topology-file)を参照してください。

</details>

<details>

<summary> <strong>TiUPを使用して展開されたクラスタの構成の修正</strong> </summary>

展開済みのクラスタの場合：

1. クラスタの構成ファイルを編集モードで開きます（`CLUSTER_NAME`をクラスタ名に置き換えます）。

    {{< copyable "shell-regular" >}}

    ```shell
    tiup cluster edit-config CLUSTER_NAME
    ```

2. `server_configs`の下の`pd`構成に構成項目を修正または追加します。`server_configs`が存在しない場合は、トップレベルで追加します：

    {{< copyable "" >}}

    ```yaml
    monitored:
      ...
    server_configs:
      tidb: ...
      tikv: ...
      pd:
        dashboard.public-path-prefix: /foo
      ...
    ```

    修正後の構成ファイルは、次のファイルに似ています：

    {{< copyable "" >}}

    ```yaml
    server_configs:
      pd:
        dashboard.public-path-prefix: /foo
      global:
        user: tidb
        ...
    ```

    または

    {{< copyable "" >}}

    ```yaml
    monitored:
      ...
    server_configs:
      tidb: ...
      tikv: ...
      pd:
        dashboard.public-path-prefix: /foo
    ```

3. 修正された構成が有効になるように、すべてのPDインスタンスに対してローリングリスタートを実行します（`CLUSTER_NAME`をクラスタ名に置き換えます）：

    {{< copyable "shell-regular" >}}

    ```shell
    tiup cluster reload CLUSTER_NAME -R pd
    ```

詳細については、[Common TiUP Operations - Modify the configuration](/maintain-tidb-using-tiup.md#modify-the-configuration)を参照してください。

</details>

TiDB Dashboardサービスがルートパスで実行されるようにしたい場合（`http://example.com:8033/`など）、次の構成を使用します：

{{< copyable "" >}}

```yaml
server_configs:
  pd:
    dashboard.public-path-prefix: /
```

> **警告:**
>
> 変更されたカスタマイズされたパスプレフィックスが有効になると、TiDB Dashboardに直接アクセスすることはできません。パスプレフィックスに一致するリバースプロキシを介してのみアクセスできます。

### ステップ2: リバースプロキシの構成を変更する

<details>
<summary> <strong>HAProxyを使用する</strong> </summary>

`http://example.com:8033/foo/`を例にすると、対応するHAProxy構成は次のようになります：

{{< copyable "" >}}

```haproxy
frontend tidb_dashboard_front
  bind *:8033
  use_backend tidb_dashboard_back if { path /foo } or { path_beg /foo/ }

backend tidb_dashboard_back
  mode http
  http-request set-path %[path,regsub(^/foo/?,/dashboard/)]
  server tidb_dashboard 192.168.0.123:2379
```

[ステップ1](#step-1-get-the-actual-tidb-dashboard-address)で取得したTiDB Dashboardの実際のアドレス（`192.168.0.123:2379`）を置き換えます。

> **警告:**

```markdown
> `use_backend`ディレクティブの`if`部分を保持して、このパスにのみサービスがリバースプロキシの背後にあることを確認してください。それ以外の場合、セキュリティリスクが発生する可能性があります。[Secure TiDB Dashboard](/dashboard/dashboard-ops-security.md)を参照してください。

TiDB Dashboardサービスをルートパス（たとえば`http://example.com:8033/`）で実行したい場合は、次の構成を使用してください：

```haproxy
frontend tidb_dashboard_front
  bind *:8033
  use_backend tidb_dashboard_back
backend tidb_dashboard_back
  mode http
  http-request set-path /dashboard%[path]
  server tidb_dashboard 192.168.0.123:2379
```

構成を変更し、変更した構成が有効になるようにHAProxyを再起動してください。

</details>

<details>
<summary> <strong>NGINXを使用</strong> </summary>

`http://example.com:8033/foo/`を例に取ると、対応するNGINXの構成は次のとおりです：

{{< copyable "" >}}

```nginx
server {
  listen 8033;
  location /foo/ {
    proxy_pass http://192.168.0.123:2379/dashboard/;
  }
}
```

`proxy_pass`ディレクティブの中の`http://192.168.0.123:2379/dashboard/`を、[Step 1](#step-1-get-the-actual-tidb-dashboard-address)で取得した実際のTiDB Dashboardアドレスに置き換えてください。

> **警告：**
>
> `proxy_pass`ディレクティブの中の`/dashboard/`パスが保持されることを確認してください。これにより、このパスにのみサービスがリバースプロキシの背後にあることが保証されます。それ以外の場合、セキュリティリスクが発生する可能性があります。[Secure TiDB Dashboard](/dashboard/dashboard-ops-security.md)を参照してください。

TiDB Dashboardサービスをルートパスで実行したい場合（たとえば`http://example.com:8033/`）、次の構成を使用してください：

{{< copyable "" >}}

```nginx
server {
  listen 8033;
  location / {
    proxy_pass http://192.168.0.123:2379/dashboard/;
  }
}
```

構成を変更し、変更した構成が有効になるようにNGINXを再起動してください。

{{< copyable "shell-regular" >}}

```shell
sudo nginx -s reload
```

</details>

## 次のステップ

ファイアウォールの構成など、TiDB Dashboardのセキュリティを強化する方法について学ぶには、[Secure TiDB Dashboard](/dashboard/dashboard-ops-security.md)を参照してください。
```