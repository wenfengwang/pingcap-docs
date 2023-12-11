---
title: TiDBデータマイグレーションをv1.0.xからv2.0+に手動でアップグレードする
summary: TiDBデータマイグレーションをv1.0.xからv2.0+に手動でアップグレードする方法について学びます。

# TiDBデータマイグレーションをv1.0.xからv2.0+に手動でアップグレードする

このドキュメントでは、TiDB DMツールをv1.0.xからv2.0+に手動でアップグレードする方法について紹介します。主なアイデアは、v1.0.xのグローバルチェックポイント情報を使用して、v2.0+クラスターで新しいデータマイグレーションタスクを開始することです。

v1.0.xからv2.0+へのTiDB DMツールの自動アップグレード方法については、[TiUPを使用してDM-Ansibleで展開された1.0クラスターを自動インポートする](/dm/maintain-dm-using-tiup.md#import-and-upgrade-a-dm-10-cluster-deployed-using-dm-ansible)を参照してください。

> **注意:**
>
> - データマイグレーションタスクがフルエクスポートまたはフルインポートのプロセス中の場合、v1.0.xからv2.0+へのDMのアップグレードはサポートされていません。
> - DMクラスターのコンポーネント間の相互作用に使用されるgRPCプロトコルが大幅に更新されたため、アップグレード前後でDMコンポーネント（dmctlを含む）が同じバージョンを使用していることを確認する必要があります。
> - v2.0+でDMクラスターのメタデータストレージ（チェックポイント、シャードDDLロックステータス、オンラインDDLメタデータなど）が大幅に更新されたため、v1.0.xのメタデータは自動的にv2.0+で再利用できません。そのため、アップグレード操作を実行する前に、次の要件が満たされていることを確認する必要があります：
>     - すべてのデータマイグレーションタスクがシャードDDL調整のプロセス中でないこと。
>     - すべてのデータマイグレーションタスクがオンラインDDL調整のプロセス中でないこと。

手動アップグレードの手順は次のとおりです。

## ステップ1：v2.0+構成ファイルを準備する

v2.0+の準備された構成ファイルには、上流データベースの構成ファイルとデータマイグレーションタスクの構成ファイルが含まれます。

### 上流データベースの構成ファイル

v2.0+では、[上流データベースの構成ファイル](/dm/dm-source-configuration-file.md)は、DM-workerのプロセス構成から分離されているため、[v1.0.x DM-worker構成](/dm/dm-worker-configuration-file.md)に基づいたソース構成を取得する必要があります。

> **注意:**
>
> アップグレード時にソース構成の`enable-gtid`が有効になっている場合、バイナリログやリレーログファイルを解析して、バイナリログ位置に対応するGTIDセットを取得する必要があります。

#### DM-Ansibleで展開されたv1.0.xクラスターのアップグレード

v1.0.x DMクラスターがDM-Ansibleによって展開されていると仮定し、次の`inventory.ini`ファイルに`dm_worker_servers`構成があるとします：

```ini
[dm_master_servers]
dm_worker1 ansible_host=172.16.10.72 server_id=101 source_id="mysql-replica-01" mysql_host=172.16.10.81 mysql_user=root mysql_password='VjX8cEeTX+qcvZ3bPaO4h0C80pe/1aU=' mysql_port=3306
dm_worker2 ansible_host=172.16.10.73 server_id=102 source_id="mysql-replica-02" mysql_host=172.16.10.82 mysql_user=root mysql_password='VjX8cEeTX+qcvZ3bPaO4h0C80pe/1aU=' mysql_port=3306
```

その後、それを以下の2つのソース構成ファイルに変換できます：

```yaml
# 元のdm_worker1に対応するソース構成。例えば、source1.yamlという名前が付けられます。
server-id: 101                                   # 元の`server_id`に対応します。
source-id: "mysql-replica-01"                    # 元の`source_id`に対応します。
from:
  host: "172.16.10.81"                           # 元の`mysql_host`に対応します。
  port: 3306                                     # 元の`mysql_port`に対応します。
  user: "root"                                   # 元の`mysql_user`に対応します。
  password: "VjX8cEeTX+qcvZ3bPaO4h0C80pe/1aU="   # 元の`mysql_password`に対応します。
```

```yaml
# 元のdm_worker2に対応するソース構成。例えば、source2.yamlという名前が付けられます。
server-id: 102                                   # 元の`server_id`に対応します。
source-id: "mysql-replica-02"                    # 元の`source_id`に対応します。
from:
  host: "172.16.10.82"                           # 元の`mysql_host`に対応します。
  port: 3306                                     # 元の`mysql_port`に対応します。
  user: "root"                                   # 元の`mysql_user`に対応します。
  password: "VjX8cEeTX+qcvZ3bPaO4h0C80pe/1aU="   # 元の`mysql_password`に対応します。
```

#### バイナリで展開されたv1.0.xクラスターのアップグレード

v1.0.x DMクラスターがバイナリで展開されていると仮定し、対応するDM-worker構成が次のようになっているとします：

```toml
log-level = "info"
log-file = "dm-worker.log"
worker-addr = ":8262"
server-id = 101
source-id = "mysql-replica-01"
flavor = "mysql"
[from]
host = "172.16.10.81"
user = "root"
password = "VjX8cEeTX+qcvZ3bPaO4h0C80pe/1aU="
port = 3306
```

その後、それを以下のソース構成ファイルに変換できます：

```yaml
server-id: 101                                   # 元の`server-id`に対応します。
source-id: "mysql-replica-01"                    # 元の`source-id`に対応します。
flavor: "mysql"                                  # 元の`flavor`に対応します。
from:
  host: "172.16.10.81"                           # 元の`from.host`に対応します。
  port: 3306                                     # 元の`from.port`に対応します。
  user: "root"                                   # 元の`from.user`に対応します。
  password: "VjX8cEeTX+qcvZ3bPaO4h0C80pe/1aU="   # 元の`from.password`に対応します。
```

### データマイグレーションタスクの構成ファイル

[データマイグレーションタスクの構成ガイド](/dm/dm-task-configuration-guide.md)によると、v2.0+は基本的にv1.0.xと互換性があります。そのため、v1.0.xの構成をそのままコピーできます。

## ステップ2：v2.0+クラスターを展開する

> **注意:**
>
> 他のv2.0+クラスターが利用可能な場合、このステップをスキップしてください。

必要なノード数に応じて、[TiUPを使用](/dm/deploy-a-dm-cluster-using-tiup.md)して新しいv2.0+クラスターを展開します。

## ステップ3：v1.0.xクラスターを停止する

元のv1.0.xクラスターがDM-Ansibleによって展開されている場合、[DM-Ansibleを使用してv1.0.xクラスターを停止](https://docs.pingcap.com/tidb-data-migration/v1.0/cluster-operations#stop-a-cluster)する必要があります。

元のv1.0.xクラスターがバイナリで展開されている場合、DM-workerおよびDM-masterプロセスを直接停止できます。

## ステップ4：データマイグレーションタスクをアップグレードする

1. [`operate-source`](/dm/dm-manage-source.md#operate-data-source)コマンドを使用して、[ステップ1](#step-1-prepare-v20-configuration-file)から上流データベースソース構成をv2.0+クラスターにロードします。

2. 下流のTiDBクラスターで、v1.0.xデータマイグレーションタスクの増分チェックポイントテーブルから対応するグローバルチェックポイント情報を取得します。

    - v1.0.xデータマイグレーション構成が`meta-schema`を指定していない（またはその値をデフォルトの`dm_meta`に指定している）と仮定し、対応するタスク名が`task_v1`であるとします。対応するチェックポイント情報がダウンストリームのTiDBの``` `dm_meta`.`task_v1_syncer_checkpoint` ```テーブルに格納されています。
    - 以下のSQLステートメントを使用して、データマイグレーションタスクに対応するすべての上流データベースソースのグローバルチェックポイント情報を取得します。

        ```sql
        > SELECT `id`, `binlog_name`, `binlog_pos` FROM `dm_meta`.`task_v1_syncer_checkpoint` WHERE `is_global`=1;
        +------------------+-------------------------+------------+
        | id               | binlog_name             | binlog_pos |
        +------------------+-------------------------+------------+
        | mysql-replica-01 | mysql-bin|000001.000123 | 15847      |
        | mysql-replica-02 | mysql-bin|000001.000456 | 10485      |
        +------------------+-------------------------+------------+
        ```

```yaml
{R}
```

```yaml
3. v1.0.xのデータ移行タスク構成ファイルを更新し、新しいv2.0+のデータ移行タスクを開始します。

   - v1.0.xのデータ移行タスク構成ファイルが `task_v1.yaml` の場合は、これをコピーして `task_v2.yaml` に名前を変更します。
   - `task_v2.yaml` に以下の変更を加えます:
       - `name` を `task_v2` などの新しい名前に変更します。
       - `task-mode` を `incremental` に変更します。
       - ステップ2で取得したグローバルチェックポイント情報に従い、各ソースの増分レプリケーションの開始点を設定します。例:

           ```yaml
           mysql-instances:
             - source-id: "mysql-replica-01"        # チェックポイント情報の `id` に対応します。
               meta:
                 binlog-name: "mysql-bin.000123"    # チェックポイント情報の `binlog_name` に対応しますが、`|000001`の部分は含みません。
                 binlog-pos: 15847                  # チェックポイント情報の `binlog_pos` に対応します。

             - source-id: "mysql-replica-02"
               meta:
                 binlog-name: "mysql-bin.000456"
                 binlog-pos: 10485
           ```

           > **注意:**
           >
           > ソース構成で `enable-gtid` が有効になっている場合、現在はビンログまたはリレーログファイルを解析し、ビンログ位置に対忋するGTIDセットを取得し、それを `meta` の `binlog-gtid` に設定する必要があります。

4. [`start-task`](/dm/dm-create-task.md) コマンドを使用して、v2.0+のデータ移行タスク構成ファイルを使ってアップグレードされたデータ移行タスクを開始します。

5. [`query-status`](/dm/dm-query-status.md) コマンドを使用して、データ移行タスクが正常に実行されているかを確認します。

データ移行タスクが正常に実行されている場合、DMのv2.0+へのアップグレードが成功していることを示します。
```