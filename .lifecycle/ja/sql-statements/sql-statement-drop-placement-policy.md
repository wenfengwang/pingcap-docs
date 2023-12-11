---
title: ドロップ配置ポリシー
summary: TiDB での ALTER PLACEMENT POLICY の使用
---

# ドロップ配置ポリシー

`ドロップ配置ポリシー` は以前に作成した配置ポリシーを削除するために使用されます。

> **注意:**
>
> この機能は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスターでは利用できません。

## 概要

```ebnf+diagram
DropPolicyStmt ::=
    "DROP" "PLACEMENT" "POLICY" IfExists ポリシー名

ポリシー名 ::=
    識別子
```

## 例

配置ポリシーは、どのテーブルやパーティションにも参照されていない場合にのみ削除できます。

{{< copyable "sql" >}}

```sql
CREATE PLACEMENT POLICY p1 FOLLOWERS=4;
CREATE TABLE t1 (a INT PRIMARY KEY) PLACEMENT POLICY=p1;
DROP PLACEMENT POLICY p1;  -- このステートメントは、配置ポリシー p1 が参照されているため失敗します。

-- 配置ポリシーを参照しているテーブルとパーティションを検索します。
SELECT table_schema, table_name FROM information_schema.tables WHERE tidb_placement_policy_name='p1';
SELECT table_schema, table_name FROM information_schema.partitions WHERE tidb_placement_policy_name='p1';

ALTER TABLE t1 PLACEMENT POLICY=default;  -- t1 から配置ポリシーを削除します。
DROP PLACEMENT POLICY p1;  -- 成功します。
```

```sql
Query OK, 0 rows affected (0.10 sec)

Query OK, 0 rows affected (0.11 sec)

ERROR 8241 (HY000): Placement policy 'p1' is still in use

+--------------+------------+
| table_schema | table_name |
+--------------+------------+
| test         | t1         |
+--------------+------------+
1 row in set (0.00 sec)

Empty set (0.01 sec)

Query OK, 0 rows affected (0.08 sec)

Query OK, 0 rows affected (0.21 sec)
```

## MySQL 互換性

このステートメントは、MySQL構文の TiDB 拡張機能です。

## 関連情報

* [SQL での配置ルール](/placement-rules-in-sql.md)
* [SHOW PLACEMENT](/sql-statements/sql-statement-show-placement.md)
* [CREATE PLACEMENT POLICY](/sql-statements/sql-statement-create-placement-policy.md)
* [ALTER PLACEMENT POLICY](/sql-statements/sql-statement-alter-placement-policy.md)