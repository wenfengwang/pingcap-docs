---
title: TiDB Lightning FAQs
summary: TiDB Lightningに関するよくある質問（FAQ）とその回答について学びます。
aliases: ['/docs/dev/tidb-lightning/tidb-lightning-faq/','/docs/dev/faq/tidb-lightning/']

# TiDB Lightning FAQs

このドキュメントでは、TiDB Lightningに関するよくある質問（FAQ）とその回答についてリストします。

## TiDB LightningがサポートするTiDB/TiKV/PDクラスターの最小バージョンは何ですか？

TiDB Lightningのバージョンはクラスターと同じでなければなりません。Localバックエンドモードを使用する場合、最も古い利用可能なバージョンは4.0.0です。ImporterバックエンドモードまたはTiDBバックエンドモードを使用する場合、最も古い利用可能なバージョンは2.0.9ですが、3.0の安定バージョンを使用することをお勧めします。

## TiDB Lightningは複数のスキーマ（データベース）のインポートをサポートしていますか？

はい。

## ターゲットデータベースの権限要件は何ですか？

権限の詳細については、「[TiDB Lightningの使用の前提条件](/tidb-lightning/tidb-lightning-requirements.md)」を参照してください。

## TiDB Lightningが1つのテーブルのインポート中にエラーが発生した場合、他のテーブルに影響しますか？プロセスは中断されますか？

1つのテーブルでエラーが発生した場合、残りのテーブルは通常通り処理されます。

## TiDB Lightningを適切に再起動するにはどうすればよいですか？

1. [`tidb-lightning`プロセスを停止](#tidb-lightningプロセスを停止)します。
2. 新しい`tidb-lightning`タスクを開始します：以前の開始コマンド（たとえば`nohup tiup tidb-lightning -config tidb-lightning.toml`）を実行します。

## インポートされたデータの整合性をどのように確保しますか？

TiDB Lightningはデフォルトでローカルデータソースとインポートされたテーブルにチェックサムを実行します。チェックサムが不一致の場合、プロセスは中断されます。これらのチェックサム情報はログから読み取れます。

また、ターゲットテーブルで[`ADMIN CHECKSUM TABLE`](/sql-statements/sql-statement-admin-checksum-table.md) SQLコマンドを実行して、インポートされたデータのチェックサムを再計算することもできます。

```sql
ADMIN CHECKSUM TABLE `schema`.`table`;
```

```
+---------+------------+---------------------+-----------+-------------+
| Db_name | Table_name | Checksum_crc64_xor  | Total_kvs | Total_bytes |
+---------+------------+---------------------+-----------+-------------+
| schema  | table      | 5505282386844578743 |         3 |          96 |
+---------+------------+---------------------+-----------+-------------+
1 行が選択されました (0.01 秒)
```

## TiDB Lightningはどのようなデータソース形式をサポートしていますか？

TiDB Lightningは以下をサポートしています。

- [Dumpling](/dumpling-overview.md)によってエクスポートされたファイル、CSVファイル、およびAmazon Auroraによって生成された[Apache Parquetファイル](/migrate-aurora-to-tidb.md)のインポート。
- ローカルディスクまたはAmazon S3ストレージからのデータの読み取り。

## TiDB Lightningはスキーマとテーブルの作成をスキップできますか？

v5.1から、TiDB Lightningは自動的にダウンストリームでスキーマとテーブルを認識できます。v5.1より前のTiDB Lightningを使用する場合は、`tidb-lightning.toml`の`[mydumper]`セクションで`no-schema = true`を設定する必要があります。これにより、TiDB Lightningは`CREATE TABLE`をスキップし、対象データベースからメタデータを直接取得します。テーブルが実際に存在しない場合、TiDB Lightningはエラーで終了します。

## 無効なデータのインポートを禁止するにはどうすればよいですか？

Strict SQLモードを有効にすることで、無効なデータのインポートを禁止できます。

デフォルトでTiDB Lightningが使用する[`sql_mode`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html)は、「"ONLY_FULL_GROUP_BY,NO_AUTO_CREATE_USER"」であり、日付`1970-00-00`などの無効なデータを許可します。

無効なデータのインポートを禁止するには、`tidb-lightning.toml`の`[tidb]`セクションで`sql-mode`設定を`"STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION"`に変更する必要があります。

```toml
...
[tidb]
sql-mode = "STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION"
...
```

## `tidb-lightning`プロセスを停止するにはどうすればよいですか？

`tidb-lightning`プロセスを停止するには、部署方法に応じて対応する操作を選択できます。

- 手動デプロイメントの場合：`tidb-lightning`が前景で実行されている場合は、<kbd>Ctrl</kbd>+<kbd>C</kbd>を押して終了します。それ以外の場合は、`ps aux | grep tidb-lightning`コマンドを使用してプロセスID（PID）を取得し、`kill -2 ${PID}`コマンドを使用してプロセスを終了します。

## TiDB Lightningは1ギガビットネットワークカードで使用できますか？

TiDB Lightningは10ギガビットネットワークカードとの組み合わせが最適です。

1ギガビットネットワークカードは、120 MB/sの合計帯域幅を提供できるため、すべての対象TiKVストアで共有する必要があります。TiDB Lightningは、物理インポートモードで1ギガビットネットワークのすべての帯域幅を簡単に飽和させ、PDによる連絡ができなくなるため、クラスターを停止させることがあります。

## なぜTiDB LightningはターゲットTiKVクラスターでこれほど多くの空き容量を必要とするのですか？

デフォルトの設定である3つのレプリカを持つ場合、ターゲットTiKVクラスターの空間要件はデータソースのサイズの6倍です。付加的な「2」の倍数は、次の要因がデータソースに反映されていないための保守的な見積もりです。

- インデックスが占めるスペース
- RocksDBにおけるスペースの増幅

## TiDB Lightningに関連するすべての中間データを完全に破壊するにはどうすればよいですか？

1. チェックポイントファイルを削除します。

    {{< copyable "shell-regular" >}}

    ```sh
    tidb-lightning-ctl --config conf/tidb-lightning.toml --checkpoint-remove=all
    ```

    何らかの理由でこのコマンドを実行できない場合は、`/tmp/tidb_lightning_checkpoint.pb`ファイルを手動で削除してください。

2. Localバックエンドを使用している場合は、構成の`sorted-kv-dir`ディレクトリを削除します。

3. 必要に応じて、TiDBクラスターで作成されたすべてのテーブルおよびデータベースを削除します。

4. 残存するメタデータをクリーンアップします。次のいずれかの条件が存在する場合、メタデータスキーマを手動でクリーンアップする必要があります。

    - TiDB Lightning v5.1.xおよびv5.2.xバージョンの場合、`tidb-lightning-ctl`コマンドはターゲットクラスターのメタデータスキーマをクリーンアップしません。手動でクリーンアップする必要があります。
    - チェックポイントファイルを手動で削除した場合は、ダウンストリームのメタデータスキーマを手動でクリーンアップする必要があります。そうしないと、後続のインポートの正確性が影響を受ける可能性があります。

    次のコマンドを使用してメタデータをクリーンアップします。

    {{< copyable "sql" >}}

    ```sql
    DROP DATABASE IF EXISTS `lightning_metadata`;
    ```

## TiDB Lightningのランタイムゴルーチン情報を取得するにはどうすればよいですか？

1. TiDB Lightningの構成ファイルで[`status-port`](/tidb-lightning/tidb-lightning-configuration.md#tidb-lightning-configuration)が指定されている場合は、この手順はスキップしてください。それ以外の場合は、TiDB Lightninigに`status-port`を有効にするためにUSR1シグナルを送信する必要があります。

    `ps`などのコマンドを使用してTiDB LightningのプロセスID（PID）を取得し、次のコマンドを実行します。

    {{< copyable "shell-regular" >}}

    ```sh
    kill -USR1 <lightning-pid>
    ```

    TiDB Lightningのログを確認してください。`starting HTTP server` / `start HTTP server` / `started HTTP server`のログに、新しく有効になった`status-port`が表示されます。

2. `http://<lightning-ip>:<status-port>/debug/pprof/goroutine?debug=2`にアクセスしてゴルーチン情報を取得してください。

## TiDB LightningはSQLの配置ルールと互換性がないのはなぜですか？

TiDB Lightningは[SQLの配置ルール](/placement-rules-in-sql.md)と互換性がありません。TiDB Lightningが配置ポリシーを含むデータをインポートするとエラーが発生します。

理由は以下の通りです。

SQLの配置ルールの目的は、特定のTiKVノードのデータ位置をテーブルまたはパーティションレベルで制御することです。TiDB LightningはテキストファイルからデータをターゲットTiDBクラスターにインポートします。データファイルが配置ルールの定義を含むようにエクスポートされている場合、TiDB Lightningは、定義に基づいて対象クラスターに対応する配置ルールポリシーを作成する必要があります。ソースクラスターとターゲットクラスターでトポロジが異なる場合、これは問題を引き起こす可能性があります。

ソースクラスターには次のようなトポロジがあるとします。

![TiDB Lightning FAQ - source cluster topology](/media/lightning-faq-source-cluster-topology.jpg)

ソースクラスターには次のような配置ポリシーがあります。

```sql
CREATE PLACEMENT POLICY p1 PRIMARY_REGION="us-east" REGIONS="us-east,us-west";
```

**シチュエーション1：** ターゲットクラスターには3つのレプリカがあり、トポロジがソースクラスターと異なる場合。このような場合、TiDB Lightningが配置ポリシーをターゲットクラスターに作成すると、エラーが報告されません。しかし、ターゲットクラスターのセマンティクスは間違っています。

![TiDB Lightning FAQ - situation 1](/media/lightning-faq-situation-1.jpg)

**シチュエーション2:** ターゲットクラスターは、フォロワーレプリカを別のTiKVノードに配置し、リージョン"us-mid"にあり、トポロジーにリージョン"us-west"を持っていない。このような場合、ターゲットクラスターで配置ポリシーを作成する際に、TiDB Lightningはエラーを報告します。

![TiDB Lightning FAQ - シチュエーション2](/media/lightning-faq-situation-2.jpg)

**回避策:**

TiDB LightningでSQLの配置ルールを使用するには、ターゲットのTiDBクラスターで関連するラベルとオブジェクトが**データをターゲットテーブルにインポートする前に**作成されていることを確認する必要があります。なぜならば、SQLの配置ルールはPDおよびTiKVレイヤーで動作するため、TiDB Lightningはインポートされたデータを格納するために使用すべきTiKVを見つけるための必要な情報を取得できます。このように、このSQLの配置ルールはTiDB Lightningには透過的です。

手順は以下の通りです:

1. データ分布のトポロジーを計画する。
2. TiKVおよびPDに必要なラベルを構成する。
3. 配置ルールポリシーを作成し、作成したポリシーをターゲットテーブルに適用する。
4. TiDB Lightningを使用して、ターゲットテーブルにデータをインポートする。

## TiDB LightningとDumplingを使用してスキーマをコピーする方法

あるスキーマから新しいスキーマにスキーマ定義とテーブルデータの両方をコピーしたい場合は、このセクションの手順に従います。この例では、`test`スキーマを`test2`という新しいスキーマにコピーする方法について学びます。

1. `-B test`を使用して必要なスキーマのみを選択して元のスキーマのバックアップを作成します。

    ```
    tiup dumpling -B test -o /tmp/bck1
    ```

2. 以下の内容を持つ`/tmp/tidb-lightning.toml`ファイルを作成します:

    ```toml
    [tidb]
    host = "127.0.0.1"
    port = 4000
    user = "root"
    
    [tikv-importer]
    backend = "tidb"
    
    [mydumper]
    data-source-dir = "/tmp/bck1"
    
    [[mydumper.files]]
    pattern = '^[a-z]*\.(.*)\.[0-9]*\.sql$'
    schema = 'test2'
    table = '$1'
    type = 'sql'
    
    [[mydumper.files]]
    pattern = '^[a-z]*\.(.*)\-schema\.sql$'
    schema = 'test2'
    table = '$1'
    type = 'table-schema'
    ```

    この構成ファイルでは、`schema = 'test2'`を設定して、元のダンプで使用されたスキーマ名と異なるスキーマ名を使用したい場合に利用します。ファイル名はテーブルの名前を決定するために使用されます。

3. この構成ファイルを使用してインポートを実行します。

    ```
    tiup tidb-lightning -config /tmp/tidb-lightning.toml
    ```