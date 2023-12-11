---
title: TiUPを使用してDMクラスタを維持する
summary: TiUPを使用してDMクラスタを維持する方法について学びます。
aliases: ['/docs/tidb-data-migration/dev/cluster-operations/']
---

# TiUPを使用してDMクラスタを維持する

このドキュメントでは、TiUP DMコンポーネントを使用してDMクラスタを維持する方法について紹介します。

まだDMクラスタを展開していない場合は、[TiUPを使用してDMクラスタを展開する](/dm/deploy-a-dm-cluster-using-tiup.md)を参照してください。

> **注意:**
>
> - 次のコンポーネント間のポートが相互に接続されていることを確認してください
>     - DM-masterノード間の`peer_port`（デフォルトは`8291`）が相互に接続されています。
>     - 各DM-masterノードがすべてのDM-workerノードの`port`に接続できる（デフォルトは`8262`）こと。
>     - 各DM-workerノードがすべてのDM-masterノードの`port`に接続できる（デフォルトは`8261`）こと。
>     - TiUPノードがすべてのDM-masterノードの`port`に接続できる（デフォルトは`8261`）こと。
>     - TiUPノードがすべてのDM-workerノードの`port`に接続できる（デフォルトは`8262`）こと。

TiUP DMコンポーネントのヘルプ情報は、次のコマンドを実行して取得できます。

```bash
tiup dm --help
```

```
プロダクション用にDMクラスタを展開する

使用法:
  tiup dm [flags]
  tiup dm [command]

利用可能なコマンド:
  deploy      プロダクション用にDMクラスタを展開する
  start       DMクラスタを開始する
  stop        DMクラスタを停止する
  restart     DMクラスタを再起動する
  list        すべてのクラスタを一覧表示する
  destroy     指定されたDMクラスタを破棄する
  audit       クラスタ操作の監査ログを表示する
  exec        DMクラスタ内のホストでシェルコマンドを実行
  edit-config DMクラスタの構成を編集する
  display     DMクラスタの情報を表示する
  reload      DMクラスタの構成を再読み込みし、必要に応じて再起動する
  upgrade     指定されたDMクラスタをアップグレードする
  patch       リモートパッケージを指定されたパッケージで置き換え、サービスを再起動する
  scale-out   DMクラスタをスケーリングアウトする
  scale-in    DMクラスタをスケーリングインする
  import      存在するDM 1.0クラスタをdm-ansibleからインポートし、2.0バージョンを再展開する
  help        任意のコマンドに関するヘルプ

フラグ:
  -h, --help               tiup-dmのヘルプ
      --native-ssh         ローカルシステムにインストールされているネイティブSSHクライアントを使用してビルドインされたSSHクライアントの代わりに使用します。
      --ssh-timeout int    SSHを介してホストに接続するタイムアウト（秒単位）、SSH接続を必要としない操作には無視されます（デフォルトは5）
  -v, --version            tiup-dmのバージョン
      --wait-timeout int   操作の完了を待機するタイムアウト（秒単位）、適合しない操作には無視されます（デフォルトは60）
  -y, --yes                すべての確認をスキップし、「はい」と仮定します
```

## クラスタリストを表示する

クラスタが正常に展開された後、次のコマンドを実行してクラスタリストを表示します：

{{< copyable "shell-root" >}}

```bash
tiup dm list
```

```
Name  User  Version  Path                                  PrivateKey
----  ----  -------  ----                                  ----------
prod-cluster  tidb  ${version}  /root/.tiup/storage/dm/clusters/test  /root/.tiup/storage/dm/clusters/test/ssh/id_rsa
```

## クラスタを開始する

クラスタが正常に展開された後、次のコマンドを実行してクラスタを開始します：

{{< copyable "shell-regular" >}}

```shell
tiup dm start prod-cluster
```

クラスタ名を忘れた場合は、`tiup dm list`を実行してクラスタリストを表示します。

## クラスタの状態を確認する

TiUPは`tiup dm display`コマンドを提供しており、このコマンドを使用するとクラスタ内の各コンポーネントの状態を簡単に確認できます。このコマンドを使用すると、各マシンにログインしてコンポーネントの状態を確認する必要はありません。コマンドの使用法は次のとおりです：

{{< copyable "shell-root" >}}

```bash
tiup dm display prod-cluster
```

```
dm Cluster: prod-cluster
dm Version: ${version}
ID                 Role          Host          Ports      OS/Arch       Status     Data Dir                           Deploy Dir
--                 ----          ----          -----      -------       ------     --------                           ----------
172.19.0.101:9093  alertmanager  172.19.0.101  9093/9094  linux/x86_64  Up         /home/tidb/data/alertmanager-9093  /home/tidb/deploy/alertmanager-9093
172.19.0.101:8261  dm-master     172.19.0.101  8261/8291  linux/x86_64  Healthy|L  /home/tidb/data/dm-master-8261     /home/tidb/deploy/dm-master-8261
172.19.0.102:8261  dm-master     172.19.0.102  8261/8291  linux/x86_64  Healthy    /home/tidb/data/dm-master-8261     /home/tidb/deploy/dm-master-8261
172.19.0.103:8261  dm-master     172.19.0.103  8261/8291  linux/x86_64  Healthy    /home/tidb/data/dm-master-8261     /home/tidb/deploy/dm-master-8261
172.19.0.101:8262  dm-worker     172.19.0.101  8262       linux/x86_64  Free       /home/tidb/data/dm-worker-8262     /home/tidb/deploy/dm-worker-8262
172.19.0.102:8262  dm-worker     172.19.0.102  8262       linux/x86_64  Free       /home/tidb/data/dm-worker-8262     /home/tidb/deploy/dm-worker-8262
172.19.0.103:8262  dm-worker     172.19.0.103  8262       linux/x86_64  Free       /home/tidb/data/dm-worker-8262     /home/tidb/deploy/dm-worker-8262
172.19.0.101:3000  grafana       172.19.0.101  3000       linux/x86_64  Up         -                                  /home/tidb/deploy/grafana-3000
172.19.0.101:9090  prometheus    172.19.0.101  9090       linux/x86_64  Up         /home/tidb/data/prometheus-9090    /home/tidb/deploy/prometheus-9090
```

`Status`列では、サービスが正常に実行されているかを示すために`Up`または`Down`を使用します。

DM-masterコンポーネントには、状態に`|L`が付加されることがあり、これはDM-masterノードがリーダーであることを示します。DM-workerコンポーネントには`Free`が付加されることがあり、現在のDM-workerノードが上流にバインドされていないことを示します。

## クラスタをスケーリングインする

クラスタをスケーリングインすると、一部のノードをオフラインにします。この操作では、指定されたノードをクラスタから削除し、残りのデータファイルを削除します。

クラスタをスケーリングインする際、次の順序でDM-masterおよびDM-workerコンポーネントでDM操作が実行されます：

1. コンポーネントプロセスを停止する。
2. DM-masterのAPIを呼び出して`member`を削除する。
3. ノードに関連するデータファイルをクリーンアップする。

スケーリングインコマンドの基本的な使用法：

```bash
tiup dm scale-in <cluster-name> -N <node-id>
```

このコマンドを使用するには、少なくともクラスタ名とノードIDを指定する必要があります。前のセクションで`tiup dm display`コマンドを使用してノードIDを取得する必要があります。

例えば、DM-workerノードを`172.16.5.140`（DM-masterのスケーリングインと同様）にスケーリングインする場合は、次のコマンドを実行します：

{{< copyable "shell-regular" >}}

```bash
tiup dm scale-in prod-cluster -N 172.16.5.140:8262
```

## クラスタをスケーリングアウトする

スケーリングアウト操作は、展開と似た内部ロジックを持っています：TiUP DMコンポーネントはまずノードのSSH接続を確認し、ターゲットノードに必要なディレクトリを作成し、その後展開操作を実行し、ノードサービスを開始します。

例えば、`prod-cluster`クラスタ内のDM-workerノードをスケーリングアウトする場合は、次の手順を実行します（DM-masterのスケーリングアウトも同様です）：

1. `scale.yaml`ファイルを作成し、新しいワーカーノードの情報を追加します：

    > **注意:**
    >
    > 既存のノードではなく、新しいノードの説明のみを含むトポロジファイルを作成する必要があります。
    > デプロイディレクトリなどの他の構成項目については、[TiUP構成パラメータの例](https://github.com/pingcap/tiup/blob/master/embed/examples/dm/topology.example.yaml)を参照してください。

    ```yaml
    ---

    worker_servers:
      - host: 172.16.5.140

    ```

```
2. スケールアウト操作を実行します。TiUP DM は、`scale.yaml` で記述されたポート、ディレクトリ、その他の情報に基づいて、対応するノードをクラスターに追加します。

    {{< copyable "shell-regular" >}}

    ```shell
    tiup dm scale-out prod-cluster scale.yaml
    ```

    コマンドを実行した後、`tiup dm display prod-cluster` を実行して、スケールアウトされたクラスターの状態を確認できます。

## ローリングアップグレード

> **注意:**
>
> v2.0.5 以降、dmctl は[クラスターのデータソースとタスク構成のエクスポートとインポート](/dm/dm-export-import-config.md)をサポートしています。
>
> アップグレード前に、`config export` を使用してクラスターの構成ファイルをエクスポートできます。アップグレード後、以前のバージョンにダウングレードする必要がある場合は、最初に以前のクラスターを再デプロイしてから、`config import` を使用して前の構成ファイルをインポートできます。
>
> v2.0.5 より前のクラスターでは、dmctl v2.0.5 以降を使用してデータソースおよびタスクの構成ファイルをエクスポートおよびインポートできます。
>
> v2.0.2 より後のクラスターでは、現在、中継 worker に関連する構成を自動的にインポートすることはサポートされていません。`start-relay` コマンドを使用して、手動で[リレーログを開始](/dm/relay-log.md#enable-and-disable-relay-log)できます。

ローリングアップグレードプロセスは、アプリケーションにできるだけ透明な形で行われ、ビジネスに影響を与えません。操作は異なるノードによって異なります。

### アップグレードコマンド

`tiup dm upgrade` コマンドを実行して、DM クラスターをアップグレードできます。たとえば、次のコマンドはクラスターを `${version}` にアップグレードします。このコマンドを実行する前に `${version}` を必要なバージョンに変更してください。

{{< copyable "shell-regular" >}}

```bash
tiup dm upgrade prod-cluster ${version}
```

## 構成の更新

コンポーネントの構成を動的に更新する場合、TiUP DM コンポーネントは各クラスターに現在の構成を保存します。この構成を編集するには、`tiup dm edit-config <cluster-name>` コマンドを実行します。例:

{{< copyable "shell-regular" >}}

```bash
tiup dm edit-config prod-cluster
```

TiUP DM は vi エディタで構成ファイルを開きます。他のエディタを使用する場合は、`EDITOR` 環境変数を使用してエディタをカスタマイズします（`export EDITOR=nano` のように）。ファイルの編集後は、変更を保存します。新しい構成をクラスターに適用するには、次のコマンドを実行します:

{{< copyable "shell-regular" >}}

```bash
tiup dm reload prod-cluster
```

このコマンドは構成を対象のマシンに送信し、構成を有効にするためにクラスターを再起動します。

## コンポーネントの更新

通常のアップグレードには `upgrade` コマンドを使用できます。ただし、デバッグなどのシナリオでは、現在実行中のコンポーネントを一時的なパッケージで置き換える必要がある場合があります。その場合は `patch` コマンドを使用します。

{{< copyable "shell-root" >}}

```bash
tiup dm patch --help
```

```
リモートパッケージを指定されたパッケージで置き換え、サービスを再起動します

使用法:
  tiup dm patch <cluster-name> <package-path> [flags]

フラグ:
  -h, --help                   patch のヘルプ
  -N, --node strings           指定したノード
      --overwrite              今後のスケールアウト操作でこのパッケージを使用します
  -R, --role strings           指定した役割
      --transfer-timeout int   dm-master リーダーを転送する際のタイムアウト秒数 (デフォルト 600)

グローバルフラグ:
      --native-ssh            ローカルシステムにインストールされたネイティブSSHクライアントを使用します
      --ssh-timeout int       SSH経由でホストに接続するタイムアウト秒数。SSH接続を必要としない操作には適用されません。 (デフォルト 5)
      --wait-timeout int      操作の完了を待機する秒数。フィットしない操作には適用されません。 (デフォルト 60)
  -y, --yes                   すべての確認をスキップし、'はい' を仮定します
```

DM-master のホットフィックスパッケージが `/tmp/dm-master-hotfix.tar.gz` にあり、クラスター内のすべての DM-master パッケージを置き換えたい場合は、次のコマンドを実行します:

{{< copyable "shell-regular" >}}

```bash
tiup dm patch prod-cluster /tmp/dm-master-hotfix.tar.gz -R dm-master
```

また、クラスター内の1つの DM-master パッケージのみを置き換えることもできます:

{{< copyable "shell-regular" >}}

```bash
tiup dm patch prod-cluster /tmp/dm--hotfix.tar.gz -N 172.16.4.5:8261
```

## DM-Ansible を使用して展開された DM 1.0 クラスターのインポートおよびアップグレード

> **注意:**
>
> - DM 1.0 クラスターにおける DM Portal コンポーネントのインポートは、TiUP ではサポートされていません。
> - インポートする前に元のクラスターを停止する必要があります。
> - 2.0 にアップグレードする必要のあるタスクについては、`stop-task` を実行しないでください。
> - TiUP は、DM クラスター v2.0.0-rc.2 またはそれ以降にのみインポートをサポートしています。
> - `import` コマンドは、DM 1.0 クラスターから新しい DM 2.0 クラスターへのデータのインポートに使用されます。既存の DM 2.0 クラスターに DM マイグレーションタスクをインポートする必要がある場合は、[v1.0.x から v2.0+ への TiDB データ移行の手動アップグレード](/dm/manually-upgrade-dm-1.0-to-2.0.md)を参照してください。
> - 一部のコンポーネントの展開ディレクトリは、元のクラスターと異なる場合があります。詳細は `display` コマンドを実行して確認できます。
> - インポートの前に `tiup update --self && tiup update dm` を実行して、TiUP DM コンポーネントが最新バージョンであることを確認してください。
> - インポート後は、1つの DM-master ノードのみがクラスターに存在します。DM-master をスケールアウトする場合は、[クラスターをスケールアウトする](#scale-out-a-cluster)を参照してください。

TiUP がリリースされる前は、一般的に DM-Ansible を使用して DM クラスターを展開していました。DM-Ansible によって展開された DM 1.0 クラスターを TiUP で引き継ぐには、`import` コマンドを使用します。

例えば、DM Ansible を使用して展開されたクラスターをインポートするには:

{{< copyable "shell-regular" >}}

```bash
tiup dm import --dir=/path/to/dm-ansible --cluster-version ${version}
```

`tiup list dm-master` を実行して、TiUP がサポートする最新のクラスターバージョンを表示できます。

`import` コマンドの手順は次の通りです:

1. TiUP は、DM-Ansible を使用して以前に展開された DM クラスターに基づいて topology ファイル [`topology.yml`](https://github.com/pingcap/tiup/blob/master/embed/examples/dm/topology.example.yaml) を生成します。
2. topology ファイルが生成されたことを確認した後、それを使用して v2.0 以降の DM クラスターを展開できます。

展開が完了したら、`tiup dm start` コマンドを実行してクラスターを開始し、DM カーネルのアップグレードプロセスを開始します。

## 操作ログの表示

操作ログを表示するには、`audit` コマンドを使用します。`audit` コマンドの使用方法は次の通りです:

```bash
使用法:
  tiup dm audit [audit-id] [flags]

フラグ:
  -h, --help   audit のヘルプ
```

`[audit-id]` 引数が指定されていない場合、次のコマンドは実行されたコマンドのリストを表示します。たとえば:

{{< copyable "shell-regular" >}}

```bash
tiup dm audit
```

```
ID      Time                  Command
--      ----                  -------
4D5kQY  2020-08-13T05:38:19Z  tiup dm display test
4D5kNv  2020-08-13T05:36:13Z  tiup dm list
4D5kNr  2020-08-13T05:36:10Z  tiup dm deploy -p prod-cluster ${version} ./examples/dm/minimal.yaml
```

最初の列は `audit-id` です。特定のコマンドの実行ログを表示するには、次のように `audit-id` 引数を渡します:

{{< copyable "shell-regular" >}}

```bash
tiup dm audit 4D5kQY
```

## DM クラスター内のホストでコマンドを実行する

DM クラスター内のホストでコマンドを実行するには、`exec` コマンドを使用します。`exec` コマンドの使用方法は次の通りです:

```bash
使用法:
  tiup dm exec <cluster-name> [flags]

フラグ:
      --command string   クラスターホストで実行するコマンド (デフォルト "ls")
  -h, --help             exec のヘルプ
  -N, --node strings     指定したノードで実行のみ
  -R, --role strings     指定した役割のホストで実行のみ
      --sudo             root 権限を使用する (デフォルト false)
```

たとえば、すべての DM ノードで `ls /tmp` を実行するには、次のコマンドを実行します:

{{< copyable "shell-regular" >}}

```bash
tiup dm exec prod-cluster --command='ls /tmp'
```

## dmctl

TiUP には DM クラスターコントローラーである `dmctl` が統合されています。
```
```bash
dmctlを使用するには、次のコマンドを実行してください:

```bash
tiup dmctl [args]
```

dmctlのバージョンを指定してください。このコマンドを実行する前に、`${version}`を必要なバージョンに変更してください:

```
tiup dmctl:${version} [args]
```

以前のdmctlコマンドでソースを追加する方法は `dmctl --master-addr master1:8261 operate-source create /tmp/source1.yml` でした。TiUPにdmctlが統合された後のコマンドは以下の通りです:

{{< copyable "shell-regular" >}}

```bash
tiup dmctl --master-addr master1:8261 operate-source create /tmp/source1.yml
```

## クラスタにシステムのネイティブSSHクライアントを使用する

クラスタマシン上で上記の操作を実行する場合、TiUPに組み込まれているSSHクライアントを使用してクラスタに接続し、コマンドを実行します。ただし、一部のシナリオでは、制御マシンシステム固有のSSHクライアントを使用して、このようなクラスタ操作を実行する必要がある場合があります。例:

- 認証のためにSSHプラグインを使用する
- カスタマイズされたSSHクライアントを使用する

その場合は、`--native-ssh`コマンドラインフラグを使用して、システムネイティブのコマンドラインツールを有効にすることができます:

- クラスタをデプロイ: `tiup dm deploy <cluster-name> <version> <topo> --native-ssh`。`<cluster-name>`にはクラスタの名前を、`<version>`にはデプロイされるDMバージョン（例:`v7.4.0`など）、`<topo>`にはトポロジファイル名を入力してください。
- クラスタを開始: `tiup dm start <cluster-name> --native-ssh`。
- クラスタをアップグレード: `tiup dm upgrade ... --native-ssh`

上記のすべてのクラスタ操作コマンドに`--native-ssh`を追加して、システムのネイティブSSHクライアントを使用できます。

各コマンドにこのフラグを追加するのを避けるために、`TIUP_NATIVE_SSH`システム変数を使用して、ローカルSSHクライアントを使用するかどうかを指定できます:

```sh
export TIUP_NATIVE_SSH=true
# または
export TIUP_NATIVE_SSH=1
# または
export TIUP_NATIVE_SSH=enable
```

この環境変数と`--native-ssh`を同時に指定する場合は、`--native-ssh`が優先されます。

> **注意:**
>
> クラスタデプロイメントのプロセス中に接続するためにパスワードを使用する場合や、キーファイルに`passphrase`が構成されている場合は、制御マシンに`sshpass`がインストールされていることを確認してください。そうでない場合、タイムアウトエラーが発生します。