---
title: プライマリとセカンダリクラスター間のデータ複製
summary: プライマリクラスターからセカンダリクラスターへのデータ複製方法を学びます。
aliases: ['/docs/dev/incremental-replication-between-clusters/', '/tidb/dev/replicate-betwwen-primary-and-secondary-clusters/']
---

# プライマリとセカンダリクラスター間のデータ複製

このドキュメントでは、TiDBプライマリ（アップストリーム）クラスターとTiDBまたはMySQLセカンダリ（ダウンストリーム）クラスターを構成し、プライマリクラスターからセカンダリクラスターへのインクリメンタルデータの複製について説明します。このプロセスには、次の手順が含まれます。

1. TiDBプライマリクラスターとTiDBまたはMySQLセカンダリクラスターを構成します。
2. プライマリクラスターからセカンダリクラスターへのインクリメンタルデータを複製します。
3. プライマリクラスターが停止した場合、Redoログを使用してデータを一貫して回復します。

実行中のTiDBクラスターからセカンダリクラスターにインクリメンタルデータを複製する場合、[BR](/br/backup-and-restore-overview.md)と[TiCDC](/ticdc/ticdc-overview.md)を使用できます。

## ステップ1. 環境をセットアップする

1. TiDBクラスターをデプロイします。

    TiUP Playgroundを使用して、アップストリームとダウンストリームの2つのTiDBクラスターをデプロイします。本番環境では、[TiUPを使用してオンラインTiDBクラスターをデプロイ・維持](/tiup/tiup-cluster.md)することでクラスターをデプロイします。

    このドキュメントでは、2つのマシンにそれぞれ以下のクラスターをデプロイします。

    - ノードA: 172.16.6.123、アップストリームTiDBクラスターのデプロイ用
    - ノードB: 172.16.6.124、ダウンストリームTiDBクラスターのデプロイ用

    ```shell
    # ノードA上でアップストリームクラスターを作成
    tiup --tag upstream playground --host 0.0.0.0 --db 1 --pd 1 --kv 1 --tiflash 0 --ticdc 1
    # ノードB上でダウンストリームクラスターを作成
    tiup --tag downstream playground --host 0.0.0.0 --db 1 --pd 1 --kv 1 --tiflash 0 --ticdc 0
    # クラスターステータスを表示
    tiup status
    ```

2. データを初期化します。

    デフォルトで、新しくデプロイされたクラスターにはテスト用のデータベースが作成されます。そのため、[sysbench](https://github.com/akopytov/sysbench#linux)を使用してテストデータを生成し、実際のシナリオにおけるデータをシミュレートできます。

    ```shell
    sysbench oltp_write_only --config-file=./tidb-config --tables=10 --table-size=10000 prepare
    ```

    このドキュメントでは、sysbenchを使用して`oltp_write_only`スクリプトを実行します。このスクリプトでは、10個のテーブルがアップストリームデータベースに生成され、それぞれに1万行が含まれます。`tidb-config`は以下の通りです。

    ```shell
    mysql-host=172.16.6.122 # アップストリームクラスターのIPアドレスに置き換えます
    mysql-port=4000
    mysql-user=root
    mysql-password=
    db-driver=mysql         # データベースドライバーをMySQLに設定します
    mysql-db=test           # データベースをテストデータベースに設定します
    report-interval=10      # データ収集間隔を10秒に設定します
    threads=10              # ワーカースレッド数を10に設定します
    time=0                  # スクリプトの実行に必要な時間を設定します。0は時間制限なしを意味します
    rate=100                # 平均TPSを100に設定します
    ```

3. サービスのワークロードをシミュレートします。

    実際のシナリオでは、サービスデータが継続的にアップストリームクラスターに書き込まれます。このドキュメントでは、sysbenchを使用してこのワークロードをシミュレートします。具体的には、以下のコマンドを実行して、10個のワーカーが合計TPSが100を超えない範囲で、sbtest1、sbtest2、sbtest3の3つのテーブルに継続的にデータを書き込みます。

    ```shell
    sysbench oltp_write_only --config-file=./tidb-config --tables=3 run
    ```

4. 外部ストレージを準備します。

    フルデータバックアップでは、アップストリームとダウンストリームの両方のクラスターがバックアップファイルにアクセスできる必要があります。バックアップファイルを保存するために、[外部ストレージ](/br/backup-and-restore-storages.md)の使用を推奨します。この例では、Minioを使用してS3互換のストレージサービスをシミュレートします。

    ```shell
    wget https://dl.min.io/server/minio/release/linux-amd64/minio
    chmod +x minio
    # アクセスキーとシークレットキーを設定してminioにアクセスできるようにします
    export HOST_IP='172.16.6.123' # アップストリームクラスターのIPアドレスに置き換えます
    export MINIO_ROOT_USER='minio'
    export MINIO_ROOT_PASSWORD='miniostorage'
    # redoとbackupディレクトリを作成します。`backup`および`redo`はバケット名です
    mkdir -p data/redo
    mkdir -p data/backup
    # ポート6060でminioを起動します
    nohup ./minio server ./data --address :6060 &
    ```

    上記のコマンドは、Minioサーバーを1つのノードでS3サービスをシミュレートするために開始します。コマンドのパラメータは以下の通りに設定されています。

    - エンドポイント: `http://${HOST_IP}:6060/`
    - アクセスキー: `minio`
    - シークレットアクセスキー: `miniostorage`
    - バケット: `redo`

    リンクは以下の通りです。

    ```shell
    s3://backup?access-key=minio&secret-access-key=miniostorage&endpoint=http://${HOST_IP}:6060&force-path-style=true
    ```

## ステップ2. フルデータを移行する

環境を設定した後、[BR](https://github.com/pingcap/tidb/tree/master/br)のバックアップおよびリストア機能を使用してフルデータを移行できます。BRは[3つの方法](/br/br-use-overview.md#deploy-and-use-br)で開始できます。このドキュメントでは、SQLステートメント`BACKUP`および`RESTORE`を使用します。

> **注意:**
>
> - `BACKUP`および`RESTORE`のSQLステートメントは実験的な機能です。本番環境で使用しないことをお勧めします。これらは予告なく変更または削除される可能性があります。バグが見つかった場合は、GitHubの[問題](https://github.com/pingcap/tidb/issues)を報告できます。
> - GCを無効にしたバックアップを本番クラスターで実行すると、クラスターのパフォーマンスに影響を与える場合があります。パフォーマンスの低下を避けるために、オフピーク時にデータをバックアップし、RATE_LIMITを適切な値に設定することをお勧めします。
> - アップストリームとダウンストリームのクラスターのバージョンが異なる場合は、[BRの互換性](/br/backup-and-restore-overview.md#some-tips)を確認する必要があります。このドキュメントでは、アップストリームとダウンストリームのクラスターが同じバージョンであると仮定します。

1. GCを無効にします。

    インクリメンタル移行中に新しく書き込まれたデータが削除されないようにするために、バックアップ前にアップストリームクラスターのGCを無効にする必要があります。これにより、履歴データが削除されません。

    GCを無効にするには、次のコマンドを実行します。

    ```sql
    MySQL [test]> SET GLOBAL tidb_gc_enable=FALSE;
    ```

    ```
    クエリ OK、0行が影響を受けました (0.01 sec)
    ```

    変更が有効になっているかを確認するために、`tidb_gc_enable`の値をクエリします。

    ```sql
    MySQL [test]> SELECT @@global.tidb_gc_enable;
    ```

    ```
    +-------------------------+
    | @@global.tidb_gc_enable |
    +-------------------------+
    |                       0 |
    +-------------------------+
    1行がセットになりました (0.00 sec)
    ```

2. データをバックアップします。

    アップストリームクラスターで`BACKUP`ステートメントを実行してデータをバックアップします。

    ```sql
    MySQL [(none)]> BACKUP DATABASE * TO 's3://backup?access-key=minio&secret-access-key=miniostorage&endpoint=http://${HOST_IP}:6060&force-path-style=true' RATE_LIMIT = 120 MB/SECOND;
    ```

    ```
    +----------------------+----------+--------------------+---------------------+---------------------+
    | デスティネーション | サイズ   | バックアップTS | キュータイム       | 実行タイム      |
    +----------------------+----------+--------------------+---------------------+---------------------+
    | local:///tmp/backup/ | 10315858 | 431434047157698561 | 2022-02-25 19:57:59 | 2022-02-25 19:57:59 |
    +----------------------+----------+--------------------+---------------------+---------------------+
    1 row in set (2.11 sec)
    ```

    `BACKUP`コマンドを実行した後に、TiDBからバックアップデータのメタデータが返されます。`BackupTS`に注意を払ってください。この値より前に生成されたデータはバックアップされます。このドキュメントでは、`BackupTS`を**データチェックの終わり**および**TiCDCによるインクリメンタル移行スキャンの開始**として使用します。

3. データをリストアします。

    ダウンストリームクラスターで`RESTORE`コマンドを実行してデータをリストアします。

    ```sql
    mysql> RESTORE DATABASE * FROM 's3://backup?access-key=minio&secret-access-key=miniostorage&endpoint=http://${HOST_IP}:6060&force-path-style=true';
    ```

    ```
    +----------------------+----------+--------------------+---------------------+---------------------+
    | local:///tmp/backup/ | 10315858 | 431434141450371074 | 2022-02-25 20:03:59 | 2022-02-25 20:03:59 |
    +----------------------+----------+--------------------+---------------------+---------------------+
    1 行のセット (41.85 秒)

4. (オプション) データを検証します。

    [sync-diff-inspector](/sync-diff-inspector/sync-diff-inspector-overview.md) を使用して、アップストリームとダウンストリーム間のデータ整合性を特定の時点でチェックします。前述の `BACKUP` 出力によると、アップストリームクラスターは 431434047157698561 でバックアップを完了しています。前述の `RESTORE` 出力によると、ダウンストリームは 431434141450371074 で復元を完了しています。

    ```shell
    sync_diff_inspector -C ./config.yaml
    ```

    sync-diff-inspector の構成方法の詳細については、[構成ファイルの説明](/sync-diff-inspector/sync-diff-inspector-overview.md#configuration-file-description) を参照してください。このドキュメントでは、構成は次のようになります:

    ```shell
    # 差分構成。
    ######################### グローバル構成 #########################
    check-thread-count = 4
    export-fix-sql = true
    check-struct-only = false

    ######################### データソース構成 #########################
    [data-sources]
    [data-sources.upstream]
            host = "172.16.6.123" # アップストリームクラスターのIPアドレスに置き換えます
            port = 4000
            user = "root"
            password = ""
            snapshot = "431434047157698561" # スナップショットを実際のバックアップ時刻に設定します
    [data-sources.downstream]
            host = "172.16.6.124" # ダウンストリームクラスターのIPアドレスに置き換えます
            port = 4000
            user = "root"
            password = ""
            snapshot = "431434141450371074" # スナップショットを実際の復元時刻に設定します

    ######################### タスク構成 #########################
    [task]
            output-dir = "./output"
            source-instances = ["upstream"]
            target-instance = "downstream"
            target-check-tables = ["*.*"]
    ```

## Step 3. 増分データを移行します

1. TiCDC を展開します。

    フルデータの移行が完了したら、TiCDC を展開し、増分データをレプリケートするように構成します。本番環境では、[TiCDC の展開](/ticdc/deploy-ticdc.md)に記載されている手順に従って TiCDC を展開してください。このドキュメントでは、テストクラスターの作成時に TiCDC ノードが起動されています。よって、TiCDC を展開する手順はスキップして changefeed の構成に進みます。

2. changefeed を作成します。

    changefeed 構成ファイル `changefeed.toml` を作成します。

    ```shell
    [consistent]
    # Consistency level, eventual means enabling consistent replication
    level = "eventual"
    # Use S3 to store redo logs. Other options are local and nfs.
    storage = "s3://redo?access-key=minio&secret-access-key=miniostorage&endpoint=http://172.16.6.125:6060&force-path-style=true"
    ```

    アップストリームクラスターで、以下のコマンドを実行して、アップストリームからダウンストリームクラスターへの changefeed を作成します:

    ```shell
    tiup cdc cli changefeed create --server=http://172.16.6.122:8300 --sink-uri="mysql://root:@172.16.6.125:4000" --changefeed-id="primary-to-secondary" --start-ts="431434047157698561"
    ```

    このコマンドでは、次のパラメータが指定されています:

    - `--server`: TiCDC クラスター内の任意のノードのIPアドレス
    - `--sink-uri`: ダウンストリームクラスターのURI
    - `--start-ts`: changefeed の開始タイムスタンプ。バックアップ時刻である必要があります（または[ステップ 2. フルデータの移行](#step-2-migrate-full-data)で言及されている BackupTS）

    changefeed 構成の詳細については、[TiCDC changefeed 構成](/ticdc/ticdc-changefeed-config.md)を参照してください。

3. GC を有効にします。

    TiCDC を使用した増分移行では、GC はレプリケートされた過去のデータのみを削除します。したがって、changefeed を作成した後、次のコマンドを実行して GC を有効にする必要があります。詳細については、[TiCDC ガベージコレクション（GC）セーフポイントの完全な動作は何ですか？](/ticdc/ticdc-faq.md#what-is-the-complete-behavior-of-ticdc-garbage-collection-gc-safepoint) を参照してください。

    GC を有効にするには、次のコマンドを実行します:

    ```sql
    MySQL [test]> SET GLOBAL tidb_gc_enable=TRUE;
    ```

    ```
    クエリは OK で、(0.01 秒で 0 行が影響を受けました)
    ```

    変更が有効になったことを確認するために、`tidb_gc_enable` の値をクエリで取得します:

    ```sql
    MySQL [test]> SELECT @@global.tidb_gc_enable;
    ```

    ```
    +-------------------------+
    | @@global.tidb_gc_enable |
    +-------------------------+
    |                       1 |
    +-------------------------+
    1 行が選択されました (0.00 秒)
    ```

## Step 4. アップストリームクラスターで災害をシミュレートします

実行中のアップストリームクラスターで深刻なイベントを作成します。たとえば、tiup playground プロセスを Ctrl+C を押して終了するなどが考えられます。

## Step 5. Redo ログを使用してデータの整合性を確保します

通常、TiCDC は並行してトランザクションをダウンストリームに書き込んでスループットを向上させます。しかし、changefeed が予期せず中断された場合、ダウンストリームに最新のデータがアップストリームのものと一致していない可能性があります。整合性の不一致を解決するために、次のコマンドを実行して、ダウンストリームのデータがアップストリームのデータと整合性が取れるようにします。

```shell
tiup cdc redo apply --storage "s3://redo?access-key=minio&secret-access-key=miniostorage&endpoint=http://172.16.6.123:6060&force-path-style=true" --tmp-dir /tmp/redo --sink-uri "mysql://root:@172.16.6.124:4000"
```

- `--storage`: S3 内の redo ログの場所と認証情報
- `--tmp-dir`: S3 からダウンロードされた redo ログのキャッシュディレクトリ
- `--sink-uri`: ダウンストリームクラスターのURI

## Step 6. プライマリクラスターとそのサービスを回復させます

前のステップを実行した後、ダウンストリーム（セカンダリ）クラスターには特定の時点でアップストリーム（プライマリ）クラスターと整合性の取れたデータが含まれています。データの信頼性を確保するために、新しいプライマリとセカンダリクラスターを設定する必要があります。

1. 新しい TiDB クラスターを Node A にプライマリクラスターとして展開します。

    ```shell
    tiup --tag upstream playground v5.4.0 --host 0.0.0.0 --db 1 --pd 1 --kv 1 --tiflash 0 --ticdc 1
    ```

2. BR を使用して、セカンダリクラスターからプライマリクラスターにデータを完全バックアップおよびリストアします。

    ```shell
    # セカンダリクラスターのフルデータをバックアップします
    tiup br --pd http://172.16.6.124:2379 backup full --storage ./backup
    # セカンダリクラスターのフルデータをリストアします
    tiup br --pd http://172.16.6.123:2379 restore full --storage ./backup
    ```

3. プライマリクラスターからセカンダリクラスターにデータをバックアップするための新しい changefeed を作成します。

    ```shell
    # changefeed を作成
    tiup cdc cli changefeed create --server=http://172.16.6.122:8300 --sink-uri="mysql://root:@172.16.6.125:4000" --changefeed-id="primary-to-secondary"
    ```