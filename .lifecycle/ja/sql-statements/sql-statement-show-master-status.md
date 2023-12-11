---
title: SHOW MASTER STATUS
summary: TiDBデータベースのSHOW MASTER STATUSの使用概要。
aliases: ['/docs/dev/sql-statements/sql-statement-show-master-status/']
---

# SHOW MASTER STATUS

`SHOW MASTER STATUS`ステートメントは、クラスター内の最新のTSOを表示します。

## 例

{{< copyable "sql" >}}

```sql
SHOW MASTER STATUS;
```

```sql
+-------------+--------------------+--------------+------------------+-------------------+
| File        | Position           | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+-------------+--------------------+--------------+------------------+-------------------+
| tidb-binlog | 416916363252072450 |              |                  |                   |
+-------------+--------------------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

## MySQL互換性

`SHOW MASTER STATUS`の出力は、MySQLに合わせて設計されています。ただし、実行結果は異なり、MySQLの結果はbinlogの位置情報であり、TiDBの結果は最新のTSO情報です。

`SHOW BINARY LOG STATUS`ステートメントは、MySQL 8.2.0以降で廃止された`SHOW MASTER STATUS`のエイリアスとしてTiDBに追加されました。

## 関連項目

<CustomContent platform="tidb">

* [SHOW PUMP STATUS](/sql-statements/sql-statement-show-pump-status.md)
* [SHOW DRAINER STATUS](/sql-statements/sql-statement-show-drainer-status.md)
* [CHANGE PUMP STATUS](/sql-statements/sql-statement-change-pump.md)
* [CHANGE DRAINER STATUS](/sql-statements/sql-statement-change-drainer.md)

</CustomContent>

<CustomContent platform="tidb-cloud">

* [`SHOW TABLE STATUS`](/sql-statements/sql-statement-show-table-status.md)

</CustomContent>