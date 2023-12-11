---
title: RESOURCE_GROUPS
summary: `RESOURCE_GROUPS`に関する情報スキーマテーブルを学ぶ。

# RESOURCE_GROUPS

`RESOURCE_GROUPS`テーブルはすべてのリソースグループに関する情報を示します。詳細については、[リソースの分離を実現するためのリソース制御の使用](/tidb-resource-control.md)を参照してください。

> **注意:**
>
> このテーブルは[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスターでは利用できません。

```sql
USE information_schema;
DESC resource_groups;
```

```sql
+------------+-------------+------+------+---------+-------+
| Field      | Type        | Null | Key  | Default | Extra |
+------------+-------------+------+------+---------+-------+
| NAME       | varchar(32) | NO   |      | NULL    |       |
| RU_PER_SEC | bigint(21)  | YES  |      | NULL    |       |
| PRIORITY   | varchar(6)  | YES  |      | NULL    |       |
| BURSTABLE  | varchar(3)  | YES  |      | NULL    |       |
+------------+-------------+------+------+---------+-------+
3 行が返されました (0.00 秒)
```

## 例

```sql
SELECT * FROM information_schema.resource_groups; -- すべてのリソースグループを表示します。TiDBには`default`リソースグループがあります。
```

```sql
+---------+------------+----------+-----------+
| NAME    | RU_PER_SEC | PRIORITY | BURSTABLE |
+---------+------------+----------+-----------+
| default | UNLIMITED  | MEDIUM   | YES       |
+---------+------------+----------+-----------+
```

```sql
CREATE RESOURCE GROUP rg1 RU_PER_SEC=1000; -- `rg1`リソースグループを作成します。
```

```sql
クエリは成功しました。0行が変更されました(0.34秒)
```

```sql
SHOW CREATE RESOURCE GROUP rg1; -- `rg1`リソースグループの定義を表示します。
```

```sql
+----------------+---------------------------------------------------------------+
| Resource_Group | Create Resource Group                                         |
+----------------+---------------------------------------------------------------+
| rg1            | CREATE RESOURCE GROUP `rg1` RU_PER_SEC=1000 PRIORITY="MEDIUM" |
+----------------+---------------------------------------------------------------+
1 行が返されました (0.00 秒)
```

```sql
SELECT * FROM information_schema.resource_groups WHERE NAME = 'rg1'; -- リソースグループ`rg1`を表示します。
```

```sql
+------+------------+----------+-----------+-------------+
| NAME | RU_PER_SEC | PRIORITY | BURSTABLE | QUERY_LIMIT |
+------+------------+----------+-----------+-------------+
| rg1  | 1000       | MEDIUM   | NO        | NULL        |
+------+------------+----------+-----------+-------------+
1 行が返されました (0.00 秒)
```

`RESOURCE_GROUPS`テーブルの列の説明は次のとおりです。

* `NAME`: リソースグループの名前。
* `RU_PER_SEC`: リソースグループのバックフィル速度。単位はRU/秒で、RUは[リクエストユニット](/tidb-resource-control.md#what-is-request-unit-ru)を意味します。
* `PRIORITY`: TiKVで処理されるタスクの絶対優先度。異なるリソースは、`PRIORITY`設定に従ってスケジュールされます。高い`PRIORITY`を持つタスクが最初にスケジュールされます。同じ`PRIORITY`を持つリソースグループの場合、`RU_PER_SEC`の設定に比例してスケジュールされます。`PRIORITY`が指定されていない場合、デフォルトの優先度は`MEDIUM`です。
* `BURSTABLE`: リソースグループが利用可能なシステムリソースを過剰使用するかどうか。

> **注意:**
>
> TiDBはクラスター初期化時に自動的に`default`リソースグループを作成します。このリソースグループでは、`RU_PER_SEC`のデフォルト値が`UNLIMITED`(つまり、`INT`型の最大値である`2147483647`)であり、`BURSTABLE`モードです。リソースグループにバインドされていないすべてのリクエストは、自動的にこの`default`リソースグループにバインドされます。新しいリソースグループの構成を作成する場合は、必要に応じて`default`リソースグループの構成を変更することをお勧めします。