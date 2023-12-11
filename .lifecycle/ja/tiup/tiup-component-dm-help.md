---
title: tiup dm ヘルプ
---

# tiup dm ヘルプ

tiup-dmコマンドラインインターフェースは、ユーザーに豊富なヘルプ情報を提供します。`help`コマンドまたは`--help`オプションを使用して表示できます。基本的に、`tiup dm help <command>`は、`tiup dm <command> --help`と同等です。

## 構文

```shell
tiup dm help [コマンド] [フラグ]
```

`[コマンド]`は、ユーザーが表示する必要のあるコマンドのヘルプ情報を指定するために使用されます。指定しない場合、`tiup-dm`のヘルプ情報が表示されます。

### -h、--help

- ヘルプ情報を出力します。
- データ型：`BOOLEAN`
- デフォルト：false

## 出力

`[コマンド]`または`tiup-dm`のヘルプ情報。

[<< 前のページに戻る - TiUP DM コマンドリスト](/tiup/tiup-component-dm.md#command-list)