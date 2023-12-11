---
title: ticloud config edit
summary: 「ticloud config edit」の参照。

# ticloud config edit

macOSまたはLinuxを使用している場合、デフォルトのテキストエディタでプロファイル構成ファイルを開くことができます：

```shell
ticloud config edit [flags]
```

Windowsを使用している場合、前述のコマンドを実行した後、プロファイル構成ファイルのパスが表示されます。

> **注意:**
>
> 形式エラーや実行の失敗を避けるために、プロファイル構成ファイルを手動で編集することはお勧めしません。代わりに、[`ticloud config create`](/tidb-cloud/ticloud-config-create.md)、[`ticloud config delete`](/tidb-cloud/ticloud-config-delete.md)、または[`ticloud config set`](/tidb-cloud/ticloud-config-set.md)を使用して構成を変更することができます。

## 例

プロファイル構成ファイルを編集する：

```shell
ticloud config edit
```

## フラグ

| フラグ       | 説明              |
|------------|--------------------------|
 | -h、--help | このコマンドのヘルプ情報 |

## 継承されたフラグ

| フラグ                 | 説明                                                                               | 必須 | 注記                                                                                                                    |
|----------------------|-------------------------------------------------------------------------------------------|----------|--------------------------------------------------------------------------------------------------------------------------|
| --no-color           | 出力のカラーを無効にします。                                                                  | いいえ       | 非インタラクティブモードでのみ動作します。インタラクティブモードでは、一部のUIコンポーネントでカラーの無効化が機能しないことがあります。 |
| -P、--profile string | このコマンドで使用されるアクティブな[user profile](/tidb-cloud/cli-reference.md#user-profile)を指定します。 | いいえ       | 非インタラクティブモードとインタラクティブモードの両方で動作します。                                                                      |

## フィードバック

TiDB Cloud CLIに関する質問や提案がある場合は、[issue](https://github.com/tidbcloud/tidbcloud-cli/issues/new/choose)を作成してください。また、どんな貢献でも歓迎します。