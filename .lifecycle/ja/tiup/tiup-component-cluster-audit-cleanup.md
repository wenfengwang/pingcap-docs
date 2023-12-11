---
title: tiup クラスターオーディットクリーンアップ
---

# tiup クラスターオーディットクリーンアップ

`tiup cluster audit cleanup` コマンドは、`tiup cluster` コマンドを実行した際に生成されるログをクリーンアップするために使用されます。

## 構文

```shell
tiup cluster audit cleanup [flags]
```

## オプション

### --retain-days

- 保持するログの日数を指定します。
- データ型：`INT`
- デフォルト値：`60`（日単位）
- デフォルトでは、過去60日以内に生成されたログが保持され、それ以前のログは削除されます。

### -h, --help

- ヘルプ情報を表示します。
- データ型：`BOOLEAN`
- デフォルト値：`false`
- このオプションを有効にするには、コマンドにこのオプションを追加し、`true`値を渡すか、値なしで渡します。

## 出力

```shell
クリーンオーディットログに成功しました
```

[<< 前のページに戻る - TiUP クラスターコマンドリスト](/tiup/tiup-component-cluster.md#command-list)