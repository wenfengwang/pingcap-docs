---
title: リソースグループの作成
summary: TiDB での CREATE RESOURCE GROUP の使用方法を学ぶ

# CREATE RESOURCE GROUP

`CREATE RESOURCE GROUP` ステートメントを使用してリソースグループを作成できます。

> **注記:**
>
> この機能は [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) クラスタでは利用できません。

## 概要

```ebnf+diagram
CreateResourceGroupStmt ::=
   "CREATE" "RESOURCE" "GROUP" IfNotExists ResourceGroupName ResourceGroupOptionList

IfNotExists ::=
    ('IF' 'NOT' 'EXISTS')?

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
|   "WATCH" EqOpt ResourceGroupRunawayWatchOption WatchDurationOption

WatchDurationOption ::=
    ("DURATION" EqOpt stringLit | "DURATION" EqOpt "UNLIMITED")?

ResourceGroupRunawayWatchOption ::=
    EXACT
|   SIMILAR
|   PLAN

ResourceGroupRunawayActionOption ::=
    DRYRUN
|   COOLDOWN
|   KILL
```

リソースグループ名パラメータ (`ResourceGroupName`) はグローバルに一意である必要があります。

TiDB は、[Request Unit (RU)](/tidb-resource-control.md#what-is-request-unit-ru) を CPU、IO、およびその他のシステムリソースの統一抽象単位としてサポートしています。

次の `DirectResourceGroupOption` がサポートされています。

| オプション       | 説明                                     | 例                    |
|---------------|-------------------------------------|------------------------|
| `RU_PER_SEC`  | 秒ごとの RU 追加率                      | `RU_PER_SEC = 500` は、このリソースグループが秒間 500 RU で追加されることを示します                   |
| `PRIORITY`    | TiKV で処理されるタスクの絶対優先度            | `PRIORITY = HIGH` は、優先度が高いことを示します。指定されていない場合、デフォルト値は `MEDIUM` です。        |
| `BURSTABLE`   | `BURSTABLE` 属性が設定されている場合、TiDB はクォータを超過した場合に対応するリソースグループが利用可能なシステムリソースを使用することを許可します。 |
| `QUERY_LIMIT` | クエリの実行がこの条件を満たすと、クエリは暴走クエリとして識別され、対応するアクションが実行されます。  | `QUERY_LIMIT=(EXEC_ELAPSED='60s', ACTION=KILL, WATCH=EXACT DURATION='10m')` は、実行時間が 60 秒を超えるとクエリが暴走クエリとして識別されます。クエリは終了されます。同じ SQL テキストを持つすべての SQL ステートメントは、次の 10 分間で即座に終了されます。 `QUERY_LIMIT=()` または `QUERY_LIMIT=NULL` は、暴走制御が有効になっていないことを意味します。[暴走クエリ](/tidb-resource-control.md#manage-queries-that-consume-more-resources-than-expected-runaway-queries)を参照してください。 |

> **注記:**
>
> - `tidb_enable_resource_control` が `ON` に設定されているときにのみ `CREATE RESOURCE GROUP` ステートメントを実行できます。
> TiDB は、クラスタの初期化時に `default` リソースグループを自動作成します。このリソースグループのデフォルトの `RU_PER_SEC` の値は `UNLIMITED` (つまり `INT` 型の最大値である `2147483647`) であり、`BURSTABLE` モードです。どのリソースグループにもバインドされていないすべてのリクエストは、自動的にこの `default` リソースグループにバインドされます。新しい構成を別のリソースグループのために作成する場合、必要に応じて `default` リソースグループの構成を変更することを推奨します。
> - 現時点では、`default` リソースグループのみが `BACKGROUND` 構成の変更をサポートしています。

## 例

`rg1` および `rg2` の 2 つのリソースグループを作成します。

```sql
DROP RESOURCE GROUP IF EXISTS rg1;
```

```sql
Query OK, 0 rows affected (0.22 sec)
```

```sql
CREATE RESOURCE GROUP IF NOT EXISTS rg1
  RU_PER_SEC = 100
  PRIORITY = HIGH
  BURSTABLE;
```

```sql
Query OK, 0 rows affected (0.08 sec)
```

```sql
CREATE RESOURCE GROUP IF NOT EXISTS rg2
  RU_PER_SEC = 200 QUERY_LIMIT=(EXEC_ELAPSED='100ms', ACTION=KILL);
```

```sql
Query OK, 0 rows affected (0.08 sec)
```

```sql
SELECT * FROM information_schema.resource_groups WHERE NAME ='rg1' or NAME = 'rg2';
```

```sql
+------+------------+----------+-----------+---------------------------------+
| NAME | RU_PER_SEC | PRIORITY | BURSTABLE | QUERY_LIMIT                     |
+------+------------+----------+-----------+---------------------------------+
| rg1  | 100        | HIGH     | YES       | NULL                            |
| rg2  | 200        | MEDIUM   | NO        | EXEC_ELAPSED=100ms, ACTION=KILL |
+------+------------+----------+-----------+---------------------------------+
2 rows in set (1.30 sec)
```

## MySQL 互換性

MySQL も [CREATE RESOURCE GROUP](https://dev.mysql.com/doc/refman/8.0/en/create-resource-group.html) をサポートしています。ただし、受け入れ可能なパラメータは TiDB と異なるため、互換性がありません。

## 関連情報

* [DROP RESOURCE GROUP](/sql-statements/sql-statement-drop-resource-group.md)
* [ALTER RESOURCE GROUP](/sql-statements/sql-statement-alter-resource-group.md)
* [ALTER USER RESOURCE GROUP](/sql-statements/sql-statement-alter-user.md#modify-the-resource-group-bound-to-the-user)
* [Request Unit (RU)](/tidb-resource-control.md#what-is-request-unit-ru)