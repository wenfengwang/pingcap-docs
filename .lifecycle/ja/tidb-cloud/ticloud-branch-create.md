---
title: ticloud branch create
summary: `ticloud branch create`の参照。

# ticloud branch create

クラスター用にブランチを作成します：

```shell
ticloud branch create [flags]
```

> **注意:**
>
> 現在、[TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスターのみ作成できます。

## 例

対話モードでブランチを作成する：

```shell
ticloud branch create
```

対話モードでブランチを作成する：

```shell
ticloud branch create --cluster-id <cluster-id> --branch-name <branch-name>
```

## フラグ

対話モードでは、CLIのプロンプトに従って必要なフラグを入力するだけで済みます。非対話モードでは、フラグを手動で入力する必要があります。

| フラグ                   | 説明                              | 必須     | 備考                                            |
|-------------------------|--------------------------------|----------|---------------------------------------------------|
| -c, --cluster-id string | ブランチを作成するクラスターのID    | はい   | 非対話モードでのみ動作します。                 |
| --branch-name string    | 作成するブランチの名前                | はい   | 非対話モードでのみ動作します。                 |
| -h, --help              | このコマンドのヘルプ情報を取得する  | いいえ | 非対話モードと対話モードの両方で動作します。|

## 継承されたフラグ

| フラグ            | 説明                                                      | 必須     | 備考                                                                                                                |
|----------------------|----------------------------------------------------------|----------|---------------------------------------------------------------------------------------------------------------------|
| --no-color           | 出力のカラーを無効にする                                    | いいえ | 非対話モードでのみ動作します。対話モードでは、一部のUIコンポーネントでカラーを無効にすることができないことがあります。    |
| -P, --profile string | このコマンドで使用するアクティブな[user profile](/tidb-cloud/cli-reference.md#user-profile)を指定します。| いいえ | 非対話モードと対話モードの両方で動作します。                                                                        |

## フィードバック

TiDB Cloud CLIに関する質問や提案がある場合は、[issue](https://github.com/tidbcloud/tidbcloud-cli/issues/new/choose)を作成してください。また、貢献を歓迎します。