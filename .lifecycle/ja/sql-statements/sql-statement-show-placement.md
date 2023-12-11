---
title: 表示プレースメント
summary: TiDBでのSHOW PLACEMENTの使用方法。

# 表示プレースメント

`SHOW PLACEMENT` は、プレースメントポリシーからすべてのプレースメントオプションをまとめ、標準形式で表示します。

> **注意:**
>
> この機能は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless) クラスターでは利用できません。

この文は、`Scheduling_State` フィールドが、Placement Driver（PD）がプレースメントのスケジューリングを行う現在の進行状況を示す結果セットを返します。

* `PENDING`: PD はまだプレースメントのスケジューリングを開始していません。これは、プレースメントルールが意味的には正しいが、現在のクラスターでは満たすことができないことを示す場合があります。たとえば、`FOLLOWERS=4` ですが、フォロワーの候補が3つの TiKV ストアしかない場合です。
* `INPROGRESS`: PD は現在、プレースメントのスケジューリングを行っています。
* `SCHEDULED`: PD はプレースメントのスケジューリングに成功しました。

## 概要

```ebnf+diagram
ShowStmt ::=
    "PLACEMENT"
```

## 例

{{< copyable "sql" >}}

```sql
CREATE PLACEMENT POLICY p1 PRIMARY_REGION="us-east-1" REGIONS="us-east-1,us-west-1" FOLLOWERS=4;
CREATE TABLE t1 (a INT) PLACEMENT POLICY=p1;
SHOW PLACEMENT;
```

```
Query OK, 0 rows affected (0.01 sec)

Query OK, 0 rows affected (0.00 sec)

+---------------+----------------------------------------------------------------------+------------------+
| Target        | Placement                                                            | Scheduling_State |
+---------------+----------------------------------------------------------------------+------------------+
| POLICY p1     | PRIMARY_REGION="us-east-1" REGIONS="us-east-1,us-west-1" FOLLOWERS=4 | NULL             |
| DATABASE test | PRIMARY_REGION="us-east-1" REGIONS="us-east-1,us-west-1" FOLLOWERS=4 | INPROGRESS       |
| TABLE test.t1 | PRIMARY_REGION="us-east-1" REGIONS="us-east-1,us-west-1" FOLLOWERS=4 | INPROGRESS       |
+---------------+----------------------------------------------------------------------+------------------+
4 rows in set (0.00 sec)
```

## MySQL 互換性

この文は、MySQL構文へのTiDBの拡張です。

## 関連情報

* [SQLにおけるプレースメントルール](/placement-rules-in-sql.md)
* [SHOW PLACEMENT FOR](/sql-statements/sql-statement-show-placement-for.md)
* [CREATE PLACEMENT POLICY](/sql-statements/sql-statement-create-placement-policy.md)