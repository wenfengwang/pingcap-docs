---
title: TiCDC の展開とメンテナンス
summary: TiCDC の展開および実行に関するハードウェアおよびソフトウェアの推奨事項、およびその展開およびメンテナンス方法について学びます。

# TiCDC の展開とメンテナンス

このドキュメントでは、TiCDC クラスターの展開とメンテナンス方法について説明し、ハードウェアおよびソフトウェアの推奨事項を含めます。TiCDC を新しい TiDB クラスターと一緒に展開するか、既存の TiDB クラスターに TiCDC コンポーネントを追加することができます。

## ソフトウェアおよびハードウェアの推奨事項

本番環境では、TiCDC のソフトウェアおよびハードウェアの推奨事項は次のとおりです。

| Linux OS       | バージョン         |
| :----------------------- | :----------: |
| Red Hat Enterprise Linux | 7.3 以降   |
| CentOS                   | 7.3 以降   |

| CPU | メモリ | ディスク | ネットワーク | TiCDC クラスターインスタンス数（本番環境の最小要件） |
| :--- | :--- | :--- | :--- | :--- |
| 16 コア以上 | 64 GB以上 | 500 GB以上の SSD | 10 Gigabit ネットワークカード（2 台が推奨） | 2 |

詳細については、[ソフトウェアおよびハードウェアの推奨事項](/hardware-and-software-requirements.md)を参照してください。

## TiUP を使用して TiDB クラスターに TiCDC を含めてデプロイする

TiUP を使用して新しい TiDB クラスターを展開する際、同時に TiCDC を展開することもできます。TiUP が使用する構成ファイルに `cdc_servers` セクションを追加するだけで、TiDB クラスターを起動できます。以下は例です。

```shell
cdc_servers:
  - host: 10.0.1.20
    gc-ttl: 86400
    data_dir: "/cdc-data"
  - host: 10.0.1.21
    gc-ttl: 86400
    data_dir: "/cdc-data"
```

詳細な手順については次を参照してください：

- 詳細な操作については、[初期化構成ファイルの編集](/production-deployment-using-tiup.md#step-3-initialize-cluster-topology-file)を参照してください。
- 詳細な構成可能なフィールドについては、[TiUP を使用して `cdc_servers` を構成](/tiup/tiup-cluster-topology-reference.md#cdc_servers)を参照してください。
- TiDB クラスターを展開するための詳細な手順については、[TiUP を使用して TiDB クラスターを展開](/production-deployment-using-tiup.md)を参照してください。

> **注意:**
>
> TiCDC をインストールする前に、TiUP 制御マシンと TiCDC ホスト間で [SSH 相互信頼およびパスワードなしの sudo を手動で構成](/check-before-deployment.md#manually-configure-the-ssh-mutual-trust-and-sudo-without-password)していることを確認してください。

## TiUP を使用して既存の TiDB クラスターに TiCDC を追加またはスケールアウトする

TiCDC クラスターのスケールアウト方法は、展開と類似しています。スケールアウトには TiUP を使用することをお勧めします。

1. `scale-out.yml` ファイルを作成し、TiCDC ノード情報を追加します。以下は例です。

    ```shell
    cdc_servers:
      - host: 10.1.1.1
        gc-ttl: 86400
        data_dir: /tidb-data/cdc-8300
      - host: 10.1.1.2
        gc-ttl: 86400
        data_dir: /tidb-data/cdc-8300
      - host: 10.0.1.4
        gc-ttl: 86400
        data_dir: /tidb-data/cdc-8300
    ```

2. TiUP 制御マシンでスケールアウトコマンドを実行します：

    ```shell
    tiup cluster scale-out <cluster-name> scale-out.yml
    ```

さらなる使用例については、[TiCDC クラスターのスケールアウト](/scale-tidb-using-tiup.md#scale-out-a-ticdc-cluster)を参照してください。

## TiUP を使用して既存の TiDB クラスターから TiCDC を削除またはスケールインする

TiCDC ノードのスケールインには TiUP を使用することをお勧めします。以下はスケールインコマンドです：

```shell
tiup cluster scale-in <cluster-name> --node 10.0.1.4:8300
```

さらなる使用例については、[TiCDC クラスターのスケールイン](/scale-tidb-using-tiup.md#scale-in-a-ticdc-cluster)を参照してください。

## TiUP を使用して TiCDC をアップグレードする

TiUP を使用して TiDB クラスターをアップグレードし、その過程で TiCDC もアップグレードすることができます。アップグレードコマンドを実行した後、TiUP は自動的に TiCDC コンポーネントをアップグレードします。以下は例です：

```shell
tiup update --self && \
tiup update --all && \
tiup cluster upgrade <cluster-name> <version> --transfer-timeout 600
```

> **注意:**
>
> 上記のコマンドで、`<cluster-name>` および `<version>` を実際のクラスター名とクラスターバージョンに置き換える必要があります。たとえば、バージョンは v7.4.0 にすることができます。

### アップグレードの注意事項

TiCDC クラスターをアップグレードする際には、以下に注意する必要があります：

- TiCDC v4.0.2 では `changefeed` を再構成しました。詳細については、[構成ファイルの互換性に関する注意事項](/ticdc/ticdc-compatibility.md#cli-and-configuration-file-compatibility)を参照してください。
- アップグレード中に問題が発生した場合は、解決策については [アップグレード FAQ](/upgrade-tidb-using-tiup.md#faq) を参照してください。
- v6.3.0 以降、TiCDC はローリングアップグレードをサポートしています。アップグレード中にレプリケーション遅延が安定し、著しく変動しません。ローリングアップグレードが自動的に有効になる条件は次のとおりです：

- TiCDC が v6.3.0 以降のバージョンであること。
    - TiUP が v1.11.3 以降のバージョンであること。
    - クラスター内で少なくとも 2 つの TiCDC インスタンスが実行されていること。

## TiUP を使用して TiCDC クラスターの構成を変更する

このセクションでは、[`tiup cluster edit-config`](/tiup/tiup-component-cluster-edit-config.md) コマンドを使用して TiCDC の構成を変更する手順について説明します。以下の例では、`gc-ttl` のデフォルト値を `86400` から `172800`（48 時間）に変更する必要があると仮定しています。

1. `tiup cluster edit-config` コマンドを実行します。`<cluster-name>` は実際のクラスター名に置き換えてください：

    ```shell
    tiup cluster edit-config <cluster-name>
    ```

2. vi エディタで `cdc` の [`server-configs`](/tiup/tiup-cluster-topology-reference.md#server_configs) を変更します：

    ```shell
    server_configs:
      tidb: {}
      tikv: {}
      pd: {}
      tiflash: {}
      tiflash-learner: {}
      pump: {}
      drainer: {}
      cdc:
        gc-ttl: 172800
    ```

    上記のコマンドで、`gc-ttl` は 48 時間に設定されています。

3. `tiup cluster reload -R cdc` コマンドを実行して構成をリロードします。

## TiUP を使用して TiCDC を停止および開始する

TiUP を使用して簡単に TiCDC ノードを停止および開始できます。コマンドは次のとおりです：

- TiCDC の停止: `tiup cluster stop -R cdc`
- TiCDC の開始: `tiup cluster start -R cdc`
- TiCDC の再起動: `tiup cluster restart -R cdc`

## TiCDC の TLS を有効化する

[TiDB コンポーネント間の TLS を有効化する](/enable-tls-between-components.md)を参照してください。

## コマンドラインツールを使用して TiCDC のステータスを表示する

以下のコマンドを実行して TiCDC クラスターのステータスを表示します。`v<CLUSTER_VERSION>` は TiCDC クラスターのバージョン（たとえば `v7.4.0` など）に置き換える必要があります：

```shell
tiup ctl:v<CLUSTER_VERSION> cdc capture list --server=http://10.0.10.25:8300
```

```shell
[
  {
    "id": "806e3a1b-0e31-477f-9dd6-f3f2c570abdd",
    "is-owner": true,
    "address": "127.0.0.1:8300",
    "cluster-id": "default"
  },
  {
    "id": "ea2a4203-56fe-43a6-b442-7b295f458ebc",
    "is-owner": false,
    "address": "127.0.0.1:8301",
    "cluster-id": "default"
  }
]
```

- `id`: サービスプロセスの ID を示します。
- `is-owner`: サービスプロセスが所有ノードかどうかを示します。
- `address`: サービスプロセスが外部にインターフェースを提供するアドレスを示します。
- `cluster-id`: TiCDC クラスターの ID を示します。デフォルト値は `default` です。