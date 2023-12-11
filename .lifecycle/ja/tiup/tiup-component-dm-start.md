---
title: tiup dm start
---

# tiup dm start

`tiup dm start`コマンドは、指定されたクラスタのすべてまたは一部のサービスを開始するために使用されます。

## 構文

```shell
tiup dm start <cluster-name> [flags]
```

`<cluster-name>`: 操作するクラスタの名前。クラスタ名を忘れた場合は、[cluster list](/tiup/tiup-component-dm-list.md)コマンドで確認できます。

## オプション

### -N, --node

- 開始するノードを指定します。指定しない場合、すべてのノードが開始されます。このオプションの値は、ノードIDのコンマ区切りリストです。ノードIDは、[`tiup dm display`](/tiup/tiup-component-dm-display.md)コマンドのクラスタステータステーブルの最初の列から取得できます。
- データ型: `STRINGS`
- このオプションがコマンドで指定されていない場合、すべてのノードが開始されます。

> **注意:**
>
> 同時に`-R, --role`オプションを指定した場合、`-N, --node`および`-R, --role`の両方の仕様に一致するサービスノードのみが開始されます。

### -R, --role

- 開始する役割を指定します。指定しない場合、すべての役割が開始されます。このオプションの値は、ノードの役割のコンマ区切りリストです。ノードの役割は、[`tiup dm display`](/tiup/tiup-component-dm-display.md)コマンドによって返されるクラスタステータステーブルの2番目の列から取得できます。
- データ型: `STRINGS`
- このオプションがコマンドで指定されていない場合、すべての役割が開始されます。

> **注意:**
>
> 同時に`-N, --node`オプションを指定した場合、`-N, --node`および`-R, --role`の両方の仕様に一致するサービスノードのみが開始されます。

### -h, --help

- ヘルプ情報を表示します。
- データ型: `BOOLEAN`
- デフォルト: false

## 出力

サービスの開始ログ。

[<< 前のページに戻る - TiUP DMコマンドリスト](/tiup/tiup-component-dm.md#command-list)