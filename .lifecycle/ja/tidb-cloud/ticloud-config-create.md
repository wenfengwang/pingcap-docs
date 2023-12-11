---
title: ticloud config create
summary: `ticloud config create`の参照。

# ticloud config create

[ユーザープロファイル](/tidb-cloud/cli-reference.md#user-profile)を作成して、ユーザープロファイルの設定を保管します。

```shell
ticloud config create [flags]
```

> **注記:**
>
> ユーザープロファイルを作成する前に、[TiDB Cloud APIキーを作成](https://docs.pingcap.com/tidbcloud/api/v1beta#section/Authentication/API-Key-Management)する必要があります。

## 例

インタラクティブモードでユーザープロファイルを作成します。

```shell
ticloud config create
```

非インタラクティブモードでユーザープロファイルを作成します。

```shell
ticloud config create --profile-name <profile-name> --public-key <public-key> --private-key <private-key>
```

## フラグ

非インタラクティブモードでは、必要なフラグを手動で入力する必要があります。インタラクティブモードでは、CLIのプロンプトに従って入力するだけです。

| フラグ                 | 説明                                       | 必須 | 注記                            |
|-----------------------|-------------------------------------------|------|-----------------------------------|
| -h, --help            | このコマンドのヘルプ情報                   | いいえ | 非インタラクティブモードとインタラクティブモードの両方で機能します。 |
| --private-key string  | TiDB Cloud APIのプライベートキー           | はい | 非インタラクティブモードでのみ機能します。 |
| --profile-name string | `.`を含めることができないプロファイルの名前 | はい | 非インタラクティブモードでのみ機能します。 |
| --public-key string   | TiDB Cloud APIのパブリックキー            | はい | 非インタラクティブモードでのみ機能します。 |

## 継承されたフラグ

| フラグ                | 説明                                     | 必須 | 注記                                                                                                              |
|----------------------|-----------------------------------------|------|--------------------------------------------------------------------------------------------------------------------|
| --no-color           | 出力のカラーを無効にする                 | いいえ | 非インタラクティブモードでのみ機能します。インタラクティブモードでは、一部のUIコンポーネントでカラーの無効化が機能しない場合があります。   |
| -P, --profile string | このコマンドで使用されるアクティブな[ユーザープロファイル](/tidb-cloud/cli-reference.md#user-profile)を指定します。 | いいえ | 非インタラクティブモードとインタラクティブモードの両方で機能します。                                                                      |

## フィードバック

TiDB Cloud CLIについてご質問や提案がある場合は、[issue](https://github.com/tidbcloud/tidbcloud-cli/issues/new/choose)を作成してください。また、コントリビューションも歓迎します。