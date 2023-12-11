# TiUP リファレンス

TiUPはTiDBエコシステムのパッケージマネージャとして機能します。TiDB、PD、TiKVなどのTiDBエコシステムのコンポーネントを管理します。

## 構文

```shell
tiup [flags] <command> [args...]        # コマンドを実行します
# または
tiup [flags] <component> [args...]      # コンポーネントを実行します
```

`--help` コマンドを使用して、特定のコマンドの情報を取得できます。各コマンドの概要には、そのパラメータとその使用方法が表示されます。必須のパラメータは角かっこで表示され、オプションのパラメータは角かっこで表示されます。

`<command>` はコマンドの名前を表します。対応するコマンドの一覧については、以下の[コマンド一覧](#command-list)を参照してください。`<component>` はコンポーネントの名前を表します。対応するコンポーネントの一覧については、以下の[コンポーネント一覧](#component-list)を参照してください。

## オプション

### --binary

- このオプションを有効にすると、指定されたバイナリファイルのパスが表示されます。
    - `tiup --binary <component>` を実行すると、最新の安定したインストール済み `<component>` コンポーネントのパスが表示されます。 `<component>` がインストールされていない場合、エラーが返されます。
    - `tiup --binary <component>:<version>` を実行すると、インストール済みの `<component>` コンポーネントの `<version>` のパスが表示されます。 この `<version>` が表示されない場合、エラーが返されます。

- データ型: `BOOLEAN`
- このオプションはデフォルトで無効になっており、そのデフォルト値は `false` です。このオプションを有効にするには、コマンドにこのオプションを追加し、`true` の値を渡すか、値を渡さないようにすることができます。

> **注意:**
>
> このオプションは、`tiup [flags] <component> [args...]` 形式のコマンドでのみ使用できます。

### --binpath

> **注意:**
>
> このオプションは、`tiup [flags] <component> [args...]` 形式のコマンドでのみ使用できます。

- 実行するコンポーネントのパスを指定します。コンポーネントを実行する際に、TiUPミラー内のバイナリファイルを使用したくない場合は、このオプションを追加して、カスタムパス内のバイナリファイルを使用するように指定することができます。
- データ型: `STRING`

### -T, --tag

- 開始するコンポーネントにタグを指定します。一部のコンポーネントは実行中にディスクストレージを使用する必要があり、TiUPはこの実行のために一時的なストレージディレクトリを割り当てます。TiUPに固定ディレクトリを割り当てる場合は、`-T/--tag` を使用してディレクトリの名前を指定し、同じタグで複数の実行で読み書きできるようにします。
- データ型: `STRING`

### -v, --version

TiUPのバージョンを表示します。

### --help

ヘルプ情報を表示します。

## コマンド一覧

TiUPには複数のコマンドがあり、これらのコマンドには複数のサブコマンドがあります。特定のコマンドとその詳細な説明については、以下のリスト内の対応するリンクをクリックしてください。

- [install](/tiup/tiup-command-install.md): コンポーネントをインストールします。
- [list](/tiup/tiup-command-list.md): コンポーネントの一覧を表示します。
- [uninstall](/tiup/tiup-command-uninstall.md): コンポーネントをアンインストールします。
- [update](/tiup/tiup-command-update.md): インストールされたコンポーネントを更新します。
- [status](/tiup/tiup-command-status.md): コンポーネントの実行状態を表示します。
- [clean](/tiup/tiup-command-clean.md): コンポーネントのデータディレクトリをクリアします。
- [mirror](/tiup/tiup-command-mirror.md): ミラーを管理します。
- [telemetry](/tiup/tiup-command-telemetry.md): テレメトリを有効または無効にします。
- [completion](/tiup/tiup-command-completion.md): TiUPコマンドを補完します。
- [env](/tiup/tiup-command-env.md): TiUP関連の環境変数を表示します。
- [help](/tiup/tiup-command-help.md): コマンドまたはコンポーネントのヘルプ情報を表示します。

## コンポーネント一覧

- [cluster](/tiup/tiup-component-cluster.md): 本番環境でTiDBクラスタを管理します。
- [dm](/tiup/tiup-component-dm.md): 本番環境でTiDBデータマイグレーション（DM）クラスタを管理します。