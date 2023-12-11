---
title: tiup dm deploy
---

# tiup dm deploy

`tiup dm deploy`コマンドは、新しいクラスタを展開するために使用されます。

## 構文

```shell
tiup dm deploy <cluster-name> <version> <topology.yaml> [flags]
```

- `<cluster-name>`: 新しいクラスタの名前で、既存のクラスタ名と同じにすることはできません。
- `<version>`: 展開するDMクラスタのバージョン番号。例: `v2.0.0`。
- `<topology.yaml>`: 準備された[topology file](/tiup/tiup-dm-topology-reference.md)。

## Options

### -u, --user

- ターゲットマシンに接続するために使用するユーザー名を指定します。このユーザーはターゲットマシンでシークレットフリーのsudoルート権限を持っている必要があります。
- データ型: `STRING`
- デフォルト: コマンドを実行する現在のユーザー。

### -i, --identity_file

- ターゲットマシンに接続するために使用するキーファイルを指定します。
- データ型: `STRING`
- デフォルト: `~/.ssh/id_rsa`

### -p, --password

- ターゲットマシンに接続するために使用するパスワードを指定します。このオプションと`-i/--identity_file`は同時に使用しないでください。
- データ型: `BOOLEAN`
- デフォルト: false

### -h, --help

- ヘルプ情報を出力します。
- データ型: `BOOLEAN`
- デフォルト: false

## Output

展開ログ。

[<< 前のページに戻る - TiUP DMコマンドリスト](/tiup/tiup-component-dm.md#command-list)