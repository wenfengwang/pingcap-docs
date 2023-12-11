---
title: ローカル TiDB クラスタの迅速なデプロイ
summary: TiUP の playground コンポーネントを使用してローカル TiDB クラスタを迅速にデプロイする方法を学びます。
aliases: ['/docs/dev/tiup/tiup-playground/','/docs/dev/reference/tools/tiup/playground/']
---

# ローカル TiDB クラスタの迅速なデプロイ

TiDB クラスタは複数のコンポーネントで構成された分散システムです。典型的な TiDB クラスタは、少なくとも 3 つの PD ノード、3 つの TiKV ノード、および 2 つの TiDB ノードから構成されています。TiDB を素早く体験したい場合、これらの多くのコンポーネントを手動でデプロイすることは時間がかかり、複雑になるかもしれません。このドキュメントでは、TiUP の playground コンポーネントとその使用方法について紹介します。

## TiUP playground の概要

playground コンポーネントの基本的な使用法は以下の通りです:

```bash
tiup playground ${version} [flags]
```

`tiup playground` コマンドを直接実行すると、TiUP はローカルにインストールされた TiDB、TiKV、および PD コンポーネントを使用するか、これらのコンポーネントの安定版をインストールして、1 つの TiKV インスタンス、1 つの TiDB インスタンス、1 つの PD インスタンス、および 1 つの TiFlash インスタンスから構成される TiDB クラスタを起動します。

このコマンドは実際に以下の操作を行います:

- このコマンドは playground コンポーネントのバージョンを指定していないため、TiUP はまずインストールされた playground コンポーネントの最新バージョンを確認します。最新バージョンが v1.12.3 であると仮定すると、このコマンドは `tiup playground:v1.12.3` と同じように動作します。
- TiUP playground で TiDB、TiKV、および PD コンポーネントをインストールしていない場合、playground コンポーネントはこれらのコンポーネントの最新安定版をインストールし、その後それらのインスタンスを起動します。
- このコマンドは TiDB、PD、および TiKV コンポーネントのバージョンを指定していないため、TiUP playground はデフォルトでそれぞれのコンポーネントの最新バージョンを使用します。最新バージョンが v7.4.0 であると仮定すると、このコマンドは `tiup playground:v1.12.3 v7.4.0` と同じように動作します。
- このコマンドは各コンポーネントの数を指定していないため、TiUP playground はデフォルトで 1 つの TiDB インスタンス、1 つの TiKV インスタンス、1 つの PD インスタンス、および 1 つの TiFlash インスタンスから構成される最小のクラスタを起動します。
- 各 TiDB コンポーネントの起動後、TiUP playground はクラスタが正常に起動したことを通知し、MySQL クライアントを介して TiDB クラスタに接続する方法や [TiDB Dashboard](/dashboard/dashboard-intro.md) へのアクセス方法など、有用な情報を提供します。

playground コンポーネントのコマンドラインフラグは以下の通りです:

```bash
Flags:
      --db int                   TiDB インスタンスの数を指定します (デフォルト: 1)
      --db.host host             TiDB のリスニングアドレスを指定します
      --db.port int              TiDB のポートを指定します
      --db.binpath string        TiDB インスタンスのバイナリパスを指定します（オプション、デバッグ用）
      --db.config string         TiDB インスタンスの構成ファイルを指定します（オプション、デバッグ用）
      --db.timeout int           起動のための TiDB の最大待ち時間を秒単位で指定します。0 は制限なしを表します。
      --drainer int              クラスタの Drainer データを指定します
      --drainer.binpath string   Drainer バイナリファイルの場所を指定します（オプション、デバッグ用）
      --drainer.config string    Drainer の構成ファイルを指定します
  -h, --help                     tiup のヘルプを表示します
      --host string              各コンポーネントのリスニングアドレスを指定します（デフォルト: `127.0.0.1`）。他のマシンからアクセスする場合は `0.0.0.0` に設定します
      --kv int                   TiKV インスタンスの数を指定します (デフォルト: 1)
      --kv.binpath string        TiKV インスタンスのバイナリパスを指定します（オプション、デバッグ用）
      --kv.config string         TiKV インスタンスの構成ファイルを指定します（オプション、デバッグ用）
      --mode string              playground モードを指定します: 'tidb'（デフォルト） および 'tikv-slim'
      --pd int                   PD インスタンスの数を指定します (デフォルト: 1)
      --pd.host host             PD のリスニングアドレスを指定します
      --pd.binpath string        PD インスタンスのバイナリパスを指定します（オプション、デバッグ用）
      --pd.config string         PD インスタンスの構成ファイルを指定します（オプション、デバッグ用）
      --pump int                 Pump インスタンスの数を指定します。値が `0` でない場合、TiDB Binlog が有効になります。
      --pump.binpath string      Pump バイナリファイルの場所を指定します（オプション、デバッグ用）
      --pump.config string       Pump の構成ファイルを指定します（オプション、デバッグ用）
      -T, --tag string           playground のタグを指定します
      --ticdc int                TiCDC インスタンスの数を指定します (デフォルト: 0)
      --ticdc.binpath string     TiCDC インスタンスのバイナリパスを指定します（オプション、デバッグ用）
      --ticdc.config string      TiCDC インスタンスの構成ファイルを指定します（オプション、デバッグ用）
      --tiflash int              TiFlash インスタンスの数を指定します (デフォルト: 1)
      --tiflash.binpath string   TiFlash インスタンスのバイナリパスを指定します（オプション、デバッグ用）
      --tiflash.config string    TiFlash インスタンスの構成ファイルを指定します（オプション、デバッグ用）
      --tiflash.timeout int      起動のための TiFlash の最大待ち時間を秒単位で指定します。0 は制限なしを表します
      -v, --version              playground のバージョンを指定します
      --without-monitor          Prometheus および Grafana の監視機能を無効にします。このフラグを追加しない場合、監視機能はデフォルトで有効になります
```

## 例

### 利用可能な TiDB バージョンを確認

```shell
tiup list tidb
```

### 特定のバージョンの TiDB クラスタを起動

```shell
tiup playground ${version}
```

`${version}` にターゲットのバージョン番号を置き換えてください。

### 夜間バージョンの TiDB クラスタを起動

```shell
tiup playground nightly
```

上記のコマンドで `nightly` は TiDB の最新の開発バージョンを示します。

### PD のデフォルト構成を上書き

まず、[PD 構成テンプレート](https://github.com/pingcap/pd/blob/master/conf/config.toml)をコピーする必要があります。コピーしたファイルを `~/config/pd.toml` に配置し、必要に応じていくつかの変更を加えた後、以下のコマンドを実行して PD のデフォルト構成を上書きできます:

```shell
tiup playground --pd.config ~/config/pd.toml
```

### デフォルトのバイナリファイルを置き換える

デフォルトでは、playground を起動すると、各コンポーネントは公式ミラーからのバイナリファイルを使用して起動されます。テスト用に一時的にローカルでコンパイルしたバイナリファイルをクラスタに配置したい場合は、置換用の `--{comp}.binpath` フラグを使用できます。たとえば、TiDB のバイナリファイルを置き換えるには、以下のコマンドを実行します:

```shell
tiup playground --db.binpath /xx/tidb-server
```

### 複数のコンポーネントインスタンスを起動

デフォルトでは、TiDB、TiKV、および PD 各コンポーネントについて、1 つのインスタンスしか起動されません。各コンポーネントの複数のインスタンスを起動するには、次のフラグを追加してください:

```shell
tiup playground --db 3 --pd 3 --kv 3
```

### TiDB クラスタの起動時にタグを指定

TiUP playground を使用して起動した TiDB クラスタを停止すると、すべてのクラスタデータがクリアされます。TiUP playground を使用して TiDB クラスタを起動し、自動的にクラスタデータがクリアされないようにするには、クラスタを起動する際にタグを指定できます。タグを指定すると、クラスタデータは `~/.tiup/data` ディレクトリに保持されます。次のコマンドを実行してタグを指定します:

```shell
tiup playground --tag <tagname>
```

この方法で起動したクラスタでは、クラスタが停止された後もデータファイルが保持されます。このタグを使用して、クラスタが停止されてからのデータを使用できるため、次回クラスタを起動する際にこのタグを使用できます。

## playground で起動した TiDB クラスタに迅速に接続

TiUP には `client` コンポーネントがあり、これを使用して playground で起動したローカル TiDB クラスタを自動的に検出して接続できます。使用法は以下の通りです:

```shell
tiup client
```

このコマンドは、現在のマシン上で playground によって起動された TiDB クラスタの一覧をコンソールに表示します。接続する TiDB クラスタを選択し、<kbd>Enter</kbd> キーをクリックすると、組込みの MySQL クライアントが開き、TiDB に接続できます。

## 起動したクラスタの情報を表示

```shell
tiup playground display
```

上記のコマンドは以下の結果を返します:

```
Pid    Role     Uptime
---    ----     ------
84518  pd       35m22.929404512s
84519  tikv     35m22.927757153s
84520  pump     35m22.92618275s
86189  tidb     exited
86526  tidb     34m28.293148663s
86190  drainer  35m19.91349249s
```

## クラスタをスケールアウト

クラスタをスケールアウトするためのコマンドラインパラメータは、クラスタを起動する際と類似しています。次のコマンドを実行して 2 つの TiDB インスタンスをスケールアウトできます:

```shell
tiup playground scale-out --db 2
```

## クラスタをスケールイン

```
You can specify a `pid` in the `tiup playground scale-in` command to scale in the corresponding instance. To view the `pid`, execute `tiup playground display`.

```shell
tiup playground scale-in --pid 86526
```