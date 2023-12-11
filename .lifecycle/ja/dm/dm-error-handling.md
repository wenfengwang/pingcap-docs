---
title: TiDBデータマイグレーションでのエラー処理
summary: TiDBデータマイグレーション（DM）を使用する際に一般的なエラーの処理方法について学びます。
aliases: ['/docs/tidb-data-migration/dev/error-handling/','/docs/tidb-data-migration/dev/troubleshoot-dm/','/docs/tidb-data-migration/dev/error-system/']
---

# TiDBデータマイグレーションでのエラー処理

このドキュメントでは、TiDBデータマイグレーション（DM）を使用する際に一般的なエラーとその処理方法について紹介します。

## エラーシステム

エラーシステムでは、通常、特定のエラーの情報は以下のとおりです。

- `code`: エラーコード。

    DMは同じエラータイプに対して同じエラーコードを使用します。エラーコードはDMバージョンが変更されても変わりません。

    一部のエラーはDMの反復処理中に削除される場合がありますが、エラーコードは変更されません。新しいエラーに対しては、既存のエラーコードではなく新しいエラーコードをDMが使用します。

- `class`: エラータイプ。

    エラーが発生したコンポーネント（エラーの発生源）をマークするために使用されます。

    以下の表には、すべてのエラータイプ、エラーの発生源、およびエラーサンプルが表示されています。

    |  <div style="width: 100px;">エラータイプ</div>    |   エラーの発生源            | エラーサンプル                                                     |
    | :-------------- | :------------------------------ | :------------------------------------------------------------ |
    | `database`       |  データベース操作         | `[code=10003:class=database:scope=downstream:level=medium] database driver: invalid connection` |
    | `functional`     |  DMの基本機能の基礎                   | `[code=11005:class=functional:scope=internal:level=high] not allowed operation: alter multiple tables in one statement` |
    | `config`         |  不正な構成                      | `[code=20005:class=config:scope=internal:level=medium] empty source-id not valid` |
    | `binlog-op`      |  バイナリログ操作          | `[code=22001:class=binlog-op:scope=internal:level=high] empty UUIDs not valid` |
    | `checkpoint`     |  チェックポイント操作  | `[code=24002:class=checkpoint:scope=internal:level=high] save point bin.1234 is older than current pos bin.1371` |
    | `task-check`     |  タスクチェック       | `[code=26003:class=task-check:scope=internal:level=medium] new table router error` |
    | `relay-event-lib`|  リレーモジュールの基本機能の実行 | `[code=28001:class=relay-event-lib:scope=internal:level=high] parse server-uuid.index` |
    | `relay-unit`     |  リレー処理ユニット    | `[code=30015:class=relay-unit:scope=upstream:level=high] TCPReader get event: ERROR 1236 (HY000): Could not open log file` |
    | `dump-unit`      |   ダンプ処理ユニット    | `[code=32001:class=dump-unit:scope=internal:level=high] mydumper runs with error: CRITICAL **: 15:12:17.559: Error connecting to database: Access denied for user 'root'@'172.17.0.1' (using password: NO)` |
    | `load-unit`      |  ロード処理ユニット    | `[code=34002:class=load-unit:scope=internal:level=high] corresponding ending of sql: ')' not found` |
    | `sync-unit`      |  シンク処理ユニット     | `[code=36027:class=sync-unit:scope=internal:level=high] Column count doesn't match value count: 9 (columns) vs 10 (values)` |
    | `dm-master`      |   DMマスターサービス | `[code=38008:class=dm-master:scope=internal:level=high] grpc request error: rpc error: code = Unavailable desc = all SubConns are in TransientFailure, latest connection error: connection error: desc = "transport: Error while dialing dial tcp 172.17.0.2:8262: connect: connection refused"` |
    | `dm-worker`      |  DMワーカーサービス  | `[code=40066:class=dm-worker:scope=internal:level=high] ExecuteDDL timeout, try use query-status to query whether the DDL is still blocking` |
    | `dm-tracer`      |  DMトレーサーサービス  | `[code=42004:class=dm-tracer:scope=internal:level=medium] trace event test.1 not found` |
    | `schema-tracker` | シーマトラッカー（増分データレプリケーション中）   | `[code=44006:class=schema-tracker:scope=internal:level=high],"cannot track DDL: ALTER TABLE test DROP COLUMN col1"` |
    | `scheduler`      | スケジューリング操作（データマイグレーションタスクの）   | `[code=46001:class=scheduler:scope=internal:level=high],"the scheduler has not started"` |
    | `dmctl`          | dmctl内でエラーが発生した場合、または他のコンポーネントとの相互作用中にエラーが発生した場合 | `[code=48001:class=dmctl:scope=internal:level=high],"can not create grpc connection"` |

- `scope`: エラーのスコープ。

    エラーが発生したときにDMオブジェクトのスコープとソースをマークするために使用されます。`scope`には `not-set`, `upstream`, `downstream`, `internal` の4つのタイプが含まれています。

    エラーロジックが直接に上流と下流データベース間のリクエストに関連している場合、スコープは `upstream` または `downstream` に設定されます。そうでない場合は、現在は  `internal` に設定されています。

- `level`: エラーレベル。

    エラーの重要度レベルには、`low`, `medium`, `high` が含まれます。

    - `low` レベルのエラーは通常、ユーザ操作と不正な入力に関連しています。データマイグレーションタスクに影響を与えません。
    - `medium` レベルのエラーは通常、ユーザの構成に関連しています。いくつかの新しく開始されたサービスに影響を与えますが、既存のDMデータマイグレーションの状態には影響しません。
    - `high` レベルのエラーは通常、可能性があるデータマイグレーションタスクの中断を避けるために、注意が必要です。

- `message`: エラーの説明。

    エラーの詳細な説明。エラーコールチェーンの各追加層をラップして格納するために [errors.Wrap](https://godoc.org/github.com/pkg/errors#hdr-Adding_context_to_an_error) モードが採用されています。最も外側の層でラップされたメッセージ説明はエラーをDM内で示し、最も内側の層でラップされたメッセージ説明はエラーの発生源を示します。

- `workaround`: エラー処理方法（オプション）

    このエラーの処理方法。一部の確認されたエラー（構成エラーなど）に対して、DMは `workaround` で対応する手動処理方法を提供しています。

- エラースタック情報（オプション）

    DMがエラースタック情報を出力するかどうかは、エラーの深刻度と必要性に依存します。エラースタックは、エラーが発生したときの完全なスタックコール情報を記録します。基本情報とエラーメッセージではエラー原因を特定できない場合、エラースタックを使用してコードの実行パスをトレースすることができます。

エラーコードの完全なリストについては、[エラーコードリスト](https://github.com/pingcap/dm/blob/master/_utils/terror_gen/errors_release.txt) を参照してください。

## トラブルシューティング

DMを実行中にエラーが発生した場合は、次の手順に従ってトラブルシューティングを行ってください。

1. `query-status` コマンドを実行して、タスクの実行状態とエラー出力を確認します。

2. エラーに関連するログファイルを確認します。ログファイルはDMマスターノードとDMワーカーノードにあります。エラーに関する重要な情報を取得するには、[error system](#error-system) を参照し、次に [Handle Common Errors](#handle-common-errors) セクションを確認して解決策を見つけます。

3. このドキュメントでカバーされていないエラーが発生し、ログや監視メトリクスをチェックして問題を解決できない場合は、PingCAPやコミュニティから [サポートを取得](/support.md) してください。

4. エラーが解決した場合は、dmctlを使用してタスクを再開してください。

    {{< copyable "shell-regular" >}}

    ```bash
    resume-task ${task name}
    ```

しかしながら、いくつかのケースではデータマイグレーションタスクをリセットする必要があります。詳細については、[データマイグレーションタスクのリセット](/dm/dm-faq.md#how-to-reset-the-data-migration-task) を参照してください。

## 一般的なエラーの処理

| <div style="width: 100px;">エラーコード</div>       | エラーの説明                                                     |  処理方法                                                    |
| :----------- | :------------------------------------------------------------ | :----------------------------------------------------------- |
| `code=10001` |  異常なデータベース操作。                                              |  エラーメッセージとエラースタックをさらに分析する。                                |
| `code=10002` | 基礎となるデータベースからの `bad connection` エラー。通常、これはDMと下流のTiDBインスタンスの間の接続が異常（ネットワーク障害またはTiDBの再起動による可能性があり）であり、現在のリクエストされたデータがTiDBに送信されていないことを示します。 |  DMはこのようなエラーに対して自動リカバリを提供します。長時間のリカバリが成功しない場合は、ネットワークやTiDBの状態を確認してください。 |
| `code=10003` | 下位のデータベースからの`無効な接続`エラー。通常は、DMとダウンストリームのTiDBインスタンス間の接続が異常（ネットワーク障害またはTiDB再起動の可能性あり）、そして現在リクエストされたデータがTiDBに部分的に送信されていることを示しています。  | DMはこのようなエラーに対して自動的にリカバリを提供します。リカバリが長時間成功しない場合は、エラーメッセージをさらに確認し、実際の状況に基づいて情報を分析してください。 |
| `code=10005` |  `QUERY`タイプのSQLステートメントを実行する際に発生します。                                         |                                                              |
| `code=10006` |  `EXECUTE`タイプのSQLステートメント（DDLステートメントおよび`INSERT`、`UPDATE`、`DELETE`タイプのDMLステートメントを含む）を実行する際に発生します。詳細なエラー情報については、通常はデータベース操作に対するエラーコードとエラー情報が含まれていますので、エラーメッセージを確認してください。|                                                              |
| `code=11006` |  DMの組み込みパーサが互換性のないDDLステートメントを解析する際に発生します。          |  解決策については[データ移行 - 互換性のないDDLステートメント](/dm/dm-faq.md#how-to-handle-incompatible-ddl-statements)を参照してください。 |
| `code=20010` |   タスク構成で提供されたデータベースパスワードの復号化時に発生します。                   |  構成タスクで提供されたダウンストリームのデータベースパスワードが[dmctlを使用して正しく暗号化されているかどうか](/dm/dm-manage-source.md#encrypt-the-database-password)を確認してください。 |
| `code=26002` |  タスクチェックに失敗し、データベース接続が確立できない場合に発生します。詳細なエラー情報については、通常はデータベース操作のためのエラーコードとエラー情報が含まれているため、エラーメッセージを確認してください。 |  DM-masterが設置されているマシンが上流にアクセスする権限を持っているかどうかを確認してください。 |
| `code=32001` |   異常なダンプ処理ユニット                                            |  エラーメッセージに`mydumper: argument list too long.`が含まれる場合は、`task.yaml`ファイルのMydumper引数`extra-args`にブロック許可リストに応じて表を追加するために`--regex`正規表現を手動で追加してください。たとえば、`hello`という名前のすべてのテーブルをエクスポートするには、`--regex '.*\.hello$'`を追加します。すべてのテーブルをエクスポートするには、`--regex '.*'`を追加します。 |
| `code=38008` |  DMコンポーネント間のgRPC通信でエラーが発生します。                                     |  `class`を確認してください。どのコンポーネントの相互作用でエラーが発生しているかを特定してください。通信エラーのタイプを決定してください。エラーがgRPC接続の確立時に発生する場合は、通信サーバーが正常に動作しているかどうかを確認してください。 |

### `無効な接続`エラーが返された場合の移行タスクの中断時の対処方法は何ですか？

#### 理由

`無効な接続`エラーは、DMとダウンストリームのTiDBデータベース間に異常が発生していること（ネットワーク障害、TiDB再起動、TiKVがビジーなど）、および現在のリクエストにおけるデータの一部がTiDBに送信されていることを示しています。

#### 解決策

DMは移行タスクでダウンストリームにデータを同時に移行する機能を持っているため、タスクが中断された際にいくつかのエラーが発生する可能性があります。`query-status`を使用してこれらのエラーを確認できます。

- インクリメンタルレプリケーションプロセス中に`無効な接続`エラーのみが発生する場合、DMはタスクを自動的に再試行します。
- バージョンの問題でDMが自動的な再試行を行わないか、または失敗した場合は、`stop-task`を使用してタスクを停止し、その後`start-task`を使用してタスクを再開してください。

### `driver: bad connection`エラーが返された場合の移行タスクの中断時の対処方は何ですか？

#### 理由

`driver: bad connection`エラーは、DMと上流のTiDBデータベース間の接続に異常が発生していること（ネットワーク障害やTiDB再起動など）、およびその時点での現在のリクエストのデータがTiDBにまだ送信されていないことを示しています。

#### 解決策

現在のバージョンのDMはエラー時に自動的にリトライします。自動的なリトライをサポートしていない以前のバージョンを使用している場合は、`stop-task`コマンドを実行してタスクを停止し、その後`start-task`を実行してタスクを再開してください。

### リレーユニットが`event from * in * diff from passed-in event *`エラーを報告したり、`get binlog error ERROR 1236 (HY000)`および`binlog checksum mismatch, data may be corrupted`などのエラーを返してデータの取得または解析が失敗した場合の対処方法

#### 理由

DMのリレーログのプルやインクリメンタルレプリケーションのプロセス中に、これらの2つのエラーが発生する可能性があります。これらのエラーは、上流のbinlogファイルのサイズが **4 GB**を超えた場合に発生する可能性があります。

**原因:** リレーログの書き込み時、DMはbinlog位置とbinlogファイルのサイズを基にイベントの検証を行い、複製されたbinlog位置をチェックポイントとして保存する必要があります。しかし、公式MySQLではbinlog位置を格納するために`uint32`を使用しており、これにより4 GBを超えるbinlogファイルの位置がオーバーフローし、上記のエラーが発生します。

#### 解決策

リレーユニットについては、以下の解決策を使用して手動で移行を回復してください。

1. エラーが発生した際、上流の対応するbinlogファイルのサイズが4GBを超えていることを特定してください。

2. DM-workerを停止します。

3. 対応するbinlogファイルを上流からリレーログディレクトリにコピーします。

4. リレーログディレクトリで、`relay.meta`ファイルを更新して次のbinlogファイルからプルするようにしてください。DM-workerで`enable_gtid`を`true`に指定した場合は、`relay.meta`ファイルを更新する際に次のbinlogファイルに対応するGTIDを修正する必要があります。それ以外の場合は、GTIDを修正する必要はありません。

    例: エラーが発生した際の`binlog-name = "mysql-bin.004451"`と`binlog-pos = 2453`をそれぞれ`binlog-name = "mysql-bin.004452"`と`binlog-pos = 4`に更新し、`relay.meta`ファイルを更新し、`binlog-gtid`を`f0e914ef-54cf-11e7-813d-6c92bf2fa791:1-138218058`に更新してください。

5. DM-workerを再起動します。

binlogレプリケーション処理ユニットについては、以下の解決策を使用して手動で移行を回復してください。

1. エラーが発生した際、上流の対応するbinlogファイルのサイズが4GBを超えていることを特定してください。

2. `stop-task`を使用して移行タスクを停止します。

3. ダウンストリームの`dm_meta`データベースのグローバルチェックポイントと各テーブルチェックポイントの`binlog_name`をエラーの発生したbinlogファイルの名前に、`binlog_pos`を移行が完了した有効な位置の値、例えば4に更新してください。

    例: エラーが発生したタスクの名前が`dm_test`、対応する`source-id`が`replica-1`で、対応するbinlogファイルが`mysql-bin|000001.004451`である場合は、次のコマンドを実行してください。

    {{< copyable "sql" >}}

    ```sql
    UPDATE dm_test_syncer_checkpoint SET binlog_name='mysql-bin|000001.004451', binlog_pos = 4 WHERE id='replica-1';
    ```

4. 移行タスク構成の`syncers`セクションで`safe-mode: true`を指定して、再起可能であることを確認してください。

5. `start-task`を使用して移行タスクを開始してください。

6. `query-status`を使用して移行タスクの状態を確認してください。元のエラーを引き起こしたリレーログファイルの移行が完了した時点で、`safe-mode`を元の値に戻し、移行タスクを再開してください。

### タスクまたはログのクエリ時に`Access denied for user 'root'@'172.31.43.27' (using password: YES)`が表示される場合

すべてのDM構成ファイル内のデータベース関連のパスワードについては、`dmctl`で暗号化されたパスワードを使用することをお勧めします。データベースのパスワードが空の場合は、暗号化する必要はありません。平文パスワードを暗号化する方法については、[dmctlを使用してデータベースパスワードを暗号化する](/dm/dm-manage-source.md#encrypt-the-database-password)を参照してください。

さらに、上流および下流のデータベースのユーザーは、対応する読み取りおよび書き込み権限を持っている必要があります。データ移行はデータ移行タスクを開始する際に自動的に[対応する権限を事前にチェック](/dm/dm-precheck.md)します。

### `load`処理ユニットが`packet for query is too large. Try adjusting the 'max_allowed_packet' variable`エラーを報告した場合

#### 理由

* MySQLクライアントおよびMySQL/TiDBサーバーの両方には`max_allowed_packet`の割り当て制限があります。`max_allowed_packet`が制限を超過すると、クライアントがエラーメッセージを受信します。現在のDMおよびTiDBサーバーの最新バージョンでは、`max_allowed_packet`のデフォルト値は`64M`です。

* DMのDump処理ユニットによってエクスポートされたSQLファイルのフルデータインポート処理ユニットは、SQLファイルの分割をサポートしていません。

#### 解決策

* Dump処理ユニットの`extra-args`の`statement-size`オプションを設定することをお勧めします：

    デフォルトの`--statement-size`設定に従って、Dump処理ユニットによって生成される`Insert Statement`のデフォルトサイズは約`1M`です。このデフォルト設定により、ほとんどの場合、load処理ユニットは`packet for query is too large. Try adjusting the 'max_allowed_packet' variable`エラーを報告しません。

    時々、データのダンプ中に以下の`WARN`ログが表示されることがあります。この`WARN`ログはダンププロセスに影響しません。これは単にワイドテーブルがダンプされていることを意味します。

    ```
    Row bigger than statement_size for xxx
    ```
```
* If the single row of the wide table exceeds `64M`, you need to modify the following configurations and make sure the configurations take effect.

    * Execute `set @@global.max_allowed_packet=134217728` (`134217728` = 128 MB) in the TiDB server.

    * First add the `max-allowed-packet: 134217728` (128 MB) to the `target-database` section in the DM task configuration file. Then, execute the `stop-task` command and execute the `start-task` command.
```