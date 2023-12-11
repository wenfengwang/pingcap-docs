---
title: TiDBからMySQL互換データベースへのデータ移行
summary: TiDBからMySQL互換データベースへのデータ移行方法を学びます。

# TiDBからMySQL互換データベースへのデータ移行

このドキュメントでは、TiDBクラスタからAurora、MySQL、MariaDBなどのMySQL互換データベースへのデータ移行方法について説明します。全体のプロセスは以下の4つのステップで構成されています。

1. 環境のセットアップ
2. フルデータの移行
3. 増分データの移行
4. サービスをMySQL互換クラスタに移行する

## ステップ1. 環境のセットアップ

1. TiDBクラスタをアップストリームに展開する。

    TiUP Playgroundを使用してTiDBクラスタを展開します。詳細については、[TiUPを使用してオンラインTiDBクラスタを展開および維持する](/tiup/tiup-cluster.md)を参照してください。

    ```shell
    # TiDBクラスタを作成
    tiup playground --db 1 --pd 1 --kv 1 --tiflash 0 --ticdc 1
    # クラスタの状態を表示
    tiup status
    ```

2. MySQLインスタンスをダウンストリームに展開する。

    - ラボ環境では、次のコマンドを実行して迅速にMySQLインスタンスを展開できます。

        ```shell
        docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -p 3306:3306 -d mysql
        ```

    - 本番環境では、[MySQLのインストール](https://dev.mysql.com/doc/refman/8.0/en/installing.html)の手順に従ってMySQLインスタンスを展開できます。

3. サービスのワークロードをシミュレートする。

    ラボ環境では、`go-tpc`を使用してTiDBクラスタにデータを書き込みます。これによりTiDBクラスタでイベントの変更が生成されます。次のコマンドを実行して、TiDBクラスタに`tpcc`という名前のデータベースを作成し、そこにデータを書き込むことができます。 

    ```shell
    tiup bench tpcc -H 127.0.0.1 -P 4000 -D tpcc --warehouses 4 prepare
    tiup bench tpcc -H 127.0.0.1 -P 4000 -D tpcc --warehouses 4 run --time 300s
    ```

    `go-tpc`の詳細については、[TiDBでTPC-Cテストを実行する方法](/benchmark/benchmark-tidb-using-tpcc.md)を参照してください。

## ステップ2. フルデータの移行

環境のセットアップが完了したら、[Dumpling](/dumpling-overview.md)を使用してアップストリームのTiDBクラスタからフルデータをエクスポートできます。

> **注意:**
>
> 本番クラスタでは、GCを無効にしてバックアップを実行するとクラスタのパフォーマンスに影響を与える可能性があります。このステップはオフピーク時間に完了することをお勧めします。

1. ガベージコレクション（GC）を無効にします。

    増分移行中に新しく書き込まれたデータが削除されないようにするために、データのフルエクスポート前にアップストリームクラスタのGCを無効にする必要があります。これにより、過去のデータが削除されなくなります。

    次のコマンドを実行してGCを無効にします。

    ```sql
    MySQL [test]> SET GLOBAL tidb_gc_enable=FALSE;
    ```

    ```
    クエリが正常に実行されました：0行が変更されました（0.01秒）
    ```

    変更が有効になっているかどうかを確認するには、`tidb_gc_enable`の値をクエリしてください。

    ```sql
    MySQL [test]> SELECT @@global.tidb_gc_enable;
    ```

    ```
    +-------------------------+：
    | @@global.tidb_gc_enable |
    +-------------------------+
    |                       0 |
    +-------------------------+
    1行が返されました（0.00秒）
    ```

2. バックアップデータを取得します。

    1. Dumplingを使用してSQL形式でデータをエクスポートします。

        ```shell
        tiup dumpling -u root -P 4000 -h 127.0.0.1 --filetype sql -t 8 -o ./dumpling_output -r 200000 -F256MiB
        ```

    2. データのエクスポートが完了したら、次のコマンドを実行してメタデータを確認してください。メタデータの`Pos`はエクスポートスナップショットのTSOであり、BackupTSとして記録できます。

        ```shell
        cat dumpling_output/metadata
        ```

        ```
        Started dump at: 2022-06-28 17:49:54
        SHOW MASTER STATUS:
                Log: tidb-binlog
                Pos: 434217889191428107
                GTID:
        Finished dump at: 2022-06-28 17:49:57
        ```

3. データを復元します。

    MyLoader（オープンソースツール）を使用してデータをダウンストリームのMySQLインスタンスにインポートします。MyLoaderのインストールと使用方法の詳細については、[MyDumpler/MyLoader](https://github.com/mydumper/mydumper)を参照してください。なお、MyLoader v0.10以前のバージョンを使用する必要があります。それ以降のバージョンでは、Dumplingによってエクスポートされたメタデータファイルを処理できません。

    DumplingによってエクスポートされたフルデータをMySQLにインポートするには、次のコマンドを実行してください。

    ```shell
    myloader -h 127.0.0.1 -P 3306 -d ./dumpling_output/
    ```

4. (オプション) データを検証します。

    [sync-diff-inspector](/sync-diff-inspector/sync-diff-inspector-overview.md)を使用して、特定の時間におけるアップストリームとダウンストリームのデータの整合性を確認できます。

    ```shell
    sync_diff_inspector -C ./config.yaml
    ```

    sync-diff-inspectorの設定方法の詳細については、[構成ファイルの説明](/sync-diff-inspector/sync-diff-inspector-overview.md#configuration-file-description)を参照してください。このドキュメントでは、以下のような構成になります：

    ```toml
    # Diff Configuration.
    ######################### Datasource config #########################
    [data-sources]
    [data-sources.upstream]
            host = "127.0.0.1" # upstreamクラスタのIPアドレスで値を置換
            port = 4000
            user = "root"
            password = ""
            snapshot = "434217889191428107" # スナップショットを実際のバックアップ時刻（[ステップ2. フルデータの移行](#step-2-migrate-full-data)の「バックアップデータ」セクションのBackupTS）に設定します
    [data-sources.downstream]
            host = "127.0.0.1" # downstreamクラスタのIPアドレスで値を置換
            port = 3306
[...]
    MySQL [test]> SELECT @@global.tidb_gc_enable;
    ```

    ```
    +-------------------------+
    | @@global.tidb_gc_enable |
    +-------------------------+
    |                       1 |
    +-------------------------+
    1 行がセットになりました（0.00 秒）
    ```

## ステップ 4. サービスの移行

チェンジフィードを作成した後、アップストリームクラスタに書き込まれたデータは、低遅延でダウンストリームクラスタにレプリケーションされます。読み取りトラフィックを徐々にダウンストリームクラスタに移行できます。一定期間、読み取りトラフィックを観測します。ダウンストリームクラスタが安定している場合、以下の手順で書き込みトラフィックもダウンストリームクラスタに移行できます。

1. アップストリームクラスタで書き込みサービスを停止します。変更フィードを停止する前に、すべてのアップストリームデータがダウンストリームにレプリケートされたことを確認します。

    ```shell
    # アップストリームクラスタからダウンストリームクラスタへの変更フィードを停止
    tiup cdc cli changefeed pause -c "upstream-to-downstream" --pd=http://172.16.6.122:2379
    # 変更フィードの状態を表示
    tiup cdc cli changefeed list
    ```

    ```
    [
      {
        "id": "upstream-to-downstream",
        "summary": {
        "state": "stopped",  # 状態が stopped であることを確認
        "tso": 434218657561968641,
        "checkpoint": "2022-06-28 18:38:45.685", # この時刻は書き込み停止の時刻よりも後であることを確認
        "error": null
        }
      }
    ]
    ```

2. 書き込みサービスをダウンストリームクラスタに移行した後、一定期間観測します。ダウンストリームクラスタが安定している場合、アップストリームクラスタを破棄できます。