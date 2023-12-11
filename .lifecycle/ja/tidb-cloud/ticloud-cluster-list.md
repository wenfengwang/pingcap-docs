---
title: ticloudクラスターリスト
summary: `ticloud cluster list`のリファレンス
---

# ticloudクラスターリスト

プロジェクト内のすべてのクラスターをリストします：

```shell
ticloud cluster list <project-id> [フラグ]
```

または、次のエイリアスコマンドを使用します：

```shell
ticloud cluster ls <project-id> [フラグ]
```

## 例

プロジェクト内のすべてのクラスターをリスト（対話モード）：

```shell
ticloud cluster list
```

指定したプロジェクト内のすべてのクラスターをリスト（非対話モード）：

```shell
ticloud cluster list <project-id>
```

指定したプロジェクト内のすべてのクラスターをJSON形式でリストします：

```shell
ticloud cluster list <project-id> -o json
```

## フラグ

非対話モードでは、必要なフラグを手動で入力する必要があります。 対話モードでは、CLIのプロンプトに従って入力するだけで済みます。

| フラグ                 | 説明                                                                                                | 必須     | ノート                                             |
|-----------------------|------------------------------------------------------------------------------------------------------|------------|------------------------------------------------------|
| -h、--help           | このコマンドのヘルプ情報                                                                                  | いいえ     | 非対話モードと対話モードの両方で動作します。 |
| -o、--output string | 出力形式（デフォルトは`human`）。 有効な値は`human`または`json`です。 完全な結果を得るには、`json`形式を使用してください。 | いいえ     | 非対話モードと対話モードの両方で動作します。 |

## 継承されたフラグ

| フラグ                 | 説明                                                                                      | 必須     | ノート                                                                                                                          |
|-----------------------|-------------------------------------------------------------------------------------------|------------|--------------------------------------------------------------------------------------------------------------------------------|
| --no-color            | 出力の色を無効にします。                                                                       | いいえ     | 非対話モードのみで動作します。 対話モードでは、一部のUIコンポーネントで色が無効にならないことがあります。 |
| -P、--profile string | このコマンドで使用されるアクティブな[user profile](/tidb-cloud/cli-reference.md#user-profile)を指定します。 | いいえ     | 対話モードと非対話モードの両方で動作します。                                                                       |

## フィードバック

TiDB Cloud CLIに関する質問や提案がある場合は、[issue](https://github.com/tidbcloud/tidbcloud-cli/issues/new/choose)を作成してください。 また、どんな貢献でも歓迎します。