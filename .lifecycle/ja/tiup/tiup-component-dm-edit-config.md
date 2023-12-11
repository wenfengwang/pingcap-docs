---
title: tiup dm edit-config
---

# tiup dm edit-config

クラスターが展開された後にクラスターサービス構成を変更する必要がある場合は、指定されたクラスターの[トポロジファイル](/tiup/tiup-dm-topology-reference.md)を変更するために、`tiup dm edit-config`コマンドを使用できます。このエディターはデフォルトで`$EDITOR`環境変数によって指定されます。`$EDITOR`環境変数が存在しない場合、`vi`エディターが使用されます。

> **注意:**
>
> + 構成を変更する場合、マシンを追加したり削除したりすることはできません。マシンを追加する方法については、[クラスターをスケールアウトする](/tiup/tiup-component-dm-scale-out.md)を参照してください。マシンを削除する方法については、[クラスターをスケールインする](/tiup/tiup-component-dm-scale-in.md)を参照してください。
> + `tiup dm edit-config`コマンドを実行した後、構成は制御マシン上でのみ変更されます。その後、`tiup dm reload`コマンドを実行して構成を再読み込みする必要があります。

## 構文

```shell
tiup dm edit-config <cluster-name> [flags]
```

`<cluster-name>`: 操作するクラスター。

## オプション

### -h, --help

- ヘルプ情報を表示します。
- データタイプ：`BOOLEAN`
- デフォルト：false

## 出力

- 通常、出力はありません。
- 変更できないフィールドを誤って変更した場合、ファイルを保存するとエラーが報告され、ファイルを再編集するようにとのメッセージが表示されます。変更できないフィールドについては、[トポロジファイル](/tiup/tiup-dm-topology-reference.md)を参照してください。

[<<前のページに戻る - TiUP DMコマンドリスト](/tiup/tiup-component-dm.md#command-list)