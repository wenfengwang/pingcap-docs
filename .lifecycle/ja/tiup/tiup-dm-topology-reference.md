---
title: TiUPを使用したDMクラスタデプロイメントのためのトポロジ構成ファイル

# TiUPを使用したDMクラスタデプロイメントのためのトポロジ構成ファイル

TiDB Data Migration（DM）クラスタを展開またはスケールするには、クラスタトポロジを記述するためのトポロジファイル（[サンプル](https://github.com/pingcap/tiup/blob/master/embed/examples/dm/topology.example.yaml)）を提供する必要があります。

同様に、クラスタトポロジを変更するには、トポロジファイルを変更する必要があります。異なる点は、クラスタが展開された後は、トポロジファイルの一部のフィールドのみを変更できることです。このドキュメントでは、トポロジファイルの各セクションとそれぞれのセクションの各フィールドについて紹介します。

## ファイル構造

TiUPを使用してDMクラスタを展開するためのトポロジ構成ファイルには、以下のセクションが含まれる場合があります。

- [global](#global): クラスタのグローバル構成。一部の構成項目はクラスタのデフォルト値を使用し、各インスタンスで個別に構成できます。
- [server_configs](#server_configs): コンポーネントのグローバル構成。各コンポーネントを個別に構成できます。インスタンスで同じキーの構成項目がある場合、インスタンスの構成項目が適用されます。
- [master_servers](#master_servers): DM-masterインスタンスの構成。この構成は、DMコンポーネントのマスターサービスが展開されるマシンを指定します。
- [worker_servers](#worker_servers): DM-workerインスタンスの構成。この構成は、DMコンポーネントのワーカーサービスが展開されるマシンを指定します。
- [monitoring_servers](#monitoring_servers): Prometheusインスタンスが展開されるマシンを指定します。TiUPは複数のPrometheusインスタンスを展開できますが、最初のインスタンスのみが使用されます。
- [grafana_servers](#grafana_servers): Grafanaインスタンスの構成。この構成は、Grafanaインスタンスが展開されるマシンを指定します。
- [alertmanager_servers](#alertmanager_servers): Alertemanagerインスタンスの構成。この構成は、Alertmanagerインスタンスが展開されるマシンを指定します。

### `global`

`global`セクションはクラスタのグローバル構成に対応し、以下のフィールドを持ちます：

- `user`: 展開されたクラスタを開始するユーザ。デフォルト値は "tidb" です。`<user>`フィールドで指定されたユーザがターゲットマシン上に存在しない場合、TiUPはユーザの自動作成を試みます。
- `group`: 自動作成されるユーザが所属するユーザグループ。デフォルト値は `<user>` フィールドと同じです。指定したグループが存在しない場合、自動的に作成されます。
- `ssh_port`: オペレーションを行うためにターゲットマシンに接続するためのSSHポート。デフォルト値は "22" です。
- `deploy_dir`: 各コンポーネントの展開ディレクトリ。デフォルト値は "deploy" です。構築ルールは以下の通りです：
    - インスタンスレベルで絶対パスの `deploy_dir` が構成されている場合、実際の展開ディレクトリはインスタンス用に構成された `deploy_dir` です。
    - インスタンスごとに`deploy_dir`を構成しない場合、デフォルト値は相対パスの `<component-name>-<component-port>` です。
    - `global.deploy_dir`が絶対パスに設定されている場合、コンポーネントは `<global.deploy_dir>/<instance.deploy_dir>` ディレクトリに展開されます。
    - `global.deploy_dir`が相対パスに設定されている場合、コンポーネントは `/home/<global.user>/<global.deploy_dir>/<instance.deploy_dir>` ディレクトリに展開されます。
- `data_dir`: データディレクトリ。デフォルト値は "data" です。構築ルールは以下の通りです。
    - インスタンスレベルで絶対パスの `data_dir` が構成されている場合、実際のデータディレクトリはインスタンス用に構成された `data_dir` です。
    - インスタンスごとに`data_dir`を構成しない場合、デフォルト値は `<global.data_dir>` です。
    - `data_dir`が相対パスに設定されている場合、コンポーネントのデータは`<deploy_dir>/<data_dir>`に格納されます。`<deploy_dir>`の構築ルールについては、`deploy_dir`フィールドの構築ルールを参照してください。
- `log_dir`: データディレクトリ。デフォルト値は "log" です。構築ルールは以下の通りです。
    - インスタンスレベルで絶対パスの `log_dir` が構成されている場合、実際のログディレクトリはインスタンス用に構成された `log_dir` です。
    - ユーザによってインスタンスごとに`log_dir`が構成されない場合、デフォルト値は `<global.log_dir>` です。
    - `log_dir` が相対パスの場合、コンポーネントのログは`<deploy_dir>/<log_dir>`に格納されます。`<deploy_dir>`の構築ルールについては、`deploy_dir`フィールドの構築ルールを参照してください。
- `os`: ターゲットマシンのオペレーティングシステム。このフィールドは、ターゲットマシンにプッシュされるコンポーネントが適応するオペレーティングシステムを制御します。デフォルト値は "linux" です。
- `arch`: ターゲットマシンのCPUアーキテクチャ。このフィールドは、ターゲットマシンにプッシュされるバイナリパッケージが適応するプラットフォームを制御します。サポートされる値は "amd64" と "arm64" です。デフォルト値は "amd64" です。
- `resource_control`: ランタイムのリソース管理。このフィールドのすべての構成はsystemdのサービスファイルに記述されます。デフォルトで制限はありません。制御できるリソースは以下の通りです：
    - `memory_limit`: ランタイムでの最大メモリ制限。たとえば、"2G" は最大2 GBのメモリを使用できることを意味します。
    - `cpu_quota`: ランタイムでの最大CPU使用量を制限します。たとえば、"200%" です。
    - `io_read_bandwidth_max`: ディスクリードの最大I/O帯域幅を制限します。たとえば、 `"/dev/disk/by-path/pci-0000:00:1f.2-scsi-0:0:0:0:0 100M"` です。
    - `io_write_bandwidth_max`: ディスクライトの最大I/O帯域幅を制限します。たとえば、 `"/dev/disk/by-path/pci-0000:00:1f.2-scsi-0:0:0:0:0 100M"` です。
    - `limit_core`: コアダンプのサイズを制御します。

`global`の構成例：

```yaml
global:
  user: "tidb"
  resource_control:
    memory_limit: "2G"
```

この例では、構成は`tidb`ユーザを使用してクラスタを開始し、各コンポーネントを動作させる際に最大2 GBのメモリ制限を設定しています。

### `server_configs`

`server_configs`はサービスを構成し、各コンポーネントの構成ファイルを生成するために使用されます。`global`セクションと同様に、`server_configs`セクションの構成はインスタンスで同じキーを持つ構成によって上書きできます。`server_configs`には主に以下のフィールドが含まれます：

- `master`: DM-masterサービスに関連する構成。サポートされるすべての構成項目については、[DM-master Configuration File](/dm/dm-master-configuration-file.md)を参照してください。
- `worker`: DM-workerサービスに関連する構成。サポートされるすべての構成項目については、[DM-worker Configuration File](/dm/dm-worker-configuration-file.md)を参照してください。

`server_configs`の構成例は以下の通りです：

```yaml
server_configs:
  master:
    log-level: info
    rpc-timeout: "30s"
    rpc-rate-limit: 10.0
    rpc-rate-burst: 40
  worker:
    log-level: info
```

## `master_servers`

`master_servers`はDMコンポーネントのマスターノードが展開されるマシンを指定します。また、各マシンでサービス構成を指定することもできます。`master_servers`は配列です。各配列要素には以下のフィールドが含まれます：

- `host`: 展開先のマシンを指定します。フィールド値はIPアドレスであり、必須です。
- `ssh_port`: オペレーションを行うためにターゲットマシンに接続するためのSSHポートを指定します。フィールドが指定されていない場合は、`global`セクションの`ssh_port`が使用されます。
- `name`: DM-masterインスタンスの名前を指定します。名前は異なるインスタンス間で一意である必要があります。そうでない場合、クラスタを展開できません。
- `port`: DM-masterがサービスを提供するポートを指定します。デフォルト値は "8261" です。
- `peer_port`: DM-master間の通信用ポートを指定します。デフォルト値は "8291" です。
- `deploy_dir`: 展開ディレクトリを指定します。フィールドが指定されていない場合、または相対ディレクトリとして指定されている場合、展開ディレクトリは`global`セクションの`deploy_dir`設定に従って生成されます。
- `data_dir`: データディレクトリを指定します。フィールドが指定されていない場合、または相対ディレクトリとして指定されている場合、データディレクトリは`global`セクションの`data_dir`設定に従って生成されます。
- `log_dir`: ログディレクトリを指定します。フィールドが指定されていない場合、または相対ディレクトリとして指定されている場合、ログディレクトリは`global`セクションの`log_dir`設定に従って生成されます。
- `numa_node`: インスタンスにNUMAポリシーを割り当てます。このフィールドを指定する前に、ターゲットマシンに[numactl](https://linux.die.net/man/8/numactl)がインストールされていることを確認する必要があります。このフィールドは、文字列型です。フィールド値はNUMAノードのID、たとえば "0,1" のような値です。
- `config`: このフィールドの構成ルールは、`server_configs` セクションの `master` と同じです。`config` が指定されている場合、`server_configs` の `master` の構成とマージされます(2つのフィールドが重なる場合、このフィールドの構成が有効になります)。その後、構成ファイルは生成され、`host` フィールドで指定されたマシンに配布されます。
- `os`: `host` フィールドで指定されたマシンのオペレーティングシステムです。フィールドが指定されていない場合、デフォルト値は `global` セクションで構成された `os` の値です。
- `arch`: `host` フィールドで指定されたマシンのアーキテクチャです。フィールドが指定されていない場合、デフォルト値は `global` セクションで構成された `arch` の値です。
- `resource_control`: このサービスのリソース制御です。このフィールドが指定されている場合、このフィールドの構成が `global` セクションの `resource_control` とマージされます(2つのフィールドが重なる場合、このフィールドの構成が有効になります)。その後、systemdの構成ファイルが生成され、`host` フィールドで指定されたマシンに配布されます。このフィールドの構成ルールは、`global` セクションの `resource_control` と同じです。
- `v1_source_path`: v1.0.x からアップグレードする際に、V1ソースの構成ファイルが配置されているディレクトリを指定できます。

`master_servers` セクションでは、展開が完了した後に次のフィールドを変更できません: 

- `host`
- `name`
- `port`
- `peer_port`
- `deploy_dir`
- `data_dir`
- `log_dir`
- `arch`
- `os`
- `v1_source_path`

`master_servers` の構成例は以下の通りです:

```yaml
master_servers:
  - host: 10.0.1.11
    name: master1
    ssh_port: 22
    port: 8261
    peer_port: 8291
    deploy_dir: "/dm-deploy/dm-master-8261"
    data_dir: "/dm-data/dm-master-8261"
    log_dir: "/dm-deploy/dm-master-8261/log"
    numa_node: "0,1"
    # 以下の構成は、`server_configs.master` の値を上書きします。
    config:
      log-level: info
      rpc-timeout: "30s"
      rpc-rate-limit: 10.0
      rpc-rate-burst: 40
  - host: 10.0.1.18
    name: master2
  - host: 10.0.1.19
    name: master3
```

## `worker_servers`

`worker_servers` は、DMコンポーネントのマスターノードが展開されるマシンを指定します。各マシンに対してサービスの構成も指定できます。`worker_servers` は配列です。各配列要素には次のフィールドが含まれます：

- `host`: 展開先のマシンを指定します。フィールド値はIPアドレスであり、必須です。
- `ssh_port`: 操作のためにターゲットマシンに接続するSSHポートを指定します。フィールドが指定されていない場合、`global` セクションの `ssh_port` が使用されます。
- `name`: DMワーカーインスタンスの名前を指定します。名前は異なるインスタンスに対して一意である必要があります。そうでない場合、クラスターを展開できません。
- `port`: DMワーカーがサービスを提供するポートを指定します。デフォルト値は "8262" です。
- `deploy_dir`: 展開ディレクトリを指定します。フィールドが指定されていないか、相対ディレクトリとして指定されている場合、展開ディレクトリは `global` セクションの `deploy_dir` 構成に従って生成されます。
- `data_dir`: データディレクトリを指定します。フィールドが指定されていないか、相対ディレクトリとして指定されている場合、データディレクトリは `global` セクションの `data_dir` 構成に従って生成されます。
- `log_dir`: ログディレクトリを指定します。フィールドが指定されていないか、相対ディレクトリとして指定されている場合、ログディレクトリは `global` セクションの `log_dir` 構成に従って生成されます。
- `numa_node`: インスタンスにNUMAポリシーを割り当てます。このフィールドを指定する前に、ターゲットマシンに [numactl](https://linux.die.net/man/8/numactl) がインストールされていることを確認する必要があります。このフィールドが指定されている場合、cpubindおよびmembindポリシーが [numactl](https://linux.die.net/man/8/numactl) を使用して割り当てられます。このフィールドは文字列型です。フィールド値は、"0,1" のようなNUMAノードのIDです。
- `config`: このフィールドの構成ルールは、`server_configs` セクションの `worker` と同じです。`config` が指定されている場合、`server_configs` の `worker` の構成とマージされます(2つのフィールドが重なる場合、このフィールドの構成が有効になります)。その後、構成ファイルは `host` フィールドで指定されたマシンに生成および配布されます。
- `os`: `host` フィールドで指定されたマシンのオペレーティングシステムです。フィールドが指定されていない場合、デフォルト値は `global` セクションで構成された `os` の値です。
- `arch`: `host` フィールドで指定されたマシンのアーキテクチャです。フィールドが指定されていない場合、デフォルト値は `global` セクションで構成された `arch` の値です。
- `resource_control`: このサービスのリソース制御です。このフィールドが指定されている場合、このフィールドの構成が `global` セクションの `resource_control` とマージされます(2つのフィールドが重なる場合、このフィールドの構成が有効になります)。その後、systemdの構成ファイルが `host` フィールドで指定されたマシンに生成および配布されます。このフィールドの構成ルールは、`global` セクションの `resource_control` と同じです。

`worker_servers` セクションでは、展開が完了した後に次のフィールドを変更できません: 

- `host`
- `name`
- `port`
- `deploy_dir`
- `data_dir`
- `log_dir`
- `arch`
- `os`

`worker_servers` の構成例は以下の通りです:

```yaml
worker_servers:
  - host: 10.0.1.12
    ssh_port: 22
    port: 8262
    deploy_dir: "/dm-deploy/dm-worker-8262"
    log_dir: "/dm-deploy/dm-worker-8262/log"
    numa_node: "0,1"
    # config は、`server_configs.worker` の値を上書きします
    config:
      log-level: info
  - host: 10.0.1.19
```

### `monitoring_servers`

`monitoring_servers` は、Prometheusサービスが展開されるマシンを指定します。マシン上でサービスの構成も指定できます。`monitoring_servers` は配列です。各配列要素には次のフィールドが含まれます:

- `host`: 展開先のマシンを指定します。フィールド値はIPアドレスであり、必須です。
- `ssh_port`: 操作のためにターゲットマシンに接続するSSHポートを指定します。フィールドが指定されていない場合、`global` セクションの `ssh_port` が使用されます。
- `port`: Prometheusがサービスを提供するポートを指定します。デフォルト値は "9090" です。
- `deploy_dir`: 展開ディレクトリを指定します。フィールドが指定されていないか、相対ディレクトリとして指定されている場合、展開ディレクトリは `global` セクションの `deploy_dir` 構成に従って生成されます。
- `data_dir`: データディレクトリを指定します。フィールドが指定されていないか、相対ディレクトリとして指定されている場合、データディレクトリは `global` セクションの `data_dir` 構成に従って生成されます。
- `log_dir`: ログディレクトリを指定します。フィールドが指定されていないか、相対ディレクトリとして指定されている場合、ログディレクトリは `global` セクションの `log_dir` 構成に従って生成されます。
- `numa_node`: インスタンスにNUMAポリシーを割り当てます。このフィールドを指定する前に、ターゲットマシンに [numactl](https://linux.die.net/man/8/numactl) がインストールされていることを確認する必要があります。このフィールドが指定されている場合、cpubindおよびmembindポリシーが [numactl](https://linux.die.net/man/8/numactl) を使用して割り当てられます。このフィールドは文字列型です。フィールド値は、"0,1" のようなNUMAノードのIDです。
- `storage_retention`: Prometheus監視データの保持時間を指定します。デフォルト値は "15d" です。
- `rule_dir`: 完全な `*.rules.yml` ファイルが格納されているローカルディレクトリを指定します。指定されたディレクトリのファイルは、クラスター構成の初期化フェーズ中にPrometheusルールとしてターゲットマシンに送信されます。
- `remote_config`: Prometheusデータをリモートに書き込んだり、リモートからデータを読み取ったりすることをサポートします。このフィールドには2つの構成があります:
    - `remote_write`: Prometheusドキュメント [`<remote_write>`](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#remote_write)を参照してください。
    - `remote_read`: Prometheusドキュメント [`<remote_read>`](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#remote_read)を参照してください。
- `external_alertmanagers`: `external_alertmanagers`フィールドが構成されている場合、Prometheusは外部クラスタにあるAlertmanagerに構成動作を警告します。このフィールドは配列で、各要素は外部Alertmanagerであり、`host`と`web_port`フィールドから構成されます。
- `os`: `host`フィールドで指定されたマシンのオペレーティングシステム。フィールドが指定されていない場合、デフォルト値は`global`セクションで構成された`os`の値です。
- `arch`: `host`フィールドで指定されたマシンのアーキテクチャ。フィールドが指定されていない場合、デフォルト値は`global`セクションで構成された`arch`の値です。
- `resource_control`: このサービスのリソース制御。このフィールドが指定されている場合、このフィールドの構成は`global`セクションの`resource_control`の構成とマージされます（2つのフィールドが重複する場合、このフィールドの構成が有効になります）、その後systemdの構成ファイルが生成され、`host`フィールドで指定されたマシンに配布されます。このフィールドの構成規則は、`global`セクションの`resource_control`と同じです。

`monitoring_servers`セクションでは、次のフィールドはデプロイ完了後に変更できません：

- `host`
- `port`
- `deploy_dir`
- `data_dir`
- `log_dir`
- `arch`
- `os`

`monitoring_servers`の構成例は次のとおりです：
```yaml
monitoring_servers:
  - host: 10.0.1.11
    rule_dir: /local/rule/dir
    remote_config:
      remote_write:
      - queue_config:
          batch_send_deadline: 5m
          capacity: 100000
          max_samples_per_send: 10000
          max_shards: 300
        url: http://127.0.0.1:8003/write
      remote_read:
      - url: http://127.0.0.1:8003/read\
      external_alertmanagers:
      - host: 10.1.1.1
      web_port: 9093
      - host: 10.1.1.2
      web_port: 9094
```

### `grafana_servers`

`grafana_servers`は、Grafanaサービスがデプロイされるマシンを指定します。また、マシン上でサービスの構成を指定することもできます。`grafana_servers`は配列です。各配列要素には次のフィールドが含まれます：

- `host`: デプロイ先のマシンを指定します。フィールド値はIPアドレスであり、必須です。
- `ssh_port`: 操作のためにターゲットマシンに接続するSSHポートを指定します。フィールドが指定されていない場合、`global`セクションの`ssh_port`が使用されます。
- `port`: Grafanaがサービスを提供するポートを指定します。デフォルト値は「3000」です。
- `deploy_dir`: デプロイディレクトリを指定します。フィールドが指定されていない場合、または相対ディレクトリとして指定されている場合、デプロイディレクトリは`global`セクションの`deploy_dir`構成に従って生成されます。
- `os`: `host`フィールドで指定されたマシンのオペレーティングシステム。フィールドが指定されていない場合、デフォルト値は`global`セクションで構成された`os`の値です。
- `arch`: `host`フィールドで指定されたマシンのアーキテクチャ。フィールドが指定されていない場合、デフォルト値は`global`セクションで構成された`arch`の値です。
- `username`: Grafanaログイン画面のユーザー名を指定します。
- `password`: Grafanaの対応するパスワードを指定します。
- `dashboard_dir`: 完全な`dashboard(*.json)`ファイルが存在するローカルディレクトリを指定します。指定されたディレクトリ内のファイルは、クラスタ構成の初期化フェーズ中にGrafanaダッシュボードとしてターゲットマシンに送信されます。
- `resource_control`: このサービスのリソース制御。このフィールドが指定されている場合、このフィールドの構成は`global`セクションの`resource_control`の構成とマージされます（2つのフィールドが重複する場合、このフィールドの構成が有効になります）、その後systemdの構成ファイルが生成され、`host`フィールドで指定されたマシンに配布されます。このフィールドの構成規則は、`global`セクションの`resource_control`と同じです。

> **注:**
>
> `grafana_servers`の`dashboard_dir`フィールドが構成されている場合、「tiup cluster rename」コマンドを実行してクラスタをリネームした後は、次の操作を実行する必要があります：
>
> 1. ローカルの「dashboards」ディレクトリで、`datasource`フィールドの値を新しいクラスタ名に更新します（`datasource`はクラスタ名に命名されます）。
> 2. `tiup cluster reload -R grafana`コマンドを実行します。

`grafana_servers`では、次のフィールドはデプロイ完了後に変更できません：

- `host`
- `port`
- `deploy_dir`
- `arch`
- `os`

`grafana_servers`の構成例は次のとおりです：
```yaml
grafana_servers:
  - host: 10.0.1.11
    dashboard_dir: /local/dashboard/dir
```

### `alertmanager_servers`

`alertmanager_servers`は、Alertmanagerサービスがデプロイされるマシンを指定します。また、各マシンにサービスの構成を指定することもできます。`alertmanager_servers`は配列です。各配列要素には次のフィールドが含まれます：

- `host`: デプロイ先のマシンを指定します。フィールド値はIPアドレスであり、必須です。
- `ssh_port`: 操作のためにターゲットマシンに接続するSSHポートを指定します。フィールドが指定されていない場合、`global`セクションの`ssh_port`が使用されます。
- `web_port`: AlertmanagerがWebサービスを提供するポートを指定します。デフォルト値は「9093」です。
- `cluster_port`: 1つのAlertmanagerと他のAlertmanagerの通信ポートを指定します。デフォルト値は「9094」です。
- `deploy_dir`: デプロイディレクトリを指定します。フィールドが指定されていない場合、または相対ディレクトリとして指定されている場合、デプロイディレクトリは`global`セクションの`deploy_dir`構成に従って生成されます。
- `data_dir`: データディレクトリを指定します。フィールドが指定されていない場合、または相対ディレクトリとして指定されている場合、データディレクトリは`global`セクションの`data_dir`構成に従って生成されます。
- `log_dir`: ログディレクトリを指定します。フィールドが指定されていない場合、または相対ディレクトリとして指定されている場合、ログディレクトリは`global`セクションの`log_dir`構成に従って生成されます。
- `numa_node`: インスタンスにNUMAポリシーを割り当てます。このフィールドを指定する前に、対象マシンに[numactl](https://linux.die.net/man/8/numactl)がインストールされていることを確認する必要があります。このフィールドが指定されている場合、[numactl](https://linux.die.net/man/8/numactl)を使用してcpubindおよびmembindポリシーを割り当てます。このフィールドは文字列型です。フィールド値はNUMAノードのID、たとえば「0,1」です。
- `config_file`: ローカルファイルを指定します。指定されたファイルは、クラスタ構成の初期化フェーズ中にAlertmanagerの構成としてターゲットマシンに送信されます。
- `os`: `host`フィールドで指定されたマシンのオペレーティングシステム。フィールドが指定されていない場合、デフォルト値は`global`セクションで構成された`os`の値です。
- `arch`: `host`フィールドで指定されたマシンのアーキテクチャ。フィールドが指定されていない場合、デフォルト値は`global`セクションで構成された`arch`の値です。
- `resource_control`: このサービスのリソース制御。このフィールドが指定されている場合、このフィールドの構成は`global`セクションの`resource_control`の構成とマージされます（2つのフィールドが重複する場合、このフィールドの構成が有効になります）、その後systemdの構成ファイルが生成され、`host`フィールドで指定されたマシンに配布されます。このフィールドの構成規則は、`global`セクションの`resource_control`と同じです。

`alertmanager_servers`では、次のフィールドはデプロイ完了後に変更できません：

- `host`
- `web_port`
- `cluster_port`
- `deploy_dir`
- `data_dir`
- `log_dir`
- `arch`
- `os`

`alertmanager_servers`の構成例は次のとおりです：
```yaml
alertmanager_servers:
  - host: 10.0.1.11
    config_file: /local/config/file
  - host: 10.0.1.12
    config_file: /local/config/file
```