---
title: 大規模データセットのMySQLシャードをTiDB Cloudに移行およびマージする
summary: 大規模データセットのMySQLシャードをTiDB Cloudに移行およびマージする方法について学びます。

# 大規模データセットのMySQLシャードをTiDB Cloudに移行およびマージする

このドキュメントでは、異なるパーティションから大規模なMySQLデータセット（例えば、1 TiBを超える）をTiDB Cloudに移行およびマージする方法について説明します。完全なデータ移行後、[TiDBデータ移行（DM）](https://docs.pingcap.com/tidb/stable/dm-overview)を使用して、ビジネスニーズに応じた増分移行を実行できます。

このドキュメントの例では、複数のMySQLインスタンス間で複雑なシャード移行タスクが使用され、自動インクリメントプライマリキーでの競合の処理が含まれます。また、この例のシナリオは、単一のMySQLインスタンス内の異なるシャードテーブルからのデータのマージにも適用されます。

## 例の環境情報

このセクションでは、例に使用される上流クラスタ、DM、および下流クラスタの基本情報について説明します。

### 上流クラスタ

上流クラスタの環境情報は次のとおりです。

- MySQLバージョン: MySQL v5.7.18
- MySQLインスタンス1:
    - スキーマ `store_01` およびテーブル `[sale_01, sale_02]`
    - スキーマ `store_02` およびテーブル `[sale_01, sale_02]`
- MySQLインスタンス2:
    - スキーマ `store_01` およびテーブル `[sale_01, sale_02]`
    - スキーマ `store_02` およびテーブル `[sale_01, sale_02]`
- テーブル構造:

  ```sql
  CREATE TABLE sale_01 (
  id bigint(20) NOT NULL auto_increment,
  uid varchar(40) NOT NULL,
  sale_num bigint DEFAULT NULL,
  PRIMARY KEY (id),
  UNIQUE KEY ind_uid (uid)
  );
  ```

### DM

DMのバージョンはv5.3.0です。TiDB DMを手動で展開する必要があります。詳細な手順については、[TiUPを使用したDMクラスタの展開](https://docs.pingcap.com/tidb/stable/deploy-a-dm-cluster-using-tiup)を参照してください。

### 外部ストレージ

このドキュメントでは、Amazon S3を例として使用しています。

### 下流クラスタ

シャードされたスキーマおよびテーブルは、`store.sales`のテーブルにマージされます。

## MySQLからTiDB Cloudへの完全なデータ移行を実行する

次の手順は、MySQLのシャードの完全なデータをTiDB Cloudに移行およびマージする手順です。

以下の例では、テーブル内のデータを **CSV** フォーマットでエクスポートするだけです。

### ステップ1. Amazon S3バケット内でディレクトリを作成する

Amazon S3バケット内で、1階層目に`store`（データベースのレベルに対応）、2階層目に`sales`（テーブルのレベルに対応）を作成します。そして、`sales`内にそれぞれのMySQLインスタンスに対応する3階層目のディレクトリを作成します。例:

- MySQLインスタンス1のデータを`s3://dumpling-s3/store/sales/instance01/`に移行
- MySQLインスタンス2のデータを`s3://dumpling-s3/store/sales/instance02/`に移行

複数のインスタンス間でシャードが存在する場合、各データベースごとに1階層目のディレクトリを作成し、各シャードテーブルごとに2階層目のディレクトリを作成します。その後、簡単に管理するために各MySQLインスタンスに3階層目のディレクトリを作成します。例えば、MySQLインスタンス1とMySQLインスタンス2から`stock_N.product_N`テーブルをTiDB Cloudの`stock.products`テーブルにマージしたい場合、以下のディレクトリを作成できます:

- `s3://dumpling-s3/stock/products/instance01/`
- `s3://dumpling-s3/stock/products/instance02/`

### ステップ2. Dumplingを使用してデータをAmazon S3にエクスポートする

Dumplingのインストール方法については、[Dumplingの概要](https://docs.pingcap.com/tidb/stable/dumpling-overview)を参照してください。

Dumplingを使用してデータをAmazon S3にエクスポートする際には、以下に注意してください:

- 上流クラスタでbinlogを有効にしてください。
- 正しいAmazon S3ディレクトリとリージョンを選択してください。
- `-t`オプションを設定して適切な並列性を選択し、上流クラスタへの影響を最小限に抑えるか、バックアップデータベースから直接エクスポートしてください。このパラメータの使用方法については、[Dumplingのオプションリスト](https://docs.pingcap.com/tidb/stable/dumpling-overview#option-list-of-dumpling)を参照してください。
- `--filetype csv`および`--no-schemas`に適切な値を設定してください。これらのパラメータの使用方法については、[Dumplingのオプションリスト](https://docs.pingcap.com/tidb/stable/dumpling-overview#option-list-of-dumpling)を参照してください。

以下の手順でデータをAmazon S3にエクスポートしてください:

1. Amazon S3バケットの`AWS_ACCESS_KEY_ID`と`AWS_SECRET_ACCESS_KEY`を取得してください。

    ```shell
    [root@localhost ~]# export AWS_ACCESS_KEY_ID={your_aws_access_key_id}
    [root@localhost ~]# export AWS_SECRET_ACCESS_KEY= {your_aws_secret_access_key}
    ```

2. MySQLインスタンス1からAmazon S3バケット内の`s3://dumpling-s3/store/sales/instance01/`ディレクトリにデータをエクスポートしてください。

    ```shell
    [root@localhost ~]# tiup dumpling -u {username} -p {password} -P {port} -h {mysql01-ip} -B store_01,store_02 -r 20000 --filetype csv --no-schemas -o "s3://dumpling-s3/store/sales/instance01/" --s3.region "ap-northeast-1"
    ```

    パラメータの詳細については、[Dumplingのオプションリスト](https://docs.pingcap.com/tidb/stable/dumpling-overview#option-list-of-dumpling)を参照してください。

3. MySQLインスタンス2からAmazon S3バケット内の`s3://dumpling-s3/store/sales/instance02/`ディレクトリにデータをエクスポートしてください。

    ```shell
    [root@localhost ~]# tiup dumpling -u {username} -p {password} -P {port} -h {mysql02-ip} -B store_01,store_02 -r 20000 --filetype csv --no-schemas -o "s3://dumpling-s3/store/sales/instance02/" --s3.region "ap-northeast-1"
    ```

詳細な手順については、[Amazon S3クラウドストレージにデータをエクスポートする](https://docs.pingcap.com/tidb/stable/dumpling-overview#export-data-to-amazon-s3-cloud-storage)を参照してください。

### ステップ3. TiDB Cloudクラスタにスキーマを作成する

TiDB Cloudクラスタで以下のスキーマを作成してください:

```sql
mysql> CREATE DATABASE store;
Query OK, 0 rows affected (0.16 sec)
mysql> use store;
Database changed
```

この例では、上流テーブル `sale_01` および `sale_02` の列 `id` は自動増分プライマリキーです。下流のデータベースでシャードされたテーブルをマージする際に競合が発生する可能性があります。以下のSQLステートメントを実行して、ID列をプライマリキーの代わりに通常のインデックスとして設定してください。

```sql
mysql> CREATE TABLE `sales` (
         `id` bigint(20) NOT NULL ,
         `uid` varchar(40) NOT NULL,
         `sale_num` bigint DEFAULT NULL,
         INDEX (`id`),
         UNIQUE KEY `ind_uid` (`uid`)
        );
Query OK, 0 rows affected (0.17 sec)
```

このような競合を解決するための解決策について詳細は、[カラムからPRIMARY KEY属性を削除する](https://docs.pingcap.com/tidb/stable/shard-merge-best-practices#remove-the-primary-key-attribute-from-the-column)を参照してください。

### ステップ4. Amazon S3アクセスを設定する

[Amazon S3アクセスの設定](/tidb-cloud/config-s3-and-gcs-access.md#configure-amazon-s3-access)の手順に従って、ソースデータにアクセスするためのロールARNを取得してください。

以下の例は、キーポリシーの設定だけを記載しています。Amazon S3のパスは各自の値に置き換えてください。

```yaml
{
   "Version": "2012-10-17",
   "Statement": [
       {
           "Sid": "VisualEditor0",
           "Effect": "Allow",
           "Action": [
               "s3:GetObject",
               "s3:GetObjectVersion"
           ],
           "Resource": [
```json
[
   {
       "Sid": "VisualEditor0",
       "Effect": "Allow",
       "Action": "s3:ListBucket",
       "Resource": "arn:aws:s3:::dumpling-s3"
   },
   {
       "Sid": "VisualEditor1",
       "Effect": "Allow",
       "Action": [
           "s3:ListBucket",
           "s3:GetBucketLocation"
       ],
       "Resource": "arn:aws:s3:::dumpling-s3"
   }
]
```

### ステップ5. データインポートタスクを実行する

Amazon S3アクセスの構成後、TiDB Cloudコンソールで以下の手順に従ってデータインポートタスクを実行できます。

1. ターゲットクラスターの **Import** ページを開きます。

    1. [TiDB Cloudコンソール](https://tidbcloud.com/)にログインし、プロジェクトの [**Clusters**](https://tidbcloud.com/console/clusters) ページに移動します。

        > **ヒント：**
        >
        > 複数のプロジェクトを持っている場合は、左下の <MDSvgIcon name="icon-left-projects" /> をクリックして別のプロジェクトに切り替えることができます。

    2. ターゲットクラスターの名前をクリックして、その概要ページに移動し、左側のナビゲーションペインで **Import** をクリックします。

2. **Import** ページで、右上の **Import Data** をクリックし、**From S3** を選択します。

3. **Import from S3** ページで、以下の情報を入力します：

    - **データ形式**：**CSV** を選択します。
    - **Bucket URI**：ソースデータのバケットURIを入力します。この例では、テーブルに対応する2番目の階層ディレクトリを使用し、たとえば`s3://dumpling-s3/store/sales`とします。これにより、TiDB CloudはMySQLインスタンスのすべてのデータを`store.sales`に一括インポートおよびマージします。
    - **Role ARN**：取得したロールARNを入力します。

    バケットの場所がクラスターと異なる場合は、クロスリージョンの準拠を確認してください。**次へ**をクリックします。

    TiDB Cloudは指定したバケットURIのデータにアクセスできるかどうかを検証を開始します。検証後、TiDB Cloudはデフォルトのファイル命名パターンを使用してデータソース内のファイルをすべてスキャンし、次のページの左側にスキャン概要結果を返します。`AccessDenied`エラーが表示された場合は、[Troubleshoot Access Denied Errors during Data Import from S3](/tidb-cloud/troubleshoot-import-access-denied-error.md)を参照してください。

4. ファイルパターンを修正し、必要に応じてテーブルフィルタールールを追加します。

    - **ファイルパターン**：特定のパターンに一致するCSVファイルを単一のターゲットテーブルにインポートする場合は、ファイルパターンを修正します。

        > **注意：**
        >
        > この機能を使用すると、1つのインポートタスクでは1つのテーブルにのみデータをインポートできます。異なるテーブルにデータをインポートする場合は、異なるターゲットテーブルを指定してそれぞれ数回インポートする必要があります。

        ファイルパターンを修正するには、**修正**をクリックし、次のフィールドでCSVファイルと単一のターゲットテーブルとの間のカスタムマッピングルールを指定し、**Scan**をクリックします。

        - **ソースファイル名**：インポートするCSVファイルの名前に一致するパターンを入力します。1つのCSVファイルのみを持つ場合は、ここにファイル名を直接入力します。CSVファイルの名前には拡張子".csv"が含まれている必要があります。

            例：

            - `my-data?.csv`：`my-data`で始まるすべてのCSVファイルと1文字（例：`my-data1.csv`および`my-data2.csv`など）は、同じターゲットテーブルにインポートされます。
            - `my-data*.csv`：`my-data`で始まるすべてのCSVファイルは、同じターゲットテーブルにインポートされます。

        - **ターゲットテーブル名**：TiDB Cloudのターゲットテーブルの名前を入力します。`${db_name}.${table_name}`形式である必要があります。たとえば、`mydb.mytable`です。このフィールドは特定の1つのテーブル名のみを受け入れるため、ワイルドカードはサポートされていません。

    - **テーブルフィルター**：インポートするテーブルをフィルタリングしたい場合は、この領域に1つ以上の[テーブルフィルター](/table-filter.md#syntax)ルールを指定できます。

5. **次へ**をクリックします。

6. **プレビュー**ページでデータのプレビューを確認できます。プレビューされたデータが期待通りでない場合は、**Click here to edit csv configuration**リンクをクリックして、セパレーター、デリミター、ヘッダー、`backslash escape`、`trim last separator`などのCSV固有の構成を更新します。

    > **注意：**
    >
    > セパレーターやデリミター、NULLの構成では、英数字の他に特定の特殊文字を使用できます。サポートされている特殊文字には、`\t`、`\b`、`\n`、`\r`、`\f`、`\u0001`が含まれます。

7. **Start Import**をクリックします。

8. インポートの進行状況が**Finished**と表示されたら、インポートされたテーブルを確認します。

データのインポートが完了したら、TiDB CloudのAmazon S3アクセスを削除する場合は、追加したポリシーを単純に削除してください。

## MySQLからTiDB Cloudへの増分データレプリケーションを実行する

指定された位置からのアップストリームクラスターのバイナリログに基づくデータ変更をTiDB Cloudにレプリケートするには、TiDB Data Migration (DM)を使用して増分レプリケーションを実行できます。

### 開始する前に

増分データをマイグレーションし、MySQLシャードをTiDB Cloudにマージする場合は、TiDB CloudはまだMySQLシャードのマイグレーションとマージをサポートしていないため、TiDB DMを手動で展開する必要があります。詳細な手順については、[TiUPを使用してDMクラスターを展開する](https://docs.pingcap.com/tidb/stable/deploy-a-dm-cluster-using-tiup)を参照してください。

### ステップ1. データソースを追加する

1. 新しいデータソースファイル `dm-source1.yaml` を作成して、DMにアップストリームデータソースを構成します。以下の内容を追加します：

    ```yaml
    # MySQL Configuration.
    source-id: "mysql-replica-01"
    # DM-workerがGTID（Global Transaction Identifier）を使用してバイナリログをプルするかどうかを指定します。
    # 前提条件として、上流のMySQLでGTIDがすでに有効になっている必要があります。
    # 上流のデータベースサービスを異なるノード間でマスターを自動的に切り替えるように構成している場合は、GTIDを有効にする必要があります。
    enable-gtid: true
    from:
     host: "${host}"           # 例: 192.168.10.101
     user: "user01"
     password: "${password}"   # 平文のパスワードはサポートされていますが、推奨されません。平文のパスワードを暗号化するには、dmctl encryptを使用することをお勧めします。
     port: ${port}             # 例: 3307
    ```

2. 別の新しいデータソースファイル `dm-source2.yaml` を作成し、以下の内容を追加します：

    ```yaml
    # MySQL Configuration.
    source-id: "mysql-replica-02"
    # DM-workerがGTID（Global Transaction Identifier）を使用してバイナリログをプルするかどうかを指定します。
    # 前提条件として、上流のMySQLでGTIDがすでに有効になっている必要があります。
    # 上流のデータベースサービスを異なるノード間でマスターを自動的に切り替えるように構成している場合は、GTIDを有効にする必要があります。
    enable-gtid: true
    from:
     host: "192.168.10.102"
     user: "user02"
     password: "${password}"
     port: 3308
    ```

3. ターミナルで以下のコマンドを実行します。`tiup dmctl`を使用して、最初のデータソース構成をDMクラスターに読み込みます：

    ```shell
    [root@localhost ~]# tiup dmctl --master-addr ${advertise-addr} operate-source create dm-source1.yaml
    ```

    上記のコマンドで使用されるパラメータは次のように説明されています：

    |パラメータ        |説明               |
    |-                |-                  |
    |`--master-addr`  |接続する`dmctl`が動作するクラスター内の任意のDM-masterノードの `{advertise-addr}`。例: 192.168.11.110:9261|
    |`operate-source create`|データソースをDMクラスターに読み込みます。|

    以下は例です：

    ```shell
    tiup is checking updates for component dmctl ...

    Starting component `dmctl`: /root/.tiup/components/dmctl/${tidb_version}/dmctl/dmctl /root/.tiup/components/dmctl/${tidb_version}/dmctl/dmctl --master-addr 192.168.11.110:9261 operate-source create dm-source1.yaml

    {
       "result": true,
       "msg": "",
       "sources": [
           {
               "result": true,
               "msg": "",
               "source": "mysql-replica-01",
               "worker": "dm-192.168.11.111-9262"
           }
       ]
    }

    ```

4. ターミナルで以下のコマンドを実行します。`tiup dmctl`を使用して、2番目のデータソース構成をDMクラスターに読み込みます：

    ```shell
    [root@localhost ~]# tiup dmctl --master-addr 192.168.11.110:9261 operate-source create dm-source2.yaml
    ```

    以下は例です：

    ```shell
    tiup is checking updates for component dmctl ...
```
    DMctlコンポーネントを開始中: /root/.tiup/components/dmctl/${tidb_version}/dmctl/dmctl /root/.tiup/components/dmctl/${tidb_version}/dmctl/dmctl --master-addr 192.168.11.110:9261 operate-source create dm-source2.yaml

    {
       "result": true,
       "msg": "",
       "sources": [
           {
               "result": true,
               "msg": "",
               "source": "mysql-replica-02",
               "worker": "dm-192.168.11.112-9262"
           }
       ]
    }
    ```

### ステップ2. レプリケーションタスクを作成する

1. レプリケーションタスクに`test-task1.yaml`ファイルを作成します。

2. DumplingによってエクスポートされたMySQLインスタンス1のメタデータファイルで開始地点を見つけます。例：

    ```toml
    Started dump at: 2022-05-25 10:16:26
    SHOW MASTER STATUS:
           Log: mysql-bin.000002
           Pos: 246546174
           GTID:b631bcad-bb10-11ec-9eee-fec83cf2b903:1-194801
    Finished dump at: 2022-05-25 10:16:27
    ```

3. DumplingによってエクスポートされたMySQLインスタンス2のメタデータファイルで開始地点を見つけます。例：

    ```toml
    Started dump at: 2022-05-25 10:20:32
    SHOW MASTER STATUS:
           Log: mysql-bin.000001
           Pos: 1312659
           GTID:cd21245e-bb10-11ec-ae16-fec83cf2b903:1-4036
    Finished dump at: 2022-05-25 10:20:32
    ```

4. タスク構成ファイル`test-task1`を編集し、各データソースの増分レプリケーションモードとレプリケーション開始地点を構成します。

    ```yaml
    ## ********* タスク構成 *********
    name: test-task1
    shard-mode: "pessimistic"
    # タスクモード。 「incremental」モードは増分データ移行のみを実行します。
    task-mode: incremental
    # timezone: "UTC"

    ## ******** データソース構成 **********
    ##（オプション）すでに完全データ移行で移行されたデータを増分的にレプリケートする必要がある場合は、増分データ移行エラーを避けるためにセーフモードを有効にする必要があります。
    ##  このシナリオは、次の場合に一般的です。完全移行データがデータソースの一貫性スナップショットに属さず、その後、DMが完全移行よりも早い位置から増分データをレプリケートし始めるケースです。
    syncers:           # SYNC処理ユニットの実行構成。
     global:           # 構成名。
       safe-mode: false # # このフィールドがtrueに設定されている場合、DMはデータソースのINSERTを対象データベース用にREPLACEに変更し、
                        # # およびデータソースのUPDATEを対象データベース用にDELETEおよびREPLACEに変更します。
                        # # これは、テーブルスキーマにプライマリキーまたはユニークインデックスが含まれる場合、DMLステートメントが繰り返しインポートされることを保証するためです。
                        # # 増分移行タスクの開始または再開の最初の1分間、DMは自動的にセーフモードを有効にします。
    mysql-instances:
    - source-id: "mysql-replica-01"
       block-allow-list:  "bw-rule-1"
       route-rules: ["store-route-rule", "sale-route-rule"]
       filter-rules: ["store-filter-rule", "sale-filter-rule"]
       syncer-config-name: "global"
       meta:
         binlog-name: "mysql-bin.000002"
         binlog-pos: 246546174
         binlog-gtid: "b631bcad-bb10-11ec-9eee-fec83cf2b903:1-194801"
    - source-id: "mysql-replica-02"
       block-allow-list:  "bw-rule-1"
       route-rules: ["store-route-rule", "sale-route-rule"]
       filter-rules: ["store-filter-rule", "sale-filter-rule"]
       syncer-config-name: "global"
       meta:
         binlog-name: "mysql-bin.000001"
         binlog-pos: 1312659
         binlog-gtid: "cd21245e-bb10-11ec-ae16-fec83cf2b903:1-4036"

    ## ******** ターゲットTiDBクラスターの構成（TiDB Cloud上） **********
    target-database:       # TiDB Cloud上のターゲットTiDBクラスター
     host: "tidb.xxxxxxx.xxxxxxxxx.ap-northeast-1.prod.aws.tidbcloud.com"
     port: 4000
     user: "root"
     password: "${password}"  # パスワードが空でない場合、dmctl暗号化された暗号を使用することをお勧めします。

    ## ******** 機能構成 **********
    routes:
     store-route-rule:
       schema-pattern: "store_*"
       target-schema: "store"
     sale-route-rule:
       schema-pattern: "store_*"
       table-pattern: "sale_*"
       target-schema: "store"
       target-table:  "sales"
    filters:
     sale-filter-rule:
       schema-pattern: "store_*"
       table-pattern: "sale_*"
       events: ["truncate table", "drop table", "delete"]
       action: Ignore
     store-filter-rule:
       schema-pattern: "store_*"
       events: ["drop database"]
       action: Ignore
    block-allow-list:
     bw-rule-1:
       do-dbs: ["store_*"]

    ## ******** チェック項目を無視する **********
    ignore-checking-items: ["table_schema","auto_increment_ID"]
    ```

詳細なタスク構成については、[DMタスク構成](https://docs.pingcap.com/tidb/stable/task-configuration-file-full)を参照してください。

データレプリケーションタスクをスムーズに実行するために、DMはタスクの開始時に自動的に事前チェックをトリガーし、チェック結果を返します。事前チェックが合格した後に、DMはレプリケーションを開始します。手動で事前チェックをトリガーするには、`check-task`コマンドを実行します：

```shell
[root@localhost ~]# tiup dmctl --master-addr 192.168.11.110:9261 check-task dm-task.yaml
```

以下は出力の例です：

```shell
tiup is checking updates for component dmctl ...

Starting component `dmctl`: /root/.tiup/components/dmctl/${tidb_version}/dmctl/dmctl /root/.tiup/components/dmctl/${tidb_version}/dmctl/dmctl --master-addr 192.168.11.110:9261 check-task dm-task.yaml

{
   "result": true,
   "msg": "check pass!!!"
}
```

### ステップ3. レプリケーションタスクを開始する

データレプリケーションタスクを開始するには、`tiup dmctl`を使用して次のコマンドを実行します：

```shell
[root@localhost ~]# tiup dmctl --master-addr ${advertise-addr}  start-task dm-task.yaml
```

上記のコマンドで使用されるパラメータの説明は次のとおりです：

|パラメータ              |説明    |
|-                      |-              |
|`--master-addr`        |`dmctl`が接続されるクラスター内のどのDM-masterノードの`{advertise-addr}`です。 例：192.168.11.110:9261|
|`start-task`           |移行タスクを開始します。|

以下は出力の例です：

```shell
tiup is checking updates for component dmctl ...

Starting component `dmctl`: /root/.tiup/components/dmctl/${tidb_version}/dmctl/dmctl /root/.tiup/components/dmctl/${tidb_version}/dmctl/dmctl --master-addr 192.168.11.110:9261 start-task dm-task.yaml

{
   "result": true,
   "msg": "",
   "sources": [
       {
           "result": true,
           "msg": "",
           "source": "mysql-replica-01",
           "worker": "dm-192.168.11.111-9262"
       },

       {
           "result": true,
           "msg": "",
           "source": "mysql-replica-02",
           "worker": "dm-192.168.11.112-9262"
       }
   ],
   "checkResult": ""
}
```

タスクが開始に失敗した場合は、プロンプトメッセージを確認して構成を修正してください。その後、タスクを開始するために上記のコマンドを再実行できます。

問題が発生した場合は、[DMエラーハンドリング](https://docs.pingcap.com/tidb/stable/dm-error-handling)と[DM FAQ](https://docs.pingcap.com/tidb/stable/dm-faq)を参照してください。

### ステップ4. レプリケーションタスクの状態を確認する

DMクラスターに進行中のレプリケーションタスクがあるかどうかや、タスクの状態を確認するには、`tiup dmctl`を使用して`query-status`コマンドを実行します：

```shell
[root@localhost ~]# tiup dmctl --master-addr 192.168.11.110:9261 query-status test-task1
```

以下は出力の例です：

```shell
{
   "result": true,
   "msg": "",
   "sources": [
       {
           "result": true,
           "msg": "",
           "sourceStatus": {
```json
      {
          "source": "mysql-replica-01",
          "worker": "dm-192.168.11.111-9262",
          "result": null,
          "relayStatus": null
      },
      "subTaskStatus": [
          {
              "name": "test-task1",
              "stage": "Running",
              "unit": "Sync",
              "result": null,
              "unresolvedDDLLockID": "",
              "sync": {
                  "totalEvents": "4048",
                  "totalTps": "3",
                  "recentTps": "3",
                  "masterBinlog": "(mysql-bin.000002, 246550002)",
                  "masterBinlogGtid": "b631bcad-bb10-11ec-9eee-fec83cf2b903:1-194813",
                  "syncerBinlog": "(mysql-bin.000002, 246550002)",
                  "syncerBinlogGtid": "b631bcad-bb10-11ec-9eee-fec83cf2b903:1-194813",
                  "blockingDDLs": [
                  ],
                  "unresolvedGroups": [
                  ],
                  "synced": true,
                  "binlogType": "remote",
                  "secondsBehindMaster": "0",
                  "blockDDLOwner": "",
                  "conflictMsg": ""
              }
          }
      ],
      {
          "result": true,
          "msg": "",
          "sourceStatus": {
              "source": "mysql-replica-02",
              "worker": "dm-192.168.11.112-9262",
              "result": null,
              "relayStatus": null
          },
          "subTaskStatus": [
              {
                  "name": "test-task1",
                  "stage": "Running",
                  "unit": "Sync",
                  "result": null,
                  "unresolvedDDLLockID": "",
                  "sync": {
                      "totalEvents": "33",
                      "totalTps": "0",
                      "recentTps": "0",
                      "masterBinlog": "(mysql-bin.000001, 1316487)",
                      "masterBinlogGtid": "cd21245e-bb10-11ec-ae16-fec83cf2b903:1-4048",
                      "syncerBinlog": "(mysql-bin.000001, 1316487)",
                      "syncerBinlogGtid": "cd21245e-bb10-11ec-ae16-fec83cf2b903:1-4048",
                      "blockingDDLs": [
                      ],
                      "unresolvedGroups": [
                      ],
                      "synced": true,
                      "binlogType": "remote",
                      "secondsBehindMaster": "0",
                      "blockDDLOwner": "",
                      "conflictMsg": ""
                  }
              }
          }
      }
  ]
}
```
```json
      {
          "source": "mysql-レプリカ-01",
          "worker": "dm-192.168.11.111-9262",
          "result": null,
          "relayStatus": null
      },
      "subTaskStatus": [
          {
              "name": "test-task1",
              "stage": "実行中",
              "unit": "同期",
              "result": null,
              "unresolvedDDLLockID": "",
              "sync": {
                  "totalEvents": "4048",
                  "totalTps": "3",
                  "recentTps": "3",
                  "masterBinlog": "(mysql-bin.000002, 246550002)",
                  "masterBinlogGtid": "b631bcad-bb10-11ec-9eee-fec83cf2b903:1-194813",
                  "syncerBinlog": "(mysql-bin.000002, 246550002)",
                  "syncerBinlogGtid": "b631bcad-bb10-11ec-9eee-fec83cf2b903:1-194813",
                  "blockingDDLs": [
                  ],
                  "unresolvedGroups": [
                  ],
                  "synced": true,
                  "binlogType": "リモート",
                  "secondsBehindMaster": "0",
                  "blockDDLOwner": "",
                  "conflictMsg": ""
              }
          }
      ],
      {
          "result": true,
          "msg": "",
          "sourceStatus": {
              "source": "mysql-レプリカ-02",
              "worker": "dm-192.168.11.112-9262",
              "result": null,
              "relayStatus": null
          },
          "subTaskStatus": [
              {
                  "name": "test-task1",
                  "stage": "実行中",
                  "unit": "同期",
                  "result": null,
                  "unresolvedDDLLockID": "",
                  "sync": {
                      "totalEvents": "33",
                      "totalTps": "0",
                      "recentTps": "0",
                      "masterBinlog": "(mysql-bin.000001, 1316487)",
                      "masterBinlogGtid": "cd21245e-bb10-11ec-ae16-fec83cf2b903:1-4048",
                      "syncerBinlog": "(mysql-bin.000001, 1316487)",
                      "syncerBinlogGtid": "cd21245e-bb10-11ec-ae16-fec83cf2b903:1-4048",
                      "blockingDDLs": [
                      ],
                      "unresolvedGroups": [
                      ],
                      "synced": true,
                      "binlogType": "リモート",
                      "secondsBehindMaster": "0",
                      "blockDDLOwner": "",
                      "conflictMsg": ""
                  }
              }
          }
      }
  ]
}
```