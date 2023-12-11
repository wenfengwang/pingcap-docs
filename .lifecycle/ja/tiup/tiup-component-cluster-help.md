---
title: tiup cluster ヘルプ
---

# tiup cluster ヘルプ

tiup-clusterは、コマンドラインインタフェースでユーザーに豊富なヘルプ情報を提供します。`help`コマンドまたは`--help`オプションを使用して取得できます。`tiup cluster ヘルプ <command>`は基本的に`tiup cluster <command> --help`と同等です。

## 構文

```shell
tiup cluster ヘルプ [command] [flags]
```

`[command]`は、ユーザーが表示するヘルプ情報を指定するために使用されます。指定しない場合、tiup-clusterのヘルプ情報が表示されます。

### -h, --help

- ヘルプ情報を出力します。
- データ型: `BOOLEAN`
- デフォルト: false

## 出力

`[command]`またはtiup-clusterのヘルプ情報。

[<<前のページに戻る - TiUP Clusterコマンドリスト](/tiup/tiup-component-cluster.md#command-list)