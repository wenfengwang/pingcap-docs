---
title: TiUPを使用してTiDBクラスターのスケーリング
summary: TiUPを使用してTiDBクラスターをスケールアップする方法について学びます。
aliases: ['/docs/dev/scale-tidb-using-tiup/', '/docs/dev/how-to/scale/with-tiup/']
---

# TiUPを使用してTiDBクラスターのスケーリング

TiDBクラスターの容量を増減させることができ、オンラインサービスを中断することなく行えます。

このドキュメントでは、TiUPを使用してTiDB、TiKV、PD、TiCDC、またはTiFlashクラスターのスケーリング方法について説明します。TiUPをインストールしていない場合は、[Step 2. コントロールマシンにTiUPをデプロイする手順](/production-deployment-using-tiup.md#step-2-deploy-tiup-on-the-control-machine)を参照してください。

現在のクラスター名のリストを表示するには、`tiup cluster list`を実行します。

たとえば、クラスターの元のトポロジーが次のような場合:

| ホストIP | サービス |
|:---|:----|
| 10.0.1.3 | TiDB + TiFlash |
| 10.0.1.4 | TiDB + PD |
| 10.0.1.5 | TiKV + モニタ |
| 10.0.1.1 | TiKV |
| 10.0.1.2 | TiKV |

## TiDB/PD/TiKVクラスターのスケールアウト

このセクションでは、`10.0.1.5`ホストにTiDBノードを追加する手順について説明します。

> **注意:**
>
> PDノードを追加する手順も同様です。TiKVノードを追加する前に、クラスターロードに応じてPDスケジューリングパラメータを事前に調整することをお勧めします。

1. スケールアウトのトポロジを構成します:

    > **注意:**
    >
    > * ポートとディレクトリ情報はデフォルトで必要ありません。
    > * 単一マシンに複数のインスタンスがデプロイされている場合は、それぞれに異なるポートとディレクトリを割り当てる必要があります。ポートやディレクトリに競合がある場合は、デプロイ時またはスケーリング時に通知が表示されます。
    > * TiUP v1.0.0以降、スケールアウト構成は、元のクラスターのグローバル構成を継承します。

    `scale-out.yml`ファイルにスケールアウトトポロジ構成を追加します:

    {{< copyable "shell-regular" >}}

    ```shell
    vi scale-out.yml
    ```

    {{< copyable "" >}}

    ```ini
    tidb_servers:
    - host: 10.0.1.5
      ssh_port: 22
      port: 4000
      status_port: 10080
      deploy_dir: /tidb-deploy/tidb-4000
      log_dir: /tidb-deploy/tidb-4000/log
    ```

    以下はTiKV構成ファイルのテンプレートです:

    {{< copyable "" >}}

    ```ini
    tikv_servers:
    - host: 10.0.1.5
      ssh_port: 22
      port: 20160
      status_port: 20180
      deploy_dir: /tidb-deploy/tikv-20160
      data_dir: /tidb-data/tikv-20160
      log_dir: /tidb-deploy/tikv-20160/log
    ```

    以下はPD構成ファイルのテンプレートです:

    {{< copyable "" >}}

    ```ini
    pd_servers:
    - host: 10.0.1.5
      ssh_port: 22
      name: pd-1
      client_port: 2379
      peer_port: 2380
      deploy_dir: /tidb-deploy/pd-2379
      data_dir: /tidb-data/pd-2379
      log_dir: /tidb-deploy/pd-2379/log
    ```

    現在のクラスターの構成を表示するには、`tiup cluster edit-config <cluster-name>`を実行します。`scale-out.yml`に`global`と`server_configs`のパラメータ構成が継承され、`scale-out.yml`にも適用されます。

2. スケールアウトコマンドを実行します:

    `scale-out`コマンドを実行する前に、`check`コマンドおよび`check --apply`コマンドを使用してクラスターの潜在的なリスクを検出し、自動的に修復します:

    1. 潜在的なリスクのチェック:

        {{< copyable "shell-regular" >}}

        ```shell
        tiup cluster check <cluster-name> scale-out.yml --cluster --user root [-p] [-i /home/root/.ssh/gcp_rsa]
        ```

    2. 自動修復の有効化:

        {{< copyable "shell-regular" >}}

        ```shell
        tiup cluster check <cluster-name> scale-out.yml --cluster --apply --user root [-p] [-i /home/root/.ssh/gcp_rsa]
        ```

    3. `scale-out`コマンドを実行します:

        {{< copyable "shell-regular" >}}

        ```shell
        tiup cluster scale-out <cluster-name> scale-out.yml [-p] [-i /home/root/.ssh/gcp_rsa]
        ```

    上記のコマンドで:

    - `scale-out.yml`はスケールアウト構成ファイルです。
    - `--user root`は、クラスターのスケールアウトを完了するために、`root`ユーザーとしてターゲットマシンにログインすることを示します。`root`ユーザーには、ターゲットマシンへの`ssh`および`sudo`特権が必要です。代わりに`ssh`および`sudo`特権を持つ他のユーザーを使用してデプロイを完了することもできます。
    - `[-i]`および`[-p]`はオプションです。パスワードなしでターゲットマシンにログインするように構成した場合、これらのパラメータは必要ありません。そうでない場合は、2つのパラメータのうち1つを選択します。`[-i]`は、ターゲットマシンにアクセスできる`root`ユーザー（または`--user`で指定された他のユーザー）の秘密鍵です。`[-p]`はユーザーパスワードを対話的に入力するために使用します。

    `Scaled cluster <cluster-name> out successfully`と表示されたら、スケールアウト操作が成功です。

3. クラスターの状態を確認します:

    {{< copyable "shell-regular" >}}

    ```shell
    tiup cluster display <cluster-name>
    ```

    ブラウザを使用して<http://10.0.1.5:3000>にアクセスし、クラスターと新しいノードの状態を監視します。

スケールアウト後、クラスタートポロジは次のようになります:

| ホストIP   | サービス   |
|:----|:----|
| 10.0.1.3   | TiDB + TiFlash   |
| 10.0.1.4   | TiDB + PD   |
| 10.0.1.5   | **TiDB** + TiKV + Monitor   |
| 10.0.1.1   | TiKV    |
| 10.0.1.2   | TiKV    |

## TiFlashクラスターのスケールアウト

このセクションでは、`10.0.1.4`ホストにTiFlashノードを追加する手順について説明します。

> **注意:**
>
> 既存のTiDBクラスターにTiFlashノードを追加する際には、以下の点に注意してください:
>
> - 現在のTiDBバージョンがTiFlashを使用することをサポートしていることを確認してください。それ以外の場合、TiDBクラスターをv5.0またはそれ以降のバージョンにアップグレードしてください。
> - `tiup ctl:v<CLUSTER_VERSION> pd -u http://<pd_ip>:<pd_port> config set enable-placement-rules true`コマンドを実行して、Placement Rules機能を有効にしてください。または、[pd-ctl](/pd-control.md)で対応するコマンドを実行してください。

1. `scale-out.yml`ファイルにノード情報を追加します:

    `scale-out.yml`ファイルを作成して、TiFlashノード情報を追加します。

    {{< copyable "" >}}

    ```ini
    tiflash_servers:
    - host: 10.0.1.4
    ```

    現在はIPアドレスのみを追加できます。ドメイン名は追加できません。

2. スケールアウトコマンドを実行します:

    {{< copyable "shell-regular" >}}

    ```shell
    tiup cluster scale-out <cluster-name> scale-out.yml
    ```

    > **注意:**
    >
    > 上記のコマンドは、仮定として、コマンドを実行するユーザーと新しいマシンに対する相互信頼が構成されているものとします。相互信頼が構成できない場合は、`-p`オプションを使用して新しいマシンのパスワードを入力するか、`-i`オプションを使用して秘密鍵ファイルを指定してください。

3. クラスターの状態を表示します:

    {{< copyable "shell-regular" >}}

    ```shell
    tiup cluster display <cluster-name>
    ```

    ブラウザを使用して<http://10.0.1.5:3000>にアクセスし、クラスターと新しいノードの状態を確認します。

スケールアウト後、クラスタートポロジは次のようになります:

| ホストIP   | サービス   |
|:----|:----|
| 10.0.1.3   | TiDB + TiFlash   |
| 10.0.1.4   | TiDB + PD + **TiFlash**    |
| 10.0.1.5   | TiDB + TiKV + Monitor   |
| 10.0.1.1   | TiKV    |
| 10.0.1.2   | TiKV    |
## TiCDC クラスタの拡張

このセクションでは、`10.0.1.3` および `10.0.1.4` ホストに 2 つの TiCDC ノードを追加する手順を示します。

1. `scale-out.yml` ファイルにノード情報を追加します:

    `scale-out.yml` ファイルを作成し、TiCDC ノード情報を追加します。

    ```ini
    cdc_servers:
      - host: 10.0.1.3
        gc-ttl: 86400
        data_dir: /tidb-data/cdc-8300
      - host: 10.0.1.4
        gc-ttl: 86400
        data_dir: /tidb-data/cdc-8300
    ```

2. 拡張コマンドを実行します:

    ```shell
    tiup cluster scale-out <cluster-name> scale-out.yml
    ```

    > **注記:**
    >
    > 上記のコマンドは、コマンドを実行するユーザーと新しいマシンの間で相互信頼が設定されていることを前提としています。相互信頼が設定できない場合は、`-p` オプションを使用して新しいマシンのパスワードを入力するか、`-i` オプションを使用してプライベートキーファイルを指定してください。

3. クラスタの状態を表示します:

    ```shell
    tiup cluster display <cluster-name>
    ```

    ブラウザを使用して <http://10.0.1.5:3000> にアクセスし、クラスタおよび新しいノードの状態を表示します。

拡張後、クラスタのトポロジは以下のようになります:

| ホストIP   | サービス   |
|:----|:----|
| 10.0.1.3   | TiDB + TiFlash + **TiCDC**  |
| 10.0.1.4   | TiDB + PD + TiFlash + **TiCDC**  |
| 10.0.1.5   | TiDB+ TiKV + モニター   |
| 10.0.1.1   | TiKV    |
| 10.0.1.2   | TiKV    |

## TiDB/PD/TiKV クラスタの縮小

このセクションでは、`10.0.1.5` ホストから TiKV ノードを削除する手順を示します。

> **注記:**
>
> - TiDB や PD ノードを削除する手順も同様です。
> - TiKV、TiFlash、TiDB バイナリログのコンポーネントは非同期でオフラインになり、停止プロセスには時間がかかるため、TiUP は異なる方法でそれらをオフラインにします。詳細については、[各コンポーネントのオフラインプロセスの取り扱い](/tiup/tiup-component-cluster-scale-in.md#particular-handling-of-components-offline-process) を参照してください。
> - TiKV の PD クライアントは PD ノードの一覧をキャッシュします。TiKV の現在のバージョンには、定期的にPDノードのリストを自動的に更新する仕組みがあり、これによりTiKVがキャッシュしたPDノードの期限切れリストの問題を緩和できます。ただし、PD のスケールアウト後は、スケール前に存在するすべての PD ノードをまとめて削除することを避けるべきです。必要な場合は、以前に存在したすべての PD ノードをオフラインにする前に、PD リーダーを新たに追加された PD ノードにスイッチする必要があります。

1. ノードID情報を表示します:

    ```shell
    tiup cluster display <cluster-name>
    ```

    ```
    Starting /root/.tiup/components/cluster/v1.12.3/cluster display <cluster-name>
    TiDB Cluster: <cluster-name>
    TiDB Version: v7.4.0
    ID              Role         Host        Ports                            Status  Data Dir                Deploy Dir
    --              ----         ----        -----                            ------  --------                ----------
    10.0.1.3:8300   cdc          10.0.1.3    8300                             Up      data/cdc-8300           deploy/cdc-8300
    10.0.1.4:8300   cdc          10.0.1.4    8300                             Up      data/cdc-8300           deploy/cdc-8300
    10.0.1.4:2379   pd           10.0.1.4    2379/2380                        Healthy data/pd-2379            deploy/pd-2379
    10.0.1.1:20160  tikv         10.0.1.1    20160/20180                      Up      data/tikv-20160         deploy/tikv-20160
    10.0.1.2:20160  tikv         10.0.1.2    20160/20180                      Up      data/tikv-20160         deploy/tikv-20160
    10.0.1.5:20160  tikv         10.0.1.5    20160/20180                      Up      data/tikv-20160         deploy/tikv-20160
    10.0.1.3:4000   tidb         10.0.1.3    4000/10080                       Up      -                       deploy/tidb-4000
    10.0.1.4:4000   tidb         10.0.1.4    4000/10080                       Up      -                       deploy/tidb-4000
    10.0.1.5:4000   tidb         10.0.1.5    4000/10080                       Up      -                       deploy/tidb-4000
    10.0.1.3:9000   tiflash      10.0.1.3    9000/8123/3930/20170/20292/8234  Up      data/tiflash-9000       deploy/tiflash-9000
    10.0.1.4:9000   tiflash      10.0.1.4    9000/8123/3930/20170/20292/8234  Up      data/tiflash-9000       deploy/tiflash-9000
    10.0.1.5:9090   prometheus   10.0.1.5    9090                             Up      data/prometheus-9090    deploy/prometheus-9090
    10.0.1.5:3000   grafana      10.0.1.5    3000                             Up      -                       deploy/grafana-3000
    10.0.1.5:9093   alertmanager 10.0.1.5    9093/9294                        Up      data/alertmanager-9093  deploy/alertmanager-9093
    ```

2. 縮小コマンドを実行します:

    ```shell
    tiup cluster scale-in <cluster-name> --node 10.0.1.5:20160
    ```

    `--node` パラメータはオフラインにするノードのIDです。

    `Scaled cluster <cluster-name> in successfully` と表示された場合、縮小操作は成功です。

3. クラスタの状態を確認します:

    縮小プロセスには時間がかかります。次のコマンドを実行して縮小の状態を確認できます:

    ```shell
    tiup cluster display <cluster-name>
    ```

    オフラインにするノードが `Tombstone` になっている場合、縮小操作は成功です。

    ブラウザを使用して <http://10.0.1.5:3000> にアクセスし、クラスタの状態を表示してください。

現在のトポロジは以下のようになります:

| ホストIP   | サービス   |
|:----|:----|
| 10.0.1.3   | TiDB + TiFlash + TiCDC  |
| 10.0.1.4   | TiDB + PD + TiFlash + TiCDC |
| 10.0.1.5   | TiDB + モニター **(TiKV が削除されました)**   |
| 10.0.1.1   | TiKV    |
| 10.0.1.2   | TiKV    |

## TiFlash クラスタの縮小

このセクションでは、`10.0.1.4` ホストから TiFlash ノードを削除する手順を示します。

### 1. 残りの TiFlash ノードの数に応じてテーブルのレプリカ数を調整します

1. スケールイン後の残りの TiFlash ノードの数に応じて、テーブルのレプリカ数を調整する必要があります。`tobe_left_nodes` はスケールイン後の TiFlash ノード数を示します。クエリ結果が空であれば、TiFlash をスケールインできます。クエリ結果が空でない場合は、関連するテーブルの TiFlash レプリカ数を修正する必要があります。

    ```sql
    SELECT * FROM information_schema.tiflash_replica WHERE REPLICA_COUNT >  'tobe_left_nodes';
    ```

2. すべてのテーブルに対して、TiFlashノードがスケールイン後のTiFlashノード数を上回るようなレプリカを持つ場合は、次のステートメントを実行します。 `new_replica_num` は `tobe_left_nodes` 以下である必要があります。

    ```sql
    ALTER TABLE <db-name>.<table-name> SET tiflash replica 'new_replica_num';
    ```

3. ステップ1を再度実行し、TiFlashノードがスケールイン後のTiFlashノード数を上回るテーブルがないことを確認します。

### 2. スケールイン操作を実行する

次のいずれかの方法でスケールイン操作を実行します。

#### Solution 1. TiUPを使用してTiFlashノードを削除する

1. 削除するノードの名前を確認します：

    {{< copyable "shell-regular" >}}

    ```shell
    tiup cluster display <cluster-name>
    ```

2. TiFlashノードを削除します（ステップ1でノード名が `10.0.1.4:9000` であると仮定）：

    {{< copyable "shell-regular" >}}

    ```shell
    tiup cluster scale-in <cluster-name> --node 10.0.1.4:9000
    ```

#### Solution 2. 手動でTiFlashノードを削除する

特別なケース（ノードを強制的に取り除く必要がある場合など）やTiUPスケールイン操作が失敗した場合には、以下の手順でTiFlashノードを手動で削除できます。

1. pd-ctlのstoreコマンドを使用して、このTiFlashノードに対応するストアIDを表示します。

    * [pd-ctl](/pd-control.md) で store コマンドを入力します（バイナリファイルは tidb-ansible ディレクトリの `resources/bin` にあります）。

    * TiUP デプロイを使用する場合は、`pd-ctl` を `tiup ctl:v<CLUSTER_VERSION> pd` に置き換えます。

    {{< copyable "shell-regular" >}}

    ```shell
    tiup ctl:v<CLUSTER_VERSION> pd -u http://<pd_ip>:<pd_port> store
    ```

    > **Note:**
    >
    > クラスター内に複数のPDインスタンスが存在する場合、上記のコマンドでアクティブなPDインスタンスのIPアドレスとポートを指定する必要があります。

2. pd-ctlでTiFlashノードを削除します：

    * pd-ctl にて `store delete <store_id>` を入力します（`<store_id>` は前のステップで見つかったTiFlashノードのストアIDです）。

    * TiUPデプロイを使用する場合は、`pd-ctl` を `tiup ctl:v<CLUSTER_VERSION> pd` に置き換えます。

    {{< copyable "shell-regular" >}}

    ```shell
    tiup ctl:v<CLUSTER_VERSION> pd -u http://<pd_ip>:<pd_port> store delete <store_id>
    ```

    > **Note:**
    >
    > クラスター内に複数のPDインスタンスが存在する場合、上記のコマンドでアクティブなPDインスタンスのIPアドレスとポートを指定する必要があります。

3. TiFlashノードのストアが消えるか、`state_name` が `Tombstone` になるまで、TiFlashプロセスを停止してください。

4. ティグメタデータの場所を確認し、TiFlash構成の `data_dir` ディレクトリの下にあるTiFlashデータファイルを手動で削除します。

5. 次のコマンドを使用して、クラスタートポロジのTiFlashノードを削除します：

    {{< copyable "shell-regular" >}}

    ```shell
    tiup cluster scale-in <cluster-name> --node <pd_ip>:<pd_port> --force
    ```

> **Note:**
>
> クラスター内のすべてのTiFlashノードが実行を停止する前に、TiFlashにレプリケートされたすべてのテーブルがキャンセルされていない場合は、PDで手動でレプリケーションルールをクリーンアップする必要があります。そうしないと、TiFlashノードを正常に取り除くことができません。

PDでレプリケーションルールを手動でクリーンアップする手順は以下の通りです。

1. 現在のPDインスタンスに関連するTiFlashのデータレプリケーションルールをすべて表示します：

    {{< copyable "shell-regular" >}}

    ```shell
    curl http://<pd_ip>:<pd_port>/pd/api/v1/config/rules/group/tiflash
    ```

    ```
    [
      {
        "group_id": "tiflash",
        "id": "table-45-r",
        "override": true,
        "start_key": "7480000000000000FF2D5F720000000000FA",
        "end_key": "7480000000000000FF2E00000000000000F8",
        "role": "learner",
        "count": 1,
        "label_constraints": [
          {
            "key": "engine",
            "op": "in",
            "values": [
              "tiflash"
            ]
          }
        ]
      }
    ]
    ```

2. TiFlashに関連するすべてのデータレプリケーションルールを削除します。`id` が `table-45-r` のルールを例として削除するコマンドは以下の通りです：

    {{< copyable "shell-regular" >}}

    ```shell
    curl -v -X DELETE http://<pd_ip>:<pd_port>/pd/api/v1/config/rule/tiflash/table-45-r
    ```

3. クラスターの状態を表示します：

    {{< copyable "shell-regular" >}}

    ```shell
    tiup cluster display <cluster-name>
    ```

    ブラウザで <http://10.0.1.5:3000> にアクセスし、クラスターの状態と新しいノードを表示します。

スケールイン後のクラスタートポロジは以下の通りです：

| ホストIP   | サービス   |
|:----|:----|
| 10.0.1.3   | TiDB + TiFlash + TiCDC  |
| 10.0.1.4   | TiDB + PD + **（TiFlashが削除されました）**  |
| 10.0.1.5   | TiDB+ Monitor  |
| 10.0.1.1   | TiKV    |
| 10.0.1.2   | TiKV    |

## TiCDCクラスターのスケールイン

このセクションでは、`10.0.1.4` ホストからTiCDCノードを削除する手順を示します。

1. ノードをオフラインにします：

    {{< copyable "shell-regular" >}}

    ```shell
    tiup cluster scale-in <cluster-name> --node 10.0.1.4:8300
    ```

2. クラスターの状態を表示します：

    {{< copyable "shell-regular" >}}

    ```shell
    tiup cluster display <cluster-name>
    ```

    ブラウザで <http://10.0.1.5:3000> にアクセスし、クラスターの状態を表示します。

現在のトポロジは以下の通りです：

| ホストIP   | サービス   |
|:----|:----|
| 10.0.1.3   | TiDB + TiFlash + TiCDC  |
| 10.0.1.4   | TiDB + PD + **（TiCDCが削除されました）**  |
| 10.0.1.5   | TiDB + Monitor  |
| 10.0.1.1   | TiKV    |
| 10.0.1.2   | TiKV    |