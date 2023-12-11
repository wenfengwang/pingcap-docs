---
title: SHOW CONFIG
summary: TiDBデータベースでのSHOW CONFIGの使用の概要
aliases: ['/docs/dev/sql-statements/sql-statement-show-config/']
---

# SHOW CONFIG

`SHOW CONFIG`ステートメントはTiDBの様々なコンポーネントの現在の構成を表示するために使用されます。構成とシステム変数は異なるディメンションで動作し、混同されるべきではありません。システム変数情報を取得したい場合は、[SHOW VARIABLES](/sql-statements/sql-statement-show-variables.md)の構文を使用してください。

> **ノート:**
>
> この機能はTiDB Self-Hostedにのみ適用され、[TiDB Cloud](https://docs.pingcap.com/tidbcloud/)では利用できません。

## 概要

**ShowStmt:**

![ShowStmt](/media/sqlgram/ShowStmt.png)

**ShowTargetFilterable:**

![ShowTargetFilterable](/media/sqlgram/ShowTargetFilterable.png)

## 例

すべての構成を表示:

{{< copyable "sql" >}}

```sql
SHOW CONFIG;
```

```
+------+----------------+-------------------------------------------------+---------------------------------------------------------------------+
| Type | Instance       | Name                                            | Value                                                               |
+------+----------------+-------------------------------------------------+---------------------------------------------------------------------+
| tidb | 127.0.0.1:4000 | advertise-address                               | 127.0.0.1                                                           |
| tidb | 127.0.0.1:4000 | binlog.binlog-socket                            |                                                                     |
| tidb | 127.0.0.1:4000 | binlog.enable                                   | false                                                               |
...
120 rows in set (0.01 sec)
```

`type`が`tidb`である構成を表示:

{{< copyable "sql" >}}

```sql
SHOW CONFIG WHERE type = 'tidb' AND name = 'advertise-address';
```

```
+------+----------------+-------------------+-----------+
| Type | Instance       | Name              | Value     |
+------+----------------+-------------------+-----------+
| tidb | 127.0.0.1:4000 | advertise-address | 127.0.0.1 |
+------+----------------+-------------------+-----------+
1 row in set (0.05 sec)
```

`type`が`tidb`である構成を`LIKE`句を使用して表示することもできます:

{{< copyable "sql" >}}

```sql
SHOW CONFIG LIKE 'tidb';
```

```
+------+----------------+-------------------------------------------------+---------------------------------------------------------------------+
| Type | Instance       | Name                                            | Value                                                               |
+------+----------------+-------------------------------------------------+---------------------------------------------------------------------+
| tidb | 127.0.0.1:4000 | advertise-address                               | 127.0.0.1                                                           |
| tidb | 127.0.0.1:4000 | binlog.binlog-socket                            |                                                                     |
| tidb | 127.0.0.1:4000 | binlog.enable                                   | false                                                               |
...
40 rows in set (0.01 sec)
```

## MySQL互換性

このステートメントはMySQL構文のTiDBの拡張です。

## 関連情報

* [SHOW VARIABLES](/sql-statements/sql-statement-show-variables.md)