---
title: ticloud インポート 開始 s3
summary: `ticloud import start s3` のリファレンス。

# ticloud インポート 開始 s3

Amazon S3 からファイルを TiDB Cloud にインポートします:

```shell
ticloud import start s3 [flags]
```

> **注意:**
>
> Amazon S3 からファイルを TiDB Cloud にインポートする前に、TiDB Cloud のために Amazon S3 バケットのアクセスを構成し、Role ARN を取得する必要があります。詳細については [Amazon S3 アクセスの構成](/tidb-cloud/config-s3-and-gcs-access.md#configure-amazon-s3-access) を参照してください。

## 例

対話モードでインポートタスクを開始:

```shell
ticloud import start s3
```

対話モードでないでインポートタスクを開始:

```shell
ticloud import start s3 --project-id <project-id> --cluster-id <cluster-id> --aws-role-arn <aws-role-arn> --data-format <data-format> --source-url <source-url>
```

カスタム CSV 形式でインポートタスクを開始:

```shell
ticloud import start s3 --project-id <project-id> --cluster-id <cluster-id> --aws-role-arn <aws-role-arn> --data-format CSV --source-url <source-url> --separator \" --delimiter \' --backslash-escape=false --trim-last-separator=true
```

## フラグ

対話モードでないで、必要なフラグを手動で入力する必要があります。 対話モードでは、CLI プロンプトに従って入力するだけで済みます。

| フラグ | 説明 | 必須 | メモ |
|---|---|---|---|
| --aws-role-arn string | Amazon S3 データソースにアクセスするために使用される AWS ロール ARN を指定します。 | はい | 対話モードでは機能しません。 |
| --backslash-escape | CSV ファイル内のフィールド内のバックスラッシュをエスケープ文字として解析するかどうか。デフォルト値は `true` です。 | いいえ | `--data-format CSV` が指定された場合に、対話モードでは機能しません。 |
| -c, --cluster-id string | クラスタ ID を指定します。 | はい | 対話モードでは機能しません。 |
| --data-format string | データ形式を指定します。有効な値は `CSV`、`SqlFile`、`Parquet`、または `AuroraSnapshot` です。 | はい | 対話モードでは機能しません。 |
| --delimiter string | CSV ファイルのクォーティングに使用されるデリミタを指定します。デフォルト値は `"` です。 | いいえ | `--data-format CSV` が指定された場合に、対話モードでは機能しません。 |
| -h, --help | このコマンドのヘルプ情報を表示します。 | いいえ | 対話モードと対話モードの両方で機能します。 |
| -p, --project-id string | プロジェクト ID を指定します。 | はい | 対話モードでは機能しません。 |
| --separator string | CSV ファイルのフィールド区切り記号を指定します。デフォルト値は `,` です。 | いいえ | `--data-format CSV` が指定された場合に、対話モードでは機能しません。 |
| --source-url string | ソースデータファイルが格納されている S3 パスです。 | はい | 対話モードでは機能しません。 |
| --trim-last-separator | 区切り記号を行終端記号として扱い、CSV ファイルのすべての末尾の区切り記号をトリミングするかどうか。デフォルト値は `false` です。 | いいえ | `--data-format CSV` が指定された場合に、対話モードでは機能しません。 |

## 継承されたフラグ

| フラグ | 説明 | 必須 | メモ |
|---|---|---|---|
| --no-color | 出力の色を無効にします。 | いいえ | 対話モードでは機能しません。対話モードでは、一部の UI コンポーネントで色の無効化が機能しないことがあります。 |
| -P, --profile string | このコマンドで使用される [ユーザープロファイル](/tidb-cloud/cli-reference.md#user-profile) を指定します。 | いいえ | 対話モードと対話モードの両方で機能します。 |

## フィードバック

TiDB Cloud CLI に関する質問や提案がある場合は、[issue](https://github.com/tidbcloud/tidbcloud-cli/issues/new/choose) を作成してください。また、どんな貢献も歓迎します。