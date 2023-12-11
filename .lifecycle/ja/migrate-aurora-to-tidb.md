---
title: Amazon Aurora から TiDB へのデータ移行
summary: Amazon Aurora から TiDB へのデータ移行方法についてのドキュメントです。この移行プロセスでは、[DB スナップショット](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Managing.Backups.html)が使用され、多くのスペースと時間を節約できます。
aliases: ['/tidb/dev/migrate-from-aurora-using-lightning','/docs/dev/migrate-from-aurora-mysql-database/','/docs/dev/how-to/migrate/from-mysql-aurora/','/docs/dev/how-to/migrate/from-aurora/', '/tidb/dev/migrate-from-aurora-mysql-database/', '/tidb/dev/migrate-from-mysql-aurora/','/tidb/stable/migrate-from-aurora-using-lightning/']
---

# Amazon Aurora から TiDB へのデータ移行

このドキュメントでは、Amazon Aurora から TiDB へのデータ移行方法について説明します。移行プロセスでは、[DB スナップショット](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Managing.Backups.html)が使用され、大量のスペースと時間を節約できます。

移行全体には次の2つのプロセスがあります。

- TiDB Lightning を使用してフルデータを TiDB にインポートする
- （オプション）DM を使用して増分データを TiDB にレプリケートする

## 前提条件

- [Dumpling と TiDB Lightning をインストールする](/migration-tools.md)。ターゲット側で対応するテーブルを手動で作成する場合は、Dumpling をインストールしないでください。
- [Dumpling が必要とする上流データベースの特権を取得する](/dumpling-overview.md#required-privileges)。
- [TiDB Lightning が必要とするターゲットデータベースの特権を取得する](/tidb-lightning/tidb-lightning-faq.md#what-are-the-privilege-requirements-for-the-target-database)。

## TiDB へのフルデータのインポート

### ステップ 1. スキーマファイルのエクスポートとインポート

このセクションでは、Amazon Aurora からスキーマファイルをエクスポートし、TiDB へインポートする方法について説明します。ターゲットデータベースでテーブルを手動で作成した場合は、このステップをスキップできます。

#### 1.1. Amazon Aurora からスキーマファイルをエクスポートする

Amazon Aurora のスナップショットファイルには DDL ステートメントが含まれていないため、Dumpling を使用してスキーマをエクスポートし、TiDB Lightning を使用してターゲットデータベースにスキーマを作成する必要があります。

次のコマンドを実行して Dumpling を使用してスキーマをエクスポートします。`--filter` パラメータを使用して必要なテーブルのスキーマのみをエクスポートするようにコマンドを設定しています。詳細な情報については、[Dumpling のオプション一覧](/dumpling-overview.md#option-list-of-dumpling)を参照してください。

```shell
export AWS_ACCESS_KEY_ID=${access_key}
export AWS_SECRET_ACCESS_KEY=${secret_key}
tiup dumpling --host ${host} --port 3306 --user root --password ${password} --filter 'my_db1.table[12],mydb.*' --consistency none --no-data --output 's3://my-bucket/schema-backup'
```

上記コマンドでエクスポートされたスキーマの URI を記録してください。例: 's3://my-bucket/schema-backup'。この URI は後でスキーマファイルをインポートする際に使用されます。

Amazon S3 にアクセスするために、この Amazon S3 ストレージ パスにアクセス権限のあるアカウントのシークレットアクセスキーとアクセスキーを Dumpling または TiDB Lightning ノードに環境変数として渡すことができます。Dumpling と TiDB Lightning は `~/.aws/credentials` からも資格情報ファイルを読み取ることができます。これにより、その Dumpling または TiDB Lightning ノードでのすべてのタスクに対して再度シークレットアクセスキーやアクセスキーを提供する必要がなくなります。

#### 1.2. スキーマファイルのインポートのための TiDB Lightning 設定ファイルの作成

新しい `tidb-lightning-schema.toml` ファイルを作成し、次の内容をファイルにコピーし、対応する内容に置き換えてください。

```toml
[tidb]

# ターゲット TiDB クラスタの情報。
host = ${host}
port = ${port}
user = "${user_name}
password = "${password}"
status-port = ${status-port}  # TiDB ステータス ポート。通常、ポートは 10080 です。
pd-addr = "${ip}:${port}"     # クラスタの PD アドレス。通常、ポートは 2379 です。

[tikv-importer]
# "local": デフォルトの物理インポートモード（"local" バックエンド）を使用します。
# インポート中、ターゲット TiDB クラスタはサービスを提供できません。
# インポートモードの詳細については、https://docs.pingcap.com/tidb/stable/tidb-lightning-overview を参照してください。
backend = "local"

# キー値ファイルのソートされた一時的なストレージ ディレクトリを設定します。
# ディレクトリは空であり、データセットをインポートするサイズよりも十分なストレージ容量が必要です。
# インポートの性能向上のため、`data-source-dir` と異なるディレクトリを使用し、I/O を排他的に使用できる Flash ストレージを使用することをお勧めします。
sorted-kv-dir = "${path}"

[mydumper]
# Amazon Aurora からエクスポートされたスキーマ ファイルのディレクトリを設定します
data-source-dir = "s3://my-bucket/schema-backup"
```

TiDB クラスタで TLS を有効にする必要がある場合は、[TiDB Lightning の構成](/tidb-lightning/tidb-lightning-configuration.md)を参照してください。

#### 1.3. スキーマファイルを TiDB にインポートする

TiDB Lightning を使用してスキーマファイルをダウンストリーム TiDB にインポートします。

```shell
export AWS_ACCESS_KEY_ID=${access_key}
export AWS_SECRET_ACCESS_KEY=${secret_key}
nohup tiup tidb-lightning -config tidb-lightning-schema.toml > nohup.out 2>&1 &
```

### ステップ 2. Amazon Aurora スナップショットを Amazon S3 にエクスポートし、インポートする

このセクションでは、Amazon Aurora スナップショットを Amazon S3 にエクスポートし、TiDB Lightning を使用してTiDBにインポートする方法について説明します。

#### 2.1. Amazon Aurora スナップショットを Amazon S3 にエクスポートする

1. 後続の増分マイグレーションのための Amazon Aurora バイナリログの名前と場所を取得します。Amazon Aurora で `SHOW MASTER STATUS` コマンドを実行し、現在のバイナリログの位置を記録します。

    ```sql
    SHOW MASTER STATUS;
    ```

    出力は次のようになります。後で使用するために、バイナリログの名前と位置を記録してください。

    ```
    +----------------------------+----------+--------------+------------------+-------------------+
    | File                       | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
    +----------------------------+----------+--------------+------------------+-------------------+
    | mysql-bin-changelog.018128 |    52806 |              |                  |                   |
    +----------------------------+----------+--------------+------------------+-------------------+
    1 row in set (0.012 sec)
    ```

2. Amazon Aurora スナップショットをエクスポートします。詳細な手順については、[DB スナップショットデータを Amazon S3 にエクスポート](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/USER_ExportSnapshot.html)を参照してください。バイナリログの位置を取得した後に、5 分以内にスナップショットをエクスポートしてください。そうしないと、記録したバイナリログの位置が古くなり、増分レプリケーション中にデータの競合が発生する可能性があります。

#### 2.2. データファイル用の TiDB Lightning 設定ファイルの作成

新しい `tidb-lightning-data.toml` 構成ファイルを作成し、次の内容をファイルにコピーし、対応する内容に置き換えてください。

```toml
[tidb]

# ターゲット TiDB クラスタの情報。
host = ${host}
port = ${port}
user = "${user_name}
password = "${password}"
status-port = ${status-port}  # TiDB ステータス ポート。通常、ポートは 10080 です。
pd-addr = "${ip}:${port}"     # クラスタの PD アドレス。通常、ポートは 2379 です。

[tikv-importer]
# "local": デフォルトの物理インポートモード（"local" バックエンド）を使用します。
# インポート中、ターゲット TiDB クラスタはサービスを提供できません。
# インポートモードの詳細については、https://docs.pingcap.com/tidb/stable/tidb-lightning-overview を参照してください。
backend = "local"

# キー値ファイルのソートされた一時的なストレージ ディレクトリを設定します。
# ディレクトリは空であり、データセットをインポートするサイズよりも十分なストレージ容量が必要です。
# インポートの性能向上のため、`data-source-dir` と異なるディレクトリを使用し、I/O を排他的に使用できる Flash ストレージを使用することをお勧めします。
sorted-kv-dir = "${path}"

[mydumper]
# Amazon Aurora からエクスポートされたスナップショットファイルのディレクトリを設定します
data-source-dir = "${s3_path}"  # 例: s3://my-bucket/sql-backup

[[mydumper.files]]
# Parquet ファイルを解析する式です。
pattern = '(?i)^(?:[^/]*/)*([a-z0-9_]+)\.([a-z0-9_]+)/(?:[^/]*/)*(?:[a-z0-9\-_.]+\.(parquet))$'
schema = '$1'
table = '$2'
type = '$3'
```

TiDB クラスタで TLS を有効にする必要がある場合は、[TiDB Lightning の構成](/tidb-lightning/tidb-lightning-configuration.md)を参照してください。

#### 2.3. TiDB へのフルデータのインポート

1. Amazon Aurora スナップショットから TiDB へのデータインポートに TiDB Lightning を使用します。

    ```shell
    export AWS_ACCESS_KEY_ID=${access_key}
    export AWS_SECRET_ACCESS_KEY=${secret_key}
    nohup tiup tidb-lightning -config tidb-lightning-data.toml > nohup.out 2>&1 &
    ```

2. インポートが開始されたら、以下のいずれかの方法でインポートの進捗状況を確認できます:

    - ログでキーワード `progress` を `grep` します。進行状況はデフォルトで5分ごとに更新されます。
    - [モニタリングダッシュボード](/tidb-lightning/monitor-tidb-lightning.md) で進行状況を確認します。
    - [TiDB Lightning Web インタフェース](/tidb-lightning/tidb-lightning-web-interface.md) で進行状況を確認します。

3.TiDB Lightning がインポートを完了した後、自動的に終了します。`tidb-lightning.log` が最後の行に `the whole procedure completed` を含んでいるかどうかを確認してください。もしあれば、インポートは成功しています。なければ、インポート中にエラーが発生しています。エラーメッセージに従ってエラーを解決してください。

> **注記:**
>
> インポートが成功しているかどうかにかかわらず、ログの最後の行に `tidb lightning exit` と表示されます。これはTiDB Lightning が正常に終了したことを意味しますが、インポートが成功したことを必ずしも意味するわけではありません。

もしインポート中に問題が発生した場合は、トラブルシューティングのために [TiDB Lightning FAQ](/tidb-lightning/tidb-lightning-faq.md) を参照してください。

## TiDB に増分データをレプリケートする（オプション）

### 前提条件

- [DM をインストールする](/dm/deploy-a-dm-cluster-using-tiup.md)。
- DM で必要なソースデータベースとターゲットデータベースの権限を取得する[/dm/dm-worker-intro.md](/dm/dm-worker-intro.md)。

### ステップ1: データソースを作成する

1. 以下のように `source1.yaml` ファイルを作成してください：

    ```yaml
    # 一意である必要があります。
    source-id: "mysql-01"
    # DM-worker がグローバルトランザクション識別子 (GTID) を使用してバイナリログを取得するかどうかを構成します。このモードを有効にするには、上流の MySQL も GTID を有効にする必要があります。上流の MySQL サービスが自動的に異なるノード間でマスターを切り替えるように構成されている場合、GTID モードが必要です。
    enable-gtid: false

    from:
      host: "${host}"         # 例: 172.16.10.81
      user: "root"
      password: "${password}" # サポートされていますが、平文のパスワードを使用することはお勧めしません。使用する前に平文のパスワードを `dmctl encrypt` を使って暗号化することをお勧めします。
      port: 3306
    ```

2. 以下のコマンドを実行して、`tiup dmctl` を使用して DM クラスタにデータソースの構成をロードしてください：

    ```shell
    tiup dmctl --master-addr ${advertise-addr} operate-source create source1.yaml
    ```

    上記のコマンドで使用されるパラメータは次のとおりです：

    |パラメータ             |説明             |
    |-                      |-              |
    |`--master-addr`        |`dmctl` が接続されるクラスタ内の任意の DM-master の `{advertise-addr}`、例：172.16.10.71:8261|
    |`operate-source create`|データソースを DM クラスタにロードします。|

### ステップ2: 移行タスクを作成する

以下のように `task1.yaml` ファイルを作成してください：

```yaml
# タスク名。同時に実行されている複数のタスクには、それぞれユニークな名前が必要です。
name: "test"
# タスクモード。オプションは次のとおりです：
# - full: 完全データ移行のみを実行します。
# - incremental: バイナリログのリアルタイムレプリケーションのみを実行します。
# - all: 完全データ移行 + バイナリログのリアルタイムレプリケーションを実行します。
task-mode: "incremental"
# ターゲット TiDB データベースの構成。
target-database:
  host: "${host}"                   # 例: 172.16.10.83
  port: 4000
  user: "root"
  password: "${password}"           # サポートされていますが、平文のパスワードを使用することはお勧めしません。使用する前に平文のパスワードを `dmctl encrypt` を使って暗号化することをお勧めします。

# ブロックおよび許可リストのためのグローバル構成。各インスタンスは名前で構成を参照できます。
block-allow-list:                     # DM のバージョンが v2.0.0-beta.2 より前の場合、black-white-list を使用してください。
  listA:                              # 名前。
    do-tables:                        # 移行される上流のテーブルの許可リスト。
    - db-name: "test_db"              # 移行されるデータベースの名前。
      tbl-name: "test_table"          # 移行されるテーブルの名前。

# データソースの構成。
mysql-instances:
  - source-id: "mysql-01"               # データソース ID、つまり、source1.yaml 内の source-id
    block-allow-list: "listA"           # 上記のブロック許可リスト構成を参照します。
#       syncer-config-name: "global"    # 同期処理ユニットの実行構成の名前。
    meta:                               # `task-mode` が `incremental` であり、以下流のデータベースのチェックポイントが存在しない場合、バイナリログのレプリケーションが始まる位置です。 チェックポイントが存在する場合は、チェックポイントが使用されます。 `meta` 構成項目も下流のデータベースのチェックポイントも存在しない場合、マイグレーションは上流の最新のバイナリログ位置から開始されます。
      binlog-name: "mysql-bin.000004"   # "ステップ1. Amazon Aurora スナップショットを Amazon S3 にエクスポート" で記録されたバイナリログの位置です。 上流のデータベースがソース-レプリカの切り替えを持つ場合、GTID モードが必要です。
      binlog-pos: 109227
      # binlog-gtid: "09bec856-ba95-11ea-850a-58f2b4af5188:1-9"

# (オプション) 完全データ移行中に移行されているデータを増分的にレプリケートする必要がある場合、安全モードを有効にして、エラーを回避する必要があります。
   # このシナリオは次の場合に一般的です: 完全な移行データがデータソースの一貫性のスナップショットに属さず、その後、DM が完全移行よりも前の位置から増分データのレプリケーションを開始する。
   # syncers:            # Sync 処理ユニットの実行構成。
   #   global:            # 構成名。
   #     safe-mode: true  # このフィールドが true に設定されている場合、DM はデータソースの INSERT をターゲットデータベースの REPLACE に変更し、データソースの UPDATE をターゲットデータベースの DELETE と REPLACE に変更します。これは、テーブルスキーマにプライマリキーまたはユニークインデックスが含まれている場合、DML ステートメントが繰り返しインポートできるようにするためです。 DM は、増分レプリケーションタスクを開始または再開してから1分間で自動的に安全モードを有効にします。
```

上記のYAMLファイルは、移行タスクに必要な最小構成です。詳細な構成項目については、[DM Advanced Task Configuration File](/dm/task-configuration-file-full.md) を参照してください。

### ステップ3. 移行タスクを実行する

移行タスクを開始する前に、エラーの発生確率を低減するために、`check-task` コマンドを実行して構成が DM の要件を満たしていることを確認することをお勧めします：

```shell
tiup dmctl --master-addr ${advertise-addr} check-task task.yaml
```

その後、移行タスクを開始するために `tiup dmctl` を実行してください：

```shell
tiup dmctl --master-addr ${advertise-addr} start-task task.yaml
```

上記のコマンドで使用されるパラメータは次のとおりです：

|パラメータ              |説明      |
|-                      |-              |
|`--master-addr`        |`dmctl` が接続されるクラスタ内の任意の DM-master の `{advertise-addr}`、例：172.16.10.71:8261|
|`start-task`           |移行タスクを開始します。|

もしタスクが開始に失敗した場合は、プロンプトメッセージを確認し、構成を修正してください。その後、タスクを開始するために上記のコマンドを再実行してください。

もし問題が発生した場合、[DM error handling](/dm/dm-error-handling.md) および [DM FAQ](/dm/dm-faq.md) を参照してください。

### ステップ4. 移行タスクの状態を確認する

DM クラスタに進行中の移行タスクがあるかどうか、およびタスクの状態を確認するには、`tiup dmctl` を使用して `query-status` コマンドを実行してください：

```shell
tiup dmctl --master-addr ${advertise-addr} query-status ${task-name}
```

結果の詳細な解釈については、[Query Status](/dm/dm-query-status.md) を参照してください。

### ステップ5. タスクのモニタリングとログの表示

移行タスクの履歴の状態およびその他の内部メトリクスを表示するために、以下の手順を実行してください。

もし TiUP を使用して DM をデプロイする際に Prometheus、Alertmanager、Grafana をデプロイした場合、デプロイ時に指定した IP アドレスとポートを使用して Grafana にアクセスできます。その後、DM 関連のモニタリングメトリクスを表示するために DM ダッシュボードを選択できます。

DM が実行されている場合、DM-worker、DM-master、および dmctl はログで関連情報を出力します。これらのコンポーネントのログディレクトリは以下のとおりです：

- DM-master: DM-master プロセスパラメータ `--log-file` で指定されます。TiUP を使用して DM をデプロイした場合、デフォルトでは `/dm-deploy/dm-master-8261/log/` です。
- DM-worker: DM-worker プロセスパラメータ `--log-file` で指定されます。TiUP を使用して DM をデプロイした場合、デフォルトでは `/dm-deploy/dm-worker-8262/log/` です。

## 次の手順

- [移行タスクを一時停止する](/dm/dm-pause-task.md)。
- [移行タスクを再開する](/dm/dm-resume-task.md)。
- [移行タスクを停止する](/dm/dm-stop-task.md)。
- [クラスターデータソースとタスク構成のエクスポートおよびインポート](/dm/dm-export-import-config.md)。
- [失敗したDDLステートメントの処理](/dm/handle-failed-ddl-statements.md).