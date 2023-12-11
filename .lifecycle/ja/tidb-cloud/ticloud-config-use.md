---
title: ticloud config use
summary: `ticloud config use`のリファレンス。

# ticloud config use

特定のプロファイルをアクティブな[ユーザープロファイル](/tidb-cloud/cli-reference.md#user-profile)として設定します：

```shell
ticloud config use <profile-name> [flags]
```

## 例

`test`プロファイルをアクティブなユーザープロファイルとして設定します：

```shell
ticloud config use test
```

## フラグ

| フラグ      | 説明                   |
|------------|-----------------------|
| -h, --help | このコマンドのヘルプ情報 |

## 継承されたフラグ

| フラグ                   | 説明                                       | 必須     | 注記                                                                                                                    |
|---------------------|-----------------------------------------|----------|------------------------------------------------------------------------------------------------------------------------|
| --no-color              | 出力のカラーを無効にします。                      | いいえ     | 非インタラクティブモードの場合のみ動作します。インタラクティブモードでは、一部のUIコンポーネントでのカラー無効化が機能しない場合があります。 |
| -P, --profile string | このコマンドで使用されるアクティブな[ユーザープロファイル](/tidb-cloud/cli-reference.md#user-profile)を指定します。 | いいえ     | 非インタラクティブモードとインタラクティブモードの両方で動作します。                                                                      |

## フィードバック

TiDB Cloud CLIに関する質問や提案がある場合は、[issue](https://github.com/tidbcloud/tidbcloud-cli/issues/new/choose)を作成してください。また、どんな貢献も歓迎します。