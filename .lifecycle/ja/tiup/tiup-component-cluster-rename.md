---
title: tiupクラスターの名前変更
---

# tiupクラスターの名前変更

クラスターの名前は [クラスターが展開されるとき](/tiup/tiup-component-cluster-deploy.md) に指定されます。クラスターを展開した後にクラスターの名前を変更したい場合は、`tiup cluster rename` コマンドを使用できます。

> **注意:**
>
> TiUPクラスターの `dashboard_dir` フィールドに `grafana_servers` が構成されている場合、`tiup cluster rename` コマンドを実行してクラスターの名前を変更した後、以下の追加ステップが必要です:
>
> + ローカルのダッシュボードディレクトリ内の `*.json` ファイルについて、各ファイルの `datasource` フィールドを新しいクラスター名に更新する必要があります。なぜなら `datasource` の値はクラスター名でなければならないからです。
> + `tiup cluster reload -R grafana` コマンドを実行します。

## 構文

```shell
tiup cluster rename <old-cluster-name> <new-cluster-name> [flags]
```

- `<old-cluster-name>`: 古いクラスター名。
- `<new-cluster-name>`: 新しいクラスター名。

## オプション

### -h, --help

- ヘルプ情報を印刷します。
- データタイプ: `BOOLEAN`
- このオプションはデフォルトで無効になっており、`false` の値です。このオプションを有効にするには、このオプションをコマンドに追加し、`true` の値を渡すか、値を渡さないようにします。

## 出力

tiup-clusterの実行ログ。

[<< 前のページに戻る - TiUPクラスターコマンドの一覧](/tiup/tiup-component-cluster.md#command-list)