---
title: ticloud config describe
summary: `ticloud config describe`の参照。

# ticloud config describe

特定の[user profile](/tidb-cloud/cli-reference.md#user-profile)のプロパティ情報を取得します：

```shell
ticloud config describe <profile-name> [flags]
```

または、次のエイリアスコマンドを使用します：

```shell
ticloud config get <profile-name> [flags]
```

## 例

ユーザープロファイルを記述します：

```shell
ticloud config describe <profile-name>
```

## フラグ

| フラグ       | 説明              |
|------------|--------------------------|
| -h、--help | このコマンドのヘルプ情報 |

## 継承されたフラグ

| フラグ                 | 説明                                   | 必須 | 備考                                                                                                                  |
|----------------------|-----------------------------------------------|----------|--------------------------------------------------------------------------------------------------------------------------|
| --no-color           | 出力のカラーを無効にします。                       | いいえ       | 非インタラクティブモードでのみ動作します。インタラクティブモードでは、一部のUIコンポーネントでカラーを無効にすることができない場合があります。 |
| -P、--profile string | このコマンドで使用されるアクティブな[user profile](/tidb-cloud/cli-reference.md#user-profile)を指定します。 | いいえ       | 非インタラクティブおよびインタラクティブの両方で動作します。                                                                      |

## フィードバック

TiDB Cloud CLIに関する質問や提案がある場合は、[issue](https://github.com/tidbcloud/tidbcloud-cli/issues/new/choose)を作成してください。また、私たちはどんな貢献でも歓迎します。