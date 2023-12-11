---
title: シャットダウン
summary: TiDBデータベースのSHUTDOWNの使用方法の概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-shutdown/']
---

# シャットダウン

`SHUTDOWN`ステートメントはTiDBでシャットダウン操作を実行するために使用されます。 `SHUTDOWN`ステートメントを実行するには、ユーザーは`SHUTDOWN権限`を持っている必要があります。

> **注意:**
>
> この機能はTiDB Self-Hostedにのみ適用され、[TiDB Cloud](https://docs.pingcap.com/tidbcloud/)では使用できません。

## 概要

**ステートメント:**

![ステートメント](/media/sqlgram/ShutdownStmt.png)

## 例

{{< copyable "sql" >}}

```sql
SHUTDOWN;
```

```
クエリが実行されました、0行が変更されました (0.00 sec)
```

## MySQL互換性

> **注意:**
>
> TiDBは分散データベースであるため、TiDBでのシャットダウン操作は、クライアントに接続されたTiDBインスタンスを停止し、TiDBクラスタ全体を停止するものではありません。

`SHUTDOWN`ステートメントは部分的にMySQLと互換性があります。互換性の問題が発生した場合は、[バグを報告](https://docs.pingcap.com/tidb/stable/support)することができます。