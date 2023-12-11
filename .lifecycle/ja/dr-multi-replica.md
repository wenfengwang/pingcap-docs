# 単一クラスタ内の複数レプリカに基づくDRソリューション

このドキュメントは、単一クラスタ内の複数レプリカに基づく災害復旧（DR）ソリューションについて説明しています。このドキュメントは次のように構成されています。

- ソリューションの紹介
- クラスタのセットアップとレプリカの構成方法
- クラスタのモニタリング方法
- DRスイッチオーバーの実行方法

## 紹介

重要なプロダクションシステムでは通常、リージョナルDRが必要であり、RPOはゼロでRTOは分単位であることがあります。Raftベースの分散データベースであるTiDBは、データの整合性と高可用性を保証する複数のレプリカを提供しており、リージョナルDRをサポートすることが可能です。同一リージョン内の可用ゾーン（AZ）間の小さなネットワーク遅延を考慮すると、ビジネストラフィックを同じリージョン内の2つのAZに同時にディスパッチし、リージョンリーダーとPDリーダーを適切に配置することで、同一リージョン内のAZ間で負荷分散を実現できます。

> **注意:**
>
> ["Region" in TiKV](/glossary.md#regionpeerraft-group) はデータの範囲を意味しますが、"region" という用語は物理的な場所を意味します。これらの用語は互換性がありません。

## クラスタのセットアップとレプリカの構成

このセクションでは、TiUPを使用して5つのレプリカを持つ3つのリージョンにまたがるTiDBクラスタを作成し、適切にデータとPDノードを配布してDRを実現する方法について説明します。

この例では、TiDBには5つのレプリカと3つのリージョンが含まれています。リージョン1はプライマリリージョン、リージョン2はセカンダリリージョンであり、リージョン3は投票に使用されます。同様に、PDクラスタにも5つのレプリカが含まれており、基本的にTiDBクラスタと同様の機能を提供しています。

1. 次のようなトポロジファイルを作成します。

    ```toml
    global:
      user: "root"
      ssh_port: 22
      deploy_dir: "/data/tidb_cluster/tidb-deploy"
      data_dir: "/data/tidb_cluster/tidb-data"

    server_configs:
      tikv:
        server.grpc-compression-type: gzip
      pd:
        replication.location-labels:  ["Region","AZ"] # PD schedules replicas according to the Region and AZ configuration of TiKV nodes.

    pd_servers:
      - host: tidb-dr-test1
        name: "pd-1"
      - host: tidb-dr-test2
        name: "pd-2"
      - host: tidb-dr-test3
        name: "pd-3"
      - host: tidb-dr-test4
        name: "pd-4"
      - host: tidb-dr-test5
        name: "pd-5"

    tidb_servers:
      - host: tidb-dr-test1
      - host: tidb-dr-test3

    tikv_servers:  # Label the Regions and AZs of each TiKV node through the labels option.
      - host: tidb-dr-test1
        config:
          server.labels: { Region: "Region1", AZ: "AZ1" }
      - host: tidb-dr-test2
        config:
          server.labels: { Region: "Region1", AZ: "AZ2" }
      - host: tidb-dr-test3
        config:
          server.labels: { Region: "Region2", AZ: "AZ3" }
      - host: tidb-dr-test4
        config:
          server.labels: { Region: "Region2", AZ: "AZ4" }
      - host: tidb-dr-test5
        config:
          server.labels: { Region: "Region3", AZ: "AZ5" }

          raftstore.raft-min-election-timeout-ticks: 1000
          raftstore.raft-max-election-timeout-ticks: 1200

    monitoring_servers:
      - host: tidb-dr-test2

    grafana_servers:
      - host: tidb-dr-test2

    alertmanager_servers:
      - host: tidb-dr-test2
    ```

    前述の設定は、次のオプションを使用してクロスリージョンDRに最適化されています:

    - `server.grpc-compression-type: gzip`：これによりTiKV内でgRPCメッセージの圧縮が有効化され、ネットワークトラフィックが削減されます。
    - `raftstore.raft-min-election-timeout-ticks` および `raftstore.raft-max-election-timeout-ticks`：これにより、リージョン3が選挙に参加するまでの時間が延長され、このリージョン内の任意のレプリカがリーダーとして選ばれることが防がれます。

2. 前述の構成ファイルを使用してクラスタを作成します。

    ```shell
    tiup cluster deploy drtest v6.4.0 ./topo.yaml
    tiup cluster start drtest --init
    tiup cluster display drtest
    ```

    クラスタのレプリカ数とリーダーの制限を構成します。

    ```shell
    tiup ctl:v6.4.0 pd config set max-replicas 5
    tiup ctl:v6.4.0 pd config set label-property reject-leader Region Region3

    # 以下の手順は、クラスタにテストデータを追加するものであり、オプションです。
    tiup bench tpcc  prepare -H 127.0.0.1 -P 4000 -D tpcc --warehouses 1
    ```

    PDリーダーの優先度を指定します。

    ```shell
    tiup ctl:v6.4.0 pd member leader_priority  pd-1 4
    tiup ctl:v6.4.0 pd member leader_priority  pd-2 3
    tiup ctl:v6.4.0 pd member leader_priority  pd-3 2
    tiup ctl:v6.4.0 pd member leader_priority  pd-4 1
    tiup ctl:v6.4.0 pd member leader_priority  pd-5 0
    ```

    > **注意:**
    >
    > 優先度番号が大きいほど、このノードがリーダーになる確率が高くなります。

3. 配置ルールを作成し、テストテーブルのプライマリレプリカをリージョン1に固定します。

    ```sql
    -- 2つの配置ルールを作成します: 最初のルールでは、リージョン1をプライマリリージョンとし、リージョン2をセカンダリリージョンとします。
    -- 2番目の配置ルールでは、リージョン1がダウンした場合、リージョン2がプライマリリージョンになります。
    MySQL [(none)]> CREATE PLACEMENT POLICY primary_rule_for_region1 PRIMARY_REGION="Region1" REGIONS="Region1, Region2,Region3";
    MySQL [(none)]> CREATE PLACEMENT POLICY secondary_rule_for_region2 PRIMARY_REGION="Region2" REGIONS="Region1,Region2,Region3";

    -- ルール primary_rule_for_region1 を対応するユーザーテーブルに適用します。
    ALTER TABLE tpcc.warehouse PLACEMENT POLICY=primary_rule_for_region1;
    ALTER TABLE tpcc.district PLACEMENT POLICY=primary_rule_for_region1;

    -- 注意: 必要に応じて、データベース名、テーブル名、および配置ルール名を変更できます。

    -- 次の問い合わせを実行して各リージョンのリーダー数を確認することで、リーダーが転送されたかどうかを確認できます。
    SELECT STORE_ID, address, leader_count, label FROM TIKV_STORE_STATUS ORDER BY store_id;
    ```

    次のSQLステートメントは、すべての非システムスキーマテーブルのリーダーを特定のリージョンに構成するSQLスクリプトを生成します。

    ```sql
    SET @region_name=primary_rule_for_region1;
    SELECT CONCAT('ALTER TABLE ', table_schema, '.', table_name, ' PLACEMENT POLICY=', @region_name, ';') FROM information_schema.tables WHERE table_schema NOT IN ('METRICS_SCHEMA', 'PERFORMANCE_SCHEMA', 'INFORMATION_SCHEMA','mysql');
    ```

## クラスタのモニタリング

クラスタ内のTiKV、TiDB、PD、およびその他のコンポーネントのパフォーマンスメトリクスは、GrafanaやTiDBダッシュボードにアクセスすることで監視できます。コンポーネントの状態に基づいて、DRスイッチオーバーを実行するかどうかを決定できます。詳細については、次のドキュメントを参照してください。

- [TiDBの主要モニタリングメトリクス](/grafana-tidb-dashboard.md)
- [TiKVの主要モニタリングメトリクス](/grafana-tikv-dashboard.md)
- [PDの主要モニタリングメトリクス](/grafana-pd-dashboard.md)
- [TiDBダッシュボードのモニタリングページ](/dashboard/dashboard-monitoring.md)

## DRスイッチオーバーの実行

このセクションでは、計画的なスイッチオーバーと非計画的なスイッチオーバーを含むDRスイッチオーバーの実行方法について説明します。

### 計画的なスイッチオーバー

計画的なスイッチオーバーは、メンテナンスの必要に応じてプライマリとセカンダリのリージョン間でスケジュールされるスイッチオーバーであり、DRシステムが正しく機能するかを検証するために使用できます。このセクションでは、計画的なスイッチオーバーを実行する方法について説明します。

1. 以下のコマンドを実行して、すべてのユーザーテーブルおよびPDリーダーをリージョン2に切り替えます。

    ```sql
    -- ルール secondary_rule_for_region2 を対応するユーザーテーブルに適用します。
    ALTER TABLE tpcc.warehouse PLACEMENT POLICY=secondary_rule_for_region2;
    ALTER TABLE tpcc.district PLACEMENT POLICY=secondary_rule_for_region2;
    ```

    注意: 必要に応じて、データベース名、テーブル名、および配置ルール名を変更できます。

    次のコマンドを実行して、リージョン1のPDノードの優先度を下げ、リージョン2のPDノードの優先度を上げることができます。

    ``` shell
    tiup ctl:v6.4.0 pd member leader_priority pd-1 2
```
    tiup ctl:v6.4.0 pd member leader_priority pd-2 1
    tiup ctl:v6.4.0 pd member leader_priority pd-3 4
    tiup ctl:v6.4.0 pd member leader_priority pd-4 3
    ```

2. PDおよびTiKVノードをGrafanaで観察し、PDおよびユーザーテーブルのリーダーが対象地域に移行されたことを確認してください。元の地域に切り替える手順は前述の手順と同じですが、このドキュメントではカバーしていません。

### 予期しないスイッチオーバー

予期しないスイッチオーバーとは、災害が発生した際のプライマリとセカンダリ地域間のスイッチオーバーを意味します。また、災害シナリオをシミュレートしてDRシステムの有効性を検証するために起動することもできます。

1. 次のコマンドを実行して地域1のすべてのTiKV、TiDB、PDノードを停止します：

    ``` shell
    tiup cluster stop drtest -N tidb-dr-test1:20160,tidb-dr-test2:20160,tidb-dr-test1:2379,tidb-dr-test2:2379
    ```

2. すべてのユーザーテーブルのリーダーを地域2に切り替えるには、次のコマンドを実行してください：

    ```sql
    -- 対応するユーザーテーブルにルールsecondary_rule_for_region2を適用します。
    ALTER TABLE tpcc.warehouse PLACEMENT POLICY=secondary_rule_for_region2;
    ALTER TABLE tpcc.district PLACEMENT POLICY=secondary_rule_for_region2;

    --- 以下のクエリを実行して各地域のリーダー数を確認し、リーダーが移行されたかどうかを確認してください。
    SELECT STORE_ID, address, leader_count, label FROM TIKV_STORE_STATUS ORDER BY store_id;
    ```

    地域1が回復した場合、地域1のユーザーテーブルのリーダーを地域1に切り替えるために、前述のコマンドに類似したコマンドを使用できます。