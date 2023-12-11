---
title: tiup cluster meta restore
---

# tiup cluster meta restore

TiUP メタファイルを復元するには、`tiup cluster meta restore` コマンドを使用してバックアップファイルから復元します。

## 構文

```shell
tiup cluster meta restore <クラスタ名> <バックアップファイル> [フラグ]
```

- `<クラスタ名>` は操作対象のクラスタの名前です。
- `<バックアップファイル>` は TiUP メタバックアップファイルへのパスです。

> **注意:**
>
> 復元操作は現在のメタファイルを上書きします。メタファイルが失われた場合にのみ復元することを推奨します。

## オプション

### -h, --help

- ヘルプ情報を表示します。
- データ型：`Boolean`
- このオプションはデフォルトで無効化されており、デフォルト値は `false` です。このオプションを有効にするには、コマンドにこのオプションを追加し、`true` 値を渡すか、または値を渡さないでください。

## 出力

tiup-cluster の実行ログ。

[<< 前のページに戻る - TiUP Cluster のコマンドリスト](/tiup/tiup-component-cluster.md#command-list)