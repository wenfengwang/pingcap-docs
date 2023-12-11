---
title: TiUPを使用してDMクラスターをデプロイする
summary: TiUP DMを使用してTiDBデータマイグレーションをデプロイする方法を学びます。
aliases: ['/docs/tidb-data-migration/dev/deploy-a-dm-cluster-using-ansible/','/docs/tools/dm/deployment/']
---

# TiUPを使用してDMクラスターをデプロイする

[TiUP](https://github.com/pingcap/tiup)は、TiDB 4.0で導入されたクラスター運用およびメンテナンスツールです。TiUPは、Golangで書かれたクラスター管理コンポーネントである[TiUP DM](/dm/maintain-dm-using-tiup.md)を提供します。TiUP DMを使用することで、DMクラスターのデプロイ、起動、停止、破棄、スケーリング、アップグレード、およびDMクラスターのパラメーターの管理を簡単に行うことができます。

TiUPは、DM v2.0以降のDMバージョンのデプロイをサポートしています。このドキュメントでは、異なるトポロジーのDMクラスターのデプロイ方法について紹介します。

> **注意:**
>
> ターゲットマシンのオペレーティングシステムがSELinuxをサポートしている場合は、SELinuxが**無効**になっていることを確認してください。

## 前提条件

DMは完全なデータレプリケーションタスクを実行する際、DM-workerは1つのアップストリームデータベースにバインドされます。DM-workerはまずデータの完全な量をローカルにエクスポートし、その後データをダウンストリームデータベースにインポートします。したがって、ワーカーホストのスペースは、エクスポートするすべてのアップストリームテーブルを保存するのに十分な大きさでなければなりません。ストレージパスは、タスクを作成する際に後で指定されます。

また、DMクラスターを展開する際には、[ハードウェアおよびソフトウェアの要件](/dm/dm-hardware-and-software-requirements.md)を満たす必要があります。

## ステップ1: コントロールマシンにTiUPをインストールする

通常のユーザーアカウント（`tidb`ユーザーを例に取る）でコントロールマシンにログインします。以下のTiUPのインストールとクラスター管理操作はすべて`tidb`ユーザーによって実行することができます。

1. 以下のコマンドを実行してTiUPをインストールします。

    {{< copyable "shell-regular" >}}

    ```shell
    curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
    ```

    インストールが完了すると、`~/.bashrc`が変更されてTiUPがPATHに追加されます。そのため、それを使用するには新しいターミナルを開くか、グローバル環境変数`source ~/.bashrc`を再宣言する必要があります。

2. TiUP DMコンポーネントをインストールします。

    {{< copyable "shell-regular" >}}

    ```shell
    tiup install dm dmctl
    ```

## ステップ2: 初期化構成ファイルを編集する

意図したクラスタートポロジーに応じて、クラスターの初期化構成ファイルを手動で作成および編集する必要があります。

[YAML構成ファイルのテンプレート](https://github.com/pingcap/tiup/blob/master/embed/examples/dm/topology.example.yaml)に従ってYAML構成ファイル（例えば、`topology.yaml`という名前のファイル）を作成する必要があります。他のシナリオについては、適切に構成を編集してください。

`tiup dm template > topology.yaml`コマンドを使用して、素早く構成ファイルのテンプレートを生成できます。

3つのDM-master、3つのDM-worker、および1つの監視コンポーネントインスタンスをデプロイする構成の例は次のとおりです。

```yaml
# グローバル変数は構成内の他のすべてのコンポーネントに適用されます。コンポーネントインスタンスで特定の値が欠落している場合、対応するグローバル変数がデフォルト値として機能します。
global:
  user: "tidb"
  ssh_port: 22
  deploy_dir: "/dm-deploy"
  data_dir: "/dm-data"

server_configs:
  master:
    log-level: info
    # rpc-timeout: "30s"
    # rpc-rate-limit: 10.0
    # rpc-rate-burst: 40
  worker:
    log-level: info

master_servers:
  - host: 10.0.1.11
    name: master1
    ssh_port: 22
    port: 8261
    # peer_port: 8291
    # deploy_dir: "/dm-deploy/dm-master-8261"
    # data_dir: "/dm-data/dm-master-8261"
    # log_dir: "/dm-deploy/dm-master-8261/log"
    # numa_node: "0,1"
    # 以下の構成は、`server_configs.master`の値を上書きするために使用されます。
    config:
      log-level: info
      # rpc-timeout: "30s"
      # rpc-rate-limit: 10.0
      # rpc-rate-burst: 40
  - host: 10.0.1.18
    name: master2
    ssh_port: 22
    port: 8261
  - host: 10.0.1.19
    name: master3
    ssh_port: 22
    port: 8261
# DMクラスターの高可用性を保証する必要がない場合、1つのDM-masterノードのみを展開し、展開されるDM-workerノードの数は移行するアップストリームMySQL/MariaDBインスタンスの数を下回ってはいけません。
# DMクラスターの高可用性を保証するためには、3つのDM-masterノードを展開することをお勧めし、展開されるDM-workerノードの数はアップストリームインスタンスの数より2つ多い（たとえば、DM-workerノードの数はアップストリームインスタンスの数より2つ多い）。
worker_servers:
  - host: 10.0.1.12
    ssh_port: 22
    port: 8262
    # deploy_dir: "/dm-deploy/dm-worker-8262"
    # log_dir: "/dm-deploy/dm-worker-8262/log"
    # numa_node: "0,1"
    # 以下の構成は、`server_configs.worker`の値を上書きするために使用されます。
    config:
      log-level: info
  - host: 10.0.1.19
    ssh_port: 22
    port: 8262

monitoring_servers:
  - host: 10.0.1.13
    ssh_port: 22
    port: 9090
    # deploy_dir: "/tidb-deploy/prometheus-8249"
    # data_dir: "/tidb-data/prometheus-8249"
    # log_dir: "/tidb-deploy/prometheus-8249/log"

grafana_servers:
  - host: 10.0.1.14
    port: 3000
    # deploy_dir: /tidb-deploy/grafana-3000

alertmanager_servers:
  - host: 10.0.1.15
    ssh_port: 22
    web_port: 9093
    # cluster_port: 9094
    # deploy_dir: "/tidb-deploy/alertmanager-9093"
    # data_dir: "/tidb-data/alertmanager-9093"
    # log_dir: "/tidb-deploy/alertmanager-9093/log"

```

> **注意:**
> 
> - 1つのホストで多くのDM-workerを実行することは推奨されません。各DM-workerには少なくとも2つのCPUコアと4GiBのメモリを割り当てる必要があります。
>
> - 次のコンポーネント間のポートが相互接続されていることを確認してください:
>     - DM-masterノード間の`peer_port`（デフォルトでは`8291`）が相互接続されていること。
>     - 各DM-masterノードがすべてのDM-workerノードの`port`（デフォルトでは`8262`）に接続できること。
>     - 各DM-workerノードがすべてのDM-masterノードの`port`（デフォルトでは`8261`）に接続できること。
>     - TiUPノードがすべてのDM-masterノードの`port`（デフォルトでは`8261`）に接続できること。
>     - TiUPノードがすべてのDM-workerノードの`port`（デフォルトでは`8262`）に接続できること。

より詳しい`master_servers.host.config`パラメータの説明については、[master parameter](https://github.com/pingcap/dm/blob/master/dm/master/dm-master.toml)を参照してください。さらに詳しい`worker_servers.host.config`パラメータの説明については、[worker parameter](https://github.com/pingcap/dm/blob/master/dm/worker/dm-worker.toml)を参照してください。

## ステップ3: デプロイコマンドを実行する

> **注意:**
>
> TiUPを使用してTiDBを展開する際にセキュリティ認証のために秘密鍵や対話型パスワードを使用することができます:
>
> - 秘密鍵を使用する場合は、`-i`または`--identity_file`を使用して鍵のパスを指定できます;
> - パスワードを使用する場合は、パスワード対話ウィンドウに入るために`-p`フラグを追加します;
> - パスワードフリーログインがターゲットマシンに構成されている場合は、認証は不要です。

{{< copyable "shell-regular" >}}

```shell
tiup dm deploy ${name} ${version} ./topology.yaml -u ${ssh_user} [-p] [-i /home/root/.ssh/gcp_rsa]
```

このステップで使用されるパラメータは以下の通りです。

|パラメータ|説明|
|-|-|
|`${name}` | DMクラスターの名前、例: dm-test|
|`${version}` | DMクラスターのバージョン。`tiup list dm-master`を実行してサポートされている他のバージョンを確認できます。 |
|`./topology.yaml`| トポロジー構成ファイルのパス|
|`-u` または `--user`| ターゲットマシンにrootユーザーまたはsshおよびsudo権限を持つ他のユーザーアカウントとしてログインしてクラスターのデプロイを完了します。|
|-pまたは--password|ターゲットホストのパスワード。指定された場合、パスワード認証が使用されます。
|-iまたは--identity_file|SSHアイデンティティファイルのパス。指定された場合、公開鍵認証が使用されます（デフォルト "/root/.ssh/id_rsa"）。

出力ログの最後に```Deployed cluster 'dm-test' successfully```と表示されます。これはデプロイが成功したことを示します。

## ステップ4: TiUPで管理されているクラスタを確認

{{< copyable "shell-regular" >}}

```shell
tiup dm list
```

TiUPは複数のDMクラスタを管理することができます。上記のコマンドは、現在TiUPで管理されているすべてのクラスタの情報を出力します。これには名前、デプロイユーザー、バージョン、およびシークレットキー情報が含まれます。

```log
Name  User  Version  Path                                  PrivateKey
----  ----  -------  ----                                  ----------
dm-test  tidb  ${version}  /root/.tiup/storage/dm/clusters/dm-test  /root/.tiup/storage/dm/clusters/dm-test/ssh/id_rsa
```

## ステップ5: デプロイされたDMクラスタの状態を確認

`dm-test`クラスタの状態を確認するには、次のコマンドを実行します。

{{< copyable "shell-regular" >}}

```shell
tiup dm display dm-test
```

期待される出力には、インスタンスID、ロール、ホスト、リッスンポート、および状態（クラスタがまだ開始されていないため、状態は `Down`/`inactive` です）、およびディレクトリ情報が含まれます。

## ステップ6: DMクラスタを起動する

{{< copyable "shell-regular" >}}

```shell
tiup dm start dm-test
```

出力ログに```Started cluster 'dm-test' successfully```が含まれている場合、起動は成功しています。

## ステップ7: DMクラスタの実行状態を確認

TiUPを使用してDMクラスタの状態を確認します。

{{< copyable "shell-regular" >}}

```shell
tiup dm display dm-test
```

出力で`Status`が`Up`である場合、クラスタの状態は正常です。

## ステップ8: dmctlを使用したマイグレーションタスクの管理

dmctlはDMクラスタを制御するためのコマンドラインツールです。[TiUP経由でdmctlを使用することをお勧めします](/dm/maintain-dm-using-tiup.md#dmctl)。

dmctlはコマンドモードと対話モードの両方をサポートしています。詳細については、[dmctlを使用したDMクラスタの管理](/dm/dmctl-introduction.md#maintain-dm-clusters-using-dmctl)を参照してください。