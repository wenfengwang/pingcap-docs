---
title: TiDBクラウド組込みメトリクス
summary: TiDBクラウド組み込みメトリクスの表示方法を学び、これらのメトリクスの意味を理解します。

# TiDBクラウド組込みメトリクス

TiDBクラウドは、Metricsページでクラスターの標準メトリクスの全セットを収集して表示します。これらのメトリクスを表示することで、パフォーマンスの問題を簡単に特定し、現在のデータベースデプロイメントが要件を満たしているかどうかを判断できます。

## Metricsページを表示

Metricsページのメトリクスを表示するには、次の手順を実行します。

1. [TiDB Cloudコンソール](https://tidbcloud.com/) に移動し、プロジェクトの [**クラスター**](https://tidbcloud.com/console/clusters) ページに移動します。

    > **ヒント:**
    >
    > 複数のプロジェクトがある場合は、左下隅の<MDSvgIcon name="icon-left-projects" />をクリックして、別のプロジェクトに切り替えることができます。

2. 対象クラスターの名前をクリックします。クラスターの概要ページが表示されます。
3. 左側のナビゲーションペインで **Metrics** をクリックします。

## メトリクスの保持ポリシー

TiDB DedicatedクラスターおよびTiDB Serverlessクラスターのメトリクスデータは7日間保持されます。

## TiDB Dedicatedクラスターのメトリクス

次のセクションでは、TiDB DedicatedクラスターのMetricsページでのメトリクスについて説明します。

### 概要

| メトリック名  | ラベル | 説明                                   |
| :------------| :------| :---------------------------------------- |
| SQLタイプ別データベース時間 | データベース時間, {SQLタイプ} | データベース時間: 秒ごとの合計データベース時間。<br/>{SQLタイプ}: `SELECT`、`INSERT`、`UPDATE`などのSQLタイプによって収集された、SQLステートメントごとのデータベース時間。 |
| 1秒あたりのクエリ | {SQLタイプ} | すべてのTiDBインスタンスで1秒あたりに実行されるSQLステートメントの数。`SELECT`、`INSERT`、`UPDATE`などのSQLタイプによって収集されます。 |
| クエリの実行時間 | avg-{SQLタイプ}、99-{SQLタイプ} | クライアントからのリクエストの受信からTiDBがリクエストを実行し、クライアントに結果を返すまでの時間。一般に、クライアントのリクエストはSQLステートメントの形で送信されますが、この時間には `COM_PING`、`COM_SLEEP`、`COM_STMT_FETCH`、`COM_SEND_LONG_DATA`などのコマンドの実行時間も含まれます。TiDBはMulti-Queryをサポートしており、クライアントは一度に複数のSQLステートメントを送信できます。例えば `select 1; select 1; select 1;`。この場合、このクエリの合計実行時間には、すべてのSQLステートメントの実行時間が含まれます。 |
| 失敗したクエリ | All, {エラータイプ} @ {インスタンス} | 各TiDBインスタンスで分析される、SQLステートメントの実行エラーの種類（構文エラーや主キーコンフリクトなど）の統計情報。エラーが発生するモジュールとエラーコードが含まれます。 |
| 1秒あたりのコマンド | Query、StmtExecute、StmtPrepare | コマンドタイプに基づいてすべてのTiDBインスタンスで処理されるコマンドの数。 |
| プランキャッシュを利用したクエリの数 | hit、miss | 全TiDBインスタンスで1秒あたりのプランキャッシュを利用したクエリの数。<br/>miss: 全TiDBインスタンスでプランキャッシュが不足しているクエリの数。 |
| 1秒あたりのトランザクション | {タイプ}-{トランザクションモデル} | 1秒あたりに実行されるトランザクションの数。 |
| トランザクションの実行時間 | avg-{トランザクションモデル}、99-{トランザクションモデル} | トランザクションの平均実行時間または99パーセンタイルの実行時間。 |
| 接続数 | All, アクティブ接続 | すべてのTiDBインスタンスへの接続数。<br/>アクティブ接続: すべてのTiDBインスタンスへのアクティブな接続数。 |
| 切断数 | {インスタンス}-{結果} | 各TiDBインスタンスへの切断されたクライアント数。 |

### 高度

| メトリック名  | ラベル | 説明                                   |
| :------------| :------| :---------------------------------------- |
| 平均アイドル接続時間 | avg-in-txn、avg-not-in-txn | 接続のアイドル時間は、接続がアイドル状態である時間を示します。<br/>avg-in-txn: トランザクション内で接続がアイドル状態にある場合の平均接続アイドル時間。<br/>avg-not-in-txn: トランザクション内で接続がアイドル状態でない場合の平均接続アイドル時間。 |
| トークン取得時間 | avg、99 | SQLステートメントのトークン取得に要する平均時間または99パーセンタイルの時間。 |
| パース時間 | avg、99 | SQLステートメントのパースに要する平均時間または99パーセンタイルの時間。 |
| コンパイル時間 | avg、99 | パースされたSQL ASTを実行プランにコンパイルするのに要する平均時間または99パーセンタイルの時間。 |
| 実行時間 | avg、99 | SQLステートメントの実行プランの実行に要する平均時間または99パーセンタイルの時間。 |
| 平均TiDB KVリクエスト実行時間 | {リクエストタイプ} | リクエストタイプ（`Get`、`Prewrite`、`Commit`など）に基づいて、すべてのTiDBインスタンスでKVリクエストの実行に要する平均時間。 |
| 平均TiKV gRPC実行時間 | {リクエストタイプ} | リクエストタイプ（`kv_get`、`kv_prewrite`、`kv_commit`など）に基づいて、すべてのTiKVインスタンスでgRPCリクエストの実行に要する平均時間。 |
| 平均/ P99 PD TSO待機/RPC実行時間 | wait-avg/99、rpc-avg/99 | Wait: PDがTSOを返すのを待機する場合の平均時間または99パーセンタイル時間、すべてのTiDBインスタンスで。 <br/>RPC: TSOリクエストをPDに送信し、PDからTSOを受信するまでの平均時間または99パーセンタイル時間、すべてのTiDBインスタンスで。 |
| 平均/ P99 ストレージ非同期書き込み実行時間 | avg、99 | 非同期書き込みに要する平均時間または99パーセンタイル時間。 平均ストレージ非同期書き込み実行時間 = 平均ストア実行時間 + 平均適用実行時間。 |
| 平均/ P99 ストア実行時間 | avg、99 | 非同期書き込み中のループで要する平均時間または99パーセンタイル時間。 |
| 平均/ P99 適用実行時間 | avg、99 | 非同期書き込み中の適用中のループで要する平均時間または99パーセンタイル時間。 |
| 平均/ P99 ログ追加実行時間 | avg、99 | Raftによるログの追加に要する平均時間または99パーセンタイル時間。 |
| 平均/ P99 ログコミット実行時間 | avg、99 | Raftによるログのコミットに要する平均時間または99パーセンタイル時間。 |
| 平均/ P99 ログ適用実行時間 | avg、99 | Raftによるログの適用に要する平均時間または99パーセンタイル時間。 |

### サーバー

| メトリック名  | ラベル | 説明                                   |
| :------------| :------| :---------------------------------------- |
| TiDBの稼働時間 | ノード | 前回の再起動以降の各TiDBノードの稼働時間。 |
| TiDBのCPU使用率 | ノード | 各TiDBノードのCPU使用率の統計情報。 |
| TiDBのメモリ使用率 | ノード | 各TiDBノードのメモリ使用率の統計情報。 |
| TiKVの稼働時間 | ノード | 前回の再起動以降の各TiKVノードの稼働時間。 |
| TiKVのCPU使用率 | ノード | 各TiKVノードのCPU使用率の統計情報。 |
| TiKVのメモリ使用率 | ノード | 各TiKVノードのメモリ使用率の統計情報。 |
| TiKVのIO Bps | ノード-write、ノード-read | 各TiKVノードの読み込みと書き込みの総入出力バイト数/秒。 |
| TiKVのストレージ使用率 | ノード | 各TiKVノードのストレージ使用率の統計情報。 |
| TiFlashの稼働時間 | ノード | 前回の再起動以降の各TiFlashノードの稼働時間。 |
| TiFlashのCPU使用率 | ノード | 各TiFlashノードのCPU使用率の統計情報。 |
| TiFlashのメモリ使用率 | ノード | 各TiFlashノードのメモリ使用率の統計情報。 |
| TiFlashのIO MBps | ノード-write、ノード-read | 各TiFlashノードの読み込みと書き込みの合計バイト/秒。 |
| TiFlashのストレージ使用率 | ノード | 各TiFlashノードのストレージ使用率の統計情報。 |

## TiDB Serverlessクラスターのメトリクス

Metricsページには、TiDB Serverlessクラスターのメトリクスを表示するための2つのタブが用意されています。

- クラスターステータス: クラスターレベルの主要なメトリクスを表示します。
- データベースステータス: データベースレベルの主要なメトリクスを表示します。

### クラスターステータス

次の表は、**クラスターステータス**タブのクラスターレベルの主要メトリクスを示しています。

| メトリック名  | ラベル | 説明                                   |
| :------------| :------| :---------------------------------------- |
| リクエストユニット | RU/秒 | リクエストユニット（RU）は、クエリやトランザクションのリソース消費を追跡するために使用する測定単位です。実行するクエリだけでなく、バックグラウンドのアクティビティで消費されることがあります。そのため、QPSが0の場合、1秒あたりのリクエストユニットが0でないことがあります。 |
| 使用されるストレージサイズ | 行ベースストレージ、列ベースストレージ | 行ストアのサイズと列ストアのサイズ。 |
| 1秒あたりのクエリ | All、{SQLタイプ} | 実行されるSQLステートメントの数/秒。`SELECT`、`INSERT`、`UPDATE`などのSQLタイプによって収集されます。 |
| 平均クエリ時間 | All、{SQLタイプ} | クライアントからのリクエストの受信からTiDB Serverlessクラスターがリクエストを実行し、クライアントに結果を返すまでの時間。 |
| 失敗したクエリ | All | 1秒あたりのSQLステートメント実行エラーの数。 |
| 1秒あたりのトランザクション | All | 1秒あたりに実行されるトランザクションの数。 |
| 平均トランザクション時間 | All | トランザクションの平均実行時間。 |
| 総接続数 | All | TiDB Serverlessクラスターへの接続数。 |

### データベースステータス
```markdown
      The following table illustrates the database-level main metrics under the **Database Status** tab.

      | Metric name  | Labels | Description                                   |
      | :------------| :------| :-------------------------------------------- |
      | QPS Per DB | All, {Database name} | The number of SQL statements executed per second on every database, which are collected by SQL types, such as `SELECT`, `INSERT`, and `UPDATE`. |
      | Average Query Duration Per DB | All, {Database name} | The duration from receiving a request from the client to a database until the database executes the request and returns the result to the client.|
      | Failed Query Per DB | All, {Database name} | The statistics of error types according to the SQL statement execution errors per second on every database.|

      ## FAQ

      **1. Why are some panes empty on this page?**

      If a pane does not provide any metrics, the possible reasons are as follows:

      - The workload of the corresponding cluster does not trigger this metric. For example, the failed query metric is always empty in the case of no failed queries.
      - The cluster version is low. You need to upgrade it to the latest version of TiDB to see these metrics.

      If all these reasons are excluded, you can contact the [PingCAP support team](/tidb-cloud/tidb-cloud-support.md) for troubleshooting.

      **2. Why might metrics be discontinuous in rare cases?**

      In some rare cases, metrics might be lost, such as when the metrics system experiences high pressure.

      If you encounter this problem, you can contact [PingCAP Support](/tidb-cloud/tidb-cloud-support.md) for troubleshooting.
```