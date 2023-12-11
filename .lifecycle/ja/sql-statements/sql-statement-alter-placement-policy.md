---
title: プレースメントポリシーの変更
summary: TiDBでのALTER PLACEMENT POLICYの使用方法。

# ALTER PLACEMENT POLICY

`ALTER PLACEMENT POLICY`は以前に作成された既存のプレースメントポリシーを変更するために使用されます。プレースメントポリシーを使用しているすべてのテーブルとパーティションは自動的に更新されます。

> **注意:**
>
> この機能は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスタでは利用できません。

`ALTER PLACEMENT POLICY`は以前のポリシーを新しい定義で_置き換えます_。以前のポリシーを新しいものと_マージする_のではありません。次の例では、`FOLLOWERS=4`は`ALTER PLACEMENT POLICY`が実行されると失われます：

```sql
CREATE PLACEMENT POLICY p1 FOLLOWERS=4;
ALTER PLACEMENT POLICY p1 PRIMARY_REGION="us-east-1" REGIONS="us-east-1,us-west-1";
```

## 概要

```ebnf+diagram
AlterPolicyStmt ::=
    "ALTER" "PLACEMENT" "POLICY" IfExists PolicyName PlacementOptionList

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

> **注意:**
>
> クラスタで利用可能なリージョンを確認するには、[`SHOW PLACEMENT LABELS`](/sql-statements/sql-statement-show-placement-labels.md)を参照してください。
>
> 利用可能なリージョンが表示されない場合は、TiKVのインストールにラベルが正しく設定されていない可能性があります。

{{< copyable "sql" >}}

```sql
CREATE PLACEMENT POLICY p1 PRIMARY_REGION="us-east-1" REGIONS="us-east-1,us-west-1";
CREATE TABLE t1 (i INT) PLACEMENT POLICY=p1; -- テーブルt1にポリシーp1を割り当てる
ALTER PLACEMENT POLICY p1 PRIMARY_REGION="us-east-1" REGIONS="us-east-1,us-west-1,us-west-2" FOLLOWERS=4; -- t1のルールが自動的に更新されます
SHOW CREATE PLACEMENT POLICY p1\G;
```

```
クエリが実行されました。変更された行: 0 (0.08秒)

クエリが実行されました。変更された行: 0 (0.10秒)

***************************[ 1. 行 ]***************************
Policy        | p1
Create Policy | CREATE PLACEMENT POLICY `p1` PRIMARY_REGION="us-east-1" REGIONS="us-east-1,us-west-1,us-west-2" FOLLOWERS=4
1 行が選択されました (0.00秒)
```

## MySQL互換性

このステートメントは、MySQL構文にTiDB拡張です。

## 関連項目

* [SQLにおけるプレースメントルール](/placement-rules-in-sql.md)
* [PLACEMENTの表示](/sql-statements/sql-statement-show-placement.md)
* [CREATE PLACEMENT POLICY](/sql-statements/sql-statement-create-placement-policy.md)
* [DROP PLACEMENT POLICY](/sql-statements/sql-statement-drop-placement-policy.md)