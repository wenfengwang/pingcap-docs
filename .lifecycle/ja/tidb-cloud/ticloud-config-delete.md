---
title: ticloud config delete
summary: `ticloud config delete`のリファレンス。

# ticloud config delete

[user profile](/tidb-cloud/cli-reference.md#user-profile)を削除します:

```shell
ticloud config delete <profile-name> [flags]
```

または、以下のエイリアスコマンドを使用できます:

```shell
ticloud config rm <profile-name> [flags]
```

## 例

ユーザープロファイルを削除する:

```shell
ticloud config delete <profile-name>
```

## フラグ

| フラグ       | 説明                           |
|------------|---------------------------------------|
| --force    | 確認なしでプロファイルを削除します |
| -h, --help | このコマンドのヘルプ情報             |

## 継承されたフラグ

| フラグ                 | 説明                                   | 必須 | 備考                                                                                                                    |
|----------------------|-----------------------------------------------|----------|--------------------------------------------------------------------------------------------------------------------------|
| --no-color           | 出力の色を無効にします。                    | いいえ       | 非インタラクティブモードでのみ動作します。インタラクティブモードでは、特定のUIコンポーネントによって色を無効にすることができない場合があります |
| -P, --profile string | このコマンドで使用されるアクティブな[user profile](/tidb-cloud/cli-reference.md#user-profile)を指定します。 | いいえ       | 非インタラクティブおよびインタラクティブモードの両方で動作します。                                                                      |

## フィードバック

TiDB Cloud CLIについてご質問やご提案がある場合は、[issue](https://github.com/tidbcloud/tidbcloud-cli/issues/new/choose)を作成してください。また、どんな貢献も歓迎します。