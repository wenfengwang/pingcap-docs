---
title: ticloud import start mysql
summary: `ticloud import start mysql`の参照。

# ticloud import start mysql

MySQL互換のデータベースからテーブルを[TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless)クラスターにインポートします:

```shell
ticloud import start mysql [flags]
```

> **注意:**
>
> - このコマンドを実行する前に、MySQLのコマンドラインツールがインストールされていることを確認してください。詳細については、[Installation](/tidb-cloud/get-started-with-cli.md#installation)を参照してください。
> - 対象のテーブルが対象のデータベースにすでに存在する場合、このコマンドをテーブルインポートに使用するには、対象のテーブル名がソースのテーブル名と同じであることを確認し、コマンドに`skip-create-table`フラグを追加してください。
> - 対象のテーブルが対象のデータベースに存在しない場合、このコマンドを実行すると自動的に対象のデータベースにソースのテーブルと同じ名前のテーブルが作成されます。

## Examples

- インタラクティブモードでインポートタスクを開始:

    ```shell
    ticloud import start mysql
    ```

- 非インタラクティブモードでインポートタスクを開始（TiDB Serverlessクラスターのデフォルトのユーザ`<username-prefix>.root`を使用）:

    ```shell
    ticloud import start mysql --project-id <project-id> --cluster-id <cluster-id> --source-host <source-host> --source-port <source-port> --source-user <source-user> --source-password <source-password> --source-database <source-database> --source-table <source-table> --target-database <target-database> --target-password <target-password>
    ```

- 非インタラクティブモードでインポートタスクを開始（特定のユーザを使用）:

    ```shell
    ticloud import start mysql --project-id <project-id> --cluster-id <cluster-id> --source-host <source-host> --source-port <source-port> --source-user <source-user> --source-password <source-password> --source-database <source-database> --source-table <source-table> --target-database <target-database> --target-password <target-password> --target-user <target-user>
    ```

- 対象のテーブルが対象のデータベースにすでに存在する場合、対象テーブルの作成をスキップするインポートタスクを開始:

    ```shell
    ticloud import start mysql --project-id <project-id> --cluster-id <cluster-id> --source-host <source-host> --source-port <source-port> --source-user <source-user> --source-password <source-password> --source-database <source-database> --source-table <source-table> --target-database <target-database> --target-password <target-password> --skip-create-table
    ```

> **注意:**
>
> MySQL 8.0では、デフォルトの照合順序として`utf8mb4_0900_ai_ci`が使用されていますが、これは現在TiDBでサポートされていません。ソーステーブルが`utf8mb4_0900_ai_ci`照合順序を使用している場合、インポートの前に、[TiDBでサポートされている照合順序](/character-set-and-collation.md#character-sets-and-collations-supported-by-tidb)にソーステーブルの照合順序を変更するか、TiDBで対象のテーブルを手動で作成する必要があります。

## フラグ

非インタラクティブモードでは、必要なフラグを手動で入力する必要があります。インタラクティブモードでは、CLIのプロンプトに従って入力するだけで済みます。

| フラグ | 説明 | 必須 | ノート |
|---|---|---|---|
| -c, --cluster-id string | クラスターIDを指定します。 | はい | 非インタラクティブモードでのみ動作します。 |
| -h, --help | このコマンドのヘルプ情報を表示します。 | いいえ | 非インタラクティブおよびインタラクティブモードの両方で動作します。 |
| -p, --project-id string | プロジェクトIDを指定します。 | はい | 非インタラクティブモードでのみ動作します。 |
| --skip-create-table | 対象のテーブルが対象のデータベースにすでに存在する場合、対象テーブルの作成をスキップします。 | いいえ | 非インタラクティブモードでのみ動作します。 |
| --source-database string | ソースのMySQLデータベースの名前。 | はい | 非インタラクティブモードでのみ動作します。 |
| --source-host string | ソースのMySQLインスタンスのホスト。 | はい | 非インタラクティブモードでのみ動作します。 |
| --source-password string | ソースのMySQLインスタンスのパスワード。 | はい | 非インタラクティブモードでのみ動作します。 |
| --source-port int | ソースのMySQLインスタンスのポート。 | はい | 非インタラクティブモードでのみ動作します。 |
| --source-table string | ソースのMySQLデータベースのソーステーブル名。 | はい | 非インタラクティブモードでのみ動作します。 |
| --source-user string | ソースのMySQLインスタンスにログインするユーザー。 | はい | 非インタラクティブモードでのみ動作します。 |
| --target-database string | TiDB Serverlessクラスターの対象データベース名。 | はい | 非インタラクティブモードでのみ動作します。 |
| --target-password string | 対象のTiDB Serverlessクラスターのパスワード。 | はい | 非インタラクティブモードでのみ動作します。 |
| --target-user string | 対象のTiDB Serverlessクラスターにログインするユーザー。 | いいえ | 非インタラクティブモードでのみ動作します。 |

## 継承されたフラグ

| フラグ | 説明 | 必須 | ノート |
|---|---|---|---|
| --no-color | 出力のカラーを無効にします。 | いいえ | 非インタラクティブモードでのみ動作します。インタラクティブモードでは、一部のUIコンポーネントでカラーを無効にすることができない場合があります。 |
| -P, --profile string | このコマンドで使用されるアクティブな[ユーザープロファイル](/tidb-cloud/cli-reference.md#user-profile)を指定します。 | いいえ | 非インタラクティブおよびインタラクティブモードの両方で動作します。 |

## フィードバック

TiDB Cloud CLIに関する質問や提案がある場合は、[issue](https://github.com/tidbcloud/tidbcloud-cli/issues/new/choose)を作成してください。また、どんな貢献も歓迎します。