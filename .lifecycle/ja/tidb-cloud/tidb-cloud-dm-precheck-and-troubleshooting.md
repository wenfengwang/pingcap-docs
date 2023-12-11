---
title: データ移行における事前チェックエラー、移行エラー、アラート
summary: データ移行を行う際の、事前チェックエラー、移行エラー、アラートの解決方法について学びます。
---

# データ移行における事前チェックエラー、移行エラー、アラート

本書では、データ移行の際に事前チェックエラーを解決したり、移行エラーをトラブルシューティングしたり、アラートを購読する方法について説明します。

## 事前チェックエラーと解決策

このセクションでは、データ移行中に表示される事前チェックエラーとそれに対応する解決策について説明します。これらのエラーは、[Data Migrationを使用してデータを移行](/tidb-cloud/migrate-from-mysql-using-data-migration.md)する際に**事前チェック**ページに表示されます。

解決策は上流データベースによって異なります。

### エラーメッセージ：mysqlのserver_idが0より大きいか確認してください

- Amazon Aurora MySQLまたはAmazon RDS: `server_id`はデフォルトで設定されています。これを設定する必要はありません。Amazon Aurora MySQLライターインスタンスを使用して、完全なデータ移行と増分データ移行の両方をサポートしていることを確認してください。
- MySQL: MySQLの`server_id`を設定するには、[Setting the Replication Source Configuration](https://dev.mysql.com/doc/refman/8.0/en/replication-howto-masterbaseconfig.html)を参照してください。

### エラーメッセージ：mysqlのbinlogが有効になっているか確認してください

- Amazon Aurora MySQL: [Amazon Aurora MySQL-Compatible clusterのバイナリログを有効にする方法](https://aws.amazon.com/premiumsupport/knowledge-center/enable-binary-logging-aurora/?nc1=h_ls)を参照してください。完全なデータ移行と増分データ移行の両方をサポートするために、Amazon Aurora MySQLライターインスタンスを使用していることを確認してください。
- Amazon RDS: [Configuring MySQL binary logging](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_LogAccess.MySQL.BinaryFormat.html)を参照してください。
- Google Cloud SQL for MySQL: GoogleはMySQLマスターデータベースのポイントインタイムリカバリを介してバイナリログを有効にします。[Enable point-in-time recovery](https://cloud.google.com/sql/docs/mysql/backup-recovery/pitr#enablingpitr)を参照してください。
- MySQL: [Setting the Replication Source Configuration](https://dev.mysql.com/doc/refman/8.0/en/replication-howto-masterbaseconfig.html)を参照してください。

### エラーメッセージ：mysqlのbinlog_formatがROWになっているか確認してください

- Amazon Aurora MySQL: [Amazon Aurora MySQL-Compatible clusterのバイナリログを有効にする方法](https://aws.amazon.com/premiumsupport/knowledge-center/enable-binary-logging-aurora/?nc1=h_ls)を参照してください。完全なデータ移行と増分データ移行の両方をサポートするために、Amazon Aurora MySQLライターインスタンスを使用していることを確認してください。
- Amazon RDS: [Configuring MySQL binary logging](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_LogAccess.MySQL.BinaryFormat.html)を参照してください。
- MySQL: `set global binlog_format=ROW;`を実行してください。[Setting The Binary Log Format](https://dev.mysql.com/doc/refman/8.0/en/binary-log-setting.html)を参照してください。

### エラーメッセージ：mysqlのbinlog_row_imageがFULLになっているか確認してください

- Amazon Aurora MySQL: `binlog_row_image`は設定できません。この事前チェック項目は失敗しません。完全なデータ移行と増分データ移行をサポートするために、Amazon Aurora MySQLライターインスタンスを使用していることを確認してください。
- Amazon RDS: プロセスは`binlog_format`パラメータを設定するのと似ていますが、変更するパラメータは`binlog_format`の代わりに`binlog_row_image`です。[Configuring MySQL binary logging](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_LogAccess.MySQL.BinaryFormat.html)を参照してください。
- MySQL: `set global binlog_row_image = FULL;`を実行してください。[Binary Logging Options and Variables](https://dev.mysql.com/doc/refman/8.0/en/replication-options-binary-log.html#sysvar_binlog_row_image)を参照してください。

### エラーメッセージ：移行されたdbsがbinlog_do_db/binlog_ignore_dbにあるか確認してください

上流データベースでバイナリログが有効になっていることを確認してください。[Check whether mysql binlog is enabled](#error-message-check-whether-mysql-binlog-is-enabled)を参照して、その後、次のメッセージに従って問題を解決してください：

- メッセージが`These dbs xxx are not in binlog_do_db xxx`に似ている場合は、移行したいすべてのデータベースがリストに含まれていることを確認してください。[--binlog-do-db=db_name](https://dev.mysql.com/doc/refman/8.0/en/replication-options-binary-log.html#option_mysqld_binlog-do-db)を参照してください。
- メッセージが`These dbs xxx are in binlog_ignore_db xxx`に似ている場合は、移行したいすべてのデータベースが無視リストに含まれていないことを確認してください。[--binlog-ignore-db=db_name](https://dev.mysql.com/doc/refman/8.0/en/replication-options-binary-log.html#option_mysqld_binlog-ignore-db)を参照してください。

Amazon Aurora MySQLの場合、この事前チェック項目は失敗しません。完全なデータ移行と増分データ移行をサポートするために、Amazon Aurora MySQLライターインスタンスを使用していることを確認してください。

Amazon RDSの場合、以下のパラメータを変更する必要があります：`replicate-do-db`、`replicate-do-table`、`replicate-ignore-db`、`replicate-ignore-table`。[Configuring MySQL binary logging](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_LogAccess.MySQL.BinaryFormat.html)を参照してください。

### エラーメッセージ：接続の同時実行数がデータベースの最大接続制限を超えていないか確認してください

上流データベースでエラーが発生した場合は、`max_connections`を次のように設定してください：

- Amazon Aurora MySQL: プロセスは`binlog_format`を設定するのと似ていますが、変更するパラメータは`max_connections`です。[How do I turn on binary logging for my Amazon Aurora MySQL-Compatible cluster](https://aws.amazon.com/premiumsupport/knowledge-center/enable-binary-logging-aurora/?nc1=h_ls)を参照してください。
- Amazon RDS: プロセスは`binlog_format`を設定するのと似ていますが、変更するパラメータは`max_connections`です。[Configuring MySQL binary logging](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_LogAccess.MySQL.BinaryFormat.html)を参照してください。
- MySQL: [max_connections](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_max_connections)のドキュメントに従って`max_connections`を構成してください。

TiDB Cloudクラスタでエラーが発生した場合は、[max_connections](https://docs.pingcap.com/tidb/stable/system-variables#max_connections)のドキュメントに従って`max_connections`を構成してください。

## 移行エラーと解決策

このセクションでは、移行中に遭遇する問題とそれに対する解決策について説明します。これらのエラーメッセージは**移行ジョブの詳細**ページに表示されます。

### エラーメッセージ："移行に必要なバイナリログはソースデータベース上に存在しなくなりました。移行が成功するために、バイナリログファイルを十分な期間保持するようにしてください。"

このエラーは、増分移行に必要なバイナリログがクリーンアップされ、新しいタスクを作成することでのみ復元できることを意味します。

増分移行に必要なバイナリログが存在することを確認してください。バイナリログの期間を延長するために`expire_logs_days`を構成することをお勧めします。一部の移行ジョブで必要な場合には`purge binary log`を使用してバイナリログをクリーンアップしないでください。

### エラーメッセージ："指定されたパラメータを使用してソースデータベースに接続できませんでした。ソースデータベースが起動しており、指定されたパラメータを使用して接続できることを確認してください。"

このエラーは、ソースデータベースへの接続に失敗したことを意味します。ソースデータベースが利用可能であり、指定されたパラメータを使用して接続できることを確認した後、**Restart**をクリックしてタスクを再開できます。

### 移行タスクが中断され、「driver: bad connection」または「invalid connection」というエラーを含んでいます

このエラーは、下流TiDBクラスタへの接続に失敗したことを意味します。下流TiDBクラスタが正常な状態（「Available」および「Modifying」を含む）であることを確認し、ジョブで指定されたユーザー名とパスワードを使用して接続できることを確認した後、**Restart**をクリックしてタスクを再開できます。

### エラーメッセージ："指定されたユーザーとパスワードを使用してTiDBクラスタに接続できませんでした。TiDBクラスタが利用可能であり、指定されたユーザーとパスワードを使用して接続できることを確認してください。"

TiDBクラスタへの接続に失敗しました。TiDBクラスタが正常な状態にあること（「Available」および「Modifying」を含む）を確認し、ジョブで指定されたユーザー名とパスワードを使用して接続できることを確認した後、**Restart**をクリックしてタスクを再開できます。

### エラーメッセージ："TiDBクラスタのストレージが足りません。TiKVノードのストレージを増やしてください。"

TiDBクラスタのストレージが不足しています。[TiKVノードのストレージを増やして](/tidb-cloud/scale-tidb-cluster.md#change-storage)から、**Restart**をクリックしてタスクを再開できます。

### エラーメッセージ："ソースデータベースに接続できませんでした。データベースが利用可能であるか、最大接続数に達していないか、およびジョブで指定されたパラメータを使用して接続できるかを確認してください。"

ソースデータベースへの接続が失敗しました。ソースデータベースが起動しており、データベースの接続数が上限に達していないこと、およびジョブで指定されたパラメータを使用して接続できることを確認した後、**Restart**をクリックしてジョブを再開できます。

## アラート

TiDB Cloudのアラートメールに登録することで、アラートが発生した際に適時通知を受けることができます。
以下はデータ移行に関するアラートです：

- "データ移行ジョブはデータのエクスポート中にエラーが発生しました"

    推奨アクション：データ移行ページのエラーメッセージを確認し、ヘルプについては[移行エラーと解決方法](#migration-errors-and-solutions)を参照してください。

- "データ移行ジョブはデータのインポート中にエラーが発生しました"

    推奨アクション：データ移行ページのエラーメッセージを確認し、ヘルプについては[移行エラーと解決方法](#migration-errors-and-solutions)を参照してください。

- "データ移行ジョブは増分データ移行中にエラーが発生しました"

    推奨アクション：データ移行ページのエラーメッセージを確認し、ヘルプについては[移行エラーと解決方法](#migration-errors-and-solutions)を参照してください。

- "増分移行中、データ移行ジョブが6時間以上一時停止しています"

    推奨アクション：データ移行ジョブを再開するか、このアラートを無視してください。

- "レプリケーション遅延が10分を超え、20分以上増加しています"

    推奨アクション：[TiDB Cloud Support](/tidb-cloud/tidb-cloud-support.md)に連絡してヘルプを受けてください。

これらのアラートに対処するための支援が必要な場合は、[TiDB Cloud Support](/tidb-cloud/tidb-cloud-support.md)に連絡してコンサルテーションを受けてください。

アラートメールの購読方法についてさらに情報が必要な場合は、[TiDB Cloud Built-in Alerting](/tidb-cloud/monitor-built-in-alerting.md)を参照してください。

## 関連情報

- [MySQL互換データベースをTiDB Cloudにデータ移行する方法](/tidb-cloud/migrate-from-mysql-using-data-migration.md)