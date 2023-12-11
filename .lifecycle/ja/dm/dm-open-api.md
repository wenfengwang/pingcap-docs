# OpenAPIを使用してDMクラスタを管理する

DMは、DMクラスタの状態とデータレプリケーションを管理するためのOpenAPI機能を提供しています。これは、[dmctlツール](/dm/dmctl-introduction.md)の機能と類似しています。

OpenAPIを有効にするには、次の操作のいずれかを実行します。

+ DMクラスタがバイナリを使用して直接展開された場合、以下の構成をDM-master構成ファイルに追加します。

    ```toml
    openapi = true
    ```

+ DMクラスタがTiUPを使用して展開された場合、以下の構成をトポロジファイルに追加します。

    ```yaml
    server_configs:
      master:
        openapi: true
    ```

> **注意:**
>
> - DMは、OpenAPI 3.0.0標準を満たす[仕様書](https://github.com/pingcap/tiflow/blob/master/dm/openapi/spec/dm.yaml)を提供しています。このドキュメントには、すべてのリクエストパラメータと返される値が含まれています。このドキュメントのyamlをコピーして、[Swagger Editor](https://editor.swagger.io/)でプレビューすることができます。
>
> - DM-masterノードを展開した後は、`http://{master-addr}/api/v1/docs`にアクセスしてオンラインでドキュメントをプレビューできます。
>
> - 構成ファイルでサポートされている一部の機能はOpenAPIでサポートされていません。その機能は完全に整合されていません。本番環境では、[構成ファイル](/dm/dm-config-overview.md)を使用することをお勧めします。

DMクラスタで次のメンテナンス操作を実行するためにAPIを使用できます。

## クラスタの管理用API

* [DM-masterノードの情報を取得する](#dm-masterノードの情報を取得する)
* [DM-masterノードを停止する](#dm-masterノードを停止する)
* [DM-workerノードの情報を取得する](#dm-workerノードの情報を取得する)
* [DM-workerノードを停止する](#dm-workerノードを停止する)

## データソースの管理用API

* [データソースを作成する](#データソースを作成する)
* [データソースを取得する](#データソースを取得する)
* [データソースを削除する](#データソースを削除する)
* [データソースを更新する](#データソースを更新する)
* [データソースを有効にする](#データソースを有効にする)
* [データソースを無効にする](#データソースを無効にする)
* [データソースの情報を取得する](#データソースの情報を取得する)
* [データソースのリストを取得する](#データソースのリストを取得する)
* [データソースのリレーログ機能を開始する](#データソースのリレーログ機能を開始する)
* [データソースのリレーログ機能を停止する](#データソースのリレーログ機能を停止する)
* [不要なリレーログファイルを削除する](#不要なリレーログファイルを削除する)
* [データソースとDM-worker間のバインディングを変更する](#データソースとdm-worker間のバインディングを変更する)
* [データソースのスキーマ名のリストを取得する](#データソースのスキーマ名のリストを取得する)
* [指定されたスキーマ内のテーブル名のリストを取得する](#指定されたスキーマ内のテーブル名のリストを取得する)

## レプリケーションタスクの管理用API

* [レプリケーションタスクを作成する](#レプリケーションタスクを作成する)
* [レプリケーションタスクを取得する](#レプリケーションタスクを取得する)
* [レプリケーションタスクを削除する](#レプリケーションタスクを削除する)
* [レプリケーションタスクを更新する](#レプリケーションタスクを更新する)
* [レプリケーションタスクを開始する](#レプリケーションタスクを開始する)
* [レプリケーションタスクを停止する](#レプリケーションタスクを停止する)
* [レプリケーションタスクの情報を取得する](#レプリケーションタスクの情報を取得する)
* [レプリケーションタスクのリストを取得する](#レプリケーションタスクのリストを取得する)
* [レプリケーションタスクの移行ルールを取得する](#レプリケーションタスクの移行ルールを取得する)
* [レプリケーションタスクに関連付けられているデータソースのスキーマ名のリストを取得する](#レプリケーションタスクに関連付けられているデータソースのスキーマ名のリストを取得する)
* [レプリケーションタスクに関連付けられているデータソース内の指定されたスキーマのテーブル名のリストを取得する](#レプリケーションタスクに関連付けられているデータソース内の指定されたスキーマのテーブル名のリストを取得する)
* [レプリケーションタスクに関連付けられているデータソースのスキーマのCREATEステートメントを取得する](#レプリケーションタスクに関連付けられているデータソースのスキーマのCREATEステートメントを取得する)
* [レプリケーションタスクに関連付けられているデータソースのスキーマのCREATEステートメントを更新する](#レプリケーションタスクに関連付けられているデータソースのスキーマのCREATEステートメントを更新する)
* [レプリケーションタスクに関連付けられているデータソースからスキーマを削除する](#レプリケーションタスクに関連付けられているデータソースからスキーマを削除する)

APIリクエストを送信した後にエラーが発生した場合、返されるエラーメッセージは以下の形式です。

```json
{
    "error_msg": "",
    "error_code": ""
}
```

上記のJSON出力から、`error_msg`はエラーメッセージを、`error_code`は対応するエラーコードを表します。

## DM-masterノードの情報を取得する

このAPIは同期インターフェースです。リクエストが成功した場合、対応するノードの情報が返されます。

### リクエストURI

`GET /api/v1/cluster/masters`

### 例

{{< copyable "shell-regular" >}}

```shell
curl -X 'GET' \
  'http://127.0.0.1:8261/api/v1/cluster/masters' \
  -H 'accept: application/json'
```

```json
{
  "total": 1,
  "data": [
    {
      "name": "master1",
      "alive": true,
      "leader": true,
      "addr": "127.0.0.1:8261"
    }
  ]
}
```

## DM-masterノードを停止する

このAPIは同期インターフェースです。リクエストが成功した場合、返される本体のステータスコードは204です。

### リクエストURI

`DELETE /api/v1/cluster/masters/{master-name}`

### 例

{{< copyable "shell-regular" >}}

```shell
curl -X 'DELETE' \
  'http://127.0.0.1:8261/api/v1/cluster/masters/master1' \
  -H 'accept: */*'
```

## DM-workerノードの情報を取得する

このAPIは同期インターフェースです。リクエストが成功した場合、対応するノードの情報が返されます。

### リクエストURI

`GET /api/v1/cluster/workers`

### 例

{{< copyable "shell-regular" >}}

```shell
curl -X 'GET' \
  'http://127.0.0.1:8261/api/v1/cluster/workers' \
  -H 'accept: application/json'
```

```json
{
  "total": 1,
  "data": [
    {
      "name": "worker1",
      "addr": "127.0.0.1:8261",
      "bound_stage": "bound",
      "bound_source_name": "mysql-01"
    }
  ]
}
```

## DM-workerノードを停止する

このAPIは同期インターフェースです。リクエストが成功した場合、返される本体のステータスコードは204です。

### リクエストURI

`DELETE /api/v1/cluster/workers/{worker-name}`

### 例

{{< copyable "shell-regular" >}}

```shell
curl -X 'DELETE' \
  'http://127.0.0.1:8261/api/v1/cluster/workers/worker1' \
  -H 'accept: */*'
```

## データソースを作成する

このAPIは同期インターフェースです。リクエストが成功した場合、対志するデータソースの情報が返されます。

### リクエストURI

`POST /api/v1/sources`

### 例

{{< copyable "shell-regular" >}}

```shell
curl -X 'POST' \
  'http://127.0.0.1:8261/api/v1/sources' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "source_name": "mysql-01",
  "host": "127.0.0.1",
  "port": 3306,
  "user": "root",
  "password": "123456",
  "enable": true,
  "enable_gtid": false,
  "security": {
    "ssl_ca_content": "",
    "ssl_cert_content": "",
    "ssl_key_content": "",
    "cert_allowed_cn": [
      "string"
    ]
  },
  "purge": {
    "interval": 3600,
    "expires": 0,
    "remain_space": 15
  }
}'
```

```json
{
  "source_name": "mysql-01",
  "host": "127.0.0.1",
"port": 3306,
  "user": "root",
  "password": "123456",
  "enable": true,
  "enable_gtid": false,
  "security": {
    "ssl_ca_content": "",
    "ssl_cert_content": "",
    "ssl_key_content": "",
    "cert_allowed_cn": [
      "string"
    ]
  },
  "purge": {
    "interval": 3600,
    "expires": 0,
    "remain_space": 15
  },
  "status_list": [
    {
      "source_name": "mysql-replica-01",
      "worker_name": "worker-1",
      "relay_status": {
        "master_binlog": "(mysql-bin.000001, 1979)",
        "master_binlog_gtid": "e9a1fc22-ec08-11e9-b2ac-0242ac110003:1-7849",
        "relay_dir": "./sub_dir",
        "relay_binlog_gtid": "e9a1fc22-ec08-11e9-b2ac-0242ac110003:1-7849",
        "relay_catch_up_master": true,
        "stage": "Running"
      },
      "error_msg": "string"
    }
  ]
}
```

## データソースを取得

このAPIは同期インターフェースです。リクエストが成功した場合、対応するデータソースの情報が返されます。

### リクエストURI

`GET /api/v1/sources/{source-name}`

### 例

{{< コピー可能 "shell-regular" >}}

```shell
curl -X 'GET' \
  'http://127.0.0.1:8261/api/v1/sources/mysql-01?with_status=true' \
  -H 'accept: application/json'
```

```json
{
  "source_name": "mysql-01",
  "host": "127.0.0.1",
  "port": 3306,
  "user": "root",
  "password": "123456",
  "enable_gtid": false,
  "enable": false,
  "flavor": "mysql",
  "task_name_list": [
    "task1"
  ],
  "security": {
    "ssl_ca_content": "",
    "ssl_cert_content": "",
    "ssl_key_content": "",
    "cert_allowed_cn": [
      "string"
    ]
  },
  "purge": {
    "interval": 3600,
    "expires": 0,
    "remain_space": 15
  },
  "status_list": [
    {
      "source_name": "mysql-replica-01",
      "worker_name": "worker-1",
      "relay_status": {
        "master_binlog": "(mysql-bin.000001, 1979)",
        "master_binlog_gtid": "e9a1fc22-ec08-11e9-b2ac-0242ac110003:1-7849",
        "relay_dir": "./sub_dir",
        "relay_binlog_gtid": "e9a1fc22-ec08-11e9-b2ac-0242ac110003:1-7849",
        "relay_catch_up_master": true,
        "stage": "Running"
      },
      "error_msg": "string"
    }
  ],
  "relay_config": {
    "enable_relay": true,
    "relay_binlog_name": "mysql-bin.000002",
    "relay_binlog_gtid": "e9a1fc22-ec08-11e9-b2ac-0242ac110003:1-7849",
    "relay_dir": "./relay_log"
  }
}
```

## データソースを削除

このAPIは同期インターフェースです。リクエストが成功した場合、返されるボディのステータスコードは204です。

### リクエストURI

`DELETE /api/v1/sources/{source-name}`

### 例

{{< コピー可能 "shell-regular" >}}

```shell
curl -X 'DELETE' \
  'http://127.0.0.1:8261/api/v1/sources/mysql-01?force=true' \
  -H 'accept: application/json'
```

## データソースを更新

このAPIは同期インターフェースです。リクエストが成功した場合、対応するデータソースの情報が返されます。

> **注：**
>
> データソース構成を更新するためにこのAPIを使用する場合は、現在のデータソースに実行中のタスクがないことを確認してください。

### リクエストURI

`PUT /api/v1/sources/{source-name}`

### 例

{{< コピー可能 "shell-regular" >}}

```shell
curl -X 'PUT' \
  'http://127.0.0.1:8261/api/v1/sources/mysql-01' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "source": {
    "source_name": "mysql-01",
    "host": "127.0.0.1",
    "port": 3306,
    "user": "root",
    "password": "123456",
    "enable_gtid": false,
    "enable": false,
    "flavor": "mysql",
    "task_name_list": [
      "task1"
    ],
    "security": {
      "ssl_ca_content": "",
      "ssl_cert_content": "",
      "ssl_key_content": "",
      "cert_allowed_cn": [
        "string"
      ]
    },
    "purge": {
      "interval": 3600,
      "expires": 0,
      "remain_space": 15
    },
    "relay_config": {
      "enable_relay": true,
      "relay_binlog_name": "mysql-bin.000002",
      "relay_binlog_gtid": "e9a1fc22-ec08-11e9-b2ac-0242ac110003:1-7849",
      "relay_dir": "./relay_log"
    }
  }
}'
```

```json
{
  "source_name": "mysql-01",
  "host": "127.0.0.1",
  "port": 3306,
  "user": "root",
  "password": "123456",
  "enable": true,
  "enable_gtid": false,
  "security": {
    "ssl_ca_content": "",
    "ssl_cert_content": "",
    "ssl_key_content": "",
    "cert_allowed_cn": [
      "string"
    ]
  },
  "purge": {
    "interval": 3600,
    "expires": 0,
    "remain_space": 15
  }
}
```

## データソースを有効にする

これは、リクエストが成功した場合にデータソースを有効にし、バッチでそれに依存するタスクのすべてのサブタスクを開始する同期インターフェースです。

### リクエストURI

`POST /api/v1/sources/{source-name}/enable`

### 例

{{< コピー可能 "shell-regular" >}}

```shell
curl -X 'POST' \
  'http://127.0.0.1:8261/api/v1/sources/mysql-01/enable' \
  -H 'accept: */*' \
  -H 'Content-Type: application/json'
```

## データソースを無効にする

これは、リクエストが成功した場合にこのデータソースを非アクティブにし、それに依存するすべてのタスクのサブタスクを一括で停止する同期インターフェースです。

### リクエストURI

`POST /api/v1/sources/{source-name}/disable`

### 例

{{< コピー可能 "shell-regular" >}}

```shell
curl -X 'POST' \
  'http://127.0.0.1:8261/api/v1/sources/mysql-01/disable' \
  -H 'accept: */*' \
  -H 'Content-Type: application/json'
```

## データソースリストを取得

このAPIは同期インターフェースです。リクエストが成功した場合、データソースリストが返されます。

### リクエストURI

`GET /api/v1/sources`

### 例

{{< コピー可能 "shell-regular" >}}

```shell
curl -X 'GET' \
  'http://127.0.0.1:8261/api/v1/sources?with_status=true' \
  -H 'accept: application/json'
```

```json
{
  "data": [
    {
      "enable_gtid": false,
      "host": "127.0.0.1",
      "password": "******",
      "port": 3306,
      "purge": {
        "expires": 0,
        "interval": 3600,
        "remain_space": 15
      },
      "security": null,
      "source_name": "mysql-01",
      "user": "root"
    },
    {
      "enable_gtid": false,
      "host": "127.0.0.1",
      "password": "******",
      "port": 3307,
      "purge": {
        "expires": 0,
        "interval": 3600,
        "remain_space": 15
      },
      "security": null,
      "source_name": "mysql-02",
      "user": "root"
    }
  ],
  "total": 2
}
```

## データソースの情報を取得
このAPIは同期インターフェースです。 リクエストが成功した場合、対応するノードの情報が返されます。

### リクエストURI

`GET /api/v1/sources/{source-name}/status`

### 例

{{< copyable "shell-regular" >}}

```shell
curl -X 'GET' \
  'http://127.0.0.1:8261/api/v1/sources/mysql-replica-01/status' \
  -H 'accept: application/json'
```

```json
{
  "total": 1,
  "data": [
    {
      "source_name": "mysql-replica-01",
      "worker_name": "worker-1",
      "relay_status": {
        "master_binlog": "(mysql-bin.000001, 1979)",
        "master_binlog_gtid": "e9a1fc22-ec08-11e9-b2ac-0242ac110003:1-7849",
        "relay_dir": "./sub_dir",
        "relay_binlog_gtid": "e9a1fc22-ec08-11e9-b2ac-0242ac110003:1-7849",
        "relay_catch_up_master": true,
        "stage": "Running"
      },
      "error_msg": "string"
    }
  ]
}
```

## データソースのリレーログ機能を開始する

このAPIは非同期インターフェースです。 リクエストが成功した場合、返される本体のステータスコードは200です。 最新のステータスについて詳しくは、[データソースの情報を取得する](#get-the-information-of-a-data-source) をご覧ください。

### リクエストURI

`POST /api/v1/sources/{source-name}/relay/enable`

### 例

{{< copyable "shell-regular" >}}

```shell
curl -X 'POST' \
  'http://127.0.0.1:8261/api/v1/sources/mysql-01/relay/enable' \
  -H 'accept: */*' \
  -H 'Content-Type: application/json' \
  -d '{
  "worker_name_list": [
    "worker-1"
  ],
  "relay_binlog_name": "mysql-bin.000002",
  "relay_binlog_gtid": "e9a1fc22-ec08-11e9-b2ac-0242ac110003:1-7849",
  "relay_dir": "./relay_log"
}'
```

## データソースのリレーログ機能を停止する

このAPIは非同期インターフェースです。 リクエストが成功した場合、返される本体のステータスコードは200です。 最新のステータスについて詳しくは、[データソースの情報を取得する](#get-the-information-of-a-data-source) をご覧ください。

### リクエストURI

`POST /api/v1/sources/{source-name}/relay/disable`

### 例

{{< copyable "shell-regular" >}}

```shell
curl -X 'POST' \
  'http://127.0.0.1:8261/api/v1/sources/mysql-01/relay/disable' \
  -H 'accept: */*' \
  -H 'Content-Type: application/json' \
  -d '{
  "worker_name_list": [
    "worker-1"
  ]
}'
```

## もはや不要なリレーログファイルを削除する

このAPIは非同期インターフェースです。 リクエストが成功した場合、返される本体のステータスコードは200です。 最新のステータスについて詳しくは、[データソースの情報を取得する](#get-the-information-of-a-data-source) をご覧ください。

### リクエストURI

`POST /api/v1/sources/{source-name}/relay/purge`

### 例

{{< copyable "shell-regular" >}}

```shell
curl -X 'POST' \
  'http://127.0.0.1:8261/api/v1/sources/mysql-01/relay/purge' \
  -H 'accept: */*' \
  -H 'Content-Type: application/json' \
  -d '{
  "relay_binlog_name": "mysql-bin.000002",
  "relay_dir": "string"
}'
```

## データソースとDMワーカー間のバインディングを変更する

このAPIは非同期インターフェースです。 リクエストが成功した場合、返される本体のステータスコードは200です。 最新のステータスについて詳しくは、[DMワーカーノードの情報を取得する](#get-the-information-of-a-dm-worker-node) をご覧ください。

### リクエストURI

`POST /api/v1/sources/{source-name}/transfer`

### 例

{{< copyable "shell-regular" >}}

```shell
curl -X 'POST' \
  'http://127.0.0.1:8261/api/v1/sources/mysql-01/transfer' \
  -H 'accept: */*' \
  -H 'Content-Type: application/json' \
  -d '{
  "worker_name": "worker-1"
}'
```

## データソースのスキーマ名のリストを取得する

このAPIは同期インターフェースです。 リクエストが成功した場合、対応するリストが返されます。

### リクエストURI

`GET /api/v1/sources/{source-name}/schemas`

### 例

{{< copyable "shell-regular" >}}

```shell
curl -X 'GET' \
  'http://127.0.0.1:8261/api/v1/sources/source-1/schemas' \
  -H 'accept: application/json'
```

```json
[
  "db1"
]
```

## データソース内の指定されたスキーマのテーブル名のリストを取得する

このAPIは同期インターフェースです。 リクエストが成功した場合、対応するリストが返されます。

### リクエストURI

`GET /api/v1/sources/{source-name}/schemas/{schema-name}`

### 例

{{< copyable "shell-regular" >}}

```shell
curl -X 'GET' \
  'http://127.0.0.1:8261/api/v1/sources/source-1/schemas/db1' \
  -H 'accept: application/json'
```

```json
[
  "table1"
]
```

## レプリケーションタスクを作成する

このAPIは同期インターフェースです。 リクエストが成功した場合、返される本体のステータスコードは200です。 成功したリクエストは、対応するレプリケーションタスクの情報を返します。

### リクエストURI

`POST /api/v1/tasks`

### 例

{{< copyable "shell-regular" >}}

```shell
curl -X 'POST' \
  'http://127.0.0.1:8261/api/v1/tasks' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "task": {
    "name": "task-1",
    "task_mode": "all",
    "shard_mode": "pessimistic",
    "meta_schema": "dm-meta",
    "enhance_online_schema_change": true,
    "on_duplicate": "overwrite",
    "target_config": {
      "host": "127.0.0.1",
      "port": 3306,
      "user": "root",
      "password": "123456",
      "security": {
        "ssl_ca_content": "",
        "ssl_cert_content": "",
        "ssl_key_content": "",
        "cert_allowed_cn": [
          "string"
        ]
      }
    },
    "binlog_filter_rule": {
      "rule-1": {
        "ignore_event": [
          "all dml"
        ],
        "ignore_sql": [
          "^Drop"
        ]
      },
      "rule-2": {
        "ignore_event": [
          "all dml"
        ],
        "ignore_sql": [
          "^Drop"
        ]
      },
      "rule-3": {
        "ignore_event": [
          "all dml"
        ],
        "ignore_sql": [
          "^Drop"
        ]
      }
    },
    "table_migrate_rule": [
      {
        "source": {
          "source_name": "source-name",
          "schema": "db-*",
          "table": "tb-*"
        },
        "target": {
          "schema": "db1",
          "table": "tb1"
        },
        "binlog_filter_rule": [
          "rule-1",
          "rule-2",
          "rule-3",
        ]
      }
    ],
    "source_config": {
      "full_migrate_conf": {
        "export_threads": 4,
        "import_threads": 16,
        "data_dir": "./exported_data",
        "consistency": "auto"
      },
      "incr_migrate_conf": {
        "repl_threads": 16,
        "repl_batch": 100
      },
      "source_conf": [
        {
          "source_name": "mysql-replica-01",
          "binlog_name": "binlog.000001",
          "binlog_pos": 4,
          "binlog_gtid": "03fc0263-28c7-11e7-a653-6c0b84d59f30:1-7041423,05474d3c-28c7-11e7-8352-203db246dd3d:1-170"
        }
      ]
    }
  }
}'
```

```json
{
  "name": "task-1",
```
  "name": "task-1",
  "task_mode": "all",
  "shard_mode": "pessimistic",
  "meta_schema": "dm-meta",
  "enhance_online_schema_change": true,
  "on_duplicate": "overwrite",
  "target_config": {
    "host": "127.0.0.1",
    "port": 3306,
    "user": "root",
    "password": "123456",
    "security": {
      "ssl_ca_content": "",
      "ssl_cert_content": "",
      "ssl_key_content": "",
      "cert_allowed_cn": [
        "string"
      ]
    }
  },
  "binlog_filter_rule": {
    "rule-1": {
      "ignore_event": [
        "all dml"
      ],
      "ignore_sql": [
        "^Drop"
      ]
    },
    "rule-2": {
      "ignore_event": [
        "all dml"
      ],
      "ignore_sql": [
        "^Drop"
      ]
    },
    "rule-3": {
      "ignore_event": [
        "all dml"
      ],
      "ignore_sql": [
        "^Drop"
      ]
    }
  },
  "table_migrate_rule": [
    {
      "source": {
        "source_name": "source-name",
        "schema": "db-*",
        "table": "tb-*"
      },
      "target": {
        "schema": "db1",
        "table": "tb1"
      },
      "binlog_filter_rule": [
        "rule-1",
        "rule-2",
        "rule-3"
      ]
    }
  ],
  "source_config": {
    "full_migrate_conf": {
      "export_threads": 4,
      "import_threads": 16,
      "data_dir": "./exported_data",
      "consistency": "auto"
    },
    "incr_migrate_conf": {
      "repl_threads": 16,
      "repl_batch": 100
    },
    "source_conf": [
      {
        "source_name": "mysql-replica-01",
        "binlog_name": "binlog.000001",
        "binlog_pos": 4,
        "binlog_gtid": "03fc0263-28c7-11e7-a653-6c0b84d59f30:1-7041423,05474d3c-28c7-11e7-8352-203db246dd3d:1-170"
      }
    ]
  }
}
```
```json
{
  "total": 1,
  "data": [
    {
      "source": {
        "source_name": "source-name",
        "schema": "db-*",
        "table": "tb-*"
      },
      "target": {
        "schema": "db1",
        "table": "tb1"
      },
      "binlog_filter_rule": [
        "rule-1",
        "rule-2",
        "rule-3"
      ]
    }
  ]
}
```
```yaml
'ユーザークエリ'
  -H 'accept: application/json'
```

```json
{
  "total": 0,
  "data": [
    {
      "source_schema": "db1",
      "source_table": "tb1",
      "target_schema": "db1",
      "target_table": "tb1"
    }
  ]
}
```

## レプリケーションタスクに関連付けられているデータソースのスキーマ名のリストを取得する

このAPIは同期インターフェイスです。リクエストが成功すると、対応するリストが返されます。

### リクエストURI

`GET /api/v1/tasks/{task-name}/sources/{source-name}/schemas`

### 例

{{< copyable "shell-regular" >}}

```shell
curl -X 'GET' \
  'http://127.0.0.1:8261/api/v1/tasks/task-1/sources/source-1/schemas' \
  -H 'accept: application/json'
```

```json
[
  "db1"
]
```

## レプリケーションタスクに関連付けられているデータソースの指定されたスキーマのテーブル名のリストを取得する

このAPIは同期インターフェイスです。リクエストが成功すると、対応するリストが返されます。

### リクエストURI

`GET /api/v1/tasks/{task-name}/sources/{source-name}/schemas/{schema-name}`

### 例

{{< copyable "shell-regular" >}}

```shell
curl -X 'GET' \
  'http://127.0.0.1:8261/api/v1/tasks/task-1/sources/source-1/schemas/db1' \
  -H 'accept: application/json'
```

```json
[
  "table1"
]
```

## レプリケーションタスクに関連付けられているデータソースのスキーマのCREATEステートメントを取得する

このAPIは同期インターフェースです。リクエストが成功すると、対応するCREATEステートメントが返されます。

### リクエストURI

`GET /api/v1/tasks/{task-name}/sources/{source-name}/schemas/{schema-name}/{table-name}`

### 例

{{< copyable "shell-regular" >}}

```shell
curl -X 'GET' \
  'http://127.0.0.1:8261/api/v1/tasks/task-1/sources/source-1/schemas/db1/table1' \
  -H 'accept: application/json'
```

```json
{
  "schema_name": "db1",
  "table_name": "table1",
  "schema_create_sql": "CREATE TABLE `t1` (`id` int(11) NOT NULL AUTO_INCREMENT,PRIMARY KEY (`id`) /*T![clustered_index] CLUSTERED */) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin"
}
```

## レプリケーションタスクに関連付けられているデータソースのスキーマのCREATEステートメントを更新する

このAPIは同期インターフェースです。リクエストが成功すると、返される本文のステータスコードが200になります。

### リクエストURI

`POST /api/v1/tasks/{task-name}/sources/{source-name}/schemas/{schema-name}/{table-name}`

### 例

{{< copyable "shell-regular" >}}

```shell
curl -X 'PUT' \
  'http://127.0.0.1:8261/api/v1/tasks/task-1/sources/task-1/schemas/db1/table1' \
  -H 'accept: */*' \
  -H 'Content-Type: application/json' \
  -d '{
  "sql_content": "CREATE TABLE `t1` ( `c1` int(11) DEFAULT NULL, `c2` int(11) DEFAULT NULL, `c3` int(11) DEFAULT NULL) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;",
  "flush": true,
  "sync": true
}'
```

## レプリケーションタスクに関連付けられているデータソースのスキーマを削除する

このAPIは同期インターフェースです。リクエストが成功すると、返される本文のステータスコードが200になります。

### リクエストURI

`DELETE /api/v1/tasks/{task-name}/sources/{source-name}/schemas/{schema-name}/{table-name}`

### 例

{{< copyable "shell-regular" >}}

```shell
curl -X 'DELETE' \
  'http://127.0.0.1:8261/api/v1/tasks/task-1/sources/source-1/schemas/db1/table1' \
  -H 'accept: */*'
```