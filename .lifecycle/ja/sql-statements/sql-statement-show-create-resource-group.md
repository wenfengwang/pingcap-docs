---
title: SHOW CREATE RESOURCE GROUP
summary: TiDBでのSHOW CREATE RESOURCE GROUPの使用方法を学ぶ。

# SHOW CREATE RESOURCE GROUP

`SHOW CREATE RESOURCE GROUP`ステートメントを使用して、リソースグループの現在の定義を表示できます。

> **注意:**
>
> この機能は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスタでは利用できません。

## 概要

```ebnf+diagram
ShowCreateResourceGroupStmt ::=
    "SHOW" "CREATE" "RESOURCE" "GROUP" ResourceGroupName

ResourceGroupName ::=
    Identifier
|   "DEFAULT"
```

## 例

リソースグループ `rg1`を作成します。

```sql
CREATE RESOURCE GROUP rg1 RU_PER_SEC=100;
Query OK, 0 rows affected (0.10 sec)
```

`rg1`の定義を表示します。

```sql
SHOW CREATE RESOURCE GROUP rg1;
***************************[ 1. row ]***************************
+----------------+------------------------------------------------------------+
| Resource_Group | Create Resource Group                                      |
+----------------+------------------------------------------------------------+
| rg1            | CREATE RESOURCE GROUP `rg1` RU_PER_SEC=100 PRIORITY=MEDIUM |
+----------------+------------------------------------------------------------+
1 row in set (0.01 sec)
```

## MySQL互換性

このステートメントはMySQL用のTiDBの拡張機能です。

## 関連情報

* [TiDB RESOURCE CONTROL](/tidb-resource-control.md)
* [CREATE RESOURCE GROUP](/sql-statements/sql-statement-alter-resource-group.md)
* [ALTER RESOURCE GROUP](/sql-statements/sql-statement-alter-resource-group.md)
* [DROP RESOURCE GROUP](/sql-statements/sql-statement-drop-resource-group.md)