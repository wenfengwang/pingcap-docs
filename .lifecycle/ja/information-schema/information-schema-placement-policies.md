---
title: PLACEMENT_POLICIES
summary: `PLACEMENT_POLICIES` の information_schema テーブルの情報を学ぶ。
aliases: ['/tidb/dev/information-schema-placement-rules']
---

# PLACEMENT_POLICIES

`PLACEMENT_POLICIES` テーブルはすべての配置ポリシーに関する情報を提供します。詳細については、[SQL における配置ルール](/placement-rules-in-sql.md) を参照してください。

> **注:**
>
> このテーブルは [TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) クラスターでは利用できません。

{{< copyable "sql" >}}

```sql
USE information_schema;
DESC placement_policies;
```

```sql
+----------------------+---------------+------+-----+---------+-------+
| Field                | Type          | Null | Key | Default | Extra |
+----------------------+---------------+------+-----+---------+-------+
| POLICY_ID            | bigint(64)    | NO   |     | <null>  |       |
| CATALOG_NAME         | varchar(512)  | NO   |     | <null>  |       |
| POLICY_NAME          | varchar(64)   | NO   |     | <null>  |       |
| PRIMARY_REGION       | varchar(1024) | YES  |     | <null>  |       |
| REGIONS              | varchar(1024) | YES  |     | <null>  |       |
| CONSTRAINTS          | varchar(1024) | YES  |     | <null>  |       |
| LEADER_CONSTRAINTS   | varchar(1024) | YES  |     | <null>  |       |
| FOLLOWER_CONSTRAINTS | varchar(1024) | YES  |     | <null>  |       |
| LEARNER_CONSTRAINTS  | varchar(1024) | YES  |     | <null>  |       |
| SCHEDULE             | varchar(20)   | YES  |     | <null>  |       |
| FOLLOWERS            | bigint(64)    | YES  |     | <null>  |       |
| LEARNERS             | bigint(64)    | YES  |     | <null>  |       |
+----------------------+---------------+------+-----+---------+-------+
12 行 in set (0.00 秒)
```

## 例

`PLACEMENT_POLICIES` テーブルは配置ポリシーをすべて表示します。配置ルールの正規バージョン（すべての配置ポリシーと配置ポリシーが割り当てられたオブジェクトを含む）を表示するには、ステートメント `SHOW PLACEMENT` を使用してください:

{{< copyable "sql" >}}

```sql
CREATE TABLE t1 (a INT); 
CREATE PLACEMENT POLICY p1 primary_region="us-east-1" regions="us-east-1";
CREATE TABLE t3 (a INT) PLACEMENT POLICY=p1;
SHOW PLACEMENT; -- table t3 を含む情報を表示します。
SELECT * FROM information_schema.placement_policies; -- t3 を除く配置ポリシーのみを表示します。
```

```sql
Query OK, 0 行が影響を受けました (0.09 秒)

Query OK, 0 行が影響を受けました (0.11 秒)

Query OK, 0 行が影響を受けました (0.08 秒)

+---------------+------------------------------------------------+------------------+
| Target        | Placement                                      | Scheduling_State |
+---------------+------------------------------------------------+------------------+
| POLICY p1     | PRIMARY_REGION="us-east-1" REGIONS="us-east-1" | NULL             |
| TABLE test.t3 | PRIMARY_REGION="us-east-1" REGIONS="us-east-1" | PENDING          |
+---------------+------------------------------------------------+------------------+
2 行 in set (0.00 秒)

+-----------+--------------+-------------+----------------+-----------+-------------+--------------------+----------------------+---------------------+----------+-----------+----------+
| POLICY_ID | CATALOG_NAME | POLICY_NAME | PRIMARY_REGION | REGIONS   | CONSTRAINTS | LEADER_CONSTRAINTS | FOLLOWER_CONSTRAINTS | LEARNER_CONSTRAINTS | SCHEDULE | FOLLOWERS | LEARNERS |
+-----------+--------------+-------------+----------------+-----------+-------------+--------------------+----------------------+---------------------+----------+-----------+----------+
| 1         | def          | p1          | us-east-1      | us-east-1 |             |                    |                      |                     |          | 2         | 0        |
+-----------+--------------+-------------+----------------+-----------+-------------+--------------------+----------------------+---------------------+----------+-----------+----------+
1 行 in set (0.00 秒)
```