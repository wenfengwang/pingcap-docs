---
title: TiUPを使用してオンラインTiDBクラスターをデプロイおよびメンテナンスする
summary: TiUPを使用してオンラインTiDBクラスターをデプロイおよびメンテナンスする手順を学びます。
aliases: ['/docs/dev/tiup/tiup-cluster/','/docs/dev/reference/tools/tiup/cluster/']
---

# TiUPを使用してオンラインTiDBクラスターをデプロイおよびメンテナンスする

このドキュメントでは、TiUPクラスターコンポーネントの使用方法に焦点を当てています。オンラインデプロイの完全な手順については、[TiUPを使用してTiDBクラスターをデプロイ](/production-deployment-using-tiup.md) を参照してください。

[ローカルテストデプロイに使用されるTiUP playgroundコンポーネント](/tiup/tiup-playground.md) と同様に、クラスター コンポーネントは本番環境に迅速にTiDBをデプロイします。 Playgroundと比較して、クラスターコンポーネントはアップグレード、スケーリング、さらに操作と監査を含むより強力な本番クラスター管理機能を提供します。

クラスターコンポーネントのヘルプ情報を取得するには、次のコマンドを実行します。

```bash
tiup cluster
```

```
Starting component `cluster`: /home/tidb/.tiup/components/cluster/v1.12.3/cluster
本番環境のTiDBクラスターをデプロイ

使い方:
  tiup cluster [command]

利用可能なコマンド:
  check       クラスターを事前チェックする
  deploy      本番環境のクラスターをデプロイ
  start       TiDBクラスターを開始します
  stop        TiDBクラスターを停止します
  restart     TiDBクラスターを再起動します
  scale-in    TiDBクラスターをスケールインします
  scale-out   TiDBクラスターをスケールアウトします
  destroy     指定されたクラスターを破棄します
  clean       (実験的) 指定されたクラスターをクリーンアップします
  upgrade     指定されたTiDBクラスターをアップグレードします
  display     TiDBクラスターの情報を表示します
  list        すべてのクラスターを一覧表示します
  audit       クラスター操作の監査ログを表示します
  import      TiDB-Ansibleから既存のTiDBクラスターをインポートします
  edit-config TiDBクラスターコンフィグを編集します
  reload      TiDBクラスターのコンフィグを再読み込みし、必要に応じて再起動します
  patch       リモートパッケージを指定されたパッケージで置き換え、サービスを再起動します
  help        任意のコマンドに関するヘルプ

フラグ:
  -c, --concurrency int     同時タスクの最大数を指定します（デフォルトは`5`です）
      --format string       (実験的) 出力のフォーマット。使用可能な値は[default, json] です (デフォルトは "default" です)
  -h, --help                TiUPのヘルプ
      --ssh string          (実験的) エグゼキュータータイプ。オプション値は 'builtin', 'system', 'none' です。
      --ssh-timeout uint    SSHを介してホストに接続するタイムアウト値（秒）。SSH接続を必要としない操作は無視されます。（デフォルトは 5 です）
  -v, --version             TiUPバージョン
      --wait-timeout uint   オペレーションの完了まで待機するタイムアウト値（秒）。適用しない操作は無視されます（デフォルトは `120` です）
  -y, --yes                 すべての確認をスキップし、 'yes' を仮定します
```

## クラスターのデプロイ

クラスターをデプロイするには、`tiup cluster deploy`コマンドを実行します。コマンドの使用法は次のとおりです。

```bash
tiup cluster deploy <cluster-name> <version> <topology.yaml> [flags]
```

このコマンドは、クラスター名、TiDBクラスターバージョン（たとえば`v7.4.0`など）、およびクラスターのトポロジファイルを指定する必要があります。

トポロジファイルの作成については、[例](https://github.com/pingcap/tiup/blob/master/embed/examples/cluster/topology.example.yaml)を参照してください。以下のファイルは、最も単純なトポロジの例です。

> **注意:**
>
> デプロイとスケーリングにTiUPクラスターコンポーネントで使用されるトポロジファイルは、[yaml](https://yaml.org/spec/1.2/spec.html)構文を使用して記述されているため、インデントが正しいことを確認してください。

```yaml
---

pd_servers:
  - host: 172.16.5.134
    name: pd-134
  - host: 172.16.5.139
    name: pd-139
  - host: 172.16.5.140
    name: pd-140

tidb_servers:
  - host: 172.16.5.134
  - host: 172.16.5.139
  - host: 172.16.5.140

tikv_servers:
  - host: 172.16.5.134
  - host: 172.16.5.139
  - host: 172.16.5.140

tiflash_servers:
  - host: 172.16.5.141
  - host: 172.16.5.142
  - host: 172.16.5.143

grafana_servers:
  - host: 172.16.5.134

monitoring_servers:
  - host: 172.16.5.134
```

デフォルトでは、TiUPはamd64アーキテクチャで実行されるバイナリファイルとして展開されます。ターゲットマシンがarm64アーキテクチャの場合は、トポロジファイルで設定できます。

```yaml
global:
  arch: "arm64"           # デフォルトですべてのマシンがarm64アーキテクチャのバイナリファイルを使用するように構成します

tidb_servers:
  - host: 172.16.5.134
    arch: "amd64"         # このマシンがamd64アーキテクチャのバイナリファイルを使用するように構成します
  - host: 172.16.5.139
    arch: "arm64"         # このマシンがarm64アーキテクチャのバイナリファイルを使用するように構成します
  - host: 172.16.5.140    # archフィールドが構成されていないマシンは、この場合はデフォルト値であるarm64を使用します。

...
```

ファイルを `/tmp/topology.yaml` として保存します。TiDB v7.4.0を使用し、クラスター名が `prod-cluster` の場合、次のコマンドを実行します。

{{< copyable "shell-regular" >}}

```shell
tiup cluster deploy -p prod-cluster v7.4.0 /tmp/topology.yaml
```

実行中、TiUPは再度トポロジを確認し、対象マシンのルートパスワードを要求します（-pフラグはパスワードの入力を意味します）。

```bash
トポロジを確認してください:
TiDB Cluster: prod-cluster
TiDB Version: v7.4.0
タイプ        ホスト          ポート                            OS/Arch       ディレクトリ
----        ----          -----                            -------       -----------
pd          172.16.5.134  2379/2380                        linux/x86_64  deploy/pd-2379,data/pd-2379
pd          172.16.5.139  2379/2380                        linux/x86_64  deploy/pd-2379,data/pd-2379
pd          172.16.5.140  2379/2380                        linux/x86_64  deploy/pd-2379,data/pd-2379
tikv        172.16.5.134  20160/20180                      linux/x86_64  deploy/tikv-20160,data/tikv-20160
tikv        172.16.5.139  20160/20180                      linux/x86_64  deploy/tikv-20160,data/tikv-20160
tikv        172.16.5.140  20160/20180                      linux/x86_64  deploy/tikv-20160,data/tikv-20160
tidb        172.16.5.134  4000/10080                       linux/x86_64  deploy/tidb-4000
tidb        172.16.5.139  4000/10080                       linux/x86_64  deploy/tidb-4000
tidb        172.16.5.140  4000/10080                       linux/x86_64  deploy/tidb-4000
tiflash     172.16.5.141  9000/8123/3930/20170/20292/8234  linux/x86_64  deploy/tiflash-9000,data/tiflash-9000
tiflash     172.16.5.142  9000/8123/3930/20170/20292/8234  linux/x86_64  deploy/tiflash-9000,data/tiflash-9000
tiflash     172.16.5.143  9000/8123/3930/20170/20292/8234  linux/x86_64  deploy/tiflash-9000,data/tiflash-9000
prometheus  172.16.5.134  9090         deploy/prometheus-9090,data/prometheus-9090
grafana     172.16.5.134  3000         deploy/grafana-3000
注:
    1. 期待するトポロジと異なる場合は、yamlファイルを確認してください。
    2. 同じホストでポート/ディレクトリの競合がないことを確認してください。
続行しますか？ [y/N]:
```

パスワードを入力した後、TiUP clusterは必要なコンポーネントをダウンロードし、それらを対応するマシンにデプロイします。以下のメッセージが表示されたら、デプロイは成功です。

```bash
```

## クラスターリストを表示

クラスターが正常にデプロイされた後は、次のコマンドを実行してクラスターリストを表示します。

{{< copyable "shell-root" >}}

```bash
tiup cluster list
```

```
Starting /root/.tiup/components/cluster/v1.12.3/cluster list
Name          User  Version    Path                                               PrivateKey
----          ----  -------    ----                                               ----------
prod-cluster  tidb  v7.4.0    /root/.tiup/storage/cluster/clusters/prod-cluster  /root/.tiup/storage/cluster/clusters/prod-cluster/ssh/id_rsa
```

## クラスターを起動する

クラスターが正常にデプロイされた後は、次のコマンドを実行してクラスターを起動します。

{{< copyable "shell-regular" >}}

```shell
tiup cluster start prod-cluster
```

もしクラスター名を忘れた場合は、`tiup cluster list`を実行してクラスターリストを表示してください。

TiUPは`systemd`を使用してデーモンプロセスを起動します。プロセスが予期せず終了した場合は、15秒後に再起動されます。

## クラスターステータスを確認する

TiUPは`tiup cluster display`コマンドを提供しており、クラスター内の各コンポーネントのステータスを表示することができます。このコマンドを使用すると、各マシンにログインしてコンポーネントのステータスを確認する必要はありません。このコマンドの使用法は次のとおりです。

{{< copyable "shell-root" >}}

```bash
tiup cluster display prod-cluster
```

```
Starting /root/.tiup/components/cluster/v1.12.3/cluster display prod-cluster
TiDB Cluster: prod-cluster
TiDB Version: v7.4.0
ID                  Role        Host          Ports                            OS/Arch       Status  Data Dir              Deploy Dir
--                  ----        ----          -----                            -------       ------  --------              ----------
172.16.5.134:3000   grafana     172.16.5.134  3000                             linux/x86_64  Up      -                     deploy/grafana-3000
172.16.5.134:2379   pd          172.16.5.134  2379/2380                        linux/x86_64  Up|L    data/pd-2379          deploy/pd-2379
172.16.5.139:2379   pd          172.16.5.139  2379/2380                        linux/x86_64  Up|UI   data/pd-2379          deploy/pd-2379
172.16.5.140:2379   pd          172.16.5.140  2379/2380                        linux/x86_64  Up      data/pd-2379          deploy/pd-2379
172.16.5.134:9090   prometheus  172.16.5.134  9090                             linux/x86_64  Up      data/prometheus-9090  deploy/prometheus-9090
172.16.5.134:4000   tidb        172.16.5.134  4000/10080                       linux/x86_64  Up      -                     deploy/tidb-4000
172.16.5.139:4000   tidb        172.16.5.139  4000/10080                       linux/x86_64  Up      -                     deploy/tidb-4000
172.16.5.140:4000   tidb        172.16.5.140  4000/10080                       linux/x86_64  Up      -                     deploy/tidb-4000
172.16.5.141:9000   tiflash     172.16.5.141  9000/8123/3930/20170/20292/8234  linux/x86_64  Up      data/tiflash-9000     deploy/tiflash-9000
172.16.5.142:9000   tiflash     172.16.5.142  9000/8123/3930/20170/20292/8234  linux/x86_64  Up      data/tiflash-9000     deploy/tiflash-9000
172.16.5.143:9000   tiflash     172.16.5.143  9000/8123/3930/20170/20292/8234  linux/x86_64  Up      data/tiflash-9000     deploy/tiflash-9000
172.16.5.134:20160  tikv        172.16.5.134  20160/20180                      linux/x86_64  Up      data/tikv-20160       deploy/tikv-20160
172.16.5.139:20160  tikv        172.16.5.139  20160/20180                      linux/x86_64  Up      data/tikv-20160       deploy/tikv-20160
172.16.5.140:20160  tikv        172.16.5.140  20160/20180                      linux/x86_64  Up      data/tikv-20160       deploy/tikv-20160
```

`Status`列は、サービスが正常に実行されているかを`Up`または`Down`で示しています。

PDコンポーネントには、`Up`または`Down`に`|L`または`|UI`が付加されることがあります。`|L`はPDノードがリーダーであることを示し、`|UI`は[「TiDB Dashboard」](/dashboard/dashboard-intro.md)がPDノードで実行されていることを示します。

## クラスターをスケーリングイン

> **注意:**
>
> このセクションはスケールインコマンドの構文のみを説明しています。オンラインスケーリングの詳細な手順については[「TiUPを使用してTiDBクラスターをスケールする」](/scale-tidb-using-tiup.md)を参照してください。

クラスターをスケーリングインすると、特定のノードをオフラインにします。この操作は特定のノードをクラスターから削除し、残りのファイルを削除します。

TiKV、TiFlash、およびTiDB Binlogコンポーネントのオフライン処理は非同期であるため（ノードをAPI経由で削除する必要がある）、プロセスには長い時間がかかります（ノードが正常にオフラインになっているかどうかを確認するための継続的な観察が必要です）。そのため、TiKV、TiFlash、およびTiDB Binlogコンポーネントに特別な処理が行われます。

- TiKV、TiFlash、およびBinlogの場合:

    - TiUPクラスターはAPI経由でノードをオフラインにし、プロセスが完了するのを待たずに直ちに終了します。
    - その後、クラスター操作に関連するコマンドが実行されると、TiUPクラスターはTiKV、TiFlash、またはBinlogノードがオフラインになっているかどうかを確認します。オフラインになっていない場合、TiUPクラスターは指定された操作を続行します。オフラインになっている場合、TiUPクラスターは以下の手順を実行します:

        1. オフラインになったノードのサービスを停止します。
        2. ノードに関連するデータファイルをクリアします。
        3. クラスタートポロジからノードを削除します。

- その他のコンポーネントの場合:

    - PDコンポーネントをダウンさせると、TiUPクラスターはAPIを介して指定されたノードをすばやくクラスターから削除し、指定されたPDノードのサービスを停止し、関連するデータファイルを削除します。
    - その他のコンポーネントをダウンさせると、TiUPクラスターはノードサービスを直ちに停止し、関連するデータファイルを削除します。

スケーリングインコマンドの基本的な使用法:

```bash
tiup cluster scale-in <cluster-name> -N <node-id>
```

このコマンドを使用するには、少なくともクラスター名とノードIDを指定する必要があります。ノードIDは、前述のセクションで`tiup cluster display`コマンドを使用して取得できます。

たとえば、`172.16.5.140`のTiKVノードをオフラインにする場合は、次のコマンドを実行します:

{{< copyable "shell-regular" >}}

```bash
tiup cluster scale-in prod-cluster -N 172.16.5.140:20160
```

`tiup cluster display`を実行することで、TiKVノードが`オフライン`であることがわかります:

{{< copyable "shell-root" >}}

```bash
tiup cluster display prod-cluster
```

```
Starting /root/.tiup/components/cluster/v1.12.3/cluster display prod-cluster
TiDB Cluster: prod-cluster
TiDB Version: v7.4.0
ID                  Role        Host          Ports                            OS/Arch       Status   Data Dir              Deploy Dir
--                  ----        ----          -----                            -------       ------   --------              ----------
172.16.5.134:3000   grafana     172.16.5.134  3000                             linux/x86_64  Up       -                     deploy/grafana-3000
172.16.5.134:2379   pd          172.16.5.134  2379/2380                        linux/x86_64  Up|L     data/pd-2379          deploy/pd-2379
172.16.5.139:2379   pd          172.16.5.139  2379/2380                        linux/x86_64  Up|UI    data/pd-2379          deploy/pd-2379
```
172.16.5.140:2379 pd 172.16.5.140 2379/2380 linux/x86_64 Up data/pd-2379 deploy/pd-2379
172.16.5.134:9090 prometheus 172.16.5.134 9090 linux/x86_64 Up data/prometheus-9090 deploy/prometheus-9090
172.16.5.134:4000 tidb 172.16.5.134 4000/10080 linux/x86_64 Up - deploy/tidb-4000
172.16.5.139:4000 tidb 172.16.5.139 4000/10080 linux/x86_64 Up - deploy/tidb-4000
172.16.5.140:4000 tidb 172.16.5.140 4000/10080 linux/x86_64 Up - deploy/tidb-4000
172.16.5.141:9000 tiflash 172.16.5.141 9000/8123/3930/20170/20292/8234 linux/x86_64 Up data/tiflash-9000 deploy/tiflash-9000
172.16.5.142:9000 tiflash 172.16.5.142 9000/8123/3930/20170/20292/8234 linux/x86_64 Up data/tiflash-9000 deploy/tiflash-9000
172.16.5.143:9000 tiflash 172.16.5.143 9000/8123/3930/20170/20292/8234 linux/x86_64 Up data/tiflash-9000 deploy/tiflash-9000
172.16.5.134:20160 tikv 172.16.5.134 20160/20180 linux/x86_64 Up data/tikv-20160 deploy/tikv-20160
172.16.5.139:20160 tikv 172.16.5.139 20160/20180 linux/x86_64 Up data/tikv-20160 deploy/tikv-20160
172.16.5.140:20160 tikv 172.16.5.140 20160/20180 linux/x86_64 Offline data/tikv-20160 deploy/tikv-20160

## クラスターのスケールアウト

> **注：**
>
> このセクションでは、スケールアウトコマンドの構文のみを説明しています。オンラインスケーリングの詳細な手順については、[TiUPを使用してTiDBクラスターをスケーリング](/scale-tidb-using-tiup.md)を参照してください。

スケールアウト操作は、デプロイと似た内部ロジックを持っています：まず、TiUPクラスターコンポーネントはノードのSSH接続を確実にし、ターゲットノードに必要なディレクトリを作成し、次にデプロイ操作を実行し、ノードサービスを起動します。

PDをスケールアウトすると、ノードは`join`によってクラスターに追加され、PDに関連するサービスの構成が更新されます。その他のサービスをスケールアウトすると、サービスは直接開始され、クラスターに追加されます。

すべてのサービスは、スケールアウト時に正当性の検証を行います。検証結果によってスケールアウトが成功したかどうかが表示されます。

`tidb-test`クラスターにTiKVノードとPDノードを追加するには、次の手順を実行します：

1. `scale.yaml`ファイルを作成し、新しいTiKVとPDノードのIPを追加します：

    > **注：**
    >
    > 新しいノードの説明を含むトポロジファイルを作成する必要があります。既存のノードの説明は含めないでください。

    ```yaml
    ---

    pd_servers:
      - host: 172.16.5.140

    tikv_servers:
      - host: 172.16.5.140
    ```

2. スケールアウト操作を実行します。TiUPクラスターは`scale.yaml`に記述されているポート、ディレクトリ、およびその他の情報に従って、対応するノードをクラスターに追加します。

    {{< copyable "shell-regular" >}}

    ```shell
    tiup cluster scale-out tidb-test scale.yaml
    ```

    コマンドを実行した後、`tiup cluster display tidb-test`を実行して、スケールアウトされたクラスターの状態を確認できます。

## ローリングアップグレード

> **注：**
>
> このセクションでは、アップグレードコマンドの構文のみを説明しています。オンラインアップグレードの詳細手順については、[TiUPを使用してTiDBをアップグレード](/upgrade-tidb-using-tiup.md)を参照してください。

ローリングアップグレード機能は、TiDBの分散機能を活用しています。アプリケーションに可能な限り透明なアップグレードプロセスを提供し、ビジネスに影響を与えません。

アップグレード前に、TiUPクラスターは各コンポーネントの設定ファイルが合理的かどうかをチェックします。合理的な場合、コンポーネントはノードごとにアップグレードされます。そうでない場合は、TiUPがエラーを報告して終了します。ノードごとの操作は異なります。

### 異なるノードのための操作

- PDノードをアップグレードする

    - まず、Leaderでないノードをアップグレードします。
    - すべてのLeaderでないノードがアップグレードされた後、Leaderノードをアップグレードします。
        - アップグレードツールは、Leaderをすでにアップグレード済みのノードに移行するようにPDにコマンドを送信します。
        - リーダーの役割が別のノードに切り替わった後、以前のリーダーノードをアップグレードします。
    - アップグレード中に、不健全なノードが検出された場合、ツールはこのアップグレード操作を停止して終了します。原因を手動で分析し、問題を修正してアップグレードを再実行する必要があります。

- TiKVノードをアップグレードする

    - まず、このTiKVノードのRegion Leaderを移行するためにPDにスケジューリング操作を追加します。これにより、アップグレードプロセスがビジネスに影響を与えないようにします。
    - リーダーが移行した後、このTiKVノードをアップグレードします。
    - アップグレードされたTiKVが正常に開始された後、リーダーのスケジューリングを削除します。

- その他のサービスをアップグレードする

    - サービスを通常停止し、ノードを更新します。

### アップグレードコマンド

アップグレードコマンドのフラグは以下の通りです：

```bash
使用法：
  cluster upgrade <cluster-name> <version> [flags]

フラグ：
      --force                  強制アップグレードはリーダーを転送しません
  -h, --help                   アップグレードのヘルプ
      --transfer-timeout int   PDおよびTiKVストアリーダーを転送する際のタイムアウト（デフォルト 600秒）

グローバルフラグ：
      --ssh string          （実験的）実行者のタイプです。オプションの値は「builtin」、「system」、「none」です。
      --wait-timeout int  操作の待機タイムアウト
      --ssh-timeout int   ホストにSSH経由で接続するまでのタイムアウト（SSH接続が必要ない操作の場合は無視されます）（デフォルト 5秒）
  -y, --yes               すべての確認をスキップして「はい」とみなします
```

例えば、以下のコマンドでクラスターをv7.4.0にアップグレードします：

{{< copyable "shell-regular" >}}

```bash
tiup cluster upgrade tidb-test v7.4.0
```

## 設定の更新

コンポーネントの設定を動的に更新する場合、TiUPクラスターコンポーネントは各クラスターの現在の設定を保存します。この設定を編集するには、`tiup cluster edit-config <cluster-name>`コマンドを実行します。例：

{{< copyable "shell-regular" >}}

```bash
tiup cluster edit-config prod-cluster
```

TiUPクラスターはviエディタで設定ファイルを開きます。他のエディタを使用する場合は、`EDITOR`環境変数を使用してエディタをカスタマイズできます。例えば、`export EDITOR=nano`です。

ファイルを編集した後は、変更内容を保存します。新しい設定をクラスターに適用するには、次のコマンドを実行します：

{{< copyable "shell-regular" >}}

```bash
tiup cluster reload prod-cluster
```

このコマンドは、設定を対象のマシンに送信し、設定を適用するためにクラスターを再起動します。

> **注：**
>
> モニタリングコンポーネントについては、対応するインスタンスにカスタム構成パスを追加するために、`tiup cluster edit-config`コマンドを実行してカスタム構成をカスタマイズしてください。例：

```yaml
---

grafana_servers:
  - host: 172.16.5.134
    dashboard_dir: /path/to/local/dashboards/dir

monitoring_servers:
  - host: 172.16.5.134
    rule_dir: /path/to/local/rules/dir

alertmanager_servers:
  - host: 172.16.5.134
    config_file: /path/to/local/alertmanager.yml
```

指定されたパスのファイルの内容と形式の要件は次のようになります：

- `grafana_servers`の`dashboard_dir`フィールドで指定されたフォルダには、完全な`*.json`ファイルを含める必要があります。
- `monitoring_servers`の`rule_dir`フィールドで指定されたフォルダには、完全な`*.rules.yml`ファイルを含める必要があります。
- `alertmanager_servers`の`config_file`フィールドで指定されたファイルの形式については、[Alertmanager構成テンプレート](https://github.com/pingcap/tiup/blob/master/embed/templates/config/alertmanager.yml)を参照してください。
```
`tiup reload` を実行すると、TiUP はまずターゲットマシンの古い設定ファイルをすべて削除し、それから制御マシンの対応する設定をターゲットマシンの対応する設定ディレクトリにアップロードします。したがって、特定の設定ファイルを変更する場合は、（変更されていないものも含めて）すべての設定ファイルが同じディレクトリにあることを確認してください。たとえば、Grafana の `tidb.json` ファイルを変更するには、まず Grafana の `dashboards` ディレクトリからすべての `*.json` ファイルをローカルディレクトリにコピーする必要があります。そうしないと、ターゲットマシンからその他の JSON ファイルが欠落します。

> **注意:**
>
> `grafana_servers` の `dashboard_dir` フィールドを構成した場合、クラスタをリネームするために `tiup cluster rename` コマンドを実行した後、以下の操作を完了する必要があります:
>
> 1. ローカルの `dashboards` ディレクトリで、クラスタ名を新しいクラスタ名に変更します。
> 2. ローカルの `dashboards` ディレクトリで、`datasource` を新しいクラスタ名に変更します。なぜなら `datasource` はクラスタ名によって名前が付けられているからです。
> 3. `tiup cluster reload -R grafana` コマンドを実行します。

## コンポーネントの更新

通常のアップグレードの場合は、`upgrade` コマンドを使用できます。しかし、デバッグなどのシナリオでは、現在実行中のコンポーネントを一時的なパッケージで置き換える必要があることがあります。それには、`patch` コマンドを使用します:

{{< copyable "shell-root" >}}

```bash
tiup cluster patch --help
```

```
指定したパッケージでリモートパッケージを置き換え、サービスを再起動する

使用法:
  cluster patch <クラスタ名> <パッケージのパス> [フラグ]

フラグ:
  -h, --help                    patch のヘルプ
  -N, --node strings            ノードを指定
      --offline                 停止中のクラスタに patch を適用
      --overwrite               このパッケージを将来のスケールアウト操作で使用
  -R, --role strings            ロールを指定
      --transfer-timeout uint   PD および TiKV ストアリーダーを転送する場合のタイムアウト秒数、TiCDC drain のキャプチャにも適用 (デフォルト 600)

グローバルフラグ:
  -c, --concurrency int     許可される並列タスクの最大数 (デフォルト 5)
      --format string       (実験的) 出力形式、使用可能な値は [default、json] (デフォルト "default")
      --ssh string          (実験的) 実行モジュールタイプ: 'builtin'、'system'、'none'。
      --ssh-timeout uint    ホストへのSSH接続のタイムアウト秒数、SSH接続を必要としない操作には無視されます (デフォルト 5)
      --wait-timeout uint   操作が完了するのを待機する秒数のタイムアウト、フィットしない操作には無視されます (デフォルト 120)
  -y, --yes                 すべての確認をスキップし、'yes'を仮定する
```

TiDB ホットフィックスパッケージが `/tmp/tidb-hotfix.tar.gz` にある場合、クラスタ内のすべての TiDB パッケージを置き換えたい場合は、以下のコマンドを実行してください:

{{< copyable "shell-regular" >}}

```bash
tiup cluster patch test-cluster /tmp/tidb-hotfix.tar.gz -R tidb
```

クラスタ内の TiDB パッケージを 1 つだけ置き換えたい場合は、以下のコマンドを実行してください:

{{< copyable "shell-regular" >}}

```bash
tiup cluster patch test-cluster /tmp/tidb-hotfix.tar.gz -N 172.16.4.5:4000
```

## TiDB Ansible クラスタのインポート

> **注意:**
>
> 現在、TiUP クラスタは TiSpark をまだ**実験的**にサポートしています。TiSpark を有効にしている TiDB クラスタをインポートすることはサポートされていません。

TiUP がリリースされる前は、TiDB Ansible がしばしば使用されて TiDB クラスタを展開していました。TiDB Ansible で展開されたクラスタを TiUP で引き継ぐためには、`import` コマンドを使用します。

`import` コマンドの使用法は次のとおりです:

{{< copyable "shell-root" >}}

```bash
tiup cluster import --help
```

```
TiDB-Ansible から既存の TiDB クラスタをインポートする

使用法:
  cluster import [フラグ]

フラグ:
  -d, --dir string         TiDB-Aansible ディレクトリへのパス
  -h, --help               import のヘルプ
      --inventory string   インベントリファイルの名前 (デフォルト "inventory.ini")
      --no-backup          ansible ディレクトリのバックアップを行わない、複数のインベントリファイルがある場合に便利
  -r, --rename NAME        インポートされたクラスタを NAME にリネームする

グローバルフラグ:
      --ssh string        (実験的) 実行モジュールの種類。オプション値は 'builtin'、'system'、'none'。
      --wait-timeout int  操作の待機時間
      --ssh-timeout int   ホストへのSSH接続のタイムアウト秒数、SSH接続を必要としない操作には無視されます (デフォルト 5)
  -y, --yes               すべての確認をスキップし、'yes'を仮定する
```

次のコマンドのいずれかを使用して、TiDB Ansible クラスタをインポートできます:

{{< copyable "shell-regular" >}}

```bash
cd tidb-ansible
tiup cluster import
```

{{< copyable "shell-regular" >}}

```bash
tiup cluster import --dir=/path/to/tidb-ansible
```

## 操作ログの表示

操作ログを表示するには、`audit` コマンドを使用します。`audit` コマンドの使用法は次のとおりです:

```bash
Usage:
  tiup cluster audit [audit-id] [フラグ]

フラグ:
  -h, --help   audit のヘルプ
```

`[audit-id]` フラグが指定されていない場合、コマンドは実行されたコマンドの一覧を表示します。たとえば:

{{< copyable "shell-regular" >}}

```bash
tiup cluster audit
```

```
Starting component `cluster`: /home/tidb/.tiup/components/cluster/v1.12.3/cluster audit
ID      Time                       Command
--      ----                       -------
4BLhr0  2023-10-12T23:55:09+08:00  /home/tidb/.tiup/components/cluster/v1.12.3/cluster deploy test v7.4.0 /tmp/topology.yaml
4BKWjF  2023-10-12T23:36:57+08:00  /home/tidb/.tiup/components/cluster/v1.12.3/cluster deploy test v7.4.0 /tmp/topology.yaml
4BKVwH  2023-10-12T23:02:08+08:00  /home/tidb/.tiup/components/cluster/v1.12.3/cluster deploy test v7.4.0 /tmp/topology.yaml
4BKKH1  2023-10-12T16:39:04+08:00  /home/tidb/.tiup/components/cluster/v1.12.3/cluster destroy test
4BKKDx  2023-10-12T16:36:57+08:00  /home/tidb/.tiup/components/cluster/v1.12.3/cluster deploy test v7.4.0 /tmp/topology.yaml
```

最初の列は `audit-id` です。特定のコマンドの実行ログを表示するには、コマンドの `audit-id` をフラグとして渡します:

{{< copyable "shell-regular" >}}

```bash
tiup cluster audit 4BLhr0
```

## TiDB クラスタのホストでコマンドを実行

TiDB クラスタのホストでコマンドを実行するには、`exec` コマンドを使用します。`exec` コマンドの使用法は次のとおりです:

```bash
Usage:
  cluster exec <クラスタ名> [フラグ]

フラグ:
      --command string   クラスタホストで実行するコマンド (デフォルト "ls")
  -h, --help             exec のヘルプ
  -N, --node strings     指定されたノードでのみ実行
  -R, --role strings     指定されたロールでのみ実行
      --sudo             root 権限を使用 (デフォルト false)

グローバルフラグ:
      --ssh-timeout int   ホストへのSSH接続のタイムアウト秒数、SSH接続を必要としない操作には無視されます (デフォルト 5)
  -y, --yes               すべての確認をスキップし、'yes'を仮定する
```

たとえば、すべての TiDB ノードで `ls /tmp` を実行するには、次のようなコマンドを実行してください:

{{< copyable "shell-regular" >}}

```bash
tiup cluster exec test-cluster --command='ls /tmp'
```

## クラスタコントローラー

TiUP がリリースされる前は、`tidb-ctl`、`tikv-ctl`、`pd-ctl` などのツールを使用してクラスタを制御できました。これらのツールをダウンロードして使用しやすくするために、TiUP は全てのツールを 1 つのコンポーネント、`ctl` に統合しています。

```bash
Usage:
  tiup ctl:v<クラスタバージョン> {tidb/pd/tikv/binlog/etcd} [フラグ]

フラグ:
  -h, --help   tiup のヘルプ
```

このコマンドは以前のツールと対応しています:

```bash
tidb-ctl [args] = tiup ctl tidb [args]
```
pd-ctl [args] = tiup ctl pd [args]
tikv-ctl [args] = tiup ctl tikv [args]
binlogctl [args] = tiup ctl bindlog [args]
etcdctl [args] = tiup ctl etcd [args]
```

たとえば、以前に`pd-ctl -u http://127.0.0.1:2379 store` を実行してストアを表示した場合、次のコマンドをTiUPで実行できます:

{{< copyable "shell-regular" >}}

```bash
tiup ctl:v<CLUSTER_VERSION> pd -u http://127.0.0.1:2379 store
```

## ターゲットマシンの環境チェック

`check`コマンドを使用して、ターゲットマシンの環境で一連のチェックを実行し、チェック結果を出力することができます。 `check`コマンドを実行することで、一般的な不合理な構成やサポートされていない状況を見つけることができます。 コマンドフラグのリストは以下のとおりです:

```bash
使い方:
  tiup cluster check <topology.yml | cluster-name> [flags]
フラグ:
      --apply                  失敗したチェックを修正しようとします
      --cluster                既存のクラスタをチェックします。入力はクラスタ名です。
      --enable-cpu             CPUスレッド数チェックを有効にする
      --enable-disk            ディスクIO（fio）チェックを有効にする
      --enable-mem             メモリサイズチェックを有効にする
  -h, --help                   ヘルプを表示します
  -i, --identity_file string   SSHの秘密鍵ファイルのパス。指定した場合、公開キー認証が使用されます
  -p, --password               ターゲットホストのパスワードを使用します。指定した場合、パスワード認証が使用されます
      --user string            SSH経由でログインするユーザー名。ユーザーはroot（またはsudo）権限を持っている必要があります。
```

デフォルトでは、このコマンドはデプロイ前に環境を確認するために使用されます。 `--クラスタ`フラグを指定してモードを切り替えることで、既存のクラスタのターゲットマシンのチェックも行うことができます。たとえば:

```bash
# デプロイ前に展開されたサーバーのチェック
tiup cluster check topology.yml --user tidb -p
# 既存のクラスタの展開されたサーバーのチェック
tiup cluster check <cluster-name> --cluster
```

CPUスレッド数のチェック、メモリサイズのチェック、およびディスクパフォーマンスのチェックはデフォルトで無効になっています。本番環境においては、これらのチェックを有効にし、すべてのチェック項目をパスするようにして最高のパフォーマンスを得ることを推奨します。

- CPU: スレッド数が16以上の場合、チェックはパスします。
- メモリ: 物理メモリの総サイズが32 GB以上の場合、チェックはパスします。
- ディスク: `data_dir`のパーティションで`fio`テストを実行し、結果を記録します。

チェックを実行する際に`--apply`フラグが指定されている場合、プログラムは自動的に失敗した項目を修復します。自動修復は、構成やシステムパラメータを変更して調整できる項目に限定されます。修復されていない他の項目は、実際の状況に応じて手動で処理する必要があります。

環境チェックはクラスタの展開に必要とされるものではありません。本番環境においては、デプロイ前に環境チェックを実施し、すべてのチェック項目をパスすることを推奨します。すべてのチェック項目をパスしない場合、クラスタはデプロイされ、通常通り実行されるかもしれませんが、最高のパフォーマンスが得られない可能性があります。

## システムのネイティブSSHクライアントを使用してクラスタに接続する

クラスタマシン上で実行されるすべての操作は、TiUPに組み込まれたSSHクライアントを使用してクラスタに接続し、コマンドを実行します。ただし、場合によっては、クラスタ操作を実行するためにコントロールマシンシステムのネイティブSSHクライアントを使用する必要があることがあります。たとえば:

- SSHプラグインを認証に使用する
- カスタマイズされたSSHクライアントを使用する

その場合は、`--ssh=system`コマンドラインフラグを使用してシステムネイティブのコマンドラインツールを有効にすることができます:

- クラスタを展開: `tiup cluster deploy <cluster-name> <version> <topo> --ssh=system`。`<cluster-name>`にクラスタの名前、`<version>`に展開するTiDBのバージョン（たとえば`v7.4.0`など）、`<topo>`にトポロジファイルを記入してください。
- クラスタを開始: `tiup cluster start <cluster-name> --ssh=system`
- クラスタをアップグレード: `tiup cluster upgrade ... --ssh=system`

上記のすべてのクラスタ操作コマンドに`--ssh=system`を追加してシステムのネイティブSSHクライアントを使用することができます。

すべてのコマンドにこのフラグを追加するのを避けるために、システム変数`TIUP_NATIVE_SSH`を使用してローカルSSHクライアントを使用するかどうかを指定することができます:

```shell
export TIUP_NATIVE_SSH=true
# または
export TIUP_NATIVE_SSH=1
# または
export TIUP_NATIVE_SSH=enable
```

この環境変数を指定し、同時に`--ssh`を指定する場合、`--ssh`の方が優先されます。

> **注意:**
>
> クラスタのデプロイプロセス中に接続にパスワード（`-p`）を使用する必要がある場合、またはキーファイルに`passphrase`が設定されている場合は、制御マシンに`sshpass`がインストールされていることを確認する必要があります。それ以外の場合、タイムアウトエラーが報告されます。

## 制御マシンの移行およびTiUPデータのバックアップ

TiUPデータはユーザーのホームディレクトリ内の`.tiup`ディレクトリに保存されています。制御マシンを移行するには、次の手順に従って`.tiup`ディレクトリを対応するターゲットマシンにコピーできます:

1. オリジナルのマシンのホームディレクトリで`tar czvf tiup.tar.gz .tiup`を実行します。
2. `tiup.tar.gz`を対象マシンのホームディレクトリにコピーします。
3. 対象マシンのホームディレクトリで`tar xzvf tiup.tar.gz`を実行します。
4. `.tiup`ディレクトリを`PATH`環境変数に追加します。

    `bash`を使用し、`tidb`ユーザーである場合、`~/.bashrc`に`export PATH=/home/tidb/.tiup/bin:$PATH`を追加し、`source ~/.bashrc`を実行します。その後、使用するシェルとユーザーに応じて対応する調整を行います。

> **注意:**
>
> 異常な状況（たとえば、制御マシンのディスク障害など）によってTiUPデータが失われることを避けるために、`.tiup`ディレクトリを定期的にバックアップすることをお勧めします。

## クラスタデプロイメントおよびO＆Mのメタファイルのバックアップと復元

操作およびメンテナンス（O＆M）のために使用されるメタファイルが失われた場合、TiUPを使用してクラスタを管理することは失敗します。メタファイルを定期的にバックアップすることをお勧めします。次のコマンドを実行してバックアップできます:

```bash
tiup cluster meta backup ${cluster_name}
```

メタファイルが失われた場合は、次のコマンドを実行してそれらを復元できます:

```bash
tiup cluster meta restore ${cluster_name} ${backup_file}
```

> **注意:**
>
> 復元操作は現在のメタファイルを上書きします。そのため、失われた場合に限り、メタファイルを復元することをお勧めします。