---
title: SHOW PRIVILEGES | TiDB SQL ステートメントリファレンス
summary: TiDBデータベースのSHOW PRIVILEGESの使用方法の概要。
aliases: ['/docs/dev/sql-statements/sql-statement-show-privileges/', '/docs/dev/reference/sql/statements/show-privileges/']
---

# SHOW PRIVILEGES

このステートメントは、TiDBで割り当て可能な権限の一覧を表示します。これは静的なリストであり、現在のユーザーの権限を反映しません。

## 概要

**ShowStmt:**

![ShowStmt](/media/sqlgram/ShowStmt.png)

## 例

```sql
mysql> show privileges;
+---------------------------------+---------------------------------------+-------------------------------------------------------+
| Privilege                       | Context                               | Comment                                               |
+---------------------------------+---------------------------------------+-------------------------------------------------------+
| Alter                           | Tables                                | テーブルの変更                                         |
| Alter routine                   | Functions,Procedures                  | 格納された関数/手続きの変更または削除              |
| Config                          | Server Admin                          | SHOW CONFIG および SET CONFIG ステートメントの使用     |
| Create                          | Databases,Tables,Indexes              | 新しいデータベースおよびテーブルの作成               |
| Create routine                  | Databases                             | CREATE FUNCTION/PROCEDURE の使用                      |
| Create role                     | Server Admin                          | 新しいロールの作成                                     |
| Create temporary tables         | Databases                             | CREATE TEMPORARY TABLE の使用                         |
| Create view                     | Tables                                | 新しいビューの作成                                     |
| Create user                     | Server Admin                          | 新しいユーザーの作成                                   |
| Delete                          | Tables                                | 既存の行の削除                                         |
| Drop                            | Databases,Tables                      | データベース、テーブル、およびビューの削除            |
| Drop role                       | Server Admin                          | ロールの削除                                           |
| Event                           | Server Admin                          | イベントの作成、変更、削除および実行                  |
| Execute                         | Functions,Procedures                  | 格納されたルーチンの実行                             |
| File                            | File access on server                 | サーバー上のファイルの読み取りおよび書き込み           |
| Grant option                    | Databases,Tables,Functions,Procedures | 所有する権限を他のユーザーに与える                    |
| Index                           | Tables                                | インデックスの作成または削除                           |
| Insert                          | Tables                                | テーブルにデータの挿入                                 |
| Lock tables                     | Databases                             | LOCK TABLES の使用（SELECT 権限と併用）               |
| Process                         | Server Admin                          | 現在実行中のクエリのプレーンテキストを表示する        |
| Proxy                           | Server Admin                          | プロキシユーザーを有効にする                          |
| References                      | Databases,Tables                      | テーブルへの参照                                        |
| Reload                          | Server Admin                          | テーブル、ログ、および権限を再読み込みまたはリフレッシュ|
| Replication client              | Server Admin                          | スレーブまたはマスターサーバーの場所を問い合わせる      |
| Replication slave               | Server Admin                          | マスターからのバイナリログイベントを読む                |
| Select                          | Tables                                | テーブルから行を取得                                   |
| Show databases                  | Server Admin                          | SHOW DATABASES ですべてのデータベースを表示する       |
| Show view                       | Tables                                | SHOW CREATE VIEW でビューを表示する                    |
| Shutdown                        | Server Admin                          | サーバーをシャットダウンする                           |
| Super                           | Server Admin                          | KILLスレッド、SET GLOBAL、CHANGE MASTERなどの使用     |
| Trigger                         | Tables                                | トリガーの使用                                         |
| Create tablespace               | Server Admin                          | 表領域の作成、変更、削除                             |
| Update                          | Tables                                | 既存の行を更新                                         |
| Usage                           | Server Admin                          | 特定の権限を持たず、接続のみが許可される                |
| BACKUP_ADMIN                    | Server Admin                          |                                                       |
| RESTORE_ADMIN                   | Server Admin                          |                                                       |
| SYSTEM_USER                     | Server Admin                          |                                                       |
| SYSTEM_VARIABLES_ADMIN          | Server Admin                          |                                                       |
| ROLE_ADMIN                      | Server Admin                          |                                                       |
| CONNECTION_ADMIN                | Server Admin                          |                                                       |
| PLACEMENT_ADMIN                 | Server Admin                          |                                                       |
| DASHBOARD_CLIENT                | Server Admin                          |                                                       |
| RESTRICTED_TABLES_ADMIN         | Server Admin                          |                                                       |
| RESTRICTED_STATUS_ADMIN         | Server Admin                          |                                                       |
| RESTRICTED_VARIABLES_ADMIN      | Server Admin                          |                                                       |
| RESTRICTED_USER_ADMIN           | Server Admin                          |                                                       |
| RESTRICTED_CONNECTION_ADMIN     | Server Admin                          |                                                       |
| RESTRICTED_REPLICA_WRITER_ADMIN | Server Admin                          |                                                       |
| RESOURCE_GROUP_ADMIN            | Server Admin                          |                                                       |
+---------------------------------+---------------------------------------+-------------------------------------------------------+
49 行の結果 (0.00 秒)
```

## MySQLとの互換性

TiDBにおける`SHOW PRIVILEGES`ステートメントはMySQLと完全に互換性があります。互換性に違いがある場合は、[バグを報告してください](https://docs.pingcap.com/tidb/stable/support)。

## 関連情報

* [SHOW GRANTS](/sql-statements/sql-statement-show-grants.md)
* [`GRANT <privileges>`](/sql-statements/sql-statement-grant-privileges.md)