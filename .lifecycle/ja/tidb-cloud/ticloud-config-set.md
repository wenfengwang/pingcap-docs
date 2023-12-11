---
title: ticloudコンフィギュレーションの設定
summary: `ticloud config set`のリファレンス。

# ticloudコンフィギュレーションの設定

アクティブな[ユーザープロファイル](/tidb-cloud/cli-reference.md#user-profile)のプロパティを設定します：

```shell
ticloud config set <property-name> <value> [flags]
```

設定可能なプロパティには、`public-key`、`private-key`、`api-url`があります。

| プロパティ  | 説明                                                        | 必須 |
|-------------|------------------------------------------------------------|------|
| public-key  | TiDB Cloud APIの公開鍵                                        | はい |
| private-key | TiDB Cloud APIの秘密鍵                                        | はい |
| api-url     | TiDB CloudのベースAPI URL（デフォルトは `https://api.tidbcloud.com`） | いいえ |

> **注:**
>
> 特定のユーザープロファイルのプロパティを設定する場合は、`-P` フラグを追加し、コマンドで対象のユーザープロファイル名を指定できます。

## 例

アクティブなプロファイルの public-keyの値を設定する：

```shell
ticloud config set public-key <public-key>
```

特定のプロファイル `test` の public-keyの値を設定する：

```shell
ticloud config set public-key <public-key> -P test
```

APIホストを設定する：

```shell
ticloud config set api-url https://api.tidbcloud.com
```

> **注:**
>
> デフォルトでは、TiDB Cloud APIのURLは `https://api.tidbcloud.com` です。通常は、設定する必要はありません。

## フラグ

| フラグ      | 説明             |
|------------|-----------------|
| -h, --help | このコマンドのヘルプ情報 |

## 継承されたフラグ

| フラグ                 | 説明                                   | 必須 | ノート                                                                                                                    |
|----------------------|---------------------------------------|------|--------------------------------------------------------------------------------------------------------------------------|
| --no-color           | 出力のカラーを無効にする。                      | いいえ | 非インタラクティブモードのみ有効。インタラクティブモードでは、一部のUIコンポーネントでのカラー無効化が機能しない場合があります。 |
| -P, --profile string | このコマンドで使用されるアクティブな[ユーザープロファイル](/tidb-cloud/cli-reference.md#user-profile)を指定します。 | いいえ | 非インタラクティブモードとインタラクティブモードの両方で動作します。                                                                      |

## フィードバック

TiDB Cloud CLIに関する質問や提案がありましたら、[issue](https://github.com/tidbcloud/tidbcloud-cli/issues/new/choose)を作成してください。また、ご提案を歓迎します。