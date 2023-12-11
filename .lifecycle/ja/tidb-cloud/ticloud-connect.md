---
title: ticloud connect
summary: `ticloud connect`の参照。

# ticloud connect

TiDB Cloudクラスターまたはブランチに接続します：

```shell
ticloud connect [flags]
```

> **Note:**
>
> - デフォルトのユーザーを使用するかどうかを尋ねられた場合、デフォルトのrootユーザーを使用するには`Y`を選択するか、別のユーザーを指定するには`n`を選択します。[TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスターの場合、デフォルトのrootユーザーの名前には`3pTAoNNegb47Uc8`などの[プレフィックス](/tidb-cloud/select-cluster-tier.md#user-name-prefix)が付いています。
> - 接続はセッションに[ANSI SQLモード](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_ansi)を強制します。セッションを終了するには、`\q`を入力してください。

## 例

対話モードでTiDB Cloudクラスターまたはブランチに接続します：

```shell
ticloud connect
```

デフォルトのユーザーを使用して、対話モードでTiDB Cloudクラスターまたはブランチに接続します：

```shell
ticloud connect -p <project-id> -c <cluster-id>
ticloud connect -p <project-id> -c <cluster-id> -b <branch-id>
```

パスワードを使用して、対話モードでデフォルトのユーザーでTiDB Cloudクラスターまたはブランチに接続します：

```shell
ticloud connect -p <project-id> -c <cluster-id> --password <password>
ticloud connect -p <project-id> -c <cluster-id> -b <branch-id> --password <password>
```

特定のユーザーを使用して、対話モードでTiDB Cloudクラスターまたはブランチに接続します：

```shell
ticloud connect -p <project-id> -c <cluster-id> -u <user-name>
ticloud connect -p <project-id> -c <cluster-id> -b <branch-id> -u <user-name>
```

## フラグ

対話モードでは、必要なフラグをCLIプロンプトに従って入力するだけで済みます。非対話モードでは、必要なフラグを手動で入力する必要があります。

| フラグ                   | 説明                              | 必須   | ノート                                                      |
|-------------------------|-----------------------------------|----------|------------------------------------------------------|
| -c, --cluster-id string | クラスターID                     | はい      | 非対話モードでのみ動作します。                  |
| -b, --branch-id string  | ブランチID                       | いいえ   | 非対話モードでのみ動作します。                  |
| -h, --help              | このコマンドのヘルプ情報         | いいえ   | 非対話および対話モードの両方で動作します。 |
| --password              | ユーザーのパスワード              | いいえ   | 非対話モードでのみ動作します。                  |
| -p, --project-id string | プロジェクトID                    | はい      | 非対話モードでのみ動作します。                  |
| -u, --user string       | ログイン用の特定のユーザー       | いいえ   | 非対話モードでのみ動作します。                  |

## 継承されたフラグ

| フラグ                 | 説明                                                                                          | 必須 | ノート                                                                 |
|----------------------|------------------------------------------------------------------------------------------------------|----------|--------------------------------------------------------------------------------------------------------------------------|
| --no-color           | 出力の色を無効にします。                                                                            | いいえ       | 非対話モードでのみ動作します。対話モードでは、一部のUIコンポーネントでカラーを無効にすることができないことがあります。                                                                     |
| -P, --profile string | このコマンドで使用されるアクティブな[ユーザープロファイル](/tidb-cloud/cli-reference.md#user-profile)を指定します。 | いいえ | 非対話および対話モードの両方で動作します。                                                                     |

## フィードバック

TiDB Cloud CLIに関する質問や提案がある場合は、[issue](https://github.com/tidbcloud/tidbcloud-cli/issues/new/choose)を作成してください。また、私たちはどんな貢献でも歓迎します。