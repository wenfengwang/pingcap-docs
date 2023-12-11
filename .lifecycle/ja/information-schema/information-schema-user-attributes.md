---
title: USER_ATTRIBUTES
summary: `USER_ATTRIBUTES` INFORMATION_SCHEMA テーブルについて学びます。

# USER_ATTRIBUTES

`USER_ATTRIBUTES` テーブルは、ユーザーコメントやユーザーアトリビュートに関する情報を提供します。この情報は `mysql.user` システムテーブルから取得されます。

```sql
USE information_schema;
DESC user_attributes;
```

```sql
+-----------+--------------+------+------+---------+-------+
| Field     | Type         | Null | Key  | Default | Extra |
+-----------+--------------+------+------+---------+-------+
| USER      | varchar(32)  | NO   |      | NULL    |       |
| HOST      | varchar(255) | NO   |      | NULL    |       |
| ATTRIBUTE | longtext     | YES  |      | NULL    |       |
+-----------+--------------+------+------+---------+-------+
3 rows in set (0.00 sec)
```

`USER_ATTRIBUTES` テーブルの各フィールドについて説明します。

* `USER`: ユーザー名。
* `HOST`: ユーザーがTiDBに接続できるホスト。このフィールドの値が `％` の場合、ユーザーは任意のホストからTiDBに接続できます。
* `ATTRIBUTE`: [`CREATE USER`](/sql-statements/sql-statement-create-user.md) や [`ALTER USER`](/sql-statements/sql-statement-alter-user.md) ステートメントで設定される、ユーザーのコメントや属性。

以下は例です：

```sql
CREATE USER testuser1 COMMENT 'This user is created only for test';
CREATE USER testuser2 ATTRIBUTE '{"email": "user@pingcap.com"}';
SELECT * FROM information_schema.user_attributes;
```

```sql
+-----------+------+---------------------------------------------------+
| USER      | HOST | ATTRIBUTE                                         |
+-----------+------+---------------------------------------------------+
| root      | %    | NULL                                              |
| testuser1 | %    | {"comment": "This user is created only for test"} |
| testuser2 | %    | {"email": "user@pingcap.com"}                     |
+-----------+------+---------------------------------------------------+
3 rows in set (0.00 sec)
```