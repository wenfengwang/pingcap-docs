---
title: SHOW DRAINER STATUS
summary: SHOW DRAINER STATUSのTiDBデータベースの使用方法の概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-show-drainer-status/']
---

# SHOW DRAINER STATUS

`SHOW DRAINER STATUS`ステートメントは、クラスタ内のすべてのDrainerノードのステータス情報を表示します。

> **注意:**
>
> この機能はTiDBセルフホストにのみ適用され、[TiDB Cloud](https://docs.pingcap.com/tidbcloud/)では利用できません。

## 例

{{< copyable "sql" >}}

```sql
SHOW DRAINER STATUS;
```

```sql
+----------|----------------|--------|--------------------|---------------------|
|  NodeID  |     Address    | State  |   Max_Commit_Ts    |    Update_Time      |
+----------|----------------|--------|--------------------|---------------------|
| drainer1 | 127.0.0.3:8249 | Online | 408553768673342532 | 2019-05-01 00:00:03 |
+----------|----------------|--------|--------------------|---------------------|
| drainer2 | 127.0.0.4:8249 | Online | 408553768673345531 | 2019-05-01 00:00:04 |
+----------|----------------|--------|--------------------|---------------------|
2 rows in set (0.00 sec)
```

## MySQL互換性

このステートメントはMySQL構文のTiDB拡張です。

## 関連項目

* [SHOW PUMP STATUS](/sql-statements/sql-statement-show-pump-status.md)
* [CHANGE PUMP STATUS](/sql-statements/sql-statement-change-pump.md)
* [CHANGE DRAINER STATUS](/sql-statements/sql-statement-change-drainer.md)