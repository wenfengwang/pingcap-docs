---
title: tiup dm reload
---

# tiup dm再読み込み

[クラスタ構成を変更](/tiup/tiup-component-dm-edit-config.md)した後は、`tiup dm reload`コマンドを使用してクラスタを再読み込みし、構成が有効になるようにする必要があります。このコマンドは、制御マシンの構成をサービスが実行されているリモートマシンに公開し、アップグレードプロセスに従ってサービスを再起動します。再起動プロセス中もクラスタは利用可能です。

## 構文

```shell
tiup dm reload <cluster-name> [flags]
```

`<cluster-name>`: 操作対象のクラスタ名

## オプション

### -N, --node

- 再起動するノードを指定します。指定しない場合、すべてのノードが再起動されます。このオプションの値は、ノードのIDのコンマ区切りのリストです。このオプションを使用すると、[`tiup dm display`](/tiup/tiup-component-dm-display.md)コマンドによって返されるクラスタステータステーブルの最初の列からノードのIDを取得できます。
- データ型： `STRINGS`
- このオプションがコマンドで指定されていない場合、すべてのノードがデフォルトで選択されます。

> **注意:**
>
> + 同時に`-R, --role`オプションを指定した場合、指定された`-N, --node`と`-R, --role`の両方に一致するサービスノードのみが再起動されます。
> + `--skip-restart`オプションが指定されている場合、`-N, --node`オプションは無効です。

### -R, --role

- 再起動する役割を指定します。指定しない場合、すべての役割が再起動されます。このオプションの値は、ノードの役割のコンマ区切りのリストです。このオプションを使用すると、[`tiup dm display`](/tiup/tiup-component-dm-display.md)コマンドによって返されるクラスタステータステーブルの2番目の列からノードの役割を取得できます。
- データ型： `STRINGS`
- このオプションがコマンドで指定されていない場合、すべての役割がデフォルトで選択されます。

> **注意:**
>
> + 同時に`-N, --node`オプションを指定した場合、指定された`-N, --node`と`-R, --role`の両方に一致するサービスノードのみが再起動されます。
> + `--skip-restart`オプションが指定されている場合、`-R, --role`オプションは無効です。

### --skip-restart

`tiup dm reload`コマンドは2つの操作を実行します：

- すべてのノード構成を更新する
- 指定されたノードを再起動する

`--skip-restart`オプションを指定した後は、構成を再読み込みしただけでノードを再起動せず、更新された構成が対応するサービスの次の再起動まで適用されず、効果がありません。

- データ型： `BOOLEAN`
- デフォルト： false

### -h, --help

- ヘルプ情報を表示します。
- データ型： `BOOLEAN`
- デフォルト： false

## 出力

tiup-dmの実行ログ。

[<< 前のページに戻る - TiUP DMコマンドリスト](/tiup/tiup-component-dm.md#command-list)