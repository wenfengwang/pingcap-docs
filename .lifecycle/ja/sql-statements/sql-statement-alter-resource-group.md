---
title: ALTER RESOURCE GROUP
summary: ALTER RESOURCE GROUPの使用方法を学ぶ

# ALTER RESOURCE GROUP

`ALTER RESOURCE GROUP` ステートメントは、データベース内でリソースグループを変更するために使用されます。

> **注:**
>
> この機能は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) クラスターでは利用できません。

## 概要

```ebnf+diagram
AlterResourceGroupStmt ::=
   "ALTER" "RESOURCE" "GROUP" IfExists ResourceGroupName ResourceGroupOptionList

IfExists ::=
    ('IF' 'EXISTS')?

ResourceGroupName ::=
    Identifier
|   "DEFAULT"

ResourceGroupOptionList ::=
    DirectResourceGroupOption
|   ResourceGroupOptionList DirectResourceGroupOption
|   ResourceGroupOptionList ',' DirectResourceGroupOption

DirectResourceGroupOption ::=
    "RU_PER_SEC" EqOpt stringLit
|   "PRIORITY" EqOpt ResourceGroupPriorityOption
|   "BURSTABLE"
|   "BURSTABLE" EqOpt Boolean
|   "QUERY_LIMIT" EqOpt '(' ResourceGroupRunawayOptionList ')'
|   "QUERY_LIMIT" EqOpt '(' ')'
|   "QUERY_LIMIT" EqOpt "NULL"
|   "BACKGROUND" EqOpt '(' BackgroundOptionList ')'
|   "BACKGROUND" EqOpt '(' ')'
|   "BACKGROUND" EqOpt "NULL"

ResourceGroupPriorityOption ::=
    LOW
|   MEDIUM
|   HIGH

ResourceGroupRunawayOptionList ::=
    DirectResourceGroupRunawayOption
|   ResourceGroupRunawayOptionList DirectResourceGroupRunawayOption
|   ResourceGroupRunawayOptionList ',' DirectResourceGroupRunawayOption

DirectResourceGroupRunawayOption ::=
    "EXEC_ELAPSED" EqOpt stringLit
|   "ACTION" EqOpt ResourceGroupRunawayActionOption
|   "WATCH" EqOpt ResourceGroupRunawayWatchOption "DURATION" EqOpt stringLit

ResourceGroupRunawayWatchOption ::=
    EXACT
|   SIMILAR

ResourceGroupRunawayActionOption ::=
    DRYRUN
|   COOLDOWN
|   KILL

BackgroundOptionList ::=
    DirectBackgroundOption
|   BackgroundOptionList DirectBackgroundOption
|   BackgroundOptionList ',' DirectBackgroundOption

DirectBackgroundOption ::=
    "TASK_TYPES" EqOpt stringLit
```

TiDBでは、[Request Unit (RU)](/tidb-resource-control.md#what-is-request-unit-ru) がCPU、IO、およびその他のシステムリソースのための統一された抽象単位であり、以下のような `DirectResourceGroupOption` をサポートしています。

| オプション    | 説明                         | 例                    |
|---------------|-------------------------------|------------------------|
| `RU_PER_SEC`  | 秒あたりのRUの補填率  | `RU_PER_SEC = 500` は、このリソースグループが秒あたり500 RUsで補充されることを示します。 |
| `PRIORITY`    | TiKVで処理されるタスクの絶対優先度  | `PRIORITY = HIGH` は、優先度が高いことを示します。指定されていない場合、デフォルト値は `MEDIUM` です。 |
| `BURSTABLE`   | `BURSTABLE` 属性が設定されている場合、TiDBは割り当てられたクォータを超えるときに対応するリソースグループが利用可能なシステムリソースを使用することを許可します。 |
| `QUERY_LIMIT` | クエリの実行がこの条件を満たすと、クエリは逃走クエリとして識別され、対応するアクションが実行されます。 | `QUERY_LIMIT=(EXEC_ELAPSED='60s', ACTION=KILL, WATCH=EXACT DURATION='10m')` は、実行時間が60秒を超えるとクエリが逃走クエリとして識別されます。クエリは中断されます。同じSQLテキストを持つすべてのSQLステートメントは、10分間即座に中断されます。`QUERY_LIMIT=()` または `QUERY_LIMIT=NULL` は、逃走制御が有効になっていないことを意味します。[逃走クエリ](/tidb-resource-control.md#manage-queries-that-consume-more-resources-than-expected-runaway-queries)を参照してください。 |
| `BACKGROUND`  | バックグラウンドタスクを構成します。詳細については、[バックグラウンドタスクの管理](/tidb-resource-control.md#manage-background-tasks)を参照してください。 | `BACKGROUND=(TASK_TYPES="br,stats")` は、バックアップ・リストアおよび統計収集関連のタスクがバックグラウンドタスクとしてスケジュールされることを示します。 |

> **注:**
>
> - `tidb_enable_resource_control` グローバル変数が `ON` に設定されている場合にのみ、`ALTER RESOURCE GROUP` ステートメントを実行できます。
> - `ALTER RESOURCE GROUP` ステートメントは、指定しないパラメータを変更せずに増分変更をサポートします。ただし、`QUERY_LIMIT` と `BACKGROUND` はまとまりとして使用され、一部だけを変更することはできません。
> - 現在、`default` リソースグループのみが `BACKGROUND` 構成の変更をサポートしています。

## 例

リソースグループ `rg1` を作成し、そのプロパティを変更します。

```sql
DROP RESOURCE GROUP IF EXISTS rg1;
```

```
Query OK, 0 rows affected (0.22 sec)
```

```sql
CREATE RESOURCE GROUP IF NOT EXISTS rg1
  RU_PER_SEC = 100
  BURSTABLE;
```

```sql
Query OK, 0 rows affected (0.08 sec)
```

```sql
SELECT * FROM information_schema.resource_groups WHERE NAME ='rg1';
```

```sql
+------+------------+----------+-----------+-------------+------------+
| NAME | RU_PER_SEC | PRIORITY | BURSTABLE | QUERY_LIMIT | BACKGROUND |
+------+------------+----------+-----------+-------------+------------+
| rg1  | 100        | MEDIUM   | NO        | NULL        | NULL       |
+------+------------+----------+-----------+-------------+------------+
1 rows in set (1.30 sec)
```

```sql
ALTER RESOURCE GROUP rg1
  RU_PER_SEC = 200
  PRIORITY = LOW
  QUERY_LIMIT = (EXEC_ELAPSED='1s' ACTION=COOLDOWN WATCH=EXACT DURATION '30s');
```

```sql
Query OK, 0 rows affected (0.08 sec)
```

```sql
SELECT * FROM information_schema.resource_groups WHERE NAME ='rg1';
```

```sql
+------+------------+----------+-----------+----------------------------------------------------------------+------------+
| NAME | RU_PER_SEC | PRIORITY | BURSTABLE | QUERY_LIMIT                                                    | BACKGROUND |
+------+------------+----------+-----------+----------------------------------------------------------------+------------+
| rg1  | 200        | LOW      | NO        | EXEC_ELAPSED='1s', ACTION=COOLDOWN, WATCH=EXACT DURATION='30s' | NULL       |
+------+------------+----------+-----------+----------------------------------------------------------------+------------+
1 rows in set (1.30 sec)
```

`default` リソースグループの `BACKGROUND` オプションを変更します。

```sql
ALTER RESOURCE GROUP default BACKGROUND = (TASK_TYPES = "br,ddl");
```

```sql
Query OK, 0 rows affected (0.08 sec)
```

```sql
SELECT * FROM information_schema.resource_groups WHERE NAME ='default';
```

```sql
+---------+------------+----------+-----------+-------------+---------------------+
| NAME    | RU_PER_SEC | PRIORITY | BURSTABLE | QUERY_LIMIT | BACKGROUND          |
+---------+------------+----------+-----------+-------------+---------------------+
| default | UNLIMITED  | MEDIUM   | YES       | NULL        | TASK_TYPES='br,ddl' |
+---------+------------+----------+-----------+-------------+---------------------+
1 rows in set (1.30 sec)
```

## MySQL互換性

MySQLも[ALTER RESOURCE GROUP](https://dev.mysql.com/doc/refman/8.0/en/alter-resource-group.html)をサポートしています。ただし、受け入れ可能なパラメータはTiDBと異なるため、互換性がありません。

## 関連項目

* [DROP RESOURCE GROUP](/sql-statements/sql-statement-drop-resource-group.md)
* [CREATE RESOURCE GROUP](/sql-statements/sql-statement-create-resource-group.md)
* [Request Unit (RU)](/tidb-resource-control.md#what-is-request-unit-ru)