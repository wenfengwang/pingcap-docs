---
title: TiDBモニタリングAPI
summary: TiDBモニタリングサービスのAPIを学ぶ。
aliases: ['/docs/dev/tidb-monitoring-api/']
---

# TiDBモニタリングAPI

次の種類のインターフェースを使用してTiDBクラスターの状態をモニタリングできます：

- [ステータスインターフェースの使用](#use-the-status-interface): このインターフェースはHTTPインターフェースを使用してコンポーネント情報を取得します。このインターフェースを使用すると、現在のTiDBサーバーの[実行状態](#running-status)とテーブルの[ストレージ情報](#storage-information)を取得できます。
- [メトリクスインターフェースの使用](#use-the-metrics-interface): このインターフェースはPrometheusを使用してコンポーネントのさまざまな操作の詳細情報を記録し、Grafanaを使用してこれらのメトリクスを表示します。

## ステータスインターフェースの使用

ステータスインターフェースはTiDBクラスター内の特定のコンポーネントの基本情報をモニタリングします。また、Keepaliveメッセージのモニターインターフェースとしても機能します。さらに、PD（Placement Driver）のステータスインターフェースでは、TiKVクラスター全体の詳細を取得できます。

### TiDBサーバー

- TiDB APIアドレス: `http://${host}:${port}`
- デフォルトポート: `10080`

### 実行状態

次の例では、`http://${host}:${port}/status`を使用してTiDBサーバーの現在の状態を取得し、サーバーが稼働しているかどうかを判断します。結果は**JSON**形式で返されます。

```bash
curl http://127.0.0.1:10080/status
{
    connections: 0,  # TiDBサーバーに接続している現在のクライアント数。
    version: "8.0.11-TiDB-v7.4.0",  # TiDBのバージョン番号。
    git_hash: "778c3f4a5a716880bcd1d71b257c8165685f0d70"  # 現在のTiDBコードのGitハッシュ。
}
```

#### ストレージ情報

次の例では、`http://${host}:${port}/schema_storage/${db}/${table}`を使用して特定のデータテーブルのストレージ情報を取得します。結果は**JSON**形式で返されます。

{{< copyable "shell-regular" >}}

```bash
curl http://127.0.0.1:10080/schema_storage/mysql/stats_histograms
```

```
{
    "table_schema": "mysql",
    "table_name": "stats_histograms",
    "table_rows": 0,
    "avg_row_length": 0,
    "data_length": 0,
    "max_data_length": 0,
    "index_length": 0,
    "data_free": 0
}
```

```bash
curl http://127.0.0.1:10080/schema_storage/test
```

```
[
    {
        "table_schema": "test",
        "table_name": "test",
        "table_rows": 0,
        "avg_row_length": 0,
        "data_length": 0,
        "max_data_length": 0,
        "index_length": 0,
        "data_free": 0
    }
]
```

### PDサーバー

- PD APIアドレス: `http://${host}:${port}/pd/api/v1/${api_name}`
- デフォルトポート: `2379`
- API名の詳細: [PD APIドキュメント](https://download.pingcap.com/pd-api-v1.html)

PDインターフェースはすべてのTiKVサーバーの状態と負荷分散の情報を提供します。単一ノードのTiKVクラスターの情報の例を以下に示します：

```bash
curl http://127.0.0.1:2379/pd/api/v1/stores
{
  "count": 1,  # TiKVノードの数。
  "stores": [  # TiKVノードのリスト。
    # 単一のTiKVノードに関する詳細情報。
    {
      "store": {
        "id": 1,
        "address": "127.0.0.1:20160",
        "version": "3.0.0-beta",
        "state_name": "Up"
      },
      "status": {
        "capacity": "20 GiB",  # 総容量。
        "available": "16 GiB",  # 利用可能な容量。
        "leader_count": 17,
        "leader_weight": 1,
        "leader_score": 17,
        "leader_size": 17,
        "region_count": 17,
        "region_weight": 1,
        "region_score": 17,
        "region_size": 17,
        "start_ts": "2019-03-21T14:09:32+08:00",  # 開始タイムスタンプ。
        "last_heartbeat_ts": "2019-03-21T14:14:22.961171958+08:00",  # 最終ハートビートのタイムスタンプ。
        "uptime": "4m50.961171958s"
      }
    }
  ]
```

## メトリクスインターフェースの使用

メトリクスインターフェースはTiDBクラスター全体の状態とパフォーマンスをモニタリングします。

- 他のデプロイ方法を使用する場合は、[PrometheusとGrafanaをデプロイ](/deploy-monitoring-services.md)してからこのインターフェースを使用してください。

PrometheusとGrafanaが正常にデプロイされたら、[Grafanaの構成](/deploy-monitoring-services.md#configure-grafana)を行ってください。