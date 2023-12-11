---
title: PUMPの状態を表示
summary: TiDBデータベースでのSHOW PUMP STATUSの使用方法の概要。
aliases: ['/docs/dev/sql-statements/sql-statement-show-pump-status/']
---

# PUMPの状態を表示

`SHOW PUMP STATUS`ステートメントは、クラスタ内のすべてのPumpノードの状態情報を表示します。

> **注意:**
>
> この機能はTiDBセルフホストにのみ適用され、[TiDB Cloud](https://docs.pingcap.com/tidbcloud/)では利用できません。

## 例

{{< copyable "sql" >}}

```sql
SHOW PUMP STATUS;
```

```sql
+--------|----------------|--------|--------------------|---------------------|
| NodeID |     Address    | State  |   Max_Commit_Ts    |    Update_Time      |
+--------|----------------|--------|--------------------|---------------------|
| pump1  | 127.0.0.1:8250 | Online | 408553768673342237 | 2019-05-01 00:00:01 |
+--------|----------------|--------|--------------------|---------------------|
| pump2  | 127.0.0.2:8250 | Online | 408553768673342335 | 2019-05-01 00:00:02 |
+--------|----------------|--------|--------------------|---------------------|
2 行が見つかりました (0.00 秒)
```

## MySQL互換性

このステートメントはMySQL構文に対するTiDBの拡張です。

## 関連項目

* [SHOW DRAINER STATUS](/sql-statements/sql-statement-show-drainer-status.md)
* [CHANGE PUMP STATUS](/sql-statements/sql-statement-change-pump.md)
* [CHANGE DRAINER STATUS](/sql-statements/sql-statement-change-drainer.md)