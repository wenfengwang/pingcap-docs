---
title: tiup cluster start
---

# tiup cluster start

`tiup cluster start`コマンドは、指定したクラスタのすべてのサービスまたは一部のサービスを起動するために使用されます。

## 構文

```shell
tiup cluster start <cluster-name> [flags]
```

`<cluster-name>`は操作するクラスタの名前です。クラスタ名を忘れた場合は、[`tiup cluster list`](/tiup/tiup-component-cluster-list.md)コマンドを使用して確認できます。

## オプション

### --init

クラスタを安全な方法で開始します。クラスタを初めて起動する場合には、このオプションを使用することをお勧めします。この方法は、起動時にTiDBルートユーザーのパスワードを生成し、コマンドラインインターフェイスでパスワードを返します。

> **注意:**
>
> - TiDBクラスタを安全に開始した後は、パスワードなしでルートユーザーを使用してデータベースにログインすることはできません。そのため、コマンドラインで返されたパスワードを記録しておく必要があります。
> - パスワードは1度だけ生成されます。パスワードを記録しないか忘れた場合は、[ルートパスワードを忘れた場合](/user-account-management.md#forget-the-root-password)を参照してパスワードを変更してください。

### -N, --node

- 開始するノードを指定します。このオプションの値は、ノードIDのカンマ区切りリストです。ノードのIDは、`tiup cluster display`コマンドによって返される[クラスタステータステーブル](/tiup/tiup-component-cluster-display.md)の最初の列から取得できます。
- データ型: `STRINGS`
- このオプションがコマンドで指定されていない場合、デフォルトですべてのノードが起動されます。

> **注意:**
>
> 同時に`-R, --role`オプションを指定した場合、`-N, --node`と`-R, --role`の両方の仕様に一致するサービスノードのみが開始されます。

### -R, --role

- 開始するノードの役割を指定します。このオプションの値は、ノードの役割のコンマ区切りリストです。ノードの役割は、`tiup cluster display`コマンドによって返される[クラスタステータステーブル](/tiup/tiup-component-cluster-display.md)の2番目の列から取得できます。
- データ型: `STRINGS`
- このオプションがコマンドで指定されていない場合、デフォルトですべての役割が起動されます。

> **注意:**
>
> 同時に`-N, --node`オプションを指定した場合、`-N, --node`と`-R, --role`の両方の仕様に一致するサービスノードのみが開始されます。

### -h, --help

- ヘルプ情報を出力します。
- データ型: `BOOLEAN`
- このオプションは、デフォルトで`false`値で無効になっています。このオプションを有効にするには、コマンドにこのオプションを追加し、`true`値を渡すか、値を渡さないでください。

## 出力

サービス起動のログ。

[<< 前のページに戻る - TiUP Clusterコマンドリスト](/tiup/tiup-component-cluster.md#command-list)