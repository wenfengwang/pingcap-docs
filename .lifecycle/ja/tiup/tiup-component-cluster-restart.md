---
title: tiup クラスタ再起動
---

# tiup クラスタ再起動

`tiup cluster restart` コマンドは、指定したクラスタのすべてまたは一部のサービスを再起動するために使用されます。

> **注意:**
>
> 再起動プロセス中、関連するサービスは一時的に利用できなくなります。

## 構文

```shell
tiup cluster restart <cluster-name> [flags]
```

`<cluster-name>`: 操作するクラスタの名前。クラスタ名を忘れた場合は、[cluster list](/tiup/tiup-component-cluster-list.md) コマンドで確認できます。

## オプション

### -N, --node

- 再起動するノードを指定します。このオプションの値は、ノードIDのコンマ区切りのリストです。ノードIDは、`tiup cluster display` コマンドによって返される[cluster status table](/tiup/tiup-component-cluster-display.md) の最初の列から取得できます。
- データ型: `STRING`
- このオプションが指定されていない場合、TiUP はデフォルトですべてのノードを再起動します。

> **注意:**
>
> 同時に `-R, --role` オプションが指定されている場合、`-N, --node` および `-R, --role` の両方の条件に一致するサービスノードが再起動されます。

### -R, --role

- 再起動するノードの役割を指定します。このオプションの値は、ノードの役割のコンマ区切りのリストです。ノードの役割は、`tiup cluster display` コマンドによって返される[cluster status table](/tiup/tiup-component-cluster-display.md) の2番目の列から取得できます。
- データ型: `STRING`
- このオプションが指定されていない場合、TiUP はデフォルトですべての役割のノードを再起動します。

> **注意:**
>
> 同時に `-N, --node` オプションが指定されている場合、`-N, --node` および `-R, --role` の両方の条件に一致するサービスノードが再起動されます。

### -h, --help

- ヘルプ情報を表示します。
- データ型: `BOOLEAN`
- このオプションはデフォルトで `false` 値で無効化されています。このオプションを有効にするには、このオプションをコマンドに追加し、`true` 値を渡すか、値を渡さないでください。

## 出力

サービス再起動プロセスのログ。

[<< 前のページに戻る - TiUP クラスタコマンドリスト](/tiup/tiup-component-cluster.md#command-list)