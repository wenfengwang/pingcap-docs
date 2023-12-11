---
title: TiCDC FAQs
summary: TiCDCを使用する際に遭遇する可能性のあるFAQを学ぶ。

# TiCDC FAQs

このドキュメントでは、TiCDCを使用する際に遭遇する可能性のある一般的な質問について紹介します。

> **ノート：**
>
>`cdc cli`コマンドで指定されるサーバーアドレスは`--server=http://127.0.0.1:8300`です。コマンドを使用する際には、実際の PD アドレスに置き換えてください。

## TiCDCでタスクを作成する際の`start-ts`の選択方法は？

レプリケーションタスクの`start-ts`は、上流の TiDB クラスター内の Timestamp Oracle (TSO) に対応しています。TiCDCはこの TSO からのデータをレプリケーションタスクで要求します。したがって、レプリケーションタスクの`start-ts`は次の要件を満たす必要があります：

- `start-ts`の値は、現行の TiDB クラスターの`tikv_gc_safe_point`の値よりも大きい必要があります。そうでない場合、タスクの作成時にエラーが発生します。
- タスクを開始する前に、ダウンストリームで`start-ts`の前にすべてのデータがあることを確認してください。上流からメッセージキューにデータをレプリケートする場合、上流とダウンストリームのデータの整合性が必要ない場合は、アプリケーションの必要に応じてこの要件を緩和できます。

`start-ts`を指定しないか、または`start-ts`を`0`と指定した場合、レプリケーションタスクを開始すると、TiCDCは現在の TSO を取得し、この TSO からタスクを開始します。

## TiCDCでタスクを作成する際に、一部のテーブルがレプリケートされない理由は？

`cdc cli changefeed create`を実行してレプリケーションタスクを作成すると、TiCDCは上流のテーブルが[レプリケーション要件](/ticdc/ticdc-overview.md#best-practices)を満たしているかどうかをチェックします。一部のテーブルが要件を満たさない場合、`some tables are not eligible to replicate`というメッセージが返され、該当するテーブルのリストが表示されます。`Y`または`y`を選択してタスクの作成を続行すると、これらのテーブルのすべての更新がレプリケーション中に自動的に無視されます。`Y`または`y`以外の入力を選択した場合、レプリケーションタスクは作成されません。

## TiCDCのレプリケーションタスクの状態を表示する方法は？

TiCDCのレプリケーションタスクの状態を表示するには、`cdc cli`を使用します。例：

{{< copyable "shell-regular" >}}

```shell
cdc cli changefeed list --server=http://127.0.0.1:8300
```

期待される出力は次のとおりです：

```json
[{
    "id": "4e24dde6-53c1-40b6-badf-63620e4940dc",
    "summary": {
      "state": "normal",
      "tso": 417886179132964865,
      "checkpoint": "2020-07-07 16:07:44.881",
      "error": null
    }
}]
```

* `checkpoint`：TiCDCは、このタイムスタンプの前のすべてのデータをダウンストリームにレプリケートしています。
* `state`：このレプリケーションタスクの状態。各状態とその意味についての詳細は、[Changefeed states](/ticdc/ticdc-changefeed-overview.md#changefeed-state-transfer)を参照してください。

> **ノート：**
>
> この機能は TiCDC 4.0.3 で導入されました。

## TiCDCの`gc-ttl`とは？

v4.0.0-rc.1 以降、PDはサービスレベルの GC セーフポイントの設定に外部サービスをサポートしています。任意のサービスが GC セーフポイントを登録および更新できます。PDは、この GC セーフポイント以降のキーバリューデータが GC によって削除されないように確保します。

レプリケーションタスクが利用できなくなったり、中断された場合、この機能により、TiCDC によって利用されるデータが TiKV によって GC によって削除されることなく保持されます。

TiCDCサーバーを起動する際に、`gc-ttl`を構成することで、GC セーフポイントの Time To Live (TTL) 期間を指定できます。また、[TiUPを使用して変更](/ticdc/deploy-ticdc.md#modify-ticdc-cluster-configurations-using-tiup)することもできます。デフォルト値は24時間です。TiCDCでは、この値が次を意味します：

- TiCDCサービスが停止した後、PDで GC セーフポイントが保持される最大時間。
- タスクが中断された後、中断されたレプリケーションタスクが中断された時間が`gc-ttl`で指定された値よりも長い場合、レプリケーションタスクは`failed`ステータスに入り、再開できず、GC セーフポイントの進行に影響を与え続けることができません。

上記の2つ目の振る舞いは、TiCDC v4.0.13以降のバージョンで導入されました。その目的は、TiCDCのレプリケーションタスクが長時間中断されることによって、上流TiKVクラスターのGCセーフポイントが長時間続き、古いデータのバージョンが多く保持され、上流クラスターの性能に影響を与えることを防ぐことです。

> **ノート：**
>
> 一部のシナリオでは、たとえば、Dumpling/BR で完全なレプリケーション後に、TiCDCを使用して増分レプリケーションを行う場合は、デフォルトの24時間の `gc-ttl` が十分ではない場合があります。 TiCDCサーバーを起動する際に、`gc-ttl`に適切な値を指定する必要があります。

## TiCDCのガベージコレクション（GC）セーフポイントの完全な動作は？

レプリケーションタスクがTiCDCサービス開始後に開始される場合、TiCDC所有者は、すべてのレプリケーションタスクの`checkpoint-ts`の最小値で、PDサービスのGCセーフポイントを更新します。サービスのGCセーフポイントにより、TiCDCはその時点およびその時点以降に生成されたデータを削除しません。レプリケーションタスクが中断されたり、手動で停止された場合、このタスクの`checkpoint-ts`は変更されません。同時に、PDの対応するサービスのGCセーフポイントも更新されません。

レプリケーションタスクが`gc-ttl`で指定された時間よりも長く中断された場合、レプリケーションタスクは`failed`ステータスに入り、再開できません。PDの対応するサービスのGCセーフポイントは継続されます。

TiCDCがサービスのGCセーフポイントに設定するTime-To-Live（TTL）は24時間であり、これはTiCDCサービスが中断された後24時間以内に回復できる場合、GCメカニズムがデータを削除しないことを意味します。

## TiCDCのタイムゾーンと上流/下流データベースのタイムゾーンの関係を理解する方法は？

||上流のタイムゾーン| TiCDCのタイムゾーン|下流のタイムゾーン|
| :-: | :-: | :-: | :-: |
| 構成方法 | [タイムゾーンサポート](/configure-time-zone.md)を参照してください | TiCDCサーバーを起動する際に`--tz`パラメーターを使用して構成します | `sink-uri`内の`time-zone`パラメーターを使用して構成します |
| 説明 | 上流のTiDBのタイムゾーンであり、タイムスタンプ型の DML 操作やタイムスタンプ型の列に関連するDDL操作に影響を与えます。| TiCDCは、上流のTiDBのタイムゾーンがTiCDCのタイムゾーン構成と同じであると仮定し、タイムスタンプ列に関連する操作を実行します。| 下流のMySQLは、下流のタイムゾーン設定に応じて、DMLおよびDDL操作でタイムスタンプを処理します。|

 > **ノート：**
 >
 > TiCDCサーバーのタイムゾーンを設定する際には注意してください。なぜなら、このタイムゾーンは時間型の変換に使用されるためです。上流のタイムゾーン、TiCDCのタイムゾーン、下流のタイムゾーンを一致させてください。TiCDCサーバーは次の優先順位でタイムゾーンを選択します：
 >
 > - TiCDCはまず、`--tz`を使用して指定されたタイムゾーンを使用します。
 > - `--tz`が使用できない場合、TiCDCは`TZ`環境変数で設定されたタイムゾーンを読み取ろうとします。
 > - `TZ`環境変数を使用できない場合、TiCDCはマシンのデフォルトのタイムゾーンを使用します。

## `--config`で構成ファイルを指定せずに`cdc cli changefeed create`でレプリケーションタスクを作成した場合の、TiCDCのデフォルト動作は？

`cdc cli changefeed create`コマンドで`-config`パラメーターを指定せずに使用すると、TiCDCは次のデフォルトの振る舞いでレプリケーションタスクを作成します：

- システムテーブルを除くすべてのテーブルが複製されます。
- [有効なインデックスを含む](/ticdc/ticdc-overview.md#best-practices)テーブルのみが複製されます。

## TiCDCはCanal形式でデータ変更を出力することをサポートしていますか？

はい。Canal形式での出力を有効にするには、`--sink-uri`パラメーターでプロトコルを`canal`として指定します。例：

{{< copyable "shell-regular" >}}

```shell
cdc cli changefeed create --server=http://127.0.0.1:8300 --sink-uri="kafka://127.0.0.1:9092/cdc-test?kafka-version=2.4.0&protocol=canal" --config changefeed.toml
```

> **ノート：**
>
> * この機能はTiCDC 4.0.2で導入されました。
> * TiCDCは現時点で、Canal形式のデータ変更の出力をKafkaなどのMQシンクにのみサポートしています。

詳細については、[TiCDC changefeed configurations](/ticdc/ticdc-changefeed-config.md) を参照してください。

## TiCDCからKafkaへのレイテンシがどんどん高くなる理由は？

* [TiCDCのレプリケーションタスクの状態を表示する方法](#how-do-i-view-the-state-of-ticdc-replication-tasks)を確認します。
* Kafkaの次のパラメータを調整します：

    * `server.properties`の`message.max.bytes`の値を`1073741824`（1 GB）に増やします。
    * `server.properties` で `replica.fetch.max.bytes` の値を `1073741824` (1 GB) に増やしてください。
    * `consumer.properties` で `fetch.message.max.bytes` の値を `message.max.bytes` の値よりも大きくするように増やしてください。

## TiCDC がデータを Kafka にレプリケートする際、TiDB で単一メッセージの最大サイズを制御できますか？

`protocol` が `avro` または `canal-json` に設定されている場合、メッセージは行ごとに送信されます。単一の Kafka メッセージには1つの行の変更のみが含まれ、通常は Kafka の制限を超えることはありません。したがって、単一のメッセージのサイズを制限する必要はありません。単一の Kafka メッセージのサイズが Kafka の制限を超える場合は、[なぜ TiCDC から Kafka への遅延がますます高くなるのですか？](/ticdc/ticdc-faq.md#why-does-the-latency-from-ticdc-to-kafka-become-higher-and-higher) を参照してください。

`protocol` が `open-protocol` に設定されている場合、メッセージはバッチで送信されます。そのため、1つの Kafka メッセージが極端に大きくなることがあります。このような状況を回避するために、`max-message-bytes` パラメータを構成して、Kafka ブローカーに送信されるデータの最大サイズを制御できます (オプション、デフォルトでは `10MB`)。また、`max-batch-size` パラメータ (オプション、デフォルトでは `16`) を構成して、各 Kafka メッセージの中の変更レコードの最大数を指定することもできます。

## トランザクションで同じ行を複数回変更した場合、TiCDC は複数の行変更イベントを出力しますか？

いいえ。同じ行をトランザクション内で複数回変更する場合、TiDB は最新の変更のみを TiKV に送信します。そのため、TiCDC は最新の変更の結果のみを取得できます。

## TiCDC がデータを Kafka にレプリケートする際、メッセージに複数の種類のデータ変更が含まれますか？

はい。単一のメッセージには複数の `update` または `delete` が含まれることがあり、`update` と `delete` が同時に存在することがあります。

## TiCDC がデータを Kafka にレプリケートする際、TiCDC Open Protocol の出力でタイムスタンプ、テーブル名、およびスキーマ名をどのように表示できますか？

この情報は、Kafka メッセージのキーに含まれています。例:

```json
{
    "ts":<TS>,
    "scm":<Schema Name>,
    "tbl":<Table Name>,
    "t":1
}
```

詳細については、[TiCDC Open Protocol イベントフォーマット](/ticdc/ticdc-open-protocol.md#event-format) を参照してください。

## TiCDC がデータを Kafka にレプリケートする際、メッセージ内のデータ変更のタイムスタンプをどのように表示できますか？

Kafka メッセージのキー内の `ts` を 18 ビット右にシフトすることで、UNIX タイムスタンプを取得できます。

## TiCDC Open Protocol では `null` はどのように表現されますか？

TiCDC Open Protocol では、`6` のタイプコードが `null` を表します。

| タイプ | コード | 出力例 | 備考 |
|:--|:--|:--|:--|
| Null | 6 | `{"t":6,"v":null}` | |

詳細については、[TiCDC Open Protocol カラムタイプコード](/ticdc/ticdc-open-protocol.md#column-type-code) を参照してください。

## TiCDC Open Protocol の Row Changed Event が `INSERT` イベントまたは `UPDATE` イベントであるかどうかをどのように特定できますか？

* `UPDATE` イベントには、`"p"` フィールドと `"u"` フィールドの両方が含まれます
* `INSERT` イベントには、`"u"` フィールドのみが含まれます
* `DELETE` イベントには、`"d"` フィールドのみが含まれます

詳細については、[Open protocol Row Changed Event format](/ticdc/ticdc-open-protocol.md#row-changed-event) を参照してください。

## TiCDC はいくらの PD ストレージを使用しますか？

TiCDC は PD の etcd を使用してメタデータを格納し、定期的に更新します。etcd の MVCC と PD のデフォルト圧縮の時間間隔が 1 時間なので、TiCDC が使用する PD ストレージの量は、この期間内に生成されるメタデータバージョンの量に比例します。ただし、v4.0.5、v4.0.6、および v4.0.7 では、TiCDC が頻繁に書き込む問題が発生しており、1時間で 1000 のテーブルが作成される場合、etcd ストレージをすべて使用して `etcdserver: mvcc: database space exceeded` エラーが返されます。このエラーが発生した後は、etcd ストレージをクリーンアップする必要があります。詳細については、[etcd maintenance space-quota](https://etcd.io/docs/v3.4.0/op-guide/maintenance/#space-quota) を参照してください。クラスタを v4.0.9 またはそれ以降のバージョンにアップグレードすることをお勧めします。

## TiCDC は大きなトランザクションのレプリケーションをサポートしていますか？それにはリスクはありますか？

TiCDC は大規模なトランザクション (5 GB より大きい) の部分的なサポートを提供しています。異なるシナリオによっては、次のリスクが存在する場合があります:

- プライマリとセカンダリのレプリケーションの遅延が大幅に上昇することがあります。
- TiCDC の内部処理能力が不十分な場合、レプリケーションタスクエラー `ErrBufferReachLimit` が発生する場合があります。
- TiCDC の内部処理能力が不十分であるか、TiCDC のダウンストリームのスループット容量が不十分な場合、メモリ不足 (OOM) が発生する場合があります。

v6.2 から、TiCDC は単一テーブルトランザクションを複数のトランザクションに分割する機能をサポートしています。これにより、大規模なトランザクションのレプリケーションの遅延とメモリの消費を大幅に削減できます。したがって、アプリケーションがトランザクションの原子性に高い要求を持たない場合は、可能な限り大規模なトランザクションの分割を有効にすることをお勧めします。分割を有効にするには、シンク URI パラメータ [`transaction-atomicity`](/ticdc/ticdc-sink-to-mysql.md#configure-sink-uri-for-mysql-or-tidb) の値を `none` に設定してください。

それでも上記のエラーが発生する場合は、BR を使用して大規模なトランザクションの増分データを復元することをお勧めします。具体的な手順は以下の通りです:

1. 大規模なトランザクションによって終了された changefeed の `checkpoint-ts` を記録し、これを BR 増分バックアップの `--lastbackupts` として使用して、[増分データバックアップ](/br/br-incremental-guide.md#back-up-incremental-data) を実行します。
2. 増分データをバックアップした後、BR ログ出力に`["Full backup Failed summary : total backup ranges: 0, total success: 0, total failed: 0"] [BackupTS=421758868510212097]` のようなログがあります。この `BackupTS` を記録します。
3. [増分データを復元](/br/br-incremental-guide.md#restore-incremental-data) します。
4. 新しい changefeed を作成し、`BackupTS` からのレプリケーションタスクを開始します。
5. 古い changefeed を削除します。

## TiCDC は、損失のある DDL 操作によって引き起こされたデータ変更をダウンストリームにレプリケートしますか？

損失のある DDL とは、TiDB で実行された場合にデータ変更を引き起こす可能性がある DDL のことです。一般的な損失のある DDL 操作には次のものがあります:

- 列の型の変更 (例: INT -> VARCHAR)
- 列の長さの変更 (例: VARCHAR(20) -> VARCHAR(10))
- 列の精度の変更 (例: DECIMAL(10, 3) -> DECIMAL(10, 2))
- 列の UNSIGNED または SIGNED 属性の変更 (例: INT UNSIGNED -> INT SIGNED)

TiDB v7.1.0 以前では、TiCDC は古いデータと新しいデータが同一の場合の DML イベントをダウンストリームにレプリケートします。ダウンストリームが MySQL の場合、これらの DML イベントは、ダウンストリームが DDL ステートメントを受け取って実行するまではデータ変更を引き起こしません。ただし、ダウンストリームが Kafka またはクラウドストレージサービスの場合、TiCDC は冗長なデータをダウンストリームに書き込みます。

TiDB v7.1.0 から、TiCDC はこれらの冗長な DML イベントを排除し、それらをダウンストリームにはレプリケートしなくなりました。

## タイムタイプフィールドのデフォルト値が、ダウンストリームの MySQL 5.7 に DDL ステートメントをレプリケートする際に不一致です。これにはどのように対処すればよいですか？

上流の TiDB で `create table test (id int primary key, ts timestamp)` ステートメントが実行されたとします。TiCDC がこのステートメントをダウンストリームの MySQL 5.7 にレプリケートすると、MySQL はデフォルトの構成を使用します。レプリケーション後のテーブルスキーマは次のようになります。`timestamp` フィールドのデフォルト値は `CURRENT_TIMESTAMP` になります。

{{< copyable "sql" >}}

```sql
mysql root@127.0.0.1:test> show create table test;
+-------+----------------------------------------------------------------------------------+
| Table | Create Table                                                                     |
+-------+----------------------------------------------------------------------------------+
| test  | CREATE TABLE `test` (                                                            |
|       |   `id` int(11) NOT NULL,                                                         |
|       |   `ts` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP, |
|       |   PRIMARY KEY (`id`)                                                             |
|       | ) ENGINE=InnoDB DEFAULT CHARSET=latin1                                           |
+-------+----------------------------------------------------------------------------------+
1 row in set
```

結果からわかるように、レプリケーション前後のテーブルスキーマが一致しないことがあります。これは、TiDB の `explicit_defaults_for_timestamp` のデフォルト値が MySQL のそれと異なるためです。詳細については、[MySQL 互換性](/mysql-compatibility.md#default-differences) を参照してください。
```markdown
自 v5.0.1 或 v4.0.13 起，对于每个到 MySQL 的复制，TiCDC 自动设置 `explicit_defaults_for_timestamp = ON`，以确保时间类型在上游和下游之间保持一致。在早于 v5.0.1 或 v4.0.13 的版本中，在使用 TiCDC 复制时间类型数据时，请注意由于不一致的 `explicit_defaults_for_timestamp` 值可能引起的兼容性问题。

## 当创建 TiCDC 复制任务时，如果我将 `safe-mode` 设置为 `true`，为什么来自上游的 `INSERT`/`UPDATE` 语句在复制到下游后变成了 `REPLACE INTO`？

TiCDC 保证所有数据至少被复制一次。当下游存在重复数据时，会发生写入冲突。为避免此问题，TiCDC 将 `INSERT` 和 `UPDATE` 语句转换为 `REPLACE INTO` 语句。此行为由 `safe-mode` 参数控制。

在早于 v6.1.3 的版本中，`safe-mode` 默认为 `true`，这意味着所有 `INSERT` 和 `UPDATE` 语句都会被转换为 `REPLACE INTO` 语句。在 v6.1.3 及之后的版本中，TiCDC 可以自动判断下游是否存在重复数据，并且 `safe-mode` 的默认值更改为 `false`。如果未检测到重复数据，TiCDC 将不进行转换，而是直接复制 `INSERT` 和 `UPDATE` 语句。

## 在复制下游为 TiDB 或 MySQL 时，下游数据库的用户需要什么权限？

当下游为 TiDB 或 MySQL 时，下游数据库的用户需要以下权限：

- `Select`
- `Index`
- `Insert`
- `Update`
- `Delete`
- `Create`
- `Drop`
- `Alter`
- `Create View`

如果需要将 `recover table` 复制到下游 TiDB，则应拥有 `Super` 权限。

## TiCDC 为什么需要使用磁盘？TiCDC 何时向磁盘写入？TiCDC 是否使用内存缓冲以提高复制性能？

在上游写入流量处于高峰时，下游可能无法及时消费所有数据，导致数据积压。TiCDC 使用磁盘来处理积压的数据。TiCDC 在正常运行时需要向磁盘写入数据。然而，这通常不会成为复制吞吐量和复制延迟的瓶颈，因为向磁盘写入仅导致百毫秒级的延迟。TiCDC 还使用内存来加速从磁盘读取数据，以提高复制性能。

## 使用 TiDB Lightning 和 BR 从上游恢复数据后，为什么使用 TiCDC 进行复制会出现停滞甚至停止的情况？

目前，TiCDC 与 TiDB Lightning 和 BR 尚未完全兼容。因此，请避免在被 TiCDC 复制的表上使用 TiDB Lightning 和 BR。

## 暂停后，变更数据捕获任务（changefeed）恢复后，其复制延迟会越来越高，并且仅在数分钟后才能恢复正常。为什么？

当变更数据捕获任务恢复时，TiCDC 需要扫描 TiKV 中数据的历史版本，以赶上暂停期间生成的增量数据日志。只有在扫描完成后，复制过程才会继续进行。扫描过程可能需要数十分钟甚至数小时。

## 我应该如何将 TiCDC 部署来在不同区域的两个 TiDB 集群之间复制数据？

对于早于 v6.5.2 的 TiCDC 版本，建议您将 TiCDC 部署在下游 TiDB 集群中。如果上游和下游之间的网络延迟较高，例如超过 100 毫秒，当 TiCDC 向下游执行 SQL 语句时可能会出现显著的延迟，这是由于 MySQL 传输协议问题引起的。而将 TiCDC 部署在下游可以极大地缓解此问题。经过优化后，从 TiCDC v6.5.2 开始，建议将 TiCDC 部署在上游 TiDB 集群中。

## 执行 DML 和 DDL 语句的顺序是怎样的？

目前，TiCDC 遵循以下顺序：

1. TiCDC 阻塞受 DDL 语句影响的表的复制进度，直到 DDL `CommiTs`。这确保了在 DDL `CommiTs` 之前执行的 DML 语句可以成功复制到下游。
2. TiCDC 继续复制 DDL 语句。如果有多个 DDL 语句，TiCDC 会按照串行方式复制它们。
3. 在下游执行完 DDL 语句后，TiCDC 将继续复制在 DDL `CommiTs` 之后执行的 DML 语句。

## 我应该如何检查上游和下游数据是否一致？

如果下游为 TiDB 集群或 MySQL 实例，建议使用 [sync-diff-inspector](/sync-diff-inspector/sync-diff-inspector-overview.md) 来比较数据。

## 单表的复制只能在单个 TiCDC 节点上运行。能否使用多个 TiCDC 节点来复制多个表的数据？

从 v7.1.0 开始，TiCDC 支持 MQ sink 来以 TiKV Region 的粒度复制数据变更日志，这实现了可扩展的处理能力，并允许 TiCDC 复制拥有大量 Region 的单表。要启用此功能，您可以在 [TiCDC 配置文件](/ticdc/ticdc-changefeed-config.md) 中配置以下参数：

```toml
[scheduler]
enable-table-across-nodes = true
```

## 如果上游存在长时间运行的未提交事务，TiCDC 复制是否会卡住？

TiDB 具有事务超时机制。当事务运行时间超过 [`max-txn-ttl`](/tidb-configuration-file.md#max-txn-ttl) 时，TiDB 会强制将其回滚。TiCDC 在复制之前会等待事务提交，这会导致复制延迟。

## 为什么我无法使用 `cdc cli` 命令操作 TiDB Operator 部署的 TiCDC 集群？

这是因为 TiDB Operator 部署的 TiCDC 集群的默认端口号为 `8301`，而 `cdc cli` 命令连接到 TiCDC 服务器的默认端口号为 `8300`。在使用 `cdc cli` 命令操作 TiDB Operator 部署的 TiCDC 集群时，您需要显式指定 `--server` 参数，如下所示：

```shell
./cdc cli changefeed list --server "127.0.0.1:8301"
[
  {
    "id": "4k-table",
    "namespace": "default",
    "summary": {
      "state": "stopped",
      "tso": 441832628003799353,
      "checkpoint": "2023-05-30 22:41:57.910",
      "error": null
    }
  },
  {
    "id": "big-table",
    "namespace": "default",
    "summary": {
      "state": "normal",
      "tso": 441872834546892882,
      "checkpoint": "2023-06-01 17:18:13.700",
      "error": null
    }
  }
]
```