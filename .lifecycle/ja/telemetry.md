---
title: テレメトリ
summary: テレメトリ機能の学習方法、機能の無効化方法、およびその状態の表示方法について学ぶ
aliases: ['/docs/dev/telemetry/']
---

# テレメトリ

テレメトリが有効になっている場合、TiDB、TiUP、TiDB Dashboardは使用状況情報を収集し、PingCAPと共有します。これにより、製品の改善方法を理解するのに役立ちます。たとえば、この使用状況情報は新機能の優先順位付けに役立ちます。

> **注記：**
>
> - 2023年2月20日以降、テレメトリ機能はTiDBとTiDB Dashboardの新バージョン、v6.6.0を含む、デフォルトで無効になります。使用状況情報は収集されず、PingCAPと共有されません。これらのバージョンにアップグレードする前に、クラスタがデフォルトのテレメトリ構成を使用している場合、アップグレード後にテレメトリ機能は無効になります。詳細なバージョンについては、[TiDBリリースタイムライン](/releases/release-timeline.md)を参照してください。
> - v1.11.3以降、新規にデプロイされたTiUPでは、デフォルトでテレメトリ機能が無効になり、使用状況情報は収集されません。v1.11.3以前のTiUPバージョンからv1.11.3以降のバージョンにアップグレードする場合、テレメトリ機能はアップグレード前と同じ状態になります。

## 共有される内容

以下のセクションでは、各コンポーネントについて共有される使用状況情報を詳しく説明しています。共有される使用状況の詳細は変わる可能性があります。これらの変更がある場合は、[リリースノート](/releases/release-notes.md)でアナウンスされます。

> **注記：**
>
> **どの場合にも、TiDBクラスタに格納されているユーザーデータは共有されません。また、[PingCAPプライバシーポリシー](https://pingcap.com/privacy-policy)もご参照ください。**

### TiDB

TiDBでテレメトリ収集機能が有効になっている場合、TiDBクラスタは6時間ごとに使用状況の詳細を収集します。これらの使用状況の詳細には、次のものが含まれますが、これらに限定されません。

- ランダムに生成されたテレメトリID。
- デプロイの特性（CPU、メモリ、ディスクのサイズ、TiDBコンポーネントのバージョン、OS名）。
- システム内のクエリリクエストのステータス（クエリリクエストの数と期間など）。
- コンポーネントの使用状況、たとえばAsync Commit機能の使用有無など。
- TiDBのテレメトリデータ送信元の匿名化されたIPアドレス。

PingCAPに共有される使用状況情報の詳細内容を表示するには、次のSQLステートメントを実行します：

{{< copyable "sql" >}}

```sql
ADMIN SHOW TELEMETRY;
```

### TiDB Dashboard

TiDB Dashboardでテレメトリ収集機能が有効にされている場合、TiDB Dashboard Web UIの使用状況の詳細が共有されます。これには、次の情報が含まれますが、これらに限定されません。

- ランダムに生成されたテレメトリID。
- ユーザーの操作情報、例えば、ユーザーがアクセスしたTiDB Dashboard Webページの名前。
- ブラウザとOS情報、ブラウザ名、OS名、および画面解像度など。

PingCAPに共有される使用状況情報の詳細内容を表示するには、[Chrome DevToolsのネットワークアクティビティインスペクタ](https://developers.google.com/web/tools/chrome-devtools/network)または[Firebaseのネットワークモニタ](https://developer.mozilla.org/en-US/docs/Tools/Network_Monitor)を使用してください。

### TiUP

TiUPでテレメトリ収集機能が有効になっている場合、TiUPの使用状況の詳細が共有されます。これには、次の情報が含まれますが、これに限定されません。

- ランダムに生成されたテレメトリID。
- TiUPコマンドの実行ステータス、実行が成功したかどうか、実行期間など。
- ハードウェアのサイズ、TiDBコンポーネントのバージョン、および変更されたデプロイ構成名などのデプロイの特性。

PingCAPに共有される使用状況情報の詳細内容を表示するには、TiUPコマンドを実行する際に`TIUP_CLUSTER_DEBUG=enable`環境変数を設定してください。例：

{{< copyable "shell-regular" >}}

```shell
TIUP_CLUSTER_DEBUG=enable tiup cluster list
```

### TiSpark

> **注記：**
>
> v3.0.3以降、テレメトリ収集はTiSparkでデフォルトで無効になります。使用状況情報は収集されず、PingCAPと共有されません。

TiSparkでテレメトリ収集機能が有効になっている場合、Sparkモジュールは、TiSparkの使用状況の詳細が共有されます。これには、次の情報が含まれますが、これに限定されません。

- ランダムに生成されたテレメトリID。
- TiSparkのいくつかの設定情報、たとえば、リードエンジン、およびストリーミングリードが有効かどうか。
- クラスタのデプロイ情報、たとえば、マシンのハードウェア情報、OS情報、およびTiSparkが配置されているノードのコンポーネントバージョン番号。

TiSparkの収集された使用状況情報はSparkログで表示できます。次のようにSparkログレベルをINFOまたはそれ以下に設定できます。

```shell
cat {spark.log} | grep Telemetry report | tail -n 1
```

## テレメトリの無効化

### デプロイ時のTiDBテレメトリの無効化

既存のTiDBクラスタでテレメトリが有効になっている場合は、各TiDBインスタンスに`enable-telemetry = false`を設定して、そのインスタンスでTiDBテレメトリの収集を無効にできます。これはクラスタを再起動するまで有効になりません。

異なるデプロイメントツールでのテレメトリの無効化の詳細な手順については次に示します。

<details>
  <summary>バイナリデプロイメント</summary>

次の内容で構成ファイル `tidb_config.toml` を作成してください：

{{< copyable "" >}}

```toml
enable-telemetry = false
```

上記の構成ファイルの設定が有効になるように、TiDBを起動する際に`--config=tidb_config.toml`コマンドラインパラメータを指定してください。

詳細については、[TiDBコンフィギュレーションオプション](/command-line-flags-for-tidb-configuration.md#--config)および[TiDBコンフィギュレーションファイル](/tidb-configuration-file.md#enable-telemetry-new-in-v402)を参照してください。

</details>

<details>
  <summary>TiUP Playgroundを使用したデプロイメント</summary>

次の内容で構成ファイル `tidb_config.toml` を作成してください：

{{< copyable "" >}}

```toml
enable-telemetry = false
```

TiUP Playgroundを起動する際に、`--db.config tidb_config.toml`コマンドラインパラメータを指定して、上記の構成ファイルの設定が有効になるようにしてください。例：

{{< copyable "shell-regular" >}}

```shell
tiup playground --db.config tidb_config.toml
```

詳細については、[ローカルTiDBクラスタの迅速なデプロイ](/tiup/tiup-playground.md)を参照してください。

</details>

<details>
  <summary>TiUP Clusterを使用したデプロイメント</summary>

次の内容を含むデプロイメントトポロジファイル `topology.yaml` を修正してください：

{{< copyable "" >}}

```yaml
server_configs:
  tidb:
    enable-telemetry: false
```

</details>

<details>
  <summary>TiDB Operatorを介したKubernetes上のデプロイメント</summary>

`tidb-cluster.yaml`またはTidbClusterカスタムリソースで`spec.tidb.config.enable-telemetry: false`を設定してください。

詳細については、[Kubernetes上でTiDB Operatorをデプロイ](https://docs.pingcap.com/tidb-in-kubernetes/stable/deploy-tidb-operator)を参照してください。

> **注記：**
>
> この設定項目は、効果を発揮するためにTiDB Operator v1.1.3以上が必要です。

</details>

### デプロイ済みのTiDBクラスタでのTiDBテレメトリの無効化

既存のTiDBクラスタで、システム変数[`tidb_enable_telemetry`](/system-variables.md#tidb_enable_telemetry-new-in-v402)を変更して、動的にTiDBテレメトリの収集を無効にできます：

{{< copyable "sql" >}}

```sql
SET GLOBAL tidb_enable_telemetry = 0;
```

> **注記：**
>
> テレメトリを無効化すると、システム変数は構成ファイルよりも優先度が高くなります。つまり、テレメトリの収集が構成ファイルによって無効にされると、システム変数の値は無視されます。

### TiDB Dashboardのテレメトリの無効化

すべてのPDインスタンスでTiDB Dashboardのテレメトリ収集を無効にするには、[`dashboard.enable-telemetry = false`](/pd-configuration-file.md#enable-telemetry)を設定してください。設定が有効になるには実行中のクラスタを再起動する必要があります。

異なるデプロイメントツールでのテレメトリの無効化の詳細な手順については次に示します。

<details>
  <summary>バイナリデプロイメント</summary>

次の内容で構成ファイル `pd_config.toml` を作成してください：

{{< copyable "" >}}

```toml
[dashboard]
enable-telemetry = false
```

上記の構成ファイルの設定が有効になるように、PDを起動する際に`--config=pd_config.toml`コマンドラインパラメータを指定してください。

詳細については、[PDコンフィギュレーションフラグ](/command-line-flags-for-pd-configuration.md#--config)と[PDコンフィギュレーションファイル](/pd-configuration-file.md#enable-telemetry)を参照してください。

</details>

<details>
```
<summary>TiUPクラスタを使用した展開</summary>

次の内容を追加して、デプロイメントトポロジーファイル`topology.yaml`を変更します：

{{< copyable "" >}}

```yaml
server_configs:
  pd:
    dashboard.enable-telemetry: false
```

</details>

<details>
  <summary>TiDB Operatorを使用したKubernetes上での展開</summary>

`tidb-cluster.yaml`またはTidbClusterカスタムリソースで`spec.pd.config.dashboard.enable-telemetry: false`を構成します。詳細については、[Kubernetes上でTiDB Operatorを展開する](https://docs.pingcap.com/tidb-in-kubernetes/stable/deploy-tidb-operator)を参照してください。

> **注意:**
>
> この構成項目には、TiDB Operator v1.1.3以降で有効になります。

</details>

### TiUPテレメトリの無効化

TiUPテレメトリの収集を無効にするには、次のコマンドを実行します：

{{< copyable "shell-regular" >}}

```shell
tiup telemetry disable
```

## テレメトリステータスを確認

TiDBテレメトリに対して、次のSQLステートメントを実行してテレメトリステータスを確認します：

{{< copyable "sql" >}}

```sql
ADMIN SHOW TELEMETRY;
```

実行結果の`DATA_PREVIEW`列が空の場合、TiDBテレメトリは無効になっています。それ以外の場合は、TiDBテレメトリは有効になっています。また、`LAST_STATUS`列に従って以前にいつ使用情報が共有されたか、共有が成功したかどうかも確認できます。

TiUPテレメトリに対して、次のコマンドを実行してテレメトリステータスを確認します：

{{< copyable "shell-regular" >}}

```shell
tiup telemetry status
```

## コンプライアンス

異なる国や地域のコンプライアンス要件を満たすために、使用情報は送信元マシンのIPアドレスに応じて異なる国のサーバに送信されます：

- 中国本土からのIPアドレスの場合、使用情報は中国本土のクラウドサーバに送信され、そこに保存されます。
- 中国本土外からのIPアドレスの場合、使用情報は米国のクラウドサーバに送信され、そこに保存されます。

詳細については、[PingCAPプライバシーポリシー](https://en.pingcap.com/privacy-policy/)を参照してください。
```