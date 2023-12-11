---
title: ticloud branch connect-info
summary: `ticloud branch connect-info`のリファレンス。

# ticloud branch connect-info

ブランチの接続文字列を取得します：

```shell
ticloud branch connect-info [フラグ]
```

## 例

対話モードでブランチの接続文字列を取得する：

```shell
ticloud branch connect-info
```

非対話モードでブランチの接続文字列を取得する：

```shell
ticloud branch connect-info --branch-id <ブランチID> --cluster-id <クラスタID> --client <クライアント名> --operating-system <オペレーティングシステム>
```

## フラグ

非対話モードでは、必要なフラグを手動で入力する必要があります。対話モードでは、CLIプロンプトに従って入力するだけで済みます。

| フラグ                      | 説明                                                                                                                                                                                                                                                                                                                                                                          | 必須    | 注意                                                 |
|---------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|------------------------------------------------------|
| -c、--cluster-id string   | ブランチが作成されたクラスタのID                                                                                                                                                                                                                                                                                                                                                                               | はい      | 非対話モードでのみ動作します。                  |
| -b、--branch-id string    | ブランチのID                                                                                                                                                                                                                                                                                                                                                                                                                | はい      | 非対話モードでのみ動作します。                  |
| --client string           | 接続に使用する希望のクライアント。サポートされるクライアントには、`general`、`mysql_cli`、`mycli`、`libmysqlclient`、`python_mysqlclient`、`pymysql`、`mysql_connector_python`、`mysql_connector_java`、`go_mysql_driver`、`node_mysql2`、`ruby_mysql2`、`php_mysqli`、`rust_mysql`、`mybatis`、`hibernate`、`spring_boot`、`gorm`、`prisma`、`sequelize_mysql2`、`django_tidb`、`sqlalchemy_mysqlclient`、および`active_record`があります。 | はい      | 非対話モードでのみ動作します。                  |
| --operating-system string | オペレーティングシステムの名前。サポートされるオペレーティングシステムには、`macOS`、`Windows`、`Ubuntu`、`CentOS`、`RedHat`、`Fedora`、`Debian`、`Arch`、`OpenSUSE`、`Alpine`、および`その他`があります。                                                                                                                                                                                                                                                    | はい      | 非対話モードでのみ動作します。                  |
| -h、--help                | このコマンドのヘルプ情報                                                                                                                                                                                                                                                                                                                                                                                                   | いいえ       | 非対話モードと対話モードの両方で動作します。 |

## 継承されたフラグ

| フラグ                 | 説明                                                                                           | 必須 | 注意                                                                                                              |
|----------------------|-------------------------------------------------------------------------------------------------------|----------|-------------------------------------------------------------------------------------------------------------------|
| --no-color           | 出力のカラーを無効にします。                                                                             | いいえ       | 非対話モードでのみ動作します。対話モードでは、一部のUIコンポーネントでカラーを無効にすることができない場合があります。  |
| -P、--profile string | このコマンドで使用されるアクティブな[ユーザープロファイル](/tidb-cloud/cli-reference.md#user-profile)を指定します。  | いいえ       | 非対話モードと対話モードの両方で動作します。                                                              |

## フィードバック

TiDB Cloud CLIに関する質問や提案がある場合は、[issue](https://github.com/tidbcloud/tidbcloud-cli/issues/new/choose)を作成してください。また、どんな貢献にも歓迎します。