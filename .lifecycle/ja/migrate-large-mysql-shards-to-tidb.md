---
title: 大規模データセットのMySQLシャードをTiDBに移行およびマージする
summary: DumplingとTiDB Lightningを使用してMySQLからTiDBに大規模なシャードデータセットを移行およびマージする方法、およびTiDB DMタスクを構成して異なるMySQLシャードからの増分データ変更をTiDBにレプリケートする方法を学びます。

# 大規模データセットのMySQLシャードをTiDBに移行およびマージする

異なるパーティションから大規模なMySQLデータセット(たとえば1 TiB以上)をTiDBに移行し、移行中にビジネスからTiDBクラスタのすべての書き込み操作を一時停止できる場合、移行を迅速に行うためにTiDB Lightningを使用できます。移行後、TiDB DMを使用してビジネスのニーズに応じて増分レプリケーションを実行できます。この文書での「大規模データセット」とは通常、1 TiB以上のデータを指します。

この文書では、移行の手順全体を具体的な例を使用して説明します。

MySQLシャードのデータサイズが1 TiB未満の場合は、[小規模データセットのMySQLシャードをTiDBに移行およびマージする](/migrate-small-mysql-shards-to-tidb.md)で説明されている手順に従うことができます。これにより、完全および増分移行がサポートされ、手順が簡単になります。

この文書の例では、`my_db1`および`my_db2`という2つのデータベースを想定しています。`my_db1`から`table1`と`table2`、`my_db2`から`table3`と`table4`の2つのテーブルをDumplingを使用してエクスポートし、その後、TiDBにそれらの4つのエクスポートされたテーブルを同じ`table5`にインポートおよびマージします。

この文書では、次の手順でデータを移行できます。

1. Dumplingを使用してフルデータをエクスポートします。この例では、2つの上流データベースからそれぞれ2つのテーブルをエクスポートします。

    - `my_db1`から`table1`および`table2`をエクスポートします。
    - `my_db2`から`table3`および`table4`をエクスポートします。

2. TiDB Lightningを起動して、TiDBの`mydb.table5`にデータを移行します。

3. (オプション) TiDB DMを使用して増分レプリケーションを実行します。

## 前提条件

移行タスクの準備をするために、次の文書を参照してください。

- [TiUPを使用してDMクラスタをデプロイする](/dm/deploy-a-dm-cluster-using-tiup.md)
- [TiUPを使用してDumplingおよびLightningをデプロイする](/migration-tools.md)
- [Dumplingのダウンストリーム特権要件](/dumpling-overview.md#export-data-from-tidb-or-mysql)
- [TiDB Lightningのダウンストリーム特権要件](/tidb-lightning/tidb-lightning-requirements.md)
- [TiDB Lightningのダウンストリームストレージスペース要件](/tidb-lightning/tidb-lightning-requirements.md)
- [DM-workerが必要とする特権](/dm/dm-worker-intro.md)

### シャードされたテーブルの競合を確認する

この移行には、異なるシャードテーブルからデータをマージする場合、プライマリキーまたはユニークインデックスの競合が発生する可能性があります。したがって、移行前にビジネスの観点から現在のシャードスキームを綿密に調査し、競合を回避する方法を見つける必要があります。詳細については、[複数のシャードテーブルでプライマリキーまたはユニークインデックスの競合を処理する](/dm/shard-merge-best-practices.md#handle-conflicts-between-primary-keys-or-unique-indexes-across-multiple-sharded-tables)を参照してください。以下に簡単な説明があります。

1〜4のテーブルが次のようなテーブル構造を持つと仮定します。

{{< copyable "sql" >}}

```sql
CREATE TABLE `table1` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `sid` bigint(20) NOT NULL,
  `pid` bigint(20) NOT NULL,
  `comment` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `sid` (`sid`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1
```

これらの4つのテーブルでは、`id`列がプライマリキーです。これにより、異なるシャードテーブルが重複した`id`範囲を生成し、移行中にターゲットテーブルでプライマリキーの競合が発生する可能性があります。一方で、`sid`列はシャーディングキーであり、インデックスがグローバルに一意であることを保証します。したがって、ターゲットの`table5`で`id`列の一意制約を削除して、データのマージの競合を避けることができます。

{{< copyable "sql" >}}

```sql
CREATE TABLE `table5` (
  `id` bigint(20) NOT NULL,
  `sid` bigint(20) NOT NULL,
  `pid` bigint(20) NOT NULL,
  `comment` varchar(255) DEFAULT NULL,
  INDEX (`id`),
  UNIQUE KEY `sid` (`sid`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1
```

## ステップ1. Dumplingを使用してフルデータをエクスポートする

複数のシャードテーブルが同じ上流MySQLインスタンスに格納されている場合は、Dumplingの`-f`パラメータを使用して1回の操作でそれらをエクスポートできます。

シャードテーブルが異なるMySQLインスタンスに格納されている場合は、それぞれをDumplingを使用してエクスポートし、エクスポート結果を同じ親ディレクトリに配置できます。

次の例では、これらの両方の方法を使用し、その後、エクスポートされたデータを同じ親ディレクトリに格納します。

まず、次のコマンドを実行して`my_db1`から`table1`および`table2`をDumplingを使用してエクスポートします。

{{< copyable "shell-regular" >}}

```shell
tiup dumpling -h ${ip} -P 3306 -u root -t 16 -r 200000 -F 256MB -B my_db1 -f 'my_db1.table[12]' -o ${data-path}/my_db1
```

上記のコマンドのパラメータについては、Dumplingパラメータの詳細については、[Dumpling概要](/dumpling-overview.md)を参照してください。

`-F`オプションを使用して、バックアッププロセスが大きすぎる単一テーブルによる中断を回避するために推奨されます。

その後、次のコマンドを実行して`my_db2`から`table3`および`table4`をDumplingを使用してエクスポートします。パスは`${data-path}/my_db1`ではなく`${data-path}/my_db2`になります。

{{< copyable "shell-regular" >}}

```shell
tiup dumpling -h ${ip} -P 3306 -u root -t 16 -r 200000 -F 256MB -B my_db2 -f 'my_db2.table[34]' -o ${data-path}/my_db2
```

前述の手順の後、すべてのソースデータテーブルが`${data-path}`ディレクトリにエクスポートされます。同じディレクトリにエクスポートされたデータを配置することで、後続のTiDB Lightningによるインポートが便利になります。

増分レプリケーションのために必要な開始位置情報は、それぞれ`${data-path}`ディレクトリの`my_db1`および`my_db2`のサブディレクトリの`metadata`ファイルに含まれています。これらはDumplingによって自動生成されるメタ情報ファイルです。増分レプリケーションを行うには、これらのファイルにバイナリログの位置情報を記録する必要があります。

## ステップ2. TiDB Lightningを起動してエクスポートされたフルデータをインポートする

移行のためにTiDB Lightningを起動する前に、チェックポイントの処理方法を理解し、必要に応じて適切な方法を選択することをお勧めします。

### チェックポイント

大量のデータを移行するということは通常数時間または数日かかります。長時間実行されるプロセスが予期せず中断される可能性があります。すでに一部のデータがインポートされている場合でも、すべてをやり直さなければならないことは非常にfrustratingです。

---

幸いにも、TiDB Lightningには`checkpoints`という機能が提供されており、TiDB Lightningは定期的にインポートの進捗状況を`checkpoints`として保存することができます。これにより、中断されたインポートタスクを最新の`checkpoint`から再開することができます。

TiDB Lightningタスクが復旧不能なエラー（例:データの破損）によってクラッシュした場合、`checkpoint`から再開するのではなく、エラーを報告してタスクを終了します。インポートされたデータの安全を確保するためには、他の手順に進む前に`tidb-lightning-ctl`コマンドを使用してこれらのエラーを解決する必要があります。オプションには以下があります:

* --checkpoint-error-destroy: このオプションは、これらのテーブルの既存のデータをすべて破壊して、失敗したターゲットテーブルにデータを最初から再度インポートすることを可能にします。
* --checkpoint-error-ignore: マイグレーションに失敗した場合、このオプションを使用して、エラーステータスをエラーが発生しなかったかのようにクリアできます。
* --checkpoint-remove: このオプションは、エラーに関係なくすべての`checkpoints`を単純にクリアします。

詳細については、[TiDB Lightning Checkpoints](/tidb-lightning/tidb-lightning-checkpoints.md)を参照してください。

### ターゲットスキーマの作成

ダウンストリームの`mydb.table5`を作成します。

{{< copyable "sql" >}}

```sql
CREATE TABLE `table5` (
  `id` bigint(20) NOT NULL,
  `sid` bigint(20) NOT NULL,
  `pid` bigint(20) NOT NULL,
  `comment` varchar(255) DEFAULT NULL,
  INDEX (`id`),
  UNIQUE KEY `sid` (`sid`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1
```

### マイグレーションタスクの開始

次の手順に従って`tidb-lightning`の開始します。

1. tomlファイルを編集します。以下の例では`tidb-lightning.toml`が使用されています。

    ```toml
    [lightning]
    # Logs
    level = "info"
    file = "tidb-lightning.log"

    [mydumper]
    data-source-dir = ${data-path}

    [tikv-importer]
    # Choose a local backend.
    # "local": The default mode. It is used for large data volumes greater than 1 TiB. During migration, downstream TiDB cannot provide services.
    # "tidb": Used for data volumes less than 1 TiB. During migration, downstream TiDB can provide services normally.
    # For more information, see [TiDB Lightning Backends](https://docs.pingcap.com/tidb/stable/tidb-lightning-backends)
    backend = "local"
    # Set the temporary directory for the sorted key value pairs. It must be empty.
    # The free space must be greater than the size of the dataset to be imported.
    # It is recommended that you use a directory different from `data-source-dir` to get better migration performance by consuming I/O resources exclusively.
    sorted-kv-dir = "${sorted-kv-dir}"

    # Set the renaming rules ('routes') from source to target tables, in order to support merging different table shards into a single target table. Here you migrate `table1` and `table2` in `my_db1`, and `table3` and `table4` in `my_db2`, to the target `table5` in downstream `my_db`.
    [[mydumper.files]]
    pattern = '(^|/)my_db1\.table[1-2]\..*\.sql$'
    schema = "my_db"
    table = "table5"
    type = "sql"

    [[mydumper.files]]
    pattern = '(^|/)my_db2\.table[3-4]\..*\.sql$'
    schema = "my_db"
    table = "table5"
    type = "sql"

    # Information of the target TiDB cluster. For example purposes only. Replace the IP address with your IP address.
    [tidb]
    # Information of the target TiDB cluster.
    # Values here are only for illustration purpose. Replace them with your own values.
    host = ${host}           # For example: "172.16.31.1"
    port = ${port}           # For example: 4000
    user = "${user_name}"    # For example: "root"
    password = "${password}" # For example: "rootroot"
    status-port = ${status-port} # The table information is read from the status port. For example: 10080
    # the IP address of the PD cluster. TiDB Lightning gets some information through the PD cluster.
    # For example: "172.16.31.3:2379".
    # When backend = "local", make sure that the values of status-port and pd-addr are correct. Otherwise an error will occur.
    pd-addr = "${ip}:${port}"
    ```

2. `tidb-lightning`を実行します。プログラムをシェルで直接呼び出すと、SIGHUPシグナルを受信した後にプロセスが予期せず終了する可能性があります。プログラムを`nohup`や`screen`や`tiup`などのツールを使用して実行し、プロセスをシェルのバックグラウンドに配置することをお勧めします。S3からのマイグレーションの場合、Amazon S3バックエンドストアにアクセス権限を持っているアカウントのSecretKeyおよびAccessKeyを環境変数としてLightningノードに渡す必要があります。`~/.aws/credentials`からの資格情報ファイルの読み取りもサポートされています。例:

    {{< copyable "shell-regular" >}}

    ```shell
    export AWS_ACCESS_KEY_ID=${access_key}
    export AWS_SECRET_ACCESS_KEY=${secret_key}
    nohup tiup tidb-lightning -config tidb-lightning.toml > nohup.out 2>&1 &
    ```

3. マイグレーションタスクを開始した後、次の方法のいずれかを使用して進捗状況を確認できます:

   - `grep`ツールを使用してログファイル内のキーワード `progress` を検索します。デフォルトでは、進捗状況を報告するメッセージは5分ごとにログファイルにフラッシュされます。
   - 監視ダッシュボードを介して進捗状況を表示します。詳細については、[TiDB Lightning Monitoring](/tidb-lightning/monitor-tidb-lightning.md)を参照してください。
   - Webページを介して進捗状況を表示します。詳細については、[Web Interface](/tidb-lightning/tidb-lightning-web-interface.md)を参照してください。

TiDB Lightningがインポートを完了したら、自動的に終了します。`tidb-lightning.log`に`the whole procedure completed`が含まれているかどうかを確認してください。含まれている場合、インポートは成功です。含まれていない場合、インポートでエラーが発生しています。エラーメッセージに従ってエラーを修正してください。

> **注意:**
>
> マイグレーションが成功したかどうかにかかわらず、ログの最後の行は常に`tidb lightning exit`になります。これは単にTiDB Lightningが正常に終了したことを意味し、インポートタスクが正常に完了したことを保証するものではありません。

マイグレーション中に問題が発生した場合は、[TiDB Lightning FAQs](/tidb-lightning/tidb-lightning-faq.md)を参照してください。

## ステップ3. （オプション）DMを使用して増分レプリケーションを実行する

指定されたソースデータベースからTiDBにビンログに基づくデータ変更を複製するには、TiDB DMを使用して増分レプリケーションを実行できます。

### データソースの追加

DMに上流データソースを追加するために新しいデータソースファイル`source1.yaml`を作成し、次の内容を追加してください:

{{< copyable "" >}}

```yaml
# Configuration.
source-id: "mysql-01" # 必ずユニークである必要があります。

# DM-workerがGTID（Global Transaction Identifier）でビンログを取得するかどうかを指定します。
# 前提条件として、上流MySQLで既にGTIDを有効にしている必要があります。
# 上流データベースサービスを異なるノード間で自動的にマスターを切り替えるように構成した場合、GTIDを有効にする必要があります。
enable-gtid: true

from:
  host: "${host}"           # 例: 172.16.10.81
  user: "root"
  password: "${password}"   # 平文パスワードはサポートされていますが、お勧めしません。平文パスワードを暗号化するためには、dmctl encryptを使用することをお勧めします。
  port: ${port}             # 例: 3306
```

次のコマンドをターミナルで実行します。`tiup dmctl`を使用してデータソース構成をDMクラスタにロードします：

{{< copyable "shell-regular" >}}

```shell
tiup dmctl --master-addr ${advertise-addr} operate-source create source1.yaml
```

パラメータは以下のように説明されています。

|パラメータ      | 説明 |
|-              |-            |
|`--master-addr`         | dmctlが接続するクラスタ内のどのDM-masterノードの{advertise-addr}です。例: 172.16.10.71:8261 |
| `operate-source create` | DMクラスタにデータソースをロードします。 |

すべてのMySQL上流インスタンスがDMにデータソースとして追加されるまで、上記の手順を繰り返してください。

### レプリケーションタスクの作成

各データソースの増分レプリケーションモードとレプリケーション開始ポイントを構成するために、`task.yaml`というタスク構成ファイルを編集して、タスクを作成してください。

{{< copyable "" >}}

```yaml
name: task-test               # タスクの名前。グローバルに一意である必要があります。
task-mode: incremental        # タスクのモード。"incremental"は完全データの移行をスキップし、増分レプリケーションのみを実行します。
# シャード化されたテーブルからの増分レプリケーションの際に必要です。デフォルトでは、「pessimistic」モードが使用されます。
# 好ましくないモードの原則と使用制限に深く理解している場合、「optimistic」モードも使用できます。
```
# 詳細については、「[分散テーブルからのデータマージおよびマイグレーション](https://docs.pingcap.com/zh/tidb/dev/feature-shard-merge/)」を参照してください。

shard-mode: "pessimistic"

# ターゲットTiDBデータベースインスタンスのアクセス情報を構成します：
target-database:              # ターゲットデータベースインスタンス
  host: "${host}"             # 例：127.0.0.1
  port: 4000
  user: "root"
  password: "${password}"     # dmctlで暗号化されたパスワードの使用を推奨します。

# 同期が必要なテーブルを構成するために、ブロック許可リストを使用します：
block-allow-list:             # データソース内のテーブルに一致するフィルター規則のセットで、どのテーブルをマイグレーションするかを決定します。 DMのバージョンがv2.0.0-beta.2以下の場合、black-white-listを使用してください。
  bw-rule-1:                  # ブロックおよび許可リストのルールID。
    do-dbs: ["my_db1"]        # マイグレーションするデータベース。ここでは、インスタンス1のmy_db1とインスタンス2のmy_db2を別々のルールとして構成し、インスタンス1のmy_db2がレプリケートされないようにする方法を示します。
  bw-rule-2:
    do-dbs: ["my_db2"]
routes:                               # 異なるシャードテーブルを単一のターゲットテーブルにマージするための、上流から下流のテーブルに対するテーブル名前変更ルール（「ルート」）。
  route-rule-1:                       # ルール名。my_db1からテーブル1とテーブル2をmy_db.table5にマイグレーションおよびマージする。
    schema-pattern: "my_db1"          # 上流スキーマ名に一致するルール。ワイルドカード"*"および"?"をサポートしています。
    table-pattern: "table[1-2]"       # 上流テーブル名に一致するルール。ワイルドカード"*"および"?"をサポートしています。
    target-schema: "my_db"            # ターゲットスキーマ名。
    target-table: "table5"            # ターゲットテーブル名。
  route-rule-2:                       # ルール名。my_db2からテーブル3とテーブル4をmy_db.table5にマイグレーションおよびマージする。
    schema-pattern: "my_db2"
    table-pattern: "table[3-4]"
    target-schema: "my_db"
    target-table: "table5"

# データソースを構成します。以下は2つのデータソースを使用した例です。
mysql-instances:
  - source-id: "mysql-01"             # データソースID。これはsource1.yaml内のsource-idです。
    block-allow-list: "bw-rule-1"     # 上記のブロックおよび許可リスト構成を使用します。 インスタンス1の `my_db1` をレプリケートします。
    route-rules: ["route-rule-1"]     # 上記で構成したルーティングルールを使用して上流テーブルをマージします。
#       syncer-config-name: "global"  # 以下のsyncers構成を使用します。
    meta:                             # `task-mode`が`incremental`であり、下流のデータベースチェックポイントが存在しない場合に、binlog複製が開始する位置。チェックポイントが存在する場合、チェックポイントが使用されます。`meta`構成項目も下流のデータベースチェックポイントも存在しない場合、マイグレーションは上流の最新のbinlog位置から開始します。
      binlog-name: "${binlog-name}"   # Step 1の`${data-path}/my_db1/metadata` に記録されたログの場所。binlog-name + binlog-posまたはbinlog-gtidを指定できます。上流データベースサービスが異なるノード間でマスターを自動的に切り替えるように構成されている場合は、ここでbinlog GTIDを使用してください。
      binlog-pos: ${binlog-position}
      # binlog-gtid:                  " 例：09bec856-ba95-11ea-850a-58f2b4af5188:1-9"
  - source-id: "mysql-02"             # データソースID。これはsource1.yaml内のsource-idです。
    block-allow-list: "bw-rule-2"     # 上記のブロックおよび許可リスト構成を使用します。 インスタンス2の `my_db2` をレプリケートします。
    route-rules: ["route-rule-2"]     # 上記で構成したルーティングルールを使用します。

#       syncer-config-name: "global"  # 以下のsyncers構成を使用します。
    meta:                             # `task-mode`がincrementalであり、下流のデータベースにチェックポイントが存在しない場合、マイグレーションの開始ポイント。チェックポイントがある場合は、チェックポイントが使用されます。
      # binlog-name: "${binlog-name}"   # Step 1の`${data-path}/my_db2/metadata` に記録されたログの場所。binlog-name + binlog-posまたはbinlog-gtidを指定できます。上流データベースサービスが異なるノード間でマスターを自動的に切り替えるように構成されている場合は、ここでbinlog GTIDを使用してください。
      # binlog-pos: ${binlog-position}
      binlog-gtid: "09bec856-ba95-11ea-850a-58f2b4af5188:1-9"
#（オプション）完全なマイグレーションでカバーされたデータの一部の変更を増分的にレプリケートする必要がある場合、データマイグレーション中にデータマイグレーションエラーを回避するためにセーフモードを有効にする必要があります。
#マイグレーション対象の完全なスナップショットの一部でない場合や、増分データが完全にマイグレーションされたデータより前の場所からレプリケートされる場合は、TiDB DMは、レプリケーション操作中にDMLを繰り返し適用できるよう、INSERTをREPLACEに変更し、UPDATEをテーブル構造内のプライマリキーまたはユニークインデックス存在する場合は、DELETEとREPLACEのペアに変更します。
#TiDB DMは、増分レプリケーションタスクを開始または再開する1分前にセーフモードを自動的に開始します。
#syncers:           # sync処理ユニットの実行パラメーター。
#  global:           # 構成名。
#trueに設定すると、DMはINSERTをREPLACEに変更し、テーブルの構造にプライマリキーやユニークインデックスが存在する場合、UPDATEをDELETEとREPLACEのペアに変更して、データソースのレプリケーション操作中にDMLを繰り返し適用できます。
#そのため、完全マイグレーションされたデータがデータソースの一貫したスナップショットの一部でない場合や、増分データがマイグレーションされたデータよりも前の場所からレプリケートされる場合に、データマイグレーション中にデータマイグレーションエラーを回避する必要があります。
#TiDB DMは、増分レプリケーションタスクを開始または再開する1分前にセーフモードを自動的に開始します。
#    safe-mode: true
```