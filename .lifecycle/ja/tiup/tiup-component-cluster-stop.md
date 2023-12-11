---
title: tiup cluster stop
---

# tiup cluster stop

`tiup cluster stop`コマンドは、指定されたクラスタのすべてまたは一部のサービスを停止するために使用されます。

> **ノート:**
>
> クラスタのコアサービスが停止されると、そのクラスタはもはやサービスを提供できません。

## 構文

```shell
tiup cluster stop <cluster-name> [flags]
```

`<cluster-name>`は操作対象のクラスタの名前です。クラスタ名を忘れた場合は、[`tiup cluster list`](/tiup/tiup-component-cluster-list.md)コマンドを使用して確認できます。

## Options

### -N, --node

- 停止するノードを指定します。このオプションの値は、ノードIDのカンマ区切りのリストです。`tiup cluster display`コマンドによって返される[クラスタステータステーブル](/tiup/tiup-component-cluster-display.md)の最初の列からノードIDを取得できます。
- データタイプ: `STRINGS`
- このオプションがコマンドで指定されていない場合、デフォルトですべてのノードが停止されます。

> **ノート:**
>
> `-R, --role`オプションも同時に指定されている場合、`-N, --node`および`-R, --role`の両方の仕様に一致するサービスノードのみが停止されます。

### -R, --role

- 停止するノードの役割を指定します。このオプションの値は、ノードの役割のカンマ区切りのリストです。`tiup cluster display`コマンドによって返される[クラスタステータステーブル](/tiup/tiup-component-cluster-display.md)の2番目の列からノードの役割を取得できます。
- データタイプ: `STRINGS`
- このオプションがコマンドで指定されていない場合、デフォルトですべての役割が停止されます。

> **ノート:**
>
> `-N, --node`オプションも同時に指定されている場合、`-N, --node`および`-R, --role`の両方の仕様に一致するサービスノードのみが停止されます。

### -h, --help

- ヘルプ情報を表示します。
- データタイプ: `BOOLEAN`
- このオプションはデフォルトで`false`値で無効になっています。このオプションを有効にするには、コマンドにこのオプションを追加し、`true`値を渡すか、値を渡さないようにします。

## 出力

サービスを停止した際のログ。

[<< 前のページに戻る - TiUP Clusterコマンドリスト](/tiup/tiup-component-cluster.md#command-list)