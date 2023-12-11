---
title: TiCDC OpenAPI v2
summary: クラスターの状態とデータレプリケーションを管理するための OpenAPI v2 インターフェイスの使用方法を学びます。
---

# TiCDC OpenAPI v2

<!-- markdownlint-disable MD024 -->

TiCDCは、TiCDCクラスターのクエリと操作のための OpenAPI 機能を提供しています。 OpenAPI 機能は [`cdc cli` ツール](/ticdc/ticdc-manage-changefeed.md) のサブセットです。

> **注記:**
>
> TiCDC OpenAPI v1は将来的に削除されます。TiCDC OpenAPI v2 の使用を推奨します。

TiCDC クラスターで次のメンテナンス操作を行うために、API を使用できます:

- [TiCDC ノードの状態情報を取得する](#get-the-status-information-of-a-ticdc-node)
- [TiCDC クラスターのヘルス状態を確認する](#check-the-health-status-of-a-ticdc-cluster)
- [レプリケーションタスクの作成](#create-a-replication-task)
- [レプリケーションタスクの削除](#remove-a-replication-task)
- [レプリケーション構成の更新](#update-the-replication-configuration)
- [レプリケーションタスクリストのクエリ](#query-the-replication-task-list)
- [特定のレプリケーションタスクのクエリ](#query-a-specific-replication-task)
- [レプリケーションタスクの一時停止](#pause-a-replication-task)
- [レプリケーションタスクの再開](#resume-a-replication-task)
- [レプリケーションサブタスクリストのクエリ](#query-the-replication-subtask-list)
- [特定のレプリケーションサブタスクのクエリ](#query-a-specific-replication-subtask)
- [TiCDCサービスプロセスリストのクエリ](#query-the-ticdc-service-process-list)
- [所有者ノードの削除](#evict-an-owner-node)
- [TiCDCサーバーのログレベルを動的に調整する](#dynamically-adjust-the-log-level-of-the-ticdc-server)

すべてのAPIのリクエスト本文と返される値はJSON形式です。成功したリクエストは`200 OK`メッセージを返します。次のセクションでAPIの具体的な使用方法について説明します。

以下の例では、TiCDCサーバーのリスニングIPアドレスは `127.0.0.1` であり、ポートは `8300` です。TiCDCにバインドされたIPアドレスとポートは、TiCDCサーバーを起動する際に `--addr=ip:port` を指定できます。

## APIエラーメッセージテンプレート

APIリクエストが送信された後、エラーが発生した場合、返されるエラーメッセージは以下の形式です:

```json
{
    "error_msg": "",
    "error_code": ""
}
```

上記のJSON出力では、`error_msg`がエラーメッセージを、`error_code`が対応するエラーコードを示します。

## APIリストインターフェイスの返却形式

APIリクエストがリソースのリストを返す場合（たとえば、すべての`Captures`のリストなど）、TiCDCの返却形式は以下のようになります:

```json
{
  "total": 2,
  "items": [
    {
      "id": "d2912e63-3349-447c-90ba-wwww",
      "is_owner": true,
      "address": "127.0.0.1:8300"
    },
    {
      "id": "d2912e63-3349-447c-90ba-xxxx",
      "is_owner": false,
      "address": "127.0.0.1:8302"
    }
  ]
}
```

上記の例では:

- `total`: リソースの総数を示します。
- `items`: このリクエストによって返されたすべてのリソースを含む配列です。配列のすべての要素は同じリソースです。

## TiCDCノードの状態情報を取得する

このAPIは同期インターフェイスです。リクエストが成功すると、対応するノードの状態情報が返されます。

### リクエストURI

`GET /api/v2/status`

### 例

以下のリクエストは、IPアドレスが `127.0.0.1` でポート番号が `8300` の TiCDCノードの状態情報を取得します。

```shell
curl -X GET http://127.0.0.1:8300/api/v2/status
```

```json
{
  "version": "v7.4.0",
  "git_hash": "10413bded1bdb2850aa6d7b94eb375102e9c44dc",
  "id": "d2912e63-3349-447c-90ba-72a4e04b5e9e",
  "pid": 1447,
  "is_owner": true,
  "liveness": 0
}
```

上記の出力のパラメータは以下の通りです:

- `version`: TiCDCの現在のバージョン番号。
- `git_hash`: Gitのハッシュ値。
- `id`: ノードのキャプチャID。
- `pid`: ノードのキャプチャプロセスID（PID）。
- `is_owner`: ノードが所有者であるかどうかを示します。
- `liveness`: このノードが稼働しているかどうか。`0` は正常を示し、`1` はノードが `graceful shutdown` 状態にあることを示します。

## TiCDCクラスターのヘルス状態を確認する

このAPIは同期インタフェースです。クラスタが正常であれば、`200 OK`が返されます。

### リクエストURI

`GET /api/v2/health`

### 例

```shell
curl -X GET http://127.0.0.1:8300/api/v2/health
```

クラスタが正常であれば、応答は `200 OK` および空のJSONオブジェクトです:

```json
{}
```

クラスタが正常でない場合、応答はエラーメッセージを含むJSONオブジェクトです。

## レプリケーションタスクの作成

このインターフェイスはレプリケーションタスクをTiCDCに送信するために使用されます。リクエストが成功すると、`200 OK`が返されます。返された結果は、サーバーがコマンドを実行することに同意したことを意味しますが、コマンドが成功裏に実行されることを保証しません。

### リクエストURI

`POST /api/v2/changefeeds`

### パラメータの説明

```json
{
  "changefeed_id": "string",
  "replica_config": {
    // ... (省略)
  },
  "sink_uri": "string",
  "start_ts": 0,
  "target_ts": 0
}
```

パラメータは以下のように説明されます:

| パラメータ名    | 説明 |
| :------------------------ | :----------------------------------------------------- |
| `changefeed_id` | `STRING` 型。レプリケーションタスクのID。 (オプション) |
| `replica_config` | レプリケーションタスクの構成パラメータ。 (オプション) |
| **`sink_uri`** | `STRING` 型。レプリケーションタスクのダウンストリームアドレス。 (**必須**) |
| `start_ts` | `UINT64` 型。changefeedの開始TSOを指定します。TiCDCクラスターはこのTSOからデータを取得し始めます。デフォルト値は現在の時刻です。 (オプション) |
| `target_ts` | `UINT64`型。 changefeedのターゲットTSOを指定します。TiCDCクラスターはこのTSOに到達するとデータの取得を停止します。デフォルト値は空で、TiCDCは自動的に停止しません。（オプション） |

`changefeed_id`、`start_ts`、`target_ts`、および`sink_uri`の意味と形式は、[cdc cliを使用してレプリケーションタスクを作成する](/ticdc/ticdc-manage-changefeed.md#create-a-replication-task) ドキュメントで説明されているものと同じです。これらのパラメータの詳細な説明については、該当のドキュメントを参照してください。 `sink_uri`で証明書のパスを指定する場合は、対応するTiCDCサーバーに対応する証明書をアップロードしていることを確認してください。

`replica_config`パラメータの説明は次のとおりです。

| パラメータ名 | 説明 |
| :------------------------ | :----------------------------------------------------- |
| `bdr_mode`                | `BOOLEAN`型。[双方向レプリケーション](/ticdc/ticdc-bidirectional-replication.md)を有効にするかどうかを決定します。デフォルト値は`false`です。（オプション） |
| `case_sensitive`          | `BOOLEAN`型。テーブル名のフィルタリングに大文字と小文字を区別するかどうかを指定します。v7.5.0からデフォルト値が`true`から`false`に変更されています。（オプション） |
| `check_gc_safe_point`     | `BOOLEAN`型。レプリケーションタスクの開始時間がGC時間よりも前かどうかを確認するかどうかを指定します。デフォルト値は`true`です。（オプション） |
| `consistent`              | Redoログの構成パラメータ（オプション） |
| `enable_sync_point`       | `BOOLEAN`型。`sync point`を有効にするかどうかを指定します。（オプション） |
| `filter`                  | `filter`の構成パラメータ（オプション） |
| `force_replicate`         | `BOOLEAN`型。デフォルト値は`false`です。`true`に設定すると、レプリケーションタスクはユニークインデックスを持たないテーブルを強制的にレプリケーションします。（オプション） |
| `ignore_ineligible_table` | `BOOLEAN`型。デフォルト値は`false`です。`true`に設定すると、レプリケーションタスクはレプリケートできないテーブルを無視します。（オプション） |
| `memory_quota`            | `UINT64`型。レプリケーションタスクのメモリ割り当て値です。（オプション） |
| `mounter`                 | `mounter`の構成パラメータ（オプション） |
| `sink`                    | `sink`の構成パラメータ（オプション） |
| `sync_point_interval`     | `STRING`型。返される値は`UINT64`型のナノ秒単位の時間です。`sync point`機能が有効な場合、このパラメータはSyncpointがアップストリームとダウンストリームのスナップショットを整列させる間隔を指定します。デフォルト値は`10m`で、最小値は`30s`です。（オプション） |
| `sync_point_retention`    | `STRING`型。返される値は`UINT64`型のナノ秒単位の時間です。`sync point`機能が有効な場合、このパラメータはSyncpointによってダウンストリームテーブルでデータが保持される期間を指定します。この期間を超過すると、データがクリーンアップされます。デフォルト値は`24h`です。（オプション） |

`consistent`パラメータは次のように記述されます。

| パラメータ名 | 説明 |
|:-----------------|:---------------------------------------|
| `flush_interval` | `UINT64`型。redoログファイルをフラッシュする間隔（オプション） |
| `level`          | `STRING`型。レプリケートされたデータの整合性レベル（オプション） |
| `max_log_size`   | `UINT64`型。redoログの最大値（オプション） |
| `storage`        | `STRING`型。ストレージの宛先アドレス（オプション） |

`filter`パラメータは次のように記述されます。

| パラメータ名 | 説明 |
|:-----------------|:---------------------------------------|
| `do_dbs`              | `STRING ARRAY`型。レプリケートするデータベース（オプション） |
| `do_tables`           | レプリケートするテーブル（オプション） |
| `ignore_dbs`          | `STRING ARRAY`型。無視するデータベース（オプション） |
| `ignore_tables`       | 無視するテーブル（オプション） |
| `event_filters`       | イベントをフィルタリングする構成（オプション） |
| `ignore_txn_...
| `include_commit_ts` | `BOOLEAN`型。CSVの行にcommit-tsを含めるかどうか。デフォルト値は`false`です。 |
| `null` | `STRING`型。CSVの列がnullの場合に表示される文字。デフォルト値は`\N`です。 |
| `quote` | `STRING`型。CSVファイル内のフィールドを囲むために使用される引用文字。値が空の場合、引用符は使用されません。デフォルト値は`"`です。 |

`sink.dispatchers`: MQタイプのシンクのためのパラメータで、イベントディスパッチャを構成するために使用できます。次のディスパッチャがサポートされています：`default`、`ts`、`rowid`、`table`。ディスパッチャのルールは次のとおりです：

- `default`: `table`モードでイベントをディスパッチします。
- `ts`: 行の変更のcommitTsを使用してハッシュ値を作成し、イベントをディスパッチします。
- `rowid`: 選択されたHandleKey列の名前と値を使用してハッシュ値を作成し、イベントをディスパッチします。
- `table`: テーブルのスキーマ名とテーブル名を使用してハッシュ値を作成し、イベントをディスパッチします。

`sink.dispatchers`は配列です。パラメータは次のとおりです：

| パラメータ名 | 説明 |
|:-----------------|:---------------------------------------|
| `matcher`    | `STRING ARRAY`型。フィルタルールと同じマッチング構文を持っています。 |
| `partition` | `STRING`型。イベントをディスパッチするターゲットパーティション。 |
| `topic`     | `STRING`型。イベントをディスパッチするターゲットトピック。 |

### 例

次のリクエストは、IDが`test5`で、`sink_uri`が`blackhome://`のレプリケーションタスクを作成します。

```shell
curl -X POST -H "'Content-type':'application/json'" http://127.0.0.1:8300/api/v2/changefeeds -d '{"changefeed_id":"test5","sink_uri":"blackhole://"}'
```

リクエストが成功した場合、`200 OK`が返されます。リクエストが失敗した場合、エラーメッセージとエラーコードが返されます。

### レスポンスボディの形式

```json
{
  "admin_job_type": 0,
  "checkpoint_time": "string",
  "checkpoint_ts": 0,
  "config": {
    "bdr_mode": true,
    "case_sensitive": false,
    "check_gc_safe_point": true,
    "consistent": {
      "flush_interval": 0,
      "level": "string",
      "max_log_size": 0,
      "storage": "string"
    },
    "enable_old_value": true,
    "enable_sync_point": true,
    "filter": {
      "do_dbs": [
        "string"
      ],
      "do_tables": [
        {
          "database_name": "string",
          "table_name": "string"
        }
      ],
      "event_filters": [
        {
          "ignore_delete_value_expr": "string",
          "ignore_event": [
            "string"
          ],
          "ignore_insert_value_expr": "string",
          "ignore_sql": [
            "string"
          ],
          "ignore_update_new_value_expr": "string",
          "ignore_update_old_value_expr": "string",
          "matcher": [
            "string"
          ]
        }
      ],
      "ignore_dbs": [
        "string"
      ],
      "ignore_tables": [
        {
          "database_name": "string",
          "table_name": "string"
        }
      ],
      "ignore_txn_start_ts": [
        0
      ],
      "rules": [
        "string"
      ]
    },
    "force_replicate": true,
    "ignore_ineligible_table": true,
    "memory_quota": 0,
    "mounter": {
      "worker_num": 0
    },
    "sink": {
      "column_selectors": [
        {
          "columns": [
            "string"
          ],
          "matcher": [
            "string"
          ]
        }
      ],
      "csv": {
        "delimiter": "string",
        "include_commit_ts": true,
        "null": "string",
        "quote": "string"
      },
      "date_separator": "string",
      "dispatchers": [
        {
          "matcher": [
            "string"
          ],
          "partition": "string",
          "topic": "string"
        }
      ],
      "enable_partition_separator": true,
      "encoder_concurrency": 0,
      "protocol": "string",
      "schema_registry": "string",
      "terminator": "string",
      "transaction_atomicity": "string"
    },
    "sync_point_interval": "string",
    "sync_point_retention": "string"
  },
  "create_time": "string",
  "creator_version": "string",
  "error": {
    "addr": "string",
    "code": "string",
    "message": "string"
  },
  "id": "string",
  "resolved_ts": 0,
  "sink_uri": "string",
  "start_ts": 0,
  "state": "string",
  "target_ts": 0,
  "task_status": [
    {
      "capture_id": "string",
      "table_ids": [
        0
      ]
    }
  ]
}
```

パラメータは次のように説明されています：

| パラメータ名 | 説明 |
|:-----------------|:---------------------------------------|
| `admin_job_type`  | `INTEGER`型。管理ジョブのタイプ。                 |
| `checkpoint_time` | `STRING`型。レプリケーションタスクの現在のチェックポイントのフォーマットされた時刻。                  |
| `checkpoint_ts`   | `STRING`型。レプリケーションタスクの現在のチェックポイントのTSO。    |
| `config`          | レプリケーションタスクの構成。意味は、レプリケーションタスクの作成時の`replica_config`構成と同じです。       |
| `create_time`     | `STRING`型。レプリケーションタスクが作成された時刻。                          |
| `creator_version` | `STRING`型。レプリケーションタスクが作成されたときのTiCDCのバージョン。         |
| `error`           | レプリケーションタスクのエラー。                      |
| `id`              | `STRING`型。レプリケーションタスクのID。                |
| `resolved_ts`     | `UINT64`型。レプリケーションタスクの解決済みTS。    |
| `sink_uri`        | `STRING`型。レプリケーションタスクのシンクURI。                                     |
| `start_ts`        | `UINT64`型。レプリケーションタスクの開始TS。                                      |
| `state`           | `STRING`型。レプリケーションタスクのステータス。`normal`、`stopped`、`error`、`failed`、`finished`のいずれかです。 |
| `target_ts`       | `UINT64`型。レプリケーションタスクのターゲットTS。                                    |
| `task_status`     | レプリケーションタスクのディスパッチの詳細な状態。 |

`task_status`パラメータは次のように説明されています：

| パラメータ名 | 説明 |
|:-----------------|:---------------------------------------|
| `capture_id` | `STRING`型。キャプチャID。                    |
| `table_ids`  | `UINT64 ARRAY`型。このキャプチャでレプリケートされているテーブルのID。 |

`error`パラメータは次のように説明されています：

| パラメータ名 | 説明 |
|:-----------------|:---------------------------------------|
| `addr` | `STRING`型。キャプチャアドレス。 |
| `code` | `STRING`型。エラーコード。          |
| `message` | `STRING`型。エラーの詳細。      |

## レプリケーションタスクの削除

このAPIは、レプリケーションタスクを削除するための冪等なインタフェースです（つまり、初期の適用を超えて結果を変更することなく複数回適用できます）。リクエストが成功した場合、`200 OK`が返されます。返された結果は、サーバーがコマンドを実行することに同意することを意味するだけであり、コマンドが正常に実行されることを保証するものではありません。

### リクエストURI

`DELETE /api/v2/changefeeds/{changefeed_id}`

### パラメータの説明

#### パスパラメータ

| パラメータ名 | 説明 |
| :-------------- | :----------------------------------- |
| `changefeed_id` | 削除するレプリケーションタスク（changefeed）のID。 |

### 例

次のリクエストは、IDが`test1`のレプリケーションタスクを削除します。

```shell
curl -X DELETE http://127.0.0.1:8300/api/v2/changefeeds/test1
```

リクエストが成功した場合、`200 OK`が返されます。リクエストが失敗した場合、エラーメッセージとエラーコードが返されます。

## レプリケーション構成の更新

このAPIは、レプリケーションタスクを更新するために使用されます。リクエストが成功した場合、`200 OK`が返されます。返された結果は、サーバーがコマンドを実行することに同意することを意味するだけであり、コマンドが正常に実行されることを保証するものではありません。

チェンジフィード構成を変更するには、`レプリケーションタスクを一時停止 -> 構成を変更 -> レプリケーションタスクを再開`という手順に従います。

### リクエストURI

`PUT /api/v2/changefeeds/{changefeed_id}`

### パラメータの説明

#### パスパラメータ

| パラメータ名 | 説明 |
| :-------------- | :----------------------------------- |
| `changefeed_id` | 更新するレプリケーションタスク（changefeed）のID。 |

#### リクエストボディのパラメータ

```json
{
  "replica_config": {
    "bdr_mode": true,
    "case_sensitive": false,
```json
{
  "check_gc_safe_point": true,
  "consistent": {
    "flush_interval": 0,
    "level": "string",
    "max_log_size": 0,
    "storage": "string"
  },
  "enable_old_value": true,
  "enable_sync_point": true,
  "filter": {
    "do_dbs": [
      "string"
    ],
    "do_tables": [
      {
        "database_name": "string",
        "table_name": "string"
      }
    ],
    "event_filters": [
      {
        "ignore_delete_value_expr": "string",
        "ignore_event": [
          "string"
        ],
        "ignore_insert_value_expr": "string",
        "ignore_sql": [
          "string"
        ],
        "ignore_update_new_value_expr": "string",
        "ignore_update_old_value_expr": "string",
        "matcher": [
          "string"
        ]
      }
    ],
    "ignore_dbs": [
      "string"
    ],
    "ignore_tables": [
      {
        "database_name": "string",
        "table_name": "string"
      }
    ],
    "ignore_txn_start_ts": [
      0
    ],
    "rules": [
      "string"
    ]
  },
  "force_replicate": true,
  "ignore_ineligible_table": true,
  "memory_quota": 0,
  "mounter": {
    "worker_num": 0
  },
  "sink": {
    "column_selectors": [
      {
        "columns": [
          "string"
        ],
        "matcher": [
          "string"
        ]
      }
    ],
    "csv": {
      "delimiter": "string",
      "include_commit_ts": true,
      "null": "string",
      "quote": "string"
    },
    "date_separator": "string",
    "dispatchers": [
      {
        "matcher": [
          "string"
        ],
        "partition": "string",
        "topic": "string"
      }
    ],
    "enable_partition_separator": true,
    "encoder_concurrency": 0,
    "protocol": "string",
    "schema_registry": "string",
    "terminator": "string",
    "transaction_atomicity": "string"
  },
  "sync_point_interval": "string",
  "sync_point_retention": "string"
}
```

現在は、APIを介して以下の設定のみ変更できます。

| パラメータ名 | 説明 |
| :-------------------- | :----------------------------------------------------- |
| `target_ts` | `UINT64` タイプ。changefeedの対象TSOを指定します（オプション） |
| `sink_uri` | `STRING` タイプ。レプリケーションタスクの下流アドレス（オプション） |
| `replica_config` | sinkの構成パラメータ。完全である必要があります（オプション） |

上記のパラメータの意味は、[レプリケーションタスクを作成する](#create-a-replication-task)セクションと同じです。詳細については、そのセクションを参照してください。

### 例

次のリクエストは、IDが`test1`のレプリケーションタスクの`target_ts`を`32`に更新します。

```shell
 curl -X PUT -H "'Content-type':'application/json'" http://127.0.0.1:8300/api/v2/changefeeds/test1 -d '{"target_ts":32}'
```

リクエストが成功した場合は、`200 OK`が返されます。リクエストが失敗した場合は、エラーメッセージとエラーコードが返されます。JSON応答ボディの意味は、[レプリケーションタスクを作成する](#create-a-replication-task)セクションと同じです。詳細については、そのセクションを参照してください。

## レプリケーションタスク一覧をクエリする

このAPIは同期インタフェースです。リクエストが成功した場合、TiCDCクラスターのすべてのレプリケーションタスク（changefeed）の基本情報が返されます。

### リクエストURI

`GET /api/v2/changefeeds`

### パラメータの説明

#### クエリパラメータ

| パラメータ名 | 説明 |
| :------ | :--------------------------------------------- |
| `state` | このパラメータが指定されている場合、指定された状態のレプリケーションタスクの情報が返されます（オプション） |

`state`の値オプションは、`all`、`normal`、`stopped`、`error`、`failed`、`finished`です。

このパラメータが指定されていない場合、`normal`、`stopped`、`failed`状態のレプリケーションタスクの基本情報がデフォルトで返されます。

### 例

次のリクエストは、`normal`状態のすべてのレプリケーションタスクの基本情報をクエリします。

```shell
curl -X GET http://127.0.0.1:8300/api/v2/changefeeds?state=normal
```

```json
{
  "total": 2,
  "items": [
    {
      "id": "test",
      "state": "normal",
      "checkpoint_tso": 439749918821711874,
      "checkpoint_time": "2023-02-27 23:46:52.888",
      "error": null
    },
    {
      "id": "test2",
      "state": "normal",
      "checkpoint_tso": 439749918821711874,
      "checkpoint_time": "2023-02-27 23:46:52.888",
      "error": null
    }
  ]
}
```

上記の応答結果のパラメータは以下のように説明されています：

- `id`: レプリケーションタスクのID。
- `state`: レプリケーションタスクの現在の[state](/ticdc/ticdc-changefeed-overview.md#changefeed-state-transfer)。
- `checkpoint_tso`: レプリケーションタスクの現在のチェックポイントのTSO。
- `checkpoint_time`: レプリケーションタスクの現在のチェックポイントのフォーマットされた時間。
- `error`: レプリケーションタスクのエラー情報。

## 特定のレプリケーションタスクをクエリする

このAPIは同期インタフェースです。リクエストが成功した場合、指定されたレプリケーションタスク（changefeed）の詳細情報が返されます。

### リクエストURI

`GET /api/v2/changefeeds/{changefeed_id}`

### パラメータの説明

#### パスパラメータ

| パラメータ名 | 説明 |
| :-------------- | :----------------------------------- |
| `changefeed_id` | クエリするレプリケーションタスク（changefeed）のID |

### 例

次のリクエストは、IDが`test1`のレプリケーションタスクの詳細情報をクエリします。

```shell
curl -X GET http://127.0.0.1:8300/api/v2/changefeeds/test1
```

JSON応答ボディの意味は、[レプリケーションタスクを作成する](#create-a-replication-task)セクションと同じです。詳細については、そのセクションを参照してください。

## レプリケーションタスクを一時停止する

このAPIは、レプリケーションタスクを一時停止します。リクエストが成功した場合、 `200 OK`が返されます。返される結果は、サーバーがコマンドを実行することに同意するだけであり、コマンドが成功して実行されることを保証するものではありません。

### リクエストURI

`POST /api/v2/changefeeds/{changefeed_id}/pause`

### パラメータの説明

#### パスパラメータ

| パラメータ名 | 説明 |
| :-------------- | :----------------------------------- |
| `changefeed_id` | 一時停止するレプリケーションタスク（changefeed）のID |

### 例

次のリクエストは、IDが`test1`のレプリケーションタスクを一時停止します。

```shell
curl -X POST http://127.0.0.1:8300/api/v2/changefeeds/test1/pause
```

リクエストが成功した場合、 `200 OK`が返されます。リクエストが失敗した場合は、エラーメッセージとエラーコードが返されます。

## レプリケーションタスクを再開する

このAPIは、レプリケーションタスクを再開します。リクエストが成功した場合、`200 OK`が返されます。返される結果は、サーバーがコマンドを実行することに同意するだけであり、コマンドが成功して実行されることを保証するものではありません。

### リクエストURI

`POST /api/v2/changefeeds/{changefeed_id}/resume`

### パラメータの説明

#### パスパラメータ

| パラメータ名 | 説明 |
| :-------------- | :----------------------------------- |
| `changefeed_id` | 再開するレプリケーションタスク（changefeed）のID |

#### リクエストボディのパラメータ

```json
{
  "overwrite_checkpoint_ts": 0
}
```

| パラメータ名 | 説明 |
| :-------------- | :----------------------------------- |
| `overwrite_checkpoint_ts` | `UINT64` タイプ。レプリケーションタスク（changefeed）を再開する際にチェックポイントTSOを再割り当てします。 |

### 例

次のリクエストは、IDが`test1`のレプリケーションタスクを再開します。

```shell
curl -X POST http://127.0.0.1:8300/api/v2/changefeeds/test1/resume -d '{}'
```

リクエストが成功した場合、`200 OK`が返されます。リクエストが失敗した場合は、エラーメッセージとエラーコードが返されます。

## レプリケーションサブタスク一覧をクエリする

このAPIは同期インタフェースです。リクエストが成功した場合、すべてのレプリケーションサブタスク（`processor`）の基本情報が返されます。

### リクエストURI

`GET /api/v2/processors`

### 例

```shell
curl -X GET http://127.0.0.1:8300/api/v2/processors
```
```
```json
{
  "total": 3,
  "items": [
    {
      "changefeed_id": "test2",
      "capture_id": "d2912e63-3349-447c-90ba-72a4e04b5e9e"
    },
    {
      "changefeed_id": "test1",
      "capture_id": "d2912e63-3349-447c-90ba-72a4e04b5e9e"
    },
    {
      "changefeed_id": "test",
      "capture_id": "d2912e63-3349-447c-90ba-72a4e04b5e9e"
    }
  ]
}
```

パラメータは以下のように記載されています：

- `changefeed_id`: changefeed ID。
- `capture_id`: キャプチャ ID。

## 特定の複製サブタスクをクエリする

このAPIは同期インターフェースです。リクエストが成功すると、指定された複製サブタスク（`processor`）の詳細情報が返されます。

### リクエストURI

`GET /api/v2/processors/{changefeed_id}/{capture_id}`

### パラメータの説明

#### パスパラメータ

| パラメータ名 | 説明 |
| :-------------- | :----------------------------------- |
| `changefeed_id` | クエリする複製サブタスクのchangefeed ID。 |
| `capture_id` | クエリする複製サブタスクのcapture ID。 |

### 例

以下のリクエストは、`changefeed_id` が `test` で `capture_id` が `561c3784-77f0-4863-ad52-65a3436db6af` のサブタスクの詳細情報をクエリします。サブタスクは `changefeed_id` と `capture_id` で特定できます。

```shell
curl -X GET http://127.0.0.1:8300/api/v2/processors/test/561c3784-77f0-4863-ad52-65a3436db6af
```

```json
{
  "table_ids": [
    80
  ]
}
```

パラメータは以下のように記載されています：

- `table_ids`: このキャプチャで複製するテーブルID。

## TiCDCサービスプロセスリストをクエリする

このAPIは同期インターフェースです。リクエストが成功すると、すべての複製プロセス（`capture`）の基本情報が返されます。

### リクエストURI

`GET /api/v2/captures`

### 例

```shell
curl -X GET http://127.0.0.1:8300/api/v2/captures
```

```json
{
  "total": 1,
  "items": [
    {
      "id": "d2912e63-3349-447c-90ba-72a4e04b5e9e",
      "is_owner": true,
      "address": "127.0.0.1:8300"
    }
  ]
}
```

パラメータは以下のように記載されています：

- `id`: キャプチャ ID。
- `is_owner`: キャプチャがオーナーかどうか。
- `address`: キャプチャのアドレス。

## オーナーノードを追放する

このAPIは非同期インターフェースです。リクエストが成功すると、 `200 OK` が返されます。返された結果は、サーバーがコマンドを実行することに同意したことを意味するだけであり、コマンドが正常に実行されることは保証されません。

### リクエストURI

`POST /api/v2/owner/resign`

### 例

以下のリクエストは、TiCDCの現在のオーナーノードを追放し、新しいオーナーノードを生成するために新しい選挙をトリガーします。

```shell
curl -X POST http://127.0.0.1:8300/api/v2/owner/resign
```

リクエストが成功すると、 `200 OK` が返されます。リクエストが失敗すると、エラーメッセージとエラーコードが返されます。

## TiCDCサーバーのログレベルを動的に調整する

このAPIは同期インターフェースです。リクエストが成功すると、 `200 OK` が返されます。

### リクエストURI

`POST /api/v2/log`

### リクエストパラメータ

#### リクエストボディのパラメータ

| パラメータ名 | 説明 |
| :---------- | :----------------- |
| `log_level` | 設定したいログレベル。 |

`log_level` は [zapが提供するログレベル](https://godoc.org/go.uber.org/zap#UnmarshalText) をサポートしています: "debug", "info", "warn", "error", "dpanic", "panic"、および"fatal"。

### 例

```shell
curl -X POST -H "'Content-type':'application/json'" http://127.0.0.1:8300/api/v2/log -d '{"log_level":"debug"}'
```

リクエストが成功すると、 `200 OK` が返されます。リクエストが失敗すると、エラーメッセージとエラーコードが返されます。