---
title: MySQLからTiDBへの大規模データセットの移行
summary: MySQLからTiDBへの大規模データセットの移行方法を学びます。

# MySQLからTiDBへの大規模データセットの移行

データ移行の対象が小さい場合、[DMを使用してデータを移行すること](/migrate-small-mysql-to-tidb.md) が容易です（完全移行および増分レプリケーションのいずれも）。ただし、DMはデータを遅い速度でインポートするため（30~50 GiB/h）、データ量が多い場合は移行に時間がかかる可能性があります。この文書での「大規模データセット」とは通常、TiB以上のデータを指します。

この文書では、DumplingとTiDB Lightningを使用して完全移行を行う方法について説明します。TiDB Lightningの[Physical Import Mode](/tidb-lightning/tidb-lightning-physical-import-mode.md) は、最大500 GiB/hの速度でデータをインポートできます。ただし、この速度はハードウェア構成、テーブルスキーマ、およびインデックスの数など、さまざまな要因に影響されます。完全な移行が完了した後、DMを使用して増分データをレプリケーションできます。

## 前提条件

- [DMをインストール](/dm/deploy-a-dm-cluster-using-tiup.md) します。
- [DumplingとTiDB Lightningをインストール](/migration-tools.md) します。
- DMで必要なソースデータベースおよびターゲットデータベースの権限を付与します(/dm/dm-worker-intro.md)。
- TiDB Lightningで必要なターゲットデータベースの権限を付与します(/tidb-lightning/tidb-lightning-faq.md#what-are-the-privilege-requirements-for-the-target-database)。
- Dumplingで必要なソースデータベースの権限を付与します(/dumpling-overview.md#export-data-from-tidb-or-mysql)。

## リソース要件

**オペレーティングシステム**: この文書の例では、新しいCentOS 7インスタンスを使用しています。ローカルホストまたはクラウド上の仮想マシンを展開できます。デフォルトでTiDB Lightningは必要に応じて多くのCPUリソースを消費するため、専用サーバーに展開することをお勧めします。これが不可能な場合は、他のTiDBコンポーネント（たとえば、`tikv-server`）と一緒に単一のサーバーに展開し、`region-concurrency`を設定して、TiDB LightningからのCPU使用率を制限することができます。通常、論理CPUの75％のサイズを構成できます。

**メモリおよびCPU**: TiDB Lightningは高いリソースを消費するため、64 GiB以上のメモリと32 CPUコア以上を割り当てることをお勧めします。最適なパフォーマンスを得るためには、CPUコアあたりのメモリ（GiB）比率が1：2よりも大きいことを確認してください。

**ディスクスペース**:

- Dumplingには、データソース全体（またはエクスポートするすべての上流テーブル）を保存できるディスクスペースが必要です。SSDが推奨されています。必要なスペースを計算するには、[Downstream storage space requirements](/tidb-lightning/tidb-lightning-requirements.md#storage-space-of-the-target-database)を参照してください。
- インポート中、TiDB Lightningはソートされたキー値ペアを保存するための一時的なスペースが必要です。ディスクスペースは、データソースから最大の単一テーブルを保持できるだけのものである必要があります。
- 全体のデータ量が大きい場合は、上流でのbinlog保存時間を増やすことができます。これにより、増分レプリケーション中にbinlogが失われないようになります。

**注意**: MySQLからDumplingによってエクスポートされた正確なデータ量を計算することは難しいですが、次のSQLステートメントを使用して`information_schema.tables`テーブルの`DATA_LENGTH`フィールドを集計することで、データ量を推定することができます。

```sql
-- すべてのスキーマのサイズを計算
SELECT 
  TABLE_SCHEMA,
  FORMAT_BYTES(SUM(DATA_LENGTH)) AS 'Data Size',
  FORMAT_BYTES(SUM(INDEX_LENGTH)) 'Index Size'
FROM
  information_schema.tables
GROUP BY
  TABLE_SCHEMA;

-- 5つの最大のテーブルを計算
SELECT 
  TABLE_NAME,
  TABLE_SCHEMA,
  FORMAT_BYTES(SUM(data_length)) AS 'Data Size',
  FORMAT_BYTES(SUM(index_length)) AS 'Index Size',
  FORMAT_BYTES(SUM(data_length+index_length)) AS 'Total Size'
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

### ターゲットTiKVクラスターのディスクスペース

ターゲットのTiKVクラスターには、インポートされたデータを保存するのに十分なディスクスペースが必要です。[標準的なハードウェア要件](/hardware-and-software-requirements.md)に加えて、ターゲットのTiKVクラスターのストレージスペースは、**データソースのサイズ×[レプリカ数](/faq/manage-cluster-faq.md#is-the-number-of-replicas-in-each-region-configurable-if-yes-how-to-configure-it)×2**よりも大きくなければなりません。たとえば、クラスターがデフォルトで3つのレプリカを使用している場合、ターゲットのTiKVクラスターはデータソースのサイズの6倍以上のストレージスペースを持っている必要があります。この計算式には、以下のような理由で`x 2`が含まれています:

- インデックスが追加のスペースを必要とする可能性があります。
- RocksDBにはスペース増幅の効果があります。

## ステップ1. MySQLからすべてのデータをエクスポート

1. 以下のコマンドを実行してMySQLからすべてのデータをエクスポートします:

    {{< copyable "shell-regular" >}}

    ```shell
    tiup dumpling -h ${ip} -P 3306 -u root -t 16 -r 200000 -F 256MiB -B my_db1 -f 'my_db1.table[12]' -o 's3://my-bucket/sql-backup'
    ```

    DumplingはデフォルトでSQLファイルとしてデータをエクスポートします。異なるファイル形式を指定するには、`--filetype`オプションを追加できます。

    上記で使用されているパラメータは次の通りです。詳細なDumplingパラメータについては、[Dumpling Overview](/dumpling-overview.md)を参照してください。

    |パラメータ              |説明|
    |-                      |-|
    |`-u`または`--user`      |MySQLユーザー|
    |`-p`または`--password`  |MySQLユーザーパスワード|
    |`-P`または`--port`      |MySQLポート|
    |`-h`または`--host`      |MySQLのIPアドレス|
    |`-t`または`--thread`    |エクスポートに使用するスレッド数|
    |`-o`または`--output`    |エクスポートされたファイルを保存するディレクトリ。ローカルパスまたは[外部ストレージURI](/external-storage-uri.md)をサポート|
    |`-r`または`--row`       |単一ファイルの最大行数|
    |`-F`                   |単一のファイルの最大サイズ（MiB単位）。推奨値: 256 MiB。|
    |-`B`または`--database`  |エクスポートするデータベースを指定|
    |`-f`または`--filter`    |パターンに一致するテーブルをエクスポートします。構文については、[table-filter](/table-filter.md)を参照|

    `${data-path}`には、エクスポートされたすべての上流テーブルを保存するスペースが十分にあることを確認してください。必要なスペースを計算するには、[Downstream storage space requirements](/tidb-lightning/tidb-lightning-requirements.md#storage-space-of-the-target-database)を参照してください。大きなテーブルがすべてのスペースを消費してエクスポートが中断されないようにするために、単一ファイルのサイズを制限するために`-F`オプションを使用することを強くお勧めします。

2. `${data-path}`ディレクトリ内の`metadata`ファイルを表示します。これはDumplingが生成したメタデータファイルです。ステップ3での増分レプリケーションに必要なbinlog位置情報を記録してください。

    ```
    SHOW MASTER STATUS:
    Log: mysql-bin.000004
    Pos: 109227
    GTID:
    ```

## ステップ2. TiDBに完全なデータをインポート

1. `tidb-lightning.toml`構成ファイルを作成します:

    {{< copyable "" >}}

    ```toml
    [lightning]
    # log.
    level = "info"
    file = "tidb-lightning.log"

    [tikv-importer]
    # "local": デフォルトのバックエンド。大容量のデータ（1 TiB以上）をインポートする場合、ローカルバックエンドが推奨されます。インポート中、ターゲットのTiDBクラスターはサービスを提供できません。
    # "tidb": データが1 TiB未満の場合、"tidb"バックエンドが推奨されます。インポート中、ターゲットのTiDBクラスターは通常サービスを提供できます。バックエンドについて詳しくは、https://docs.pingcap.com/tidb/stable/tidb-lightning-backends を参照してください。
    backend = "local"
    # ソートされたキー値ファイルの一時的な保存ディレクトリを設定します。ディレクトリは空でなければならず、ストレージスペースはインポートするデータセットのサイズよりも大きくなければなりません。よりよいインポートのパフォーマンスを得るため、`data-source-dir`とは異なるディレクトリを使用し、I/Oを専有できるフラッシュストレージを使用することを推奨します。
    sorted-kv-dir = "${sorted-kv-dir}"

    [mydumper]
    # データソースディレクトリ。MySQLからデータをエクスポートするのと同じディレクトリです "ステップ1. MySQLからすべてのデータをエクスポート".
    data-source-dir = "${data-path}" # ローカルパスまたはS3パス。例: 's3://my-bucket/sql-backup'。

    [tidb]
    # ターゲットのTiDBクラスター情報。
    host = ${host}                # 例: 172.16.32.1
    port = ${port}                # 例: 4000
```
    user = "${user_name}"         # 例: "root"
    password = "${password}"      # 例: "rootroot"
    status-port = ${status-port}  # インポート中、TiDB LightningはTiDBステータスポートからテーブルのスキーマ情報を取得する必要があります。 例: 10080
    pd-addr = "${ip}:${port}"     # PDクラスタのアドレス、例: 172.16.31.3:2379。 TiDB LightningはPDからいくつかの情報を取得します。 backend = "local"の場合は、status-portとpd-addrを正しく指定する必要があります。さもなければ、インポートが異常になります。

    ```
    TiDB Lightningの構成に関する詳細情報については、[TiDB Lightningの構成](/tidb-lightning/tidb-lightning-configuration.md)を参照してください。

2. `tidb-lightning`を実行してインポートを開始します。プログラムをコマンドラインで直接起動した場合、プロセスが予期せず終了することがあります。その場合は、`nohup`または`screen`ツールを使用してプログラムを実行することをお勧めします。例:

    S3からデータをインポートする場合は、S3ストレージパスへのアクセス権を持つSecretKeyとAccessKeyを環境変数としてTiDB Lightningノードに渡します。または`~/.aws/credentials`から認証情報を読み取ることもできます。

    {{< copyable "shell-regular" >}}

    ```shell
    export AWS_ACCESS_KEY_ID=${access_key}
    export AWS_SECRET_ACCESS_KEY=${secret_key}
    nohup tiup tidb-lightning -config tidb-lightning.toml > nohup.out 2>&1 &
    ```

3. インポートを開始した後、次の方法のいずれかで進捗状況を確認できます:

    - ログでキーワード`progress`を`grep`します。デフォルトでは、進捗は5分ごとに更新されます。
    - [モニタリングダッシュボード](/tidb-lightning/monitor-tidb-lightning.md)で進捗を確認します。
    - [TiDB Lightningウェブインターフェース](/tidb-lightning/tidb-lightning-web-interface.md)で進捗を確認します。

4. TiDB Lightningがインポートを完了したら、自動的に終了します。`tidb-lightning.log`に`the whole procedure completed`が含まれているかどうかを確認します。含まれている場合は、インポートに成功しています。含まれていない場合は、インポートでエラーが発生しています。エラーメッセージに従ってエラーを修正します。

> **注意:**
>
> インポートが成功したかどうかに関係なく、ログの最終行に`tidb lightning exit`と表示されます。これはTiDB Lightningが正常に終了したことを意味しますが、インポートが成功したことを必ずしも意味しません。

インポートに失敗した場合は、トラブルシューティングのために[TiDB Lightning FAQ](/tidb-lightning/tidb-lightning-faq.md)を参照してください。

## ステップ3. TiDBへの増分データのレプリケーション

### データソースの追加

1. 以下のように`source1.yaml`ファイルを作成します:

    {{< copyable "" >}}

    ```yaml
    # ユニークである必要があります。
    source-id: "mysql-01"

    # DM-workerがグローバルトランザクション識別子(GTID)を使用してバイナリログを取得するかどうかを構成します。このモードを有効にするには、上流のMySQLもGTIDを有効にする必要があります。上流のMySQLサービスが自動的に異なるノード間でマスターを切り替えるように構成されている場合は、GTIDモードが必要です。
    enable-gtid: true

    from:
      host: "${host}"           # 例: 172.16.10.81
      user: "root"
      password: "${password}"   # サポートされていますが、平文パスワードを使用することはお勧めしません。使用する前に`dmctl encrypt`を使用して平文パスワードを暗号化することをお勧めします。
      port: 3306
    ```

2. 以下のコマンドを実行して`source1.yaml`ファイルを使用してDMクラスタにデータソースの構成を読み込みます:

    {{< copyable "shell-regular" >}}

    ```shell
    tiup dmctl --master-addr ${advertise-addr} operate-source create source1.yaml
    ```

    上記のコマンドで使用されるパラメータは次のように説明されます:

    |パラメータ              |説明    |
    |-                      |-              |
    |`--master-addr`        |`dmctl`が接続されるクラスタ内の任意のDM-masterの`{advertise-addr}`。 例: 172.16.10.71:8261|
    |`operate-source create`|データソースをDMクラスタに読み込みます。|

### レプリケーションタスクの追加

1. `task.yaml`ファイルを編集します。増分レプリケーションモードと各データソースの開始ポイントを構成します:

    {{< copyable "shell-regular" >}}

    ```yaml
    name: task-test                      # タスク名。グローバルにユニークである必要があります。
    task-mode: incremental               # タスクモード。 "incremental"モードは増分データのレプリケーションのみを行います。

    # ターゲットTiDBデータベースを構成します。
    target-database:                     # ターゲットデータベースインスタンス。
      host: "${host}"                    # 例: 127.0.0.1
      port: 4000
      user: "root"
      password: "${password}"            # 使用する前に`dmctl encrypt`を使用して平文パスワードを暗号化することをお勧めします。

    # ブロックおよび許可リストを使用して、レプリケーションするテーブルを指定します。
    block-allow-list:                    # ソースデータベースインスタンスのテーブルに一致するフィルタリングルールのコレクション。DMバージョンがv2.0.0-beta.2より前の場合は、black-white-listを使用します。
      bw-rule-1:                         # ブロック許可リスト構成項目ID。
        do-dbs: ["${db-name}"]           # レプリケートするデータベースの名前。

    # データソースを構成します。
    mysql-instances:
      - source-id: "mysql-01"            # データソースID、つまりsource1.yaml内のsource-id
        block-allow-list: "bw-rule-1"    # 上記のブロック許可リスト構成を使用できます。
        # syncer-config-name: "global"    # 以下で増分データの構成を使用できます。
        meta:                            # 'task-mode'が'incremental'であり、ダウンストリームデータベースのチェックポイントが存在しない場合、バイナリログレプリケーションが開始される位置を構成します。 チェックポイントが存在する場合、チェックポイントが使用されます。 `meta`構成項目または下流のデータベースのチェックポイントがいずれも存在しない場合、マイグレーションは上流の最新のバイナリログ位置から開始されます。
          # binlog-name: "mysql-bin.000004"  # "Step 1. Export all data from MySQL"で記録されたバイナリログ位置。上流データベースサービスが自動的に異なるノード間でマスターを切り替えるように構成されている場合、GTIDモードが必要です。
          # binlog-pos: 109227
          binlog-gtid: "09bec856-ba95-11ea-850a-58f2b4af5188:1-9"

    # (オプション)完全なデータ移行の後に既に移行されたデータを増分的にレプリケートする必要がある場合、安全モードを有効にして、増分データレプリケーションエラーを回避するためにセーフモードを有効にする必要があります。
    # このシナリオは次の場合に一般的です: 完全な移行データがデータソースの一貫性のスナップショットに属さない場合、その後、DMは完全な移行よりも前の位置から増分データをレプリケートし始めます。
    # syncers:            # sync処理ユニットの実行構成。
    #   global:           # 構成名。
    #     safe-mode: true # このフィールドがtrueに設定されている場合、DMはデータソースのINSERTをターゲットデータベースのREPLACEに変更し、データソースのUPDATEをターゲットデータベースのDELETEとREPLACEに変更します。これは、テーブルスキーマに主キーまたは一意のインデックスが含まれている場合、DMLステートメントが繰り返しインポートできるようにするためです。増分レプリケーションタスクを開始または再開した直後の最初の1分間に、DMは自動的にセーフモードを有効にします。
    ```

    上記のYAMLはマイグレーションタスクに必要な最小構成です。より多くの構成項目については、[DMの詳細タスク構成ファイル](/dm/task-configuration-file-full.md)を参照してください。

    マイグレーションタスクを開始する前に、エラーの発生確率を減らすために、構成がDMの要件を満たしていることを確認するために`check-task`コマンドを実行することをお勧めします:

    {{< copyable "shell-regular" >}}

    ```shell
    tiup dmctl --master-addr ${advertise-addr} check-task task.yaml
    ```

2. 以下のコマンドを実行してマイグレーションタスクを開始します:

    {{< copyable "shell-regular" >}}

    ```shell
    tiup dmctl --master-addr ${advertise-addr} start-task task.yaml
    ```

    上記のコマンドで使用されるパラメータは次のように説明されます:

    |パラメータ              |説明    |
    |-                      |-              |
    |`--master-addr`        |`dmctl`が接続されるクラスタ内の任意のDM-masterの`{advertise-addr}`。 例: 172.16.10.71:8261|
    |`start-task`           |マイグレーションタスクを開始します。|

    タスクの開始に失敗した場合は、プロンプトメッセージを確認して構成を修正します。その後、タスクを開始するために上記のコマンドを再実行できます。

    問題が発生した場合は、[DMエラーハンドリング](/dm/dm-error-handling.md)と[DM FAQ](/dm/dm-faq.md)を参照してください。

### マイグレーションタスクのステータスを確認
```shell
tiup dmctl --master-addr ${advertise-addr} query-status ${task-name}
```

組み合わせパラメーター`${advertise-addr}`を使用して`tiup dmctl`の`query-status`コマンドを実行して、DMクラスターが進行中の移行タスクを持っているかどうかを確認し、タスクの状態を表示します。

結果の詳細な解釈については、「[クエリステータス](/dm/dm-query-status.md)」を参照してください。

### タスクの監視とログの表示

移行タスクの過去の状態やその他の内部メトリクスを表示するには、以下の手順を実行してください。

TiUPを使用してDMを展開する際にPrometheus、Alertmanager、Grafanaが展開されている場合、展開時に指定されたIPアドレスとポートを使用してGrafanaにアクセスできます。その後、DMダッシュボードを選択して、DM関連の監視メトリクスを表示できます。

DMが実行されている時、DM-worker、DM-master、そしてdmctlは関連する情報をログに出力します。これらのコンポーネントのログディレクトリは次のとおりです。

- DM-master: DM-masterプロセスパラメーター`--log-file`で指定されます。TiUPを使用してDMを展開する場合、ログディレクトリはデフォルトで`/dm-deploy/dm-master-8261/log/`です。
- DM-worker: DM-workerプロセスパラメーター`--log-file`で指定されます。TiUPを使用してDMを展開する場合、ログディレクトリはデフォルトで`/dm-deploy/dm-worker-8262/log/`です。

## 次の手順

- [データ移行タスクを一時停止する](/dm/dm-pause-task.md)
- [データ移行タスクを再開する](/dm/dm-resume-task.md)
- [データ移行タスクを停止する](/dm/dm-stop-task.md)
- [クラスターのデータソースとタスク構成のエクスポートとインポート](/dm/dm-export-import-config.md)
- [失敗したDDLステートメントの処理](/dm/handle-failed-ddl-statements.md)