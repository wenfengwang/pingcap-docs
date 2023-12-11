---
title: 権限管理
summary: 権限の管理方法を学ぶ。
aliases: ['/docs/dev/privilege-management/','/docs/dev/reference/security/privilege-system/']
---

# 権限管理

TiDBはMySQL 5.7の権限管理システムをサポートしており、構文や権限の種類もサポートしています。また、以下のMySQL 8.0からの機能もサポートしています。

* TiDB 3.0からはSQLロールがサポートされます。
* TiDB 5.1からはダイナミック権限がサポートされます。

このドキュメントでは、権限に関連するTiDB操作、TiDB操作に必要な権限、および権限システムの実装を紹介します。

## 権限に関連する操作

### 権限の付与

`GRANT`ステートメントはユーザーアカウントに権限を付与します。

例えば、次のステートメントを使用して、`xxx`ユーザーに`test`データベースの読み取り権限を付与します。

```sql
GRANT SELECT ON test.* TO 'xxx'@'%';
```

次のステートメントを使用して、`xxx`ユーザーに全てのデータベースへの全権限を付与します。

```sql
GRANT ALL PRIVILEGES ON *.* TO 'xxx'@'%';
```

デフォルトでは、ユーザーが存在しない場合、`GRANT`ステートメントはエラーを返します。この動作は、SQLモード`NO_AUTO_CREATE_USER`が指定されているかどうかに依存します。

```sql
mysql> SET sql_mode=DEFAULT;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @@sql_mode;
+-------------------------------------------------------------------------------------------------------------------------------------------+
| @@sql_mode                                                                                                                                |
+-------------------------------------------------------------------------------------------------------------------------------------------+
| ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION |
+-------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> SELECT * FROM mysql.user WHERE user='idontexist';
Empty set (0.00 sec)

mysql> GRANT ALL PRIVILEGES ON test.* TO 'idontexist';
ERROR 1105 (HY000): You are not allowed to create a user with GRANT

mysql> SELECT user,host,authentication_string FROM mysql.user WHERE user='idontexist';
Empty set (0.00 sec)
```

以下の例では、SQLモード`NO_AUTO_CREATE_USER`が設定されていなかったため、ユーザー`idontexist`が自動的に空のパスワードで作成されますが、これは**推奨しません**。ユーザー名を誤入力すると、新しいユーザーが空のパスワードで作成されるため、セキュリティリスクが発生します。

```sql
mysql> SET @@sql_mode='ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION';
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @@sql_mode;
+-----------------------------------------------------------------------------------------------------------------------+
| @@sql_mode                                                                                                            |
+-----------------------------------------------------------------------------------------------------------------------+
| ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION |
+-----------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> SELECT * FROM mysql.user WHERE user='idontexist';
Empty set (0.00 sec)

mysql> GRANT ALL PRIVILEGES ON test.* TO 'idontexist';
Query OK, 1 row affected (0.05 sec)

mysql> SELECT user,host,authentication_string FROM mysql.user WHERE user='idontexist';
+------------+------+-----------------------+
| user       | host | authentication_string |
+------------+------+-----------------------+
| idontexist | %    |                       |
+------------+------+-----------------------+
1 row in set (0.01 sec)
```

`GRANT`を使用してデータベースに権限を付与する際に、ファジー一致を使用することができます。

```sql
mysql> GRANT ALL PRIVILEGES ON `te%`.* TO genius;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT user,host,db FROM mysql.db WHERE user='genius';
+--------|------|-----+
| user   | host | db  |
+--------|------|-----+
| genius | %    | te% |
+--------|------|-----+
1 row in set (0.00 sec)
```

この例では、`te%`の`%`により、`te`で始まるすべてのデータベースに権限が付与されます。

### 権限の取り消し

`REVOKE`ステートメントを使用すると、システム管理者はユーザーアカウントから権限を取り消すことができます。

`REVOKE`ステートメントは`REVOKE`ステートメントと対応します:

```sql
REVOKE ALL PRIVILEGES ON `test`.* FROM 'genius'@'localhost';
```

> **ノート:**
>
> 権限を取り消すには、完全一致が必要です。一致する結果が見つからない場合、エラーが表示されます:

```sql
mysql> REVOKE ALL PRIVILEGES ON `te%`.* FROM 'genius'@'%';
ERROR 1141 (42000): There is no such grant defined for user 'genius' on host '%'
```

ファジー一致、エスケープ、文字列、識別子について:

```sql
mysql> GRANT ALL PRIVILEGES ON `te\%`.* TO 'genius'@'localhost';
Query OK, 0 rows affected (0.00 sec)
```

この例では、`te%`という名前のデータベースを完全一致で見つけます。ここでは、`%`をワイルドカードとして考慮されないように、`%`の前にエスケープ文字`\`を使用しています。

文字列はシングルクォーテーション（''）で囲まれ、識別子はバッククォート（``）で囲まれます。以下の違いを参照してください:

```sql
mysql> GRANT ALL PRIVILEGES ON 'test'.* TO 'genius'@'localhost';
ERROR 1064 (42000): You have an error in your SQL syntax; check the
manual that corresponds to your MySQL server version for the right
syntax to use near ''test'.* to 'genius'@'localhost'' at line 1

mysql> GRANT ALL PRIVILEGES ON `test`.* TO 'genius'@'localhost';
Query OK, 0 rows affected (0.00 sec)
```

特殊キーワードをテーブル名として使用したい場合は、バッククォート（``）で囲んでください。例:

```sql
mysql> CREATE TABLE `select` (id int);
Query OK, 0 rows affected (0.27 sec)
```

### ユーザーへの付与済み権限の確認

`SHOW GRANTS`ステートメントを使用して、ユーザーに付与された権限を確認できます。例:

```sql
SHOW GRANTS; -- 現在のユーザーに付与された権限を表示

+-------------------------------------------------------------+
| Grants for User                                             |
+-------------------------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION |
+-------------------------------------------------------------+
SHOW GRANTS FOR 'root'@'%'; -- 特定のユーザーに付与された権限を表示
```

例えば、`rw_user@192.168.%`というユーザーを作成し、`test.write_table`テーブルの書き込み権限とグローバルな読み取り権限を付与します。

```sql
CREATE USER `rw_user`@`192.168.%`;
GRANT SELECT ON *.* TO `rw_user`@`192.168.%`;
GRANT INSERT, UPDATE ON `test`.`write_table` TO `rw_user`@`192.168.%`;
```

`rw_user@192.168.%`ユーザーに付与された権限を表示します:

```sql
SHOW GRANTS FOR `rw_user`@`192.168.%`;

+------------------------------------------------------------------+
| Grants for rw_user@192.168.%                                     |
+------------------------------------------------------------------+
| GRANT Select ON *.* TO 'rw_user'@'192.168.%'                     |
| GRANT Insert,Update ON test.write_table TO 'rw_user'@'192.168.%' |
+------------------------------------------------------------------+
```

### ダイナミック権限

v5.1以降、TiDBでは、MySQL 8.0から借りたダイナミック権限をサポートしています。ダイナミック権限は`SUPER`権限を置き換えて、特定の操作に対する細かいアクセスを実装することを意図しています。例えば、ダイナミック権限を使用することで、システム管理者は`BACKUP`および`RESTORE`の操作しか実行できないユーザーアカウントを作成できます。

ダイナミック権限には以下が含まれます:

* `BACKUP_ADMIN`
* `RESTORE_ADMIN`
* `SYSTEM_USER`
* `SYSTEM_VARIABLES_ADMIN`
* `ROLE_ADMIN`
* `CONNECTION_ADMIN`
* `PLACEMENT_ADMIN`: 権限所有者に、配置ポリシーを作成、変更、削除する権限を付与します。
* `DASHBOARD_CLIENT`: 権限所有者にTiDBダッシュボードにログインする権限を付与します。
* `RESTRICTED_TABLES_ADMIN`: SEMが有効になっている場合に、権限所有者にシステムテーブルを表示する権限を付与します。
* `RESTRICTED_STATUS_ADMIN`: SEMが有効になっている場合に、権限所有者に[`SHOW [GLOBAL|SESSION] STATUS`](/sql-statements/sql-statement-show-status.md)で全ステータス変数を表示する権限を付与します。
* `RESTRICTED_VARIABLES_ADMIN`: SEMが有効になっている場合に、権限所有者にすべてのシステム変数を表示する権限を付与します。
* `RESTRICTED_USER_ADMIN`: SEMが有効になっている場合に、権限所有者がSUPERユーザーによってアクセス権を取り消されないようにする権限を付与します。
* `RESTRICTED_CONNECTION_ADMIN`: `RESTRICTED_USER_ADMIN`ユーザーの接続を切断する権限を付与します。この権限は`KILL`ステートメントおよび`KILL TIDB`ステートメントに影響します。
* `RESTRICTED_REPLICA_WRITER_ADMIN`は、TiDBクラスタで読み取り専用モードが有効になっているときにも、特権所有者が書き込みや更新操作を実行できるようにします。詳細については、[`tidb_restricted_read_only`](/system-variables.md#tidb_restricted_read_only-new-in-v520)をご覧ください。

ダイナミック特権の全セットを表示するには、`SHOW PRIVILEGES`ステートメントを実行します。プラグインが新しい特権を追加できるため、割り当て可能な特権のリストはTiDBのインストールに基づいて異なる場合があります。

## `SUPER`特権

- `SUPER`特権を持つユーザーは、ほとんどの操作を実行できます。デフォルトでは、`root`ユーザーだけがこの特権を持っています。他のユーザーにこの特権を付与する際には注意してください。
- `SUPER`特権は、[MySQL 8.0で非推奨](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#dynamic-privileges-migration-from-super)とされており、より細かいアクセス制御を提供するために[ダイナミック特権](#dynamic-privileges)で置き換えられます。

## TiDB操作に必要な特権

TiDBユーザーの特権は、`INFORMATION_SCHEMA.USER_PRIVILEGES`テーブルで確認できます。例：

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.USER_PRIVILEGES WHERE grantee = "'root'@'%'";
+------------+---------------+-------------------------+--------------+
| GRANTEE    | TABLE_CATALOG | PRIVILEGE_TYPE          | IS_GRANTABLE |
+------------+---------------+-------------------------+--------------+
| 'root'@'%' | def           | Select                  | YES          |
| 'root'@'%' | def           | Insert                  | YES          |
| 'root'@'%' | def           | Update                  | YES          |
| 'root'@'%' | def           | Delete                  | YES          |
| 'root'@'%' | def           | Create                  | YES          |
| 'root'@'%' | def           | Drop                    | YES          |
| 'root'@'%' | def           | Process                 | YES          |
| 'root'@'%' | def           | References              | YES          |
| 'root'@'%' | def           | Alter                   | YES          |
| 'root'@'%' | def           | Show Databases          | YES          |
| 'root'@'%' | def           | Super                   | YES          |
| 'root'@'%' | def           | Execute                 | YES          |
| 'root'@'%' | def           | Index                   | YES          |
| 'root'@'%' | def           | Create User             | YES          |
| 'root'@'%' | def           | Create Tablespace       | YES          |
| 'root'@'%' | def           | Trigger                 | YES          |
| 'root'@'%' | def           | Create View             | YES          |
| 'root'@'%' | def           | Show View               | YES          |
| 'root'@'%' | def           | Create Role             | YES          |
| 'root'@'%' | def           | Drop Role               | YES          |
| 'root'@'%' | def           | CREATE TEMPORARY TABLES | YES          |
| 'root'@'%' | def           | LOCK TABLES             | YES          |
| 'root'@'%' | def           | CREATE ROUTINE          | YES          |
| 'root'@'%' | def           | ALTER ROUTINE           | YES          |
| 'root'@'%' | def           | EVENT                   | YES          |
| 'root'@'%' | def           | SHUTDOWN                | YES          |
| 'root'@'%' | def           | RELOAD                  | YES          |
| 'root'@'%' | def           | FILE                    | YES          |
| 'root'@'%' | def           | CONFIG                  | YES          |
| 'root'@'%' | def           | REPLICATION CLIENT      | YES          |
| 'root'@'%' | def           | REPLICATION SLAVE       | YES          |
+------------+---------------+-------------------------+--------------+
31 rows in set (0.00 sec)
```

### ALTER

全ての`ALTER`ステートメントについて、ユーザーは該当するテーブルに対する`ALTER`特権を持っている必要があります。
`ALTER...DROP`および`ALTER...RENAME TO`以外のステートメントについては、対応するテーブルに対する`INSERT`および`CREATE`特権が必要です。
`ALTER...DROP`ステートメントについては、対応するテーブルに対する`DROP`特権が必要です。
`ALTER...RENAME TO`ステートメントについては、リネーム前のテーブルに対する`DROP`特権と、リネーム後のテーブルに対する`CREATE`および`INSERT`特権が必要です。

> **Note:**
>
> MySQL 5.7のドキュメントによると、`INSERT`および`CREATE`特権がテーブルの`ALTER`操作を行うために必要とされています。ただし、MySQL 5.7.25では、この場合には`ALTER`特権のみが必要です。現在、TiDBの`ALTER`特権は、MySQLでの実際の動作と一致しています。

### BACKUP

`SUPER`または`BACKUP_ADMIN`特権が必要です。

### CANCEL IMPORT JOB

他のユーザーが作成したジョブをキャンセルするには、`SUPER`特権が必要です。それ以外の場合は、現在のユーザーが作成したジョブのみキャンセルできます。

### CREATE DATABASE

データベースに対する`CREATE`特権が必要です。

### CREATE INDEX

テーブルに対する`INDEX`特権が必要です。

### CREATE TABLE

テーブルに対する`CREATE`特権が必要です。

`CREATE TABLE...LIKE...`ステートメントを実行するには、テーブルに対する`SELECT`特権が必要です。

### CREATE VIEW

`CREATE VIEW`特権が必要です。

> **Note:**
>
> 現在のユーザーがビューを作成していない場合、`CREATE VIEW`および`SUPER`特権の両方が必要です。

### DROP DATABASE

テーブルに対する`DROP`特権が必要です。

### DROP INDEX

テーブルに対する`INDEX`特権が必要です。

### DROP TABLES

テーブルに対する`DROP`特権が必要です。

### IMPORT INTO

対象テーブルに対する`SELECT`、`UPDATE`、`INSERT`、`DELETE`、`ALTER`特権が必要です。TiDBにローカルに保存されたファイルをインポートする場合は、`FILE`特権も必要です。

### LOAD DATA

テーブルに対する`INSERT`特権が必要です。`REPLACE INTO`を使用する場合は、`DELETE`特権も必要です。

### TRUNCATE TABLE

テーブルに対する`DROP`特権が必要です。

### RENAME TABLE

リネーム前のテーブルに対する`ALTER`および`DROP`特権、リネーム後のテーブルに対する`CREATE`および`INSERT`特権が必要です。

### ANALYZE TABLE

テーブルに対する`INSERT`および`SELECT`特権が必要です。

### LOCK STATS

テーブルに対する`INSERT`および`SELECT`特権が必要です。

### UNLOCK STATS

テーブルに対する`INSERT`および`SELECT`特権が必要です。

### SHOW

`SHOW CREATE TABLE`には、テーブルへの任意の単一の特権が必要です。

`SHOW CREATE VIEW`には、`SHOW VIEW`特権が必要です。

`SHOW GRANTS`には、`mysql`データベースへの`SELECT`特権が必要です。対象ユーザーが現在のユーザーの場合、`SHOW GRANTS`には特権が必要ありません。

`SHOW PROCESSLIST`には、他のユーザーに属する接続を表示するために`SUPER`特権が必要です。

`SHOW IMPORT JOB`には、他のユーザーに属するジョブを表示するために`SUPER`特権が必要です。そうでなければ、現在のユーザーが作成したジョブのみが表示されます。

`SHOW STATS_LOCKED`には、`mysql.stats_table_locked`テーブルへの`SELECT`特権が必要です。

### CREATE ROLE/USER

`CREATE ROLE`には、`CREATE ROLE`特権が必要です。

`CREATE USER`には、`CREATE USER`特権が必要です。

### DROP ROLE/USER

`DROP ROLE`には、`DROP ROLE`特権が必要です。

`DROP USER`には、`CREATE USER`特権が必要です。

### ALTER USER

`CREATE USER`特権が必要です。

### GRANT

`GRANT`には、`GRANT`特権と`GRANT`によって付与される特権が必要です。

ユーザーを暗示的に作成するには、追加で`CREATE USER`特権が必要です。

`GRANT ROLE`には、`SUPER`または`ROLE_ADMIN`特権が必要です。

### REVOKE

`REVOKE`には、`GRANT`特権および`REVOKE`ステートメントの対象となる特権が必要です。

`REVOKE ROLE`には、`SUPER`または`ROLE_ADMIN`特権が必要です。

### SET GLOBAL

グローバル変数を設定するには、`SUPER`または`SYSTEM_VARIABLES_ADMIN`特権が必要です。

### ADMIN

`SUPER`特権が必要です。

### SET DEFAULT ROLE

`SUPER`特権が必要です。

### KILL

他のユーザーセッションを終了するには、`SUPER`または`CONNECTION_ADMIN`特権が必要です。

### CREATE RESOURCE GROUP

`SUPER`または`RESOURCE_GROUP_ADMIN`特権が必要です。

### ALTER RESOURCE GROUP

`SUPER`または`RESOURCE_GROUP_ADMIN`特権が必要です。

### DROP RESOURCE GROUP

`SUPER`または`RESOURCE_GROUP_ADMIN`特権が必要です。

### CALIBRATE RESOURCE

`SUPER`または`RESOURCE_GROUP_ADMIN`特権が必要です。

## 特権システムの実装

### 特権テーブル

以下のシステムテーブルは特別であり、全ての特権関連データが格納されています：

- `mysql.user`（ユーザーアカウント、グローバル特権）
- `mysql.db` (database-level privilege)
- `mysql.tables_priv` (table-level privilege)
- `mysql.columns_priv` (column-level privilege; not currently supported)

これらのテーブルには、データの有効範囲と特権情報が含まれています。例えば、`mysql.user` テーブルでは次のようになります。

```sql
mysql> SELECT User,Host,Select_priv,Insert_priv FROM mysql.user LIMIT 1;
+------|------|-------------|-------------+
| User | Host | Select_priv | Insert_priv |
+------|------|-------------|-------------+
| root | %    | Y           | Y           |
+------|------|-------------|-------------+
1 row in set (0.00 sec)
```

このレコードでは、`Host` と `User` が、`root` ユーザーから任意のホスト (`%`) への接続要求が受け入れられることを決定します。`Select_priv` と `Insert_priv` は、ユーザーがグローバルな `Select` と `Insert` 特権を持っていることを意味します。`mysql.user` テーブルの有効範囲はグローバルです。

`mysql.db` の `Host` と `User` が、ユーザーがアクセスできるデータベースを決定します。有効範囲はデータベースです。

> **注意:**
>
> 特権テーブルを `GRANT`、`CREATE USER`、`DROP USER` などの指定された構文を使用して更新することをお勧めします。特権テーブルの直接の編集は特権キャッシュを自動的に更新せず、`FLUSH PRIVILEGES` が実行されるまで予測不能な動作を引き起こす可能性があります。

### 接続検証

クライアントが接続要求を送信すると、TiDBサーバーはログイン操作を検証します。TiDBサーバーはまず`mysql.user` テーブルをチェックします。`User` と `Host` のレコードが接続要求と一致する場合、TiDBサーバーはその後 `authentication_string` を検証します。

ユーザーの識別は、`Host`（接続を開始するホスト）と `User`（ユーザー名）の2つの情報に基づいています。ユーザー名が空でない場合、ユーザー名の完全一致が必要です。

`User`+`Host` は `user` テーブル内の複数の行に一致する場合があります。このようなシナリオに対処するため、`user` テーブルの行はソートされます。クライアントが接続する際に、テーブルの行は1つずつチェックされ、最初に一致した行が使用されます。ソートする際、Host が User より優先されます。

### リクエスト検証

接続が成功すると、リクエスト検証プロセスは操作に特権があるかどうかをチェックします。

データベース関連のリクエスト (`INSERT`、`UPDATE`) の場合、リクエスト検証プロセスは最初に `mysql.user` テーブルでユーザーのグローバル特権をチェックします。特権が付与されている場合、直接アクセスできます。そうでない場合は、`mysql.db` テーブルをチェックします。

`user` テーブルは、デフォルトのデータベースに関係なくグローバルな特権を持っています。例えば、`user` の `DELETE` 特権は、行やテーブル、データベースに適用できます。

`db` テーブルでは、空のユーザーは匿名ユーザー名に一致させるために使用されます。`User` 列にワイルドカードを使用することはできません。`Host` 列と `Db` 列の値にはパターンマッチングに `%` と `_` を使用できます。

`user` および `db` テーブルのデータもメモリに読み込まれる際にソートされます。

`tables_priv` および `columns_priv` の `%` の使用方法は似ていますが、`Db`、`Table_name`、`Column_name` の値には `%` を含めることはできません。読み込まれる際もソートが行われます。

### 有効化タイミング

TiDBが起動すると、いくつかの特権チェックテーブルがメモリに読み込まれ、キャッシュされたデータが特権を検証するために使用されます。`GRANT`、`REVOKE`、`CREATE USER`、`DROP USER` などの特権管理ステートメントを実行すると、すぐに有効になります。

`INSERT`、`DELETE`、`UPDATE` などのステートメントを使用して `mysql.user` などのテーブルを手動で編集すると、すぐには有効になりません。これはMySQLと互換性があり、以下のステートメントで特権キャッシュを更新できます。

```sql
FLUSH PRIVILEGES;
```