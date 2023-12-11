---
title: SHOW [BACKUPS|RESTORES] | TiDB SQLステートメントリファレンス
summary: TiDBデータベースのSHOW [BACKUPS|RESTORES]の使用法の概要。
aliases: ['/docs/dev/sql-statements/sql-statement-show-backups/']
---

# SHOW [BACKUPS|RESTORES]

これらのステートメントは、TiDBインスタンスで実行されたすべての待機中、実行中、最近完了した [`BACKUP`](/sql-statements/sql-statement-backup.md) および [`RESTORE`](/sql-statements/sql-statement-restore.md) タスクのリストを表示します。

両方のステートメントを実行するには `SUPER` 権限が必要です。

`SHOW BACKUPS` を使用して `BACKUP` タスクを照会し、`SHOW RESTORES` を使用して `RESTORE` タスクを照会します。

> **注意:**
>
> この機能は [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) クラスターでは利用できません。

`br` コマンドラインツールで開始されたバックアップおよびリストアは表示されません。

## 概要

```ebnf+diagram
ShowBRIEStmt ::=
    "SHOW" ("BACKUPS" | "RESTORES") ShowLikeOrWhere?

ShowLikeOrWhere ::=
    "LIKE" SimpleExpr
|   "WHERE" Expression
```

## 例

1つの接続で、次のステートメントを実行します:

{{< copyable "sql" >}}

```sql
BACKUP DATABASE `test` TO 's3://example-bucket/backup-01';
```

バックアップが完了する前に、新しい接続で `SHOW BACKUPS` を実行します:

{{< copyable "sql" >}}

```sql
SHOW BACKUPS;
```

```sql
+--------------------------------+---------+----------+---------------------+---------------------+-------------+------------+---------+
| Destination                    | State   | Progress | Queue_time          | Execution_time      | Finish_time | Connection | Message |
+--------------------------------+---------+----------+---------------------+---------------------+-------------+------------+---------+
| s3://example-bucket/backup-01/ | Backup  | 98.38    | 2020-04-12 23:09:03 | 2020-04-12 23:09:25 |        NULL |          4 | NULL    |
+--------------------------------+---------+----------+---------------------+---------------------+-------------+------------+---------+
1 row in set (0.00 sec)
```

上記の結果の最初の行は、次のように説明されます:

| 列 | 説明 |
| :-------- | :--------- |
| `Destination` | 宛先URL（すべてのパラメータが秘密のキーを漏洩させないように削除されています） |
| `State` | タスクの状態 |
| `Progress` | 現在の状態での推定進捗率 |
| `Queue_time` | タスクが待機状態になった時刻 |
| `Execution_time` | タスクが開始された時刻；待機中のタスクでは値が `0000-00-00 00:00:00` です |
| `Finish_time` | タスクが完了したときのタイムスタンプ；待機中および実行中のタスクでは値が `0000-00-00 00:00:00` です |
| `Connection` | このタスクを実行している接続ID |
| `Message` | 詳細メッセージ |

可能な状態は次のとおりです:

| 状態 | 説明 |
| :-----|:------------|
| Backup | バックアップ作成中 |
| Wait | 実行待ち |
| Checksum | チェックサム操作実行中 |

接続IDは、[`KILL TIDB QUERY`](/sql-statements/sql-statement-kill.md) ステートメントを使用して、バックアップ/リストアタスクをキャンセルするために使用できます。

{{< copyable "sql" >}}

```sql
KILL TIDB QUERY 4;
```

```sql
Query OK, 0 rows affected (0.00 sec)
```

### フィルタリング

`LIKE` 句を使用して、宛先URLをワイルドカード式と一致させることでタスクをフィルタリングします。

{{< copyable "sql" >}}

```sql
SHOW BACKUPS LIKE 's3://%';
```

列でフィルタリングするには `WHERE` 句を使用します。

{{< copyable "sql" >}}

```sql
SHOW BACKUPS WHERE `Progress` < 25.0;
```

## MySQL互換性

このステートメントは、MySQL構文のTiDB拡張です。

## 関連情報

* [BACKUP](/sql-statements/sql-statement-backup.md)
* [RESTORE](/sql-statements/sql-statement-restore.md)