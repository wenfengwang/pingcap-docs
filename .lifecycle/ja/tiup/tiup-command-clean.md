---
title: tiup clean
---

# tiup clean

`tiup clean`コマンドは、コンポーネントの操作中に生成されたデータを消去するために使用されます。

## 構文

```shell
tiup clean [name] [flags]
```

`[name]`の値は、[`status`コマンド](/tiup/tiup-command-status.md)によって出力される`Name`フィールドです。`[name]`を省略する場合は、`tiup clean`コマンドで`--all`オプションを追加する必要があります。

## オプション

### --all

- すべての操作レコードを消去します
- データ型: ブール値
- デフォルト: false

## 出力

```
インスタンスのクリーニング: `%s`、ディレクトリ: %s
```

[<< 前のページに戻る - TiUPリファレンスコマンドリスト](/tiup/tiup-reference.md#command-list)