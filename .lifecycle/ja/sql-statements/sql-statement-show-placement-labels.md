---
title: プレースメントラベルの表示
summary: TiDB での SHOW PLACEMENT LABELS の使用方法。

# プレースメントラベルの表示

`SHOW PLACEMENT LABELS` は、プレースメントルールで利用可能なラベルと値を要約するために使用されます。

> **注意:**
>
> この機能は [TiDB サーバーレス](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) クラスターでは利用できません。

## 概要

```ebnf+diagram
ShowStmt ::=
    "PLACEMENT" "LABELS"
```

## 例

{{< copyable "sql" >}}

```sql
SHOW PLACEMENT LABELS;
```

```
+--------+----------------+
| Key    | Values         |
+--------+----------------+
| region | ["us-east-1"]  |
| zone   | ["us-east-1a"] |
+--------+----------------+
2 行が返されました (0.00 秒)
```

## MySQL 互換性

この文は、MySQL 構文への TiDB 拡張です。

## 関連情報

* [SQL におけるプレースメントルール](/placement-rules-in-sql.md)
* [SHOW PLACEMENT](/sql-statements/sql-statement-show-placement.md)
* [CREATE PLACEMENT POLICY](/sql-statements/sql-statement-create-placement-policy.md)