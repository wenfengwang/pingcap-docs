---
title: リソースグループの削除
summary: TiDBでのDROP RESOURCE GROUPの使用方法を学ぶ。

# リソースグループの削除

`DROP RESOURCE GROUP`ステートメントを使用してリソースグループを削除できます。

> **注意:**
>
> この機能は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスターでは利用できません。

## 概要

```ebnf+diagram
DropResourceGroupStmt ::=
    "DROP" "RESOURCE" "GROUP" IfExists リソースグループ名

IfExists ::=
    ('IF' 'EXISTS')?

ResourceGroupName ::=
    識別子
|   "DEFAULT"
```

> **注意:**
>
> - `tidb_enable_resource_control`というグローバル変数が`ON`に設定されている場合にのみ、`DROP RESOURCE GROUP`ステートメントを実行できます。
> - `DEFAULT`リソースグループは予約されており、削除することはできません。

## 例

`rg1`という名前のリソースグループを削除します。

```sql
DROP RESOURCE GROUP IF EXISTS rg1;
```

```sql
クエリ OK、0行が変更されました (0.22秒)
```

```sql
CREATE RESOURCE GROUP IF NOT EXISTS rg1 RU_PER_SEC = 500 BURSTABLE;
```

```sql
クエリ OK、0行が変更されました (0.08秒)
```

```sql
SELECT * FROM information_schema.resource_groups WHERE NAME ='rg1';
```

```sql
+------+------------+----------+-----------+-------------+
| NAME | RU_PER_SEC | PRIORITY | BURSTABLE | QUERY_LIMIT |
+------+------------+----------+-----------+-------------+
| rg1  | 500        | MEDIUM   | YES       | NULL        |
+------+------------+----------+-----------+-------------+
1行の結果 (0.01秒)
```

```sql
DROP RESOURCE GROUP IF EXISTS rg1;
```

```sql
クエリ OK、1行が変更されました (0.09秒)
```

```
SELECT * FROM information_schema.resource_groups WHERE NAME ='rg1';
```

```sql
空のセット (0.00秒)
```

## MySQL互換性

MySQLも[DROP RESOURCE GROUP](https://dev.mysql.com/doc/refman/8.0/en/drop-resource-group.html)をサポートしていますが、TiDBは`FORCE`パラメータをサポートしていません。

## 関連情報

* [ALTER RESOURCE GROUP](/sql-statements/sql-statement-alter-resource-group.md)
* [CREATE RESOURCE GROUP](/sql-statements/sql-statement-create-resource-group.md)
* [Request Unit (RU)](/tidb-resource-control.md#what-is-request-unit-ru)