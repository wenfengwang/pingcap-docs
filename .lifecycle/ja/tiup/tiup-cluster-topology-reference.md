```yaml
---
title: TiUPを使用したTiDB展開のためのトポロジ構成ファイル
---

# TiUPを使用したTiDB展開のためのトポロジ構成ファイル

TiUPを使用してTiDBを展開またはスケールするには、クラスタのトポロジを記述するためのトポロジファイル（[サンプル](https://github.com/pingcap/tiup/blob/master/embed/examples/cluster/topology.example.yaml)）を提供する必要があります。

同様に、クラスタのトポロジを変更するには、トポロジファイルを変更する必要があります。違いは、クラスタが展開された後は、トポロジファイルの一部のフィールドのみを変更できる点です。このドキュメントでは、トポロジファイルの各セクションと各セクション内の各フィールドについて紹介します。

## ファイル構造

TiUPを使用してTiDBを展開するためのトポロジ構成ファイルには、次のセクションが含まれる場合があります。

- [global](#global): クラスタのグローバル構成。一部の構成項目はデフォルト値を使用し、各インスタンスで個別に構成できます。
- [monitored](#monitored): モニタリングサービスの構成、つまり、blackbox_exporter および `node_exporter` の構成。各マシンには `node_exporter` と `blackbox_exporter` が展開されます。
- [server_configs](#server_configs): コンポーネントのグローバル構成。各コンポーネントを個別に構成できます。インスタンスで同じ名前の構成項目を持つ場合、インスタンスの構成項目が適用されます。
- [component_versions](#component_versions): コンポーネントのバージョン。コンポーネントがクラスタバージョンを使用しない場合に構成できます。このセクションは、tiup-cluster v1.14.0 で導入されました。
- [pd_servers](#pd_servers): PD インスタンスの構成。この構成では、PD コンポーネントが展開されるマシンを指定します。
- [tidb_servers](#tidb_servers): TiDB インスタンスの構成。この構成では、TiDB コンポーネントが展開されるマシンを指定します。
- [tikv_servers](#tikv_servers): TiKV インスタンスの構成。この構成では、TiKV コンポーネントが展開されるマシンを指定します。
- [tiflash_servers](#tiflash_servers): TiFlash インスタンスの構成。この構成では、TiFlash コンポーネントが展開されるマシンを指定します。
- [pump_servers](#pump_servers): Pump インスタンスの構成。この構成では、Pump コンポーネントが展開されるマシンを指定します。
- [drainer_servers](#drainer_servers): Drainer インスタンスの構成。この構成では、Drainer コンポーネントが展開されるマシンを指定します。
- [cdc_servers](#cdc_servers): TiCDC インスタンスの構成。この構成では、TiCDC コンポーネントが展開されるマシンを指定します。
- [tispark_masters](#tispark_masters): TiSpark master インスタンスの構成。この構成では、TiSpark master コンポーネントが展開されるマシンを指定します。TiSpark master ノードは 1 つだけ展開できます。
- [tispark_workers](#tispark_workers): TiSpark worker インスタンスの構成。この構成では、TiSpark worker コンポーネントが展開されるマシンを指定します。
- [monitoring_servers](#monitoring_servers): Prometheus および NGMonitoring が展開されるマシンを指定します。TiUPは複数の Prometheus インスタンスを展開できますが、最初のインスタンスのみが使用されます。
- [grafana_servers](#grafana_servers): Grafana インスタンスの構成。この構成では、Grafana が展開されるマシンを指定します。
- [alertmanager_servers](#alertmanager_servers): Alertmanager インスタンスの構成。この構成では、Alertmanager が展開されるマシンを指定します。

### `global`

`global` セクションは、クラスタのグローバル構成に対応し、次のフィールドを持ちます。

- `user`: 展開されたクラスタを開始するために使用するユーザー。デフォルト値は `"tidb"` です。`<user>` フィールドで指定されたユーザーがターゲットマシンに存在しない場合、このユーザーが自動的に作成されます。

- `group`: ユーザーが所属するユーザーグループ。ユーザーが作成されるときに指定されます。値は `<user>` フィールドの値をデフォルト値とします。指定されたグループが存在しない場合、自動的に作成されます。

- `ssh_port`: 操作のためにターゲットマシンに接続するための SSH ポートを指定します。デフォルト値は `22` です。

- `enable_tls`: クラスタで TLS を有効にするかどうかを指定します。TLS を有効にした後は、コンポーネント間またはクライアントとコンポーネント間の接続に生成された TLS 証明書を使用する必要があります。デフォルト値は `false` です。

- `listen_host`: デフォルトのリッスン IP アドレスを指定します。空欄の場合、各インスタンスは `host` フィールドに `:` が含まれるかどうかに応じて自動的に `::` または `0.0.0.0` に設定します。このフィールドは tiup-cluster v1.14.0 で導入されました。

- `deploy_dir`: 各コンポーネントの展開ディレクトリ。デフォルト値は `"deployed"` です。適用ルールは次のとおりです:

    - インスタンスレベルで `deploy_dir` の絶対パスが構成されている場合、インスタンスのために構成された `deploy_dir` が実際の展開ディレクトリです。

    - 各インスタンスについて、`deploy_dir` を構成しない場合、デフォルト値は相対パスの `<component-name>-<component-port>` です。

    - `global.deploy_dir` が絶対パスである場合、コンポーネントは `<global.deploy_dir>/<instance.deploy_dir>` ディレクトリに展開されます。

    - `global.deploy_dir` が相対パスである場合、コンポーネントは `/home/<global.user>/<global.deploy_dir>/<instance.deploy_dir>` ディレクトリに展開されます。

- `data_dir`: データディレクトリ。デフォルト値は `"data"` です。適用ルールは次のとおりです:

    - インスタンスレベルで `data_dir` の絶対パスが構成されている場合、インスタンスのために構成された `data_dir` が実際の展開ディレクトリです。

    - 各インスタンスについて、`data_dir` を構成しない場合、デフォルト値は `<global.data_dir>` です。

    - `data_dir` が相対パスの場合、コンポーネントのデータは `<deploy_dir>/<data_dir>` に配置されます。`<deploy_dir>` の計算ルールについては、`deploy_dir` フィールドの適用ルールを参照してください。

- `log_dir`: ログディレクトリ。デフォルト値は `"log"` です。適用ルールは次のとおりです:

    - インスタンスレベルで絶対パス `log_dir` が構成されている場合、インスタンスのために構成された `log_dir` が実際のログディレクトリです。

    - 各インスタンスで `log_dir` を構成しない場合、デフォルト値は `<global.log_dir>` です。

    - `log_dir` が相対パスの場合、コンポーネントのログは `<deploy_dir>/<log_dir>` に配置されます。`<deploy_dir>` の計算ルールについては、`deploy_dir` フィールドの適用ルールを参照してください。

- `os`: ターゲットマシンのオペレーティングシステム。このフィールドは、ターゲットマシンにプッシュされるコンポーネントが適応するオペレーティングシステムを制御します。デフォルト値は "linux" です。

- `arch`: ターゲットマシンの CPU アーキテクチャ。このフィールドは、ターゲットマシンにプッシュされるバイナリパッケージが適応するプラットフォームを制御します。サポートされる値は "amd64" および "arm64" です。デフォルト値は "amd64" です。

- `resource_control`: 実行時のリソース制御。このフィールドのすべての構成は systemd のサービスファイルに書き込まれます。デフォルトでは制限はありません。制御できるリソースは次のとおりです:

    - `memory_limit`: 最大ランタイムメモリを制限します。たとえば、"2G" は 2 GB の最大メモリが使用できることを意味します。

    - `cpu_quota`: ランタイムでの最大 CPU 使用率を制限します。たとえば、"200%" です。

    - `io_read_bandwidth_max`: ディスク読み取りの最大 I/O バンド幅を制限します。たとえば、`"/dev/disk/by-path/pci-0000:00:1f.2-scsi-0:0:0:0 100M"` です。

    - `io_write_bandwidth_max`: ディスク書き込みの最大 I/O バンド幅を制限します。たとえば、`/dev/disk/by-path/pci-0000:00:1f.2-scsi-0:0:0:0 100M` です。

    - `limit_core`: コアダンプのサイズを制御します。

`global` の構成例は次のとおりです:

```yaml
global:
  user: "tidb"
  resource_control:
    memory_limit: "2G"
```

上記の構成では、`tidb` ユーザーがクラスタを開始するために使用されます。同時に、各コンポーネントは実行時に最大 2 GB のメモリに制限されます。

### `monitored`

`monitored` は、ターゲットマシン上のモニタリングサービスを構成するために使用されます: [`node_exporter`](https://github.com/prometheus/node_exporter) および [`blackbox_exporter`](https://github.com/prometheus/blackbox_exporter)。次のフィールドが含まれます:

- `node_exporter_port`: `node_exporter` のサービスポート。デフォルト値は `9100` です。

- `blackbox_exporter_port`: `blackbox_exporter` のサービスポート。デフォルト値は `9115` です。

- `deploy_dir`: 展開ディレクトリを指定します。指定されていない場合、または相対ディレクトリとして指定されている場合、ディレクトリは `global` で構成された `deploy_dir` ディレクトリに従って生成されます。

- `data_dir`: データディレクトリを指定します。指定されていない場合、または相対ディレクトリとして指定されている場合、ディレクトリは `global` で構成された `data_dir` ディレクトリに従って生成されます。

- `log_dir`: ログディレクトリを指定します。指定されていない場合、または相対ディレクトリとして指定されている場合、ログは `global` で構成された `log_dir` ディレクトリに従って生成されます。
```
```yaml
monitored:
  node_exporter_port: 9100
  blackbox_exporter_port: 9115
```
上記の構成は、`node_exporter`が`9100`ポートを使用し、`blackbox_exporter`が`9115`ポートを使用するように指定されています。

### `server_configs`

`server_configs`は、サービスを構成し、各コンポーネントの構成ファイルを生成するために使用されます。このセクションの構成は`global`セクション同様、インスタンス内の同じ名前の構成によって上書きされることがあります。`server_configs`には主に次のフィールドが含まれています。

- `tidb`：TiDBサービス関連の構成。完全な構成については、[TiDB構成ファイル](/tidb-configuration-file.md)を参照してください。

- `tikv`：TiKVサービス関連の構成。完全な構成については、[TiKV構成ファイル](/tikv-configuration-file.md)を参照してください。

- `pd`：PDサービス関連の構成。完全な構成については、[PD構成ファイル](/pd-configuration-file.md)を参照してください。

- `tiflash`：TiFlashサービス関連の構成。完全な構成については、[TiFlash構成ファイル](/tiflash/tiflash-configuration.md)を参照してください。

- `tiflash_learner`：各TiFlashノードには特別な組み込みTiKVがあります。この構成項目は通常、この特別なTiKVを構成するために使用されます。この構成項目の内容を変更することは一般には勧められていません。

- `pump`：Pumpサービス関連の構成。完全な構成については、[TiDBバイナリログ構成ファイル](/tidb-binlog/tidb-binlog-configuration-file.md#pump)を参照してください。

- `drainer`：Drainerサービス関連の構成。完全な構成については、[TiDBバイナリログ構成ファイル](/tidb-binlog/tidb-binlog-configuration-file.md#drainer)を参照してください。

- `cdc`：TiCDCサービス関連の構成。完全な構成については、[TiCDCの展開](/ticdc/deploy-ticdc.md)を参照してください。

`server_configs`の構成例は次の通りです。

```yaml
server_configs:
  tidb:
    lease: "45s"
    split-table: true
    token-limit: 1000
    instance.tidb_enable_ddl: true
  tikv:
    log-level: "info"
    readpool.unified.min-thread-count: 1
```
上記の構成は、TiDBおよびTiKVのグローバル構成を指定しています。

### `component_versions`

> **注意：**
>
> TiDB、TiKV、PD、TiCDCなどのバージョン番号を共有するコンポーネントについては、混在バージョンの展開シナリオで正常に動作することを保証する完全なテストがありません。このセクションは、テスト環境でのみ使用するか、[技術サポート](/support.md)のサポートを受けて使用してください。

`component_versions`は特定のコンポーネントのバージョン番号を指定するために使用されます。

- `component_versions`が構成されていない場合、各コンポーネントは、PD、TiKVなどのTiDBクラスタと同じバージョン番号を使用するか、最新バージョンを使用します（Alertmanagerなど）。
- `component_versions`が構成されている場合、対応するコンポーネントは指定されたバージョンを使用し、このバージョンは後続のクラスタの拡張およびアップグレード操作で使用されます。

特定のコンポーネントの特定のバージョンを使用する必要がある場合にのみ構成してください。

`component_versions`には次のフィールドが含まれています。

- `tikv`：TiKVコンポーネントのバージョン
- `tiflash`：TiFlashコンポーネントのバージョン
- `pd`：PDコンポーネントのバージョン
- `tidb_dashboard`：スタンドアロンTiDBダッシュボードコンポーネントのバージョン
- `pump`：Pumpコンポーネントのバージョン
- `drainer`：Drainerコンポーネントのバージョン
- `cdc`：CDCコンポーネントのバージョン
- `kvcdc`：TiKV-CDCコンポーネントのバージョン
- `tiproxy`：Tiproxyコンポーネントのバージョン
- `prometheus`：Prometheusコンポーネントのバージョン
- `grafana`：Grafanaコンポーネントのバージョン
- `alertmanager`：Alertmanagerコンポーネントのバージョン

次に、`component_versions`の構成例を示します。

```yaml
component_versions:
  kvcdc: "v1.1.1"
```
上記の構成は、TiKV-CDCのバージョン番号を`v1.1.1`と指定しています。

### `pd_servers`

`pd_servers`は、PDサービスが展開されるマシンを指定します。また、各マシンのサービス構成も指定します。`pd_servers`は配列であり、配列の各要素には次のフィールドが含まれています。

- `host`：PDサービスが展開されるマシンを指定します。フィールドの値はIPアドレスであり、必須です。

- `listen_host`：マシンに複数のIPアドレスがある場合、`listen_host`はサービスのリッスンIPアドレスを指定します。デフォルト値は`0.0.0.0`です。

- `ssh_port`：操作のために対象のマシンに接続するSSHポートを指定します。指定されていない場合、`global`セクションの`ssh_port`が使用されます。

- `name`：PDインスタンスの名前を指定します。異なるインスタンスは一意の名前を持たなければなりません。そうでない場合、インスタンスを展開できません。

- `client_port`：PDがクライアントに接続するために使用するポートを指定します。デフォルト値は`2379`です。

- `peer_port`：PD間の通信に使用するポートを指定します。デフォルト値は`2380`です。

- `deploy_dir`：展開ディレクトリを指定します。指定されていない場合、または相対ディレクトリとして指定されている場合、ディレクトリは`global`で構成された`deploy_dir`ディレクトリに従って生成されます。

- `data_dir`：データディレクトリを指定します。指定されていない場合、または相対ディレクトリとして指定されている場合、ディレクトリは`global`で構成された`data_dir`ディレクトリに従って生成されます。

- `log_dir`：ログディレクトリを指定します。指定されていない場合、または相対ディレクトリとして指定されている場合、ログは`global`で構成された`log_dir`ディレクトリに従って生成されます。

- `numa_node`：インスタンスにNUMAポリシーを割り当てます。このフィールドを指定する前に、対象のマシンに[numactl](https://linux.die.net/man/8/numactl)がインストールされていることを確認する必要があります。このフィールドは文字列型です。フィールド値はNUMAノードのIDであり、たとえば "0,1" です。

- `config`：このフィールドの構成ルールは、`server_configs`の`pd`構成ルールと同じです。このフィールドが構成されている場合、フィールドの内容は`server_configs`の`pd`コンテンツとマージされます（2つのフィールドが重なる場合、このフィールドの内容が有効になります）。その後、生成された構成ファイルが指定されたマシンに送信されます。

- `os`：`host`で指定されたマシンのオペレーティングシステム。このフィールドが指定されていない場合、デフォルト値は`global`の`os`の値です。

- `arch`：`host`で指定されたマシンのアーキテクチャ。このフィールドが指定されていない場合、デフォルト値は`global`の`arch`の値です。

- `resource_control`：サービスのリソース制御。このフィールドが構成されている場合、フィールドの内容は`global`の`resource_control`コンテンツとマージされます（2つのフィールドが重なる場合、このフィールドの内容が有効になります）。その後、生成されたsystemd構成ファイルが指定されたマシンに送信されます。`resource_control`の構成ルールは、`global`の`resource_control`コンテンツと同じです。

上記のフィールドについては、展開後にこれらの構成フィールドを変更することはできません。

- `host`
- `listen_host`
- `name`
- `client_port`
- `peer_port`
- `deploy_dir`
- `data_dir`
- `log_dir`
- `arch`
- `os`

`pd_servers`の構成例は次の通りです。

```yaml
pd_servers:
  - host: 10.0.1.11
    config:
      schedule.max-merge-region-size: 20
      schedule.max-merge-region-keys: 200000
  - host: 10.0.1.12
```
上記の構成は、PDを`10.0.1.11`および`10.0.1.12`に展開し、`10.0.1.11`のPDに特定の構成を行うように指定しています。

### `tidb_servers`

`tidb_servers`は、TiDBサービスが展開されるマシンを指定します。また、各マシンのサービス構成も指定します。`tidb_servers`は配列であり、配列の各要素には次のフィールドが含まれています。

- `host`：TiDBサービスが展開されるマシンを指定します。フィールドの値はIPアドレスであり、必須です。

- `listen_host`：マシンに複数のIPアドレスがある場合、`listen_host`はサービスのリッスンIPアドレスを指定します。デフォルト値は`0.0.0.0`です。

- `ssh_port`：操作のために対象のマシンに接続するSSHポートを指定します。指定されていない場合、`global`セクションの`ssh_port`が使用されます。

- `port`：TiDBサービスのリッスンポートを指定します。これはMySQLクライアントへの接続を提供するために使用されます。デフォルト値は`4000`です。
```
- `status_port`: TiDBステータスサービスのリスニングポートで、HTTPリクエスト経由でTiDBサービスのステータスを外部から閲覧するために使用されます。デフォルト値は `10080` です。

- `deploy_dir`: デプロイディレクトリを指定します。指定されていないか、相対ディレクトリとして指定されている場合、`global` に構成された `deploy_dir` ディレクトリに従ってディレクトリが生成されます。

- `log_dir`: ログディレクトリを指定します。指定されていないか、相対ディレクトリとして指定されている場合、`global` に構成された `log_dir` ディレクトリに従ってログが生成されます。

- `numa_node`: インスタンスにNUMAポリシーを割り当てます。このフィールドを指定する前に、対象のマシンに [numactl](https://linux.die.net/man/8/numactl) がインストールされていることを確認する必要があります。このフィールドが指定された場合、cpubindおよびmembindポリシーは [numactl](https://linux.die.net/man/8/numactl) を使用して割り当てられます。このフィールドは文字列型です。フィールド値は "0,1" のようなNUMAノードのIDです。

- `config`: このフィールドの構成ルールは、`server_configs` の `tidb` 構成ルールと同じです。このフィールドを構成すると、このフィールドの内容は `server_configs` の `tidb` コンテンツと結合されます（2つのフィールドが重複している場合、このフィールドの内容が有効になります）。その後、構成ファイルが生成され、`host` で指定されたマシンに送信されます。

- `os`: `host` で指定されたマシンのオペレーティングシステムです。このフィールドが指定されていない場合、デフォルト値は `global` での `os` 値です。

- `arch`: `host` で指定されたマシンのアーキテクチャです。このフィールドが指定されていない場合、デフォルト値は `global` での `arch` 値です。

- `resource_control`: サービスのリソース制御を指定します。このフィールドが構成されている場合、フィールドのコンテンツは `global` の `resource_control` コンテンツと結合されます（2つのフィールドが重複している場合、このフィールドの内容が有効になります）。その後、systemdの構成ファイルが生成され、`host` で指定されたマシンに送信されます。`resource_control` の構成ルールは、`global` の `resource_control` コンテンツと同じです。

上記のフィールドに関して、デプロイ後にこれらの構成されたフィールドを変更することはできません。

- `host`
- `listen_host`
- `port`
- `status_port`
- `deploy_dir`
- `log_dir`
- `arch`
- `os`

`tidb_servers` の構成例は次のようになります：

```yaml
tidb_servers:
  - host: 10.0.1.14
    config:
      log.level: warn
      log.slow-query-file: tidb-slow-overwrited.log
  - host: 10.0.1.15
```

### `tikv_servers`

`tikv_servers` はTiKVサービスがデプロイされるマシンを指定します。また、配下の各マシンでのサービス構成も指定します。`tikv_servers` は配列であり、配列の各要素には以下のフィールドが含まれます：

- `host`: TiKVサービスがデプロイされるマシンを指定します。フィールド値はIPアドレスであり、必須です。

- `listen_host`: マシンに複数のIPアドレスがある場合、`listen_host` はサービスのリスニングIPアドレスを指定します。デフォルト値は `0.0.0.0` です。

- `ssh_port`: 操作のために対象のマシンに接続するSSHポートを指定します。指定されていない場合、 `global` セクションの `ssh_port` が使用されます。

- `port`: TiKVサービスのリスニングポートです。デフォルト値は `20160` です。

- `status_port`: TiKVステータスサービスのリスニングポートです。デフォルト値は `20180` です。

- `deploy_dir`: デプロイディレクトリを指定します。指定されていないか、相対ディレクトリとして指定されている場合、`global` に構成された `deploy_dir` ディレクトリに従ってディレクトリが生成されます。

- `data_dir`: データディレクトリを指定します。指定されていないか、相対ディレクトリとして指定されている場合、`global` に構成された `data_dir` ディレクトリに従ってディレクトリが生成されます。

- `log_dir`: ログディレクトリを指定します。指定されていないか、相対ディレクトリとして指定されている場合、`global` に構成された `log_dir` ディレクトリに従ってログが生成されます。

- `numa_node`: インスタンスにNUMAポリシーを割り当てます。このフィールドを指定する前に、対象のマシンに [numactl](https://linux.die.net/man/8/numactl) がインストールされていることを確認する必要があります。このフィールドが指定された場合、cpubindおよびmembindポリシーは [numactl](https://linux.die.net/man/8/numactl) を使用して割り当てられます。このフィールドは文字列型です。フィールド値は "0,1" のようなNUMAノードのIDです。

- `config`: このフィールドの構成ルールは、`server_configs` の `tikv` 構成ルールと同じです。このフィールドを構成すると、このフィールドの内容は `server_configs` の `tikv` コンテンツと結合されます（2つのフィールドが重複している場合、このフィールドの内容が有効になります）。その後、構成ファイルが生成され、`host` で指定されたマシンに送信されます。

- `os`: `host` で指定されたマシンのオペレーティングシステムです。このフィールドが指定されていない場合、デフォルト値は `global` での `os` 値です。

- `arch`: `host` で指定されたマシンのアーキテクチャです。このフィールドが指定されていない場合、デフォルト値は `global` での `arch` 値です。

- `resource_control`: サービスのリソース制御を指定します。このフィールドが構成されている場合、フィールドのコンテンツは `global` の `resource_control` コンテンツと結合されます（2つのフィールドが重複している場合、このフィールドの内容が有効になります）。その後、systemdの構成ファイルが生成され、`host` で指定されたマシンに送信されます。`resource_control` の構成ルールは、`global` の `resource_control` コンテンツと同じです。

上記のフィールドに関して、デプロイ後にこれらの構成されたフィールドを変更することはできません。

- `host`
- `listen_host`
- `port`
- `status_port`
- `deploy_dir`
- `data_dir`
- `log_dir`
- `arch`
- `os`

`tikv_servers` の構成例は次のようになります：

```yaml
tikv_servers:
  - host: 10.0.1.14
    config:
      server.labels: { zone: "zone1", host: "host1" }
  - host: 10.0.1.15
    config:
      server.labels: { zone: "zone1", host: "host2" }
```

### `tiflash_servers`

`tiflash_servers` はTiFlashサービスがデプロイされるマシンを指定します。また、配下の各マシンでのサービス構成も指定します。このセクションは配列であり、配列の各要素には以下のフィールドが含まれます：

- `host`: TiFlashサービスがデプロイされるマシンを指定します。フィールド値はIPアドレスであり、必須です。

- `ssh_port`: 操作のために対象のマシンに接続するSSHポートを指定します。指定されていない場合、 `global` セクションの `ssh_port` が使用されます。

- `tcp_port`: TiFlashのTCPサービスのポートです。デフォルト値は `9000` です。

- `flash_service_port`: TiFlashがサービスを提供するポートで、TiDBはこのポートを経由してTiFlashからデータを読み取ります。デフォルト値は `3930` です。

- `metrics_port`: TiFlashのステータスポートで、メトリックデータの出力に使用されます。デフォルト値は `8234` です。

- `flash_proxy_port`: 組み込みTiKVのポートです。デフォルト値は `20170` です。

- `flash_proxy_status_port`: 組み込まれたTiKVのステータスポートです。デフォルト値は `20292` です。

- `deploy_dir`: デプロイディレクトリを指定します。指定されていないか、相対ディレクトリとして指定されている場合、`global` に構成された `deploy_dir` ディレクトリに従ってディレクトリが生成されます。

- `data_dir`: データディレクトリを指定します。指定されていないか、相対ディレクトリとして指定されている場合、`global` に構成された `data_dir` ディレクトリに従ってディレクトリが生成されます。TiFlashはコンマで区切られた複数の `data_dir` ディレクトリをサポートします。

- `log_dir`: ログディレクトリを指定します。指定されていないか、相対ディレクトリとして指定されている場合、`global` に構成された `log_dir` ディレクトリに従ってログが生成されます。

- `tmp_path`: TiFlash一時ファイルのストレージパスです。デフォルト値は [`path` または `storage.latest.dir` の最初のディレクトリ] + "/tmp" です。

- `numa_node`: インスタンスにNUMAポリシーを割り当てます。このフィールドを指定する前に、対象のマシンに [numactl](https://linux.die.net/man/8/numactl) がインストールされていることを確認する必要があります。このフィールドが指定された場合、cpubindおよびmembindポリシーは [numactl](https://linux.die.net/man/8/numactl) を使用して割り当てられます。このフィールドは文字列型です。フィールド値は "0,1" のようなNUMAノードのIDです。
- `config`: このフィールドの設定ルールは、`server_configs` の `tiflash` 設定ルールと同じです。このフィールドが設定されている場合、このフィールドの内容は、`server_configs` の `tiflash` 内容とマージされます（もし、2 つ以上のフィールドが重なっている場合、このフィールドの内容が有効になります）。次に、設定ファイルが生成され、`host` で指定されたマシンに送信されます。

- `learner_config`: 各 TiFlash ノードには、特別な組み込みの TiKV があります。この設定項目は、この特別な TiKV を設定するために使用されます。一般に、この設定項目の内容を変更することはお勧めしません。

- `os`: `host` で指定されたマシンのオペレーティング·システムです。このフィールドが指定されていない場合、デフォルト値は `global` の `os` の値です。

- `arch`: `host` で指定されたマシンのアーキテクチャです。このフィールドが指定されていない場合、デフォルト値は `global` の `arch` の値です。

- `resource_control`: サービスのリソース制御です。このフィールドが設定されている場合、このフィールドの内容は、`global` の `resource_control` 内容とマージされます（もし、2 つ以上のフィールドが重なっている場合、このフィールドの内容が有効になります）。次に、systemd の設定ファイルが生成され、`host` で指定されたマシンに送信されます。`resource_control` の設定規則は、`global` の `resource_control` 内容と同じです。

デプロイ後、上記のフィールドについては、`data_dir` に対してディレクトリを追加するだけできます。以下のフィールドについては、設定したフィールドを変更できません：

- `host`
- `tcp_port`
- `http_port`
- `flash_service_port`
- `flash_proxy_port`
- `flash_proxy_status_port`
- `metrics_port`
- `deploy_dir`
- `log_dir`
- `tmp_path`
- `arch`
- `os`

`tiflash_servers` の構成例は次のようになります：

```yaml
tiflash_servers:
  - host: 10.0.1.21
  - host: 10.0.1.22
```

### `pump_servers`

`pump_servers` は、TiDB Binlog の Pump サービスを展開するマシンを指定します。また、各マシンでのサービス設定も指定します。`pump_servers` は配列であり、配列の各要素には次のフィールドが含まれます：

- `host`: Pump サービスが展開されるマシンを指定します。このフィールドの値はIPアドレスであり、必須です。

- `ssh_port`: 操作のためにターゲット·マシンに接続するSSHポートを指定します。指定されていない場合、`global` セクションの `ssh_port` が使用されます。

- `port`: Pump サービスのリスニングポートです。デフォルト値は `8250` です。

- `deploy_dir`: 展開ディレクトリを指定します。指定されていないか、相対ディレクトリとして指定されている場合、`global` で構成された `deploy_dir` ディレクトリに従ってディレクトリが生成されます。 

- `data_dir`: データディレクトリを指定します。指定されていないか、相対ディレクトリとして指定されている場合、`global` で構成された `data_dir` ディレクトリに従ってディレクトリが生成されます。

- `log_dir`: ログディレクトリを指定します。指定されていないか、相対ディレクトリとして指定されている場合、ログは `global` で構成された `log_dir` ディレクトリに従って生成されます。

- `numa_node`: インスタンスに NUMA ポリシーを割り当てます。このフィールドを指定する前に、ターゲット·マシンに [numactl](https://linux.die.net/man/8/numactl) がインストールされていることを確認する必要があります。このフィールドが指定されている場合、[numactl](https://linux.die.net/man/8/numactl) を使用して、cpubind および membind ポリシーが割り当てられます。このフィールドは、文字列型です。フィールド値は NUMA ノードの ID です。たとえば、"0,1" などです。

- `config`: このフィールドの設定ルールは、`server_configs` の `pump` 設定ルールと同じです。このフィールドが設定されている場合、このフィールドの内容は、`server_configs` の `pump` 内容とマージされます（もし、2 つ以上のフィールドが重なっている場合、このフィールドの内容が有効になります）。次に、設定ファイルが生成され、`host` で指定されたマシンに送信されます。

- `os`: `host` で指定されたマシンのオペレーティング·システムです。このフィールドが指定されていない場合、デフォルト値は `global` の `os` の値です。

- `arch`: `host` で指定されたマシンのアーキテクチャです。このフィールドが指定されていない場合、デフォルト値は `global` の `arch` の値です。

- `resource_control`: サービスのリソース制御です。このフィールドが設定されている場合、このフィールドの内容は、`global` の `resource_control` 内容とマージされます（もし、2 つ以上のフィールドが重なっている場合、https://linux.die.net/man/8/numactl)[numactl] を使用して、cpubind および membind ポリシーが割り当てられます。このフィールドは、文字列型です。フィールド値は NUMA ノードの ID です。たとえば、"0,1" などです。

- `config`: このフィールドの設定ルールは、`server_configs` の `drainer` 設定ルールと同じです。このフィールドが設定されている場合、このフィールドの内容は、`server_configs` の `drainer` 内容とマージされます（もし、2 つ以上のフィールドが重なっている場合、このフィールドの内容が有効になります）。次に、設定ファイルが生成され、`host` で指定されたマシンに送信されます。

- `os`: `host` で指定されたマシンのオペレーティング·システムです。このフィールドが指定されていない場合、デフォルト値は `global` の `os` の値です。

- `arch`: `host` で指定されたマシンのアーキテクチャです。このフィールドが指定されていない場合、デフォルト値は `global` の `arch` の値です。

- `resource_control`: サービスのリソース制御です。このフィールドが設定されている場合、このフィールドの内容は、`global` の `resource_control` 内容とマージされます（もし、2 つ以上のフィールドが重なっている場合、このフィールドの内容が有効になります）。次に、systemd の設定ファイルが生成され、`host` で指定されたマシンに送信されます。`resource_control` の設定規則は、`global` の `resource_control` 内容と同じです。

上記のフィールドについて、デプロイ後に設定されたフィールドは変更できません：

- `host`
- `port`
- `deploy_dir`
- `data_dir`
- `log_dir`
- `arch`
- `os`

`drainer_servers` の構成例は次のようになります：

```yaml
drainer_servers:
  - host: 10.0.1.21
    config:
      initial-commit-ts: -1
      syncer.db-type: "mysql"
      syncer.to.host: "127.0.0.1"
      syncer.to.user: "root"
      syncer.to.password: ""
      syncer.to.port: 3306
      syncer.ignore-table:
```yaml
- db-name: test
  tbl-name: log
- db-name: test
  tbl-name: audit
```

### `cdc_servers`

`cdc_servers`は、TiCDCサービスがデプロイされているマシンを指定します。また、各マシンのサービス構成も指定します。 `cdc_servers`は配列です。各配列要素には次のフィールドが含まれます。

- `host`：TiCDCサービスがデプロイされているマシンを指定します。フィールド値はIPアドレスであり、必須です。

- `ssh_port`：対象のマシンに接続するためのSSHポートを指定します。指定されていない場合、`ssh_port`は`global`セクションの`ssh_port`が使用されます。

- `port`：TiCDCサービスのリッスンポートです。デフォルト値は `8300` です。

- `deploy_dir`：デプロイディレクトリを指定します。指定されていない場合、または相対ディレクトリとして指定されている場合、ディレクトリは`global`で構成された`deploy_dir`ディレクトリに従って生成されます。

- `data_dir`：データディレクトリを指定します。指定されていない場合、または相対ディレクトリとして指定されている場合、ディレクトリは`global`で構成された`data_dir`ディレクトリに従って生成されます。

- `log_dir`：ログディレクトリを指定します。指定されていない場合、または相対ディレクトリとして指定されている場合、ログは`global`で構成された`log_dir`ディレクトリに従って生成されます。

- `gc-ttl`：PD内のTiCDCによって設定されたサービスレベルのGCセーフポイントの有効期間（TTL）を秒単位で指定します。デフォルト値は `86400` で、つまり24時間です。

- `tz`：TiCDCサービスが使用するタイムゾーンです。TiCDCは、内部的にタイムスタンプなどの時間データ型を変換したり、データを下流にレプリケートする際にこのタイムゾーンを使用します。デフォルト値はプロセスが実行されるローカルタイムゾーンです。

- `numa_node`：インスタンスにNUMAポリシーを割り当てます。このフィールドを指定する前に、対象のマシンに[numactl](https://linux.die.net/man/8/numactl)がインストールされていることを確認する必要があります。このフィールドが指定されている場合、cpubindおよびmembindポリシーが[numactl](https://linux.die.net/man/8/numactl)を使用して割り当てられます。このフィールドは文字列型です。フィールド値は "0,1" のようなNUMAノードのIDです。

- `config`：フィールドの内容は、`server_configs`の`cdc`コンテンツとマージされます（2つのフィールドが重複している場合、このフィールドの内容が効果を持ちます）。次に、構成ファイルが生成され、`host`で指定されたマシンに送信されます。

- `os`：`host`で指定されたマシンのオペレーティングシステムです。このフィールドが指定されていない場合、デフォルト値は`global`の`os`の値です。

- `arch`：`host`で指定されたマシンのアーキテクチャです。このフィールドが指定されていない場合、デフォルト値は`global`の`arch`の値です。

- `resource_control`：サービスのリソース制御です。このフィールドが構成されている場合、フィールドの内容は`global`の`resource_control`コンテンツとマージされます（2つのフィールドが重複している場合、このフィールドの内容が効果を持ちます）。次に、systemd構成ファイルが生成され、`host`で指定されたマシンに送信されます。 `resource_control`の構成ルールは、`global`の`resource_control`の内容と同じです。

- `ticdc_cluster_id`：サービスに対応するTiCDCクラスターIDを指定します。このフィールドが指定されていない場合、サービスはデフォルトのTiCDCクラスターに参加します。このフィールドは、TiDB v6.3.0以降のバージョンでのみ効果があります。

上記のフィールドについて、デプロイ後にこれらの構成済みフィールドを変更することはできません：

- `host`
- `port`
- `deploy_dir`
- `data_dir`
- `log_dir`
- `arch`
- `os`
- `ticdc_cluster_id`

`cdc_servers`の構成例は次のようになります:

```yaml
cdc_servers:
  - host: 10.0.1.20
    gc-ttl: 86400
    data_dir: "/cdc-data"
  - host: 10.0.1.21
    gc-ttl: 86400
    data_dir: "/cdc-data"
```

### `tispark_masters`

`tispark_masters`は、TiSparkのマスターノードがデプロイされているマシンを指定します。また、各マシンのサービス構成も指定します。 `tispark_masters`は配列です。各配列要素には次のフィールドが含まれます。

- `host`：TiSparkマスターがデプロイされているマシンを指定します。フィールド値はIPアドレスであり、必須です。

- `listen_host`：マシンに複数のIPアドレスがある場合、 `listen_host` はサービスのリッスンIPアドレスを指定します。デフォルト値は `0.0.0.0` です。

- `ssh_port`：対象のマシンに接続するためのSSHポートを指定します。指定されていない場合、`ssh_port`は`global`セクションの`ssh_port`が使用されます。

- `port`：Sparkのリッスンポートです。ノード間の通信に使用されます。デフォルト値は `7077` です。

- `web_port`：SparkのWebポートで、Webサービスとタスクの状態を提供します。デフォルト値は `8080` です。

- `deploy_dir`：デプロイディレクトリを指定します。指定されていない場合、または相対ディレクトリとして指定されている場合、ディレクトリは`global`で構成された`deploy_dir`ディレクトリに従って生成されます。

- `java_home`：使用するJRE環境のパスを指定します。このパラメータは`JAVA_HOME`システム環境変数に対応します。

- `spark_config`：TiSparkサービスを構成するための設定です。次に、構成ファイルが生成され、`host`で指定されたマシンに送信されます。

- `spark_env`：Sparkが起動する際の環境変数を構成します。

- `os`：`host`で指定されたマシンのオペレーティングシステムです。このフィールドが指定されていない場合、デフォルト値は`global`の`os`の値です。

- `arch`：`host`で指定されたマシンのアーキテクチャです。このフィールドが指定されていない場合、デフォルト値は`global`の`arch`の値です。

上記のフィールドについて、デプロイ後にこれらの構成済みフィールドを変更することはできません：

- `host`
- `listen_host`
- `port`
- `web_port`
- `deploy_dir`
- `arch`
- `os`

`cdc_servers`の構成例は次のようになります:

```yaml
tispark_masters:
  - host: 10.0.1.21
    spark_config:
      spark.driver.memory: "2g"
      spark.eventLog.enabled: "False"
      spark.tispark.grpc.framesize: 2147483647
      spark.tispark.grpc.timeout_in_sec: 100
      spark.tispark.meta.reload_period_in_sec: 60
      spark.tispark.request.command.priority: "Low"
      spark.tispark.table.scan_concurrency: 256
    spark_env:
      SPARK_EXECUTOR_CORES: 5
      SPARK_EXECUTOR_MEMORY: "10g"
      SPARK_WORKER_CORES: 5
      SPARK_WORKER_MEMORY: "10g"
  - host: 10.0.1.22
```

### `tispark_workers`

`tispark_workers`は、TiSparkのワーカーノードがデプロイされているマシンを指定します。また、各マシンのサービス構成も指定します。 `tispark_workers`は配列です。各配列要素には次のフィールドが含まれます。

- `host`：TiSparkワーカーがデプロイされているマシンを指定します。フィールド値はIPアドレスであり、必須です。

- `listen_host`：マシンに複数のIPアドレスがある場合、 `listen_host` はサービスのリッスンIPアドレスを指定します。デフォルト値は `0.0.0.0` です。

- `ssh_port`：対象のマシンに接続するためのSSHポートを指定します。指定されていない場合、`ssh_port`は`global`セクションの`ssh_port`が使用されます。

- `port`：Sparkのリッスンポートです。ノード間の通信に使用されます。デフォルト値は `7077` です。

- `web_port`：SparkのWebポートで、Webサービスとタスクの状態を提供します。デフォルト値は `8080` です。

- `deploy_dir`：デプロイディレクトリを指定します。指定されていない場合、または相対ディレクトリとして指定されている場合、ディレクトリは`global`で構成された`deploy_dir`ディレクトリに従って生成されます。

- `java_home`：使用するJRE環境のパスを指定します。このパラメータは`JAVA_HOME`システム環境変数に対応します。

- `os`：`host`で指定されたマシンのオペレーティングシステムです。このフィールドが指定されていない場合、デフォルト値は`global`の`os`の値です。

- `arch`：`host`で指定されたマシンのアーキテクチャです。このフィールドが指定されていない場合、デフォルト値は`global`の`arch`の値です。

上記のフィールドについて、デプロイ後にこれらの構成済みフィールドを変更することはできません：

- `host`
- `listen_host`
- `port`
- `web_port`
- `deploy_dir`
- `arch`
- `os`

`monitoring_servers`
```
`monitoring_servers`は、Prometheusサービスが展開されたマシンを指定します。また、各マシンでのサービス構成も指定します。 `monitoring_servers`は配列です。各配列要素には次のフィールドが含まれます：

- `host`：監視サービスが展開されるマシンを指定します。フィールド値はIPアドレスであり、必須です。

- `ng_port`：NgMonitoringがリッスンするポートを指定します。TiUP v1.7.0で導入されました。このフィールドは[Continuous Profiling](/dashboard/dashboard-profiling.md)および[Top SQL](/dashboard/top-sql.md)をサポートしています。デフォルト値は`12020`です。

- `ssh_port`：操作のためにターゲットマシンに接続するSSHポートを指定します。指定されていない場合、 `global`セクションの `ssh_port`が使用されます。

- `port`：Prometheusサービスのリスニングポートです。デフォルト値は`9090`です。

- `deploy_dir`：展開ディレクトリを指定します。指定されていないか、相対ディレクトリとして指定されている場合、 `global`で構成された`deploy_dir`ディレクトリに従ってディレクトリが生成されます。

- `data_dir`：データディレクトリを指定します。指定されていないか、相対ディレクトリとして指定されている場合、 `global`で構成された`data_dir`ディレクトリに従ってディレクトリが生成されます。

- `log_dir`：ログディレクトリを指定します。指定されていないか、相対ディレクトリとして指定されている場合、ログは `global`で構成された`log_dir`ディレクトリに従って生成されます。

- `numa_node`：インスタンスにNUMAポリシーを割り当てます。このフィールドを指定する前に、ターゲットマシンに[numactl](https://linux.die.net/man/8/numactl)がインストールされていることを確認する必要があります。このフィールドが指定されている場合、[numactl](https://linux.die.net/man/8/numactl)を使用してcpubindおよびmembindポリシーが割り当てられます。このフィールドは文字列型です。フィールド値はNUMAノードのID、例えば "0,1" です。

- `storage_retention`：Prometheus監視データの保持期間です。デフォルト値は `"30d"` です。

- `rule_dir`：完全な `*.rules.yml`ファイルを含むローカルディレクトリを指定します。これらのファイルは、Prometheusのルールとしてクラスタ構成の初期化フェーズ中にターゲットマシンに転送されます。

- `remote_config`：Prometheusデータをリモートに書き込む、またはリモートからデータを読み取ることをサポートします。このフィールドには2つの構成があります：
    - `remote_write`：Prometheusドキュメント[`<remote_write>`](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#remote_write)を参照してください。
    - `remote_read`：Prometheusドキュメント[`<remote_read>`](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#remote_read)を参照してください。

- `external_alertmanagers`：`external_alertmanagers`フィールドが構成されている場合、Prometheusはクラスタ外部のAlertmanagerに対する構成動作を実行します。このフィールドは配列で、各要素は外部Alertmanagerであり、 `host`および`web_port`フィールドから構成されます。

- `os`：`host`で指定されたマシンのオペレーティングシステムです。このフィールドが指定されていない場合、デフォルト値は`global`の`os`値です。

- `arch`：`host`で指定されたマシンのアーキテクチャです。このフィールドが指定されていない場合、デフォルト値は`global`の`arch`値です。

- `resource_control`：サービスのリソース制御です。このフィールドが構成されている場合、フィールド内容は`global`での`resource_control`内容とマージされます（両フィールドが重複している場合、このフィールドの内容が適用されます）。その後、systemd構成ファイルが生成され、`host`で指定されたマシンに送信されます。 `resource_control`の構成ルールは、 `global`での`resource_control`内容と同じです。

上記のフィールドについて、以下の構成されたフィールドをデプロイ後に変更することはできません：

- `host`
- `port`
- `deploy_dir`
- `data_dir`
- `log_dir`
- `arch`
- `os`

`monitoring_servers`の構成例は次のようになります：

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
      - url: http://127.0.0.1:8003/read
      external_alertmanagers:
      - host: 10.1.1.1
        web_port: 9093
      - host: 10.1.1.2
        web_port: 9094
```

### `grafana_servers`

`grafana_servers`は、Grafanaサービスが展開されるマシンを指定します。また、各マシンでのサービス構成も指定します。 `grafana_servers` は配列です。各配列要素には次のフィールドが含まれます：

- `host`：Grafanaサービスが展開されるマシンを指定します。フィールド値はIPアドレスであり、必須です。

- `ssh_port`：操作のためにターゲットマシンに接続するSSHポートを指定します。指定されていない場合、 `global`セクションの `ssh_port`が使用されます。

- `port`：Grafanaサービスのリスニングポートです。デフォルト値は`3000`です。

- `deploy_dir`：展開ディレクトリを指定します。指定されていないか、相対ディレクトリとして指定されている場合、 `global`で構成された`deploy_dir`ディレクトリに従ってディレクトリが生成されます。

- `os`：`host`で指定されたマシンのオペレーティングシステムです。このフィールドが指定されていない場合、デフォルト値は`global`の`os`値です。

- `arch`：`host`で指定されたマシンのアーキテクチャです。このフィールドが指定されていない場合、デフォルト値は`global`の`arch`値です。

- `username`：Grafanaログインインターフェース上のユーザー名です。

- `password`：Grafanaに対応するパスワードです。

- `dashboard_dir`：完全な`dashboard(*.json)`ファイルを含むローカルディレクトリを指定します。これらのファイルは、Grafanaのダッシュボードとしてクラスタ構成の初期化フェーズ中にターゲットマシンに転送されます。

- `resource_control`：サービスのリソース制御です。このフィールドが構成されている場合、フィールド内容は`global`での`resource_control`内容とマージされます（両フィールドが重複している場合、このフィールドの内容が適用されます）。その後、systemd構成ファイルが生成され、`host`で指定されたマシンに送信されます。 `resource_control`の構成ルールは、`global`での`resource_control`内容と同じです。

> **注：**
>
> `grafana_servers`の`dashboard_dir`フィールドが構成されている場合、 `tiup cluster rename`コマンドを実行してクラスタ名を変更した後、次の操作を実行する必要があります：
>
> 1. ローカルダッシュボードディレクトリ内の`*.json`ファイルの `datasource`フィールドの値を新しいクラスタ名に更新する（`datasource`はクラスタ名に基づいて命名されているため）。
> 2. `tiup cluster reload -R grafana`コマンドを実行します。

上記のフィールドについて、以下の構成されたフィールドをデプロイ後に変更することはできません：

- `host`
- `port`
- `deploy_dir`
- `arch`
- `os`

`grafana_servers`の構成例は次のようになります：

```yaml
grafana_servers:
  - host: 10.0.1.11
    dashboard_dir: /local/dashboard/dir
```

### `alertmanager_servers`

`alertmanager_servers`は、Alertmanagerサービスが展開されたマシンを指定します。また、各マシンでのサービス構成も指定します。 `alertmanager_servers`は配列です。各配列要素には次のフィールドが含まれます：

- `host`：Alertmanagerサービスが展開されるマシンを指定します。フィールドの値はIPアドレスであり、必須です。

- `ssh_port`：操作のためにターゲットマシンに接続するSSHポートを指定します。指定されていない場合、 `global`セクションの `ssh_port`が使用されます。

- `web_port`：AlertmanagerがWebサービスを提供するために使用するポートを指定します。デフォルト値は`9093`です。

- `cluster_port`：1つのAlertmanagerと他のAlertmanager間の通信ポートを指定します。デフォルト値は`9094`です。

- `deploy_dir`：展開ディレクトリを指定します。指定されていないか、相対ディレクトリとして指定されている場合、 `global`で構成された`deploy_dir`ディレクトリに従ってディレクトリが生成されます。

- `data_dir`：データディレクトリを指定します。指定されていないか、相対ディレクトリとして指定されている場合、 `global`で構成された`data_dir`ディレクトリに従ってディレクトリが生成されます。

- `log_dir`：ログディレクトリを指定します。指定されていないか、相対ディレクトリとして指定されている場合、ログは `global`で構成された`log_dir`ディレクトリに従って生成されます。
- `numa_node`: インスタンスにNUMAポリシーを割り当てます。このフィールドを指定する前に、対象のマシンに[numactl](https://linux.die.net/man/8/numactl)がインストールされていることを確認する必要があります。このフィールドが指定されている場合、cpubindとmembindポリシーは[numactl](https://linux.die.net/man/8/numactl)を使用して割り当てられます。このフィールドは文字列タイプです。フィールド値は"0,1"などのNUMAノードのIDです。

- `config_file`: クラスター構成の初期化フェーズ中に、ターゲットマシンに転送されるローカルファイルを指定します。これはAlertmanagerの構成です。

- `os`: `host`で指定されたマシンのオペレーティングシステムです。このフィールドが指定されていない場合、デフォルト値は`global`の`os`の値です。

- `arch`: `host`で指定されたマシンのアーキテクチャです。このフィールドが指定されていない場合、デフォルト値は`global`の`arch`の値です。

- `resource_control`: サービスのリソース制御を行います。このフィールドが設定されている場合、フィールドの内容は`global`の`resource_control`の内容とマージされます（両フィールドが重複する場合、このフィールドの内容が有効になります）。その後、systemd構成ファイルが生成され、`host`で指定されたマシンに送信されます。`resource_control`の構成ルールは、`global`の`resource_control`の内容と同じです。

上記のフィールドについては、デプロイ後にこれらの構成済みのフィールドを変更することはできません：

- `host`
- `web_port`
- `cluster_port`
- `deploy_dir`
- `data_dir`
- `log_dir`
- `arch`
- `os`

`alertmanager_servers`の構成例は以下のとおりです：

```yaml
alertmanager_servers:
  - host: 10.0.1.11
    config_file: /local/config/file
  - host: 10.0.1.12
    config_file: /local/config/file
```