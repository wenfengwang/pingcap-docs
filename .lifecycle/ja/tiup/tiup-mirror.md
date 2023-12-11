---
title: プライベートミラーの作成
summary: プライベートミラーの作成方法を学びましょう。
aliases: ['/tidb/dev/tiup-mirrors','/docs/dev/tiup/tiup-mirrors/','/docs/dev/reference/tools/tiup/mirrors/']
---

# プライベートミラーの作成

通常、プライベートクラウドを作成する際には、公式の TiUP ミラーにアクセスできない隔離されたネットワーク環境が必要です。そのため、`mirror` コマンドによって主に実装されたプライベートミラーを作成することができます。また、`mirror` コマンドをオフライン展開にも使用できます。プライベートミラーには、自身で構築してパッケージ化したコンポーネントを使用することも可能です。

## TiUP `mirror` 概要

次のコマンドを実行して、`mirror` コマンドのヘルプ情報を取得します:

{{< copyable "shell-regular" >}}

```bash
tiup mirror --help
```

```bash
`mirror` コマンドは、TiUP のコンポーネントリポジトリを管理するために使用されます。これを使用して、プライベートリポジトリを作成したり、既存のリポジトリに新しいコンポーネントを追加したりすることができます。このリポジトリはオンラインまたはオフラインのどちらでも使用できます。また、コンポーネントやリポジトリ自体のキー、ユーザー、バージョンを管理するための有用なユーティリティも提供されています。

使用法:
  tiup mirror <command> [flags]

利用可能なコマンド:
  init        空のリポジトリを初期化します
  sign        マニフェストファイルに署名を追加します
  genkey      新しい鍵ペアを生成します
  clone       ローカルミラーをリモートミラーからクローンし、選択したすべてのコンポーネントをダウンロードします
  merge       2 つ以上のオフラインミラーをマージします
  publish     コンポーネントを公開します
  show        ミラーアドレスを表示します
  set         ミラーアドレスを設定します
  modify      公開されたコンポーネントを変更します
  renew       公開されたコンポーネントのマニフェストを更新します
  grant       新しいオーナーに権限を与えます
  rotate      root.json をローテートします

グローバルフラグ:
      --help                 このコマンドのヘルプ
```

## ミラーをクローンする

`tiup mirror clone` コマンドを実行して、ローカルミラーを構築できます:

{{< copyable "shell-regular" >}}

```bash
tiup mirror clone <target-dir> [global-version] [flags]
```

- `target-dir`: クローンしたデータが保存されるディレクトリを指定するために使用されます。
- `global-version`: すべてのコンポーネントに対して迅速にグローバルバージョンを設定するために使用されます。

`tiup mirror clone` コマンドには多くのオプションフラグが提供されています (将来的にはさらに多くのフラグが提供されるかもしれません)。これらのフラグは、意図した使用方法に応じて以下のカテゴリに分類されます:

- クローニング時にバージョンをプレフィックスマッチングするかどうかを決定します

    `--prefix` フラグが指定されている場合、クローン時にバージョン番号がプレフィックスと一致するようにマッチングされます。たとえば、"v5.0.0" を `--prefix` として指定した場合、"v5.0.0-rc" および "v5.0.0" が一致します。

- 完全なクローンを使用するかどうかを決定します

    `--full` フラグを指定すると、公式ミラーを完全にクローンすることができます。

    > **注意:**
    >
    > `--full`、`global-version` フラグ、およびコンポーネントのバージョンが指定されていない場合は、メタ情報のみがクローンされます。

- 特定のプラットフォームからパッケージをクローンするかどうかを決定します

    特定のプラットフォーム向けにパッケージのみをクローンする場合は、`-os` および `-arch` を使用してプラットフォームを指定します。たとえば:

    - `tiup mirror clone <target-dir> [global-version] --os=linux` コマンドを実行して、Linux 向けにクローンします。
    - `tiup mirror clone <target-dir> [global-version] --arch=amd64` コマンドを実行して、amd64 向けにクローンします。
    - `tiup mirror clone <target-dir> [global-version] --os=linux --arch=amd64` コマンドを実行して、Linux/amd64 向けにクローンします。

- パッケージの特定のバージョンをクローンするかどうかを決定します

    特定のバージョン (すべてのバージョンではない) のコンポーネントをクローンする場合は、`--<component>=<version>` を使用してこのバージョンを指定します。たとえば:

    - `tiup mirror clone <target-dir> --tidb v7.4.0` コマンドを実行して、TiDB コンポーネントの v7.4.0 バージョンをクローンします。
    - `tiup mirror clone <target-dir> --tidb v7.4.0 --tikv all` コマンドを実行して、TiDB コンポーネントの v7.4.0 バージョンとすべての TiKV コンポーネントのバージョンをクローンします。
    - `tiup mirror clone <target-dir> v7.4.0` コマンドを実行して、クラスタ内のすべてのコンポーネントの v7.4.0 バージョンをクローンします。

クローニング後、署名キーが自動的に設定されます。

### プライベートリポジトリを管理する

`tiup mirror clone` でクローンしたリポジトリは、SCP、NFS でファイルを共有したり、HTTP または HTTPS プロトコルを介してリポジトリを利用可能にすることで、ホスト間で共有できます。`tiup mirror set <location>` を使用してリポジトリの場所を指定します。

```bash
tiup mirror set /shared_data/tiup
```

```bash
tiup mirror set https://tiup-mirror.example.com/
```

> **注意:**
>
> `tiup mirror clone` を実行するマシンで `tiup mirror set...` を実行すると、次回 `tiup mirror clone...` を実行する際に、マシンはローカルミラーからクローンし、リモートではなくなります。そのため、プライベートミラーを更新する前に `tiup mirror set --reset` を実行して、ミラーをリセットする必要があります。

ミラーの別の利用方法として、`TIUP_MIRRORS` 環境変数を使用することができます。次に、プライベートリポジトリを使用して `tiup list` を実行する例を示します。

```bash
export TIUP_MIRRORS=/shared_data/tiup
tiup list
```

`TIUP_MIRRORS` 設定は、例えば `tiup mirror set` を使用してミラー設定を恒常的に変更することができます。詳細については、[tiup issue #651](https://github.com/pingcap/tiup/issues/651) を参照してください。

### プライベートリポジトリの更新

同じ `target-dir` で再び `tiup mirror clone` コマンドを実行すると、マシンは新しいマニフェストを作成し、利用可能なコンポーネントの最新バージョンをダウンロードします。

> **注意:**
>
> マニフェストを再作成する前に、すべてのコンポーネントおよびバージョン (以前にダウンロードされたものも含む) が含まれていることを確認してください。

## カスタムリポジトリ

自身で構築した TiDB、TiKV、PD などの TiDB コンポーネントを使用するために、カスタムリポジトリを作成することができます。また、独自の tiup コンポーネントを作成することも可能です。

独自のコンポーネントを作成するには、`tiup package` コマンドを実行し、[コンポーネントパッケージング](https://github.com/pingcap/tiup/blob/master/doc/user/package.md) で指示されているとおりに操作します。

### カスタムリポジトリの作成

`/data/mirror` で空のリポジトリを作成するには:

```bash
tiup mirror init /data/mirror
```

リポジトリを作成する際、キーは `/data/mirror/keys` に書き込まれます。

新しいプライベートキーを `~/.tiup/keys/private.json` に作成するには:

```bash
tiup mirror genkey
```

`jdoe` に `~/.tiup/keys/private.json` のプライベートキーを `/data/mirror` の所有権を付与するには:

```bash
tiup mirror set /data/mirror
tiup mirror grant jdoe
```

### カスタムコンポーネントの操作

1. hello というカスタムコンポーネントを作成します。

    ```bash
    $ cat > hello.c << END
    > #include <stdio.h>
    int main() {
      printf("hello\n");
      return (0);
    }
    END
    $ gcc hello.c -o hello
    $ tiup package hello --entry hello --name hello --release v0.0.1
    ```

    `package/hello-v0.0.1-linux-amd64.tar.gz` が作成されます。

2. リポジトリとプライベートキーを作成し、リポジトリへの所有権を付与します。

    ```bash
    $ tiup mirror init /tmp/m
    $ tiup mirror genkey
    $ tiup mirror set /tmp/m
    $ tiup mirror grant $USER
    ```

    ```bash
    tiup mirror publish hello v0.0.1 package/hello-v0.0.1-linux-amd64.tar.gz hello
    ```

3. コンポーネントを実行します。まだインストールされていない場合は、まずダウンロードされます。

    ```bash
    $ tiup hello
    ```

    ```
    コンポーネント `hello` のバージョンがインストールされていません。リポジトリからダウンロードします。
    コンポーネント `hello` を開始中: /home/dvaneeden/.tiup/components/hello/v0.0.1/hello
    hello
    ```

    `tiup mirror merge` を使用すると、カスタムコンポーネントが含まれるリポジトリを別のリポジトリにマージできます。これは、すべてのコンポーネントが `/data/my_custom_components` で現在の `$USER` によって署名されていると想定しています。

    ```bash
    $ tiup mirror set /data/my_mirror
    $ tiup mirror grant $USER
    $ tiup mirror merge /data/my_custom_components
    ```