```yaml
---
title: tiup ミラーマージ
---

# tiup ミラーマージ

`tiup mirror merge` コマンドは、1つ以上のミラーを現在のミラーにマージするために使用されます。

このコマンドを実行するには、以下の条件を満たす必要があります:

- 対象ミラーのすべてのコンポーネントの所有者IDが現在のミラーに存在していること。
- このコマンドを実行するユーザーの `${TIUP_HOME}/keys` ディレクトリに、現在のミラーの上記所有者IDに対応するすべてのプライベートキーが含まれていること（[`tiup mirror set`](/tiup/tiup-command-mirror-set.md) コマンドを使用して、現在のミラーを現在認可されている変更が可能なミラーに切り替えることができます）。

## 構文

```shell
tiup mirror merge <mirror-dir-1> [mirror-dir-N] [flags]
```

- `<mirror-dir-1>`: 現在のミラーにマージされる最初のミラー
- `[mirror-dir-N]`: 現在のミラーにマージされる N 番目のミラー

## オプション

なし

## 出力

- コマンドが正常に実行された場合、出力はありません。
- 現在のミラーに対象ミラーのコンポーネントの所有者が存在しない場合、または `${TIUP_HOME}/keys` に所有者のプライベートキーが存在しない場合、TiUP は `Error: missing owner keys for owner %s on component %s` エラーを報告します。

[<< 前のページに戻る - TiUP Mirrorコマンドリスト](/tiup/tiup-command-mirror.md#command-list)
```