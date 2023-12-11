---
title: リソースグループの設定
summary: TiDBデータベースにおけるSET RESOURCE GROUPの使用方法の概要です。

---

# リソースグループの設定

`SET RESOURCE GROUP`は、現在のセッションでリソースグループを設定するために使用されます。

> **注意:**
>
> この機能は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスターでは利用できません。

## 概要

**SetResourceGroupStmt:**

```ebnf+diagram
SetResourceGroupStmt ::=
    "SET" "RESOURCE" "GROUP" ResourceGroupName

ResourceGroupName ::=
    Identifier
|   "DEFAULT"
```

## 例

ユーザー`user1`を作成し、2つのリソースグループ`rg1`と`rg2`を作成し、ユーザー`user1`をリソースグループ`rg1`にバインドします。

```sql
CREATE USER 'user1';
CREATE RESOURCE GROUP 'rg1' RU_PER_SEC = 1000;
ALTER USER 'user1' RESOURCE GROUP `rg1`;
```

ユーザー`user1`でログインし、現在のユーザーにバインドされたリソースグループを表示します。

```sql
SELECT CURRENT_RESOURCE_GROUP();
```

```
+--------------------------+
| CURRENT_RESOURCE_GROUP() |
+--------------------------+
| rg1                      |
+--------------------------+
1 row in set (0.00 sec)
```

`SET RESOURCE GROUP`を実行して現在のセッションのリソースグループを`rg2`に設定します。

```sql
SET RESOURCE GROUP `rg2`;
SELECT CURRENT_RESOURCE_GROUP();
```

```
+--------------------------+
| CURRENT_RESOURCE_GROUP() |
+--------------------------+
| rg2                      |
+--------------------------+
1 row in set (0.00 sec)
```

`SET RESOURCE GROUP`を実行して現在のセッションをデフォルトのリソースグループを使用するように指定します。

```sql
SET RESOURCE GROUP `default`;
SELECT CURRENT_RESOURCE_GROUP();
```

```sql
+--------------------------+
| CURRENT_RESOURCE_GROUP() |
+--------------------------+
| default                  |
+--------------------------+
1 row in set (0.00 sec)
```

## MySQL互換性

MySQLも[SET RESOURCE GROUP](https://dev.mysql.com/doc/refman/8.0/en/set-resource-group.html)をサポートしていますが、受け入れられるパラメータはTiDBとは異なります。互換性がありません。

## 関連項目

* [CREATE RESOURCE GROUP](/sql-statements/sql-statement-create-resource-group.md)
* [DROP RESOURCE GROUP](/sql-statements/sql-statement-drop-resource-group.md)
* [ALTER RESOURCE GROUP](/sql-statements/sql-statement-alter-resource-group.md)
* [リソース制御](/tidb-resource-control.md)