---
title: Troubleshoot TiCDC
summary: TiCDCを使用する際に遭遇する問題のトラブルシューティング方法を学ぶ
aliases: ['/docs/dev/ticdc/troubleshoot-ticdc/']
---

# Troubleshoot TiCDC

このドキュメントでは、TiCDCを使用する際に遭遇する一般的なエラーと、それに対応するメンテナンスおよびトラブルシューティング方法について紹介します。

> **注意:**
>
> このドキュメントでは、`cdc cli`コマンドで指定されたサーバーアドレスは `server=http://127.0.0.1:8300` です。コマンドを使用する際は、実際のTiCDCサーバーアドレスにアドレスを置き換えてください。

## TiCDC複製の中断

### TiCDC複製タスクが中断されたかどうかを知る方法

- Grafanaダッシュボードで複製タスクの `changefeed checkpoint` 監視メトリクス (適切な `changefeed id` を選択) を確認します。メトリクス値が変わらないか、 `checkpoint lag` メトリクスが増加し続ける場合、複製タスクが中断している可能性があります。
- `exit error count` 監視メトリクスを確認します。メトリクス値が `0` より大きい場合、複製タスクでエラーが発生しています。
- `cdc cli changefeed list` および `cdc cli changefeed query` を実行して、複製タスクの状態を確認します。`stopped` はタスクが停止しており、 `error` 項目は詳細なエラーメッセージを提供します。エラーが発生した後は、トラブルシューティングのためにTiCDCサーバーログで `error on running processor` を検索できます。
- 極端なケースでは、TiCDCサービスが再起動されます。トラブルシューティングのためにTiCDCサーバーログで `FATAL` レベルのログを検索できます。

### 複製タスクが手動で停止されたかどうかを知る方法

`cdc cli`を実行して、複製タスクが手動で停止されたかどうかを知ることができます。例：

{{< copyable "shell-regular" >}}

```shell
cdc cli changefeed query --server=http://127.0.0.1:8300 --changefeed-id 28c43ffc-2316-4f4f-a70b-d1a7c59ba79f
```

上記コマンドの出力にて `admin-job-type` がこの複製タスクの状態を示します。各状態とその意味についての詳細は、[Changefeed states](/ticdc/ticdc-changefeed-overview.md#changefeed-state-transfer) を参照してください。

### 複製中断を処理する方法

次の既知のシナリオで複製タスクが中断される可能性があります：

- 下流が異常続き、多くのリトライの後にもTiCDCが失敗する。

    - このシナリオでは、TiCDCはタスク情報を保存します。TiCDCはPDにサービスGCセーフポイントを設定しているため、タスクチェックポイント以降のデータは `gc-ttl` の有効期間内にTiKV GCによってクリーンアップされません。

    - 処理方法: 下流が正常に戻った後、 `cdc cli changefeed resume` を実行して複製タスクを再開できます。

- 下流の非互換なSQL文によって複製が続行できない。

    - このシナリオでは、TiCDCはタスク情報を保存します。TiCDCはPDにサービスGCセーフポイントを設定しているため、タスクチェックポイント以降のデータは `gc-ttl` の有効期間内にTiKV GCによってクリーンアップされません。

    - 処理手順:
        1. `cdc cli changefeed query` コマンドを使用して複製タスクのステータス情報をクエリし、`checkpoint-ts` の値を記録します。
        2. 新しいタスク構成ファイルを使用し、 `ignore-txn-start-ts` パラメーターを追加して指定された `start-ts` に対応するトランザクションをスキップします。
        3. `cdc cli changefeed pause -c <changefeed-id>` を実行して複製タスクを一時停止します。
        4. `cdc cli changefeed update -c <changefeed-id> --config <config-file-path>` を実行して新しいタスク構成ファイルを指定します。
        5. `cdc cli changefeed resume -c <changefeed-id>` を実行して複製タスクを再開します。

### タスク中断後のTiCDC再起動時に発生するOOMを処理するにはどうすればよいですか？

TiDBクラスターとTiCDCクラスターを最新バージョンに更新してください。OOMの問題は **v4.0.14およびそれ以降のv4.0バージョン、v5.0.2およびそれ以降のv5.0バージョン、最新バージョン** で既に解決されています。

## 複製タスクまたはMySQLへのデータ複製の作成時に `Error 1298: Unknown or incorrect time zone: 'UTC'` エラーを処理するにはどうすればよいですか？

このエラーは、下流のMySQLがタイムゾーンをロードしていない場合に返されます。[`mysql_tzinfo_to_sql`](https://dev.mysql.com/doc/refman/8.0/en/mysql-tzinfo-to-sql.html)を実行してタイムゾーンをロードできます。タイムゾーンをロードした後、通常通りタスクを作成しデータを複製できます。

{{< copyable "shell-regular" >}}

```shell
mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -u root mysql -p
```

上記コマンドの出力が以下の出力に似ている場合、インポートが成功しています。

```
Enter password:
Warning: Unable to load '/usr/share/zoneinfo/iso3166.tab' as time zone. Skipping it.
Warning: Unable to load '/usr/share/zoneinfo/leap-seconds.list' as time zone. Skipping it.
Warning: Unable to load '/usr/share/zoneinfo/zone.tab' as time zone. Skipping it.
Warning: Unable to load '/usr/share/zoneinfo/zone1970.tab' as time zone. Skipping it.
```

下流が特殊なMySQL環境（パブリッククラウドRDSまたは一部のMySQL派生バージョン）でタイムゾーンのインポートを行うことができない場合、 `time-zone` を空の値に設定して、デフォルトのタイムゾーンを使用できます（ `time-zone=""` ）。

TiCDCでタイムゾーンを使用する際は、 `time-zone="Asia/Shanghai"` など、タイムゾーンを明示的に指定することをお勧めします。また、TiCDCサーバー構成で指定された `tz` およびSink URIで指定された `time-zone` が下流データベースのタイムゾーン構成と一致していることを確認してください。これにより、不整合なタイムゾーンによるデータの不整合を防ぐことができます。

## TiCDCのアップグレードによって構成ファイルの非互換性の問題を処理する方法は？

[互換性に関する注意事項](/ticdc/ticdc-compatibility.md)を参照してください。

## TiCDCタスクの `start-ts` タイムスタンプが現在の時間とかなり異なる。このタスクの実行中に複製が中断され、エラー `[CDC:ErrBufferReachLimit]` が発生する。どうすればよいですか？

v4.0.9から、複製タスクで統一ソーター機能を有効にするか、増分バックアップとリストアのためにBRツールを使用し、新しい時間からTiCDC複製タスクを開始することができます。

## TiCDCの下流がMySQLに類似したデータベースで、TiCDCが時間のかかるDDL文を実行すると、他のすべてのchangefeedがブロックされます。どうすればよいですか？

1. 時間のかかるDDL文を含むchangefeedの実行を一時停止します。その後、他のchangefeedがブロックされなくなります。
2. TiCDCログで `apply job` フィールドを検索し、時間のかかるDDL文の `start-ts` を確認します。
3. 下流でDDL文を手動で実行します。実行が完了した後は、次の操作を続けます。
4. changefeed構成を変更し、 `start-ts` を `ignore-txn-start-ts` 構成項目に追加します。
5. 一時停止したchangefeedを再開します。

## TiCDCを使用してメッセージをKafkaにレプリケートする際、Kafkaが `Message was too large` エラーを返します。なぜですか？

TiCDC v4.0.8またはそれ以前のバージョンでは、Kafkaの `max-message-bytes` 設定を構成するだけでは、Kafkaに出力されるメッセージのサイズを効果的に制御することができません。メッセージの受信バイト数の制限を増やすには、Kafkaサーバー構成に以下の構成を追加してください。

```
# ブローカーが受信するメッセージの最大バイト数
```
message.max.bytes=2147483648
replica.fetch.max.bytes=2147483648
fetch.message.max.bytes=2147483648

## TiCDCレプリケーション中にDDLステートメントの実行が失敗したかどうかをどのように確認し、レプリケーションを再開できますか？

DDLステートメントの実行が失敗した場合、レプリケーションタスク（changefeed）は自動的に停止します。チェックポイントTSはDDLステートメントの完了TSから1を引いたものです。TiCDCにこのステートメントの再実行を試みさせたい場合は、`cdc cli changefeed resume` を使用してレプリケーションタスクを再開します。例：

{{< copyable "shell-regular" >}}

```shell
cdc cli changefeed resume -c test-cf --server=http://127.0.0.1:8300
```

失敗したDDLステートメントをスキップしたい場合は、changefeedのstart-tsをチェックポイントTS（DDLステートメントが失敗したタイムスタンプ）に1を加えたものに設定し、`cdc cli changefeed create` コマンドを実行して新しいchangefeedタスクを作成します。たとえば、DDLステートメントが失敗したチェックポイントTSが `415241823337054209` の場合、次のコマンドを実行してこのDDLステートメントをスキップします：

{{< copyable "shell-regular" >}}

```shell
cdc cli changefeed remove --server=http://127.0.0.1:8300 --changefeed-id simple-replication-task
cdc cli changefeed create --server=http://127.0.0.1:8300 --sink-uri="mysql://root:123456@127.0.0.1:3306/" --changefeed-id="simple-replication-task" --sort-engine="unified" --start-ts 415241823337054210
```