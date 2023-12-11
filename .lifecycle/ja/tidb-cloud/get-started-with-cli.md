---
title: TiDB Cloud CLI クイックスタート
summary: TiDB Cloud CLI を使用して TiDB Cloud リソースを管理する方法を学びます。
---

# TiDB Cloud CLI クイックスタート

TiDB Cloud では、コマンドラインインターフェース（CLI）[`ticloud`](https://github.com/tidbcloud/tidbcloud-cli) を使用して、わずか数行のコマンドでターミナルから TiDB Cloud とやり取りすることができます。例えば、`ticloud` を使用して次の操作を簡単に実行できます。

- クラスタの作成、削除、および一覧表示。
- Amazon S3 やローカルファイルからデータをクラスタにインポート。

## 開始する前に

- TiDB Cloud アカウントを取得してください。アカウントをお持ちでない場合は、[無料トライアルの申し込み](https://tidbcloud.com/free-trial)を行ってください。
- [TiDB Cloud API キーの作成](https://docs.pingcap.com/tidbcloud/api/v1beta#section/Authentication/API-Key-Management)。

## インストール

<SimpleTab>
<div label="macOS/Linux">

macOS や Linux の場合、次のいずれかの方法で `ticloud` をインストールできます。

- スクリプトを使用してインストールする（推奨）

    ```shell
    curl https://raw.githubusercontent.com/tidbcloud/tidbcloud-cli/main/install.sh | sh
    ```

- [TiUP](https://tiup.io/) を使用してインストールする

    ```shell
    tiup install cloud
    ```

- 手動でインストールする

    [リリース](https://github.com/tidbcloud/tidbcloud-cli/releases/latest) ページから事前にコンパイルされたバイナリをダウンロードし、インストールする場所にコピーします。

- GitHub Actions でインストールする

    GitHub Action で `ticloud` を設定するには、[`setup-tidbcloud-cli`](https://github.com/tidbcloud/setup-tidbcloud-cli) を使用してください。

パッケージマネージャを使用して MySQL コマンドラインクライアントをインストールします（インストールしていない場合）。

- Debian ベースのディストリビューション:

    ```shell
    sudo apt-get install mysql-client
    ```

- RPM ベースのディストリビューション:

    ```shell
    sudo yum install mysql
    ```

- macOS:

  ```shell
  brew install mysql-client
  ```

</div>

<div label="Windows">

Windows の場合、次のいずれかの方法で `ticloud` をインストールできます。

- 手動でインストールする

    [リリース](https://github.com/tidbcloud/tidbcloud-cli/releases/latest) ページから事前にコンパイルされたバイナリをダウンロードし、インストールする場所にコピーします。

- GitHub Actions でインストールする

    GitHub Actions で `ticloud` を設定するには、[`setup-tidbcloud-cli`](https://github.com/tidbcloud/setup-tidbcloud-cli) を使用してください。

パッケージマネージャを使用して MySQL コマンドラインクライアントをインストールします。インストール方法については、[MySQL インストーラー for Windows](https://dev.mysql.com/doc/refman/8.0/en/mysql-installer.html) の手順を参照してください。Windows で `ticloud connect` を起動する際には、PATH 環境変数に `mysql.exe` を含むディレクトリが必要です。

</div>
</SimpleTab>

## TiDB Cloud CLI の使用

利用可能なすべてのコマンドを表示します。

```shell
ticloud --help
```

最新バージョンを使用していることを確認します。

```shell
ticloud version
```

最新バージョンを使用していない場合は、最新バージョンに更新します。

```shell
ticloud update
```

### TiUP を使用した TiDB Cloud CLI の使用

TiDB Cloud CLI は [TiUP](https://tiup.io/) でも利用できます。コンポーネント名は `cloud` です。

利用可能なすべてのコマンドを表示します。

```shell
tiup cloud --help
```

`tiup cloud <command>` を使用してコマンドを実行します。例:

```shell
tiup cloud cluster create
```

TiUP で最新バージョンに更新します。

```shell
tiup update cloud
```

## クイックスタート

[TiDB サーバーレス](/tidb-cloud/select-cluster-tier.md#tidb-serverless) は TiDB Cloud を使用して始める最適な方法です。このセクションでは、TiDB Cloud CLI を使用して TiDB サーバーレスクラスタを作成する方法について説明します。

### ユーザープロファイルを作成する

クラスタを作成する前に、TiDB Cloud API キーを使用してユーザープロファイルを作成する必要があります。

```shell
ticloud config create
```

> **注意:**
>
> プロファイル名には `.` を含めないでください。

### TiDB サーバーレスクラスタを作成する

TiDB サーバーレスクラスタを作成するには、次のコマンドを入力し、その後 CLI のプロンプトに従って必要な情報を提供し、パスワードを設定してください。

```shell
ticloud cluster create
```

### クラスタに接続する

クラスタが作成されたら、クラスタに接続できます。

```shell
ticloud connect
```

デフォルトのユーザーを使用するかどうかのプロンプトが表示された場合は、`Y` を選択し、クラスタ作成時に設定したパスワードを入力してください。

## 次のステップ

TiDB Cloud CLI のその他の機能を見るには、[CLI リファレンス](/tidb-cloud/cli-reference.md) を参照してください。

## フィードバック

TiDB Cloud CLI に関するご質問や提案がある場合は、[issue](https://github.com/tidbcloud/tidbcloud-cli/issues/new/choose) を作成してください。また、どんな貢献も歓迎します。