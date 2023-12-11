---
title: チェンジフィードの管理
summary: TiCDCのチェンジフィードを管理する方法について学びます。
aliases: ['/tidb/dev/manage-ticdc']
---

# チェンジフィードの管理

本書では、TiCDCコマンドラインツール `cdc cli` を使用して、TiCDCチェンジフィードを作成および管理する方法について説明します。また、TiCDCのHTTPインターフェイスを介してもチェンジフィードを管理できます。詳細については、[TiCDC OpenAPI](/ticdc/ticdc-open-api.md) を参照してください。

## 複製タスクの作成

以下のコマンドを実行して複製タスクを作成します。

```shell
cdc cli changefeed create --server=http://10.0.10.25:8300 --sink-uri="mysql://root:123456@127.0.0.1:3306/" --changefeed-id="simple-replication-task"
```

```shell
チェンジフィードが正常に作成されました！
ID: simple-replication-task
Info: {"upstream_id":7178706266519722477,"namespace":"default","id":"simple-replication-task","sink_uri":"mysql://root:xxxxx@127.0.0.1:4000/?time-zone=","create_time":"2023-11-28T15:05:46.679218+08:00","start_ts":438156275634929669,"engine":"unified","config":{"case_sensitive":false,"enable_old_value":true,"force_replicate":false,"ignore_ineligible_table":false,"check_gc_safe_point":true,"enable_sync_point":true,"bdr_mode":false,"sync_point_interval":30000000000,"sync_point_retention":3600000000000,"filter":{"rules":["test.*"],"event_filters":null},"mounter":{"worker_num":16},"sink":{"protocol":"","schema_registry":"","csv":{"delimiter":",","quote":"\"","null":"\\N","include_commit_ts":false},"column_selectors":null,"transaction_atomicity":"none","encoder_concurrency":16,"terminator":"\r\n","date_separator":"none","enable_partition_separator":false},"consistent":{"level":"none","max_log_size":64,"flush_interval":2000,"storage":""}},"state":"normal","creator_version":"v7.5.0"}
```

## 複製タスクリストのクエリ

以下のコマンドを実行して複製タスクリストをクエリします。

```shell
cdc cli changefeed list --server=http://10.0.10.25:8300
```

```shell
[{
    "id": "simple-replication-task",
    "summary": {
      "state": "normal",
      "tso": 417886179132964865,
      "checkpoint": "2020-07-07 16:07:44.881",
      "error": null
    }
}]
```

- `checkpoint` は、TiCDCがこの時点より前にデータをダウンストリームにレプリケーションしたことを示しています。
- `state` は、複製タスクの状態を示しています。
    - `normal`: 複製タスクは正常に実行されています。
    - `stopped`: 複製タスクは停止されています（手動で一時停止）。
    - `error`: 複製タスクはエラーにより停止されています。
    - `removed`: 複製タスクは削除されています。この状態のタスクは `--all` オプションを指定した場合にのみ表示されます。このオプションを指定しない場合は、 `changefeed query` コマンドを実行してこれらのタスクを表示します。
    - `finished`: 複製タスクは完了しています（データが `target-ts` にレプリケートされています）。この状態のタスクは `--all` オプションを指定した場合にのみ表示されます。このオプションを指定しない場合は、 `changefeed query` コマンドを実行してこれらのタスクを表示します。

## 特定の複製タスクのクエリ

特定の複製タスクをクエリするには、 `changefeed query` コマンドを実行します。クエリ結果には、タスク情報とタスク状態が含まれます。クエリ結果を簡略化し、基本的な複製状態とチェックポイント情報のみを含めるには、 `--simple` または `-s` 引数を指定できます。この引数を指定しない場合は、詳細なタスク構成、複製状態、複製テーブル情報が出力されます。

```shell
cdc cli changefeed query -s --server=http://10.0.10.25:8300 --changefeed-id=simple-replication-task
```

```shell
{
 "state": "normal",
 "tso": 419035700154597378,
 "checkpoint": "2020-08-27 10:12:19.579",
 "error": null
}
```

前述のコマンドと結果では以下の内容が含まれます:

+ `state`: 現在のチェンジフィードの複製状態です。各状態は `changefeed list` の状態と一致する必要があります。
+ `tso`: 現在のチェンジフィード内でダウンストリームに正常にレプリケートされた最大トランザクションTSOを表します。
+ `checkpoint`: 現在のチェンジフィード内でダウンストリームに正常にレプリケートされた最大トランザクションTSOに対応する時間を表します。
+ `error`: 現在のチェンジフィードでエラーが発生したかどうかを記録します。

```shell
cdc cli changefeed query --server=http://10.0.10.25:8300 --changefeed-id=simple-replication-task
```

```shell
{
  "info": {
    "sink-uri": "mysql://127.0.0.1:3306/?max-txn-row=20\u0026worker-number=4",
    "opts": {},
    "create-time": "2020-08-27T10:33:41.687983832+08:00",
    "start-ts": 419036036249681921,
    "target-ts": 0,
    "admin-job-type": 0,
    "sort-engine": "unified",
    "sort-dir": ".",
    "config": {
      "case-sensitive": false,
      "filter": {
        "rules": [
          "*.*"
        ],
        "ignore-txn-start-ts": null,
        "ddl-allow-list": null
      },
      "mounter": {
        "worker-num": 16
      },
      "sink": {
        "dispatchers": null,
      },
      "scheduler": {
        "type": "table-number",
        "polling-time": -1
      }
    },
    "state": "normal",
    "history": null,
    "error": null
  },
  "status": {
    "resolved-ts": 419036036249681921,
    "checkpoint-ts": 419036036249681921,
    "admin-job-type": 0
  },
  "count": 0,
  "task-status": [
    {
      "capture-id": "97173367-75dc-490c-ae2d-4e990f90da0f",
      "status": {
        "tables": {
          "47": {
            "start-ts": 419036036249681921
          }
        },
        "operation": null,
        "admin-job-type": 0
      }
    }
  ]
}
```

前述のコマンドと結果では以下の内容が含まれます:

- `info`: クエリされたチェンジフィードの複製構成です。
- `status`: クエリされたチェンジフィードの複製状態です。
    - `resolved-ts`: 現在のチェンジフィード内の最大トランザクションTSです。このTSはTiKVからTiCDCに正常に送信されていることに注意してください。
    - `checkpoint-ts`: 現在のチェンジフィード内の最大トランザクションTSです。このTSは正常にダウンストリームに書き込まれていることに注意してください。
    - `admin-job-type`: チェンジフィードのステータス:
        - `0`: 状態は正常です。
        - `1`: タスクが一時停止しています。タスクが一時停止されると、すべての複製された `processor` が終了します。タスクの構成および複製ステータスは保持されるため、 `checkpiont-ts` からタスクを再開できます。
        - `2`: タスクが再開されています。チェンジフィードのタスクは `checkpoint-ts` から再開されます。
        - `3`: タスクが削除されました。タスクが削除されると、すべての複製された `processor` が終了し、複製タスクの構成情報はクリアされます。後でのクエリのために複製ステータスだけが保持されます。
- `task-status` は、クエリされたチェンジフィードの各複製サブタスクの状態を示します。

## 複製タスクを一時停止

以下のコマンドを実行して複製タスクを一時停止します。

```shell
cdc cli changefeed pause --server=http://10.0.10.25:8300 --changefeed-id simple-replication-task
```

前述のコマンドでは:

- `--changefeed-id=uuid` は、一時停止したい複製タスクに対応するチェンジフィードのIDを表します。

## 複製タスクを再開

以下のコマンドを実行して一時停止した複製タスクを再開します。

```shell
cdc cli changefeed resume --server=http://10.0.10.25:8300 --changefeed-id simple-replication-task
```

- `--changefeed-id=uuid` は、再開したい複製タスクに対応するチェンジフィードのIDを表します。
- `--overwrite-checkpoint-ts`: v6.2.0以降では、再開する複製タスクの開始TSOを指定できます。TiCDCは指定したTSOからデータを取得します。この引数は `now` または特定のTSO（例: 434873584621453313）を受け入れます。指定したTSOは（GC安全ポイント、CurrentTSO]の範囲内である必要があります。この引数が指定されていない場合、TiCDCはデフォルトで現在の `checkpoint-ts` からデータをレプリケートします。
- `--no-confirm`: レプリケーションが再開される際、関連情報を確認する必要はありません。デフォルトは `false` です。

> **注意:**
>
> - `--overwrite-checkpoint-ts` (`t2`) で指定された TSO がチェンジフィードの現在のチェックポイント TSO (`t1`) よりも大きい場合、`t1` と `t2` の間のデータはダウンストリームにレプリケートされません。これによりデータが失われます。`cdc cli changefeed query` を実行することで `t1` を取得できます。
> - `--overwrite-checkpoint-ts` (`t2`) で指定された TSO がチェンジフィードの現在のチェックポイント TSO (`t1`) よりも小さい場合、TiCDC は古いタイムポイント (`t2`) からデータを抽出します。これによりデータの重複が発生する可能性があります（たとえば、ダウンストリームが MQ シンクの場合）。

## レプリケーションタスクの削除

次のコマンドを実行してレプリケーションタスクを削除します:

```shell
cdc cli changefeed remove --server=http://10.0.10.25:8300 --changefeed-id simple-replication-task
```

上記のコマンドでは:

- `--changefeed-id=uuid` は削除したいレプリケーションタスクに対応するチェンジフィードのIDを表します。

## タスク構成の更新

TiCDC はレプリケーションタスクの構成変更をサポートしています（動的には変更できません）。チェンジフィード構成を変更するには、タスクを一時停止し、構成を変更した後にタスクを再開します。

```shell
cdc cli changefeed pause -c test-cf --server=http://10.0.10.25:8300
cdc cli changefeed update -c test-cf --server=http://10.0.10.25:8300 --sink-uri="mysql://127.0.0.1:3306/?max-txn-row=20&worker-number=8" --config=changefeed.toml
cdc cli changefeed resume -c test-cf --server=http://10.0.10.25:8300
```

現在、以下の構成項目を変更できます:

- チェンジフィードの `sink-uri` 。
- チェンジフィード構成ファイルとファイル内のすべての構成項目。
- チェンジフィードの `target-ts` 。

## レプリケーションのサブタスク（`processor`）の処理ユニットを管理

- `processor` のリストをクエリする:

    ```shell
    cdc cli processor list --server=http://10.0.10.25:8300
    ```

    ```shell
    [
            {
                    "id": "9f84ff74-abf9-407f-a6e2-56aa35b33888",
                    "capture-id": "b293999a-4168-4988-a4f4-35d9589b226b",
                    "changefeed-id": "simple-replication-task"
            }
    ]
    ```

- 特定のレプリケーションタスクのステータスに対応する特定のチェンジフィードをクエリする:

    ```shell
    cdc cli processor query --server=http://10.0.10.25:8300 --changefeed-id=simple-replication-task --capture-id=b293999a-4168-4988-a4f4-35d9589b226b
    ```

    ```shell
    {
      "status": {
        "tables": {
          "56": {    # 56 は TiDB 内のテーブルの tidb_table_id に対応したレプリケーションテーブルのID
            "start-ts": 417474117955485702
          }
        },
        "operation": null,
        "admin-job-type": 0
      },
      "position": {
        "checkpoint-ts": 417474143881789441,
        "resolved-ts": 417474143881789441,
        "count": 0
      }
    }
    ```

    上記のコマンドでは:

    - `status.tables`: 各キー番号は TiDB 内のテーブルの `tidb_table_id` に対応したレプリケーションテーブルのIDを表します。
    - `resolved-ts`: 現在の processor 内でソートされたデータの中で最大の TSO 。
    - `checkpoint-ts`: 現在の processor でダウンストリームに正常に書き込まれた最大の TSO 。

## 新しい照合順序フレームワークが有効なテーブルをレプリケートする

v4.0.15、v5.0.4、v5.1.1、v5.2.0から、TiCDC は[新しい照合順序フレームワーク](/character-set-and-collation.md#new-framework-for-collations)が有効なテーブルをサポートしています。

## 有効なインデックスのないテーブルをレプリケートする

v4.0.8から、TiCDC はタスク構成を変更することで有効なインデックスのないテーブルをレプリケートすることをサポートしています。この機能を有効にするには、チェンジフィード構成ファイルを次のように構成します:

```toml
force-replicate = true
```

> **警告:**
>
> `force-replicate` が `true` に設定されている場合、データの一貫性は保証されません。有効なインデックスのないテーブルに対して、`INSERT` や `REPLACE` などの操作は再入可能ではないため、データの冗長性が生じる恐れがあります。TiCDC はレプリケーション処理中にデータが一度以上しか配信されないことを保証します。したがって、有効なインデックスのないテーブルをレプリケートするためにこの機能を有効にすると、確実にデータの冗長性が発生します。データの冗長性を受け入れない場合、`AUTO RANDOM` 属性を持つ主キー列を追加するなど、有効なインデックスを追加することをお勧めします。

## 統一ソーター

> **注意:**
>
> v6.0.0から、TiCDC はデフォルトで DB ソーターエンジンを使用し、統一ソーターを使用しなくなりました。`sort engine` 項目を設定しないようにしてください。

統一ソーターは TiCDC のソートエンジンです。次のシナリオによって引き起こされる OOM 問題を緩和します:

+ TiCDC のデータレプリケーションタスクが長時間一時停止され、大量の増分データが蓄積され、レプリケートが必要となります。
+ データレプリケーションタスクが初期タイムスタンプから開始されたため、大量の増分データをレプリケートする必要があります。

v4.0.13以降で `cdc cli` を使用して作成されたチェンジフィードでは、統一ソーターがデフォルトで有効になっています。v4.0.13より前に存在したチェンジフィードについては、以前の構成が使用されます。

チェンジフィードで統一ソーター機能が有効になっているかどうかを確認するには、次の例のコマンドを実行できます（PD インスタンスのIPアドレスが `http://10.0.10.25:2379` であると仮定します）:

```shell
cdc cli --server="http://10.0.10.25:8300" changefeed query --changefeed-id=simple-replication-task | grep 'sort-engine'
```

上記のコマンドの出力で、`sort-engine` の値が "unified" であれば、チェンジフィードで統一ソーターが有効になっていることを意味します。

> **注意:**
>
> + サーバーがメカニカルハードドライブやレイテンシが高いまたは帯域幅に制限のある他のストレージデバイスを使用している場合、統一ソーターのパフォーマンスには大きな影響があります。
> + デフォルトでは、統一ソーターは一時ファイルを保存するために `data_dir` を使用します。無料ディスク容量が500 GiB以上であることを確認することをお勧めします。本番環境では、各ノードの空きディスク容量が（ビジネスのピーク時の最大 `checkpoint-ts` 遅延）*（アップストリームの書き込みトラフィック）以上であることをお勧めします。また、`changefeed` 作成後に大量の過去データをレプリケートする予定がある場合は、各ノードの空きスペースがレプリケートデータの量よりも大きいことを確認してください。