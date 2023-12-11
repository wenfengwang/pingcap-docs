---
title: TiCDC OpenAPI v1
summary: クラスタステータスとデータ複製を管理するためのOpenAPIインターフェースの使用方法を学びます。

# TiCDC OpenAPI v1

> **注意**
>
> TiCDC OpenAPI v1 は非推奨となり、将来削除されます。[TiCDC OpenAPI v2](/ticdc/ticdc-open-api-v2.md) を使用することを推奨します。

TiCDCは、TiCDCクラスタのクエリおよび操作のためのOpenAPI機能を提供しており、これは[`cdc cli`ツール](/ticdc/ticdc-manage-changefeed.md)の機能に類似しています。

以下のメンテナンス操作を行うためにAPIを使用できます：

- [TiCDCノードのステータス情報を取得する](#get-the-status-information-of-a-ticdc-node)
- [TiCDCクラスタのヘルスステータスを確認する](#check-the-health-status-of-a-ticdc-cluster)
- [レプリケーションタスクを作成する](#create-a-replication-task)
- [レプリケーションタスクを削除する](#remove-a-replication-task)
- [レプリケーション構成を更新する](#update-the-replication-configuration)
- [レプリケーションタスクリストをクエリする](#query-the-replication-task-list)
- [特定のレプリケーションタスクをクエリする](#query-a-specific-replication-task)
- [レプリケーションタスクを一時停止する](#pause-a-replication-task)
- [レプリケーションタスクを再開する](#resume-a-replication-task)
- [レプリケーションサブタスクリストをクエリする](#query-the-replication-subtask-list)
- [特定のレプリケーションサブタスクをクエリする](#query-a-specific-replication-subtask)
- [TiCDCサービスプロセスリストをクエリする](#query-the-ticdc-service-process-list)
- [所有者ノードを追い出す](#evict-an-owner-node)
- [レプリケーションタスク内のすべてのテーブルの手動ロードバランシングをトリガーする](#manually-trigger-the-load-balancing-of-all-tables-in-a-replication-task)
- [テーブルを別のノードに手動でスケジュールする](#manually-schedule-a-table-to-another-node)
- [TiCDCサーバのログレベルを動的に調整する](#dynamically-adjust-the-log-level-of-the-ticdc-server)

すべてのAPIのリクエストボディと返される値はJSON形式です。次のセクションでは、APIの具体的な使用方法について説明します。

以下の例では、TiCDCサーバのリスニングIPアドレスは `127.0.0.1` で、ポートは `8300` です。TiCDCサーバを起動する際に、`--addr=ip:port` を使用して指定のIPとポートをバインドできます。

## APIエラーメッセージテンプレート

APIリクエストを送信した後、エラーが発生した場合は、返されるエラーメッセージは以下の形式です：

```json
{
    "error_msg": "",
    "error_code": ""
}
```

上記のJSON出力から、`error_msg` はエラーメッセージを、`error_code` は対応するエラーコードを示します。

## TiCDCノードのステータス情報を取得する

このAPIは同期インタフェースです。リクエストが成功した場合、対応するノードのステータス情報が返されます。

### リクエストURI

`GET /api/v1/status`

### 例

以下のリクエストは、IPアドレスが `127.0.0.1` でポート番号が `8300` のTiCDCノードのステータス情報を取得します。

{{< copyable "shell-regular" >}}

```shell
curl -X GET http://127.0.0.1:8300/api/v1/status
```

```json
{
    "version": "v5.2.0-master-dirty",
    "git_hash": "f191cd00c53fdf7a2b1c9308a355092f9bf8824e",
    "id": "c6a43c16-0717-45af-afd6-8b3e01e44f5d",
    "pid": 25432,
    "is_owner": true
}
```

上記の出力の各フィールドは、以下のように説明されています：

- version: 現在のTiCDCのバージョン番号。
- git_hash: Gitのハッシュ値。
- id: ノードのキャプチャID。
- pid: ノードのキャプチャプロセスPID。
- is_owner: ノードが所有者であるかどうかを示します。

## TiCDCクラスタのヘルスステータスを確認する

このAPIは同期インタフェースです。クラスタが健康であれば、`200 OK` が返されます。

### リクエストURI

`GET /api/v1/health`

### 例

{{< copyable "shell-regular" >}}

```shell
curl -X GET http://127.0.0.1:8300/api/v1/health
```

## レプリケーションタスクを作成する

このAPIは非同期インタフェースです。リクエストが成功した場合、`202 Accepted` が返されます。返される結果は、サーバーがコマンドを実行することに同意したことを意味しますが、コマンドが成功裏に実行されることを保証するものではありません。

### リクエストURI

`POST /api/v1/changefeeds`

### パラメータの説明

`cdc cli`コマンドを使用してレプリケーションタスクを作成する際のオプションパラメータと比較して、APIを使用してそのようなタスクを作成するためのオプションパラメータは完全ではありません。このAPIは、以下のパラメータをサポートしています。

#### リクエストボディのパラメータ

| パラメータ名 | 説明 |
| :------------------------ | :---------------------- ------------------------------- |
| `changefeed_id` | `STRING`型。レプリケーションタスクのID。 (オプション) |
| `start_ts` | `UINT64`型。changefeedの開始TSOを指定します。 (オプション) |
| `target_ts` | `UINT64`型。changefeedのターゲットTSOを指定します。 (オプション) |
| **`sink_uri`** | `STRING`型。レプリケーションタスクのダウンストリームアドレス。 (**必須**) |
| `force_replicate` | `BOOLEAN`型。一意のインデックスを持たないテーブルを強制的にレプリケートするかどうかを決定します。 (オプション) |
| `ignore_ineligible_table` | `BOOLEAN`型。レプリケートできないテーブルを無視するかどうかを決定します。 (オプション) |
| `filter_rules` | `STRING`型の配列。テーブルスキーマフィルタリングのルール。 (オプション) |
| `ignore_txn_start_ts` | `UINT64`型の配列。指定された start_ts のトランザクションを無視します。 (オプション) |
| `mounter_worker_num` | `INT`型。マウンタースレッドの数。 (オプション) |
| `sink_config` | sinkの構成パラメータ。 (オプション) |

`changefeed_id`、`start_ts`、`target_ts`、`sink_uri` の意味と形式は、[レプリケーションタスクを作成する](/ticdc/ticdc-manage-changefeed.md#create-a-replication-task) ドキュメントに記載されているものと同じです。これらのパラメータの詳細な説明については、このドキュメントをご覧ください。`sink_uri` で証明書パスを指定する場合は、対応する証明書を対応するTiCDCサーバにアップロードしていることを確認してください。

上記の表の他のパラメータについて詳細に説明します。

`force_replicate`： このパラメータのデフォルト値は `false` です。`true` と指定された場合、TiCDCは一意のインデックスを持たないテーブルを強制的にレプリケートしようとします。

`ignore_ineligible_table`： このパラメータのデフォルト値は `false` です。`true` と指定された場合、TiCDCはレプリケートできないテーブルを無視します。

`filter_rules`： テーブルスキーマフィルタリングのルール。例： `filter_rules = ['foo*.*','bar*.*']`。詳細については、[Table Filter](/table-filter.md) ドキュメントを参照してください。

`ignore_txn_start_ts`： このパラメータが指定された場合、指定された start_ts が無視されます。例： `ignore-txn-start-ts = [1, 2]`。

`mounter_worker_num`： マウンターのスレッド数。マウンターは、TiKVからの出力データをデコードするために使用されます。デフォルト値は `16` です。

sinkの構成パラメータは以下の通りです：

```json
{
  "dispatchers":[
    {"matcher":["test1.*", "test2.*"], "dispatcher":"ts"},
    {"matcher":["test3.*", "test4.*"], "dispatcher":"rowid"}
  ],
  "protocal":"canal-json"
}
```

`dispatchers`： MQタイプのsinkの場合、dispatchersを使用してイベントディスパッチャを構成できます。`default`、`ts`、`rowid`、`table`の4つのディスパッチャがサポートされています。ディスパッチャのルールは以下の通りです：

- `default`： `table` モードでイベントをディスパッチします。
- `ts`： 行の変更のcommitTsを使用してハッシュ値を作成し、イベントをディスパッチします。
- `rowid`： 選択したHandleKey列の名前と値を使用してハッシュ値を作成し、イベントをディスパッチします。
- `table`： テーブルのスキーマ名とテーブル名を使用してハッシュ値を作成し、イベントをディスパッチします。

`matcher`： matcherの一致構文は、フィルター規則の構文と同じです。

`protocol`： MQタイプのsinkの場合、メッセージのプロトコル形式を指定できます。現在、以下のプロトコルがサポートされています： `canal-json`、 `open-protocol`、 `canal`、 `avro`、 `maxwell`。

### 例

以下のリクエストは、`test5` のIDと `sink_uri` が `blackhole://` のレプリケーションタスクを作成します。

{{< copyable "shell-regular" >}}

```shell
```shell
curl -X POST -H "'Content-type':'application/json'" http://127.0.0.1:8300/api/v1/changefeeds -d '{"changefeed_id":"test5","sink_uri":"blackhole://"}'
```

リクエストが成功した場合、`202 Accepted` が返されます。リクエストが失敗した場合、エラーメッセージとエラーコードが返されます。

## レプリケーションタスクの削除

このAPIは非同期インターフェースです。リクエストが成功した場合、`202 Accepted` が返されます。返される結果は、サーバーがコマンドを実行することに同意したことを意味するだけであり、コマンドが成功することを保証するものではありません。

### リクエストURI

`DELETE /api/v1/changefeeds/{changefeed_id}`

### パラメータの説明

#### パスパラメータ

| パラメータ名    | 説明                                       |
| :-------------- | :---------------------------------------- |
| `changefeed_id` | 削除するレプリケーションタスク（changefeed）のID。 |

### 例

次のリクエストは、IDが`test1`のレプリケーションタスクを削除します。

{{< copyable "shell-regular" >}}

```shell
curl -X DELETE http://127.0.0.1:8300/api/v1/changefeeds/test1
```

リクエストが成功した場合、`202 Accepted` が返されます。リクエストが失敗した場合、エラーメッセージとエラーコードが返されます。

## レプリケーション構成の更新

このAPIは非同期インターフェースです。リクエストが成功した場合、`202 Accepted` が返されます。返される結果は、サーバーがコマンドを実行することに同意したことを意味するだけであり、コマンドが成功することを保証するものではありません。

Changefeed構成を変更するには、次の手順に従います：`レプリケーションタスクを一時停止 -> 構成を変更する -> レプリケーションタスクを再開`。

### リクエストURI

`PUT /api/v1/changefeeds/{changefeed_id}`

### パラメータの説明

#### パスパラメータ

| パラメータ名    | 説明                                       |
| :-------------- | :---------------------------------------- |
| `changefeed_id` | 更新するレプリケーションタスク（changefeed）のID。 |

#### リクエストボディのパラメータ

現在、API経由で次の構成のみを変更できます。

| パラメータ名        | 説明                                             |
| :-------------------- | :-------------------------- --------------------------- |
| `target_ts`          | `UINT64` 型。ChangefeedのターゲットTSOを指定します。（オプション） |
| `sink_uri`           | `STRING` 型。レプリケーションタスクのダウンストリームアドレス。（オプション） |
| `filter_rules`       | `STRING` 型の配列。テーブルスキーマのフィルタリングルール。（オプション） |
| `ignore_txn_start_ts`| `UINT64` 型の配列。指定したstart_tsのトランザクションを無視します。（オプション） |
| `mounter_worker_num` | `INT` 型。マウンタースレッド数。（オプション） |
| `sink_config`        | Sinkの構成パラメータ。（オプション） |

上記のパラメータの意味については、[レプリケーションタスクを作成](#create-a-replication-task)のセクションと同様です。詳細については、そのセクションを参照してください。

### 例

次のリクエストは、IDが`test1`のレプリケーションタスクの`mounter_worker_num`を`32`に更新します。

{{< copyable "shell-regular" >}}

```shell
 curl -X PUT -H "'Content-type':'application/json'" http://127.0.0.1:8300/api/v1/changefeeds/test1 -d '{"mounter_worker_num":32}'
```

リクエストが成功した場合、`202 Accepted` が返されます。リクエストが失敗した場合、エラーメッセージとエラーコードが返されます。

## レプリケーションタスクリストのクエリ

このAPIは同期インターフェースです。リクエストが成功した場合、TiCDCクラスター内のすべてのノードの基本情報が返されます。

### リクエストURI

`GET /api/v1/changefeeds`

### パラメータの説明

#### クエリパラメータ

| パラメータ名 | 説明                                       |
| :------ | :---------------------------------------- ----- |
| `state` | このパラメータが指定されている場合、このステートのレプリケーション状態情報のみが返されます。（オプション） |

`state`の値オプションは、`all`、`normal`、`stopped`、`error`、`failed`、`finished`です。

このパラメータが指定されていない場合、通常、停止、または失敗した状態にあるレプリケーションタスクの基本情報がデフォルトで返されます。

### 例

次のリクエストは、状態が`normal`であるすべてのレプリケーションタスクの基本情報をクエリします。

{{< copyable "shell-regular" >}}

```shell
curl -X GET http://127.0.0.1:8300/api/v1/changefeeds?state=normal
```

```json
[
    {
        "id": "test1",
        "state": "normal",
        "checkpoint_tso": 426921294362574849,
        "checkpoint_time": "2021-08-10 14:04:54.242",
        "error": null
    },
    {
        "id": "test2",
        "state": "normal",
        "checkpoint_tso": 426921294362574849,
        "checkpoint_time": "2021-08-10 14:04:54.242",
        "error": null
    }
]
```

上記の返される結果のフィールドは、以下の通りです：

- id: レプリケーションタスクのID。
- state: レプリケーションタスクの現在の[状態](/ticdc/ticdc-changefeed-overview.md#changefeed-state-transfer)。
- checkpoint_tso: レプリケーションタスクの現在のチェックポイントのTSO表現。
- checkpoint_time: レプリケーションタスクの現在のチェックポイントのフォーマットされた時間表現。
- error: レプリケーションタスクのエラー情報。

## 特定のレプリケーションタスクのクエリ

このAPIは同期インターフェースです。リクエストが成功した場合、指定されたレプリケーションタスクの詳細情報が返されます。

### リクエストURI

`GET /api/v1/changefeeds/{changefeed_id}`

### パラメータの説明

#### パスパラメータ

| パラメータ名    | 説明                                       |
| :-------------- | :---------------------------------------- |
| `changefeed_id` | クエリするレプリケーションタスク（changefeed）のID。 |

### 例

次のリクエストは、IDが`test1`のレプリケーションタスクの詳細情報をクエリします。

{{< copyable "shell-regular" >}}

```shell
curl -X GET http://127.0.0.1:8300/api/v1/changefeeds/test1
```

```json
{
    "id": "test1",
    "sink_uri": "blackhole://",
    "create_time": "2021-08-10 11:41:30.642",
    "start_ts": 426919038970232833,
    "target_ts": 0,
    "checkpoint_tso": 426921014615867393,
    "checkpoint_time": "2021-08-10 13:47:07.093",
    "sort_engine": "unified",
    "state": "normal",
    "error": null,
    "error_history": null,
    "creator_version": "",
    "task_status": [
        {
            "capture_id": "d8924259-f52f-4dfb-97a9-c48d26395945",
            "table_ids": [
                63,
                65
            ],
            "table_operations": {}
        }
    ]
}
```

## レプリケーションタスクを一時停止

このAPIは非同期インターフェースです。リクエストが成功した場合、`202 Accepted` が返されます。返される結果は、サーバーがコマンドを実行することに同意したことを意味するだけであり、コマンドが成功することを保証するものではありません。

### リクエストURI

`POST /api/v1/changefeeds/{changefeed_id}/pause`

### パラメータの説明

#### パスパラメータ

| パラメータ名    | 説明                                       |
| :-------------- | :---------------------------------------- |
| `changefeed_id` | 一時停止するレプリケーションタスク（changefeed）のID。 |

### 例

次のリクエストは、IDが`test1`のレプリケーションタスクを一時停止します。

{{< copyable "shell-regular" >}}

```shell
curl -X POST http://127.0.0.1:8300/api/v1/changefeeds/test1/pause
```

リクエストが成功した場合、`202 Accepted` が返されます。リクエストが失敗した場合、エラーメッセージとエラーコードが返されます。

## レプリケーションタスクを再開

このAPIは非同期インターフェースです。リクエストが成功した場合、`202 Accepted` が返されます。返される結果は、サーバーがコマンドを実行することに同意したことを意味するだけであり、コマンドが成功することを保証するものではありません。

### リクエストURI

`POST /api/v1/changefeeds/{changefeed_id}/resume`

### パラメータの説明

#### パスパラメータ

| パラメータ名    | 説明                                       |
| :-------------- | :---------------------------------------- |
| `changefeed_id` | 再開するレプリケーションタスク（changefeed）のID。 |

### 例

次のリクエストは、IDが`test1`のレプリケーションタスクを再開します。

{{< copyable "shell-regular" >}}

```shell
curl -X POST http://127.0.0.1:8300/api/v1/changefeeds/test1/resume
```
```
```
Configuration Error
```