---
title: DMクラスターパフォーマンステスト
summary: DMクラスターのパフォーマンスをテストする方法を学びます。

# DMクラスターパフォーマンステスト

このドキュメントでは、DMクラスターのパフォーマンステストシナリオを構築し、データ移行に関するスピードテストと遅延テストを行う方法について説明します。

## データ移行フロー

DMクラスターのデータ移行パフォーマンスをテストするために、簡単なデータ移行フロー、つまり、MySQL -> DM -> TiDB を使用することができます。

## テスト環境の展開

- TiUPを使用してTiDBテストクラスタを展開し、すべてのデフォルト構成を使用します。
- MySQLサービスを展開します。binlogの`ROW`モードを有効にし、他の構成項目にはデフォルト構成を使用します。
- DMクラスターを展開し、DM-workerとDM-masterを用意します。

## パフォーマンステスト

### テーブルスキーマ

パフォーマンステストには、次のスキーマのテーブルを使用します：

{{< copyable "sql" >}}

```sql
CREATE TABLE `sbtest` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `k` int(11) NOT NULL DEFAULT '0',
  `c` char(120) CHARSET utf8mb4 COLLATE utf8mb4_bin NOT NULL DEFAULT '',
  `pad` char(60) CHARSET utf8mb4 COLLATE utf8mb4_bin NOT NULL DEFAULT '',
  PRIMARY KEY (`id`),
  KEY `k_1` (`k`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin
```

### フルインポートベンチマークケース

#### テストデータの生成

`sysbench`を使用して上流にテストテーブルを作成し、フルインポートのテストデータを生成します。以下の`sysbench`コマンドを実行してテストデータを生成します：

{{< copyable "shell-regular" >}}

```bash
sysbench --test=oltp_insert --tables=4 --mysql-host=172.16.4.40 --mysql-port=3306 --mysql-user=root --mysql-db=dm_benchmark --db-driver=mysql --table-size=50000000 prepare
```

#### データ移行タスクの作成

1. 上流のMySQLソースを作成し、`source-id`を`source-1`に設定します。詳細については、[データソース構成をロードする](/dm/dm-manage-source.md#operate-data-source)を参照してください。

2. マイグレーションタスク（`full`モードで）を作成します。次に、タスク構成のテンプレートです：

  ```yaml
  ---
  name: test-full
  task-mode: full

  # 実際のテスト環境のTiDB情報を使用して、マイグレーションタスクを構成します。
  target-database:
    host: "192.168.0.1"
    port: 4000
    user: "root"
    password: ""

  mysql-instances:
    -
      source-id: "source-1"
      block-allow-list:  "instance"
      mydumper-config-name: "global"
      loader-thread: 16

  # sysbenchがデータを生成するデータベースの名前を構成します。
  block-allow-list:
    instance:
      do-dbs: ["dm_benchmark"]

  mydumpers:
    global:
      rows: 32000
      threads: 32
  ```

マイグレーションタスクの作成方法の詳細については、[データ移行タスクの作成](/dm/dm-create-task.md)を参照してください。

> **注意:**
>
> - 一つのテーブルからデータを同時にエクスポートするために、`mydumpers`構成項目の`rows`オプションを使用できます。これによりデータエクスポートが高速化されます。
> - 異なる構成でのパフォーマンスをテストする場合、`mysql-instances`構成の`loader-thread`、および`mydumpers`構成項目の`rows`および`threads`を調整することができます。

#### テスト結果の取得

DM-workerログを確認します。`all data files have been finished`というメッセージが表示されたら、フルデータがインポートされたことを意味します。この場合、データをインポートするためにかかった時間を確認できます。以下はサンプルログです：

```
 [INFO] [loader.go:604] ["all data files have been finished"] [task=test] [unit=load] ["cost time"=52.439796ms]
```

テストデータのサイズとデータのインポートにかかった時間に応じて、フルデータの移行速度を計算できます。

### 増分レプリケーションベンチマークケース

#### テーブルの初期化

`sysbench`を使用して上流にテストテーブルを作成します。

#### データ移行タスクの作成

1. 上流のMySQLのソースを作成し、`source-id`を`source-1`に設定します（[フルインポートベンチマークケース](#full-import-benchmark-case)にソースを作成した場合、再度作成する必要はありません）。詳細については、[データソース構成をロードする](/dm/dm-manage-source.md#operate-data-source)を参照してください。

2. DMマイグレーションタスク（`all`モードで）を作成します。次に、タスク構成ファイルの例です：

  ```yaml
  ---
  name: test-all
  task-mode: all

  # 実際のテスト環境のTiDB情報を使用して、マイグレーションタスクを構成します。
  target-database:
    host: "192.168.0.1"
    port: 4000
    user: "root"
    password: ""

  mysql-instances:
    -
      source-id: "source-1"
      block-allow-list:  "instance"
      syncer-config-name: "global"

  # sysbenchがデータを生成するデータベースの名前を構成します。
  block-allow-list:
    instance:
      do-dbs: ["dm_benchmark"]

  syncers:
    global:
      worker-count: 16
      batch: 100
  ```

データ移行タスクの作成方法の詳細については、[データ移行タスクの作成](/dm/dm-create-task.md)を参照してください。

> **注意:**
>
> 異なる構成でのパフォーマンスをテストする場合、`syncers`構成項目の`worker-count`および`batch`を調整することができます。

#### 増分データの生成

上流で連続的に増分データを生成するには、`sysbench`コマンドを実行します：

{{< copyable "shell-regular" >}}

```bash
sysbench --test=oltp_insert --tables=4 --num-threads=32 --mysql-host=172.17.4.40 --mysql-port=3306 --mysql-user=root --mysql-db=dm_benchmark --db-driver=mysql --report-interval=10 --time=1800 run
```

> **注記:**
>
> 異なるシナリオでのデータ移行パフォーマンスをテストするには、異なる`sysbench`ステートメントを使用できます。

#### テスト結果の取得

DMのマイグレーション状況を観察するには、`query-status`コマンドを実行します。DMのモニタリングメトリクスを観察するには、Grafanaを使用できます。ここで言うモニタリングメトリクスとは、`finished sqls jobs`（単位時間あたりの完了したジョブ数）およびその他関連メトリクスを指します。詳細については、[Binlog Migration Monitoring Metrics](/dm/monitor-a-dm-cluster.md#binlog-replication)を参照してください。