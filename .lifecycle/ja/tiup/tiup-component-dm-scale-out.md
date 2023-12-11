---
title: tiup dmスケールアウト
---

# tiup dmスケールアウト

`tiup dm scale-out`コマンドは、クラスターのスケーリングアウトに使用されます。クラスターのスケーリングアウトの内部ロジックは、クラスターのデプロイメントと類似しています。`tiup-dm`コンポーネントはまず新しいノードにSSH接続を確立し、ターゲットノード上に必要なディレクトリを作成し、その後デプロイメントを実行してサービスを開始します。

## 構文

```shell
tiup dm scale-out <cluster-name> <topology.yaml> [flags]
```

`<cluster-name>`: 操作するクラスターの名前。クラスター名を忘れた場合は、[cluster list](/tiup/tiup-component-dm-list.md)コマンドで確認できます。

`<topology.yaml>`: 準備が整った[topology file](/tiup/tiup-dm-topology-reference.md)です。このトポロジーファイルには、現在のクラスターに追加される新しいノードのみを含める必要があります。

## オプション

### -u, --user

- ターゲットマシンへの接続に使用するユーザー名を指定します。このユーザーはターゲットマシンで秘密フリーのsudoルート権限を持っている必要があります。
- データ型： `STRING`
- デフォルト：コマンドを実行する現在のユーザー

### -i, --identity_file

- ターゲットマシンに接続するために使用するキーファイルを指定します。
- データ型： `STRING`
- このオプションがコマンドで指定されていない場合、デフォルトで `~/.ssh/id_rsa` ファイルが使用されてターゲットマシンに接続されます。

### -p, --password

- ターゲットマシンに接続するために使用するパスワードを指定します。このオプションと `-i/--identity_file` を同時に使用しないでください。
- データ型： `BOOLEAN`
- デフォルト：false

### -h, --help

- ヘルプ情報を出力します。
- データ型： `BOOLEAN`
- デフォルト：false

## 出力

スケールアウトのログ。

[<< 前のページに戻る - TiUP DMコマンドリスト](/tiup/tiup-component-dm.md#command-list)