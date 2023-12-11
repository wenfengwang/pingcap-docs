---
title: DumplingとTiDB Lightningを使用したデータのバックアップとリストア
summary: DumplingとTiDB Lightningを使用してTiDBの完全なデータをバックアップおよびリストアする方法について学びます。
---

# DumplingとTiDB Lightningを使用したデータのバックアップとリストア

このドキュメントでは、DumplingとTiDB Lightningを使用してTiDBの完全なデータをバックアップおよびリストアする方法について紹介します。

少量のデータ（たとえば、50 GiB未満）のバックアップが必要で、高速なバックアップが不要な場合は、[Dumpling](/dumpling-overview.md)を使用してTiDBデータベースからデータをエクスポートし、その後、[TiDB Lightning](/tidb-lightning/tidb-lightning-overview.md)を使用してデータを別のTiDBデータベースにリストアできます。

より大きなデータベースをバックアップする必要がある場合、推奨される方法は[BR](/br/backup-and-restore-overview.md)を使用することです。ただし、Dumplingは大きなデータベースのエクスポートに使用できますが、その場合はBRがより適したツールです。

## 必要条件

- Dumplingのインストール:

    ```shell
    tiup install dumpling
    ```

- TiDB Lightningのインストール:

    ```shell
    tiup install tidb-lightning
    ```

- [Dumplingに必要なソースデータベースの権限を付与](/dumpling-overview.md#export-data-from-tidb-or-mysql)
- [TiDB Lightningに必要なターゲットデータベースの権限を付与](/tidb-lightning/tidb-lightning-requirements.md#privileges-of-the-target-database)

## リソース要件

**オペレーティングシステム**: このドキュメントの例では、新しいCentOS 7インスタンスを使用します。ローカルホストまたはクラウド上に仮想マシンを展開できます。TiDB Lightningは、デフォルトで必要なだけCPUリソースを消費するため、専用サーバーに展開することをお勧めします。これが不可能な場合は、他のTiDBコンポーネント（たとえば、`tikv-server`）と一緒に単一サーバーに展開し、`region-concurrency`を構成してTiDB LightningからのCPU使用率を制限できます。通常、サイズを論理CPUの75%に構成できます。

**メモリとCPU**: TiDB Lightningは高いリソースを消費するため、64 GiB以上のメモリと32以上のCPUコアを割り当てることをお勧めします。最高のパフォーマンスを得るためには、CPUコア対メモリ（GiB）の比率が1:2よりも大きいことを確認してください。

**ディスク容量**:

外部ストレージとしてAmazon S3、Google Cloud Storage（GCS）、またはAzure Blob Storageを使用することをお勧めします。このようなクラウドストレージを使用すると、ディスク容量の制約なしにバックアップファイルを高速に保存できます。

1つのバックアップタスクのデータをローカルディスクに保存する必要がある場合、次の制限事項に注意してください:

- Dumplingには、データソース全体を保存できるディスク容量が必要です（またはエクスポートするすべての上流テーブルを保存できるディスク容量）。必要な容量を計算するには、[ダウンストリームストレージスペース要件](/tidb-lightning/tidb-lightning-requirements.md#storage-space-of-the-target-database)を参照してください。
- インポート中、TiDB Lightningはソートされたキー値ペアを一時的に保存するためのディスク容量が必要です。ディスク容量は、データソースからの最大単一テーブルを保持できるだけのものである必要があります。

**注意**: MySQLからDumplingによってエクスポートされた正確なデータ量を計算することは難しいことがありますが、`information_schema.tables`テーブルの`DATA_LENGTH`フィールドを集計するために次のSQL文を使用して、データ量を推定できます:

```sql
-- すべてのスキーマのサイズを計算
SELECT
  TABLE_SCHEMA,
  FORMAT_BYTES(SUM(DATA_LENGTH)) AS 'データサイズ',
  FORMAT_BYTES(SUM(INDEX_LENGTH)) 'インデックスサイズ'
FROM
  information_schema.tables
GROUP BY
  TABLE_SCHEMA;

-- 5つの最大のテーブルを計算
SELECT 
  TABLE_NAME,
  TABLE_SCHEMA,
  FORMAT_BYTES(SUM(data_length)) AS 'データサイズ',
  FORMAT_BYTES(SUM(index_length)) AS 'インデックスサイズ',
  FORMAT_BYTES(SUM(data_length+index_length)) AS '合計サイズ'
FROM
  information_schema.tables
GROUP BY
  TABLE_NAME,
  TABLE_SCHEMA
ORDER BY
  SUM(DATA_LENGTH+INDEX_LENGTH) DESC
LIMIT
  5;
```

### ターゲットTiKVクラスターのディスク容量

ターゲットのTiKVクラスターには、インポートされたデータを保存するための十分なディスク容量が必要です。[標準的なハードウェア要件](/hardware-and-software-requirements.md)に加えて、ターゲットのTiKVクラスターのストレージスペースは**データソースのサイズ x [レプリカの数](/faq/manage-cluster-faq.md#is-the-number-of-replicas-in-each-region-configurable-if-yes-how-to-configure-it) x 2**よりも大きい必要があります。たとえば、クラスターがデフォルトで3つのレプリカを使用する場合、ターゲットのTiKVクラスターは、データソースのサイズよりも6倍以上のストレージスペースを持つ必要があります。この式には x 2 が含まれています。なぜなら:

- インデックスは追加のスペースを取る可能性があるからです。
- RocksDBにはスペースの増幅効果があります。

## Dumplingを使用した完全なデータのバックアップ

1. 次のコマンドを実行して、TiDBからデータを`Amazon S3`の`s3://my-bucket/sql-backup`にエクスポートします:

    ```shell
    tiup dumpling -h ${ip} -P 3306 -u root -t 16 -r 200000 -F 256MiB -B my_db1 -f 'my_db1.table[12]' -o 's3://my-bucket/sql-backup'
    ```

    DumplingはデフォルトでSQLファイル形式でデータをエクスポートします。異なるファイル形式を指定するには、`--filetype`オプションを追加できます。

    Dumplingのその他の構成については、[Dumplingのオプション一覧](/dumpling-overview.md#option-list-of-dumpling)を参照してください。

2. エクスポートが完了したら、バックアップファイルを`Amazon S3`の`s3://my-bucket/sql-backup`ディレクトリで確認できます。

## TiDB Lightningを使用した完全なデータのリストア

1. `tidb-lightning.toml`ファイルを編集して、`Amazon S3`の`s3://my-bucket/sql-backup`からDumplingを使用してバックアップされた完全なデータをターゲットTiDBクラスターにインポートします:

    ```toml
    [lightning]
    # log
    level = "info"
    file = "tidb-lightning.log"

    [tikv-importer]
    # "local": デフォルトのバックエンド。大容量のデータ（1 TiB以上）をインポートする場合にお勧めです。インポート中は、ターゲットTiDBクラスターは任意のサービスを提供できません。
    # "tidb": "tidb"バックエンドは、1 TiB未満のデータをインポートする場合にお勧めです。インポート中、ターゲットTiDBクラスターは正常にサービスを提供できます。バックエンドについての詳細は、https://docs.pingcap.com/tidb/stable/tidb-lightning-backendsを参照してください。
    backend = "local"
    # ソートされたKey-Valueファイルの一時保存ディレクトリを設定します。ディレクトリは空であり、ストレージスペースはインポートするデータセットのサイズよりも大きい必要があります。より良いインポートパフォーマンスのためには、`data-source-dir`とは異なるディレクトリを使用し、I/Oを専有できるフラッシュストレージを使用することをお勧めします。
    sorted-kv-dir = "${sorted-kv-dir}"

    [mydumper]
    # データソースディレクトリ。Dumplingがデータをエクスポートする場所と同じディレクトリです。
    data-source-dir = "${data-path}" # ローカルパスまたはS3パス。たとえば、's3://my-bucket/sql-backup'

    [tidb]
    # ターゲットTiDBクラスターの情報。
    host = ${host}                # 例: 172.16.32.1
    port = ${port}                # 例: 4000
    user = "${user_name}"         # 例: "root"
    password = "${password}"      # 例: "rootroot"
    status-port = ${status-port}  # インポート中、TiDB LightningはTiDBステータスポートからテーブルスキーマ情報を取得する必要があります。例: 10080
    pd-addr = "${ip}:${port}"     # PDクラスターのアドレス。例: 172.16.31.3:2379。TiDB LightningはPDからいくつかの情報を取得します。backendが "local"の場合は、status-portとpd-addrを正しく指定する必要があります。そうでない場合、インポートは異常になります。
    ```

TiDB Lightningの構成の詳細については、[TiDB Lightningの構成](/tidb-lightning/tidb-lightning-configuration.md)を参照してください。

2. `tidb-lightning`を実行してインポートを開始します。プログラムをコマンドラインで直接起動すると、`SIGHUP`シグナルを受信した後にプロセスが予期せず終了する場合があります。この場合、`nohup`や`screen`ツールを使用してプログラムを実行することをお勧めします。たとえば:

S3からデータをインポートする場合は、アクセスキーとシークレットキーを環境変数としてTiDB Lightningノードに渡します。または、`~/.aws/credentials`から資格情報を読み取ることもできます。

    ```shell
    export AWS_ACCESS_KEY_ID=${access_key}
    export AWS_SECRET_ACCESS_KEY=${secret_key}
    nohup tiup tidb-lightning -config tidb-lightning.toml > nohup.out 2>&1 &
    ```

3. インポートが開始されたら、ログでキーワード`progress`を`grep`してインポートの進捗状況を確認できます。進捗はデフォルトで5分ごとに更新されます。

4. TiDB Lightningがインポートを完了した後、自動的に終了します。`tidb-lightning.log`に、「the whole procedure completed」というフレーズが最後の行に含まれているかどうかを確認してください。含まれている場合、インポートは成功です。含まれていない場合、インポート中にエラーが発生しています。エラーメッセージの指示に従ってエラーを解決してください。

> **注意:**
>
> インポートが成功したかどうかに関係なく、ログの最後の行には`tidb lightning exit`と表示されます。これはTiDB Lightningが通常通り終了したことを意味しますが、必ずしもインポートが成功したことを意味するものではありません。

インポートに失敗した場合は、トラブルシューティングのために[TiDB Lightning FAQ](/tidb-lightning/tidb-lightning-faq.md)を参照してください。