---
title: より多くの列を持つ下流TiDBテーブルにデータを移行する
summary: 対応する上流テーブルよりも多くの列を持つ下流TiDBテーブルへデータを移行する方法について学ぶ
aliases: ['/tidb/dev/usage-scenario-downstream-more-columns/']
---

# より多くの列を持つ下流TiDBテーブルにデータを移行する

このドキュメントは、対応する上流テーブルよりも多くの列を持つ下流TiDBテーブルにデータを移行する際に必要な追加手順を提供します。通常の移行手順については、以下の移行シナリオを参照してください。

- [MySQLからTiDBへの小規模データセットの移行](/migrate-small-mysql-to-tidb.md)
- [MySQLからTiDBへの大規模データセットの移行](/migrate-large-mysql-to-tidb.md)
- [小規模なMySQLシャードのマージとTiDBへの移行](/migrate-small-mysql-shards-to-tidb.md)
- [大規模なMySQLシャードのマージとTiDBへの移行](/migrate-large-mysql-shards-to-tidb.md)

## DMを使用して、より多くの列を持つ下流のTiDBテーブルにデータを移行する

上流のbinlogを複製する際、DMは下流の現在のテーブルスキーマを使用してbinlogを解析し、対応するDMLステートメントを生成しようとします。上流のbinlogのテーブルの列数が、下流のテーブルスキーマの列数と一致しない場合、次のエラーが発生します。

```json
"errors": [
    {
        "ErrCode": 36027,
        "ErrClass": "sync-unit",
        "ErrScope": "internal",
        "ErrLevel": "high",
        "Message": "startLocation: [position: (mysql-bin.000001, 2022), gtid-set:09bec856-ba95-11ea-850a-58f2b4af5188:1-9 ], endLocation: [ position: (mysql-bin.000001, 2022), gtid-set: 09bec856-ba95-11ea-850a-58f2b4af5188:1-9]: gen insert sqls failed, schema: log, table: messages: Column count doesn't match value count: 3 (columns) vs 2 (values)",
        "RawCause": "",
        "Workaround": ""
    }
]
```

以下は上流のテーブルスキーマの例です。

```sql
# 上流のテーブルスキーマ
CREATE TABLE `messages` (
  `id` int(11) NOT NULL,
  PRIMARY KEY (`id`)
)
```

以下は下流のテーブルスキーマの例です。

```sql
# 下流のテーブルスキーマ
CREATE TABLE `messages` (
  `id` int(11) NOT NULL,
  `message` varchar(255) DEFAULT NULL, # これは下流のテーブルにのみ存在する追加の列です。
  PRIMARY KEY (`id`)
)
```

DMは上流で生成されたbinlogイベントを解析する際に下流のテーブルスキーマを使用しようとしますが、上記の `Column count doesn't match` エラーが発生します。

このような場合、`binlog-schema` コマンドを使用してデータソースから移行されるテーブルのためにテーブルスキーマを設定することができます。指定されたテーブルスキーマは、DMによって複製されるbinlogイベントデータに対応する必要があります。シャード化されたテーブルを移行している場合は、各シャードテーブルに対して、DMでbinlogイベントデータを解析するためにテーブルスキーマを設定する必要があります。手順は次のとおりです。

1. DMでSQLファイルを作成し、上流のテーブルスキーマに対応する `CREATE TABLE` ステートメントをファイルに追加します。たとえば、次のテーブルスキーマを `log.messages.sql` に保存します。DM v6.0以降のバージョンでは、SQLファイルを作成せずに `--from-source` または `--from-target` フラグを追加してテーブルスキーマを更新することができます。詳細については、[移行するテーブルのスキーマの管理](/dm/dm-manage-schema.md)を参照してください。

    ```sql
    # 上流のテーブルスキーマ
    CREATE TABLE `messages` (
    `id` int(11) NOT NULL,
    PRIMARY KEY (`id`)
    )
    ```

2. `binlog-schema` コマンドを使用して、データソースから移行されるテーブルのためにテーブルスキーマを設定します。この時点で、上記の `Column count doesn't match` エラーのためにデータ移行タスクが一時停止状態にある必要があります。

    {{< copyable "shell-regular" >}}

    ```
    tiup dmctl --master-addr ${advertise-addr} binlog-schema update -s ${source-id} ${task-name} ${database-name} ${table-name} ${schema-file}
    ```

    このコマンドのパラメータの説明は以下のとおりです。

    |パラメータ |説明|
    |:-- |:---|
    |`-master-addr` | DM-masterが外部に公開するアドレスである`${advertise-addr}`を指定します。|
    |`binlog-schema set`| スキーマ情報を手動で設定します。|
    |`-s` | ソースを指定します。`${source-id}` はMySQLデータのソースIDを示します。|
    |`${task-name}`| データ移行タスクの `task.yaml` 設定ファイルで定義された移行タスクの名前を指定します。|
    |`${database-name}`| データベースを指定します。`${database-name}` は上流データベースの名前を示します。|
    |`${table-name}`| 上流テーブルの名前を指定します。|
    |`${schema-file}`| 設定するテーブルスキーマファイルを指定します。|

    例:

    {{< copyable "shell-regular" >}}

    ```
    tiup dmctl --master-addr 172.16.10.71:8261 binlog-schema update -s mysql-01 task-test -d log -t message log.message.sql
    ```

3. `resume-task` コマンドを使用して、一時停止状態の移行タスクを再開します。

    {{< copyable "shell-regular" >}}

    ```
    tiup dmctl --master-addr ${advertise-addr} resume-task ${task-name}
    ```

4. `query-status` コマンドを使用して、データ移行タスクが正常に実行されているかを確認します。

    {{< copyable "shell-regular" >}}

    ```
    tiup dmctl --master-addr ${advertise-addr} query-status resume-task ${task-name}
    ```