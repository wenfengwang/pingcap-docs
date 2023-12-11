---
title: CHANGE DRAINER
summary: TiDBデータベースのCHANGE DRAINERの使用方法の概要。
aliases: ['/docs/dev/sql-statements/sql-statement-change-drainer/']
---

# CHANGE DRAINER

`CHANGE DRAINER`ステートメントは、クラスタ内のDrainerのステータス情報を変更します。

> **ノート：**
>
> この機能はTiDB Self-Hostedにのみ適用され、[TiDB Cloud](https://docs.pingcap.com/tidbcloud/)では使用できません。

> **ヒント：**
>
> Drainerの状態は実行中に自動的にPDに報告されます。 Drainerが異常な状態にあり、その状態がPDに格納されている状態情報と一貫していない場合にのみ、`CHANGE DRAINER`ステートメントを使用して、PDに格納されている状態情報を変更できます。

## 例

{{< copyable "sql" >}}

```sql
SHOW DRAINER STATUS;
```

```sql
+----------|----------------|--------|--------------------|---------------------|
|  NodeID  |     Address    | State  |   Max_Commit_Ts    |    Update_Time      |
+----------|----------------|--------|--------------------|---------------------|
| drainer1 | 127.0.0.3:8249 | Online | 408553768673342532 | 2019-04-30 00:00:03 |
+----------|----------------|--------|--------------------|---------------------|
| drainer2 | 127.0.0.4:8249 | Online | 408553768673345531 | 2019-05-01 00:00:04 |
+----------|----------------|--------|--------------------|---------------------|
2 rows in set (0.00 sec)
```

drainer1の状態は1日以上更新されていないことがわかります。Drainerは異常な状態にありますが、`State`は`Online`のままです。`CHANGE DRAINER`を使用した後、Drainerの`State`が 'paused' に変更されます：

{{< copyable "sql" >}}

```sql
CHANGE DRAINER TO NODE_STATE ='paused' FOR NODE_ID 'drainer1';
```

```sql
Query OK, 0 rows affected (0.01 sec)
```

{{< copyable "sql" >}}

```sql
SHOW DRAINER STATUS;
```

```sql
+----------|----------------|--------|--------------------|---------------------|
|  NodeID  |     Address    | State  |   Max_Commit_Ts    |    Update_Time      |
+----------|----------------|--------|--------------------|---------------------|
| drainer1 | 127.0.0.3:8249 | Paused | 408553768673342532 | 2019-04-30 00:00:03 |
+----------|----------------|--------|--------------------|---------------------|
| drainer2 | 127.0.0.4:8249 | Online | 408553768673345531 | 2019-05-01 00:00:04 |
+----------|----------------|--------|--------------------|---------------------|
2 rows in set (0.00 sec)
```

## MySQL互換性

このステートメントは、MySQL構文へのTiDB拡張です。

## 関連項目

* [SHOW PUMP STATUS](/sql-statements/sql-statement-show-pump-status.md)
* [SHOW DRAINER STATUS](/sql-statements/sql-statement-show-drainer-status.md)
* [CHANGE PUMP STATUS](/sql-statements/sql-statement-change-pump.md)