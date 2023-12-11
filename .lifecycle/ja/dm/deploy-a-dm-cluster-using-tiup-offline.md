---
title: TiUPを使用してオフラインでDMクラスターをデプロイする
summary: TiUPを使用してオフラインでDMクラスターをデプロイする方法について紹介します。

# TiUPを使用してオフラインでDMクラスターをデプロイする

このドキュメントでは、TiUPを使用してオフラインでDMクラスターをデプロイする方法について説明します。

## ステップ1: TiUPオフラインコンポーネントパッケージの準備

- オンラインでTiUPパッケージマネージャをインストールします。

    1. TiUPツールをインストールします:

        ```shell
        curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
        ```

    2. グローバル環境変数を再宣言します:

        ```shell
        source .bash_profile
        ```

    3. TiUPがインストールされているか確認します:

        ```shell
        which tiup
        ```

- TiUPを使用してミラーを取得します。

    1. インターネットにアクセスできるマシンで必要なコンポーネントを取得します:

        ```bash
        # ${version}を必要なバージョンに変更できます。
        tiup mirror clone tidb-dm-${version}-linux-amd64 --os=linux --arch=amd64 \
            --dm-master=${version} --dm-worker=${version} --dmctl=${version} \
            --alertmanager=v0.17.0 --grafana=v4.0.3 --prometheus=v4.0.3 \
            --tiup=v$(tiup --version|grep 'tiup'|awk -F ' ' '{print $1}') --dm=v$(tiup --version|grep 'tiup'|awk -F ' ' '{print $1}')
        ```

        上記のコマンドにより、TiUPで管理されるコンポーネントパッケージが含まれる `tidb-dm-${version}-linux-amd64` というディレクトリが現在のディレクトリに作成されます。

    2. `tar`コマンドを使用してコンポーネントパッケージをパッケージ化し、絶縁された環境の制御マシンにパッケージを送信します:

        ```bash
        tar czvf tidb-dm-${version}-linux-amd64.tar.gz tidb-dm-${version}-linux-amd64
        ```

        `tidb-dm-${version}-linux-amd64.tar.gz`は独立したオフライン環境パッケージです。

## ステップ2: オフラインTiUPコンポーネントのデプロイ

パッケージをターゲットクラスターの制御マシンに送信した後、次のコマンドを実行してTiUPコンポーネントをインストールします:

```bash
# ${version}を必要なバージョンに変更できます。
tar xzvf tidb-dm-${version}-linux-amd64.tar.gz
sh tidb-dm-${version}-linux-amd64/local_install.sh
source /home/tidb/.bash_profile
```

`local_install.sh`スクリプトは自動的に、`tiup mirror set tidb-dm-${version}-linux-amd64`コマンドを実行して現在のミラーアドレスを `tidb-dm-${version}-linux-amd64` に設定します。

ミラーを別のディレクトリに切り替えるには、手動で `tiup mirror set <ミラーディレクトリ>`コマンドを実行します。公式ミラーに切り替える場合は、`tiup mirror set https://tiup-mirrors.pingcap.com`を実行します。

## ステップ3: 初期化構成ファイルの編集

異なるクラスタトポロジに応じて、クラスタ初期化構成ファイルを編集する必要があります。

完全な構成テンプレートについては、[TiUP構成パラメータテンプレート](https://github.com/pingcap/tiup/blob/master/embed/examples/dm/topology.example.yaml)を参照してください。 `topology.yaml`という構成ファイルを作成します。その他の組み合わせシナリオでは、テンプレートに従って必要に応じて構成ファイルを編集します。

3つのDMマスター、3つのDMワーカー、および1つの監視コンポーネントインスタンスをデプロイする構成は次の通りです:

```yaml
---
global:
  user: "tidb"
  ssh_port: 22
  deploy_dir: "/home/tidb/dm/deploy"
  data_dir: "/home/tidb/dm/data"
  # arch: "amd64"

master_servers:
  - host: 172.19.0.101
  - host: 172.19.0.102
  - host: 172.19.0.103

worker_servers:
  - host: 172.19.0.101
  - host: 172.19.0.102
  - host: 172.19.0.103

monitoring_servers:
  - host: 172.19.0.101

grafana_servers:
  - host: 172.19.0.101

alertmanager_servers:
  - host: 172.19.0.101
```

> **注意:**
>
> - DMクラスタの高可用性を確保する必要がない場合は、DMマスターノードを1つだけデプロイし、デプロイするDMワーカーノードの数は、移行する上流のMySQL/MariaDBインスタンスの数より少なくてはなりません。
>
> - DMクラスタの高可用性を確保するためには、3つのDMマスターノードをデプロイすることをお勧めします。デプロイするDMワーカーノードの数は、移行する上流のMySQL/MariaDBインスタンスの数より多くなければなりません（たとえば、DMワーカーノードの数は、上流インスタンスの数より2つ多い）。
>
> - グローバルに有効なパラメータを設定する場合は、構成ファイルの `server_configs`セクションに対応するコンポーネントのパラメータを構成します。
>
> - 特定のノードで有効なパラメータを設定する場合は、そのノードの `config`にパラメータを構成します。
>
> - `.`を使用して構成のサブカテゴリを示します。たとえば、`log.slow-threshold`のように使用します。その他のフォーマットについては、[TiUP構成テンプレート](https://github.com/pingcap/tiup/blob/master/embed/examples/dm/topology.example.yaml)を参照してください。
>
> - その他のパラメータの説明については、[マスター `config.toml.example`](https://github.com/pingcap/dm/blob/master/dm/master/dm-master.toml)および[ワーカー `config.toml.example`](https://github.com/pingcap/dm/blob/master/dm/worker/dm-worker.toml)を参照してください。

## ステップ4: デプロイコマンドの実行

> **注意:**
>
> TiUPを使用してDMをデプロイする際には、セキュリティ認証のためにシークレットキーまたは対話型パスワードを使用できます:
>
> - シークレットキーを使用する場合は、`-i`または`--identity_file`を介してキーのパスを指定できます;
> - パスワードを使用する場合は、`-p`フラグを追加してパスワードの対話ウィンドウに入力します;
> - ターゲットマシンへのパスワードフリーログインが構成されている場合、認証は不要です。

```shell
tiup dm deploy dm-test ${version} ./topology.yaml --user root [-p] [-i /home/root/.ssh/gcp_rsa]
```

上記のコマンドでは、以下のようになります:

- デプロイされるDMクラスターの名前は `dm-test` です。
- DMクラスターのバージョンは `${version}` です。最新のバージョンは `tiup list dm-master`を実行して確認できます。
- 初期化構成ファイルは `topology.yaml` です。
- `--user root`: `root` キーを使用してターゲットマシンにログインしてクラスターデプロイを完了します。または、`ssh`および`sudo`権限を持つ他のユーザを使用してデプロイを完了することもできます。
- `[-i]` および `[-p]`: オプションです。パスワードなしでターゲットマシンにログインを構成した場合、これらのパラメータは必要ありません。必要な場合は、2つのパラメータのいずれかを選択します。`[-i]`はターゲットマシンにアクセスできる `root` ユーザ（または`--user`で指定された他のユーザ）のプライベートキーです。`[-p]`はユーザパスワードを対話的に入力するために使用します。
- TiUP DMは埋め込みSSHクライアントを使用します。制御マシンのシステムのネイティブSSHクライアントを使用したい場合は、[クラスタに接続するためにシステムのネイティブSSHクライアントを使用する](/dm/maintain-dm-using-tiup.md#use-the-systems-native-ssh-client-to-connect-to-cluster)に従って構成を編集してください。

ログの最後に `Deployed cluster `dm-test` successfully` と表示されます。これはデプロイが成功したことを示しています。

## ステップ5: TiUPで管理されているクラスタの確認

```shell
tiup dm list
```

TiUPは複数のDMクラスタを管理することができます。上記のコマンドは現在TiUPで管理されているすべてのクラスタの情報を出力し、クラスタの名前、デプロイユーザ、バージョン、およびシークレットキーの情報を含みます:

```log
Name  User  Version  Path                                  PrivateKey
```
----  ----  -------  ----                                  ----------
dm-test  tidb  ${version}  /root/.tiup/storage/dm/clusters/dm-test  /root/.tiup/storage/dm/clusters/dm-test/ssh/id_rsa
```

## ステップ 6：展開されたDMクラスタのステータスを確認する

`dm-test` クラスタのステータスを確認するには、次のコマンドを実行します：

{{< copyable "shell-regular" >}}

```shell
tiup dm display dm-test
```

期待される出力には、インスタンスID、ロール、ホスト、リスニングポート、およびステータス（クラスタがまだ開始されていないため、ステータスは `Down`/`inactive` です）、および `dm-test` クラスタのディレクトリ情報が含まれます。

## ステップ 7：クラスタを起動する

{{< copyable "shell-regular" >}}

```shell
tiup dm start dm-test
```

出力ログに ```Started cluster `dm-test` successfully``` が含まれている場合、起動は成功しています。

## ステップ 8：クラスタの実行状態を検証する

TiUPを使用してDMクラスタのステータスを確認します：

{{< copyable "shell-regular" >}}

```shell
tiup dm display dm-test
```

出力に `Status` が `Up` である場合、クラスタのステータスは正常です。