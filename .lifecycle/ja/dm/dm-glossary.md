---
title: TiDBデータ移行用語集
summary: TiDBデータ移行で使用される用語を学ぶ
aliases: ['/docs/tidb-data-migration/dev/glossary/']
---

# TiDBデータ移行用語集

この文書では、TiDB Data Migration（DM）のログ、モニタリング、設定、およびドキュメントで使用される用語をリストします。

## B

### バイナリログ

TiDB DMでは、バイナリログはTiDBデータベースで生成されたバイナリログファイルを指します。MySQLまたはMariaDBと同じ意味合いです。詳細は[MySQL Binary Log](https://dev.mysql.com/doc/dev/mysql-server/latest/page_protocol_replication.html)、[MariaDB Binary Log](https://mariadb.com/kb/en/library/binary-log/)を参照してください。

### バイナリログイベント

バイナリログイベントは、MySQLまたはMariaDBサーバーインスタンスで行われたデータ変更に関する情報です。これらのバイナリログイベントはバイナリログファイルに保存されます。詳細は[MySQL Binlog Event](https://dev.mysql.com/doc/dev/mysql-server/latest/page_protocol_replication_binlog_event.html)、[MariaDB Binlog Event](https://mariadb.com/kb/en/library/1-binlog-events/)を参照してください。

### バイナリログイベントフィルター

[バイナリログイベントフィルター](/dm/dm-binlog-event-filter.md)は、ブロックと許可リストのフィルタリングルールよりも細かいフィルタリング機能です。詳細は[バイナリログイベントフィルター](/dm/dm-binlog-event-filter.md)を参照してください。

### バイナリログ位置

バイナリログ位置は、バイナリログファイル内のバイナリログイベントのオフセット情報を指します。詳細は[MySQL `SHOW BINLOG EVENTS`](https://dev.mysql.com/doc/refman/8.0/en/show-binlog-events.html)、[MariaDB `SHOW BINLOG EVENTS`](https://mariadb.com/kb/en/library/show-binlog-events/)を参照してください。

### バイナリログレプリケーション処理ユニット/同期ユニット

バイナリログレプリケーション処理ユニットは、DM-workerでアップストリームのバイナリログまたはローカルの中継ログを読み取り、これらのログをダウンストリームに移行するために使用される処理ユニットです。各サブタスクには1つのバイナリログレプリケーション処理ユニットが対応します。現在のドキュメントでは、バイナリログレプリケーション処理ユニットは同期処理ユニットとも呼ばれます。

### ブロックおよび許可テーブルリスト

ブロックおよび許可テーブルリストは、特定のデータベースやテーブルのすべての操作をフィルタリングしたり、移行したりする機能です。詳細は[ブロックおよび許可テーブルリスト](/dm/dm-block-allow-table-lists.md)を参照してください。この機能は、[MySQLレプリケーションフィルタリング](https://dev.mysql.com/doc/refman/8.0/en/replication-rules.html)や[MariaDBレプリケーションフィルタ](https://mariadb.com/kb/en/replication-filters/)に類似しています。

## C

### チェックポイント

チェックポイントは、フルデータインポートまたは増分レプリケーションタスクが一時停止および再開される位置、または停止および再起動される位置を示します。

- フルインポートタスクでは、チェックポイントは、インポート中のファイル内で正常にインポートされたデータのオフセットやその他の情報に対応します。チェックポイントはデータインポートタスクと同期して更新されます。
- 増分レプリケーションでは、チェックポイントは、[バイナリログ位置](#binlog-position)および成功裏に解析および移行された[バイナリログイベント](#binlog-event)のその他の情報に対応します。チェックポイントはDDL操作が正常に移行された後、または最終更新後30秒後に更新されます。

また、[中継処理ユニット](#relay-processing-unit)に対応する`relay.meta`情報は、チェックポイントと同様に機能します。中継処理ユニットは、アップストリームから[バイナリログイベント](#binlog-event)を取得し、このイベントを[中継ログ](#relay-log)に書き込み、このイベントに対応する[バイナリログ位置](#binlog-position)またはGTID情報を`relay.meta`に書き込みます。

## D

### ダンプ処理ユニット/ダンプユニット

ダンプ処理ユニットは、DM-workerでアップストリームからすべてのデータをエクスポートするために使用される処理ユニットです。各サブタスクには、1つのダンプ処理ユニットが対応します。

## G

### GTID

GTIDはMySQLまたはMariaDBのグローバルトランザクションIDです。この機能を有効にすると、GTID情報はバイナリログファイルに記録されます。複数のGTIDがGTIDセットを形成します。詳細は[MySQL GTID Format and Storage](https://dev.mysql.com/doc/refman/8.0/en/replication-gtids-concepts.html)、[MariaDB Global Transaction ID](https://mariadb.com/kb/en/library/gtid/)を参照してください。

## L

### ロード処理ユニット/ロードユニット

ロード処理ユニットは、DM-workerで完全にエクスポートされたデータをダウンストリームにインポートするために使用される処理ユニットです。各サブタスクには、1つのロード処理ユニットが対応します。現在のドキュメントでは、ロード処理ユニットはインポート処理ユニットとも呼ばれます。

## M

### 移行

TiDBデータ移行ツールを使用して、アップストリームデータベースの**フルデータ**をダウンストリームデータベースにコピーするプロセスです。

"フル"と明記されており、"フルまたは増分"が明示的に記載されておらず、"フル+増分"が明示的に記載されている場合、"レプリケーション"ではなく"移行"を使用してください。

## R

### 中継ログ

中継ログは、DM-workerがアップストリームのMySQLまたはMariaDBから取得し、ローカルディスクに保存するバイナリログファイルを指します。中継ログの形式は標準的なバイナリログファイルであり、互換性のあるバージョンの[mysqlbinlog](https://dev.mysql.com/doc/refman/8.0/en/mysqlbinlog.html)などのツールで解析できます。その役割は[MySQL中継ログ](https://dev.mysql.com/doc/refman/8.0/en/replica-logs-relaylog.html)および[MariaDB中継ログ](https://mariadb.com/kb/en/library/relay-log/)に類似しています。

TiDB DMの中継ログのディレクトリ構造、初期移行ルール、およびデータの削除などの詳細については、[TiDB DM中継ログ](/dm/relay-log.md)を参照してください。

### 中継処理ユニット

中継処理ユニットは、DM-workerでバイナリログファイルをアップストリームから取得し、中継ログにデータを書き込むために使用される処理ユニットです。各DM-workerインスタンスには、1つの中継処理ユニットしかありません。

### レプリケーション

TiDBデータ移行ツールを使用して、アップストリームデータベースの**増分データ**をダウンストリームデータベースにコピーするプロセスです。

明示的に"増分"が記載されている場合は、"移行"ではなく"レプリケーション"を使用してください。

## S

### セーフモード

セーフモードは、テーブルスキーマに主キーまたは一意のインデックスが存在する場合、DMLステートメントを複数回インポートできるモードです。このモードでは、アップストリームからの一部のステートメントは、再度書き直した後にダウンストリームに移行されます。`INSERT`ステートメントは`REPLACE`として書き直され、`UPDATE`ステートメントは`DELETE`と`REPLACE`として書き直されます。

このモードは以下のいずれかの状況で有効になります:

- タスク構成ファイルの`safe-mode`パラメータが`true`に設定されている場合、セーフモードは有効のままです。
- シャードマージシナリオでは、DDLステートメントがすべてのシャードテーブルでレプリケートされるまで、セーフモードは有効のままです。
- フルマイグレーションタスクのダンプ処理ユニットに`--consistency none`引数が設定されている場合、エクスポートの開始時のバイナリログの変更がエクスポートされたデータに影響を与えるかどうかが判断できません。そのため、これらのバイナリログの増分レプリケーションのためにセーフモードが有効のままです。
- タスクがエラーで一時停止され、その後再開された場合、一部のデータに対する操作が2回実行される可能性があります。

### シャードDDL

シャードDDLは、アップストリームのシャードテーブルで実行されるDDLステートメントです。このステートメントは、シャードテーブルのマージプロセスでTiDB DMによって調整および移行する必要があります。現在のドキュメントでは、シャードDDLはシャーディングDDLとも呼ばれます。

### シャードDDLロック

シャードDDLロックは、シャードDDLの移行を調整するロックメカニズムです。詳細については、[悲観的モードでのシャードテーブルのマージとデータの移行の実装原則](/dm/feature-shard-merge-pessimistic.md#principles)を参照してください。現在のドキュメントでは、シャードDDLロックはシャーディングDDLロックとも呼ばれます。

### シャードグループ

シャードグループは、すべてのアップストリームのシャードテーブルがダウンストリームの同じテーブルにマージおよび移行される必要があるグループを指します。TiDB DMの実装には、2段階のシャードグループが使用されます。詳細については、[悲観的モードでのシャードテーブルのマージとデータの移行の実装原則](/dm/feature-shard-merge-pessimistic.md#principles)を参照してください。現在のドキュメントでは、シャードグループはシャーディンググループとも呼ばれます。

### サブタスク

サブタスクは、各DM-workerインスタンスで実行されているデータ移行タスクの一部です。異なるタスク構成では、単一のデータ移行タスクには1つ以上のサブタスクがある場合があります。

### サブタスクステータス

サブタスクステータスは、データ移行サブタスクのステータスを示します。現在のステータスオプションには、「新規」「実行中」「一時停止」「停止」「終了」があります。データ移行タスクまたはサブタスクのステータスについての詳細については、[サブタスクステータス](/dm/dm-query-status.md#subtask-status)を参照してください。

## T

### テーブルルーティング
```md
The table routing feature enables DM to migrate a certain table of the upstream MySQL or MariaDB instance to the specified table in the downstream, which can be used to merge and migrate sharded tables. Refer to [table routing](/dm/dm-table-routing.md) for details.

### タスク

`start-task` コマンドを正常に実行した後に開始されるデータ移行タスク。異なるタスク構成では、単一の移行タスクは単一の DM-worker インスタンスまたは複数の DM-worker インスタンスで同時に実行できます。

### タスクのステータス

タスクのステータスは、データ移行タスクのステータスを指します。タスクのステータスは、すべてのサブタスクのステータスに依存します。詳細については、[サブタスクステータス](/dm/dm-query-status.md#subtask-status) を参照してください。
```