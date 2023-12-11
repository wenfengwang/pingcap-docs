---
title: SESSION_CONNECT_ATTRS
summary: `SESSION_CONNECT_ATTRS` パフォーマンススキーマテーブルについて学ぶ。

# SESSION\_CONNECT\_ATTRS

`SESSION_CONNECT_ATTRS` テーブルは接続属性に関する情報を提供します。セッション属性は、接続を確立する際にクライアントによって送信されるキーと値のペアです。

一般的な属性:

| 属性名            | 例              | 説明                       |
|-------------------|---------------|----------------------------|
| `_client_name`    | `libmysql`    | クライアントライブラリ名     |
| `_client_version` | `8.0.33`      | クライアントライブラリのバージョン |
| `_os`             | `Linux`       | オペレーティングシステム     |
| `_pid`            | `712927`      | プロセスID                   |
| `_platform`       | `x86_64`      | CPUアーキテクチャ            |
| `program_name`    | `mysqlsh`     | プログラム名                 |

`SESSION_CONNECT_ATTRS` テーブルの列を以下のように表示できます:

{{< copyable "sql" >}}

```sql
USE performance_schema;
DESCRIBE session_connect_attrs;
```

```
+------------------+---------------------+------+-----+---------+-------+
| Field            | Type                | Null | Key | Default | Extra |
+------------------+---------------------+------+-----+---------+-------+
| PROCESSLIST_ID   | bigint(20) unsigned | NO   |     | NULL    |       |
| ATTR_NAME        | varchar(32)         | NO   |     | NULL    |       |
| ATTR_VALUE       | varchar(1024)       | YES  |     | NULL    |       |
| ORDINAL_POSITION | int(11)             | YES  |     | NULL    |       |
+------------------+---------------------+------+-----+---------+-------+
```

`SESSION_CONNECT_ATTRS` テーブルに格納されているセッション属性の情報は以下のように表示できます:

{{< copyable "sql" >}}

```sql
USE performance_schema;
TABLE SESSION_CONNECT_ATTRS;
```

```
+----------------+-----------------+------------+------------------+
| PROCESSLIST_ID | ATTR_NAME       | ATTR_VALUE | ORDINAL_POSITION |
+----------------+-----------------+------------+------------------+
|        2097154 | _client_name    | libmysql   |                0 |
|        2097154 | _client_version | 8.1.0      |                1 |
|        2097154 | _os             | Linux      |                2 |
|        2097154 | _pid            | 1299203    |                3 |
|        2097154 | _platform       | x86_64     |                4 |
|        2097154 | program_name    | mysqlsh    |                5 |
+----------------+-----------------+------------+------------------+
```

`SESSION_CONNECT_ATTRS` テーブルのフィールドは以下のように説明されています:

* `PROCESSLIST_ID`: セッションのプロセスリストID。
* `ATTR_NAME`: 属性名。
* `ATTR_VALUE`: 属性値。
* `ORDINAL_POSITION`: 名前/値のペアの序数位置。