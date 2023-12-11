---
title: ロールベースのアクセス制御
summary: この文書では、TiDBのRBAC操作と実装について紹介します。
aliases: ['/docs/dev/role-based-access-control/', '/docs/dev/reference/security/role-based-access-control/']
---

# ロールベースのアクセス制御

TiDBのロールベースのアクセス制御（RBAC）システムの実装は、MySQL 8.0と類似しています。TiDBは、MySQLのほとんどのRBAC構文と互換性があります。

この文書では、TiDBのRBAC関連の操作と実装について紹介します。

## RBAC操作

ロールは一連の権限の集合です。以下の操作を行うことができます：

- ロールの作成。
- ロールの削除。
- ロールに対する権限の付与。
- 他のユーザーにロールの付与。ユーザーがロールを有効にした後は、ロールに含まれる権限を取得することができます。

### ロールの作成

例えば、次のステートメントを使用して、`app_developer`、`app_read`、`app_write`のロールを作成できます：

{{< copyable "sql" >}}

```sql
CREATE ROLE 'app_developer', 'app_read', 'app_write';
```

ロールの命名形式とルールについては、[TiDBユーザーアカウント管理](/user-account-management.md)を参照してください。

ロールは`mysql.user`テーブルに格納され、ロール名のホスト名部分（省略された場合）はデフォルトで`'%'`になります。作成しようとしているロールの名前は一意でなければなりません。そうでない場合はエラーが報告されます。

ロールを作成するには、`CREATE ROLE`または`CREATE USER`権限が必要です。

### ロールへの権限の付与

ロールに対する権限を付与する操作は、ユーザーに対する権限を付与する操作と同じです。詳細については、[TiDB権限管理](/privilege-management.md)を参照してください。

例えば、次のステートメントを使用して、`app_read`ロールに`app_db`データベースの読み取り権限を付与できます：

{{< copyable "sql" >}}

```sql
GRANT SELECT ON app_db.* TO 'app_read'@'%';
```

次のステートメントを使用して、`app_write`ロールに`app_db`データベースへのデータ書き込み権限を付与できます：

{{< copyable "sql" >}}

```sql
GRANT INSERT, UPDATE, DELETE ON app_db.* TO 'app_write'@'%';
```

次のステートメントを使用して、`app_developer`ロールに`app_db`データベースのすべての権限を付与できます：

{{< copyable "sql" >}}

```sql
GRANT ALL ON app_db.* TO 'app_developer';
```

### ユーザーにロールを付与

ユーザー`dev1`が`app_db`に対するすべての権限を持つdeveloperロールを持っており、ユーザー`read_user1`と`read_user2`が`app_db`に対する読み取り専用権限を持ち、ユーザー`rw_user1`が`app_db`に対する読み取りと書き込み権限を持っていると仮定します。

次のように`CREATE USER`を使用してユーザーを作成します：

{{< copyable "sql" >}}

```sql
CREATE USER 'dev1'@'localhost' IDENTIFIED BY 'dev1pass';
CREATE USER 'read_user1'@'localhost' IDENTIFIED BY 'read_user1pass';
CREATE USER 'read_user2'@'localhost' IDENTIFIED BY 'read_user2pass';
CREATE USER 'rw_user1'@'localhost' IDENTIFIED BY 'rw_user1pass';
```

次に、`GRANT`を使用してユーザーにロールを付与します。

```sql
GRANT 'app_developer' TO 'dev1'@'localhost';
GRANT 'app_read' TO 'read_user1'@'localhost', 'read_user2'@'localhost';
GRANT 'app_read', 'app_write' TO 'rw_user1'@'localhost';
```

ロールをユーザーに付与したり、ロールを取り消したりするには、`SUPER`権限が必要です。

ユーザーにロールを付与することは、ロールをすぐに有効にすることを意味しません。ロールを有効にすることは別の操作です。

以下の操作は、「関係ループ」を形成する可能性があります：

```sql
CREATE USER 'u1', 'u2';
CREATE ROLE 'r1', 'r2';

GRANT 'u1' TO 'u1';
GRANT 'r1' TO 'r1';

GRANT 'r2' TO 'u2';
GRANT 'u2' TO 'r2';
```

TiDBはこの多段階の認可関係をサポートしています。特権の継承を実装するために使用できます。

### ロールの権限を確認

`SHOW GRANTS`ステートメントを使用して、ユーザーに付与されている権限を確認できます。

他のユーザーの権限関連情報を確認するには、`mysql`データベースの`SELECT`権限が必要です。

{{< copyable "sql" >}}

```sql
SHOW GRANTS FOR 'dev1'@'localhost';
```

```
+-------------------------------------------------+
| Grants for dev1@localhost                       |
+-------------------------------------------------+
| GRANT USAGE ON *.* TO `dev1`@`localhost`        |
| GRANT `app_developer`@`%` TO `dev1`@`localhost` |
+-------------------------------------------------+
```

`SHOW GRANTS`に`USING`オプションを使用して、ロールの権限を確認できます：

{{< copyable "sql" >}}

```sql
SHOW GRANTS FOR 'dev1'@'localhost' USING 'app_developer';
```

```sql
+----------------------------------------------------------+
| Grants for dev1@localhost                                |
+----------------------------------------------------------+
| GRANT USAGE ON *.* TO `dev1`@`localhost`                 |
| GRANT ALL PRIVILEGES ON `app_db`.* TO `dev1`@`localhost` |
| GRANT `app_developer`@`%` TO `dev1`@`localhost`          |
+----------------------------------------------------------+
```

{{< copyable "sql" >}}

```sql
SHOW GRANTS FOR 'rw_user1'@'localhost' USING 'app_read', 'app_write';
```

```
+------------------------------------------------------------------------------+
| Grants for rw_user1@localhost                                                |
+------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO `rw_user1`@`localhost`                                 |
| GRANT SELECT, INSERT, UPDATE, DELETE ON `app_db`.* TO `rw_user1`@`localhost` |
| GRANT `app_read`@`%`,`app_write`@`%` TO `rw_user1`@`localhost`               |
+------------------------------------------------------------------------------+
```

{{< copyable "sql" >}}

```sql
SHOW GRANTS FOR 'read_user1'@'localhost' USING 'app_read';
```

```
+--------------------------------------------------------+
| Grants for read_user1@localhost                        |
+--------------------------------------------------------+
| GRANT USAGE ON *.* TO `read_user1`@`localhost`         |
| GRANT SELECT ON `app_db`.* TO `read_user1`@`localhost` |
| GRANT `app_read`@`%` TO `read_user1`@`localhost`       |
+--------------------------------------------------------+
```

`SHOW GRANTS`または`SHOW GRANTS FOR CURRENT_USER()`を使用して、現在のユーザーの権限を確認できます。`SHOW GRANTS`と`SHOW GRANTS FOR CURRENT_USER()`には、以下の点で異なる点があります：

- `SHOW GRANTS`は、現在のユーザーに有効なロールの権限を表示します。
- `SHOW GRANTS FOR CURRENT_USER()`は、有効なロールの権限を表示しません。

### デフォルトのロールを設定

ユーザーにロールを付与した後でないと、ロールは直ちに有効になりません。ユーザーがこのロールを有効にした後にのみ、ロールが所有する権限を使用できます。

ユーザーにデフォルトのロールを設定することができます。ユーザーがログインすると、デフォルトのロールが自動的に有効になります。

{{< copyable "sql" >}}

```sql
SET DEFAULT ROLE
    {NONE | ALL | role [, role ] ...}
    TO user [, user ]
```

例えば、次のステートメントを使用して、`rw_user1@localhost`のデフォルトロールを`app_read`と`app_write`に設定できます：

{{< copyable "sql" >}}

```sql
SET DEFAULT ROLE app_read, app_write TO 'rw_user1'@'localhost';
```

次のステートメントを使用して、`dev1@localhost`のデフォルトロールをすべてのロールに設定できます：

{{< copyable "sql" >}}

```sql
SET DEFAULT ROLE ALL TO 'dev1'@'localhost';
```

次のステートメントを使用して、`dev1@localhost`のデフォルトロールをすべて無効にできます：

{{< copyable "sql" >}}

```sql
SET DEFAULT ROLE NONE TO 'dev1'@'localhost';
```

> **注記:**
>
> デフォルトのロールを設定する前に、そのロールをユーザーに付与する必要があります。

### 現在のセッションでのロールの有効化

現在のセッションでロールを有効にすることができます。

```sql
SET ROLE {
    DEFAULT
  | NONE
  | ALL
  | ALL EXCEPT role [, role ] ...
  | role [, role ] ...
}
```

例えば、`rw_user1`がログインした後、次のステートメントを使用して、現在のセッションでのみ有効な`app_read`と`app_write`のロールを有効にできます：

{{< copyable "sql" >}}

```sql
SET ROLE 'app_read', 'app_write';
```

次の文を使用して、現在のユーザーのデフォルトロールを有効にすることができます。

{{< copyable "sql" >}}

```sql
SET ROLE DEFAULT
```

次の文を使用して、現在のユーザーに付与されたすべてのロールを有効にすることができます。

{{< copyable "sql" >}}

```sql
SET ROLE ALL
```

次の文を使用して、すべてのロールを無効にすることができます。

{{< copyable "sql" >}}

```sql
SET ROLE NONE
```

次の文を使用して、`app_read` を除くすべてのロールを有効にすることができます。

{{< copyable "sql" >}}

```sql
SET ROLE ALL EXCEPT 'app_read'
```

> **注意:**
>
> `SET ROLE` を使用してロールを有効にする場合、このロールは現在のセッションでのみ有効です。

### 現在有効なロールを確認する

現在のユーザーは `CURRENT_ROLE()` 関数を使用して、現在のユーザーが有効にしたロールを確認できます。

例えば、`rw_user1'@'localhost` にデフォルトのロールを付与できます。

{{< copyable "sql" >}}

```sql
SET DEFAULT ROLE ALL TO 'rw_user1'@'localhost';
```

`rw_user1@localhost` がログインした後、以下の文を実行できます。

{{< copyable "sql" >}}

```sql
SELECT CURRENT_ROLE();
```

```
+--------------------------------+
| CURRENT_ROLE()                 |
+--------------------------------+
| `app_read`@`%`,`app_write`@`%` |
+--------------------------------+
```

{{< copyable "sql" >}}

```sql
SET ROLE 'app_read'; SELECT CURRENT_ROLE();
```

```
+----------------+
| CURRENT_ROLE() |
+----------------+
| `app_read`@`%` |
+----------------+
```

### ロールを取り消す

次の文を使用して、`read_user1@localhost` および `read_user2@localhost` ユーザーに付与された `app_read` ロールを取り消すことができます。

{{< copyable "sql" >}}

```sql
REVOKE 'app_read' FROM 'read_user1'@'localhost', 'read_user2'@'localhost';
```

次の文を使用して、`rw_user1@localhost` ユーザーに付与された `app_read` および `app_write` ロールを取り消すことができます。

{{< copyable "sql" >}}

```sql
REVOKE 'app_read', 'app_write' FROM 'rw_user1'@'localhost';
```

ユーザーからロールを取り消す操作はアトミックです。ロールの取り消しに失敗した場合、この操作はロールバックされます。

### 権限を取り消す

`REVOKE` 文は `GRANT` の逆です。`REVOKE` を使用して `app_write` の権限を取り消すことができます。

{{< copyable "sql" >}}

```sql
REVOKE INSERT, UPDATE, DELETE ON app_db.* FROM 'app_write';
```

詳細については、[TiDB Privilege Management](/privilege-management.md) を参照してください。

### ロールを削除する

次の文を使用して、`app_read` および `app_write` ロールを削除できます。

{{< copyable "sql" >}}

```sql
DROP ROLE 'app_read', 'app_write';
```

この操作は、`mysql.user` テーブル内の `app_read` および `app_write` のロールレコードと関連する権限テーブル内の関連レコードを削除し、2 つのロールに関連する権限を終了させます。

ロールを削除するには、`DROP ROLE` または `DROP USER` 権限が必要です。

### 権限テーブル

RBAC システムでは、4 つのシステム[権限テーブル](/privilege-management.md#privilege-table)に加えて、2 つの新しいシステム権限テーブルが導入されています。

- `mysql.role_edges`: ロールとユーザーの認可関係を記録します。
- `mysql.default_roles`: 各ユーザーにデフォルトで有効になっているロールを記録します。

#### `mysql.role_edges`

`mysql.role_edges` には、次のデータが含まれています。

{{< copyable "sql" >}}

```sql
SELECT * FROM mysql.role_edges;
```

```
+-----------+-----------+---------+---------+-------------------+
| FROM_HOST | FROM_USER | TO_HOST | TO_USER | WITH_ADMIN_OPTION |
+-----------+-----------+---------+---------+-------------------+
| %         | r_1       | %       | u_1     | N                 |
+-----------+-----------+---------+---------+-------------------+
1 行が返されました (0.00 秒)
```

- `FROM_HOST` および `FROM_USER` はそれぞれロールのホスト名とユーザー名を示します。
- `TO_HOST` および `TO_USER` は、ロールが付与されるユーザーのホスト名およびユーザー名を示します。

#### `mysql.default_roles`

`mysql.default_roles` は、デフォルトのロールが各ユーザーに対してデフォルトで有効になっているかを示します。

{{< copyable "sql" >}}

```sql
SELECT * FROM mysql.default_roles;
```

```
+------+------+-------------------+-------------------+
| HOST | USER | DEFAULT_ROLE_HOST | DEFAULT_ROLE_USER |
+------+------+-------------------+-------------------+
| %    | u_1  | %                 | r_1               |
| %    | u_1  | %                 | r_2               |
+------+------+-------------------+-------------------+
2 行が返されました (0.00 秒)
```

- `HOST` および `USER` はユーザーのホスト名とユーザー名を示します。
- `DEFAULT_ROLE_HOST` および `DEFAULT_ROLE_USER` はデフォルトのロールのホスト名とユーザー名をそれぞれ示します。

### 参照

RBAC、ユーザー管理、および権限管理は密接に関連していますので、以下のリソースで操作の詳細を参照できます:

- [TiDB Privilege Management](/privilege-management.md)
- [TiDB User Account Management](/user-account-management.md)