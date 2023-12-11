---
title: 1つのTiDBクラスターから別のTiDBクラスターへの移行
summary: 1つのTiDBクラスターから別のTiDBクラスターへデータを移行する方法について学びます。

# 1つのTiDBクラスターから別のTiDBクラスターへの移行

このドキュメントでは、1つのTiDBクラスターから別のTiDBクラスターへのデータ移行方法について説明します。この機能は以下のシナリオに適用されます。

- データベースの分割: TiDBクラスターが過大である場合やクラスターサービス間の影響を避けたい場合に、データベースを分割できます。
- データベースの移動: データセンターの変更など、物理的なデータベースの移動。
- より新しいバージョンのTiDBクラスターへのデータ移行: データセキュリティと正確さの要件を満たすため、データをより新しいバージョンのTiDBクラスターに移行します。

このドキュメントでは、完全な移行プロセスを例示し、以下の手順を含んでいます。

1. 環境をセットアップする。

2. フルデータを移行する。

3. 増分データを移行する。

4. サービスを新しいTiDBクラスターに移行する。

## ステップ1. 環境をセットアップする

1. TiDBクラスターを展開する。

    TiUP Playgroundを使用して、上流と下流の2つのTiDBクラスターを展開します。詳細については、[TiUPを使用したオンラインTiDBクラスターの展開とメンテナンス](/tiup/tiup-cluster.md)を参照してください。

    ```shell
    # 上流クラスターの作成
    tiup --tag upstream playground --host 0.0.0.0 --db 1 --pd 1 --kv 1 --tiflash 0 --ticdc 1
    # 下流クラスターの作成
    tiup --tag downstream playground --host 0.0.0.0 --db 1 --pd 1 --kv 1 --tiflash 0 --ticdc 1
    # クラスターステータスの表示
    tiup status
    ```

2. データを初期化する。

    デフォルトでは、新しく展開されたクラスターにテストデータベースが作成されます。したがって、[sysbench](https://github.com/akopytov/sysbench#linux) を使用してテストデータを生成し、実際のシナリオでデータをシミュレートできます。

    ```shell
    sysbench oltp_write_only --config-file=./tidb-config --tables=10 --table-size=10000 prepare
    ```

    このドキュメントでは、sysbenchを使用して`oltp_write_only`スクリプトを実行します。このスクリプトでは、テストデータベースに10個のテーブルが生成され、それぞれに10,000行が含まれます。tidb-configは次の通りです。

    ```shell
    mysql-host=172.16.6.122 # 上流クラスターのIPアドレスで値を置き換えます
    mysql-port=4000
    mysql-user=root
    mysql-password=
    db-driver=mysql         # データベースドライバをMySQLに設定
    mysql-db=test           # データベースをテストデータベースに設定
    report-interval=10      # データ収集間隔を10秒に設定
    threads=10              # ワーカースレッド数を10に設定
    time=0                  # スクリプトの実行に必要な時間を設定します。Oは時間制限なしを意味します
    rate=100                # 平均TPSを100に設定
    ```

3. サービスのワークロードをシミュレートする。

    実際のシナリオでは、サービスデータは上流クラスターに継続的に書き込まれます。このドキュメントでは、sysbenchを使用してこのワークロードをシミュレートします。具体的には、以下のコマンドを実行して、10のワーカーを有効にし、合計TPSが100を超えないように3つのテーブル（sbtest1、sbtest2、sbtest3）にデータを継続的に書き込みます。

    ```shell
    sysbench oltp_write_only --config-file=./tidb-config --tables=3 run
    ```

4. 外部ストレージを準備する。

    完全バックアップでは、上流クラスターと下流クラスターの両方がバックアップファイルにアクセスできる必要があります。バックアップファイルの格納には、[外部ストレージ](/br/backup-and-restore-storages.md)の使用が推奨されます。このドキュメントでは、Minioを使用してS3互換ストレージサービスをシミュレートします。

    ```shell
    wget https://dl.min.io/server/minio/release/linux-amd64/minio
    chmod +x minio
    # アクセスキーとアクセスシークレットIDを設定してminioにアクセスします
    export HOST_IP='172.16.6.122' # 上流クラスターのIPアドレスで値を置き換えます
    export MINIO_ROOT_USER='minio'
    export MINIO_ROOT_PASSWORD='miniostorage'
    # データディレクトリを作成します。backupはバケット名です。
    mkdir -p data/backup
    # ポート6060でminioを起動します
    ./minio server ./data --address :6060 &
    ```

    上記のコマンドは、minioサーバーを1ノードで起動してS3サービスをシミュレートします。コマンド内のパラメータは次のように設定されています。

    - エンドポイント: `http://${HOST_IP}:6060/`
    - アクセスキー: `minio`
    - シークレットアクセスキー: `miniostorage`
    - バケット: `backup`

    アクセスリンクは次のようになります。

    ```shell
    s3://backup?access-key=minio&secret-access-key=miniostorage&endpoint=http://${HOST_IP}:6060&force-path-style=true
    ```

## ステップ2. フルデータを移行する

環境をセットアップした後、[BR](https://github.com/pingcap/tidb/tree/master/br)のバックアップとリストア機能を使用してフルデータを移行できます。BRは[3つの方法](/br/br-use-overview.md#deploy-and-use-br)で開始できます。このドキュメントでは、`BACKUP` と `RESTORE` のSQLステートメントを使用します。

> **注意:**
>
> - `BACKUP` と `RESTORE` のSQLステートメントは実験段階です。本番環境では使用しないでください。これらは予告なく変更または削除される可能性があります。バグを見つけた場合は、GitHubの[issue](https://github.com/pingcap/tidb/issues)で報告できます。
> - GCを無効にした状態でバックアップを実行するとクラスターパフォーマンスに影響が出る可能性があります。パフォーマンス低下を避けるために、オフピーク時間にデータのバックアップを行い、`RATE_LIMIT` を適切な値に設定することが推奨されます。
> - 上流と下流のクラスターのバージョンが異なる場合、[BRの互換性](/br/backup-and-restore-overview.md#before-you-use)を確認する必要があります。このドキュメントでは、上流と下流のクラスターが同じバージョンであると仮定しています。

1. GCを無効にする。

    増分移行中に新しく書き込まれたデータが削除されないようにするために、バックアップ前に上流クラスターのGCを無効にする必要があります。これにより、過去のデータが削除されません。

    以下のコマンドを実行して、GCを無効にします。

    ```sql
    MySQL [test]> SET GLOBAL tidb_gc_enable=FALSE;
    ```

    ```
    Query OK, 0 rows affected (0.01 sec)
    ```

    変更が有効になっていることを確認するには、`tidb_gc_enable` の値をクエリしてください。

    ```sql
    MySQL [test]> SELECT @@global.tidb_gc_enable;
    ```

    ```
    +-------------------------+:
    | @@global.tidb_gc_enable |
    +-------------------------+
    |                       0 |
    +-------------------------+
    1 row in set (0.00 sec)
    ```

2. データをバックアップする。

    上流クラスターで`BACKUP` ステートメントを実行してデータをバックアップします。

    ```sql
    MySQL [(none)]> BACKUP DATABASE * TO 's3://backup?access-key=minio&secret-access-key=miniostorage&endpoint=http://${HOST_IP}:6060&force-path-style=true' RATE_LIMIT = 120 MB/SECOND;
    ```

    ```
    +---------------+----------+--------------------+---------------------+---------------------+
    | Destination   | Size     | BackupTS           | Queue Time          | Execution Time      |
    +---------------+----------+--------------------+---------------------+---------------------+
    | s3://backup   | 10315858 | 431434047157698561 | 2022-02-25 19:57:59 | 2022-02-25 19:57:59 |
    +---------------+----------+--------------------+---------------------+---------------------+
    1 row in set (2.11 sec)
    ```

    `BACKUP`コマンドを実行した後、TiDBはバックアップデータのメタデータを返します。`BackupTS` に注意してください。この値がバックアップされる前に生成されたデータです。このドキュメントでは、`BackupTS` を **データチェックの終わり** および **TiCDCによる増分移行スキャンの開始** として使用します。

3. データをリストアする。

    下流クラスターで`RESTORE` ステートメントを実行してデータをリストアします。

    ```sql
    mysql> RESTORE DATABASE * FROM 's3://backup?access-key=minio&secret-access-key=miniostorage&endpoint=http://${HOST_IP}:6060&force-path-style=true';
    ```

    ```
    +--------------+-----------+--------------------+---------------------+---------------------+
    | Destination  | Size      | BackupTS           | Queue Time          | Execution Time      |
    +--------------+-----------+--------------------+---------------------+---------------------+
    | s3://backup  | 10315858  | 431434141450371074 | 2022-02-25 20:03:59 | 2022-02-25 20:03:59 |
    +--------------+-----------+--------------------+---------------------+---------------------+
    1 row in set (41.85 sec)
    ```

4. (オプション) データを検証する。

指定された時点で上流と下流のデータ整合性をチェックするには、[sync-diff-inspector](/sync-diff-inspector/sync-diff-inspector-overview.md) を使用できます。前の `BACKUP` 出力によると、上流クラスターは 431434047157698561 でバックアップを終了します。前の `RESTORE` 出力によると、下流は 431434141450371074 でリストアを終了します。

```shell
sync_diff_inspector -C ./config.yaml
```

sync-diff-inspector の構成方法の詳細については、[Configuration file description](/sync-diff-inspector/sync-diff-inspector-overview.md#configuration-file-description) を参照してください。このドキュメントでは、構成は次のとおりです。

```shell
# 差分構成。
######################### データソース構成 #########################
[data-sources]
[data-sources.upstream]
    host = "172.16.6.122" # 上流クラスターの IP アドレスに値を置き換えてください
    port = 4000
    user = "root"
    password = ""
    snapshot = "431434047157698561" # スナップショットを実際のバックアップ時刻に設定します（[ステップ 2. フルデータの移行](#step-2-migrate-full-data)の [データのバックアップ](#back-up-data) セクションの BackupTS ）
[data-sources.downstream]
    host = "172.16.6.125" # 下流クラスターの IP アドレスに値を置き換えてください
    port = 4000
    user = "root"
    password = ""

######################### タスク構成 #########################
[task]
    output-dir = "./output"
    source-instances = ["upstream"]
    target-instance = "downstream"
    target-check-tables = ["*.*"]
```

## ステップ 3. 増分データの移行

1. TiCDC を展開します。

    全データの移行が完了したら、増分データをレプリケートするために TiCDC を展開して構成します。プロダクション環境では、[TiCDC の展開](/ticdc/deploy-ticdc.md) で指示されているように TiCDC を展開します。このドキュメントでは、テストクラスターの作成時に TiCDC ノードが起動しています。したがって、TiCDC の展開手順をスキップし、チェンジフィード構成を進めることができます。

2. チェンジフィードを作成します。

    上流クラスターで、以下のコマンドを実行して、上流から下流クラスターへのチェンジフィードを作成します：

    {{< copyable "shell-regular" >}}

    ```shell
    tiup cdc cli changefeed create --server=http://172.16.6.122:8300 --sink-uri="mysql://root:@172.16.6.125:4000" --changefeed-id="upstream-to-downstream" --start-ts="431434047157698561"
    ```

    このコマンドでは、パラメータは次のとおりです：

    - `--server`: TiCDC クラスター内の任意のノードの IP アドレス
    - `--sink-uri`: 下流クラスターの URI
    - `--changefeed-id`: changefeed ID は、正規表現の形式である必要があります。^[a-zA-Z0-9]+(\-[a-zA-Z0-9]+)*$
    - `--start-ts`: チェンジフィードの開始タイムスタンプは、バックアップ時刻である必要があります。（[ステップ 2. フルデータの移行](#step-2-migrate-full-data)の [データのバックアップ](#back-up-data) セクション内の BackupTS ）

    チェンジフィードの構成についての詳細は、[チェンジフィード構成ファイル](/ticdc/ticdc-changefeed-config.md) を参照してください。

3. GC を有効にします。

    TiCDC を使用した増分移行では、GC は複製された履歴データのみを削除します。したがって、チェンジフィードを作成した後、次のコマンドを実行して GC を有効にする必要があります。詳細については、[TiCDC ガベージコレクション（GC）セーフポイントの完全な動作は何ですか？](/ticdc/ticdc-faq.md#what-is-the-complete-behavior-of-ticdc-garbage-collection-gc-safepoint) を参照してください。

    GC を有効にするには、次のコマンドを実行します：

    ```sql
    MySQL [test]> SET GLOBAL tidb_gc_enable=TRUE;
    ```

    ```
    クエリが OK です。 (0.01 秒)
    ```

    変更が有効になったことを確認するには、`tidb_gc_enable` の値をクエリしてください：

    ```sql
    MySQL [test]> SELECT @@global.tidb_gc_enable;
    ```

    ```
    +-------------------------+
    | @@global.tidb_gc_enable |
    +-------------------------+
    |                       1 |
    +-------------------------+
    1 行が返されました (0.00 秒)
    ```

## ステップ 4. 新しい TiDB クラスターにサービスを移行する

チェンジフィードを作成した後、上流クラスターに書き込まれたデータは低レイテンシで下流クラスターにレプリケートされます。読み取りトラフィックを下流クラスターに徐々に移行し、一定期間観察します。下流クラスターが安定している場合、次の手順を実行して書き込みトラフィックを下流クラスターに移行することができます：

1. 上流クラスターで書き込みサービスを停止します。チェンジフィードを停止する前にすべての上流データが下流にレプリケートされていることを確認してください。

    ```shell
    # 上流クラスターから下流クラスターへのチェンジフィードを停止します
    tiup cdc cli changefeed pause -c "upstream-to-downstream" --server=http://172.16.6.122:8300

    # チェンジフィードのステータスを表示します
    tiup cdc cli changefeed list
    ```

    ```
    [
      {
        "id": "upstream-to-downstream",
        "summary": {
        "state": "stopped",  # ステータスが stopped であることを確認してください
        "tso": 431747241184329729,
        "checkpoint": "2022-03-11 15:50:20.387", # この時刻は書き込みを停止した時刻より後である必要があります
        "error": null
        }
      }
    ]
    ```

2. 下流から上流へのチェンジフィードを作成します。上流と下流のデータが整合しており、クラスターに新しいデータが書き込まれていないため、`start-ts` は指定しないでデフォルト設定を使用できます。

    ```shell
    tiup cdc cli changefeed create --server=http://172.16.6.125:8300 --sink-uri="mysql://root:@172.16.6.122:4000" --changefeed-id="downstream -to-upstream"
    ```

3. 書き込みサービスを下流クラスターに移行した後、一定期間観察します。下流クラスターが安定している場合、上流クラスターを破棄することができます。