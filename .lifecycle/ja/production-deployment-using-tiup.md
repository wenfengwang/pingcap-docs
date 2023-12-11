---
title: TiUPを使用してTiDBクラスターをデプロイする
summary: TiUPを使用して簡単にTiDBクラスターをデプロイする方法を学びます。
aliases: ['/docs/dev/production-deployment-using-tiup/', '/docs/dev/how-to/deploy/orchestrated/tiup/', '/docs/dev/tiflash/deploy-tiflash/', '/docs/dev/reference/tiflash/deploy/', '/tidb/dev/deploy-tidb-from-dbdeployer/', '/docs/dev/deploy-tidb-from-dbdeployer/', '/docs/dev/how-to/get-started/deploy-tidb-from-dbdeployer/', '/tidb/dev/deploy-tidb-from-homebrew/', '/docs/dev/deploy-tidb-from-homebrew/', '/docs/dev/how-to/get-started/deploy-tidb-from-homebrew/', '/tidb/dev/production-offline-deployment-using-tiup', '/docs/dev/production-offline-deployment-using-tiup/', '/tidb/dev/deploy-tidb-from-binary', '/tidb/dev/production-deployment-from-binary-tarball', '/tidb/dev/test-deployment-from-binary-tarball', '/tidb/dev/deploy-test-cluster-using-docker-compose', '/tidb/dev/test-deployment-using-docker']
---

# TiUPを使用してTiDBクラスターをデプロイする

[TiUP](https://github.com/pingcap/tiup)はTiDB 4.0で導入されたクラスター操作および保守ツールです。TiUPはGolangで書かれたクラスター管理コンポーネントである[TiUPクラスター](https://github.com/pingcap/tiup/tree/master/components/cluster)を提供します。TiUPクラスターを使用すると、TiDBクラスターのデプロイ、起動、停止、破棄、スケーリング、アップグレード、およびTiDBクラスターのパラメータの管理など、日常のデータベース操作を簡単に行うことができます。

TiUPは、TiDB、TiFlash、TiDB Binlog、TiCDC、およびモニタリングシステムのデプロイをサポートしています。このドキュメントでは、異なるトポロジのTiDBクラスターのデプロイ方法について紹介します。

## ステップ1. 必要条件と事前チェック

以下のドキュメントを読んでいることを確認してください:

- [ハードウェアおよびソフトウェアの要件](/hardware-and-software-requirements.md)
- [デプロイ前の環境とシステム構成のチェック](/check-before-deployment.md)

## ステップ2. 制御マシンにTiUPをデプロイする

TiUPを制御マシンにオンラインデプロイまたはオフラインデプロイのいずれかの方法で配置することができます。

### TiUPをオンラインでデプロイする

通常のユーザーアカウント（例: `tidb`ユーザー）を使用して制御マシンにログインします。以降のTiUPのインストールとクラスター管理は、`tidb`ユーザーによって実行されます。

1. 以下のコマンドを実行してTiUPをインストールします:

    {{< copyable "shell-regular" >}}

    ```shell
    curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
    ```

2. TiUPの環境変数を設定します:

    1. グローバル環境変数を再宣言します:

        {{< copyable "shell-regular" >}}

        ```shell
        source .bash_profile
        ```

    2. TiUPがインストールされているかどうかを確認します:

        {{< copyable "shell-regular" >}}

        ```shell
        which tiup
        ```

3. TiUPクラスターコンポーネントをインストールします:

    {{< copyable "shell-regular" >}}

    ```shell
    tiup cluster
    ```

4. TiUPがすでにインストールされている場合は、TiUPクラスターコンポーネントを最新バージョンにアップデートします:

    {{< copyable "shell-regular" >}}

    ```shell
    tiup update --self && tiup update cluster
    ```

    `Update successfully!`が表示された場合は、TiUPクラスターが正常に更新されています。

5. TiUPクラスーターの現在のバージョンを確認します:

    {{< copyable "shell-regular" >}}

    ```shell
    tiup --binary cluster
    ```

### TiUPをオフラインでデプロイする

TiUPを使用してオフラインでTiDBクラスターをデプロイする場合は、以下の手順を実行してください:

#### TiUPオフラインコンポーネントパッケージの準備

方法1: [公式ダウンロードページ](https://www.pingcap.com/download/)で、ターゲットのTiDBバージョンのオフラインミラーパッケージ（TiUPオフラインパッケージを含む）を選択します。サーバーパッケージとツールキットパッケージを同時にダウンロードする必要があることに注意してください。

方法2: `tiup mirror clone`を使用してオフラインコンポーネントパッケージを手動で作成します。手順は以下のとおりです:

1. TiUPパッケージマネージャーをオンラインでインストールします。

    1. TiUPツールをインストールします:

        {{< copyable "shell-regular" >}}

        ```shell
        curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
        ```

    2. グローバル環境変数を再宣言します:

        {{< copyable "shell-regular" >}}

        ```shell
        source .bash_profile
        ```

    3. TiUPがインストールされているかどうかを確認します:

        {{< copyable "shell-regular" >}}

        ```shell
        which tiup
        ```

2. TiUPを使用してミラーを取得します。

    1. インターネットにアクセスできるマシンで必要なコンポーネントを取得します:

        {{< copyable "shell-regular" >}}

        ```shell
        tiup mirror clone tidb-community-server-${version}-linux-amd64 ${version} --os=linux --arch=amd64
        ```

        上記のコマンドは、`tidb-community-server-${version}-linux-amd64`というディレクトリを作成します。これにはクラスターを開始するために必要なコンポーネントパッケージが含まれています。

    2. `tar`コマンドを使用してコンポーネントパッケージをパッケージ化し、そのパッケージを制御マシンで孤立した環境に送信します:

        {{< copyable "shell-regular" >}}

        ```bash
        tar czvf tidb-community-server-${version}-linux-amd64.tar.gz tidb-community-server-${version}-linux-amd64
        ```

        `tidb-community-server-${version}-linux-amd64.tar.gz`は独立したオフライン環境パッケージです。

3. オフラインミラーをカスタマイズしたり、既存のオフラインミラーの内容を調整したりします。

    既存のオフラインミラーを調整したい場合（例: コンポーネントの新しいバージョンを追加する場合）、以下の手順を実行します:

    1. オフラインミラーを取得する際に、コンポーネントやバージョン情報などをパラメータを使用して指定することで、不完全なオフラインミラーを取得することができます。たとえば、次のコマンドを実行して、TiUP v1.12.3およびTiUPクラスーターv1.12.3のみを含むオフラインミラーを取得できます:

        {{< copyable "shell-regular" >}}

        ```bash
        tiup mirror clone tiup-custom-mirror-v1.12.3 --tiup v1.12.3 --cluster v1.12.3
        ```

        特定のプラットフォームのコンポーネントのみを必要とする場合は、`--os`または`--arch`パラメータを使用して指定することができます。

    2. "TiUPを使用してミラーを取得"のステップ2を参照し、この不完全なオフラインミラーを制御マシンで孤立した環境に送信します。

    3. 制御マシンでの現在のオフラインミラーのパスを確認します。TiUPツールが最新バージョンの場合、次のコマンドを実行することで現在のミラーアドレスを取得できます:

        {{< copyable "shell-regular" >}}

        ```bash
        tiup mirror show
        ```

        上記のコマンドの出力が"show"コマンドが存在しないと指摘している場合、TiUPの古いバージョンを使用している可能性があります。この場合は、現在のミラーアドレスを`$HOME/.tiup/tiup.toml`から取得することができます。このミラーアドレスを記録しておきます。以下の手順では、このアドレスを`${base_mirror}`と呼びます。

    4. 不完全なオフラインミラーを既存のオフラインミラーにマージします:

        まず、現在のオフラインミラー内の`keys`ディレクトリを`$HOME/.tiup/`ディレクトリにコピーします:

        {{< copyable "shell-regular" >}}

        ```bash
        cp -r ${base_mirror}/keys $HOME/.tiup/
        ```

        次に、TiUPコマンドを使用して不完全なオフラインミラーを使用中のミラーにマージします:

        {{< copyable "shell-regular" >}}

        ```bash
        tiup mirror merge tiup-custom-mirror-v1.12.3
        ```

    5. 上記の手順が完了したら、`tiup list`コマンドを実行して結果を確認します。このドキュメントの例では、`tiup list tiup`および`tiup list cluster`の出力は、それぞれ`v1.12.3`の対応するコンポーネントが利用可能であることを示しています。

#### オフラインTiUPコンポーネントをデプロイする

パッケージをターゲットクラスーターの制御マシンに送信した後、以下のコマンドを実行してTiUPコンポーネントをインストールします:

{{< copyable "shell-regular" >}}

```bash
tar xzvf tidb-community-server-${version}-linux-amd64.tar.gz && \
sh tidb-community-server-${version}-linux-amd64/local_install.sh && \
source /home/tidb/.bash_profile
```

`local_install.sh`スクリプトは、`tiup mirror set tidb-community-server-${version}-linux-amd64`コマンドを自動的に実行して、現在のミラーアドレスを`tidb-community-server-${version}-linux-amd64`に設定します。

#### オフラインパッケージをマージする

[公式ダウンロードページ](https://www.pingcap.com/download/)からオフラインパッケージをダウンロードした場合は、サーバーパッケージとツールキットパッケージをオフラインミラーにマージする必要があります。`tiup mirror clone`コマンドを使用してオフラインコンポーネントパッケージを手動でパッケージ化した場合は、このステップをスキップできます。

次のコマンドを実行して、オフラインツールキットパッケージをサーバーパッケージディレクトリにマージします：

```bash
tar xf tidb-community-toolkit-${version}-linux-amd64.tar.gz
ls -ld tidb-community-server-${version}-linux-amd64 tidb-community-toolkit-${version}-linux-amd64
cd tidb-community-server-${version}-linux-amd64/
cp -rp keys ~/.tiup/
tiup mirror merge ../tidb-community-toolkit-${version}-linux-amd64
```

ミラーを別のディレクトリに切り替えるには、`tiup mirror set <mirror-dir>`コマンドを実行します。オンライン環境にミラーを切り替えるには、`tiup mirror set https://tiup-mirrors.pingcap.com`コマンドを実行します。

## ステップ3. クラスタートポロジーファイルの初期化

次のコマンドを実行してクラスタートポロジーファイルを作成します：

{{< copyable "shell-regular" >}}

```shell
tiup cluster template > topology.yaml
```

以下の二つの一般的なシナリオにおいて、コマンドを実行して推奨トポロジーテンプレートを生成できます：

- ハイブリッドデプロイメント：単一のマシンに複数のインスタンスを展開します。詳細については、[ハイブリッドデプロイメントトポロジー](/hybrid-deployment-topology.md) を参照してください。

    {{< copyable "shell-regular" >}}

    ```shell
    tiup cluster template --full > topology.yaml
    ```

- 地理的に分散したデプロイメント：TiDBクラスターを地理的に分散したデータセンターにデプロイします。詳細については、[地理的に分散したデプロイメントトポロジー](/geo-distributed-deployment-topology.md) を参照してください。

    {{< copyable "shell-regular" >}}

    ```shell
    tiup cluster template --multi-dc > topology.yaml
    ```

`vi topology.yaml`を実行して構成ファイルの内容を確認します：

{{< copyable "shell-regular" >}}

```shell
global:
  user: "tidb"
  ssh_port: 22
  deploy_dir: "/tidb-deploy"
  data_dir: "/tidb-data"
server_configs: {}
pd_servers:
  - host: 10.0.1.4
  - host: 10.0.1.5
  - host: 10.0.1.6
tidb_servers:
  - host: 10.0.1.7
  - host: 10.0.1.8
  - host: 10.0.1.9
tikv_servers:
  - host: 10.0.1.1
  - host: 10.0.1.2
  - host: 10.0.1.3
monitoring_servers:
  - host: 10.0.1.4
grafana_servers:
  - host: 10.0.1.4
alertmanager_servers:
  - host: 10.0.1.4
```

以下の例では、七つの一般的なシナリオが含まれています。対応するリンクのトポロジーの記述とテンプレートに従って、構成ファイル（`topology.yaml`と呼ばれる）を変更する必要があります。その他のシナリオについては、対応する構成テンプレートを編集してください。

| アプリケーション | 構成タスク | 構成ファイルテンプレート | トポロジーの説明 |
| :-- | :-- | :-- | :-- |
| OLTP | [最小限のトポロジーを展開](/minimal-deployment-topology.md) | [シンプルな最小限の構成テンプレート](https://github.com/pingcap/docs/blob/master/config-templates/simple-mini.yaml) <br/> [フル最小限の構成テンプレート](https://github.com/pingcap/docs/blob/master/config-templates/complex-mini.yaml) | これは、tidb-server、tikv-server、pd-serverを含む基本的なクラスタートポロジーです。 |
| HTAP | [TiFlashトポロジーを展開](/tiflash-deployment-topology.md) | [シンプルなTiFlash構成テンプレート](https://github.com/pingcap/docs/blob/master/config-templates/simple-tiflash.yaml) <br/> [フルTiFlash構成テンプレート](https://github.com/pingcap/docs/blob/master/config-templates/complex-tiflash.yaml) | これは、TiFlashを最小限のクラスタートポロジーとともに展開するものです。TiFlashは列指向のストレージエンジンであり、徐々に標準的なクラスタートポロジーとなっています。 |
| [TiCDC](/ticdc/ticdc-overview.md)を使用した増分データの複製 | [TiCDCトポロジーを展開](/ticdc-deployment-topology.md) | [シンプルなTiCDC構成テンプレート](https://github.com/pingcap/docs/blob/master/config-templates/simple-cdc.yaml) <br/> [フルTiCDC構成テンプレート](https://github.com/pingcap/docs/blob/master/config-templates/complex-cdc.yaml) | これは、TiCDCを最小限のクラスタートポロジーとともに展開するものです。TiCDCはTiDB、MySQL、Kafka、MQ、およびストレージサービスなど、複数のダウンストリームプラットフォームをサポートしています。 |
| [TiDB Binlog](/tidb-binlog/tidb-binlog-overview.md)を使用した増分データの複製 | [TiDB Binlogトポロジーを展開](/tidb-binlog-deployment-topology.md) | [TiDB Binlog（MySQLをダウンストリームとする）のシンプルな構成テンプレート](https://github.com/pingcap/docs/blob/master/config-templates/simple-tidb-binlog.yaml) <br/> [TiDB Binlog（ファイルをダウンストリームとする）のシンプルな構成テンプレート](https://github.com/pingcap/docs/blob/master/config-templates/simple-file-binlog.yaml) <br/> [フルTiDB Binlog構成テンプレート](https://github.com/pingcap/docs/blob/master/config-templates/complex-tidb-binlog.yaml) | これは、TiDB Binlogを最小限のクラスタートポロジーとともに展開するものです。 |
| Spark上でOLAPを使用する | [TiSparkトポロジーを展開](/tispark-deployment-topology.md) | [シンプルなTiSpark構成テンプレート](https://github.com/pingcap/docs/blob/master/config-templates/simple-tispark.yaml) <br/> [フルTiSpark構成テンプレート](https://github.com/pingcap/docs/blob/master/config-templates/complex-tispark.yaml) | これは、TiSparkを最小限のクラスタートポロジーとともに展開するものです。TiSparkは、Apache SparkをTiDB/TiKVの上で実行し、OLAPクエリに回答するために構築されたコンポーネントです。現在、TiUP clusterでのTiSparkのサポートは **実験的** です。 |
| 単一のマシンに複数のインスタンスを展開する | [ハイブリッドトポロジーを展開](/hybrid-deployment-topology.md) | [ハイブリッドデプロイメントのシンプルな構成テンプレート](https://github.com/pingcap/docs/blob/master/config-templates/simple-multi-instance.yaml) <br/> [ハイブリッドデプロイメントのフル構成テンプレート](https://github.com/pingcap/docs/blob/master/config-templates/complex-multi-instance.yaml) | ディレクトリ、ポート、リソースの比率、ラベルなどの追加構成が必要な場合、デプロイメントトポロジーも適用されます。 |
| データセンター間にTiDBクラスターを展開する | [地理的に分散した展開トポロジーを展開](/geo-distributed-deployment-topology.md) | [地理的に分散した展開の構成テンプレート](https://github.com/pingcap/docs/blob/master/config-templates/geo-redundancy-deployment.yaml) | このトポロジーは、2つの都市に3つのデータセンターの典型的なアーキテクチャを取り上げています。地理的に分散した展開アーキテクチャと注目すべきキー構成を説明しています。 |

> **Note:**
>
> - グローバルに有効にする必要があるパラメータについては、構成ファイルの対応するコンポーネントのこれらのパラメータを`server_configs`セクションに構成します。
> - 特定のノードで有効にする必要があるパラメータについては、そのノードの`config`にパラメータを構成します。
> - `.`を使用して、`log.slow-threshold`などの構成のサブカテゴリを示します。詳細なフォーマットについては、[TiUP構成テンプレート](https://github.com/pingcap/tiup/blob/master/embed/examples/cluster/topology.example.yaml)を参照してください。
> - ターゲットマシンで作成するユーザーグループ名を指定する必要がある場合は、[この例](https://github.com/pingcap/tiup/blob/master/embed/examples/cluster/topology.example.yaml#L7)をご覧ください。

さらなる構成の説明については、以下の構成例をご覧ください：

- [TiDB `config.toml.example`](https://github.com/pingcap/tidb/blob/master/pkg/config/config.toml.example)
- [TiKV `config.toml.example`](https://github.com/tikv/tikv/blob/master/etc/config-template.toml)
- [PD `config.toml.example`](https://github.com/pingcap/pd/blob/master/conf/config.toml)
- [TiFlash `config.toml.example`](https://github.com/pingcap/tiflash/blob/master/etc/config-template.toml)

## ステップ4. デプロイコマンドを実行する

> **Note:**
>
> TiUPを使用してTiDBを展開する際に、セキュリティ認証のためにシークレットキーまたは対話型パスワードを使用できます：
>
> - シークレットキーを使用する場合は、`-i`または`--identity_file`を介してキーのパスを指定します。
> - パスワードを使用する場合は、`-p`フラグを追加してパスワードの入力ウィンドウを開きます。
> - ターゲットマシンへのパスワードフリーログインが構成されている場合は、認証は不要です。
> 一般的に、TiUPは`topology.yaml`ファイルで指定したユーザーとグループをターゲットマシン上で作成しますが、次の例外があります。

> - `topology.yaml`で構成されたユーザー名が既にターゲットマシン上で存在する場合。
> - `--skip-create-user`オプションを使用して、ユーザーの作成手順を明示的にスキップしている場合。

`deploy`コマンドを実行する前に、`check`および`check --apply`コマンドを使用して、クラスター内の潜在的なリスクを検出し自動的に修復します。

1. 潜在的なリスクのチェック：

    {{< copyable "shell-regular" >}}

    ```shell
    tiup cluster check ./topology.yaml --user root [-p] [-i /home/root/.ssh/gcp_rsa]
    ```

2. 自動修復の有効化：

    {{< copyable "shell-regular" >}}

    ```shell
    tiup cluster check ./topology.yaml --apply --user root [-p] [-i /home/root/.ssh/gcp_rsa]
    ```

3. TiDBクラスターをデプロイする：

    {{< copyable "shell-regular" >}}

    ```shell
    tiup cluster deploy tidb-test v7.4.0 ./topology.yaml --user root [-p] [-i /home/root/.ssh/gcp_rsa]
    ```

上記の`tiup cluster deploy`コマンドでは、以下が含まれます。

- `tidb-test`はデプロイされるTiDBクラスターの名前です。
- `v7.4.0`はデプロイされるTiDBクラスターのバージョンです。「tiup list tidb」を実行することで、最新のサポートされているバージョンを確認できます。
- `topology.yaml`は初期化構成ファイルです。
- `--user root`は、クラスターのデプロイを完了するためにターゲットマシンに`root`ユーザーとしてログインすることを示します。 `root`ユーザーは、ターゲットマシンへの`ssh`および`sudo`の権限を持っていることが期待されます。 代わりに、デプロイを完了するために`ssh`および`sudo`の権限を持つ他のユーザーを使用することもできます。
- `[-i]`および`[-p]`はオプションです。パスワードなしでターゲットマシンへのログインを構成した場合、これらのパラメータは必要ありません。 そうでない場合は、2つのパラメータのいずれかを選択します。 `[-i]`は、ターゲットマシンにアクセス権を持つ`root`(または`--user`で指定された他のユーザー)の秘密鍵です。 `[-p]`はユーザーパスワードを対話的に入力するために使用します。

出力ログの最後に、`Deployed cluster 'tidb-test' successfully`が表示されます。これはデプロイが成功したことを示します。

## ステップ5. TiUPで管理されるクラスターの確認

{{< copyable "shell-regular" >}}

```shell
tiup cluster list
```

TiUPは複数のTiDBクラスターを管理することができます。前述のコマンドは、現在TiUPで管理されているすべてのクラスターの情報を出力し、クラスター名、デプロイユーザー、バージョン、およびシークレットキー情報を含みます。

## ステップ6. デプロイされたTiDBクラスターのステータスを確認

例えば、次のコマンドを実行して`tidb-test`クラスターのステータスを確認します。

{{< copyable "shell-regular" >}}

```shell
tiup cluster display tidb-test
```

期待される出力には、インスタンスID、ロール、ホスト、リスニングポート、ステータス（クラスターがまだ開始されていないため、ステータスは`Down`/`inactive`になります）、およびディレクトリ情報が含まれます。

## ステップ7. TiDBクラスターを起動

TiUPクラスターv1.9.0以降、セーフ開始が新しい開始方法として導入されました。この方法でデータベースを開始することで、データベースのセキュリティが向上します。この方法を使用することをお勧めします。

セーフ開始後、TiUPはTiDBルートユーザーのパスワードを自動的に生成し、そのパスワードをコマンドラインインターフェイスで返します。

> **注意:**
>
> - TiDBクラスターを安全に開始した後、パスワードなしでルートユーザーでTiDBにログインすることはできません。したがって、将来のログインのために、コマンドの出力で返されたパスワードを記録する必要があります。
>
> - パスワードは1回だけ生成されます。それを記録しなかったり忘れた場合、[ルートパスワードを忘れた場合](/user-account-management.md#forget-the-root-password)を参照して、パスワードを変更してください。

メソッド1: セーフ開始

{{< copyable "shell-regular" >}}

```shell
tiup cluster start tidb-test --init
```

出力が次のようになっている場合、開始に成功しています。

{{< copyable "shell-regular" >}}

```shell
Started cluster 'tidb-test' successfully.
TiDBデータベースのルートパスワードが変更されました。
新しいパスワードは: 'y_+3Hwp=*AWz8971s6' です。
どこか安全な場所にコピーして記録してください。これは1度だけ表示され、保存されません。
将来、生成されたパスワードを再取得することはできません。
```

メソッド2: 標準的な開始

{{< copyable "shell-regular" >}}

```shell
tiup cluster start tidb-test
```

出力ログに```Started cluster 'tidb-test' successfully```が含まれている場合、開始に成功しています。標準的な開始後、パスワードなしでデータベースにログインすることができます。

## ステップ8. TiDBクラスターの実行状態の検証

{{< copyable "shell-regular" >}}

```shell
tiup cluster display tidb-test
```

出力ログに`Up`の状態が表示されている場合、クラスターは正常に実行されています。

## 関連情報

TiDBクラスターと一緒に[TiFlash](/tiflash/tiflash-overview.md)を展開した場合は、次のドキュメントを参照してください。

- [TiFlashの使用](/tiflash/tiflash-overview.md#use-tiflash)
- [TiFlashクラスターのメンテナンス](/tiflash/maintain-tiflash.md)
- [TiFlashアラートのルールと対処方法](/tiflash/tiflash-alert-rules.md)
- [TiFlashのトラブルシューティング](/tiflash/troubleshoot-tiflash.md)

TiDBクラスターと一緒に[TiCDC](/ticdc/ticdc-overview.md)を展開した場合は、次のドキュメントを参照してください。

- [Changefeedの概要](/ticdc/ticdc-changefeed-overview.md)
- [Changefeedの管理](/ticdc/ticdc-manage-changefeed.md)
- [TiCDCのトラブルシューティング](/ticdc/troubleshoot-ticdc.md)
- [TiCDCのFAQ](/ticdc/ticdc-faq.md)