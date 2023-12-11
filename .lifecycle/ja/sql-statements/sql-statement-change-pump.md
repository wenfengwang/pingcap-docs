---
title: ポンプの変更
summary: TiDBデータベースのCHANGE PUMPの使用方法の概要。
aliases: ['/docs/dev/sql-statements/sql-statement-change-pump/']
---

# CHANGE PUMP

`CHANGE PUMP`ステートメントは、クラスター内のPumpのステータス情報を変更します。

> **注意:**
>
> この機能はTiDB自己ホストのみに適用され、[TiDB Cloud](https://docs.pingcap.com/tidbcloud/)では利用できません。

> **ヒント:**
>
> Pumpの状態は実行中にPDに自動的に報告されます。Pumpが異常な状況にある場合、かつその状態がPDに格納されている状態情報と一致しない場合にのみ、`CHANGE PUMP`ステートメントを使用してPDに格納されている状態情報を変更できます。

## 例

{{< copyable "sql" >}}

```sql
SHOW PUMP STATUS;
```

```sql
+--------|----------------|--------|--------------------|---------------------|
| NodeID |     Address    | State  |   Max_Commit_Ts    |    Update_Time      |
+--------|----------------|--------|--------------------|---------------------|
| pump1  | 127.0.0.1:8250 | Online | 408553768673342237 | 2019-04-30 00:00:01 |
+--------|----------------|--------|--------------------|---------------------|
| pump2  | 127.0.0.2:8250 | Online | 408553768673342335 | 2019-05-01 00:00:02 |
+--------|----------------|--------|--------------------|---------------------|
2 rows in set (0.00 sec)
```

pump1の状態が1日以上更新されていないことがわかります。Pumpは異常な状態にありますが、`State`は`Online`のままです。`CHANGE PUMP`を使用した後、Pumpの`State`が'paused'に変更されます：

{{< copyable "sql" >}}

```sql
CHANGE PUMP TO NODE_STATE ='paused' FOR NODE_ID 'pump1';
```

```sql
Query OK, 0 rows affected (0.01 sec)
```

{{< copyable "sql" >}}

```sql
SHOW PUMP STATUS;
```

```sql
+--------|----------------|--------|--------------------|---------------------|
| NodeID |     Address    | State  |   Max_Commit_Ts    |    Update_Time      |
+--------|----------------|--------|--------------------|---------------------|
| pump1  | 127.0.0.1:8250 | Paused | 408553768673342237 | 2019-04-30 00:00:01 |
+--------|----------------|--------|--------------------|---------------------|
| pump2  | 127.0.0.2:8250 | Online | 408553768673342335 | 2019-05-01 00:00:02 |
+--------|----------------|--------|--------------------|---------------------|
2 rows in set (0.00 sec)
```

## MySQL互換性

このステートメントは、MySQL構文へのTiDBの拡張です。

## 関連項目

* [SHOW PUMP STATUS](/sql-statements/sql-statement-show-pump-status.md)
* [SHOW DRAINER STATUS](/sql-statements/sql-statement-show-drainer-status.md)
* [CHANGE DRAINER STATUS](/sql-statements/sql-statement-change-drainer.md)