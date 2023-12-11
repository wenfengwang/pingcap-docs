---
title: ALTER RANGE
summary: ALTER RANGEの使用方法の概要について

# ALTER RANGE

現在、`ALTER RANGE`ステートメントは、TiDBで特定の配置ポリシーの範囲を変更するためにのみ使用できます。

## 概要

```ebnf+diagram
AlterRangeStmt ::=
    'ALTER' 'RANGE' 識別子 PlacementPolicyOption
```

`ALTER RANGE`は、以下の2つのパラメーターをサポートしています:

- `global`: クラスター内のすべてのデータの範囲を示します。
- `meta`: TiDBに格納されている内部メタデータの範囲を示します。

## 例

```sql
CREATE PLACEMENT POLICY `deploy111` CONSTRAINTS='{"+region=us-east-1":1, "+region=us-east-2": 1, "+region=us-west-1": 1}';
CREATE PLACEMENT POLICY `five_replicas` FOLLOWERS=4;

ALTER RANGE global PLACEMENT POLICY = "deploy111";
ALTER RANGE meta PLACEMENT POLICY = "five_replicas";
```

上記の例では、2つの配置ポリシー（`deploy111`および`five_replicas`）を作成し、異なる地域の制約を指定し、`deploy111`配置ポリシーをクラスタ範囲内のすべてのデータに適用し、`five_replicas`配置ポリシーをメタデータ範囲に適用しています。