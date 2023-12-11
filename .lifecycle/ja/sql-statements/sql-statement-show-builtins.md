---
title: SHOW BUILTINS
summary: TiDBでのSHOW BUILTINSの使用方法。
aliases: ['/docs/dev/sql-statements/sql-statement-show-builtins/']
---

# SHOW BUILTINS

`SHOW BUILTINS`は、TiDBでサポートされている組み込み関数を一覧表示するために使用されます。

## 概要

**ShowBuiltinsStmt:**

![ShowBuiltinsStmt](/media/sqlgram/ShowBuiltinsStmt.png)

## 例

{{< copyable "sql" >}}

```sql
SHOW BUILTINS;
```

```
+-----------------------------+
| Supported_builtin_functions |
+-----------------------------+
| abs                         |
| acos                        |
| adddate                     |
| addtime                     |
| aes_decrypt                 |
| aes_encrypt                 |
| and                         |
| any_value                   |
| ascii                       |
| asin                        |
| atan                        |
| atan2                       |
| benchmark                   |
| bin                         |
| bit_count                   |
| bit_length                  |
| (以下省略)
+-----------------------------+
268 rows in set (0.00 sec)
```

## MySQL互換性

この文は、TiDB構文のMySQL拡張です。