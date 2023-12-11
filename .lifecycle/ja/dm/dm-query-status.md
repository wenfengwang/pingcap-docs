---
title: TiDBデータ移行でのタスクステータスのクエリ
summary: データレプリケーションタスクのステータスをクエリする方法について学びます。
aliases: ['/docs/tidb-data-migration/dev/query-status/']
---

# TiDBデータ移行でのタスクステータスのクエリ

このドキュメントでは、DMの`query-status`コマンドを使用して、タスクのステータスやサブタスクのステータスをクエリする方法について紹介します。

## クエリ結果

{{< copyable "" >}}

```bash
» query-status
```

```
{
    "result": true,     # クエリが成功したかどうか。
    "msg": "",          # クエリが失敗した理由を説明します。
    "tasks": [          # 移行タスク一覧。
        {
            "taskName": "test",         # タスク名。
            "taskStatus": "Running",    # タスクのステータス。
            "sources": [                # 上流MySQL一覧。
                "mysql-replica-01",
                "mysql-replica-02"
            ]
        },
        {
            "taskName": "test2",
            "taskStatus": "Paused",
            "sources": [
                "mysql-replica-01",
                "mysql-replica-02"
            ]
        }
    ]
}
```

`tasks`セクションの`taskStatus`の詳細な説明については、「[タスクステータス](#task-status)」を参照してください。

以下の手順に従って`query-status`を使用することをお勧めします：

1. すべての進行中のタスクが正常な状態にあるかどうかを確認するために`query-status`を使用します。
2. タスクでエラーが発生した場合は、`query-status <taskName>`コマンドを使用して詳細なエラー情報を確認してください。このコマンドの`<taskName>`はエラーに遭遇したタスクの名前を指定します。

## タスクステータス

DM移行タスクのステータスは、DM-workerに割り当てられた各サブタスクのステータスに依存します。サブタスクステータスの詳細については、「[サブタスクステータス](#subtask-status)」を参照してください。以下の表は、サブタスクステータスがタスクステータスにどのように関連するかを示しています。

|  タスクのサブタスクステータス | タスクステータス |
| :--- | :--- |
| サブタスクが`paused`状態でエラー情報が返される。 | `エラー - サブタスクでエラーが発生しました` |
| Syncフェーズの1つのサブタスクが`Running`状態ですが、そのRelay処理ユニットが実行されていません(`Error`/`Paused`/`Stopped`状態)。 | `エラー - Relayの状態はエラー/Paused/Stoppedです` |
| サブタスクが`Paused`状態でエラー情報が返されない。 | `Paused` |
| すべてのサブタスクが`New`状態。 | `New` |
| すべてのサブタスクが`Finished`状態。 | `Finished` |
| すべてのサブタスクが`Stopped`状態。 | `Stopped` |
| その他の状況 | `Running` |

## 詳細なクエリ結果

{{< copyable "" >}}

```bash
» query-status test
```

```
» query-status
{
    "result": true,     # クエリが成功したかどうか。
    "msg": "",          # クエリが失敗した原因を説明します。
    "sources": [                            # 上流MySQL一覧。
        {
            "result": true,
            "msg": "",
            "sourceStatus": {                   # 上流MySQLデータベースの情報。
                "source": "mysql-replica-01",
                "worker": "worker1",
                "result": null,
                "relayStatus": null
            },
            "subTaskStatus": [              # 上流MySQLデータベースのすべてのサブタスクの情報。
                {
                    "name": "test",         # サブタスクの名前。
                    "stage": "Running",     # サブタスクの実行ステータス（"New"、"Running"、"Paused"、"Stopped"、"Finished"を含む）。
                    "unit": "Sync",         # DMの処理ユニット（"Check"、"Dump"、"Load"、"Sync"を含む）。
                    "result": null,         # サブタスクが失敗した場合はエラー情報を表示します。
                    "unresolvedDDLLockID": "test-`test`.`t_target`",    # シャーディングDDLロックID。異常な状態でシャーディングDDLロックを手動で処理するために使用します。
                    "sync": {                   # "Sync"処理ユニットの複製情報。この情報は現在の処理ユニットと同じコンポーネントに関するものです。
                        "masterBinlog": "(bin.000001, 3234)",                               # 上流データベースのbinlog位置。
                        "masterBinlogGtid": "c0149e17-dff1-11e8-b6a8-0242ac110004:1-14",    # 上流データベースのGTID情報。
                        "syncerBinlog": "(bin.000001, 2525)",                               # "Sync"処理ユニットで複製されたbinlogの位置。
                        "syncerBinlogGtid": "",                                             # GTIDを使用して複製されたbinlog位置。
                        "blockingDDLs": [       # 現在ブロックされているDDLリスト。すべての上流テーブルが"synced"状態にある場合にのみ空ではありません。この場合、実行またはスキップするシャーディングDDLステートメントを示します。
                            "USE `test`; ALTER TABLE `test`.`t_target` DROP COLUMN `age`;"
                        ],
                        "unresolvedGroups": [   # 解決されていないシャーディンググループ。
                            {
                                "target": "`test`.`t_target`",                  # 複製される下流データベースのテーブル。
                                "DDLs": [
                                    "USE `test`; ALTER TABLE `test`.`t_target` DROP COLUMN `age`;"
                                ],
                                "firstPos": "(bin|000001.000001, 3130)",        # シャーディングDDLステートメントの開始位置。
                                "synced": [                                     # "Sync"ユニットによって実行されたシャーディングDDLステートメントを読み取った上流シャーディングテーブル。
                                    "`test`.`t2`"
                                    "`test`.`t3`"
                                    "`test`.`t1`"
                                ],
                                "unsynced": [                                   # このシャーディングDDLが実行されていない上流テーブル。上流テーブルに未完成の複製がある場合、`blockingDDLs`は空です。
                                ]
                            }
                        ],
                        "synced": false         # 増分複製が上流に追いつき、上流と同じbinlog位置になったかどうか。`Sync`バックグラウンドではリアルタイムでセーブポイントが更新されないため、`synced`の`false`が常に複製遅延が発生していることを意味するわけではありません。
                        "totalRows": "12",      # このサブタスクで複製された合計行数。
                        "totalRps": "1",        # このサブタスクでの1秒あたりの複製行数。
                        "recentRps": "1"        # このサブタスクでの最後の1秒間の複製行数。
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
                    "unit": "Load",
                    "result": null,
                    "unresolvedDDLLockID": "",
                    "load": {                   # "Load"処理ユニットの複製情報。
                        "finishedBytes": "115",          # 複製されたバイト数。
                        "totalBytes": "452",               # 複製する必要のある合計バイト数。
                        "progress": "25.44 %",         # 読み込みプロセスの進捗。
                        "bps": "2734"                        # 完全読み込みのスピード。
                    }
                }
            ]
        },
        {
            "result": true,
            "sourceStatus": {
                "source": "mysql-replica-03",
                "worker": "worker3",
                "result": null,
                "relayStatus": null
            },
            "subTaskStatus": [
                {
                    "name": "test",
                    "stage": "Paused",
                    "unit": "Load",
                    "result": {                 # エラーの例。
                        "isCanceled": false,
                        "errors": [
                            {
                                "Type": "ExecSQL",
                                "msg": "Error 1062: Duplicate entry '1155173304420532225' for key 'PRIMARY'\n/home/jenkins/workspace/build_dm/go/src/github.com/pingcap/tidb-enterprise-tools/loader/db.go:160: \n/home/jenkins/workspace/build_dm/go/src/github.com/pingcap/tidb-enterprise-tools/loader/db.go:105: \n/home/jenkins/workspace/build_dm/go/src/github.com/pingcap/tidb-enterprise-tools/loader/loader.go:138: file test.t1.sql"
                            }
                        ],
                        "detail": null
                    },
                    "unresolvedDDLLockID": "",
                    "load": {
                        "finishedBytes": "0",
                        "totalBytes": "156",
                        "progress": "0.00 %",
                        "bps": "0"
                    }
                }
            ]
        },
        {
            "result": true,
            "msg": "",
```json
"sourceStatus": {
    "source": "mysql-replica-04",
    "worker": "worker4",
    "result": null,
    "relayStatus": null
},
"subTaskStatus": [
    {
        "name": "test",
        "stage": "Running",
        "unit": "Dump",
        "result": null,
        "unresolvedDDLLockID": "",
        "dump": {                        # The replication information of the `Dump` processing unit.
            "totalTables": "10",         # The number of tables to be dumped.
            "completedTables": "3",      # The number of tables that have been dumped.
            "finishedBytes": "2542",     # The number of bytes that have been dumped.
            "finishedRows": "32",        # The number of rows that have been dumped.
            "estimateTotalRows": "563",  # The estimated number of rows to be dumped.
            "progress": "30.52 %",       # The progress of the dumping process.
            "bps": "445"                 # The dumping speed.
        }
    }
]
},
]
```

For the status description and status switch relationship of "stage" of "subTaskStatus" of "sources", see the [subtask status](#subtask-status).

For operation details of "unresolvedDDLLockID" of "subTaskStatus" of "sources", see [Handle Sharding DDL Locks Manually](/dm/manually-handling-sharding-ddl-locks.md).

## Subtask status

### Status description

- `New`:

    - The initial status.
    - If the subtask does not encounter an error, it is switched to `Running`; otherwise it is switched to `Paused`.

- `Running`: The normal running status.

- `Paused`:

    - The paused status.
    - If the subtask encounters an error, it is switched to `Paused`.
    - If you run `pause-task` when the subtask is in the `Running` status, the task is switched to `Paused`.
    - When the subtask is in this status, you can run the `resume-task` command to resume the task.

- `Stopped`:

    - The stopped status.
    - If you run `stop-task` when the subtask is in the `Running` or `Paused` status, the task is switched to `Stopped`.
    - When the subtask is in this status, you cannot use `resume-task` to resume the task.

- `Finished`:

    - The finished subtask status.
    - Only when the full replication subtask is finished normally, the task is switched to this status.

### Status switch diagram

```
                                         エラーが発生
                            新規 --------------------------------|
                             |                                  |
                             |           resume-task            |
                             |  |----------------------------|  |
                             |  |                            |  |
                             |  |                            |  |
                             v  v        エラーが発生         |  v
  完了 <-------------- 実行中 ----------------------------> 一時停止
                             ^  |          または pause-task |
                             |  |                            |
                  タスクの開始 |  | タスクの停止             |
                             |  |                            |
                             |  v       タスクの停止         |
                           停止 <--------------------------- |
```