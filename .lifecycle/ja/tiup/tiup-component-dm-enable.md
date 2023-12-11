---
title: tiup dm の有効化
---

# tiup dm の有効化

`tiup dm enable` コマンドは、マシンが再起動された後にクラスターサービスの自動有効化を設定するために使用されます。このコマンドは、指定したノードで `systemctl enable <service>` を実行することで、サービスの自動有効化を有効にします。

## 構文

```shell
tiup dm enable <cluster-name> [flags]
```

`<cluster-name>` は、サービスの自動有効化を有効にするクラスターです。

## オプション

### -N, --node

- サービスの自動有効化を有効にするノードを指定します。このオプションの値は、ノードIDのコンマ区切りのリストです。ノードIDは、[`tiup dm display`](/tiup/tiup-component-dm-display.md) コマンドによって返されるクラスターのステータステーブルの最初の列から取得できます。
- データ型: `STRINGS`
- このオプションがコマンドで指定されていない場合、すべてのノードの自動有効化がデフォルトで有効になります。

> **注意:**
>
> `-R, --role` オプションが同時に指定されている場合、`-N, --node` と `-R, --role` の両方の仕様に一致するサービスの自動有効化が有効になります。

### -R, --role

- サービスの自動有効化を有効にする役割を指定します。このオプションの値は、コンマ区切りのノードの役割のリストです。ノードの役割は、[`tiup dm display`](/tiup/tiup-component-dm-display.md) コマンドによって返されるクラスターのステータステーブルの2番目の列から取得できます。
- データ型: `STRINGS`
- このオプションがコマンドで指定されていない場合、すべての役割の自動有効化がデフォルトで有効になります。

> **注意:**
>
> `-N, --node` オプションが同時に指定されている場合、`-N, --node` と `-R, --role` の両方の仕様に一致するサービスの自動有効化が有効になります。

### -h, --help

ヘルプ情報を表示します。

## 出力

tiup-dm の実行ログ。

[<< 前のページに戻る - TiUP DM コマンドリスト](/tiup/tiup-component-dm.md#command-list)