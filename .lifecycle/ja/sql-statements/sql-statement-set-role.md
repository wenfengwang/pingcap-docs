---
title: SET ROLE | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースのSET ROLEの使用概要です。
aliases: ['/docs/dev/sql-statements/sql-statement-set-role/']
---

# SET ROLE

`SET ROLE`ステートメントは、現在のセッションでロールを有効にするために使用されます。ロールを有効にした後、ユーザーはそのロールの特権を使用できます。

## 概要

**SetRoleStmt:**

![SetRoleStmt](/media/sqlgram/SetRoleStmt.png)

**SetRoleOpt:**

![SetRoleOpt](/media/sqlgram/SetRoleOpt.png)

**SetDefaultRoleOpt:**

![SetDefaultRoleOpt](/media/sqlgram/SetDefaultRoleOpt.png)

## 例

ユーザー`'u1'@'%'`を作成し、`'r1'@'%'`、`'r2'@'%'`、`'r3'@'%'`の3つのロールを作成します。これらのロールを`'u1'@'%'`に付与し、`'u1'@'%'`のデフォルトロールとして`'r1'@'%'`を設定します。

{{< copyable "sql" >}}

```sql
CREATE USER 'u1'@'%';
CREATE ROLE 'r1', 'r2', 'r3';
GRANT 'r1', 'r2', 'r3' TO 'u1'@'%';
SET DEFAULT ROLE 'r1' TO 'u1'@'%';
```

`'u1'@'%'`でログインし、以下の`SET ROLE`ステートメントを実行してすべてのロールを有効にします。

{{< copyable "sql" >}}

```sql
SET ROLE ALL;
SELECT CURRENT_ROLE();
```

```
+----------------------------+
| CURRENT_ROLE()             |
+----------------------------+
| `r1`@`%`,`r2`@`%`,`r3`@`%` |
+----------------------------+
1 row in set (0.000 sec)
```

以下の`SET ROLE`ステートメントを実行して、`'r2'`と`'r3'`を有効にします。

{{< copyable "sql" >}}

```sql
SET ROLE 'r2', 'r3';
SELECT CURRENT_ROLE();
```

```
+-------------------+
| CURRENT_ROLE()    |
+-------------------+
| `r2`@`%`,`r3`@`%` |
+-------------------+
1 row in set (0.000 sec)
```

以下の`SET ROLE`ステートメントを実行して、デフォルトのロールを有効にします。

{{< copyable "sql" >}}

```sql
SET ROLE DEFAULT;
SELECT CURRENT_ROLE();
```

```
+----------------+
| CURRENT_ROLE() |
+----------------+
| `r1`@`%`       |
+----------------+
1 row in set (0.000 sec)
```

以下の`SET ROLE`ステートメントを実行して、すべての有効なロールをキャンセルします。

{{< copyable "sql" >}}

```sql
SET ROLE NONE;
SELECT CURRENT_ROLE();
```

```
+----------------+
| CURRENT_ROLE() |
+----------------+
|                |
+----------------+
1 row in set (0.000 sec)
```

## MySQLの互換性

TiDBの`SET ROLE`ステートメントはMySQL 8.0のロール機能と完全に互換性があります。互換性に違いがある場合は、[バグを報告してください](https://docs.pingcap.com/tidb/stable/support)。

## 関連項目

* [CREATE ROLE](/sql-statements/sql-statement-create-role.md)
* [DROP ROLE](/sql-statements/sql-statement-drop-role.md)
* [`GRANT <role>`](/sql-statements/sql-statement-grant-role.md)
* [`REVOKE <role>`](/sql-statements/sql-statement-revoke-role.md)
* [SET DEFAULT ROLE](/sql-statements/sql-statement-set-default-role.md)

<CustomContent platform="tidb">

* [ロールベースアクセス制御](/role-based-access-control.md)

</CustomContent>