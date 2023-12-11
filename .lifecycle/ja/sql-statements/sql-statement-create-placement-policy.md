---
title: プレイスメントポリシーの作成
summary: TiDBにおけるCREATE PLACEMENT POLICYの使用方法。

# プレイスメントポリシーの作成

`CREATE PLACEMENT POLICY`は、後でテーブル、パーティション、またはデータベーススキーマに割り当てることができる名前付きの配置ポリシーを作成するために使用されます。

> **注記:**
>
> この機能は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスターでは利用できません。

## 概要

```ebnf+diagram
CreatePolicyStmt ::=
    "CREATE" "PLACEMENT" "POLICY" IfNotExists PolicyName PlacementOptionList

PolicyName ::=
    Identifier

PlacementOptionList ::=
    PlacementOption
|   PlacementOptionList PlacementOption
|   PlacementOptionList ',' PlacementOption

PlacementOption ::=
    CommonPlacementOption
|   SugarPlacementOption
|   AdvancedPlacementOption

CommonPlacementOption ::=
    "FOLLOWERS" EqOpt LengthNum

SugarPlacementOption ::=
    "PRIMARY_REGION" EqOpt stringLit
|   "REGIONS" EqOpt stringLit
|   "SCHEDULE" EqOpt stringLit

AdvancedPlacementOption ::=
    "LEARNERS" EqOpt LengthNum
|   "CONSTRAINTS" EqOpt stringLit
|   "LEADER_CONSTRAINTS" EqOpt stringLit
|   "FOLLOWER_CONSTRAINTS" EqOpt stringLit
|   "LEARNER_CONSTRAINTS" EqOpt stringLit
|   "SURVIVAL_PREFERENCES" EqOpt stringLit
```

## 例

> **注記:**
>
> クラスターで利用可能なリージョンを知るには、[`SHOW PLACEMENT LABELS`](/sql-statements/sql-statement-show-placement-labels.md)を参照してください。
>
> 利用可能なリージョンが表示されない場合、TiKVのインストールが正しくラベルが設定されていない可能性があります。

{{< copyable "sql" >}}

```sql
CREATE PLACEMENT POLICY p1 PRIMARY_REGION="us-east-1" REGIONS="us-east-1,us-west-1" FOLLOWERS=4;
CREATE TABLE t1 (a INT) PLACEMENT POLICY=p1;
SHOW CREATE PLACEMENT POLICY p1;
```

```
クエリが正常に実行されました、0 行が影響を受けました (0.08 sec)

クエリが正常に実行されました、0 行が影響を受けました (0.10 sec)

+--------+---------------------------------------------------------------------------------------------------+
| Policy | Create Policy                                                                                     |
+--------+---------------------------------------------------------------------------------------------------+
| p1     | CREATE PLACEMENT POLICY `p1` PRIMARY_REGION="us-east-1" REGIONS="us-east-1,us-west-1" FOLLOWERS=4 |
+--------+---------------------------------------------------------------------------------------------------+
1 行がセットされました (0.00 sec)
```

## MySQL互換性

このステートメントは、MySQL構文のTiDB拡張機能です。

## 関連項目

* [SQLにおける配置ルール](/placement-rules-in-sql.md)
* [PLACEMENTの表示](/sql-statements/sql-statement-show-placement.md)
* [PLACEMENT POLICYの変更](/sql-statements/sql-statement-alter-placement-policy.md)
* [PLACEMENT POLICYの削除](/sql-statements/sql-statement-drop-placement-policy.md)