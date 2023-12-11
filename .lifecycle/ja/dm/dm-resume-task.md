---
title: データ移行タスクを再開する
summary: データ移行タスクを再開する方法を学びます。
---

# データ移行タスクを再開する

`resume-task`コマンドを使用して、`一時停止`状態のデータ移行タスクを再開できます。これは、タスクを一時停止させたエラーを処理した後に、手動でデータ移行タスクを再開したい場合に一般的に使用されます。

{{< copyable "" >}}

```bash
help resume-task
```

```
指定された一時停止状態のタスクを再開します

使用法:
 dmctl resume-task [-s source ...] <task-name | task-file> [flags]

フラグ:
 -h、--help   resume-taskのヘルプ

グローバルフラグ:
 -s、--source strings   MySQLソースID
```

## 使用例

{{< copyable "" >}}

```bash
resume-task [-s "mysql-replica-01"] タスク名
```

## フラグの説明

- `-s`: (オプション) データ移行タスクのサブタスクを再開したいMySQLソースを指定します。指定された場合、コマンドは指定されたMySQLソース上のサブタスクのみを再開します。
- `task-name | task-file`: (必須) タスク名またはタスクファイルパスを指定します。

## 返される結果

{{< copyable "" >}}

```bash
resume-task test
```

```
{
    "op": "再開",
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