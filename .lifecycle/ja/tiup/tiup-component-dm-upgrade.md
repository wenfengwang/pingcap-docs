---
title: tiup dm のアップグレード
---

# tiup dm のアップグレード

`tiup dm upgrade`コマンドは、指定されたクラスタを特定のバージョンにアップグレードするために使用されます。

## 構文

```shell
tiup dm upgrade <cluster-name> <version> [flags]
```

- `<cluster-name>` は操作対象のクラスタ名です。クラスタ名を忘れた場合は、[`tiup dm list`](/tiup/tiup-component-dm-list.md)コマンドを使用して確認できます。
- `<version>` はアップグレード対象のバージョン（例：`v7.4.0`）です。現在、後のバージョンへのアップグレードのみが許可されており、以前のバージョンへのアップグレードは許可されていません。つまり、ダウングレードは許可されていません。また、ナイトリーバージョンへのアップグレードも許可されていません。

## オプション

### --offline

- 現在のクラスタがオフラインであることを宣言します。このオプションが指定されている場合、TiUP DMはサービスを再起動せずにクラスタコンポーネントのバイナリファイルのみを置換します。

### -h、--help

- ヘルプ情報を表示します。
- データ型：`BOOLEAN`
- このオプションはデフォルトで`false`値で無効になっています。このオプションを有効にするには、コマンドにこのオプションを追加し、`true`値を渡すか、値を渡さないでください。

## 出力

サービスアップグレードプロセスのログ。

[<< 前のページに戻る - TiUP DMコマンドリスト](/tiup/tiup-component-dm.md#command-list)