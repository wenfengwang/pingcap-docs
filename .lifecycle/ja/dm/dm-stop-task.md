---
title: データ移行タスクを停止する
summary: データ移行タスクの停止方法について学びます。
---

# データ移行タスクの停止

`stop-task`コマンドを使用して、データ移行タスクを停止できます。`stop-task`と`pause-task`の違いについては、[データ移行タスクの一時停止](/dm/dm-pause-task.md)を参照してください。

{{< copyable "" >}}

```bash
help stop-task
```

```
指定されたタスクを停止します

使用法:
 dmctl stop-task [-s source ...] <task-name | task-file> [flags]

フラグ:
 -h, --help   stop-taskのヘルプ

グローバルフラグ：
 -s, --source strings   MySQLソースID
```

## 使用例

{{< copyable "" >}}

```bash
stop-task [-s "mysql-replica-01"]  task-name
```

## フラグの説明

- `-s`:（オプション）停止したい移行タスクのサブタスクが実行されているMySQLソースを指定します。設定されている場合、指定されたMySQLソース上のサブタスクのみが停止されます。
- `task-name | task-file`:（必須）タスク名またはタスクファイルのパスを指定します。

## 返される結果

{{< copyable "" >}}

```bash
stop-task test
```

```
{
    "op": "Stop",
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