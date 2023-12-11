---
title: ticloud クラスタ接続情報
summary: `ticloud cluster connect-info` のリファレンス。

# ticloud クラスタ接続情報

クラスタの接続文字列を取得します:

```shell
ticloud cluster connect-info [フラグ]
```

> **注意:**
>
> 現時点では、このコマンドは、[TiDB Serverless](/tidb-cloud/select-cluster-tier.md#tidb-serverless) クラスタの接続文字列の取得のみをサポートしています。

## 例

対話モードでクラスタの接続文字列を取得します:

```shell
ticloud cluster connect-info
```

非対話モードでクラスタの接続文字列を取得します:

```shell
ticloud cluster connect-info --project-id <project-id> --cluster-id <cluster-id> --client <client-name> --operating-system <operating-system>
```

## フラグ

非対話モードでは、必要なフラグを手動で入力する必要があります。対話モードでは、CLI プロンプトに従って入力するだけで済みます。

| フラグ                       | 説明                                                                                                                                                                                                                                                                                                                                                                | 必須 | 備考                                                      |
|----------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|------------------------------------------------------|
| -p, --project-id string    | クラスタが作成されたプロジェクトの ID                                                                                                                                                                                                                                                                                                                | はい      | 非対話モードでのみ有効です。                  |
| -c, --cluster-id string    | クラスタの ID                                                                                                                                                                                                                                                                                                                                                      | はい      | 非対話モードでのみ有効です。                  |
| --client string            | 接続に使用するクライアント。サポートされているクライアントは、`general`、`mysql_cli`、`mycli`、`libmysqlclient`、`python_mysqlclient`、`pymysql`、`mysql_connector_python`、`mysql_connector_java`、`go_mysql_driver`、`node_mysql2`、`ruby_mysql2`、`php_mysqli`、`rust_mysql`、`mybatis`、`hibernate`、`spring_boot`、`gorm`、`prisma`、`sequelize_mysql2`、`django_tidb`、`sqlalchemy_mysqlclient` および `active_record`。 | はい      | 非対話モードでのみ有効です。                  |
| --operating-system string  | オペレーティング システム名。サポートされているオペレーティング システムは、`macOS`、`Windows`、`Ubuntu`、`CentOS`、`RedHat`、`Fedora`、`Debian`、`Arch`、`OpenSUSE`、`Alpine` および `Others`。                                                                                                                    | はい      | 非対話モードでのみ有効です。                  |
| -h, --help                 | このコマンドのヘルプ情報                                                                                                                                                                                                                                                                                                                                          | いいえ       | 非対話および対話モードの両方で動作します。 |

## 継承されたフラグ

| フラグ                 | 説明                                                                                           | 必須 | 備考                                                                                                              |
|----------------------|-------------------------------------------------------------------------------------------------------|----------|-------------------------------------------------------------------------------------------------------------------|
| --no-color           | 出力のカラーを無効にします。                                                                             | いいえ       | 非対話モードでのみ有効です。対話モードでは、一部の UI コンポーネントでカラーを無効にできない場合があります。  |
| -P, --profile string | このコマンドで使用されるアクティブな [ユーザープロファイル](/tidb-cloud/cli-reference.md#user-profile) を指定します。  | いいえ       | 非対話および対話モードの両方で動作します。                                                              |

## フィードバック

TiDB Cloud CLI に関するご質問やご提案がある場合は、[issue](https://github.com/tidbcloud/tidbcloud-cli/issues/new/choose) を作成してください。また、どんな貢献も歓迎します。