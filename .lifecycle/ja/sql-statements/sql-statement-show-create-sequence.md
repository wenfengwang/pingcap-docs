---
title: シーケンスの作成を表示
summary: TiDBデータベースでのSHOW CREATE SEQUENCEの使用概要。
aliases: ['/docs/dev/sql-statements/sql-statement-show-create-sequence/','/docs/dev/reference/sql/statements/show-create-sequence/']
---

# SHOW CREATE SEQUENCE

`SHOW CREATE SEQUENCE`は、`SHOW CREATE TABLE`と類似したシーケンスの詳細情報を表示します。

## 構文

**ShowCreateSequenceStmt:**

![ShowCreateSequenceStmt](/media/sqlgram/ShowCreateSequenceStmt.png)

**TableName:**

![TableName](/media/sqlgram/TableName.png)

## 例

{{< copyable "sql" >}}

```sql
CREATE SEQUENCE seq;
```

```
クエリは正常終了しました。影響を受けた行: 0(実行時間：0.03秒)
```

{{< copyable "sql" >}}

```sql
SHOW CREATE SEQUENCE seq;
```

```
+-------+----------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                               |
+-------+----------------------------------------------------------------------------------------------------------------------------+
| seq   | CREATE SEQUENCE `seq` start with 1 minvalue 1 maxvalue 9223372036854775806 increment by 1 cache 1000 nocycle ENGINE=InnoDB |
+-------+----------------------------------------------------------------------------------------------------------------------------+
1 行が選択されました (実行時間：0.00秒)
```

## MySQL互換性

このステートメントはTiDBの拡張機能です。実装は、MariaDBで利用可能なシーケンスにモデル化されています。

## 関連項目

* [CREATE SEQUENCE](/sql-statements/sql-statement-create-sequence.md)
* [DROP SEQUENCE](/sql-statements/sql-statement-drop-sequence.md)