---
title: TiDBデータ移行における失敗したDDLステートメントの処理方法
summary: TiDBデータ移行ツールを使用してデータを移行する際に失敗したDDLステートメントを処理する方法について学びます。
aliases: ['/docs/tidb-data-migration/dev/skip-or-replace-abnormal-sql-statements/']
---

# TiDBデータ移行における失敗したDDLステートメントの処理方法

このドキュメントでは、TiDBデータ移行（DM）ツールを使用してデータを移行する際に失敗したDDLステートメントを処理する方法について紹介します。

現在、TiDBはすべてのMySQL構文と完全に互換性があるわけではありません（[TiDBがサポートしているDDLステートメント](/mysql-compatibility.md#ddl-operations)を参照）。そのため、DMがMySQLからTiDBにデータを移行する際に、TiDBが対応していないDDLステートメントがあるとエラーが発生し、移行プロセスが中断されることがあります。この場合、DMの`binlog`コマンドを使用して移行を再開することができます。

## 制限事項

次の状況ではこのコマンドを使用しないでください。

* ダウンストリームのTiDBで失敗したDDLステートメントをスキップすることが実際の本番環境で許容されない場合。
* 失敗したDDLステートメントを他のDDLステートメントで置き換えることができない場合。
* 他のDDLステートメントをダウンストリームのTiDBに挿入してはいけない場合。

たとえば、`DROP PRIMARY KEY`の場合、このシナリオでは、新しいテーブルを作成して（DDLステートメントを実行した後）、この新しいテーブルにデータをすべて再インポートすることしかできません。

## サポートされているシナリオ

移行中、TiDBがサポートしていないDDLステートメントがアップストリームで実行され、ダウンストリームに移行された結果、移行タスクが中断されることがあります。

- もし、このDDLステートメントがダウンストリームのTiDBでスキップされることを許容する場合、`binlog skip <task-name>`を使用してこのDDLステートメントの移行をスキップし、移行を再開することができます。
- もし、このDDLステートメントが他のDDLステートメントで置き換えられることを許容する場合、`binlog replace <task-name>`を使用してこのDDLステートメントを他のDDLステートメントで置き換え、移行を再開することができます。
- もし、他のDDLステートメントがダウンストリームのTiDBに挿入されることを許容する場合、`binlog inject <task-name>`を使用して他のDDLステートメントを挿入し、移行を再開することができます。

## コマンド

失敗したDDLステートメントを手動で処理するためにdmctlを使用する際、よく使用されるコマンドには `query-status` と `binlog` があります。

### query-status

`query-status` コマンドは、各MySQLインスタンスのサブタスクやリレーユニットなどの現在のステータスをクエリするために使用されます。詳細については、「[ステータスのクエリ](/dm/dm-query-status.md)」を参照してください。

### binlog

`binlog` コマンドは、バイナリログ操作を管理および表示するために使用されます。このコマンドは、DM v6.0およびそれ以降のバージョンでのみサポートされています。それ以前のバージョンでは、`handle-error` コマンドを使用してください。

`binlog` の使用法は次のとおりです:

```bash
binlog -h
```

```bash
バイナリログ操作の管理または表示

使用法:
  dmctl binlog [command]

利用可能なコマンド:
  inject      現在のエラーイベントまたは特定のバイナリログ位置（binlog-pos）にDDLステートメントを挿入します
  list        バイナリログ位置（binlog-pos）でのエラーハンドルコマンドまたはバイナリログ位置（binlog-pos）後のエラーハンドルコマンドを一覧表示します
  replace     特定のバイナリログ位置（binlog-pos）の現在のエラーイベントまたは特定のバイナリログ位置（binlog-pos）にDDLステートメントを置換します
  revert      現在のバイナリログ操作または特定のバイナリログ位置（binlog-pos）の操作を元に戻します
  skip        特定のバイナリログ位置（binlog-pos）のDDLステートメントをスキップします

フラグ:
  -b, --binlog-pos string   バイナリログイベントを一致させるために使用される位置。 現在のバイナリ操作が実行されます。指定されていない場合、DMは自動的に現在の失敗したDDLステートメントと`binlog-pos`を設定します。
  -h, --help                binlog のヘルプ

グローバルフラグ:
  -s, --source strings   MySQL ソースID。

コマンドの詳細については、「dmctl binlog [command] --help」を使用してください。
```

`binlog` は次のサブコマンドをサポートしています:

* `inject`：現在のエラーイベントまたは特定のバイナリログ位置にDDLステートメントを挿入します。バイナリログ位置を指定するには、`-b, --binlog-pos` を参照してください。
* `list`：現在のバイナリログ位置または現在のバイナリログ位置の後で、すべての有効な`inject`、`skip`、`replace`操作を一覧表示します。バイナリログ位置を指定するには、`-b, --binlog-pos`を参照してください。
* `replace`：特定のバイナリログ位置でのDDLステートメントを他のDDLステートメントで置き換えます。バイナリログ位置を指定するには、`-b, --binlog-pos`を参照してください。
* `revert`：指定されたバイナリログ操作の前の操作を取り消します（以前の操作が効果を生じない場合にのみ）。バイナリログ位置を指定するには、`-b, --binlog-pos`を参照してください。
* `skip`：特定のバイナリログ位置でのDDLステートメントをスキップします。バイナリログ位置を指定するには、`-b, --binlog-pos`を参照してください。

`binlog` は次のフラグをサポートしています:

+ `-b, --binlog-pos`:
    - タイプ: 文字列。
    - バイナリログ位置を指定します。バイナリログイベントの位置が `binlog-pos` と一致した場合、操作が実行されます。指定されていない場合、DMは失敗したDDLステートメントの現在の位置を自動的に`binlog-pos`に設定します。
    - 形式: `binlog-filename:binlog-pos`、例: `mysql-bin|000001.000003:3270`。
    - 移行中にエラーが発生した場合、バイナリログ位置は`query-status`によって返される`startLocation`の`position`から取得できます。移行のエラーが発生する前に、バイナリログ位置はアップストリームMySQLインスタンスで`SHOW BINLOG EVENTS`を使用して取得できます。

+ `-s, --source`:
    - タイプ: 文字列。
    - 事前に設定された操作を実行するMySQLインスタンスを指定します。

## 使用例

### 移行が中断された場合にDDLをスキップする

移行が中断された際にDDLステートメントをスキップする必要がある場合、`binlog skip`コマンドを実行します:

```bash
binlog skip -h
```

```bash
現在のエラーイベントまたは特定のバイナリログ位置（binlog-pos）のイベントをスキップします

使用法:
  dmctl binlog skip <task-name> [flags]

フラグ:
  -h, --help   skipのヘルプ

グローバル フラグ:
  -b, --binlog-pos string   バイナリログイベントを一致させるために使用される位置。binlog操作が一致した場合、操作が実行されます。形式は「mysql-bin|000001.000003:3270」となります
  -s, --source strings      MySQLソースID。
```

#### シャードマージのないシナリオ

アップストリームのテーブル `db1.tbl1` をダウンストリームのTiDBに移行する必要があるとします。初期のテーブルスキーマは次のとおりです:

{{< copyable "sql" >}}

```sql
SHOW CREATE TABLE db1.tbl1;
```

```sql
+-------+--------------------------------------------------+
| Table | Create Table                                     |
+-------+--------------------------------------------------+
| tbl1  | CREATE TABLE `tbl1` (
  `c1` int(11) NOT NULL,
  `c2` decimal(11,3) DEFAULT NULL,
  PRIMARY KEY (`c1`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1 |
+-------+--------------------------------------------------+
```

そして、次のDDLステートメントがアップストリームで実行され、テーブルスキーマが変更されることになります（つまり、c2のDECIMAL(11,3)をDECIMAL(10,3)に変更します）:

{{< copyable "sql" >}}

```sql
ALTER TABLE db1.tbl1 CHANGE c2 c2 DECIMAL (10, 3);
```

このDDLステートメントはTiDBでサポートされていないため、DMの移行タスクが中断されます。`query-status <task-name>`コマンドを実行すると、次のエラーが表示されます:

```
ERROR 8200 (HY000): Unsupported modify column: can't change decimal column precision
```

このDDLステートメントがダウンストリームのTiDBで実行されないことが実際の本番環境で許容されると仮定します（つまり、元のテーブルスキーマが保持される）。その場合は、`binlog skip <task-name>`を使用してこのDDLステートメントをスキップして移行を再開することができます。手順は次のとおりです:

1. 現在の失敗したDDLステートメントをスキップするために `binlog skip <task-name>` を実行します:

    {{< copyable "" >}}

    ```bash
    » binlog skip test
    ```

    ```
    {
        "result": true,
        "msg": "",
        "sources": [
            {
                "result": true,
                "msg": "",
                "source": "mysql-replica-01",
                "worker": "worker1"
            }
        ]
    }
    ```

2. タスクのステータスを表示するために `query-status <task-name>` を実行します:

    {{< copyable "" >}}

    ```bash
    » query-status test
    ```

    <details><summary> 実行結果を表示します。</summary>

    ```

    {
        "result": true,
        "msg": "",
        "sources": [
            {
                "result": true,
                "msg": "",
                "sourceStatus": {
                    "source": "mysql-replica-01",
                    "worker": "worker1",
```json
{
  "result": null,
  "relayStatus": null
},
"subTaskStatus": [
  {
    "name": "test",
    "stage": "Running",
    "unit": "Sync",
    "result": null,
    "unresolvedDDLLockID": "",
    "sync": {
      "masterBinlog": "(DESKTOP-T561TSO-bin.000001, 2388)",
      "masterBinlogGtid": "143bdef3-dd4a-11ea-8b00-00155de45f57:1-10",
      "syncerBinlog": "(DESKTOP-T561TSO-bin.000001, 2388)",
      "syncerBinlogGtid": "143bdef3-dd4a-11ea-8b00-00155de45f57:1-4",
      "blockingDDLs": [
      ],
      "unresolvedGroups": [
      ],
      "synced": true,
      "binlogType": "remote",
      "totalRows": "4",
      "totalRps": "0",
      "recentRps": "0"
    }
  }
]
}
```

<label><summary> 実行結果を参照してください。</summary>

```
{
  "result": true,
  "msg": "",
  "sources": [
    {
      "result": true,
      "msg": "",
      "sourceStatus": {
        "source": "mysql-replica-01",
        "worker": "worker1",
        "result": null,
        "relayStatus": null
      },
      "subTaskStatus": [
        {
          "name": "test",
          "stage": "Running",
          "unit": "Sync",
          "result": null,
          "unresolvedDDLLockID": "",
          "sync": {
            "masterBinlog": "(DESKTOP-T561TSO-bin.000001, 2388)",
            "masterBinlogGtid": "143bdef3-dd4a-11ea-8b00-00155de45f57:1-10",
            "syncerBinlog": "(DESKTOP-T561TSO-bin.000001, 2388)",
            "syncerBinlogGtid": "143bdef3-dd4a-11ea-8b00-00155de45f57:1-4",
            "blockingDDLs": [
            ],
            "unresolvedGroups": [
            ],
            "synced": true,
            "binlogType": "remote",
            "totalRows": "4",
            "totalRps": "0",
            "recentRps": "0"
          }
        }
      ]
    },
    {
      "result": true,
      "msg": "",
      "sourceStatus": {
        "source": "mysql-replica-02",
        "worker": "worker2",
        "result": null,
        "relayStatus": null
      },
      "subTaskStatus": [
        {
          "name": "test",
          "stage": "Running",
          "unit": "Sync",
          "result": null,
          "unresolvedDDLLockID": "",
          "sync": {
            "masterBinlog": "(DESKTOP-T561TSO-bin.000001, 2388)",
            "masterBinlogGtid": "143bdef3-dd4a-11ea-8b00-00155de45f57:1-10",
            "syncerBinlog": "(DESKTOP-T561TSO-bin.000001, 2388)",
            "syncerBinlogGtid": "143bdef3-dd4a-11ea-8b00-00155de45f57:1-4",
            "blockingDDLs": [
            ],
            "unresolvedGroups": [
            ],
            "synced": true,
            "binlogType": "remote",
            "totalRows": "4",
            "totalRps": "0",
            "recentRps": "0"
          }
        }
      ]
    }
  ]
}
```

</details>

タスクがエラーなく正常に実行され、間違ったDDLが４つスキップされていることが分かります。
```markdown
  + {{< copyable "sql" >}}
  ```SQL
  +-------+-----------------------------------------------------------------------------------------------------------+
  | テーブル | テーブルの作成                                                                                                    |
  +-------+-----------------------------------------------------------------------------------------------------------+
  | tb    | CREATE TABLE `tbl1` (
    `id` int(11) DEFAULT NULL,
    PRIMARY KEY (`id`)
  ) ENGINE=InnoDB DEFAULT CHARSET=latin1 COLLATE=latin1_bin |
  +-------+-----------------------------------------------------------------------------------------------------------+
  ```

  {{< copyable "sql" >}}
  ```sql
  +-------+-----------------------------------------------------------------------------------------------------------+
  | テーブル | テーブルの作成                                                                                                    |
  +-------+-----------------------------------------------------------------------------------------------------------+
  | tb    | CREATE TABLE `shard_table` (
    `id` int(11) DEFAULT NULL,
    PRIMARY KEY (`id`)
  ) ENGINE=InnoDB DEFAULT CHARSET=latin1 COLLATE=latin1_bin |
  +-------+-----------------------------------------------------------------------------------------------------------+
  ```

  {{< copyable "" >}}
  ```markdown
    <details><summary> 実行結果を確認します。</summary>

    ```
    {
        "result": true,
        "msg": "",
        "sources": [
            {
                "result": true,
                "msg": "",
                "sourceStatus": {
                    "source": "mysql-replica-01",
                    "worker": "worker1",
                    "result": null,
                    "relayStatus": null
                },
                "subTaskStatus": [
                    {
                        "name": "test",
                        "stage": "Running",
                        "unit": "Sync",
                        "result": null,
                        "unresolvedDDLLockID": "",
                        "sync": {
                            "masterBinlog": "(DESKTOP-T561TSO-bin.000001, 2388)",
                            "masterBinlogGtid": "143bdef3-dd4a-11ea-8b00-00155de45f57:1-10",
                            "syncerBinlog": "(DESKTOP-T561TSO-bin.000001, 2388)",
                            "syncerBinlogGtid": "143bdef3-dd4a-11ea-8b00-00155de45f57:1-4",
                            "blockingDDLs": [
                            ],
                            "unresolvedGroups": [
                            ],
                            "synced": true,
                            "binlogType": "remote",
                            "totalRows": "4",
                            "totalRps": "0",
                            "recentRps": "0"
                        }
                    }
                ]
            }
        ]
    }
    ```

    </details>
  ```
```
```
      + "Message": "detect inconsistent DDL sequence from source ... ddls: [ALTER TABLE `shard_db`.`tb` ADD COLUMN `new_col` INT UNIQUE KEY] source: `shard_db_2`.`shard_table_2`], right DDL sequence should be ..."
    }
    ```

3. Execute `handle-error <task-name> replace` again to replace the wrong DDL statements in MySQL instance 1 and 2:

    {{< copyable "" >}}

    ```bash
    » binlog replace test -s mysql-replica-01 "ALTER TABLE `shard_db_1`.`shard_table_2` ADD COLUMN `new_col` INT;ALTER TABLE `shard_db_1`.`shard_table_2` ADD UNIQUE(`new_col`)";
    ```

    ```
    {
        "result": true,
        "msg": "",
        "sources": [
            {
                "result": true,
                "msg": "",
                "source": "mysql-replica-01",
                "worker": "worker1"
            }
        ]
    }
    ```

    {{< copyable "" >}}

    ```bash
    » binlog replace test -s mysql-replica-02 "ALTER TABLE `shard_db_2`.`shard_table_2` ADD COLUMN `new_col` INT;ALTER TABLE `shard_db_2`.`shard_table_2` ADD UNIQUE(`new_col`)";
    ```

    ```
    {
        "result": true,
        "msg": "",
        "sources": [
            {
                "result": true,
                "msg": "",
                "source": "mysql-replica-02",
                "worker": "worker2"
            }
        ]
    }
    ```

4. Use `query-status <task-name>` to view the task status:

    {{< copyable "" >}}

    ```bash
    » query-status test
    ```

    <details><summary> See the execution result.</summary>

    ```
    {
        "result": true,
        "msg": "",
        "sources": [
            {
                "result": true,
                "msg": "",
                "sourceStatus": {
                    "source": "mysql-replica-01",
                    "worker": "worker1",
                    "result": null,
                    "relayStatus": null
                },
                "subTaskStatus": [
                    {
                        "name": "test",
                        "stage": "Running",
                        "unit": "Sync",
                        "result": null,
                        "unresolvedDDLLockID": "",
                        "sync": {
                            "masterBinlog": "(DESKTOP-T561TSO-bin.000001, 2388)",
                            "masterBinlogGtid": "143bdef3-dd4a-11ea-8b00-00155de45f57:1-10",
                            "syncerBinlog": "(DESKTOP-T561TSO-bin.000001, 2388)",
                            "syncerBinlogGtid": "143bdef3-dd4a-11ea-8b00-00155de45f57:1-4",
                            "blockingDDLs": [
                            ],
                            "unresolvedGroups": [
                            ],
                            "unresolvedGroups": [
                            ],
                            "synced": true,
                            "binlogType": "remote",
                            "totalRows": "4",
                            "totalRps": "0",
                            "recentRps": "0"
                        }
                    }
                ]
            },
            {
                "result": true,
                "msg": "",
                "sourceStatus": {
                    "source": "mysql-replica-02",
                    "worker": "worker2",
                    "result": null,
                    "relayStatus": null
                },
                "subTaskStatus": [
                    {
                        "name": "test",
                        "stage": "Running",
                        "unit": "Sync",
                        "result": null,
                        "unresolvedDDLLockID": "",
                        "sync": {
                            "masterBinlog": "(DESKTOP-T561TSO-bin.000001, 2388)",
                            "masterBinlogGtid": "143bdef3-dd4a-11ea-8b00-00155de45f57:1-10",
                            "syncerBinlog": "(DESKTOP-T561TSO-bin.000001, 2388)",
                            "syncerBinlogGtid": "143bdef3-dd4a-11ea-8b00-00155de45f57:1-4",
                            "blockingDDLs": [
                            ],
                            "unresolvedGroups": [
                            ],
                            "unresolvedGroups": [
                            ],
                            "synced": try,
                            "binlogType": "remote",
                            "totalRows": "4",
                            "totalRps": "0",
                            "recentRps": "0"
                        }
                    }
                ]
            }
        ]
    }
    ```

    </details>

    You can see that the task runs normally with no error and all four wrong DDL statements are replaced.

### Other commands

For the usage of other commands of `binlog`, refer to the `binlog skip` and `binlog replace` examples above.
```