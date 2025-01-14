---
title: tiup クラスタのスケールアウト
---

# tiup クラスタのスケールアウト

`tiup cluster scale-out` コマンドは、クラスタのスケールアウトに使用されます。 クラスタのスケールアウトの内部ロジックは、クラスタのデプロイと類似しています。 tiup-cluster コンポーネントはまず新しいノードとのSSH接続を確立し、ターゲットノード上に必要なディレクトリを作成し、その後デプロイを実行し、サービスを開始します。

PDをスケールアウトすると、新しいPDノードがクラスタに追加され、サービスに関連付けられたPDの構成が更新されます。他のサービスは直接開始され、クラスタに追加されます。

## 構文

```shell
tiup cluster scale-out <cluster-name> <topology.yaml> [flags]
```

`<cluster-name>`: 操作対象のクラスタの名前。クラスタ名を忘れた場合は、[`cluster list`](/tiup/tiup-component-dm-list.md) コマンドで確認できます。

`<topology.yaml>`: 準備された [トポロジファイル](/tiup/tiup-dm-topology-reference.md)。このトポロジファイルには、現在のクラスタに追加する新しいノードのみを含める必要があります。

## オプション

### -u, --user

- ターゲットマシンに接続するために使用されるユーザー名を指定します。 このユーザーは、ターゲットマシンで秘密フリーのsudo root権限を持っている必要があります。
- データタイプ: `STRING`
- デフォルト: コマンドを実行する現在のユーザー

### -i, --identity_file

- ターゲットマシンに接続するために使用するキーファイルを指定します。
- データタイプ: `STRING`
- このオプションがコマンドで指定されていない場合、デフォルトで `~/.ssh/id_rsa` ファイルが使用されます。

### -p, --password

- ターゲットマシンに接続するために使用するパスワードを指定します。 このオプションと `-i/--identity_file` を同時に使用しないでください。
- データタイプ: `BOOLEAN`
- デフォルト: false

### --no-labels

- このオプションは、ラベルチェックをスキップするために使用されます。
- 1つの物理マシンに複数のTiKVノードが展開されると、PDはクラスタのトポロジを知らないため、リージョンの複数のレプリカを同じ物理マシンの異なるTiKVノードにスケジュールする可能性があります。したがって、この物理マシンを単一障害点とすることがあります。 このリスクを避けるために、ラベルを使用して、PDに同じリージョンを同じマシンにスケジュールしないように指示できます。 ラベルの構成については、[トポロジラベルによるレプリカのスケジュール](/schedule-replicas-by-topology-labels.md) を参照してください。
- テスト環境では、このリスクは問題ではないかもしれません。 この場合は、`--no-labels` を使用してチェックをスキップできます。
- データタイプ: `BOOLEAN`
- デフォルト: false

### --skip-create-user

- クラスタデプロイ中、tiup-cluster は、トポロジファイルで指定されたユーザー名の存在をチェックし、存在しない場合は作成します。 このチェックをスキップするには、`--skip-create-user` オプションを使用できます。
- データタイプ: `BOOLEAN`
- デフォルト: false

### -h, --help

- ヘルプ情報を表示します。
- データタイプ: `BOOLEAN`
- デフォルト: false

## 出力

スケールアウトのログ。

[<< 前のページに戻る - TiUP クラスタコマンドリスト](/tiup/tiup-component-cluster.md#command-list)