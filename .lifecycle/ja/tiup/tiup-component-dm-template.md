---
title: tiup dm テンプレート
---

# tiup dm テンプレート

クラスターを展開する前に、クラスターの [トポロジファイル](/tiup/tiup-dm-topology-reference.md) を準備する必要があります。TiUP には組み込みのトポロジファイルテンプレートがあり、このテンプレートを修正して最終的なトポロジファイルを作成できます。組み込みのテンプレート内容を出力するには、 `tiup dm template` コマンドを使用できます。

## 構文

```shell
tiup dm template [flags]
```

このオプションが指定されていない場合、デフォルトのテンプレートには以下のインスタンスが含まれます。

- 3 つの DM マスターインスタンス
- 3 つの DM ワーカーインスタンス
- 1 つの Prometheus インスタンス
- 1 つの Grafana インスタンス
- 1 つの Alertmanager インスタンス

## オプション

### --full

- 設定可能なパラメータでコメントが付いた詳細なトポロジテンプレートを出力します。このオプションを有効にするには、コマンドに追加します。
- このオプションが指定されていない場合、デフォルトで簡易なトポロジテンプレートが出力されます。

### -h, --help

ヘルプ情報を出力します。

## 出力

指定したオプションに応じてトポロジテンプレートを出力し、展開用のトポロジファイルにリダイレクトできます。

[<< 前のページに戻る - TiUP DM コマンドリスト](/tiup/tiup-component-dm.md#command-list)