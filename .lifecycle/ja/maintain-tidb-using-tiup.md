---
title: TiUP共通操作
summary: TiUPを使用してTiDBクラスタを操作および管理する際の共通操作について学びます。
aliases: ['/docs/dev/maintain-tidb-using-tiup/','/docs/dev/how-to/maintain/tiup-operations/']
---

# TiUP共通操作

このドキュメントでは、TiUPを使用してTiDBクラスタを操作および管理する際の以下の共通操作について説明します。

- クラスタリストを表示する
- クラスタを起動する
- クラスタの状態を表示する
- 構成を変更する
- クラスタを停止する
- クラスタを削除する

## クラスタリストを表示する

TiUPクラスタコンポーネントを使用して複数のTiDBクラスタを管理できます。TiDBクラスタが展開されると、クラスタはTiUPクラスタリストに表示されます。

リストを表示するには、次のコマンドを実行します。

{{< copyable "shell-regular" >}}

```bash
tiup cluster list
```

## クラスタを起動する

TiDBクラスタのコンポーネントは、以下の順序で起動されます:

**PD > TiKV > Pump > TiDB > TiFlash > Drainer > TiCDC > Prometheus > Grafana > Alertmanager**

クラスタを起動するには、次のコマンドを実行します。

{{< copyable "shell-regular" >}}

```bash
tiup cluster start ${cluster-name}
```

> **注意:**
>
> `${cluster-name}`をクラスタの名前に置き換えてください。クラスタ名を忘れた場合は、`tiup cluster list`を実行して確認してください。

コマンドに`-R`または`-N`パラメータを追加することで、一部のコンポーネントのみを起動できます。例:

- このコマンドはPDコンポーネントのみを起動します:

    {{< copyable "shell-regular" >}}

    ```bash
    tiup cluster start ${cluster-name} -R pd
    ```

- このコマンドは`1.2.3.4`および`1.2.3.5`ホスト上のPDコンポーネントのみを起動します:

    {{< copyable "shell-regular" >}}

    ```bash
    tiup cluster start ${cluster-name} -N 1.2.3.4:2379,1.2.3.5:2379
    ```

> **注意:**
>
> `-R`または`-N`パラメータを使用して指定したコンポーネントを起動する場合は、起動順序が正しいことを確認してください。例えば、TiKVコンポーネントの前にPDコンポーネントを起動してください。さもなければ、起動に失敗する可能性があります。

## クラスタの状態を表示する

クラスタを起動した後、各コンポーネントの状態を確認して正常に動作していることを確認してください。TiUPには`display`コマンドが用意されているため、各マシンにログインしてコンポーネントの状態を確認する必要はありません。

{{< copyable "shell-regular" >}}

```bash
tiup cluster display ${cluster-name}
```

## 構成を変更する

クラスタが稼働中の場合、コンポーネントのパラメータを変更する必要がある場合は、`edit-config`コマンドを実行します。具体的な手順は次の通りです:

1. クラスタの構成ファイルを編集モードで開きます:

    {{< copyable "shell-regular" >}}

    ```bash
    tiup cluster edit-config ${cluster-name}
    ```

2. パラメータを構成します:

    - コンポーネント全体にグローバルに影響する設定の場合、`server_configs`を編集します:

        ```
        server_configs:
          tidb:
            log.slow-threshold: 300
        ```

    - 特定のノードに影響する設定の場合は、ノードの`config`内で構成を編集します:

        ```
        tidb_servers:
        - host: 10.0.1.11
          port: 4000
          config:
              log.slow-threshold: 300
        ```

    パラメータのフォーマットについては、[TiUPパラメータテンプレート](https://github.com/pingcap/tiup/blob/master/embed/examples/cluster/topology.example.yaml)を参照してください。

    **構成項目の階層を表すには、`.`を使用します**。

    コンポーネントの構成パラメータの詳細については、[TiDB `config.toml.example`](https://github.com/pingcap/tidb/blob/master/pkg/config/config.toml.example)、[TiKV `config.toml.example`](https://github.com/tikv/tikv/blob/master/etc/config-template.toml)、[PD `config.toml.example`](https://github.com/tikv/pd/blob/master/conf/config.toml)を参照してください。

3. `reload`コマンドを実行して構成をローリング更新し、対応するコンポーネントを再起動します:

    {{< copyable "shell-regular" >}}

    ```bash
    tiup cluster reload ${cluster-name} [-N <nodes>] [-R <roles>]
    ```

### 例

TiDBサーバでトランザクションサイズ制限パラメータ(`performance`モジュールの`txn-total-size-limit`[performance](https://github.com/pingcap/tidb/blob/master/pkg/config/config.toml.example)に`1G`という値を設定する場合は、次のように構成を編集します:

```
server_configs:
  tidb:
    performance.txn-total-size-limit: 1073741824
```

その後、`tiup cluster reload ${cluster-name} -R tidb`コマンドを実行してTiDBコンポーネントをローリング再起動します。

## ホットフィックスパッケージで置き換える

通常のアップグレードについては、[TiUPを使用したTiDBのアップグレード](/upgrade-tidb-using-tiup.md)を参照してください。ただし、デバッグなどのシナリオでは、現在実行中のコンポーネントを一時的なパッケージで置き換える必要がある場合があります。これを実現するには、`patch`コマンドを使用します。

{{< copyable "shell-root" >}}

```bash
tiup cluster patch --help
```

```
指定したパッケージでリモートパッケージを置き換えてサービスを再起動します

使用方法:
  cluster patch <cluster-name> <package-path> [flags]

フラグ:
  -h, --help                   patchのヘルプ
  -N, --node strings           ノードを指定します
      --overwrite              このパッケージを将来のスケールアウト操作で使用します
  -R, --role strings           役割を指定します
      --transfer-timeout int   PDおよびTiKVストアリーダーを転送する際のタイムアウト(デフォルト 600)

グローバルフラグ:

      --native-ssh        システムのネイティブSSHクライアントを使用します
      --wait-timeout int  操作の待機タイムアウト
      --ssh-timeout int   SSH接続時のタイムアウト秒数を指定します、SSH接続が不要な操作には無視されます。(デフォルト 5)
  -y, --yes               すべての確認をスキップして「はい」と仮定します
```

TiDBのホットフィックスパッケージが`/tmp/tidb-hotfix.tar.gz`にあり、クラスタ内のすべてのTiDBパッケージを置き換えたい場合は、次のコマンドを実行してください:

{{< copyable "shell-regular" >}}

```bash
tiup cluster patch test-cluster /tmp/tidb-hotfix.tar.gz -R tidb
```

また、クラスタ内の1つのTiDBパッケージのみを置き換えることもできます:

{{< copyable "shell-regular" >}}

```bash
tiup cluster patch test-cluster /tmp/tidb-hotfix.tar.gz -N 172.16.4.5:4000
```

## クラスタの名前を変更する

クラスタを展開して起動した後、`tiup cluster rename`コマンドを使用してクラスタの名前を変更できます:

{{< copyable "shell-regular" >}}

```bash
tiup cluster rename ${cluster-name} ${new-name}
```

> **注意:**
>
> + クラスタの名前を変更する操作は監視システム(PrometheusおよびGrafana)を再起動します。
> + クラスタが名前を変更した後、一部のパネルに古いクラスタ名が残る可能性があります。これらは手動で削除する必要があります。

## クラスタを停止する

TiDBクラスタのコンポーネントは、以下の順序で停止されます(監視コンポーネントも停止されます):

**Alertmanager > Grafana > Prometheus > TiCDC > Drainer > TiFlash > TiDB > Pump > TiKV > PD**

クラスタを停止するには、次のコマンドを実行します:

{{< copyable "shell-regular" >}}

```bash
tiup cluster stop ${cluster-name}
```

`start`コマンドと同様に、`stop`コマンドも`-R`または`-N`パラメータを追加して一部のコンポーネントを停止することができます。例:

- このコマンドはTiDBコンポーネントのみを停止します:

    {{< copyable "shell-regular" >}}

    ```bash
    tiup cluster stop ${cluster-name} -R tidb
    ```

- このコマンドは`1.2.3.4`および`1.2.3.5`ホスト上のTiDBコンポーネントのみを停止します:

    {{< copyable "shell-regular" >}}

    ```bash
    tiup cluster stop ${cluster-name} -N 1.2.3.4:4000,1.2.3.5:4000
    ```

## クラスタデータをクリーンアップする

クラスタデータをクリーンアップする操作は、すべてのサービスを停止し、データディレクトリおよび/またはログディレクトリをクリーンアップします。この操作は元に戻すことができないので、**慎重に**進めてください。

- クラスタ内のすべてのサービスのデータをクリーンアップしますが、ログは保持します:

    {{< copyable "shell-regular" >}}

    ```bash
    tiup cluster clean ${cluster-name} --data
    ```

- クラスタ内のすべてのサービスのログをクリーンアップしますが、データは保持します:
    {{< コピー可能 "shell-regular" >}}

    ```bash
    tiup cluster clean ${cluster-name} --log
    ```

- クラスタ内のすべてのサービスのデータとログをクリアします：

    {{< コピー可能 "shell-regular" >}}

    ```bash
    tiup cluster clean ${cluster-name} --all
    ```

- Prometheusを除くすべてのサービスのログとデータをクリアします：

    {{< コピー可能 "shell-regular" >}}

    ```bash
    tiup cluster clean ${cluster-name} --all --ignore-role prometheus
    ```

- `172.16.13.11:9000` インスタンスを除くすべてのサービスのログとデータをクリアします：

    {{< コピー可能 "shell-regular" >}}

    ```bash
    tiup cluster clean ${cluster-name} --all --ignore-node 172.16.13.11:9000
    ```

- `172.16.13.12` ノードを除くすべてのサービスのログとデータをクリアします：

    {{< コピー可能 "shell-regular" >}}

    ```bash
    tiup cluster clean ${cluster-name} --all --ignore-node 172.16.13.12
    ```

## クラスタを破棄

破棄操作はサービスを停止し、データディレクトリと展開ディレクトリをクリアします。この操作は元に戻すことができませんので、**注意して**進めてください。

{{< コピー可能 "shell-regular" >}}

```bash
tiup cluster destroy ${cluster-name}
```