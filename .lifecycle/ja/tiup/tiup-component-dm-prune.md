---
title: tiup dmのプルーン
---

# tiup dmのプルーン

クラスタを縮小すると、通常問題が発生しないですが(etcdの/tiup/tiup-component-dm-scale-in.md)、etcdのメタデータがわずかにクリーンアップされないことがあります。メタデータをクリーンアップする必要がある場合は、`tiup dm prune`コマンドを手動で実行できます。

## 構文

```shell
tiup dm prune <cluster-name> [flags]
```

## オプション

### -h, --help

- ヘルプ情報を表示します。
- データタイプ: `BOOLEAN`
- デフォルト: false

## 出力

クリーンアッププロセスのログ。

[<< 前のページに戻る - TiUP DMコマンドリスト](/tiup/tiup-component-dm.md#command-list)