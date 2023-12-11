---
title: データ移行タスクを一時停止する
summary: TiDBデータ移行においてデータ移行タスクを一時停止する方法を学びます。

# データ移行タスクを一時停止する

`pause-task`コマンドを使用してデータ移行タスクを一時停止することができます。

`pause-task`は`stop-task`とは異なります:

- `pause-task`はデータ移行タスクを一時停止するだけです。`query-status`を使用してタスクの状態情報（メモリに保持されている）をクエリできます。`stop-task`はデータ移行タスクを終了し、このタスクに関連するすべての情報をメモリから削除します。つまり、`query-status`を使用して状態情報をクエリすることはできません。チェックポイントなどの"dm_meta"や、下流に移行されたデータは削除されません。
- `pause-task`を実行して移行タスクを一時停止した場合、同じ名前の新しいタスクを開始することはできません。また、一時停止されたタスクのリレーログは削除されません。一方、`stop-task`を実行してタスクを停止した場合、同じ名前の新しいタスクを開始でき、停止されたタスクのリレーログは削除されます。
- 通常、`pause-task`はトラブルシューティングのためにタスクを一時停止するために使用されます。一方、`stop-task`は永続的にデータ移行タスクを削除するか、`start-task`と協力して構成情報を更新するために使用されます。

{{< copyable "" >}}

```bash
help pause-task
```

```
指定した実行中のタスクを一時停止する

使用法:
 dmctl pause-task [-s source ...] <task-name | task-file> [flags]

フラグ:
 -h、--help   pause-taskのヘルプを表示する

グローバルフラグ:
 -s、--source strings   MySQLソースID
```

## 使用例

{{< copyable "" >}}

```bash
pause-task [-s "mysql-replica-01"] task-name
```

## フラグの説明

- `-s`:（オプション）移行タスクのサブタスクを一時停止するMySQLソースを指定します。設定されている場合、このコマンドは指定されたMySQLソース上でのサブタスクのみを一時停止します。
- `task-name | task-file`:（必須）タスク名またはタスクファイルのパスを指定します。

## 返される結果

{{< copyable "" >}}

```bash
pause-task test
```

```
{
    "op": "Pause",
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