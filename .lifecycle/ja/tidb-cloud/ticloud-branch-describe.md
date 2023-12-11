---
title: ticloud branch describe
summary: `ticloud branch describe`のリファレンス。

# ticloud branch describe

ブランチの情報（エンドポイント、[ユーザー名のプレフィックス](/tidb-cloud/select-cluster-tier.md#user-name-prefix)、使用法）を取得する：

```shell
ticloud branch describe [flags]
```

または、次のエイリアスコマンドを使用する：

```shell
ticloud branch get [flags]
```

## 例

インタラクティブモードでブランチ情報を取得する：

```shell
ticloud branch describe
```

非インタラクティブモードでブランチ情報を取得する：

```shell
ticloud branch describe --branch-id <branch-id> --cluster-id <cluster-id>
```

## フラグ

非インタラクティブモードでは、必要なフラグを手動で入力する必要があります。 インタラクティブモードでは、CLIのプロンプトに従って入力するだけで済みます。

| フラグ                  | 説明                           | 必須     | 備考                                           |
|------------------------|-------------------------------|----------|----------------------------------------------|
| -b、--branch-id string  | ブランチのID                  | はい     | 非インタラクティブモードでのみ有効です。   |
| -h、--help              | このコマンドのヘルプ情報      | いいえ   | 非インタラクティブおよびインタラクティブモードの両方で有効です。  |
| -c、--cluster-id string | ブランチのクラスターID        | はい   | 非インタラクティブモードでのみ有効です。   |

## 継承されたフラグ

| フラグ                | 説明                                                                                              | 必須     | 備考                                                |
|----------------------|--------------------------------------------------------------------------------------------------|----------|-----------------------------------------------------|
| --no-color           | 出力のカラーを無効にする                                                                           | いいえ   | 非インタラクティブモードでのみ有効です。 インタラクティブモードでは、一部のUIコンポーネントでカラーを無効にすることができないかもしれません。|
| -P、--profile string | このコマンドで使用されるアクティブな[ユーザープロファイル](/tidb-cloud/cli-reference.md#user-profile)を指定します。 | いいえ   | 非インタラクティブおよびインタラクティブモードの両方で有効です。|

## フィードバック

TiDB Cloud CLIについてご質問や提案がある場合は、[問題](https://github.com/tidbcloud/tidbcloud-cli/issues/new/choose)を作成していただくか、貢献を歓迎します。