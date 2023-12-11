---
title: tiup cluster edit-config
---

# tiup cluster edit-config

クラスタが展開された後にクラスタ構成を変更する必要がある場合は、クラスタの[topology file](/tiup/tiup-cluster-topology-reference.md)を編集するために`tiup cluster edit-config`コマンドを使用できます。このエディタは、デフォルトで`$EDITOR`環境変数で指定されます。`$EDITOR`環境変数が存在しない場合、`vi`エディタが使用されます。

> **注意:**
>
> + 構成を変更する場合、マシンを追加したり削除したりすることはできません。マシンを追加する方法については、[クラスタをスケーリングアウトする](/tiup/tiup-component-cluster-scale-out.md)を参照してください。マシンを削除する方法については、[クラスタをスケーリングインする](/tiup/tiup-component-cluster-scale-in.md)を参照してください。
> + `tiup cluster edit-config`コマンドを実行した後、構成は制御マシンのみが変更されます。その後、`tiup cluster reload`コマンドを実行して構成を再読み込みする必要があります。

## 構文

```shell
tiup cluster edit-config <cluster-name> [flags]
```

`<cluster-name>`は操作対象のクラスタです。

## オプション

### -h, --help

- ヘルプ情報を出力します。
- データ型: `BOOLEAN`
- このオプションはデフォルトで無効になっており、既定値は`false`です。このオプションを有効にするには、コマンドにこのオプションを追加し、`true`の値を渡すか、値を渡さないでください。

## 出力

- コマンドが正常に実行された場合、出力はありません。
- 変更できないフィールドを誤って変更した場合、ファイルを保存するとエラーが報告され、ファイルを再編集するように促されます。変更できないフィールドについては、[topology file](/tiup/tiup-cluster-topology-reference.md)を参照してください。

[<< 前のページに戻る - TiUPクラスタのコマンド一覧](/tiup/tiup-component-cluster.md#command-list)