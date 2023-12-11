---
title: SHOW PROFILES
summary: TiDBデータベースでのSHOW PROFILESの使用方法の概要。
aliases: ['/docs/dev/sql-statements/sql-statement-show-profiles/']
---

# SHOW PROFILES

`SHOW PROFILES`ステートメントは現在、空の結果のみを返します。

## 概要

**ShowStmt:**

![ShowStmt](/media/sqlgram/ShowStmt.png)

## 例

{{< copyable "sql" >}}

```sql
SHOW PROFILES;
```

```
Empty set (0.00 sec)
```

## MySQL互換性

このステートメントは、MySQLとの互換性のためにのみ含まれています。`SHOW PROFILES`を実行すると常に空の結果が返されます。