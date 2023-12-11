---
title: TiDBデータ移行FAQ
summary: TiDBデータ移行（DM）に関するよくある質問（FAQ）
aliases: ['/docs/tidb-data-migration/dev/faq/']
---

# TiDBデータ移行FAQ

このドキュメントでは、TiDBデータ移行（DM）に関するよくある質問（FAQ）をまとめています。

## DMはAlibaba RDSまたは他のクラウドデータベースからのデータ移行をサポートしていますか？

現時点では、DMは標準版のMySQLまたはMariaDBのバイナリログのデコードのみをサポートしています。Alibaba Cloud RDSや他のクラウドデータベースについてはテストされていません。バイナリログが標準形式であることが確認されていれば、サポートされます。

Alibaba Cloud RDSでは、プライマリキーのない上流テーブルのバイナリログには隠しプライマリキーカラムが含まれており、元のテーブル構造と一貫性がありません。

次に、既知の互換性のない問題があります:

- **Alibaba Cloud RDS**では、プライマリキーのない上流テーブルのバイナリログには隠しプライマリキーカラムが含まれており、元のテーブル構造と一貫性がありません。
- **HUAWEI Cloud RDS**では、バイナリログファイルの直接読み取りはサポートされていません。詳細については[Can HUAWEI Cloud RDS Directly Read Binlog Backup Files?](https://support.huaweicloud.com/en-us/rds_faq/rds_faq_0210.html)を参照してください。

## タスク構成のブロックおよび許可リストの正規表現は、`non-capturing (?!)`をサポートしていますか？

現時点では、DMはサポートしておらず、Golang標準ライブラリの正規表現のみをサポートしています。Golangがサポートする正規表現については、[re2-syntax](https://github.com/google/re2/wiki/Syntax)を参照してください。

## 上流で実行されるステートメントに複数のDDL操作が含まれる場合、DMはそのような移行をサポートしていますか？

DMは、複数のDDL変更操作を含む単一のステートメントを複数の単一のDDL操作を含むステートメントに分割しようと試みますが、すべてのケースをカバーできない場合があります。上流で実行されるステートメントには1つのDDL操作のみを含めるか、テスト環境で検証することを推奨します。サポートされていない場合は、DMリポジトリに[issue](https://github.com/pingcap/dm/issues)を報告してください。

## 互換性のないDDLステートメントを処理する方法は？

TiDBでサポートされていないDDLステートメントに遭遇した場合、dmctlを使用してそのDDLステートメントを手動で処理する必要があります（DDLステートメントをスキップするか、指定されたDDLステートメントでDDLステートメントを置換する）。詳細については、[失敗したDDLステートメントの処理](/dm/handle-failed-ddl-statements.md)を参照してください。

> **注意:**
>
> 現時点では、TiDBはMySQLがサポートするすべてのDDLステートメントと互換性があるわけではありません。[MySQL互換性](/mysql-compatibility.md#ddl-operations)を参照してください。

## DMはビュー関連のDDLステートメントやDMLステートメントをTiDBにレプリケートしますか？

現時点では、DMはビュー関連のDDLステートメントをダウンストリームのTiDBクラスターにはレプリケートしませんし、ビュー関連のDMLステートメントをダウンストリームのTiDBクラスターにもレプリケートしません。

## データ移行タスクをリセットする方法は？

データ移行中に例外が発生し、データ移行タスクが再開できない場合は、タスクをリセットしてデータを再移行する必要があります:

1. `stop-task`コマンドを実行して異常なデータ移行タスクを停止します。

2. ダウンストリームに移行されたデータを削除します。

3. 次のいずれかの方法でデータ移行タスクを再開します。

   - タスク構成ファイルで新しいタスク名を指定します。その後、`start-task {task-config-file}`を実行します。
   - `start-task --remove-meta {task-config-file}`を実行します。

## `online-ddl: true`が設定されているgh-ostテーブル関連のDDL操作に関連するエラーの処理方法は？

上記のエラーは次の理由で発生する場合があります:

最後の`rename ghost_table to origin table`ステップで、DMはメモリ内のDDL情報を読み取り、それを元のテーブルのDDLに復元します。

ただし、メモリ内のDDL情報は以下のいずれかの方法で取得されます:

- DM [は`alter ghost_table`操作中にgh-ostテーブルを処理](/dm/feature-online-ddl.md#online-schema-change-gh-ost)し、`ghost_table`のDDL情報を記録します。
- DM-workerを再起動してタスクを開始すると、DMは`dm_meta.{task_name}_onlineddl`からDDLを読み取ります。

したがって、増分レプリケーションのプロセスで、指定されたPosが`alter ghost_table`のDDLをスキップしても、Posがまだgh-ostのonline-ddlプロセスにある場合、ghost_tableがメモリに書き込まれるか、`dm_meta.{task_name}_onlineddl`に正しく書き込まれません。そのような場合には、上記のエラーが返されます。

このエラーを回避するためには、次の手順を実行できます:

1. タスクの`online-ddl-scheme`または`online-ddl`構成を削除します。

2. `block-allow-list.ignore-tables`で`_{table_name}_gho`、`_{table_name}_ghc`、`_{table_name}_del`を設定します。

3. ダウンストリームのTiDBで上流のDDLを手動で実行します。

4. gh-ostプロセスの後のポジションにレプリケートされたら、`online-ddl-scheme`または`online-ddl`構成を再有効にし、`block-allow-list.ignore-tables`のコメントを外します。

## 既存のデータ移行タスクにテーブルを追加する方法は？

実行中のデータ移行タスクにテーブルを追加する必要がある場合、次の方法でそれに対処できます。

> **注意:**
>
> 既存のデータ移行タスクにテーブルを追加する操作は複雑ですので、必要な場合にのみ行うことをお勧めします。

### `Dump`ステージで

MySQLはエクスポートにスナップショットを指定できないため、エクスポート中にデータ移行タスクの更新をサポートせず、その後再開することもできません。そのため、`Dump`ステージで移行する必要があるテーブルを動的に追加することはできません。

本当にテーブルを追加する必要がある場合は、新しい構成ファイルを使用してタスクを直接再起動することをお勧めします。

### `Load`ステージで

エクスポート中に、複数のデータ移行タスクは通常、異なるバイナリログポジションを持っています。そのため、`Load`ステージでタスクをマージすると、バイナリログポジションで合意に達することができないかもしれません。そのため、`Load`ステージでデータ移行タスクにテーブルを追加することはお勧めしません。

### `Sync`ステージで

データ移行タスクが`Sync`ステージにある場合、構成ファイルに追加のテーブルを設定してタスクを再起動すると、DMは新たに追加されたテーブルに対してフルエクスポートおよびインポートを再実行しません。その代わりに、DMは以前のチェックポイントからインクリメンタルなレプリケーションを継続します。

したがって、新たに追加されたテーブルのフルデータがダウンストリームにインポートされていない場合は、新しいデータ移行タスクを使用してダウンストリームにフルデータをエクスポートおよびインポートする必要があります。

既存の移行タスクに対応するグローバルチェックポイント（`is_global=1`）のポジション情報を`checkpoint-T`として記録します（例: `(mysql-bin.000100, 1234)`）。追加するテーブルのフルエクスポートの`metedata`（または`Sync`ステージの別のデータ移行タスクのチェックポイント）のポジション情報を`checkpoint-S`として記録します（例: `(mysql-bin.000099, 5678)`）。次の手順で、移行タスクにテーブルを追加できます:

1. 既存の移行タスクを`stop-task`を使用して停止します。追加するテーブルが別の実行中の移行タスクに属する場合は、そのタスクも停止します。

2. MySQLクライアントを使用して、既存の移行タスクに対応するチェックポイントテーブルの情報を手動で更新し、`checkpoint-T`と`checkpoint-S`の小さい値にします。この例では、（`mysql-bin.000099, 5678`）です。

    - 更新するチェックポイントテーブルは、`{dm_meta}`スキーマ内の`{task-name}_syncer_checkpoint`です。

    - 更新するチェックポイントの行は`id=(source-id)`および`is_global=1`に一致します。

    - 更新するチェックポイントの列は`binlog_name`および`binlog_pos`です。

3. タスクで`syncers`に`safe-mode: true`を設定して再入可能な実行を確保します。

4. `start-task`を使用してタスクを再開します。

5. `query-status`を介してタスクの状態を観察します。`syncerBinlog`が`checkpoint-T`と`checkpoint-S`の大きい値を超えたときに、`safe-mode`を元の値に復元してタスクを再開します。この例では、（`mysql-bin.000100, 1234`）です。

## フルインポート中に発生する`packet for query is too large. Try adjusting the 'max_allowed_packet' variable`エラーの処理方法は？

以下のパラメータをデフォルトの67108864（64M）よりも大きな値に設定します。

- TiDBサーバーのグローバル変数: `max_allowed_packet`。
- タスク構成ファイルの設定項目: `target-database.max-allowed-packet`。詳細については、[DM Advanced Task Configuration File](/dm/task-configuration-file-full.md)を参照してください。
## [既存の DM 1.0 クラスタの DM マイグレーションタスクが DM 2.0 以降のクラスタで実行中に発生する `Error 1054: Unknown column 'binlog_gtid' in 'field list'` エラーの対処方法](/dm/manually-upgrade-dm-1.0-to-2.0.md)

DM v2.0 以降では、既存の DM 1.0 クラスタのタスク構成ファイルを使用して `start-task` コマンドを直接実行しても、増分データレプリケーションを続行しようとすると `Error 1054: Unknown column 'binlog_gtid' in 'field list'` エラーが発生します。

このエラーは、[既存の DM 1.0 クラスタの DM マイグレーションタスクを DM 2.0 クラスタに手動でインポートすることで処理できます](/dm/manually-upgrade-dm-1.0-to-2.0.md)。

## TiUP が一部の DM バージョン（たとえば、v2.0.0-hotfix）のデプロイに失敗する理由

`tiup list dm-master` コマンドを使用して、TiUP がデプロイをサポートする DM バージョンを表示できます。このコマンドに表示されない DM バージョンは、TiUP が管理しません。

## DM がデータをレプリケーションしているときに発生する `parse mydumper metadata error: EOF` エラーの対処方法

このエラーをさらに分析するには、エラーメッセージとログファイルを確認する必要があります。原因は、ダンプユニットが適切なメタデータファイルを生成できない可能性があるため、権限が不足していることが考えられます。

## DM はシャーディングされたスキーマとテーブルをレプリケートしているときに致命的なエラーを報告しないが、下流のデータが失われている事例の原因

`block-allow-list` および `table-route` の構成項目を確認してください:

- `block-allow-list` の下流データベースおよびテーブルの名前を構成する必要があります。`do-tables` の前に「~」を追加して、正規表現を使用して名前を一致させることができます。
- `table-route` は、正規表現の代わりにワイルドカード文字を使用してテーブル名を一致させます。たとえば、`table_parttern_[0-63]` は `table_parttern_0` から `table_pattern_6` までの 7 つのテーブルにだけ一致します。

## 上流からのレプリケーションが行われていないときに、DM が `replicate lag` モニターメトリクスにデータが表示されない理由

DM 1.0 では、モニターデータを生成するために `enable-heartbeat` を有効にする必要があります。DM 2.0 以降のバージョンでは、モニターメトリクス `replicate lag` にデータが表示されないのは、この機能がサポートされていないためです。

## DM がタスクを開始する際に `fail to initial unit Sync of subtask` エラーが発生し、エラーメッセージの中に `context deadline exceeded` が表示される場合の対処方法

これは、DM 2.0.0 の既知の問題であり、DM 2.0.1 で修正されます。この問題は、レプリケーションタスクに対処するテーブルが多数ある場合に引き起こされる可能性があります。この問題を解消するには、TiUP を使用して DM をアップグレードするか、GitHub の [DM のリリースページ](https://github.com/pingcap/tiflow/releases) から 2.0.0-hotfix バージョンをダウンロードし、実行可能ファイルを手動で置き換えることができます。

## DM がデータをレプリケートする際に `duplicate entry` エラーが発生した場合の対処方法

まず、次の事項を確認してください:

- レプリケーションタスクで `disable-detect` が構成されていないか（v2.0.7 およびそれ以前のバージョン）
- データが手動で挿入されたり、他のレプリケーションプログラムによって挿入されたりしていないか
- このテーブルに関連する DML フィルタが構成されていないか

トラブルシューティングを容易にするために、まず下流の TiDB インスタンスの一般ログファイルを収集し、次に [TiDB コミュニティの Slack チャンネル](https://tidbcommunity.slack.com/archives/CH7TTLL7P) で技術サポートを求めることができます。次の例では、一般ログファイルの収集方法を示しています:

```bash
# 一般ログの収集を有効にする
curl -X POST -d "tidb_general_log=1" http://{TiDBIP}:10080/settings
# 一般ログの収集を無効にする
curl -X POST -d "tidb_general_log=0" http://{TiDBIP}:10080/settings
```

`duplicate entry` エラーが発生した場合は、競合するデータを含むレコードを確認するためにログファイルを確認する必要があります。

## なぜ一部のモニタリングパネルに `No data point` が表示されるのか？

一部のパネルにデータが表示されないのは正常です。たとえばエラーが報告されず、DDL ロックがない、またはリレーログ機能が有効にされていない場合は、対応するパネルに `No data point` が表示されます。各パネルの詳細な説明については、[DM モニタリングメトリクス](/dm/monitor-a-dm-cluster.md) を参照してください。

## DM v1.0 において、タスクがエラー状態にある場合に `sql-skip` コマンドが一部のステートメントをスキップできない理由

まず、`sql-skip` を実行した後もビンロッグの位置が進行しているかどうかを確認する必要があります。ビンロッグの位置が進行している場合、`sql-skip` が有効になっていることを意味します。このエラーが継続する理由は、上流から複数の未サポートの DDL ステートメントが送信されるためです。これらのステートメントに一致するパターンを設定するために、`sql-skip -s <sql-pattern>` を使用できます。

エラーメッセージには、`parse statement` 情報が含まれる場合があります。たとえば:

```
if the DDL is not needed, you can use a filter ... lines...
```

このタイプのエラーが発生する理由は、上流から送信された DDL ステートメント（`ALTER EVENT` など）を TiDB パーサが解析できないためであり、`sql-skip` が期待どおりに機能しないことです。このステートメントをフィルタリングし、`schema-pattern: "*"` を設定するために、[ビンロッグ イベント フィルタ](/dm/dm-binlog-event-filter.md) を構成ファイルに追加することができます。DM v2.0.1 以降では、DM は `EVENT` に関連するステートメントを事前にフィルタリングします。

DM v6.0 以降、`binlog` が `sql-skip` および `handle-error` を置き換えました。この問題を回避するために `binlog` コマンドを使用できます。

## DM がデータをレプリケートする際に `REPLACE` ステートメントが下流に繰り返し表示される理由

タスクの実行後 1 分以内に安全なモードが有効になります。タスクがエラー後に自動的に再開されるか、高可用性スケジューリングが行われると、1 分以内に安全なモードが有効になります。

DM-worker ログファイルを確認し、`change count` を含む行を検索できます。行中の `new count` がゼロでない場合、安全なモードが有効になっています。それがなぜ有効になっているかを確認するには、発生したタイミングと、それ以前にどのようなエラーが報告されているかを確認してください。

## DM v2.0 において、フル インポート タスクがタスクの実行中に DM が再起動すると失敗する理由

DM v2.0.1 およびそれ以前のバージョンでは、フル インポートが完了する前に DM が再起動した場合、上流データソースと DM-worker ノード間のバインディングが変更される可能性があります。たとえば、ダンプユニットの中間データが DM-worker ノード A にあるが、ロードユニットが DM-worker ノード B で実行されている可能性があります。そのため、操作が失敗する可能性があります。

この問題の解決策は次の 2 つです:

- データ容量が小さい場合（1 TB 未満）またはタスクがシャーディングされたテーブルをマージする場合は、次の手順を実行してください:

    1. 下流のデータベースでインポートされたデータをクリーンアップします
    2. エクスポートされたデータのディレクトリ内のすべてのファイルを削除します
    3. dmctl を使用してタスクを削除し、`start-task --remove-meta` コマンドを実行して新しいタスクを作成します

    新しいタスクが開始されたら、不要な DM ワーカーノードがないことを確認することをお勧めします。また、フル インポート中に DM クラスタを再起動またはアップグレードしないでください。

- データ容量が大きい場合（1 TB を超える）は、次の手順を実行してください:

    1. 下流のデータベースでインポートされたデータをクリーンアップします
    2. DM worker ノードに TiDB-Lightning をデプロイします
    3. TiDB-Lightning の Local-backend モードを使用して、DM ダンプユニットがエクスポートしたデータをインポートします
    4. フル インポートが完了したら、タスク構成ファイルを次のように編集してタスクを再起動します:
        - `task-mode` を `incremental` に変更します
        - `mysql-instance.meta.pos` の値を、ダンプユニットが出力したメタデータファイルに記録された位置に設定します

## 増分タスク中に DM が再起動した場合に、インクリメンタル タスクでメタデータファイルに記録された上流のビンログ位置がパージされているときに、DM が `ERROR 1236 (HY000): The slave is connecting using CHANGE MASTER TO MASTER_AUTO_POSITION = 1, but the master has purged binary logs containing GTIDs that the slave requires.` エラーを報告する理由

このエラーは、ダンプユニットによって出力されたメタデータファイルに記録された上流のビンログ位置がフルマイグレーション中にパージされたことを示しています。

この問題が発生した場合は、タスクを一時停止し、下流のデータベースで移行されたすべてのデータを削除し、`--remove-meta` オプションを使用して新しいタスクを開始する必要があります。

この問題を事前に回避するために、次のように構成できます:

1. 上流の MySQL データベースで `expire_logs_days` の値を増やして、フルマイグレーションタスクが完了する前に必要なビンログファイルが誤ってパージされるのを防ぎます。データ容量が大きい場合、ダンプリングと TiDB-Lightning を同時に使用してタスクを高速化することをお勧めします。
2. このタスクでリレーログ機能を有効にして、DMがバイナリログのポジションがパージされていてもリレーログからデータを読み取れるようにします。

## DMクラスタがTiUP v1.3.0またはv1.3.1を使用して展開されている場合、なぜDMクラスタのGrafanaダッシュボードに `failed to fetch dashboard` と表示されるのか？

これはTiUPの既知のバグであり、TiUP v1.3.2で修正されています。この問題への2つの解決策は以下の通りです。

- 解決策1：
   1. コマンド `tiup update --self && tiup update dm` を使用してTiUPを後のバージョンにアップグレードします。
   2. クラスタ内のGrafanaノードをスケールインしてからスケールアウトし、Grafanaサービスを再起動します。

- 解決策2：
   1. `deploy/grafana-$port/bin/public` フォルダをバックアップします。
   2. [TiUP DMオフラインパッケージ](https://download.pingcap.org/tidb-dm-v2.0.1-linux-amd64.tar.gz)をダウンロードして展開します。
   3. オフラインパッケージ内の `grafana-v4.0.3-**.tar.gz` を展開します。
   4. `deploy/grafana-$port/bin/public` フォルダを `grafana-v4.0.3-**.tar.gz` 内の `public` フォルダで置き換えます。
   5. `tiup dm restart $cluster_name -R grafana` を実行してGrafanaサービスを再起動します。

## DM v2.0では、タスクに `enable-relay` と `enable-gtid` が同時に有効になっている場合に、コマンド `query-status` のクエリ結果が、SyncerチェックポイントのGTIDsが不連続で表示されるのはなぜですか？

これはDMの既知のバグであり、DM v2.0.2で修正されています。次の2つの条件が同時に完全に満たされると、このバグが発生します。

1. ソース構成ファイルでパラメータ `enable-relay` と `enable-gtid` が `true` に設定されている。
2. 上流のデータベースが **MySQLセカンダリデータベース** である。データベースの `previous_gtids` をクエリするために `show binlog events in '<newest-binlog>' limit 2` コマンドを実行した場合、以下の例のように不連続な結果となると、バグが発生します。

```
mysql> show binlog events in 'mysql-bin.000005' limit 2;
+------------------+------+----------------+-----------+-------------+--------------------------------------------------------------------+
| Log_name         | Pos  | Event_type     | Server_id | End_log_pos | Info                                                               |
+------------------+------+----------------+-----------+-------------+--------------------------------------------------------------------+
| mysql-bin.000005 |    4 | Format_desc    |    123452 |         123 | Server ver: 5.7.32-35-log, Binlog ver: 4                           |
| mysql-bin.000005 |  123 | Previous_gtids |    123452 |         194 | d3618e68-6052-11eb-a68b-0242ac110002:6-7                           |
+------------------+------+----------------+-----------+-------------+--------------------------------------------------------------------+
```

バグは、`dmctl` で `query-status <task>` を実行してタスク情報をクエリし、`subTaskStatus.sync.syncerBinlogGtid` が不連続であるが `subTaskStatus.sync.masterBinlogGtid` が連続していることが分かる場合に発生します。以下の例を参照してください。

```
query-status test
{
    ...
    "sources": [
        {
            ...
            "sourceStatus": {
                "source": "mysql1",
                ...
                "relayStatus": {
                    "masterBinlog": "(mysql-bin.000006, 744)",
                    "masterBinlogGtid": "f8004e25-6067-11eb-9fa3-0242ac110003:1-50",
                    ...
                }
            },
            "subTaskStatus": [
                {
                    ...
                    "sync": {
                        ...
                        "masterBinlog": "(mysql-bin.000006, 744)",
                        "masterBinlogGtid": "f8004e25-6067-11eb-9fa3-0242ac110003:1-50",
                        "syncerBinlog": "(mysql-bin|000001.000006, 738)",
                        "syncerBinlogGtid": "f8004e25-6067-11eb-9fa3-0242ac110003:1-20:40-49",
                        ...
                        "synced": false,
                        "binlogType": "local"
                    }
                }
            ]
        },
        {
            ...
            "sourceStatus": {
                "source": "mysql2",
                ...
                "relayStatus": {
                    "masterBinlog": "(mysql-bin.000007, 1979)",
                    "masterBinlogGtid": "ddb8974e-6064-11eb-8357-0242ac110002:1-25",
                    ...
                }
            },
            "subTaskStatus": [
                {
                    ...
                    "sync": {
                        "masterBinlog": "(mysql-bin.000007, 1979)",
                        "masterBinlogGtid": "ddb8974e-6064-11eb-8357-0242ac110002:1-25",
                        "syncerBinlog": "(mysql-bin|000001.000008, 1979)",
                        "syncerBinlogGtid": "ddb8974e-6064-11eb-8357-0242ac110002:1-25",
                        ...
                        "synced": true,
                        "binlogType": "local"
                    }
                }
            ]
        }
    ]
}
```

上記の例では、データソース `mysql1` の `syncerBinlogGtid` が不連続であります。この場合、次のいずれかの方法でデータ損失を処理できます。

- 現在の時点からフルエクスポートタスクのメタデータに記録された位置までの上流のバイナリログが削除されていない場合、次の手順を実行できます：
   1. 現在のタスクを停止し、不連続なGTIDを持つすべてのデータソースを削除します。
   2. すべてのソース構成ファイルで `enable-relay` を `false` に設定します。
   3. 不連続なGTIDを持つデータソース（上記の例での `mysql1` など）を増分タスクに変更し、`mysql-instances.meta` をそれぞれのフルエクスポートタスクのメタデータ情報（`binlog-name`、`binlog-pos`、`binlog-gtid` 情報を含む）で構成します。
   4. `task.yaml` の `syncers.safe-mode` を `true` に設定し、タスクを再起動します。
   5. 増分タスクがすべての不足データをダウンストリームにレプリケートした後、タスクを停止し、`task.yaml` の `safe-mode` を `false` に変更します。
   6. 再度タスクを再起動します。

- 上流のバイナリログが削除されているが、ローカルのリレーログが残っている場合、次の手順を実行できます：
   1. 現在のタスクを停止します。
   2. 不連続なGTIDを持つデータソース（上記の例での `mysql1` など）を増分タスクに変更し、各フルエクスポートタスクのメタデータ情報（`binlog-name`、`binlog-pos`、`binlog-gtid` 情報を含む）を `mysql-instances.meta` で構成します。
   3. 増分タスクの `task.yaml` で `binlog-gtid` の前の値を `previous_gtids` の前の値に変更します。上記の例では、`1-y` を `6-y` に変更します。
   4. 増分タスクの `task.yaml` で `syncers.safe-mode` を `true` に設定し、タスクを再起動します。
   5. 増分タスクがすべての不足データをダウンストリームにレプリケートした後、タスクを停止し、`task.yaml` の `safe-mode` を `false` に変更します。
   6. 再度タスクを再起動します。
   7. データソースを再起動し、ソース構成ファイルで `enable-relay` または `enable-gtid` を `false` に設定します。

- 上記の条件のいずれにも該当しない場合、またはタスクのデータ量が少ない場合、次の手順を実行できます：
   1. ダウンストリームデータベースにインポートされたデータをクリアアップします。
   2. データソースを再起動し、ソース構成ファイルで `enable-relay` または `enable-gtid` を `false` に設定します。
   3. 新しいタスクを作成し、コマンド `start-task task.yaml --remove-meta` を実行して、最初からデータを移行します。

上記の第1および第2の解決策の場合、最初に表示されたとおりに通常レプリケート可能なデータソース（上記の例での `mysql2` など）について、増分タスクを設定する際に関連する `mysql-instances.meta` を `subTaskStatus.sync` からの `syncerBinlog` と `syncerBinlogGtid` の情報で構成してください。

## DM v2.0で、仮想IP環境でのDM-workerとMySQLインスタンスの接続を切り替える際に `heartbeat` 機能が有効化されている状態で `heartbeat config is different from previous used: serverID not equal` エラーを処理する方法は？

`heartbeat` 機能は、DM v2.0およびそれ以降のバージョンではデフォルトで無効になっています。タスク構成ファイルで機能を有効にした場合、高可用性機能に干渉します。この問題を解決するには、タスク構成ファイルで `enable-heartbeat` を `false` に設定し、その後、タスク構成ファイルを再読み込みします。DMは、今後のリリースで強制的に `heartbeat` 機能を無効にします。
## DMマスターノードが再起動後にクラスターに参加できず、「fail to start embed etcd, RawCause: member xxx has already been bootstrapped」というエラーがDMによって報告される理由は何ですか？

DMマスターが起動すると、DMは現在のディレクトリにetcd情報を記録します。DMマスターが再起動後、ディレクトリが変更されると、DMはetcd情報にアクセスできなくなり、それによって再起動が失敗します。

この問題を解決するには、TiUPを使用してDMクラスターを維持することをお勧めします。バイナリファイルを使用してデプロイする必要がある場合は、DMマスターの設定ファイルで`data-dir`を絶対パスで構成するか、コマンドを実行する現在のディレクトリに注意を払う必要があります。

## dmctlを使用してコマンドを実行した場合に、なぜDMマスターに接続できないのですか？

dmctlを使用してコマンドを実行する際、`--master-addr`のパラメータ値を指定してもDMマスターへの接続が失敗することがあります。「RawCause: context deadline exceeded, Workaround: please check your network connection.」のようなエラーメッセージが表示されますが、`telnet <master-addr>`のようなコマンドを使用してネットワーク接続を確認しても例外は見つかりません。

この場合、環境変数`https_proxy`（**https**であることに注意）を確認できます。この変数が設定されている場合、dmctlは`https_proxy`で指定されたホストとポートに自動的に接続します。ホストに対応する`proxy`転送サービスがない場合、接続に失敗します。

この問題を解決するには、`https_proxy`が必須かどうかを確認します。必要ない場合は設定をキャンセルします。それ以外の場合は、元のdmctlコマンドの前に環境変数設定`https_proxy="" ./dmctl --master-addr "x.x.x.x:8261"`を追加します。

> **注意:**
>
> `proxy`に関連する環境変数には`http_proxy`、`https_proxy`、`no_proxy`が含まれます。上記の手順を実行しても接続エラーが解消しない場合は、`http_proxy`と`no_proxy`の設定パラメータが正しいかどうかを確認してください。

## DMのバージョン2.0.2から2.0.6までのstart-relayコマンドを実行したときに返されるエラーを処理する方法は？

```
flush local meta, Rawcause: open relay-dir/xxx.000001/relay.metayyyy: no such file or directory
```

上記のエラーは、次の場合に発生する可能性があります。

- DMがv2.0.1以前からv2.0.2 - v2.0.6にアップグレードされ、リレーログがアップグレード前に開始され、アップグレード後に再起動された場合。
- stop-relayコマンドを実行してリレーログを一時停止し、それを再開する場合。

このエラーを回避するには、次のオプションがあります。

- リレーログを再起動します：

    ```
    » stop-relay -s sourceID workerName
    » start-relay -s sourceID workerName
    ```

- DMをv2.0.7またはそれ以降のバージョンにアップグレードします。

## ロードユニットが `Unknown character set` エラーを報告する理由は何ですか？

TiDBはすべてのMySQLの文字セットをサポートしていません。そのため、完全なインポート中にサポートされていない文字セットが使用されると、DMはこのエラーを報告します。このエラーをバイパスするためには、TiDBがサポートする[文字セット](/character-set-and-collation.md)を使用して下流で事前にテーブルスキーマを作成することができます。