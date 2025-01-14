---
title: ticloud プロジェクト一覧
summary: `ticloud project list`の参照。
---

# ticloud プロジェクト一覧

すべてのアクセス可能なプロジェクトを一覧表示します：

```shell
ticloud project list [flags]
```

または、以下のエイリアスコマンドを使用できます：

```shell
ticloud project ls [flags]
```

## 例

すべてのアクセス可能なプロジェクトを一覧表示します：

```shell
ticloud project list
```

JSON形式ですべてのアクセス可能なプロジェクトを一覧表示します：

```shell
ticloud project list -o json
```

## フラグ

対話モードでない場合、必要なフラグを手動で入力する必要があります。対話モードでは、CLIのプロンプトに従って入力するだけで済みます。

| フラグ                | 説明                                                                                       | 必須    | 注意                                               |
|----------------------|-------------------------------------------------------------------------------------------|----------|---------------------------------------------------|
| -h, --help            | このコマンドのヘルプ情報                                                                    | いいえ   | 対話モードと対話モードの両方で動作します。      |
| -o, --output string   | 出力形式（デフォルトは `human`）。有効な値は `human` または `json` です。完全な結果を得るには、 `json` 形式を使用します。 | いいえ   | 対話モードと対話モードの両方で動作します。      |

## 継承されたフラグ

| フラグ                 | 説明                                                                             | 必須    | 注意                                                                                                          |
|-----------------------|---------------------------------------------------------------------------------|----------|---------------------------------------------------------------------------------------------------------------|
| --no-color            | 出力のカラーを無効にします。                                                       | いいえ   | 対話モードでは動作しません。一部のUIコンポーネントでは、色を無効にすることができない場合があります。 |
| -P, --profile string  | このコマンドで使用されるアクティブな[ユーザープロファイル](/tidb-cloud/cli-reference.md#user-profile)を指定します。 | いいえ   | 対話モードと対話モードの両方で動作します。     |

## フィードバック

TiDB Cloud CLIについてご質問やご提案がある場合は、[issue](https://github.com/tidbcloud/tidbcloud-cli/issues/new/choose)を作成してください。また、どんな貢献でも歓迎します。