---
title: TiSparkユーザーガイド
summary: TiSparkを使用して、オンライン取引と分析の両方を一括で提供するHTAPソリューションとして機能することで、HTAPソリューションを提供するために、Apache SparkをTiDB/TiKVの上で実行するための薄いレイヤーを構築し、複雑なOLAPクエリに回答します。

エイリアス: ['/docs/dev/tispark-overview/','/docs/dev/reference/tispark/','/docs/dev/get-started-with-tispark/','/docs/dev/how-to/get-started/tispark/','/docs/dev/how-to/deploy/tispark/','/tidb/dev/get-started-with-tispark/','/tidb/stable/get-started-with-tispark']

---

# TiSparkユーザーガイド

![TiSparkアーキテクチャ](/media/tispark-architecture.png)

## TiSpark vs TiFlash

[TiSpark](https://github.com/pingcap/tispark) は、Apache SparkをTiDB / TiKVの上で実行するための薄いレイヤーで、複雑なOLAPクエリに回答します。これはSparkプラットフォームと分散TiKVクラスタの両方を活用し、TiDB（分散OLTPデータベース）とシームレスに結合して、オンライン取引と分析の両方を提供するHTAPソリューションとして機能します。

[TiFlash](/tiflash/tiflash-overview.md) はHTAPを可能にする別のツールです。TiFlashとTiSparkの両方は、複数のホストを使用してOLTPデータでのOLAPクエリの実行を可能にします。TiFlashはデータを列形式で保存するため、より効率的な分析クエリを可能にします。TiFlashとTiSparkは一緒に使用できます。

## TiSparkとは

TiSparkは、TiKVクラスタとPDクラスタに依存します。また、Sparkクラスタをセットアップする必要があります。このドキュメントでは、TiSparkのセットアップと使用方法について簡単に紹介します。Apache Sparkの基本的な知識が必要です。詳細は、[Apache Sparkのウェブサイト](https://spark.apache.org/docs/latest/index.html)を参照してください。

Spark Catalystエンジンと深く統合することで、TiSparkは計算を正確に制御します。これにより、SparkがTiKVからデータを効率的に読み取ることができます。また、TiSparkはインデックス検索をサポートし、高速なポイントクエリを可能にします。TiSparkは計算をTiKVにプッシュすることでデータの処理量を減らし、データクエリを加速します。同時に、TiSparkはTiDB組込みの統計情報を使用して最適なクエリプランを選択できます。

TiSparkとTiDBを使用すると、ETLの構築や保守をせずに、同じプラットフォームでトランザクションと分析タスクを実行できます。これにより、システムアーキテクチャが簡素化され、メンテナンスコストが削減されます。

TiDBで次のSparkエコシステムツールを使用できます。
- TiSpark：データ分析とETL
- TiKV：データの取得
- スケジューリングシステム：レポートの生成

また、TiSparkはTiKVへの分散書き込みをサポートしています。JDBCおよびSparkを使用したTiDBへの書き込みと比べて、TiKVへの分散書き込みはトランザクションを実行でき（すべてのデータが正常に書き込まれるか、すべての書き込みが失敗するか）、書き込みが速くなります。

> **警告:**
>
> TiSparkはTiKVに直接アクセスするため、TiDBサーバーで使用されるアクセス制御メカニズムはTiSparkに適用されません。TiSpark v2.5.0から、TiSparkはユーザー認証と認可をサポートしています。詳細については、[セキュリティ](/tispark-overview.md#security)を参照してください。

## 必要条件

+ TiSparkはSpark >= 2.3をサポートしています。
+ TiSparkにはJDK 1.8およびScala 2.11/2.12が必要です。
+ TiSparkは`YARN`、`Mesos`、`Standalone`などのSparkモードで実行できます。

## Sparkの推奨デプロイメント構成

> **警告:**
>
> この[ドキュメント](/tispark-deployment-topology.md)に記載されているようにTiUPを使用してTiSparkをデプロイすることは非推奨です。

TiSparkはSparkのTiDBコネクタであるため、使用するには実行中のSparkクラスタが必要です。

このドキュメントでは、Sparkのデプロイに関する基本的なアドバイスを提供します。詳細なハードウェアの推奨事項については、[Spark公式ウェブサイト](https://spark.apache.org/docs/latest/hardware-provisioning.html)を参照してください。

Sparkクラスタの独立したデプロイメントの場合：

+ Sparkには32 GBのメモリを割り当てることをお勧めします。メモリのうち少なくとも25%はオペレーティングシステムとバッファキャッシュに予約してください。
+ Sparkには1台あたり8から16個のコアを割り当てることをお勧めします。まず、すべてのCPUコアをSparkに割り当てる必要があります。

次は、`spark-env.sh`構成に基づいた例です：

```
SPARK_EXECUTOR_MEMORY = 32g
SPARK_WORKER_MEMORY = 32g
SPARK_WORKER_CORES = 8
```

## TiSparkの取得

TiSparkはTiKVの読み取りおよび書き込みの能力を提供するSpark用のサードパーティのjarパッケージです。

### mysql-connector-jの取得

GPLライセンスの制限により、`mysql-connector-java`の依存関係はもはや提供されません。

次のバージョンのTiSparkのjarには、`mysql-connector-java`が含まれなくなります。

- TiSpark > 3.0.1
- TiSpark > 2.5.1（TiSpark 2.5.x用）
- TiSpark > 2.4.3（TiSpark 2.4.x用）

ただし、TiSparkは書き込みと認証に`mysql-connector-java`が必要です。そのような場合、次のいずれかの方法で`mysql-connector-java`を手動でインポートする必要があります。

- `mysql-connector-java`をspark jarsファイルに配置します。

- Sparkジョブを提出する際に`mysql-connector-java`をインポートします。次の例を参照してください：

```
spark-submit --jars tispark-assembly-3.0_2.12-3.1.0-SNAPSHOT.jar,mysql-connector-java-8.0.29.jar
```

### TiSparkバージョンの選択

TiDBおよびSparkのバージョンに応じてTiSparkのバージョンを選択できます。

| TiSparkバージョン | TiDB、TiKV、PDバージョン | Sparkバージョン | Scalaバージョン |
| ---------------  |------------------------| ------------- | ------------- |
| 2.4.x-scala_2.11 | 5.x, 4.x               | 2.3.x, 2.4.x   | 2.11          |
| 2.4.x-scala_2.12 | 5.x, 4.x               | 2.4.x         | 2.12          |
| 2.5.x            | 5.x, 4.x               | 3.0.x, 3.1.x   | 2.12          |
| 3.0.x            | 5.x, 4.x               | 3.0.x, 3.1.x, 3.2.x|2.12|
| 3.1.x            | 6.x, 5.x, 4.x          | 3.0.x, 3.1.x, 3.2.x, 3.3.x|2.12|

TiSpark 2.4.4、2.5.2、3.0.2、3.1.1は最新安定版であり、強くお勧めします。

### TiSpark jarの取得

次のいずれかの方法でTiSpark jarを取得できます。

- [Maven Central](https://search.maven.org/)から取得し、[`pingcap`](http://search.maven.org/#search%7Cga%7C1%7Cpingcap)を検索します。
- [TiSparkリリース](https://github.com/pingcap/tispark/releases)から取得します。
- 以下の手順でソースからビルドします

> **注：**
>
> 現在、java8がTiSparkをビルドする唯一の選択肢です。チェックするためには、mvn -versionを実行してください。

```
git clone https://github.com/pingcap/tispark.git
```

TiSparkのルートディレクトリで次のコマンドを実行します。

```
// -Dmaven.test.skip=trueを追加してテストをスキップ
mvn clean install -Dmaven.test.skip=true
// または、プロパティを追加してsparkのバージョンを指定できます
mvn clean install -Dmaven.test.skip=true -Pspark3.2.1
```

### TiSpark jarのアーティファクトID

TiSparkのアーティファクトIDはTiSparkのバージョンごとに異なります。

| TiSparkバージョン               | Artifact ID                                        |
|-------------------------------| -------------------------------------------------- |
| 2.4.x-\${scala_version}, 2.5.0 | tispark-assembly                                   |
| 2.5.1                         | tispark-assembly-\${spark_version}                  |
| 3.0.x, 3.1.x                  | tispark-assembly-\${spark_version}-\${scala_version} |

## インストールの開始

このドキュメントでは、spark-shellでTiSparkを使用する方法について説明します。

### spark-shellの起動

spark-shellでTiSparkを使用するには、次の構成を`spark-defaults.conf`に追加します。

```
spark.sql.extensions  org.apache.spark.sql.TiExtensions
spark.tispark.pd.addresses  ${your_pd_adress}
spark.sql.catalog.tidb_catalog  org.apache.spark.sql.catalyst.catalog.TiCatalog
spark.sql.catalog.tidb_catalog.pd.addresses  ${your_pd_adress}
```

`--jars`オプションを使用してspark-shellを起動します。

```
spark-shell --jars tispark-assembly-{version}.jar
```

### TiSparkバージョンの取得

spark-shellで次のコマンドを実行して、TiSparkのバージョン情報を取得できます。

```scala
spark.sql("select ti_version()").collect
```

### TiSparkを使用したデータの読み取り

```
Spark SQLを使用してTiKVからデータを読み取ることができます。

```scala
spark.sql("use tidb_catalog")
spark.sql("select count(*) from ${database}.${table}").show
```

### TiSparkを使用したデータの書き込み

ACIDが保証されているTiKVにデータを書き込むためにSpark DataSource APIを使用することができます。

```scala
val tidbOptions: Map[String, String] = Map(
  "tidb.addr" -> "127.0.0.1",
  "tidb.password" -> "",
  "tidb.port" -> "4000",
  "tidb.user" -> "root"
)

val customerDF = spark.sql("select * from customer limit 100000")

customerDF.write
.format("tidb")
.option("database", "tpch_test")
.option("table", "cust_test_select")
.options(tidbOptions)
.mode("append")
.save()
```

詳細については、[Data Source API User Guide](https://github.com/pingcap/tispark/blob/master/docs/features/datasource_api_userguide.md)を参照してください。

TiSpark 3.1以降では、Spark SQLを使用してもデータを書き込むことができます。詳細については、[insert SQL](https://github.com/pingcap/tispark/blob/master/docs/features/insert_sql_userguide.md)を参照してください。

### JDBC DataSourceを使用したデータの書き込み

TiSparkを使用せずにSpark JDBCを使用してTiDBに書き込むこともできます。

これはTiSparkの範囲外です。このドキュメントはここでのみ例を提供します。詳細については、[JDBC To Other Databases](https://spark.apache.org/docs/latest/sql-data-sources-jdbc.html)を参照してください。

```scala
import org.apache.spark.sql.execution.datasources.jdbc.JDBCOptions

val customer = spark.sql("select * from customer limit 100000")
// ソースをリパーティションしてノード間でのバランスを取り、同時実行性を上げる必要があるかもしれません
val df = customer.repartition(32)
df.write
.mode(saveMode = "append")
.format("jdbc")
.option("driver", "com.mysql.jdbc.Driver")
// ホストとポートをそれぞれの値に置き換えて、rewrite batchを使用するようにしてください
.option("url", "jdbc:mysql://127.0.0.1:4000/test?rewriteBatchedStatements=true")
.option("useSSL", "false")
// テスト結果によると、`150`に設定することが良い実践です
.option(JDBCOptions.JDBC_BATCH_INSERT_SIZE, 150)
.option("dbtable", s"cust_test_select") // ここにデータベース名とテーブル名を指定してください
.option("isolationLevel", "NONE") // isolationLevelをNONEに設定してください
.option("user", "root") // TiDBのユーザーをここに指定してください
.save()
```

大きな単一トランザクションを避けるために、`isolationLevel`を`NONE`に設定し、またTiDB OOMを避けるために、`ISOLATION LEVEL does not support`エラーも避けるために設定してください（現在TiDBは`REPEATABLE-READ`のみをサポートしています）。

### TiSparkを使用したデータの削除

Spark SQLを使用してTiKVからデータを削除することができます。

```
spark.sql("use tidb_catalog")
spark.sql("delete from ${database}.${table} where xxx")
```

詳細については、[delete feature](https://github.com/pingcap/tispark/blob/master/docs/features/delete_userguide.md)を参照してください。

### 他のデータソースとの連携

次のようにして、複数のカタログを使用して異なるデータソースからデータを読み取ることができます。

```
// Hiveから読み取り
spark.sql("select * from spark_catalog.default.t").show

// HiveのテーブルとTiDBのテーブルを結合
spark.sql("select t1.id,t2.id from spark_catalog.default.t t1 left join tidb_catalog.test.t t2").show
```

## TiSparkの設定

以下の表にある設定は、`spark-defaults.conf`に一緒に配置するか、他のSpark設定プロパティと同じ方法で渡すことができます。

| キー                                             | デフォルト値    | 説明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
|-------------------------------------------------|------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `spark.tispark.pd.addresses`                    | `127.0.0.1:2379` | PDクラスターのアドレス。コンマで分割されています。                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `spark.tispark.grpc.framesize`                  | `2147483647`     | gRPC応答の最大フレームサイズ（デフォルトは2G）。                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `spark.tispark.grpc.timeout_in_sec`             | `10`             | gRPCのタイムアウト時間（秒）。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `spark.tispark.plan.allow_agg_pushdown`         | `true`           | 集計がTiKVにプッシュされるかどうか（TiKVノードがビジーの場合）。                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| `spark.tispark.plan.allow_index_read`           | `true`           | プランニングでインデックスが有効かどうか（TiKVに大きな負荷をかける可能性があります）。                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `spark.tispark.index.scan_batch_size`           | `20000`          | 並列インデックススキャンでの行キーの数。                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `spark.tispark.index.scan_concurrency`          | `5`              | 行キーを取得するためのインデックススキャンの最大スレッド数（各JVM内のタスク間で共有）。                                                                                                                                                                                                                                                                                                                                                                                                              |
| `spark.tispark.table.scan_concurrency`          | `512`            | テーブルスキャンのための最大スレッド数（各JVM内のタスク間で共有）。                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `spark.tispark.request.command.priority`        | `Low`            | `Low`、`Normal`、`High`の値オプションがあります。この設定はTiKVに割り当てられるリソースに影響を与えます。`Low`が推奨されています。なぜなら、OLTPワークロードが妨害されないからです。                                                                                                                                                                                                                                                                                                                                                   |
| `spark.tispark.coprocess.codec_format`          | `chblock`        | コプロセッサのデフォルトのコーデック形式を維持します。使用可能なオプションは、`default`、`chblock`、`chunk`です。                                                                                                                                                                                                                                                                                                                                                                                                                |
| `spark.tispark.coprocess.streaming`             | `false`          | レスポンスの取得にストリーミングを使用するかどうか（実験的）。                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `spark.tispark.plan.unsupported_pushdown_exprs` |                  | カンマで区切られた式のリスト。非常に古いバージョンのTiKVを使用している場合、サポートされていない場合、一部の式のプッシュダウンを無効にする場合があります。                                                                                                                                                                                                                                                                                                                                                      |
| `spark.tispark.plan.downgrade.index_threshold`  | `1000000000`     | 1つのリージョンでのインデックススキャンの範囲が元のリクエストでこの限界を超える場合、このリージョンのリクエストをインデックススキャンではなく計画されたテーブルスキャンにダウングレードします。デフォルトでは、ダウングレードは無効になっています。                                                                                                                                                                                                                                                                                                           |
| `spark.tispark.show_rowid`                      | `false`          | IDが存在する場合、行IDを表示するかどうか。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `spark.tispark.db_prefix`                       |                  | すべてのTiDBのデータベースに対する追加プレフィックスを示す文字列。この文字列は、Hiveデータベースと同じ名前のTiDBのデータベースを区別します。                                                                                                                                                                                                                                                                                                                                                       |
| `spark.tispark.request.isolation.level`         | `SI`             | TiDBクラスターのロックを解決するかどうか。"RC"を使用すると、`tso`よりも新しいレコードの最新バージョンを取得し、ロックを無視します。"SI"を使用すると、解決されたロックがコミットされたかどうかに応じてレコードを取得します。                                                                                                                                                                                                                                                                                                                                                   |
| `spark.tispark.coprocessor.chunk_batch_size`    | `1024`           | コプロセッサからフェッチされた行。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `spark.tispark.isolation_read_engines`          | `tikv,tiflash`   | TiSparkの読み取り可能なエンジンのリスト、カンマで区切られています。リストされていないストレージエンジンは読み取られません。                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `spark.tispark.stale_read`                      | optional         | ステールリードタイムスタンプ（ミリ秒）。詳細については[こちら](https://github.com/pingcap/tispark/blob/master/docs/features/stale_read.md)を参照してください。                                                                                                                                                                                                                                                                                                                                                                                  |
| `spark.tispark.tikv.tls_enable`                 | `false`          | TiSpark TLSを有効にするかどうか。　                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `spark.tispark.tikv.trust_cert_collection`      |                  | TiKVクライアントの信頼できる証明書。例：`/home/tispark/config/root.pem` ファイルにはX.509証明書コレクションが含まれている必要があります。                                                                                                                                                                                                                                                                                                                                                                                          |
| `spark.tispark.tikv.key_cert_chain`             |                  | TiKVクライアントのX.509証明書チェーンファイル。例：`/home/tispark/config/client.pem`。                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `spark.tispark.tikv.key_file`                   |                  | TiKVクライアントのPKCS#8プライベートキーファイル。例：`/home/tispark/client_pkcs8.key`。                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `spark.tispark.tikv.jks_enable`                 | `false`          | X.509証明書の代わりにJAVAキーストアを使用するかどうか。                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `spark.tispark.tikv.jks_trust_path`             |                  | TiKVクライアント用のJKS形式証明書。`keytool`で生成されたもの。例：`/home/tispark/config/tikv-truststore`。                                                                                                                                                                                                                                                                                                                                                                                                  |
| `spark.tispark.tikv.jks_trust_password`         |                  | `spark.tispark.tikv.jks_trust_path` のパスワード。                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `spark.tispark.tikv.jks_key_path`               |                  | TiKVクライアント用のJKS形式のキー。`keytool` によって生成され、例えば、`/home/tispark/config/tikv-clientstore`。                                                                                                                                                                                                                                                                                                                                                                                                         |
| `spark.tispark.tikv.jks_key_password`           |                  | `spark.tispark.tikv.jks_key_path` のパスワード。                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `spark.tispark.jdbc.tls_enable`                 | `false`          | JDBCコネクタを使用する際にTLSを有効にするかどうか。                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `spark.tispark.jdbc.server_cert_store`          |                  | JDBCの信頼された証明書。`keytool` によって生成されたJava keystore（JKS）形式の証明書で、例えば、`/home/tispark/config/jdbc-truststore`。デフォルト値は "" で、これはTiSparkがTiDBサーバーを検証しないことを意味します。                                                                                                                                                                                                                                                                             |
| `spark.tispark.jdbc.server_cert_password`       |                  | `spark.tispark.jdbc.server_cert_store` のパスワード。                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `spark.tispark.jdbc.client_cert_store`          |                  | JDBCのPKCS#12証明書。`keytool` によって生成されたJKS形式の証明書で、例えば、`/home/tispark/config/jdbc-clientstore`。デフォルトは "" で、これはTiDBサーバーがTiSparkを検証しないことを意味します。                                                                                                                                                                                                                                                                                                             |
| `spark.tispark.jdbc.client_cert_password`       |                  | `spark.tispark.jdbc.client_cert_store` のパスワード。                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `spark.tispark.tikv.tls_reload_interval`        | `10s`            | 証明書を再読み込みするかどうかをチェックする間隔。デフォルト値は `10s`（10秒）。                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `spark.tispark.tikv.conn_recycle_time`          | `60s`            | TiKVとの期限切れのコネクションをクリーニングする間隔。証明書の再読み込みが有効な場合のみ有効です。デフォルト値は `60s`（60秒）。                                                                                                                                                                                                                                                                                                                                                           |
| `spark.tispark.host_mapping`                    |                  | ルートマップで、公開IPアドレスとイントラネットIPアドレスのマッピングを構成するために使用されます。TiDBクラスターがイントラネットで実行されている場合、外部のSparkクラスターがアクセスするために、イントラネットIPアドレスのセットを公開IPアドレスにマッピングできます。フォーマットは`{イントラネットIP1}:{公開IP1};{イントラネットIP2}:{公開IP2}`で、例えば、`192.168.0.2:8.8.8.8;192.168.0.3:9.9.9.9`。                                                                                                                         |
| `spark.tispark.new_collation_enable`            |                  | TiDBで[new collation](https://docs.pingcap.com/tidb/stable/character-set-and-collation#new-framework-for-collations)が有効になっている場合、この構成は `true` に設定できます。TiDBで`new collation`が有効になっていない場合、この構成は `false` に設定できます。この項目が構成されていない場合、TiSparkはTiDBバージョンに基づいて自動的に`new collation`を構成します。構成ルールは次の通りです。TiDBバージョンがv6.0.0以上の場合、`true` です。それ以外の場合、`false` です。 |
| `spark.tispark.replica_read`                    | `leader`         | 読み取り対象のレプリカのタイプ。値のオプションは `leader`、`follower`、`learner` です。複数のタイプを同時に指定でき、TiSparkは順序に従ってタイプを選択します。それ |
| `spark.tispark.replica_read.label`              |                  | 対象となるTiKVノードのラベル。フォーマットは `label_x=value_x,label_y=value_y` で、項目は論理連接で結ばれます。 |

### TLSの構成

TiSpark TLSはTiKVクライアントTLSとJDBCコネクタTLSの2つのパーツに分かれます。TiSparkでTLSを有効にするには、両方を構成する必要があります。 `spark.tispark.tikv.xxx` はTiKVクライアントがPDとTiKVサーバーとの間でTLS接続を作成するために使用されます。 `spark.tispark.jdbc.xxx` はJDBCがTiDBサーバーとの間でTLS接続を作成するために使用されます。

TiSpark TLSを有効にすると、 `tikv.trust_cert_collection`、`tikv.key_cert_chain`、`tikv.key_file` の構成か、`tikv.jks_enable`、`tikv.jks_trust_path`、`tikv.jks_key_path` の構成のいずれかを構成する必要があります。 `jdbc.server_cert_store` と `jdbc.client_cert_store` はオプションです。

TiSparkはTLSv1.2およびTLSv1.3のみをサポートしています。

* 以下は、TiKVクライアントでX.509証明書を使用してTLS構成を開く例です。

```
spark.tispark.tikv.tls_enable                                  true
spark.tispark.tikv.trust_cert_collection                       /home/tispark/root.pem
spark.tispark.tikv.key_cert_chain                              /home/tispark/client.pem
spark.tispark.tikv.key_file                                    /home/tispark/client.key
```

* 以下は、TiKVクライアントでJKS構成を有効にする例です。

```
spark.tispark.tikv.tls_enable                                  true
spark.tispark.tikv.jks_enable                                  true
spark.tispark.tikv.jks_key_path                                /home/tispark/config/tikv-truststore
spark.tispark.tikv.jks_key_password                            tikv_trustore_password
spark.tispark.tikv.jks_trust_path                              /home/tispark/config/tikv-clientstore
spark.tispark.tikv.jks_trust_password                          tikv_clientstore_password
```

X.509証明書とJKS証明書の両方が構成されている場合、JKSが優先されます。つまり、TLSビルダーは最初にJKS証明書を使用します。したがって、単なる一般的なPEM証明書を使用したい場合は、`spark.tispark.tikv.jks_enable=true` を設定しないでください。

* 以下は、JDBCコネクタでTLSを有効にする例です。

```
spark.tispark.jdbc.tls_enable                                  true
spark.tispark.jdbc.server_cert_store                           /home/tispark/jdbc-truststore
spark.tispark.jdbc.server_cert_password                        jdbc_truststore_password
spark.tispark.jdbc.client_cert_store                           /home/tispark/jdbc-clientstore
spark.tispark.jdbc.client_cert_password                        jdbc_clientstore_password
```

- TiDB TLSを開く方法の詳細については、[TiDBクライアントとサーバーの間でTLSを有効にする](/enable-tls-between-clients-and-servers.md) を参照してください。
- JAVAキーストアを生成する方法の詳細については、[SSLを使用したセキュアな接続](https://dev.mysql.com/doc/connector-j/en/connector-j-reference-using-ssl.html) を参照してください。

### Log4jの構成

`spark-shell` または `spark-sql` を起動してクエリを実行すると、次のような警告が表示される場合があります。

```
Failed to get database ****, returning NoSuchObjectException
Failed to get database ****, returning NoSuchObjectException
```

ここで `****` はデータベース名です。

この警告は無害であり、Sparkが自身のカタログで `****` を見つけられないために発生します。これらの警告は無視して構いません。

これらを抑制するには、`${SPARK_HOME}/conf/log4j.properties` に次のテキストを追加してください。

```
# tispark disable "WARN ObjectStore:568 - Failed to get database"
log4j.logger.org.apache.hadoop.hive.metastore.ObjectStore=ERROR
```

### タイムゾーンの構成

`-Duser.timezone=GMT-7` のように `-Duser.timezone` システムプロパティを使用してタイムゾーンを設定し、`Timestamp` タイプに影響を与えます。

`spark.sql.session.timeZone` を使用しないでください。

## 機能

TiSparkの主な機能は次のとおりです。

| 機能サポート                        | TiSpark 2.4.x | TiSpark 2.5.x | TiSpark 3.0.x | TiSpark 3.1.x |
|------------------------------------| ------------- | ------------- | ----------- |---------------|
| tidb_catalogを使用しないSQL選択         | ✔           | ✔           |             |               |
| tidb_catalogを使用したSQL選択            |               | ✔           | ✔         | ✔             |
| DataFrameの追加                    | ✔           | ✔           | ✔         | ✔             |
| DataFrameの読み取り                | ✔           | ✔           | ✔         | ✔             |
| データベースの表示                  | ✔           | ✔           | ✔         | ✔             |
| テーブルの表示                     | ✔           | ✔           | ✔         | ✔             |
| 認証                               |               | ✔           | ✔         | ✔             |
| 削除                               |               |               | ✔         | ✔             |
| 挿入                               |               |               |           | ✔              |
| TLS                                |               |               | ✔         | ✔             |
| DataFrameの認証                    |               |               |             | ✔             |

### 式インデックスのサポート

TiDB v5.0は[式インデックス](/sql-statements/sql-statement-create-index.md#expression-index) をサポートしています。
TiSparkは現在、`expression index`を持つテーブルからデータを取得する機能に対応していますが、`expression index`はTiSparkのプランナーによって使用されません。

### TiFlashとの連携

TiSparkは、設定`spark.tispark.isolation_read_engines`を通じてTiFlashからデータを読み取ることができます。

### パーティションテーブルのサポート

**TiDBからのパーティションテーブルの読み込み**

TiSparkは、TiDBからの範囲およびハッシュパーティションテーブルを読み込むことができます。

現在、TiSparkはMySQL/TiDBパーティションテーブル構文`select col_name from table_name partition(partition_name)`をサポートしていません。ただし、`where`条件を使用してパーティションをフィルタリングすることができます。

TiSparkは、パーティションタイプおよびテーブルに関連付けられたパーティション式に応じて、パーティションプルーニングを適用するかどうかを決定します。

パーティションプルーニングは、以下のいずれかの条件を満たす場合にのみ、範囲パーティションに対して適用されます。

+ 列式
+ `YEAR($argument)`（引数が列であり、その型がdatetimeまたはdatetimeとして解析できる文字列リテラルである）。

パーティションプルーニングが適用できない場合、TiSparkの読み取りはすべてのパーティションを対象としたテーブルスキャンと同等です。

**パーティションテーブルへの書き込み**

現在、TiSparkは次の条件に該当する範囲およびハッシュパーティションテーブルにデータを書き込むことしかサポートしていません。

+ パーティション式が列式である場合。
+ パーティション式が`YEAR($argument)`であり、引数が列であり、その型がdatetimeまたはdatetimeとして解析できる文字列リテラルである場合。

パーティションテーブルへの書き込み方法は2つあります。

- 置換と追加のセマンティクスをサポートするパーティションテーブルにデータを書き込むために、データソースAPIを使用します。
- Spark SQLを使用してdeleteステートメントを使用します。

> **注:**
>
> 現在、TiSparkはutf8mb4_bin照合が有効なパーティションテーブルにのみ書き込みをサポートしています。

### セキュリティ

TiSpark v2.5.0以降を使用している場合、TiDBを使用してTiSparkユーザーの認証と認可を行うことができます。

認証および認可機能はデフォルトで無効になっています。有効にするには、Spark構成ファイル`spark-defaults.conf`に次の設定を追加してください。

```
// 認証および認可を有効にする
spark.sql.auth.enable true

// TiDB情報を構成する
spark.sql.tidb.addr $your_tidb_server_address
spark.sql.tidb.port $your_tidb_server_port
spark.sql.tidb.user $your_tidb_server_user
spark.sql.tidb.password $your_tidb_server_password
```

詳細については、[TiDBサーバーを介した認可および認証](https://github.com/pingcap/tispark/blob/master/docs/features/authorization_userguide.md)を参照してください。

### その他の機能

- [プッシュダウン](https://github.com/pingcap/tispark/blob/master/docs/features/push_down.md)
- [TiSparkでの削除](https://github.com/pingcap/tispark/blob/master/docs/features/delete_userguide.md)
- [古いデータの読み取り](https://github.com/pingcap/tispark/blob/master/docs/features/stale_read.md)
- [複数のカタログを使用したTiSpark](https://github.com/pingcap/tispark/wiki/TiSpark-with-multiple-catalogs)
- [TiSpark TLS](#tls-configurations)
- [TiSparkプラン](https://github.com/pingcap/tispark/blob/master/docs/features/query_execution_plan_in_TiSpark.md)

## 統計情報

TiSparkは統計情報を使用して次のことを行います。

+ 最小推定コストを持つクエリプランで使用するインデックスを決定すること。
+ 効率的なブロードキャスト結合を可能にする小さなテーブルのブロードキャスト。

TiSparkが統計情報にアクセスできるようにするには、関連するテーブルが分析されていることを確認してください。

テーブルの分析の詳細については、[統計情報の概要](/statistics.md)を参照してください。

TiSpark 2.0以降では、統計情報はデフォルトで自動的に読み込まれます。

## FAQ

[TiSpark FAQ](https://github.com/pingcap/tispark/wiki/TiSpark-FAQ)を参照してください。