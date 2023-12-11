---
title: TiUPを使用してTiDBをアップグレードする
summary: TiUPを使用してTiDBをアップグレードする方法について学びます。
aliases: ['/docs/dev/upgrade-tidb-using-tiup/','/docs/dev/how-to/upgrade/using-tiup/','/tidb/dev/upgrade-tidb-using-tiup-offline','/docs/dev/upgrade-tidb-using-tiup-offline/']
---

# TiUPを使用してTiDBをアップグレードする

このドキュメントは、次のアップグレードパスを対象としています:

- TiDB 4.0 バージョンから TiDB 7.4 へのアップグレード。
- TiDB 5.0-5.4 バージョンから TiDB 7.4 へのアップグレード。
- TiDB 6.0-6.6 から TiDB 7.4 へのアップグレード。
- TiDB 7.0-7.3 から TiDB 7.4 へのアップグレード。

> **警告:**
>
> 1. TiFlashをオンラインで5.3以前から5.3以降にアップグレードすることはできません。代わりに、まず早期バージョンのすべてのTiFlashインスタンスを停止し、その後クラスタをオフラインでアップグレードする必要があります。他のコンポーネント(たとえばTiDBやTiKV)がオンラインアップグレードをサポートしていない場合は、[オンラインアップグレード](#online-upgrade)での警告に従ってください。
> 2. アップグレードプロセス中にDDLステートメントを実行しないでください。さもないと、未定義の動作の問題が発生する可能性があります。
> 3. DDLステートメントがクラスタで実行中の場合は、TiDBクラスタをアップグレードしないでください(通常は`ADD INDEX`や列タイプの変更など、時間のかかるDDLステートメントが実行されている場合です)。アップグレード前には、[`ADMIN SHOW DDL`](/sql-statements/sql-statement-admin-show-ddl.md)コマンドを使用して、TiDBクラスタで実行中のDDLジョブを確認することをお勧めします。クラスタでDDLジョブが実行されている場合は、アップグレードするために、DDLの実行が終了するまで待機するか、[`ADMIN CANCEL DDL`](/sql-statements/sql-statement-admin-cancel-ddl.md)コマンドを使用してDDLジョブをキャンセルしてからクラスタをアップグレードしてください。

> **メモ:**
>
> - アップグレード対象のクラスタがv3.1またはそれ以前のバージョン(v3.0またはv2.1)の場合、直接v7.4.0にアップグレードすることはサポートされていません。まずクラスタをv4.0にアップグレードし、その後v7.4.0にアップグレードする必要があります。
> - アップグレード対象のクラスタがv6.2よりも古い場合、アップグレード中にクラスタがv6.2またはそれ以降のバージョンにアップグレードするときにスタックすることがあります。このようなシナリオでの問題の修正方法については、[問題の修正方法](#how-to-fix-the-issue-that-the-upgrade-gets-stuck-when-upgrading-to-v620-or-later-versions)を参照してください。
> - TiDBノードは、[`server-version`](/tidb-configuration-file.md#server-version)構成項目の値を使用して、現在のTiDBバージョンを検証します。したがって、予期しない動作を避けるために、TiDBクラスタをアップグレードする前に、`server-version`の値を空に設定するか、現在のTiDBクラスタの実際のバージョンに設定する必要があります。

## アップグレードの注意点

- TiDBは現在、バージョンのダウングレードやアップグレード後の以前のバージョンへのロールバックをサポートしていません。
- TiDB Ansibleを使用して管理されているv4.0クラスタの場合は、[Upgrade TiDB Using TiUP(v4.0)](https://docs.pingcap.com/tidb/v4.0/upgrade-tidb-using-tiup#import-tidb-ansible-and-the-inventoryini-configuration-to-tiup)に従い、新しい管理のためにクラスタをTiUP(`tiup cluster`)にインポートする必要があります。その後、このドキュメントに従ってクラスタをv7.4.0にアップグレードできます。
- v3.1よりも古いバージョンのクラスタをv7.4.0にアップグレードする場合:
    1. [TiDB Ansible](https://docs.pingcap.com/tidb/v3.0/upgrade-tidb-using-ansible)を使用してこのバージョンを3.0にアップグレードします。
    2. TiDB Ansible構成をTiUP(`tiup cluster`)にインポートします。
    3. [Upgrade TiDB Using TiUP(v4.0)](https://docs.pingcap.com/tidb/v4.0/upgrade-tidb-using-tiup#import-tidb-ansible-and-the-inventoryini-configuration-to-tiup)に従い、3.0バージョンを4.0に更新します。
    4. その後、このドキュメントに従ってクラスタをv7.4.0にアップグレードします。
- TiDB Binlog、TiCDC、TiFlashなどのコンポーネントのバージョンをアップグレードすることができます。
- v6.3.0よりも古いバージョンのTiFlashをv6.3.0およびそれ以降のバージョンにアップグレードする場合、Linux AMD64アーキテクチャのCPUはAVX2インストラクションセットをサポートし、Linux ARM64アーキテクチャのCPUはARMv8インストラクションセットアーキテクチャをサポートしている必要があります。詳細については、[v6.3.0リリースノート](/releases/release-6.3.0.md#others)の説明を参照してください。
- 異なるバージョンの詳細な互換性変更については、各バージョンの[リリースノート](/releases/release-notes.md)を参照してください。対応するリリースノートの「互換性の変更」セクションに従って、クラスタ構成を変更してください。
- v5.3よりも古いバージョンからv5.3またはそれ以降のバージョンにアップグレードするクラスタでは、デフォルトでデプロイされたPrometheusがv2.8.1からv2.27.1にアップグレードされます。Prometheus v2.27.1ではさらに多くの機能が提供され、セキュリティの問題が修正されています。v2.27.1でのアラートの表現はv2.8.1とは異なります。詳細については、詳細については、[Prometheusのコミット](https://github.com/prometheus/prometheus/commit/7646cbca328278585be15fa615e22f2a50b47d06)を参照してください。

## 準備

このセクションでは、TiDBクラスタをアップグレードする前に必要な準備作業について説明します。これには、TiUPおよびTiUP Clusterコンポーネントのアップグレードが含まれます。

### ステップ1: 互換性の変更を確認する

TiDB v7.4.0のリリースノートにある[互換性の変更](/releases/release-7.4.0.md#compatibility-changes)を確認してください。アップグレードに影響を与える変更がある場合は、適切な対応を行ってください。

### ステップ2: TiUPまたはTiUPオフラインミラーをアップグレードする

TiDBクラスタをアップグレードする前に、まずTiUPまたはTiUPミラーをアップグレードする必要があります。

#### TiUPおよびTiUP Clusterのアップグレード

> **メモ:**
>
> アップグレード対象のクラスタの制御マシンが`https://tiup-mirrors.pingcap.com`にアクセスできない場合は、このセクションをスキップして、[TiUPオフラインミラーをアップグレードする](#upgrade-tiup-offline-mirror)を参照してください。

1. TiUPのバージョンをアップグレードします。TiUPのバージョンが`1.11.3`またはそれ以降であることが推奨されています。

    {{< copyable "shell-regular" >}}

    ```shell
    tiup update --self
    tiup --version
    ```

2. TiUP Clusterのバージョンをアップグレードします。TiUP Clusterのバージョンが`1.11.3`またはそれ以降であることが推奨されています。

    {{< copyable "shell-regular" >}}

    ```shell
    tiup update cluster
    tiup cluster --version
    ```

#### TiUPオフラインミラーをアップグレードする

> **メモ:**
>
> アップグレード対象のクラスタがオフライン方法を使用して展開されていない場合は、このステップをスキップしてください。

新しいバージョンのTiUPミラーをダウンロードし、それを制御マシンにアップロードするために、[TiUPオフラインをデプロイする](/production-deployment-using-tiup.md#deploy-tiup-offline)を参照してください。`local_install.sh`を実行した後、TiUPは上書きアップグレードを完了します。

{{< copyable "shell-regular" >}}

```shell
tar xzvf tidb-community-server-${version}-linux-amd64.tar.gz
sh tidb-community-server-${version}-linux-amd64/local_install.sh
source /home/tidb/.bash_profile
```

上書きアップグレード後、次のコマンドを実行して、サーバーとツールキットのオフラインミラーをサーバーディレクトリにマージします:

{{< copyable "shell-regular" >}}

```bash
tar xf tidb-community-toolkit-${version}-linux-amd64.tar.gz
ls -ld tidb-community-server-${version}-linux-amd64 tidb-community-toolkit-${version}-linux-amd64
cd tidb-community-server-${version}-linux-amd64/
cp -rp keys ~/.tiup/
tiup mirror merge ../tidb-community-toolkit-${version}-linux-amd64
```

ミラーをマージした後、次のコマンドを実行して、TiUP Clusterコンポーネントをアップグレードします:

{{< copyable "shell-regular" >}}

```shell
tiup update cluster
```

これにより、オフラインミラーが正常にアップグレードされます。上書き後にTiUP操作でエラーが発生する場合は、`manifest`が更新されていない可能性があります。TiUPを再実行する前に、TiUPを実行する前に`rm -rf ~/.tiup/manifests/*`を試してみてください。

### ステップ3: TiUPトポロジー構成ファイルを編集する

> **メモ:**
>
> 次のいずれかの状況が適用される場合は、このステップをスキップしてください:
> +`オリジナルのクラスターの構成パラメータを変更していない または tiup cluster を使用して構成パラメータを変更しましたが、もう変更が不要です。`
> +`アップグレード後に、変更されていない構成項目については、v7.4.0のデフォルトのパラメータ値を使用することを希望します。`

1. トポロジファイルを編集するには、`vi`編集モードに入力します：

    {{< copyable "shell-regular" >}}

    ```shell
    tiup cluster edit-config <cluster-name>
    ```

2. [トポロジ](https://github.com/pingcap/tiup/blob/master/embed/examples/cluster/topology.example.yaml)構成テンプレートの形式を参照し、トポロジファイルの`server_configs`セクションで変更するパラメータを入力します。

3. 修正後、<kbd>:</kbd> + <kbd>w</kbd> + <kbd>q</kbd>を入力して変更内容を保存して編集モードを終了します。変更を確認するには<kbd>Y</kbd>を押します。

> **注意:**
>
> v6.6.0にクラスターをアップグレードする前に、v4.0で変更したパラメータがv7.4.0で互換性があることを確認してください。詳細は、[TiKV構成ファイル](/tikv-configuration-file.md)を参照してください。

### ステップ4: 現在のクラスターの健全性状態を確認します

アップグレード中の未定義の動作やその他の問題を避けるために、アップグレード前に現在のクラスターのリージョンの健全性状態を確認することをお勧めします。これには、`check`サブコマンドを使用できます。

{{< copyable "shell-regular" >}}

```shell
tiup cluster check <cluster-name> --cluster
```

コマンドが実行された後、"Region status"の検査結果が出力されます。

+ 検査結果が "All Regions are healthy" の場合、現在のクラスターのすべてのリージョンが健康であり、アップグレードを継続できます。
+ 検査結果が "Regions are not fully healthy: m miss-peer, n pending-peer" で "Please fix unhealthy regions before other operations." のプロンプトが表示された場合、現在のクラスターの一部のリージョンが異常です。異常をトラブルシューティングして、検査結果が "All Regions are healthy" になるまで解決する必要があります。その後、アップグレードを継続できます。

### ステップ5: クラスターのDDLおよびバックアップ状態を確認します

アップグレード中の未定義の動作やその他の予期せぬ問題を避けるために、アップグレード前に以下の項目をチェックすることをお勧めします。

- クラスターDDL：[`ADMIN SHOW DDL`](/sql-statements/sql-statement-admin-show-ddl.md)ステートメントを実行して、進行中のDDLジョブがあるかどうかを確認することをお勧めします。ジョブがある場合は、実行を待つか、[`ADMIN CANCEL DDL`](/sql-statements/sql-statement-admin-cancel-ddl.md)ステートメントを実行してキャンセルします。
- クラスターバックアップ：[`SHOW [BACKUPS|RESTORES]`](/sql-statements/sql-statement-show-backups.md)ステートメントを実行して、クラスターで進行中のバックアップやリストアタスクがあるかどうかを確認することをお勧めします。タスクがある場合は、アップグレードを行う前に完了するまで待ちます。

## TiDBクラスターのアップグレード

このセクションでは、TiDBクラスターのアップグレード方法と、アップグレード後のバージョンの検証方法について説明します。

### 指定バージョンにTiDBクラスターをアップグレード

クラスターをオンラインアップグレードまたはオフラインアップグレードのいずれかの方法でアップグレードできます。

デフォルトでは、TiUP Clusterはオンラインメソッドを使用してTiDBクラスターをアップグレードします。このため、TiDBクラスターはアップグレードプロセス中もサービスを提供できます。オンラインメソッドでは、アップグレードおよび再起動の前に各ノードでリーダーが1つずつ移行されます。したがって、大規模なクラスターでは、アップグレード全体の操作が完了するまで時間がかかります。

データベース停止のためのメンテナンスウィンドウがある場合、オフラインアップグレードメソッドを使用してアップグレード操作を迅速に実行できます。

#### オンラインアップグレード

{{< copyable "shell-regular" >}}

```shell
tiup cluster upgrade <cluster-name> <version>
```

たとえば、クラスターをv7.4.0にアップグレードする場合：

{{< copyable "shell-regular" >}}

```shell
tiup cluster upgrade <cluster-name> v7.4.0
```

> **注意:**
>
> + オンラインアップグレードはすべてのコンポーネントを1つずつアップグレードします。TiKVのアップグレード中、TiKVインスタンスのリーダーはすべて追放され、インスタンスが停止します。デフォルトのタイムアウト時間は5分（300秒）です。このタイムアウト時間を超えると、インスタンスが直接停止します。
>
> + リーダーを追放せずにクラスターを直ちにアップグレードするには、`--force`パラメータを使用できます。ただし、アップグレード中に発生したエラーは無視されます。つまり、アップグレードの失敗について通知されません。そのため、`--force`パラメータを慎重に使用してください。
>
> + 安定したパフォーマンスを維持するためには、TiKVインスタンスのすべてのリーダーが追放されることを確認してください。たとえば、`--transfer-timeout 3600`（単位：秒）のような大きな値を`--transfer-timeout`に設定できます。
>
> + TiFlashのバージョンを5.3未満から5.3以上にアップグレードするには、TiFlashを停止してからアップグレードする必要があります。次の手順に従ってTiFlashを中断せずにアップグレードすることができます：
>    1. TiFlashインスタンスを停止します：`tiup cluster stop <cluster-name> -R tiflash`
>    2. クラスターを再起動せずに（ファイルの更新のみで）TiDBクラスターをアップグレードします：`tiup cluster upgrade <cluster-name> <version> --offline`  例：`tiup cluster upgrade <cluster-name> v6.3.0 --offline`
>    3. TiDBクラスターを再読み込みします：`tiup cluster reload <cluster-name>`。再読み込み後、TiFlashインスタンスが起動され、手動で起動する必要はありません。
>
> + TiDB Binlogを使用してクラスターをローリングアップデートする場合は、クラスター化インデックステーブルを作成しないようにしてください。

#### アップグレード中のコンポーネントのバージョンを指定

tiup-cluster v1.14.0から、クラスターアップグレード中に特定のコンポーネントを特定のバージョンに指定できます。これらのコンポーネントは、別のバージョンを指定しない限り、後続のアップグレードで固定されたバージョンのままです。

> **注意:**
>
> TiDB、TiKV、PD、およびTiCDCなどのバージョン番号を共有するコンポーネントについては、混在バージョンの展開シナリオで正しく動作することを保証するための完全なテストはありません。このセクションは、テスト環境または[技術サポート](/support.md)の支援を受ける場合にのみ使用することを確認してください。

```shell
tiup cluster upgrade -h | grep "version string"
      --alertmanager-version string        alertmanagerのバージョンを固定し、クラスターバージョンに従わなくなります。
      --blackbox-exporter-version string   blackbox-exporterのバージョンを固定し、クラスターバージョンに従わなくなります。
      --cdc-version string                 cdcのバージョンを固定し、クラスターバージョンに従わなくなります。
      --ignore-version-check               対象バージョンが現在のバージョンよりも大きいかどうかをチェックしません。
      --node-exporter-version string       node-exporterのバージョンを固定し、クラスターバージョンに従わなくなります。
      --pd-version string                  pdのバージョンを固定し、クラスターバージョンに従わなくなります。
      --tidb-dashboard-version string      tidb-dashboardのバージョンを固定し、クラスターバージョンに従わなくなります。
      --tiflash-version string             tiflashのバージョンを固定し、クラスターバージョンに従わなくなります。
      --tikv-cdc-version string            tikv-cdcのバージョンを固定し、クラスターバージョンに従わなくなります。
      --tikv-version string                tikvのバージョンを固定し、クラスターバージョンに従わなくなります。
      --tiproxy-version string             tiproxyのバージョンを固定し、クラスターバージョンに従わなくなります。
```

#### オフラインアップグレード

1. オフラインアップグレードの前に、まずクラスター全体を停止する必要があります。

    {{< copyable "shell-regular" >}}

    ```shell
    tiup cluster stop <cluster-name>
    ```

2. `--offline`オプションを使用して`upgrade`コマンドを使用してオフラインアップグレードを実行します。`<cluster-name>`にクラスターの名前、`<version>`にアップグレードするバージョン（例：`v7.4.0`）を入力します。

    {{< copyable "shell-regular" >}}

    ```shell
    tiup cluster upgrade <cluster-name> <version> --offline
    ```

3. アップグレード後、クラスターは自動的に再起動されません。再起動するには`start`コマンドを使用する必要があります。

    {{< copyable "shell-regular" >}}

    ```shell
    tiup cluster start <cluster-name>
    ```

### クラスターのバージョンを確認する

`display`コマンドを実行して最新のクラスターバージョン`TiDB Version`を表示します：

{{< copyable "shell-regular" >}}

```shell
tiup cluster display <cluster-name>
```

```
Cluster type:       tidb
Cluster name:       <cluster-name>
Cluster version:    v7.4.0
```

## FAQ

このセクションでは、TiUPを使用してTiDBクラスターを更新する際に遭遇する一般的な問題について説明します。

### エラーが発生し、アップグレードが中断された場合、このエラーを修正してアップグレードを再開する方法は？

      + ユーザーが提供する入力エラーを検証し、対処する前のプロンプトや診断手順を実行してください。それが完了すれば、完了ログを提供ください。その後に対処に入ります。
      + 再度 `tiup cluster upgrade` コマンドを実行して、アップグレードを再開してください。アップグレード操作は以前にアップグレードされたノードを再起動します。アップグレードされたノードを再起動したくない場合は、`replay` サブコマンドを使用して操作を再試行してください。

1. 操作記録を確認するために `tiup cluster audit` を実行してください：

    {{< copyable "shell-regular" >}}

    ```shell
    tiup cluster audit
    ```

    失敗したアップグレードの操作記録を見つけ、この操作記録のIDを保持してください。このIDは次のステップでの `<audit-id>` の値です。

2. 対応する操作を再試行するために `tiup cluster replay <audit-id>` を実行してください：

    {{< copyable "shell-regular" >}}

    ```shell
    tiup cluster replay <audit-id>
    ```

### v6.2.0 以降のバージョンにアップグレードする際にアップグレードがスタックする問題の解決方法

v6.2.0 から、TiDB は[並行 DDL フレームワーク](/ddl-introduction.md#how-the-online-ddl-asynchronous-change-works-in-tidb)をデフォルトで有効にして、並列 DDL の実行を可能とします。このフレームワークは、DDL ジョブのストレージを KV キューからテーブルキューに変更します。この変更により、いくつかのシナリオでアップグレードがスタックする可能性があります。次のシナリオはこの問題を引き起こす可能性があるものと、それに対する対応策です：

- プラグインの読み込みによりアップグレードがスタックする

    アップグレード中に、DDL ステートメントの実行を必要とする特定のプラグインを読み込むと、アップグレードがスタックする可能性があります。

    **解決策**：アップグレード中にプラグインの読み込みを避けてください。アップグレードが完了した後にプラグインを読み込んでください。

- オフラインアップグレードのため `kill -9` コマンドを使用したことによるアップグレードのスタック

    - 注意事項：オフラインアップグレードを実行するために `kill -9` コマンドを使用しないでください。必要な場合は、新しいバージョンの TiDB ノードを 2 分後に再起動してください。
    - アップグレードが既にスタックしている場合、影響を受ける TiDB ノードを再起動してください。問題が発生したばかりの場合は、ノードを 2 分後に再起動することをお勧めします。

- DDL オーナーの変更によりアップグレードがスタックする

    マルチインスタンスのシナリオにおいて、ネットワークやハードウェアの障害が DDL オーナーの変更を引き起こす可能性があります。アップグレードフェーズで未完了の DDL ステートメントがある場合、アップグレードがスタックする可能性があります。

    **解決策**：

    1. スタックした TiDB ノードを終了します（`kill -9` を使用しないでください）。
    2. 新しいバージョンの TiDB ノードを再起動してください。

### アップグレード中に leader の追い出し待ちが長すぎる場合、このステップをスキップしてスピーディなアップグレードを行う方法

`--force` を指定することで、PD リーダーの転送および TiKV リーダーの追い出しのプロセスをスキップし、クラスタを直接再起動してバージョンをアップグレードします。これはオンラインで実行されているクラスタに大きな影響を与えます。次のコマンドでは、`<version>` を `v7.4.0` などにアップグレードするバージョンに置き換えてください。

{{< copyable "shell-regular" >}}

```shell
tiup cluster upgrade <cluster-name> <version> --force
```

### TiDB クラスタをアップグレードした後に pd-ctl などのツールのバージョンを更新する方法

TiUP を使用して、対応するバージョンの `ctl` コンポーネントをインストールすることで、ツールのバージョンをアップグレードできます：

{{< copyable "shell-regular" >}}

```shell
tiup install ctl:v7.4.0
```