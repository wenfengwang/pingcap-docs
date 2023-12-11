---
title: OLTPシナリオのパフォーマンスチューニングプラクティス
summary: このドキュメントでは、OLTPワークロードのパフォーマンスを分析しチューニングする方法について述べます。
---

# OLTPシナリオのパフォーマンスチューニングプラクティス

TiDBは、TiDBダッシュボード上の[Top SQL](/dashboard/top-sql.md)および[Continuous Profiling](/dashboard/continuous-profiling.md)機能、およびTiDBの[パフォーマンス概要ダッシュボード](/grafana-performance-overview-dashboard.md)など、包括的なパフォーマンス診断および分析機能を提供しています。

このドキュメントでは、これらの機能を活用して、同じOLTPワークロードのパフォーマンスを7つの異なるランタイムシナリオで分析し比較する方法について記載し、効率的にTiDBのパフォーマンスを分析しチューニングするためのパフォーマンスチューニングプロセスを示します。

> **注意:**
>
> [Top SQL](/dashboard/top-sql.md)および[Continuous Profiling](/dashboard/continuous-profiling.md)はデフォルトで有効になっていません。事前に有効にする必要があります。

これらのシナリオにおいて異なるJDBC構成で同じアプリケーションを実行することで、アプリケーションとデータベースの異なる相互作用が全体システムのパフォーマンスにどのように影響するかを示し、より良いパフォーマンスのためにTiDBを使用したJavaアプリケーションの[ベストプラクティス](/best-practices/java-app-best-practices.md)を適用できるようにします。

## 環境の説明

このドキュメントでは、コアバンキングのOLTPワークロードをデモンストレーションします。シミュレーション環境の構成は以下の通りです。

- ワークロードのアプリケーション開発言語: JAVA
- ビジネスで使用されるSQLステートメント: 合計200ステートメント中、90%がSELECTステートメントです。典型的なリード重視のOLTPワークロードです。
- トランザクションで使用されるテーブル: 合計60のテーブルあります。12個のテーブルは更新操作に関与し、残りの48個のテーブルは読み取り専用です。
- アプリケーションが使用する分離レベル: `read committed`
- TiDBクラスタ構成: 3つのTiDBノードと3つのTiKVノードで構成されており、各ノードに16 CPUが割り当てられています。
- クライアントサーバー構成: 36 CPU

## シナリオ1. クエリインタフェースの使用

### アプリケーション設定

アプリケーションは、以下のJDBC設定を使用してクエリインタフェースを介してデータベースに接続します。

```
useServerPrepStmts=false
```

### パフォーマンス分析

#### TiDBダッシュボード

TiDBダッシュボードのTop SQLページから、非ビジネスSQLタイプ`SELECT @@session.tx_isolation`が最も多くのリソースを消費していることがわかります。TiDBはこれらのタイプのSQLステートメントを迅速に処理しますが、これらのSQLステートメントは最も多くの実行を行い、最も高い全体CPU時間の消費量につながっています。

![dashboard-for-query-interface](/media/performance/case1.png)

TiDBの次のフレームチャートから、`Compile`および`Optimize`などの関数のCPU消費がSQL実行時に著しくであることがわかります。アプリケーションがクエリインタフェースを使用しているため、TiDBは実行計画キャッシュを使用できません。TiDBは各SQLステートメントの実行計画をコンパイルおよび生成する必要があります。

![flame-graph-for-query-interface](/media/performance/7.1.png)

- ExecuteStmt cpu = 38% cpu time = 23.84s
- Compile cpu = 27%  cpu time = 17.17s
- Optimize cpu = 26% cpu time = 16.41s

#### パフォーマンス概要ダッシュボード

次のパフォーマンス概要ダッシュボードから、データベース時間概要およびQPSを確認できます。

![performance-overview-1-for-query-interface](/media/performance/j-1.png)

- SQLタイプ別データベース時間: `Select`ステートメントのタイプが最も時間を取っています。
- SQLフェーズ別データベース時間: `execute`および`compile`フェーズが最も時間を取っています。
- SQL実行時間概要: `Get`、`Cop`および`tso wait`が最も時間を取っています。
- タイプ別CPS: `Query`コマンドのみが使用されています。
- 実行計画キャッシュOPSを使用したクエリ数: 実行計画キャッシュがヒットしていないことを示すデータはありません。
- クエリの実行時間では、`execute`および`compile`のレイテンシが最も高い割合を占めています。
- 平均QPS = 56.8k

クラスタのリソース消費を確認します: TiDB CPUの平均利用率は925%、TiKV CPUの平均利用率は201%、TiKV IOの平均スループットは18.7 MB/sです。TiDBのリソース消費が著しく高いです。

![performance-overview-2-for-query-interface](/media/performance/5.png)

### 分析結論

無用な非ビジネスSQLステートメントを排除する必要があります。これらのステートメントは多くの実行を行い、TiDB CPU使用率が高くなっています。

## シナリオ2. maxPerformance設定の使用

### アプリケーション設定

シナリオ1のJDBC接続文字列に新たなパラメータ`useConfigs=maxPerformance`を追加します。このパラメータは、例えば`select @@session.transaction_read_only`のようなJDBCからデータベースに送信されたSQLステートメントを排除するために使用できます。完全な構成は次の通りです。

```
useServerPrepStmts=false&useConfigs=maxPerformance
```

### パフォーマンス分析

#### TiDBダッシュボード

TiDBダッシュボードのTop SQLページから、最も多くのリソースを消費していた`SELECT @@session.tx_isolation`が消えていることがわかります。

![dashboard-for-maxPerformance](/media/performance/case2.png)

次のTiDBのフレームチャートから、`Compile`および`Optimize`などの関数のCPU消費がSQL実行時に依然として著しいことがわかります。

![flame-graph-for-maxPerformance](/media/performance/20220507-145257.jpg)

- ExecuteStmt cpu = 43% cpu time =35.84s
- Compile cpu = 31% cpu time =25.61s
- Optimize cpu = 30% cpu time = 24.74s

#### パフォーマンス概要ダッシュボード

以下は、データベース時間概要とQPSのデータです。

![performance-overview-1-for-maxPerformance](/media/performance/j-2.png)

- SQLタイプ別データベース時間: `Select`ステートメントのタイプが最も時間を取っています。
- SQLフェーズ別データベース時間: `execute`および`compile`フェーズが最も時間を取っています。
- SQL実行時間概要: `Get`、`Cop`、`Prewrite`および`tso wait`が最も時間を取っています。
- データベース時間では、`execute`および`compile`が最も高い割合を占めています。
- タイプ別CPS: `Query`コマンドのみが使用されています。
- 平均QPS = 24.2k (56.3kから24.2kまで)
- 実行計画キャッシュがヒットしていない。

シナリオ1からシナリオ2への変更により、平均TiDB CPU使用率が925%から874%に下がり、平均TiKV CPU使用率が201%から約250%に増加しています。

![performance-overview-2-for-maxPerformance](/media/performance/9.1.1.png)

主要遅延メトリクスの変化は次の通りです。

![performance-overview-3-for-maxPerformance](/media/performance/9.2.2.png)

- 平均クエリ時間 = 1.12ms (479μsから1.12msへ)
- 平均解析時間 = 84.7μs (37.2μsから84.7μsへ)
- 平均コンパイル時間 = 370μs (166μsから370μsへ)
- 平均実行時間 = 626μs (251μsから626μsへ)

### 分析結論

シナリオ1と比較して、シナリオ2のQPSが大幅に低下しています。平均クエリ時間および平均`parse`、`compile`、`execute`時間が大幅に増加しています。これは、シナリオ1の`select @@session.transaction_read_only`のようなSQLステートメントが多く実行され、処理時間が短いため、平均パフォーマンスデータが低下するためです。シナリオ2では、こうしたステートメントがブロックされ、ビジネス関連のSQLステートメントのみが残るため、平均実行時間が増加します。

アプリケーションがクエリインタフェースを使用する場合、TiDBは実行計画キャッシュを使用できないため、TiDBが高いリソースを消費して実行計画をコンパイルすることとなります。この場合、TiDB CPU消費を減らし遅延を減らすために、TiDBの実行計画キャッシュを使用するPrepared Statementインタフェースを使用することをお勧めします。

## シナリオ3. 実行計画キャッシュを無効にしたPrepared Statementインタフェースの使用

### アプリケーション設定

アプリケーションは、以下の接続構成を使用して、Prepared Statementインタフェースを有効にします。シナリオ2と比較して、JDBCパラメータ`useServerPrepStmts`の値を`true`に変更します。

```
useServerPrepStmts=true&useConfigs=maxPerformance"
```

### パフォーマンス分析

#### TiDBダッシュボード

以下は、TiDBのフレームチャートから、Prepared Statementインタフェースを有効にした後も`CompileExecutePreparedStmt`および`Optimize`のCPU消費が依然として著しいことがわかります。

![flame-graph-for-PrepStmts](/media/performance/3.1.1.png)

- ExecutePreparedStmt cpu = 31%  cpu time = 23.10s
- preparedStmtExec cpu = 30% cpu time = 22.92s
- CompileExecutePreparedStmt cpu = 24% cpu time = 17.83s
- Optimize cpu = 23%  cpu time = 17.29s

#### パフォーマンス概要ダッシュボード

Prepared Statementインタフェースを使用した後、データベース時間の概要とQPSのデータは次の通りです。
```
      + {R}
      + {R}
    + {R}
  + {R}
```
- データベースのSQLタイプ別時間：`Select`ステートメントが最も時間を取り、`一般的`ステートメントは消えます。
- SQLフェーズ別データベース時間：`実行`フェーズが大部分の時間を取ります。
- SQL実行時間の概要：`tso wait`、`Get`、および`Cop`が大部分の時間を取ります。
- 実行計画キャッシュがヒットしています。実行計画キャッシュOPSの値は、おおよそ1秒あたりの`StmtExecute`と等しいです。
- タイプ別CPS：`StmtExecute`コマンドのみが使用されています。
- 平均QPS = 30.9k(22.1kから30.9kまで)

TiDBのCPU利用率の平均が827%から577%に低下します。QPSの増加に伴い、平均TiKV CPU利用率が313%に増加します。

![performance-overview-for-2-command](/media/performance/j-5-cpu.png)

主要な待ち時間の指標は以下の通りです：

![performance-overview-for-3-command](/media/performance/j-5-duration.png)

- 平均クエリの実行時間=690μs(426μsから690μsまで)
- 平均パース時間=13.5μs(12.3μsから13.5μsまで)
- 平均コンパイル時間=49.7μs(53.3μsから49.7μsまで)
- 平均実行時間=623μs(699μsから623μsまで)
- 平均PD TSO待ち時間=196μs(224μsから196μsまで)
- トランザクションの中での接続アイドル時間の平均=608μs(250μsから608μsまで)

### 分析の結論

- シナリオ4と比較して、シナリオ5の**CPS By Type**パネルには`StmtExecute`コマンドしかないため、ネットワークの往復が2回回避され、全体的なシステムのQPSが増加します。
- QPSの増加に伴い、解析時間、コンパイル時間、実行時間は短縮されますが、クエリの実行時間は増加します。これはTiDBが`StmtPrepare`および`StmtClose`を非常に速く処理し、これら2つのコマンドタイプをなくすことで平均クエリ実行時間が増加するためです。
- データベースのSQLフェーズ別時間では、`execute`が最も時間を取り、データベース時間に近いです。一方、SQL実行時間の概要では、`tso wait`が最も時間を取り、`execute`時間の4分の1以上がTSO待ちにかかっています。
- 総`tso wait`時間は1秒あたり5.46秒です。平均`tso wait`時間は196μsで、1秒あたりの`tso cmd`の数は28kであり、30.9kのQPSに非常に近いです。これはTiDBの`read committed`アイソレーションレベルの実装によるもので、トランザクション内の各SQLステートメントはPDからTSOをリクエストする必要があります。

TiDB v6.0では、`rc read`を提供しており、`set global tidb_rc_read_check_ts=on;`というグローバル変数によって`read committed`アイソレーションレベルが最適化され、`tso cmd`が減少します。この機能は、トランザクション内の各ステートメントが`start-ts`および`commit-ts`をPDから取得する必要がある`repeatable-read`アイソレーションレベルと同じように動作します。トランザクション内のステートメントは、まずTiKVからデータを読むために`start-ts`を使用します。TiKVから読まれたデータが`start-ts`よりも古ければ、データは直接返されます。読まれたデータが`start-ts`よりも新しければ、データは破棄されます。TiDBはPDからTSOを要求し、読み取りを再試行します。その後のステートメントの`for update ts`は、最新のPD TSOを使用します。

## シナリオ6：`tidb_rc_read_check_ts`変数を有効にしてTSOリクエストを減らす

### アプリケーションの構成

シナリオ5と比較して、アプリケーションの構成はそのままです。唯一の違いは、`set global tidb_rc_read_check_ts=on;`変数を使用してTSOリクエストを減らすように構成されていることです。

### パフォーマンスの分析

#### ダッシュボード

TiDB CPUのflame chartには、大きな変化はありません。

- ExecutePreparedStmt cpu = 22% cpu time = 8.4s

![flame-graph-for-rc-read](/media/performance/6.2.2.png)

#### パフォーマンス概要ダッシュボード

RC readを使用した後、QPSは30.9kから34.9kに増加し、1秒あたりの`tso wait`による時間が5.46秒から456ミリ秒に減少します。

![performance-overview-1-for-rc-read](/media/performance/j-6.png)

- データベースのSQLタイプ別時間：`Select`ステートメントタイプが最も時間を取ります。
- データベースのSQLフェーズ別時間：`実行`および`コンパイル`フェーズが最も時間を取ります。
- SQL実行時間の概要：`Get`、`Cop`、および`Prewrite`が最も時間を取ります。
- 実行計画キャッシュがヒットしています。実行計画キャッシュOPSの値は、おおよそ1秒あたりの`StmtExecute`と等しいです。
- タイプ別CPS：`StmtExecute`コマンドのみが使用されています。
- 平均QPS = 34.9k(30.9kから34.9kまで)

1秒あたりの`tsocmd`が28.3kから2.7kに減少します。

![performance-overview-2-for-rc-read](/media/performance/j-6-cmd.png)

平均TiDB CPU利用率が577%から603%に増加します。

![performance-overview-3-for-rc-read](/media/performance/j-6-cpu.png)

主要な待ち時間の指標は以下の通りです：

![performance-overview-4-for-rc-read](/media/performance/j-6-duration.png)

- 平均クエリ実行時間=533μs(690μsから533μsまで)
- 平均パース時間=13.4μs(13.5μsから13.4μsまで)
- 平均コンパイル時間=50.3μs(49.7μsから50.3μsまで)
- 平均実行時間=466μs(623μsから466μsまで)
- 平均PD TSO待ち時間=171μs(196μsから171μsまで)

### 分析の結論

`set global tidb_rc_read_check_ts=on;`を有効にした後、RC Readは`tsocmd`の回数を大幅に減らし、その結果、`tso wait`時間と平均クエリ実行時間が減少し、QPSが向上します。

現在のデータベース時間と待ち時間のボトルネックは`実行`フェーズにあり、ここでは`Get`および`Cop`読み取りリクエストが最も多くなります。このワークロードの多くのテーブルは読み取り専用またはほとんど変更されないため、TiDB v6.0.0以降でサポートされている小さなテーブルキャッシュ機能を使用して、これらの小さなテーブルのデータをキャッシュし、KV読み取りリクエストの待ち時間とリソース消費を削減することができます。

## シナリオ7：小さなテーブルキャッシュを使用する

### アプリケーションの構成

シナリオ6と比較して、アプリケーションの構成はそのままです。唯一の違いは、シナリオ7では`alter table t1 cache;`などのSQLステートメントを使用して、それらの読み取り専用テーブルをキャッシュすることです。

### パフォーマンスの分析

#### TiDBダッシュボード

TiDB CPUのflame chartには、大きな変化はありません。

![flame-graph-for-table-cache](/media/performance/7.2.png)

#### パフォーマンス概要ダッシュボード

QPSは34.9kから40.9kに増加し、`実行`フェーズで最も時間を取るKVリクエストのタイプは`Prewrite`と`Commit`に変わります。1秒あたりの`Get`によるデータベース時間は5.33秒から1.75秒に減少し、1秒あたりの`Cop`によるデータベース時間は3.87秒から1.09秒に減少します。

![performance-overview-1-for-table-cache](/media/performance/j-7.png)

- データベースのSQLタイプ別時間：`Select`ステートメントタイプが最も時間を取ります。
- データベースのSQLフェーズ別時間：`実行`および`コンパイル`フェーズが最も時間を取ります。
- SQL実行時間の概要：`Prewrite`、`Commit`、および`Get`が最も時間を取ります。
- 実行計画キャッシュがヒットしています。実行計画キャッシュOPSの値は、おおよそ1秒あたりの`StmtExecute`と等しいです。
- タイプ別CPS：`StmtExecute`コマンドのみが使用されています。
- 平均QPS = 40.9k(34.9kから40.9kまで)

平均TiDB CPU利用率は603%から478%に減少し、平均TiKV CPU利用率は346%から256%に減少します。

![performance-overview-2-for-table-cache](/media/performance/j-7-cpu.png)

平均クエリの待ち時間は533μsから313μsに減少し、平均`実行`待ち時間は466μsから250μsに減少します。

![performance-overview-3-for-table-cache](/media/performance/j-7-duration.png)

- 平均クエリ実行時間=313μs(533μsから313μsまで)
- 平均パース時間=11.9μs(13.4μsから11.9μsまで)
- 平均コンパイル時間 = 47.7μs（50.3μs から 47.7μs）
- 平均実行時間 = 251μs（466μs から 251μs）

### 分析結論

すべての読み取り専用テーブルをキャッシュした後、`実行時間`が著しく減少しました。なぜなら、すべての読み取り専用テーブルが TiDB にキャッシュされており、これらのテーブルのデータを TiKV でクエリする必要がないため、クエリ時間が短縮され、QPS が増加します。

これは楽観的な結果ですが、実際のビジネスにおける読み取り専用テーブルのデータは TiDB がすべてキャッシュするには大きすぎるかもしれません。別の制限事項は、小さなテーブルのキャッシング機能が書き込み操作をサポートしているにも関わらず、書き込み操作には、すべての TiDB ノードのキャッシュがまず無効化されるまでのデフォルトの待機時間が 3 秒必要であり、厳密なレイテンシ要件を持つアプリケーションには適していない可能性があります。

## 要約

以下の表に、7つの異なるシナリオのパフォーマンスがリストされています。

| メトリクス | シナリオ1 | シナリオ2 | シナリオ3 | シナリオ4 | シナリオ5 | シナリオ6 | シナリオ7 | シナリオ2 と シナリオ5 の比較（%） | シナリオ3 と シナリオ7 の比較（%） |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| クエリ時間  | 479μs | 1120μs | 528μs | 426μs | 690μs | 533μs | 313μs | -38% | -51% |
| QPS            | 56.3k |  24.2k | 19.7k | 22.1k | 30.9k | 34.9k | 40.9k | +28% | +108% |

これらのシナリオでは、シナリオ2はアプリケーションがクエリインターフェースを使用する一般的なシナリオであり、シナリオ5はアプリケーションがプリペアドステートメントインターフェースを使用する理想的なシナリオです。

- シナリオ2 と シナリオ5 を比較すると、Java アプリケーション開発のベストプラクティスを使用し、クライアント側でプリペアドステートメントオブジェクトをキャッシュすることで、各 SQL ステートメントには実行計画キャッシュにアクセスするためにコマンドとデータベースの相互作用が1つしか必要なくなり、クエリレイテンシが38%低下し、QPSが28%増加します。これにより、平均 TiDB CPU 使用率が936% から 577% に低下します。
- シナリオ2 と シナリオ7 を比較すると、シナリオ5 の上に RC Read や小さなテーブルキャッシュなどの最新の TiDB 最適化機能を使用すると、レイテンシが51%減少し、QPSが108%増加します。同様に、平均 TiDB CPU 使用率が936% から 478% に低下します。

各シナリオのパフォーマンスを比較することで、次の結論を得ることができます。

- TiDB の実行計画キャッシュは OLTP のパフォーマンスチューニングにおいて重要な役割を果たしています。v6.0.0 から導入された RC Read や小さなテーブルキャッシュ機能も、このワークロードのさらなるパフォーマンスチューニングに重要な役割を果たしています。
- TiDB は MySQL プロトコルのさまざまなコマンドと互換性があります。プリペアドステートメントインターフェースを使用し、以下の JDBC 接続パラメーターを設定すると、アプリケーションは最適なパフォーマンスを実現できます。

    ```
    useServerPrepStmts=true&cachePrepStmts=true&prepStmtCacheSize=1000&prepStmtCacheSqlLimit=20480&useConfigs= maxPerformance
    ```

- パフォーマンスの分析とチューニングには、TiDB Dashboard（たとえば、Top SQL 機能とContinuous Profiling 機能）および Performance Overview ダッシュボードを使用することをお勧めします。

    - [Top SQL](/dashboard/top-sql.md) 機能では、データベースの実行中に各 SQL ステートメントの CPU 消費を視覚的にモニタリングおよび調査し、データベースのパフォーマンスの問題をトラブルシューティングできます。
    - [Continuous Profiling](/dashboard/continuous-profiling.md) を使用すると、TiDB、TiKV、および PD の各インスタンスからパフォーマンスデータを継続的に収集できます。アプリケーションがTiDB と相互作用するために、異なるインターフェースを使用する場合、TiDB の CPU 消費の差は膨大です。
    - [Performance Overview Dashboard](/grafana-performance-overview-dashboard.md) を使用すると、データベースの時間と SQL 実行時間の詳細情報を一覧表示できます。データベース時間に基づいてパフォーマンスのボトルネックが TiDB にあるかどうかを判断し、全体システムのパフォーマンスボトルネックを特定するために、データベース時間とレイテンシの分解などを使用できます。TiDB 内のパフォーマンスボトルネックを特定し、それに応じてパフォーマンスを調整できます。

これらの機能を組み合わせて使用することで、実際のアプリケーションの効率的なパフォーマンスの分析とチューニングが可能です。