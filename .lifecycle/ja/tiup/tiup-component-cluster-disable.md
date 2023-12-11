---
title: tiup cluster disable
---

# tiup cluster disable

クラスターサービスが存在するマシンを再起動すると、クラスターサービスは自動的に有効になります。クラスターサービスの自動有効化を無効にするには、`tiup cluster disable`コマンドを使用できます。このコマンドは指定されたノード上で`systemctl disable <service>`を実行して、サービスの自動有効化を無効にします。

## 構文

```shell
tiup cluster disable <cluster-name> [flags]
```

`<cluster-name>`: 自動有効化を無効にするクラスター。

## オプション

### -N, --node

- サービスの自動有効化を無効にするノードを指定します。このオプションの値は、カンマ区切りのノードIDのリストです。ノードIDは、[`tiup cluster display`](/tiup/tiup-component-cluster-display.md)コマンドによって返されるクラスターステータステーブルの最初の列から取得できます。
- データ型: `STRINGS`
- このオプションがコマンドで指定されていない場合、デフォルトですべてのノードの自動有効化が無効になります。

> **注意:**
>
> 同時に`-R, --role`オプションが指定されている場合、`-N, --node`と`-R, --role`の両方の仕様に一致するサービスの自動有効化が無効になります。

### -R, --role

- サービスの自動有効化を無効にするロールを指定します。このオプションの値は、カンマ区切りのノードロールのリストです。ノードの役割は、[`tiup cluster display`](/tiup/tiup-component-cluster-display.md)コマンドによって返されるクラスターステータステーブルの2番目の列から取得できます。
- データ型: `STRINGS`
- このオプションがコマンドで指定されていない場合、デフォルトですべての役割の自動有効化が無効になります。

> **注意:**
>
> 同時に`-N, --node`オプションが指定されている場合、`-N, --node`と`-R, --role`の両方の仕様に一致するサービスの自動有効化が無効になります。

### -h, --help

- ヘルプ情報を出力します。
- データ型: `BOOLEAN`
- このオプションはデフォルトで無効になっており、デフォルト値は`false`です。このオプションを有効にするには、コマンドにこのオプションを追加し、`true`値を渡すか、値を渡さないでください。

## 出力

tiup-clusterの実行ログ。

[<<前のページに戻る - TiUP Clusterコマンドリスト](/tiup/tiup-component-cluster.md#command-list)