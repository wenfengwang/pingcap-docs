---
title: tiup クラスター メタ バックアップ
---

# tiup クラスター メタ バックアップ

TiUP メタファイルはクラスターの運用およびメンテナンス（OM）に使用されます。このファイルが紛失すると、TiUP を使用してクラスターを管理することはできません。このような状況を避けるために、`tiup cluster meta backup` コマンドを定期的に使用して TiUP メタファイルをバックアップできます。

## 構文

```shell
tiup cluster meta backup <cluster-name> [flags]
```

`<cluster-name>` は操作対象のクラスターの名前です。クラスター名を忘れた場合は、[`tiup dm list`](/tiup/tiup-component-dm-list.md) コマンドを使用して確認できます。

## オプション

### --file (string, 現在のディレクトリをデフォルトとしています)

TiUP メタバックアップファイルを保存するターゲットディレクトリを指定します。

### -h, --help

- ヘルプ情報を表示します。
- データ型：`Boolean`
- このオプションはデフォルトで無効になっており、デフォルト値は `false` です。このオプションを有効にするには、コマンドにこのオプションを追加し、`true` 値を渡すか、値を渡さないでください。

## 出力

tiup-cluster の実行ログ。

[<< 前のページに戻る - TiUP クラスター コマンドリスト](/tiup/tiup-component-cluster.md#command-list)