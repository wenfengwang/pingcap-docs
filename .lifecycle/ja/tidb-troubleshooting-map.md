---
title: TiDB トラブルシューティングマップ
summary: TiDB における一般的なエラーのトラブルシューティング方法を学びます。
aliases: ['/docs/dev/tidb-troubleshooting-map/','/docs/dev/how-to/troubleshoot/diagnose-map/']
---

# TiDB トラブルシューティングマップ

このドキュメントは、TiDB およびその他のコンポーネントにおける一般的な問題をまとめたものです。このマップを使用して、関連する問題が発生した場合に診断し、解決することができます。

## 1. サービスが利用できない

### 1.1 クライアントが `Region is Unavailable` エラーを報告する場合

- 1.1.1 `Region is Unavailable` エラーは通常、一定期間 Region が利用できないために発生します。`TiKV サーバーがビジーである`と遭遇する可能性があります。また、`not leader` や `epoch not match` で TiKV へのリクエストが失敗したり、TiKV へのリクエストがタイムアウトしたりすることがあります。このような場合、TiDB は `backoff` リトライメカニズムを実行します。`backoff` がデフォルトで閾値（20秒）を超えると、エラーがクライアントに送信されます。`backoff` の閾値内では、このエラーはクライアントには表示されません。

- 1.1.2 複数の TiKV インスタンスが同時に OOM になると、OOM 期間中に Leader が存在しなくなります。[case-991](https://github.com/pingcap/tidb-map/blob/master/maps/diagnose-case-study/case991.md) を参照してください。

- 1.1.3 TiKV が `TiKV サーバーがビジー` を報告し、`backoff` 時間を超える。詳細については、[#4.3](#43-the-client-reports-the-server-is-busy-error) を参照してください。`TiKV サーバーがビジー` は内部のフロー制御メカニズムの結果であり、`backoff` 時間には含まれません。この問題は修正されます。

- 1.1.4 複数の TiKV インスタンスが起動に失敗すると、Region に Leader が存在しなくなります。物理的なマシンに複数の TiKV インスタンスが展開されている場合、ラベルが適切に構成されていないと、物理的なマシンの障害により Region に Leader が存在しなくなる可能性があります。[case-228](https://github.com/pingcap/tidb-map/blob/master/maps/diagnose-case-study/case228.md) を参照してください。

- 1.1.5 Follower の適用が前のエポックで遅延しており、Follower が Leader になった後に `epoch not match` でリクエストを拒否します。[case-958](https://github.com/pingcap/tidb-map/blob/master/maps/diagnose-case-study/case958.md) を参照してください（TiKV はメカニズムを最適化する必要があります）。

### 1.2 PD のエラーがサービスの利用不可を引き起こす

[5 PD issues](#5-pd-issues) を参照してください。

## 2. レイテンシが著しく増加する

### 2.1 一時的な増加

- 2.1.1 誤った TiDB 実行計画がレイテンシの増加を引き起こすことがあります。[#3.3](#33-wrong-execution-plan) を参照してください。

- 2.1.2 PD リーダー選出の問題や OOM により、レイテンシが増加することがあります。[#5.2](#52-pd-election) および [#5.3](#53-pd-oom) を参照してください。

- 2.1.3 一部の TiKV インスタンスにおける多数の Leader のドロップが発生することがあります。[#4.4](#44-some-tikv-nodes-drop-leader-frequently) を参照してください。

- 2.1.4 その他の原因については、[増加した読み書きのレイテンシについてトラブルシューティングする](/troubleshoot-cpu-issues.md) を参照してください。

### 2.2 持続的で著しい増加

- 2.2.1 TiKV の単一スレッドのボトルネック

    - TiKV インスタンス内の Region が多すぎて単一の gRPC スレッドがボトルネックとなる場合があります（**Grafana** -> **TiKV 詳細** -> **スレッド CPU/gRPC CPU パースレッド** メトリクスを確認してください）。v3.x 以降のバージョンでは、この問題を解決するために `Hibernate Region` を有効にすることができます。[case-612](https://github.com/pingcap/tidb-map/blob/master/maps/diagnose-case-study/case612.md) を参照してください。

    - v3.0 より前のバージョンでは、raftstore スレッドや apply スレッドがボトルネックになる場合（**Grafana** -> **TiKV 詳細** -> **スレッド CPU/raft store CPU** および **Async apply CPU** のメトリクスが `80%` を超える）、TiKV インスタンスをスケーリングアウト（v2.x）するか、マルチスレッディングを備えた v3.x にアップグレードすることができます。<!-- [case-517](https://github.com/pingcap/tidb-map/blob/master/maps/diagnose-case-study/case517.md) を参照してください。 -->

- 2.2.2 CPU ロードが増加する。

- 2.2.3 TiKV の書き込みが遅い。[#4.5](#45-tikv-write-is-slow) を参照してください。

- 2.2.4 誤った TiDB 実行計画。[#3.3](#33-wrong-execution-plan) を参照してください。

- 2.2.5 その他の原因については、[増加した読み書きのレイテンシについてトラブルシューティングする](/troubleshoot-cpu-issues.md) を参照してください。

## 3. TiDB の問題

### 3.1 DDL

- 3.1.1 `decimal` フィールドの長さを変更しようとすると、`ERROR 1105 (HY000): unsupported modify decimal column precision` エラーが報告されることがあります。<!-- [case-1004](https://github.com/pingcap/tidb-map/blob/master/maps/diagnose-case-study/case517.md) を参照してください。--> TiDB では `decimal` フィールドの長さを変更することはサポートされていません。

- 3.1.2 TiDB DDL ジョブが停止したり実行が遅くなったりする場合（`admin show ddl jobs` を使用して DDL 進行状況を確認してください）

    - 原因 1: 他のコンポーネント（PD/TiKV）とのネットワークの問題。

    - 原因 2: TiDB の初期バージョン（v3.0.8 より前）では、高い並行性で多くのゴルーチンが存在し、内部の負荷が高い。

    - 原因 3: 初期バージョン（v2.1.15 および v3.0.0-rc1 より前）では、PD インスタンスが TiDB キーを削除できないため、すべての DDL 変更が 2 つのリースを待たなければならない。

    - その他の原因については、[バグを報告してください](https://github.com/pingcap/tidb/issues/new?labels=type%2Fbug&template=bug-report.md)。

    - 解決策:

        - 原因 1 の場合は、TiDB と TiKV/PD のネットワーク接続を確認してください。
        - 原因 2 および 3 は後のバージョンで既に修正されています。TiDB を最新バージョンにアップグレードすることができます。
        - その他の原因については、DDL オーナーの移行ソリューションを使用できます。

    - DDL オーナーの移行:

        - TiDB サーバーに接続できる場合は、オーナー選出コマンドを再実行してください: `curl -X POST http://{TiDBIP}:10080/ddl/owner/resign`
        - TiDB サーバーに接続できない場合は、PD クラスターの etcd から DDL オーナーを削除して再選出をトリガーします: `tidb-ctl etcd delowner [LeaseID] [flags] + ownerKey`

- 3.1.3 TiDB がログで `information schema is changed` エラーを報告します

    - 詳細な原因と解決策については、[「information schema is changed」エラーが報告される理由](/faq/sql-faq.md#what-triggers-the-information-schema-is-changed-error) を参照してください。

    - 背景: `情報スキーマは変更された` エラーが報告される理由は、各 DDL 変更操作の `schema state` の数が `schema version` の数と一致するためです。たとえば、`create table` 操作は 1 つのバージョン変更を持ち、`add column` 操作は 4 つのバージョン変更を持ちます。そのため、多くのカラム変更操作は `schema version` を速く増加させる可能性があります。詳細については、[オンラインスキーマ変更](https://static.googleusercontent.com/media/research.google.com/zh-CN//pubs/archive/41376.pdf) を参照してください。

- 3.1.4 TiDB がログで `information schema is out of date` を報告します

    - 原因 1: DML ステートメントを含むトランザクションの実行時間が 1 つの DDL リースを超え、トランザクションがコミットされるとエラーが報告されます。

    - 原因 2: DML ステートメントの実行中に TiDB サーバーが PD または TiKV に接続できません。その結果、TiDB サーバーは 1 つの DDL リース（デフォルトで `45s`）内に新しいスキーマを読み込まなかったり、TiDB サーバーが `keep alive` 設定で PD から切断されたりします。

    - 原因 3: TiKV の負荷が高いかネットワークがタイムアウトしています。**Grafana** -> **TiDB** および **TiKV** でノードの負荷を確認してください。

    - 解決策:

        - 原因 1 の場合は、TiDB が起動されるときに DML 操作を再試行してください。
        - 原因 2 の場合は、TiDB サーバーと PD/TiKV 間のネットワークを確認してください。
        - 原因 3 の場合は、なぜ TiKV がビジーかを調査してください。[#4 TiKV issues](#4-tikv-issues) を参照してください。

### 3.2 OOM の問題

- 3.2.1 症状
- クライアント: クライアントはエラー`ERROR 2013 (HY000): Lost connection to MySQL server during query`を報告しています。

    - ログを確認してください

        - `dmesg -T | grep tidb-server`を実行します。その結果は、エラーが発生した時点の周辺にOOMキラーログを表示します。

        - エラー発生後の時間帯（つまりtidb-serverが再起動される時刻）に`tidb.log`で"Welcome to TiDB"のログをgrepします。

        - `tidb_stderr.log`で`fatal error: runtime: out of memory`または`cannot allocate memory`をgrepします。

        - v2.1.8またはそれ以前のバージョンの場合、`tidb_stderr.log`で`fatal error: stack overflow`をgrepできます。

    - モニター: tidb-serverインスタンスのメモリ使用量が短時間で急激に増加しています。

- 3.2.2 OOMを引き起こすSQLステートメントを特定する（現時点で全てのTiDBバージョンは正確にSQLを特定できません。特定のSQLでOOMが発生しているかどうかは引き続き分析する必要があります。）

    - v3.0.0以降のバージョンでは、`tidb.log`で"expensive_query"をgrepします。そのログメッセージには、タイムアウトしたSQLクエリやメモリクォータを超過したSQLクエリが記録されています。

    - v3.0.0未満のバージョンでは、`tidb.log`で"memory exceeds quota"をgrepして、メモリクォータを超過したSQLクエリを特定します。

  > **注意:**
  >
  > 単一のSQLメモリ使用量のデフォルト閾値は`1GB`です。このパラメータはシステム変数[`tidb_mem_quota_query`](/system-variables.md#tidb_mem_quota_query)を設定することで変更できます。

- 3.2.3 OOM問題の緩和

    - `SWAP`を有効にすることで、大きなクエリによるメモリの過剰使用によるOOM問題を緩和できます。メモリが不足すると、この方法は大きなクエリのパフォーマンスにI/Oオーバーヘッドが発生する可能性があります。パフォーマンスへの影響の程度は、残りのメモリ空間とディスクI/Oの速度に依存します。

- 3.2.4 OOMの典型的な理由

    - SQLクエリに`join`が含まれています。`explain`を使用してSQLステートメントを表示すると、`join`操作で`HashJoin`アルゴリズムが選択され、`inner`テーブルが大きいことがわかります。

    - 単一の`UPDATE/DELETE`クエリのデータ量が大きすぎる場合。Chineseの[case-882](https://github.com/pingcap/tidb-map/blob/master/maps/diagnose-case-study/case882.md)を参照してください。

    - SQLに`Union`で接続された複数のサブクエリが含まれています。Chineseの[case-1828](https://github.com/pingcap/tidb-map/blob/master/maps/diagnose-case-study/case1828.md)を参照してください。

OOMのトラブルシューティングの詳細については、[Troubleshoot TiDB OOM Issues](/troubleshoot-tidb-oom.md)を参照してください。

### 3.3 誤った実行計画

- 3.3.1 症状

    - SQLクエリの実行時間が前回の実行と比較してかなり長いか、実行計画が突然変わりました。実行計画が遅いログに記録されている場合、実行計画を直接比較できます。

    - MySQLなど他のデータベースと比較して、SQLクエリの実行時間が長いです。他のデータベースと実行計画を比較して、`Join Order`などの違いを確認します。

    - スローログにおいてSQL実行時間`Scan Keys`の数が多いです。

- 3.3.2 実行計画を調査する

    - `explain analyze {SQL}`を使用します。実行時間が許容範囲内の場合、`explain analyze`の結果の`count`と`execution info`の`row`の数を比較します。`TableScan/IndexScan`行で大きな違いが見つかれば、統計情報が正しくない可能性があります。他の行で大きな違いが見つかった場合、問題は統計情報にある可能性があります。

    - `select count(*)`。実行計画に`join`操作が含まれている場合、`explain analyze`には時間がかかるかもしれません。`TableScan/IndexScan`の条件に対して`select count(*)`を実行し、`explain`結果の`row count`情報を比較します。

- 3.3.3 緩和方法

    - v3.0以降のバージョンでは、`SQL Bind`機能を使用して実行計画をバインドします。

    - 統計情報を更新します。統計情報による問題がほぼ確実であれば、[統計情報をダンプ](/statistics.md#export-statistics)します。更新が必要な統計情報の例としては、`show stats_meta`での`modify count/row count`が一定値（例えば0.3）を上回る場合や、テーブルに時間列のインデックスがある場合などがあります。自動統計情報収集が構成されている場合は、`tidb_auto_analyze_ratio`システム変数が大きすぎる（例えば0.3より大きい）か、現在の時刻が`tidb_auto_analyze_start_time`と`tidb_auto_analyze_end_time`の間にあるかどうかを確認します。

    - その他の状況については、[バグを報告](https://github.com/pingcap/tidb/issues/new?labels=type%2Fbug&template=bug-report.md)してください。

### 3.4 SQL実行エラー

- 3.4.1 クライアントは`ERROR 1265(01000) Data Truncated`エラーを報告しています。これはTiDBの内部的な`Decimal`型の精度の計算がMySQLと互換性がないためです。この問題はv3.0.10で修正されました（[#14438](https://github.com/pingcap/tidb/pull/14438)）。

    - 原因：

        MySQLでは、2つの大きな精度の`Decimal`が除算され、結果が最大のDecimal精度（`30`）を超える場合、最大でも`30`桁だけが残され、エラーは報告されません。

        TiDBでは、計算結果はMySQLと同じですが、`Decimal`を表すデータ構造の内部で、実際の精度を保持するフィールドがまだ存在します。

        例として、`(0.1^30) / 10`を考えてみます。TiDBとMySQLの結果はどちらも`0`ですが、最大でも精度は`30`です。しかし、TiDBではDecimal精度のフィールドは`31`のままです。

        複数の`Decimal`の除算を行うと、結果は正しくても、この精度フィールドはどんどん大きくなり、最終的にTiDBでの閾値（`72`）を超えて`Data Truncated`エラーが報告されます。

        `Decimal`の乗算にはこの問題はありません。領域外に出ることが回避され、精度は最大精度制限に設定されます。

    - 解決方法: これを回避するには、`Cast(xx as decimal(a, b))`を手動で追加できます。ここで、`a`と`b`はターゲットの精度です。

### 3.5 スロークエリの問題

スロークエリを特定する方法については、[Identify slow queries](/identify-slow-queries.md)を参照してください。スロークエリの分析と処理については、[Analyze slow queries](/analyze-slow-queries.md)を参照してください。

### 3.6 ホットスポットの問題

分散データベースとして、TiDBは負荷分散メカニズムを提供し、アプリケーションの負荷をできるだけ均等に異なるコンピューティングまたはストレージノードに分散し、サーバーリソースをより効率的に利用します。ただし、特定のシナリオでは、一部のアプリケーション負荷を適切に分散できないことがあり、パフォーマンスに影響を与え、負荷の高い単一点（ホットスポット）を形成することがあります。

TiDBは、ホットスポットのトラブルシューティング、解決、または回避の完全なソリューションを提供しています。負荷のホットスポットをバランスさせることで、全体的なパフォーマンスが向上し、QPSが改善され、レイテンシが低減されます。詳細な解決策については、[Troubleshoot Hotspot Issues](/troubleshoot-hot-spot-issues.md)を参照してください。

### 3.7 高ディスクI/O使用量

CPUのボトルネックやトランザクションの競合によるボトルネックをトラブルシューティングした後、TiDBの応答が遅くなる場合は、現在のシステムのボトルネックを特定するためにI/Oメトリクスをチェックする必要があります。TiDBでの高I/O使用量の問題を特定し処理する方法については、[Troubleshoot High Disk I/O Usage](/troubleshoot-high-disk-io.md)を参照してください。

### 3.8 ロックの競合

TiDBは完全な分散トランザクションをサポートしています。v3.0から、TiDBは楽観的トランザクションモードと悲観的トランザクションモードを提供しています。ロック関連の問題のトラブルシューティング方法や楽観的・悲観的なロックの競合を処理する方法については、[Troubleshoot Lock Conflicts](/troubleshoot-lock-conflicts.md)を参照してください。

### 3.9 データとインデックスの不整合

TiDBはトランザクションを実行したり、[`ADMIN CHECK [TABLE|INDEX]`](/sql-statements/sql-statement-admin-check-table-index.md)ステートメントを実行した際に、データとインデックスの整合性をチェックします。レコードのキー値とそれに対応するインデックスのキー値が整合しない場合、つまり行データを格納するキー値ペアとそれに対応するインデックスを格納するキー値ペアとが整合しない場合（例: インデックスが多すぎるか、または存在しない）、TiDBはデータ不整合エラーを報告し、関連するエラーをエラーログに出力します。

不整合エラーやその回避方法については、[Troubleshoot Inconsistency Between Data and Indexes](/troubleshoot-data-inconsistency-errors.md)を参照してください。

## 4. TiKVの問題
### 4.1 TiKVがパニックを起こし起動に失敗する

- 4.1.1 `sync-log = false`。マシンの電源が切られた後に`unexpected raft log index: last_index X < applied_index Y`のエラーが返される。

    この問題は予期されています。`tikv-ctl`を使用してリージョンを復元できます。

- 4.1.2 TiKVが仮想マシン上に展開されている場合、仮想マシンが停止されたり、物理マシンの電源が切られると`entries[X, Y] is unavailable from storage`のエラーが報告されます。

    この問題は予期されています。仮想マシンの`fsync`は信頼性がないため、`tikv-ctl`を使用してリージョンを復元する必要があります。

- 4.1.3 その他の予期しない原因については、[バグを報告](https://github.com/tikv/tikv/issues/new?template=bug-report.md)してください。

### 4.2 TiKVのOOM

- 4.2.1 `block-cache`の構成が大きすぎる場合、OOMを引き起こす可能性があります。

    問題の原因を確認するには、監視ツール **Grafana** -> **TiKV-details** でRocksDBの`block cache size`を確認してください。

    また、`[storage.block-cache] capacity = # "1GB"`パラメータが適切に設定されているかどうかも確認してください。デフォルトでは、TiKVの`block-cache`はマシンの合計メモリの`45%`に設定されています。TiKVをコンテナに展開する際は、明示的にこのパラメータを指定する必要があります。なぜなら、TiKVは物理マシンのメモリを取得するため、それがコンテナのメモリ制限を超える可能性があるからです。

- 4.2.2 Coprocessorが多くの大規模クエリを受信し、大量のデータを返しています。このため、gRPCがデータを返すスピードでくださいなくなり、OOMが発生します。

    原因を確認するには、監視ツール **Grafana** -> **TiKV-details** -> **coprocessor overview** で`response size`が`network outbound`トラフィックを超えていないかどうかを確認してください。

- 4.2.3 他のコンポーネントがメモリを多く使用している。

    この問題は予期しないものです。[バグを報告](https://github.com/tikv/tikv/issues/new?template=bug-report.md)してください。

### 4.3 クライアントが`server is busy`のエラーを報告しています

`server is busy`はTiKVのフロー制御メカニズムによって引き起こされ、`tidb/ti-client`にTiKVが現在過度のプレッシャーを感じており、後でリトライするよう通知しています。

- 4.3.1 TiKV RocksDBが`write stall`に遭遇しています。

    TiKVのインスタンスには2つのRocksDBインスタンスがあり、Raftログを保存する`data/raft`と実際のデータを保存する`data/db`です。`grep "Stalling" RocksDB`を実行して、`LOG`で始まるファイル（`LOG`が現在のログです）でスタールの具体的な原因を確認できます。

    - 多くの`level0 sst`がスタールを引き起こしています。`[rocksdb] max-sub-compactions = 2`（または3）パラメータを追加して`level0 sst`のコンパクションを加速することができます。level0からlevel1へのコンパクションタスクは、複数のサブタスク（サブタスクの最大数は`max-sub-compactions`の値）に分割され、並行して実行されます。[case-815](https://github.com/pingcap/tidb-map/blob/master/maps/diagnose-case-study/case815.md)も参照してください。

    - 多くの`pending compaction bytes`がスタールを引き起こしています。ディスクI/Oがビジネスピーク時の書き込み操作のペースに追いついていません。対応するCFの`soft-pending-compaction-bytes-limit`および`hard-pending-compaction-bytes-limit`を増やすことで、この問題を緩和することができます。

        - `[rocksdb.defaultcf] soft-pending-compaction-bytes-limit`のデフォルト値は`64GB`です。ペンディングコンパクションバイトが閾値に達すると、RocksDBは書き込み速度を低下させます。`[rocksdb.defaultcf] soft-pending-compaction-bytes-limit`を`128GB`に設定できます。

        - `hard-pending-compaction-bytes-limit`のデフォルト値は`256GB`です。ペンディングコンパクションバイトが閾値に達すると（これはほぼ起こらない可能性が高いですが、ペンディングコンパクションバイトが`soft-pending-compaction-bytes-limit`に達した後にRocksDBは書き込みを停止します）、RocksDBは書き込み操作を停止します。`hard-pending-compaction-bytes-limit`を`512GB`に設定できます。

        - ディスクI/O容量が長時間にわたって書き込みに追いつかない場合は、ディスクをスケーリングアップすることをお勧めします。ディスクのスループットが上限に達し、書き込みスタールが発生する場合（たとえば、SATA SSDがNVME SSDよりもはるかに低い場合）、より高い圧縮率の圧縮アルゴリズムを適用することで、CPUリソースと引き換えにディスクリソースが得られ、ディスクの負荷が軽減されます。

        - デフォルトCFのコンパクションに高い圧力がかかった場合、`[rocksdb.defaultcf] compression-per-level`パラメータを`["no", "no", "zstd", "zstd", "zstd", "zstd", "zstd"]`から`["no", "no", "lz4", "lz4", "lz4", "zstd", "zstd"]`に変更します。

    - 多くのメムテーブルがスタールを引き起こしています。これは通常、即時書き込みの量が多く、メムテーブルがディスクにゆっくりフラッシュされる場合に発生します。ディスクの書き込み速度を向上させることができず、この問題がビジネスピーク時にのみ発生する場合は、対応するCFの`max-write-buffer-number`を増やすことで緩和することができます。

        - 例えば、`[rocksdb.defaultcf] max-write-buffer-number`をデフォルトの`5`から`8`に設定します。なお、これによりピーク時にメモリ使用量が増える可能性があるため、注意してください。

- 4.3.2 `scheduler too busy`

    - 重大な書き込み競合が発生しています。`latch wait duration`が高くなっています。`latch wait duration`は、監視ツール **Grafana** -> **TiKV-details** -> **scheduler prewrite**/**scheduler commit** で確認できます。スケジューラに書き込みタスクが積み重なり、保留中の書き込みタスクが`[storage] scheduler-pending-write-threshold`（100MB）で設定した閾値を超えている場合、原因を確認することができます。

    - 書き込みが遅く、書き込まれるデータが`[storage] scheduler-pending-write-threshold`（100MB）で設定された閾値を超えることで書き込みタスクが積み重なっています。[4.5](#45-tikv-write-is-slow)を参照してください。

- 4.3.3 `raftstore is busy`. メッセージの処理が受信よりも遅くなっています。短期的な`channel full`状態はサービスに影響を与えませんが、長期にわたるエラーが続くとリーダースイッチを引き起こす可能性があります。

    - `append log`がスタールに遭遇しています。[4.3.1](#43-the-client-reports-the-server-is-busy-error)を参照してください。
    - `append log duration`が高く、これがメッセージの処理が遅くなる原因です。`append log duration`が高い理由については、[4.5](#45-tikv-write-is-slow)を参照してください。
    - raftstoreが一度に大量のメッセージを受け取り、処理に失敗しています（TiKV Raft messagesダッシュボードで確認）。通常、短期的な`channel full`状態はサービスに影響を与えません。

- 4.3.4 TiKV coprocessorが待機しています。たまっているタスクの数が`coprocessor threads * readpool.coprocessor.max-tasks-per-worker-[normal|low|high]`を超えています。大量の大規模クエリがcoprocessorでタスクがたまっています。実行計画の変更によってテーブルスキャン操作が大量に発生したかどうかを確認する必要があります。[3.3](#33-wrong-execution-plan)も参照してください。

### 4.4 一部のTiKVノードが頻繁にLeaderをドロップする

- 4.4.1 TiKVの再起動による再選出

    - TiKVがパニックを起こした後、systemdによって起動され正常に動作します。この問題が予期しない場合は、発生した場合は[バグを報告](https://github.com/tikv/tikv/issues/new?template=bug-report.md)してください。

    - TiKVが第三者によって停止または終了され、systemdによって起動されます。`dmesg`とTiKVのログを確認して原因を確認してください。

    - TiKVがOOMで再起動が発生しています。[4.2](#42-tikv-oom)を参照してください。

    - TiKVが動的に`THP`（Transparent Hugepage）を調整しているためにハングしています。[case-500](https://github.com/pingcap/tidb-map/blob/master/maps/diagnose-case-study/case500.md)も参照してください。
- 4.4.2 TiKV RocksDB encounters write stall and thus results in re-election. You can check if the monitor **Grafana** -> **TiKV-details** -> **errors** shows `server is busy`. Refer to [4.3.1](#43-the-client-reports-the-server-is-busy-error).

- 4.4.3 Re-election because of network isolation.

### 4.5 TiKV write is slow

- 4.5.1 Check whether the TiKV write is low by viewing the `prewrite/commit/raw-put` duration of TiKV gRPC (only for RawKV clusters). Generally, you can locate the slow phase according to the [performance-map](https://github.com/pingcap/tidb-map/blob/master/maps/performance-map.png). Some common situations are listed as follows.

- 4.5.2 The scheduler CPU is busy (only for transaction kv).

    The `scheduler command duration` of prewrite/commit is longer than the sum of `scheduler latch wait duration` and `storage async write duration`. The scheduler worker has a high CPU demand, such as over 80% of `scheduler-worker-pool-size` * 100%, or the CPU resources of the entire machine are relatively limited. If the write workload is large, check if `[storage] scheduler-worker-pool-size` is set too small.

    For other situations, [report a bug](https://github.com/tikv/tikv/issues/new?template=bug-report.md).

- 4.5.3 Append log is slow.

    The **Raft IO**/`append log duration` in TiKV Grafana is high, usually because the disk write operation is slow. You can verify the cause by checking the `WAL Sync Duration max` value of RocksDB - raft.

    For other situations, [report a bug](https://github.com/tikv/tikv/issues/new?template=bug-report.md).

- 4.5.4 The raftstore thread is busy.

    The **Raft Propose**/`propose wait duration` is significantly larger than the append log duration in TiKV Grafana. Take the following methods:

    - Check whether the `[raftstore] store-pool-size` configuration value is too small. It is recommended to set the value between `1` and `5` and not too large.
    - Check whether the CPU resources on the machine are insufficient.

- 4.5.5 Apply is slow.

    The **Raft IO**/`apply log duration` in TiKV Grafana is high, which usually comes with a high **Raft Propose**/`apply wait duration`. The possible causes are as follows:

    - `[raftstore] apply-pool-size` is too small (it is recommended to set the value between `1` and `5` and not too large), and the **Thread CPU**/`apply CPU` is large.

    - The CPU resources on the machine are insufficient.

    - Region write hot spot. A single apply thread has high CPU usage. Currently, we cannot properly address the hot spot problem on a single Region, which is being improved. To view the CPU usage of each thread, modify the Grafana expression and add `by (instance, name)`.

    - RocksDB write is slow. **RocksDB kv**/`max write duration` is high. A single Raft log might contain multiple KVs. When writing into RocksDB, 128 KVs are written into RocksDB in a write batch. Therefore, an apply log might be associated with multiple writes in RocksDB.

    - For other situations, [report a bug](https://github.com/tikv/tikv/issues/new?template=bug-report.md).

- 4.5.6 Raft commit log is slow.

    The **Raft IO**/`commit log duration` in TiKV Grafana is high (this metric is only supported in Grafana after v4.x). Every Region corresponds to an independent Raft group. Raft has a flow control mechanism, similar to the sliding window mechanism of TCP. You can control the size of the sliding window by configuring the `[raftstore] raft-max-inflight-msgs = 256` parameter. If there is a write hot spot and the `commit log duration` is high, you can adjust the parameter, such as increasing it to `1024`.

- 4.5.7 For other situations, refer to the write path on [performance-map](https://github.com/pingcap/tidb-map/blob/master/maps/performance-map.png) and analyze the cause.

## 5. PD issues

### 5.1 PD scheduling

- 5.1.1 Merge

    - Empty Regions across tables cannot be merged. You need to modify the `[coprocessor] split-region-on-table` parameter in TiKV, which is set to `false` in v4.x by default. See [case-896](https://github.com/pingcap/tidb-map/blob/master/maps/diagnose-case-study/case896.md) in Chinese.

    - Region merge is slow. You can check whether the merged operator is generated by accessing the monitor dashboard in **Grafana** -> **PD** -> **operator**. To accelerate the merge, increase the value of `merge-schedule-limit`.

- 5.1.2 Add replicas or take replicas online/offline

    - The TiKV disk uses 80% of the capacity, and PD does not add replicas. In this situation, the number of miss peers increases, so TiKV needs to be scaled out. See [case-801](https://github.com/pingcap/tidb-map/blob/master/maps/diagnose-case-study/case801.md) in Chinese.

    - When a TiKV node is taken offline, some Region cannot be migrated to other nodes. This issue has been fixed in v3.0.4 ([#5526](https://github.com/tikv/tikv/pull/5526)). See [case-870](https://github.com/pingcap/tidb-map/blob/master/maps/diagnose-case-study/case870.md) in Chinese.

- 5.1.3 Balance

    - The Leader/Region count is not evenly distributed. See [case-394](https://github.com/pingcap/tidb-map/blob/master/maps/diagnose-case-study/case394.md) and [case-759](https://github.com/pingcap/tidb-map/blob/master/maps/diagnose-case-study/case759.md) in Chinese. The major cause is that the balance performs scheduling based on the size of Region/Leader, so this might result in the uneven distribution of the count. In TiDB 4.0, the `[leader-schedule-policy]` parameter is introduced, which enables you to set the scheduling policy of Leader to be `count`-based or `size`-based.

### 5.2 PD election

- 5.2.1 PD switches Leader.

    - Cause 1: Disk. The disk where the PD node is located has full I/O load. Investigate whether PD is deployed with other components with high I/O demand and the health of the disk. You can verify the cause by viewing the monitor metrics in **Grafana** -> **disk performance** -> **latency**/**load**. You can also use the FIO tool to run a check on the disk if necessary. See [case-292](https://github.com/pingcap/tidb-map/blob/master/maps/diagnose-case-study/case292.md) in Chinese.

    - Cause 2: Network. The PD log shows `lost the TCP streaming connection`. You need to check whether there is a problem with the network between PD nodes and verify the cause by viewing `round trip` in the monitor **Grafana** -> **PD** -> **etcd**. See [case-177](https://github.com/pingcap/tidb-map/blob/master/maps/diagnose-case-study/case177.md) in Chinese.

    - Cause 3: High system load. The log shows `server is likely overloaded`. See [case-214](https://github.com/pingcap/tidb-map/blob/master/maps/diagnose-case-study/case214.md) in Chinese.

- 5.2.2 PD cannot elect a Leader or the election is slow.

    - PD cannot elect a Leader: The PD log shows `lease is not expired`. [This issue](https://github.com/etcd-io/etcd/issues/10355) has been fixed in v3.0.x and v2.1.19. See [case-875](https://github.com/pingcap/tidb-map/blob/master/maps/diagnose-case-study/case875.md) in Chinese.

    - The election is slow: The Region loading duration is long. You can check this issue by running `grep "regions cost"` in the PD log. If the result is in seconds, such as `load 460927 regions cost 11.77099s`, it means the Region loading is slow. You can enable the `region storage` feature in v3.0 by setting `use-region-storage` to `true`, which significantly reduce the Region loading duration. See [case-429](https://github.com/pingcap/tidb-map/blob/master/maps/diagnose-case-study/case429.md) in Chinese.
- 5.2.3 TiDBがSQLステートメントを実行する際にPDがタイムアウトしました。

    - PDにリーダーが存在しないか、リーダーが切り替わります。[5.2.1](#52-pd-election) と [5.2.2](#52-pd-election) を参照してください。

    - ネットワークの問題です。モニター **Grafana** -> **blackbox_exporter** -> **ping latency** にアクセスしてTiDBからPDリーダーへのネットワークが正常に動作しているか確認してください。

    - PDがパニックを起こしています。[バグを報告する](https://github.com/pingcap/pd/issues/new?labels=kind%2Fbug&template=bug-report.md)。

    - PDがOOMになっています。[5.3](#53-pd-oom) を参照してください。

    - 問題に他の原因がある場合は、`curl http://127.0.0.1:2379/debug/pprof/goroutine?debug=2` を実行してgoroutineを取得し、[バグを報告する](https://github.com/pingcap/pd/issues/new?labels=kind%2Fbug&template=bug-report.md)。

- 5.2.4 その他の問題

    - PDが`FATAL`エラーを報告し、ログに`range failed to find revision pair`が表示されます。この問題はv3.0.8で修正されています([#2040](https://github.com/pingcap/pd/pull/2040))。詳細については、[case-947](https://github.com/pingcap/tidb-map/blob/master/maps/diagnose-case-study/case947.md) をご覧ください。

    - その他の状況については、[バグを報告する](https://github.com/pingcap/pd/issues/new?labels=kind%2Fbug&template=bug-report.md)。

### 5.3 PD OOM

- 5.3.1 `/api/v1/regions` インターフェースを使用すると、PDのOOMが発生する場合があります。この問題はv3.0.8で修正されています([#1986](https://github.com/pingcap/pd/pull/1986))。

- 5.3.2 ローリングアップグレード中にPDがOOMします。gRPCメッセージのサイズに制限がなく、モニターによるとTCP InSegsが比較的大きいことが表示されます。この問題はv3.0.6で修正されています([#1952](https://github.com/pingcap/pd/pull/1952))。<!--詳細については、[case-852](https://github.com/pingcap/tidb-map/blob/master/maps/diagnose-case-study/case852.md) をご覧ください。-->

### 5.4 Grafanaの表示

- 5.4.1 **Grafana** -> **PD** -> **cluster** -> **role** のモニターでフォロワーが表示されます。このGrafanaの表現の問題はv3.0.8で修正されています。

## 6. 生態系ツール

### 6.1 TiDB Binlog

- 6.1.1 TiDB Binlogは、TiDBからの変更を収集し、下流のTiDBやMySQLプラットフォームへのバックアップとレプリケーションを提供するツールです。詳細については、[TiDB Binlog on GitHub](https://github.com/pingcap/tidb-binlog) をご覧ください。

- 6.1.2 Pump/Drainerのステータスの`Update Time`が正常に更新されており、ログに異常が表示されていませんが、データが下流に書き込まれていません。

    - TiDB構成でBinlogが有効になっていません。TiDBの`[binlog]`構成を変更してください。

- 6.1.3 Drainerの`sarama`が`EOF`エラーを報告しています。

    - DrainerのKafkaクライアントのバージョンがKafkaのバージョンと一致していません。`[syncer.to] kafka-version`構成を変更する必要があります。

- 6.1.4 DrainerがKafkaに書き込めず、パニックが発生し、Kafkaが`Message was too large`エラーを報告しています。

    - Binlogデータが大きすぎるため、Kafkaに書き込まれる単一のメッセージが大きすぎます。Kafkaの以下の構成を変更する必要があります:

        ```conf
        message.max.bytes=1073741824
        replica.fetch.max.bytes=1073741824
        fetch.message.max.bytes=1073741824
        ```

        詳細については、[case-789](https://github.com/pingcap/tidb-map/blob/master/maps/diagnose-case-study/case789.md) をご覧ください。

- 6.1.5 上流と下流のデータが一貫していません

    - 一部のTiDBノードでBinlogが有効になっていません。v3.0.6以降のバージョンでは、<http://127.0.0.1:10080/info/all> インターフェースにアクセスしてすべてのノードのbinlogステータスを確認できます。v3.0.6より前のバージョンでは、構成ファイルを表示してbinlogのステータスを確認できます。

    - 一部のTiDBノードが`ignore binlog`ステータスになっています。v3.0.6以降のバージョンでは、<http://127.0.0.1:10080/info/all> インターフェースにアクセスしてすべてのノードのbinlogステータスを確認できます。v3.0.6より前のバージョンでは、TiDBログを表示して`ignore binlog`キーワードが含まれているか確認できます。

    - 上流と下流のタイムスタンプ列の値が一貫していません。

        - これは異なるタイムゾーンによるものです。Drainerが上流と下流のデータベースと同じタイムゾーンにあることを確認する必要があります。Drainerは`/etc/localtime`からタイムゾーンを取得し、`TZ`環境変数をサポートしていません。詳細については、[case-826](https://github.com/pingcap/tidb-map/blob/master/maps/diagnose-case-study/case826.md) をご覧ください。

        - TiDBでは、タイムスタンプのデフォルト値は`null`ですが、MySQL 5.7（MySQL 8を除く）では同じデフォルト値は現在時刻です。したがって、上流のTiDBのタイムスタンプが`null`であり、下流がMySQL 5.7の場合、タイムスタンプ列のデータが一貫していません。Binlogを有効にする前に上流で`set @@global.explicit_defaults_for_timestamp=on;`を実行する必要があります。

    - その他の状況については、[バグを報告する](https://github.com/pingcap/tidb-binlog/issues/new?labels=bug&template=bug-report.md)。

- 6.1.6 レプリケーションが遅い

    - 下流がTiDB/MySQLであり、上流で頻繁なDDL操作が行われています。詳細については、[case-1023](https://github.com/pingcap/tidb-map/blob/master/maps/diagnose-case-study/case1023.md) をご覧ください。

    - 下流がTiDB/MySQLであり、レプリケーションされるテーブルに主キーまたはユニークインデックスが存在しないため、Binlogのパフォーマンスが低下しています。主キーまたはユニークインデックスを追加することをお勧めします。

    - 下流がファイルに出力される場合は、出力ディスクまたはネットワークディスクが遅いかどうかを確認してください。

    - その他の状況については、[バグを報告する](https://github.com/pingcap/tidb-binlog/issues/new?labels=bug&template=bug-report.md)。

- 6.1.7 PumpがBinlogを書き込めず、`no space left on device`エラーを報告しています。

    - Pumpが正常にBinlogデータを書き込むためのローカルディスク容量が不足しています。ディスク容量をクリアしてからPumpを再起動してください。

- 6.1.8 Pumpが起動される際に`fail to notify all living drainer`エラーを報告しています。

    - 原因: Pumpが起動されると、`online`状態のすべてのDrainerノードに通知します。Drainerに通知できない場合、このエラーログが出力されます。

    - 解決方法: binlogctlツールを使用して各Drainerノードが正常かどうかを確認してください。`online`状態のすべてのDrainerノードが正常に動作していることを確認するためです。Drainerノードの状態が実際の動作状況と一致していない場合は、binlogctlツールを使用してその状態を変更し、その後にPumpを再起動してください。case [fail-to-notify-all-living-drainer](/tidb-binlog/handle-tidb-binlog-errors.md#fail-to-notify-all-living-drainer-is-returned-when-pump-is-started) をご覧ください。

- 6.1.9 Drainerが`gen update sqls failed: table xxx: row data is corruption []`エラーを報告しています。

    - トリガー: 上流でこのテーブルに対して`DROP COLUMN` DDLを実行する際にそのテーブルにDML操作が行われた場合に発生します。この問題はv3.0.6で修正されています。[case-820](https://github.com/pingcap/tidb-map/blob/master/maps/diagnose-case-study/case820.md) をご覧ください。

- 6.1.10 Drainerのレプリケーションが中断しています。プロセスはアクティブのままですが、チェックポイントが更新されません。

    - この問題はv3.0.4で修正されています。[case-741](https://github.com/pingcap/tidb-map/blob/master/maps/diagnose-case-study/case741.md) をご覧ください。

- 6.1.11 任意のコンポーネントがパニックしました。

    - [バグを報告する](https://github.com/pingcap/tidb-binlog/issues/new?labels=bug&template=bug-report.md)。


### 6.2 データ移行
- 6.2.1 TiDB Data Migration (DM)は、MySQL/MariaDBからTiDBへのデータ移行をサポートする移行ツールです。詳細は[GitHubのDMページ](https://github.com/pingcap/dm/)をご覧ください。

- 6.2.2 `query status`を実行するか、ログを確認すると`Access denied for user 'root'@'172.31.43.27' (using password: YES)`と表示されます。

    - DMのすべての設定ファイルにおけるデータベース関連のパスワードは`dmctl`によって暗号化する必要があります。データベースのパスワードが空の場合、パスワードを暗号化する必要はありません。v1.0.6以降では平文パスワードを使用できます。
    - DM操作中、上流および下流のデータベースのユーザーはそれぞれ対応する読み取りおよび書き込み権限を持つ必要があります。データ移行は、データ複製タスクの起動時に自動的に[対応する権限を事前チェック](/dm/dm-precheck.md)します。
    - DMクラスターにおける異なるバージョンのDM-worker/DM-master/dmctlを展開する場合は、[AskTUGのケーススタディ](https://asktug.com/t/dm1-0-0-ga-access-denied-for-user/1049/5)を参照してください。

- 6.2.3 複製タスクが`driver: bad connection`エラーで中断された場合、

    - `driver: bad connection`エラーは、DMと下流のTiDBデータベースの接続に異常が発生したことを示し（例えばネットワーク障害やTiDBの再起動）、現在のリクエストのデータがまだTiDBに送信されていないことを表します。

        - DM 1.0.0 GAより前のバージョンの場合、`stop-task`を実行してタスクを停止し、その後`start-task`を実行してタスクを再開します。
        - DM 1.0.0 GA以上のバージョンの場合、このタイプのエラーの自動リトライ機構が追加されています。[#265](https://github.com/pingcap/dm/pull/265)を参照してください。

- 6.2.4 複製タスクが`invalid connection`エラーで中断された場合、

    - `invalid connection`エラーは、DMと下流のTiDBデータベースの接続に異常が発生したことを示し（例えばネットワーク障害、TiDBの再起動、TiKVのビジー状態）、現在のリクエストの一部のデータがTiDBに送信されていることを表します。複製タスクにおいて、DMは下流にデータを同時に複製する機能があり、そのためタスクが中断されるといくつかのエラーが発生する可能性があります。これらのエラーは`query-status`または`query-error`を実行して確認できます。

        - 増分複製プロセス中に`invalid connection`エラーのみが発生した場合、DMはタスクを自動的にリトライします。
        - DMが自動的にリトライしないか、バージョンの問題で自動的なリトライに失敗した場合（自動リトライはv1.0.0-rc.1で導入されています）、`stop-task`を使用してタスクを停止し、その後`start-task`を使用してタスクを再開します。

- 6.2.5 リレー単位が`event from * in * diff from passed-in event *`エラーを報告する、または`get binlog error ERROR 1236 (HY000) and binlog checksum mismatch, data may be corrupted returned`などの未処理可能なエラーが発生する場合、

    - DMがリレーログを取得するプロセスや増分複製プロセスにおいて、上流のバイナリログファイルのサイズが4 GBを超えると、これらの2つのエラーが発生する可能性があります。

    - 原因: リレーログの書き込み時、DMはバイナリログ位置とバイナリログファイルのサイズに基づいてイベントの検証を行い、複製したバイナリログ位置をチェックポイントとして保存する必要があります。ただし、公式のMySQLはバイナリログ位置をuint32で保存しているため、4 GBを超えるバイナリログファイルのバイナリログ位置がオーバーフローし、上記のエラーが発生します。

    - 対処方法:

        - リレープロセス単位については、[複製を手動で回復](https://pingcap.com/docs/tidb-data-migration/dev/error-handling/#the-relay-unit-throws-error-event-from--in--diff-from-passed-in-event--or-a-replication-task-is-interrupted-with-failing-to-get-or-parse-binlog-errors-like-get-binlog-error-error-1236-hy000-and-binlog-checksum-mismatch-data-may-be-corrupted-returned)してください。
        - バイナリログ複製プロセス単位については、[複製を手動で回復](https://pingcap.com/docs/tidb-data-migration/dev/error-handling/#the-relay-unit-throws-error-event-from--in--diff-from-passed-in-event--or-a-replication-task-is-interrupted-with-failing-to-get-or-parse-binlog-errors-like-get-binlog-error-error-1236-hy000-and-binlog-checksum-mismatch-data-may-be-corrupted-returned)してください。

- 6.2.6 DM複製が中断し、ログに`ERROR 1236 (HY000) The slave is connecting using CHANGE MASTER TO MASTER_AUTO_POSITION = 1, but the master has purged binary logs containing GTIDs that the slave requires.`と返される場合、

    - マスターバイナリログがパージされていないかを確認してください。
    - `relay.meta`に記録された位置情報を確認してください。

        - `relay.meta`には空のGTID情報が記録されています。DM-workerは終了時または30秒ごとに上流のGTID情報を`relay.meta`に記録します。DM-workerが上流のGTID情報を取得しないと、空のGTID情報が`relay.meta`に保存されます。中国語の[case-772](https://github.com/pingcap/tidb-map/blob/master/maps/diagnose-case-study/case772.md)を参照してください。

        - `relay.meta`に記録されたバイナリログイベントが不完全な回復プロセスをトリガーし、誤ったGTID情報が記録される場合があります。この問題はv1.0.2で修正されており、以前のバージョンで発生する可能性があります。<!-- 中国語の[case-764](https://github.com/pingcap/tidb-map/blob/master/maps/diagnose-case-study/case764.md)を参照してください。-->

- 6.2.7 DM複製プロセスが`Error 1366: incorrect utf8 value eda0bdedb29d(\ufffd\ufffd\ufffd\ufffd\ufffd\ufffd)`というエラーを返す場合、

    - この値はMySQL 8.0またはTiDBに正常に書き込むことができませんが、MySQL 5.7には書き込むことができます。`tidb_skip_utf8_check`パラメータを有効にすることで、データ形式のチェックをスキップできます。

### 6.3 TiDB Lightning

- 6.3.1 TiDB Lightningは、大量のデータをTiDBクラスターに高速にフルインポートするツールです。[GitHubのTiDB Lightningページ](https://github.com/pingcap/tidb/tree/master/br/pkg/lightning)をご覧ください。

- 6.3.2 インポート速度が遅すぎる場合、

    - `region-concurrency`が設定されすぎてスレッド衝突が発生し、パフォーマンスが低下している可能性があります。トラブルシューティングの方法は次の3つです。

        - ログの先頭から`region-concurrency`を検索して設定を確認できます。
        - TiDB Lightningが他のサービス（例えばImporter）とサーバーを共有している場合は、そのサーバーのCPUコア数の75％に`region-concurrency`を手動で設定する必要があります。
        - CPUにクォータがある場合（例えば、Kubernetesの設定で制限されている場合）、TiDB Lightningがこれを読み取れないかもしれません。この場合も`region-concurrency`を手動で減らす必要があります。

    - さらにインデックスを追加するたびに、各行ごとに新しいKVペアが導入されます。N個のインデックスがある場合、インポートする実際のサイズはおおよそ（N+1）倍の[Dumpling](/dumpling-overview.md)出力サイズになります。インデックスが無視できる場合は、インポートが完了したらまずスキーマから削除し、その後`CREATE INDEX`を使用して再追加してください。
    - TiDB Lightningのバージョンが古い可能性があります。インポート速度が向上する可能性のある最新バージョンを試してみてください。

- 6.3.3 `checksum failed: checksum mismatched remote vs local`。

    - 原因1: テーブルにはすでにデータがある可能性があります。古いデータは最終的なチェックサムに影響を与える可能性があります。

    - 原因2: ターゲットデータベースのチェックサムが0の場合、つまりデータがインポートされていない可能性があるため、クラスターが過負荷でデータを受け入れられない可能性があります。

    - 原因3: データソースがマシンによって生成され、[Dumpling](/dumpling-overview.md)でバックアップされていない場合、テーブルの制約を尊重する必要があります。例:

        - `AUTO_INCREMENT`列は正の値である必要があり、値が「0」を含んではなりません。
        - UNIQUEおよびPRIMARY KEYには重複したエントリが含まれていてはなりません。

    - 対処方法: [トラブルシューティングソリューション](/tidb-lightning/troubleshoot-tidb-lightning.md#checksum-failed-checksum-mismatched-remote-vs-local)を参照してください。

- 6.3.4 `Checkpoint for … has invalid status:(error code)`

    - 原因: チェックポイントが有効になっており、Lightning/Importerが以前に異常終了したことがあります。誤ってデータの破損を防ぐため、TiDB Lightningはエラーが解消されるまで開始しません。エラーコードは25未満の整数で、可能な値は`0, 3, 6, 9, 12, 14, 15, 17, 18, 20および21`です。整数値はインポートプロセスの途中で予期しない終了が発生したステップを示します。
    - 解決策: [トラブルシューティング解決策](/tidb-lightning/troubleshoot-tidb-lightning.md#checkpoint-for--has-invalid-status-error-code)を参照してください。

- 6.3.5 `cannot guess encoding for input file, please convert to UTF-8 manually`

    - 原因: TiDB LightningはUTF-8とGB-18030エンコーディングのみをサポートしています。このエラーはそのいずれのエンコーディングにも含まれていないことを意味します。また、歴史的なALTER TABLE実行により、UTF-8とGB-18030の文字列が混在している可能性もあります。

    - 解決策: [トラブルシューティング解決策](/tidb-lightning/troubleshoot-tidb-lightning.md#cannot-guess-encoding-for-input-file-please-convert-to-utf-8-manually)を参照してください。

- 6.3.6 `[sql2kv] sql encode error = [types:1292]invalid time format: '{1970 1 1 0 45 0 0}'`

    - 原因: タイムスタンプ型エントリに存在しない時間値が含まれています。これは、夏時間の変更や、時間値がサポートされている範囲（1970年1月1日から2038年1月19日まで）を超えていることが原因です。

    - 解決策: [トラブルシューティング解決策](/tidb-lightning/troubleshoot-tidb-lightning.md#sql2kv-sql-encode-error--types1292invalid-time-format-1970-1-1-)を参照してください。

## 7. 共通のログ解析

### 7.1 TiDB

- 7.1.1 `GC life time is shorter than transaction duration`.

    トランザクションの期間がGCライフタイム（デフォルトで10分）を超過しています。

    [`tidb_gc_life_time`](/system-variables.md#tidb_gc_life_time-new-in-v50)システム変数を変更することでGCライフタイムを延長できます。ただし、このパラメータを変更することは推奨されていません。なぜなら、このトランザクションが多くの`UPDATE`や`DELETE`文を含む場合、古いバージョンがたくさんたまる可能性があるためです。

- 7.1.2 `txn takes too much time`.

    このエラーは、コミットされていないトランザクションが長時間（590秒以上）経過した場合に返されます。

    アプリケーションがこのような長時間のトランザクションを実行する必要がある場合は、`[tikv-client] max-txn-time-use = 590`パラメータとGCライフタイムを増やすことでこの問題を回避できます。アプリケーションがこのような長いトランザクション時間を必要とするかどうかを確認することをお勧めします。

- 7.1.3 `coprocessor.go`が`request outdated`を報告。

    このエラーは、TiKVに送信されたコプロセッサリクエストがTiKVで60秒以上待機している場合に返されます。

    TiKVのコプロセッサが長時間キューに入っている原因を調査する必要があります。

- 7.1.4 `region_cache.go`で大量の`switch region peer to next due to send request fail`が報告され、エラーメッセージが`context deadline exceeded`です。

    TiKVへのリクエストがタイムアウトし、リージョンキャッシュがリクエストを他のノードに切り替えるトリガーになります。ログの`addr`フィールドで`grep "<addr> cancelled`コマンドを実行し、`grep`の結果に従って次の手順を実行できます。

    - `send request is cancelled`: 送信フェーズでリクエストがタイムアウトしました。監視 **Grafana** -> **TiDB** -> **Batch Client**/`Pending Request Count by TiKV` を調べ、Pending Request Countが128より大きいかどうかを確認してください:

        - 値が128より大きい場合、送信がKVの処理能力を超えているために送信がたまります。
        - 値が128より大きくない場合は、対応するKVの操作と保守変更による報告であるかログを確認し、それ以外の場合はこのエラーが予期しないものであり、[バグを報告](https://github.com/pingcap/tidb/issues/new?labels=type%2Fbug&template=bug-report.md)する必要があります。

    - `wait response is cancelled`: TiKVに送信後、リクエストがタイムアウトしました。その際の対応するTiKVアドレスの応答時間とPDおよびその時のKVのリージョンログをチェックする必要があります。

- 7.1.5 `distsql.go`が`inconsistent index`を報告。

    データインデックスが一貫していないようです。報告されたインデックスがあるテーブルで`admin check table <TableName>`コマンドを実行してください。チェックが失敗した場合は、次のコマンドを実行してガベージコレクションを無効にし、[バグを報告](https://github.com/pingcap/tidb/issues/new?labels=type%2Fbug&template=bug-report.md)してください:

    ```sql
    SET GLOBAL tidb_gc_enable = 0;
    ```

### 7.2 TiKV

- 7.2.1 `key is locked`.

    読み取りと書き込みに競合があります。読み取りリクエストはコミットされていないデータに遭遇し、データがコミットされるまで待機する必要があります。

    このエラーがわずかしかない場合、ビジネスに影響はありませんが、このエラーが多い場合は、読み書きの競合がビジネスに影響を及ぼす可能性があります。

- 7.2.2 `write conflict`.

    これは楽観的トランザクションでの書き込み競合です。複数のトランザクションが同じキーを変更する場合、1つのトランザクションのみが成功し、他のトランザクションは自動的にタイムスタンプを再取得して操作を再試行します。これにより、ビジネスに影響はありません。

    競合が深刻な場合、複数のリトライ後にトランザクションが失敗する可能性があります。この場合、悲観的ロックを使用することをお勧めします。エラーと解決策の詳細については、[楽観的トランザクションでの書き込み競合のトラブルシューティング](/troubleshoot-write-conflicts.md)を参照してください。

- 7.2.3 `TxnLockNotFound`.

    このトランザクションのコミットが遅すぎるため、他のトランザクションによってロールバックされました。このトランザクションは自動的にリトライされるため、通常ビジネスに影響はありません。サイズが0.25 MB以下のトランザクションに対して、デフォルトのTTLは3秒です。詳細は[`LockNotFound`エラー](/troubleshoot-lock-conflicts.md#locknotfound-error)を参照してください。

- 7.2.4 `PessimisticLockNotFound`.

    `TxnLockNotFound`と類似しています。悲観的なトランザクションのコミットが遅く、他のトランザクションによってロールバックされました。

- 7.2.5 `stale_epoch`.

    リクエストのエポックが古くなっており、TiDBがルーティングを更新してからリクエストを再送します。ビジネスには影響しません。リージョンが分割/統合されたり、レプリカが移行したりすると、エポックが変化します。

- 7.2.6 `peer is not leader`.

    リクエストがリーダーでないレプリカに送信されました。エラー応答が最新のリーダーを示している場合、TiDBはローカルのルーティングをエラーに従って更新し、最新のリーダーに新しいリクエストを送信します。通常、ビジネスには影響しません。

    v3.0以降のバージョンでは、以前のリーダーへのリクエストが失敗した場合、TiDBは他のピアを試みることがあり、「peer is not leader」がTiKVログで頻繁に発生する可能性があります。対応するリージョンの「switch region peer to next due to send request fail」のログを確認し、送信の失敗の根本原因を確定することができます。詳細は[7.1.4](#71-tidb)を参照してください。

    このエラーは、他の理由でリージョンがリーダーを持たない場合にも返されることがあります。詳細については、[4.4](#44-some-tikv-nodes-drop-leader-frequently)を参照してください。