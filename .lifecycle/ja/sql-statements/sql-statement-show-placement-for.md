---
title: 表示場所の表示
summary: TiDBにおけるSHOW PLACEMENT FORの使用法。

---

# SHOW PLACEMENT FOR

`SHOW PLACEMENT FOR` はすべての配置オプションを要約し、特定のテーブル、データベーススキーマ、またはパーティションのための正規形でそれらを表示します。

> **注意:**
>
> この機能は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスターでは利用できません。

このステートメントは、`Scheduling_State` フィールドが配置のスケジューリングにおける配置ドライバ（PD）の現在の進行状況を示す結果セットを返します:

* `PENDING`: PDはまだ配置のスケジューリングを開始していません。これは、配置規則が意味的に正しいが、現在のクラスタを満たすことができないことを示す場合があります。たとえば、`FOLLOWERS=4` でもフォロワー候補が3つしかない場合などです。
* `INPROGRESS`: PDは現在、配置のスケジューリングを行っています。
* `SCHEDULED`: PDは配置のスケジューリングを正常に行いました。

## 概要

```ebnf+diagram
ShowStmt ::=
    "PLACEMENT" "FOR" ShowPlacementTarget

ShowPlacementTarget ::=
    DatabaseSym DBName
|   "TABLE" TableName
|   "TABLE" TableName "PARTITION" Identifier
```

## 例

{{< copyable "sql" >}}

```sql
CREATE PLACEMENT POLICY p1 PRIMARY_REGION="us-east-1" REGIONS="us-east-1,us-west-1" FOLLOWERS=4;
ALTER DATABASE test PLACEMENT POLICY=p1;
CREATE TABLE t1 (a INT);
SHOW PLACEMENT FOR DATABASE test;
SHOW PLACEMENT FOR TABLE t1;
SHOW CREATE TABLE t1\G;
CREATE TABLE t3 (a INT) PARTITION BY RANGE (a) (PARTITION p1 VALUES LESS THAN (10), PARTITION p2 VALUES LESS THAN (20));
SHOW PLACEMENT FOR TABLE t3 PARTITION p1\G;
```

```
クエリが成功しました。 (実行時間: 0.02 sec)

クエリが成功しました。 (実行時間: 0.00 sec)

クエリが成功しました。 (実行時間: 0.01 sec)

+---------------+----------------------------------------------------------------------+------------------+
| Target        | Placement                                                            | Scheduling_State |
+---------------+----------------------------------------------------------------------+------------------+
| DATABASE test | PRIMARY_REGION="us-east-1" REGIONS="us-east-1,us-west-1" FOLLOWERS=4 | INPROGRESS       |
+---------------+----------------------------------------------------------------------+------------------+
1 行が選択されました (実行時間: 0.00 sec)

+---------------+-------------+------------------+
| Target        | Placement   | Scheduling_State |
+---------------+-------------+------------------+
| TABLE test.t1 | FOLLOWERS=4 | INPROGRESS       |
+---------------+-------------+------------------+
1 行が選択されました (実行時間: 0.00 sec)

***************************[ 1. 行 ]***************************
Table        | t1
Create Table | CREATE TABLE `t1` (
  `a` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin /*T![placement] PLACEMENT POLICY=`p1` */
1 行が選択されました (実行時間: 0.00 sec)

***************************[ 1. 行 ]***************************
Target           | TABLE test.t3 PARTITION p1
Placement        | PRIMARY_REGION="us-east-1" REGIONS="us-east-1,us-west-1" FOLLOWERS=4
Scheduling_State | PENDING
1 行が選択されました (実行時間: 0.00 sec)
```

## MySQL互換性

このステートメントは、MySQL構文のTiDB拡張です。

## 関連情報

* [SQLにおける配置規則](/placement-rules-in-sql.md)
* [SHOW PLACEMENT](/sql-statements/sql-statement-show-placement.md)
* [CREATE PLACEMENT POLICY](/sql-statements/sql-statement-create-placement-policy.md)