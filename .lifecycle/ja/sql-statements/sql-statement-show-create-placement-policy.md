---
title: PLACEMENT POLICYの表示
summary: TiDBにおけるSHOW CREATE PLACEMENT POLICYの使用法。

# PLACEMENT POLICYの表示

`SHOW CREATE PLACEMENT POLICY`は、配置ポリシーの定義を表示するために使用されます。現在の配置ポリシーの定義を確認し、別のTiDBクラスターで再作成する際に使用することができます。

> **注意:**
>
> この機能は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスターでは利用できません。

## 概要

```ebnf+diagram
ShowCreatePlacementPolicyStmt ::=
    "SHOW" "CREATE" "PLACEMENT" "POLICY" ポリシー名

PolicyName ::=
    識別子
```

## 例

{{< copyable "sql" >}}

```sql
CREATE PLACEMENT POLICY p1 PRIMARY_REGION="us-east-1" REGIONS="us-east-1,us-west-1" FOLLOWERS=4;
CREATE TABLE t1 (a INT) PLACEMENT POLICY=p1;
SHOW CREATE PLACEMENT POLICY p1\G;
```

```
クエリが成功しました。0件の行が影響を受けました (0.08秒)

クエリが成功しました。0件の行が影響を受けました (0.10秒)

***************************[ 1. 行 ]***************************
ポリシー      | p1
ポリシー作成  | CREATE PLACEMENT POLICY `p1` PRIMARY_REGION="us-east-1" REGIONS="us-east-1,us-west-1" FOLLOWERS=4
1 行が選択されました (0.00秒)
```

## MySQL互換性

この文は、MySQLの構文に対するTiDBの拡張です。

## 関連項目

* [SQLにおける配置ルール](/placement-rules-in-sql.md)
* [SHOW PLACEMENT](/sql-statements/sql-statement-show-placement.md)
* [CREATE PLACEMENT POLICY](/sql-statements/sql-statement-create-placement-policy.md)
* [ALTER PLACEMENT POLICY](/sql-statements/sql-statement-alter-placement-policy.md)
* [DROP PLACEMENT POLICY](/sql-statements/sql-statement-drop-placement-policy.md)