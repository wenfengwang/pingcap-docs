---
title: ticloud import start local
summary: `ticloud import start local`の参照。

# ticloud import start local

[tiDBサーバーレス](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスターにローカルファイルをインポートします：

```shell
ticloud import start local <file-path> [flags]
```

> **注意:**
>
> 現在、1つのインポートタスクにつき1つのCSVファイルのみをインポートできます。

## 例

対話モードでインポートタスクを開始します：

```shell
ticloud import start local <file-path>
```

非対話モードでインポートタスクを開始します：

```shell
ticloud import start local <file-path> --project-id <project-id> --cluster-id <cluster-id> --data-format <data-format> --target-database <target-database> --target-table <target-table>
```

カスタムCSV形式でインポートタスクを開始します：

```shell
ticloud import start local <file-path> --project-id <project-id> --cluster-id <cluster-id> --data-format CSV --target-database <target-database> --target-table <target-table> --separator \" --delimiter \' --backslash-escape=false --trim-last-separator=true
```

## フラグ

非対話モードでは、必要なフラグを手動で入力する必要があります。対話モードでは、CLIプロンプトに従って入力するだけです。

| フラグ | 説明 | 必須 | 注意 |
|---|---|---|---|
| --backslash-escape | CSVファイル内のバックスラッシュをエスケープ文字として解析するかどうか。デフォルト値は`true`です。 | いいえ | `--data-format CSV`が指定されている場合にのみ、非対話モードで動作します。 |
| -c、--cluster-id string | クラスターIDを指定します。 | はい | 非対話モードでのみ動作します。 |
| --data-format string | データ形式を指定します。現在は`CSV`のみサポートされています。 | はい | 非対話モードでのみ動作します。 |
| --delimiter string | CSVファイルの引用符に使用する区切り文字を指定します。デフォルト値は`"`です。 | いいえ | `--data-format CSV`が指定されている場合にのみ、非対話モードで動作します。 |
| -h、--help | このコマンドのヘルプ情報を表示します。 | いいえ | 非対話モードと対話モードの両方で動作します。 |
| -p、--project-id string | プロジェクトIDを指定します。 | はい | 非対話モードでのみ動作します。 |
| --separator string | CSVファイルのフィールドセパレータを指定します。デフォルト値は`,`です。 | いいえ | `--data-format CSV`が指定されている場合にのみ、非対話モードで動作します。 |
| --target-database string | データをインポートするターゲットデータベースを指定します。 | はい | 非対話モードでのみ動作します。 |
| --target-table string | データをインポートするターゲットテーブルを指定します。 | はい | 非対話モードでのみ動作します。 |
| --trim-last-separator | セパレータを行終端記号として扱い、すべての末尾のセパレータをトリミングするかどうか。デフォルト値は`false`です。 | いいえ | `--data-format CSV`が指定されている場合にのみ、非対話モードで動作します。 |

## 継承されたフラグ

| フラグ | 説明 | 必須 | 注意 |
|---|---|---|---|
| --no-color | 出力のカラーを無効にします。 | いいえ | 非対話モードでのみ動作します。 対話モードでは、一部のUIコンポーネントでカラーを無効にすることができない場合があります。 |
| -P、--profile string | このコマンドで使用されるアクティブな[user profile](/tidb-cloud/cli-reference.md#user-profile)を指定します。 | いいえ | 非対話モードと対話モードの両方で動作します。 |

## フィードバック

TiDB Cloud CLIに関する質問や提案がある場合は、[issue](https://github.com/tidbcloud/tidbcloud-cli/issues/new/choose)を作成してください。また、すべての貢献を歓迎します。