---
title: TiDB 3.0.1リリースノート
aliases: ['/docs/dev/releases/release-3.0.1/','/docs/dev/releases/3.0.1/']
---

# TiDB 3.0.1リリースノート

リリース日: 2019年7月16日

TiDBバージョン: 3.0.1

TiDB Ansibleバージョン: 3.0.1

## TiDB

+ `MAX_EXECUTION_TIME`機能のサポートを追加しました [#11026](https://github.com/pingcap/tidb/pull/11026)
+ リージョンの分割のバックオフ時間を制御するための`tidb_wait_split_region_finish_backoff`セッション変数を追加しました [#11166](https://github.com/pingcap/tidb/pull/11166)
+ インクリメンタルギャップのロードに基づいて自動調整し、その範囲は1000〜2000000であるauto-increment IDに割り当てられた増分ギャップをサポートしました [#11006](https://github.com/pingcap/tidb/pull/11006)
+ 動的にプラグインを有効または無効にするための`ADMIN PLUGINS ENABLE`/`ADMIN PLUGINS DISABLE` SQLステートメントを追加しました [#11157](https://github.com/pingcap/tidb/pull/11157)
+ 監査プラグインにセッション接続情報を追加しました [#11013](https://github.com/pingcap/tidb/pull/11013)
+ リージョンの分割期間中のデフォルトの動作を、PDにスケジューリングを完了するまで待機するように変更しました [#11166](https://github.com/pingcap/tidb/pull/11166)
+ 誤った結果を回避するために、ウィンドウ関数をプリペアプランキャッシュにキャッシュしないようにしました [#11048](https://github.com/pingcap/tidb/pull/11048)
+ 保存された生成列の定義を変更する`ALTER`ステートメントを禁止しました [#11068](https://github.com/pingcap/tidb/pull/11068)
+ 仮想生成列を保存された生成列に変更することを禁止しました [#11068](https://github.com/pingcap/tidb/pull/11068)
+ インデックスで生成された列の式を変更することを禁止しました [#11068](https://github.com/pingcap/tidb/pull/11068)
+ ARM64アーキテクチャでTiDBをコンパイルするサポートを追加しました [#11150](https://github.com/pingcap/tidb/pull/11150)
+ データベースまたはテーブルの照合順序を変更するサポートを追加しましたが、データベース/テーブルの文字セットはUTF-8またはutf8mb4でなければなりません [#11086](https://github.com/pingcap/tidb/pull/11086)
+ `SELECT`サブクエリが`UPDATE … SELECT`ステートメントで`UPDATE`式内の列を解析できず、列が誤って削除されたときにエラーが報告される問題を修正しました [#11252](https://github.com/pingcap/tidb/pull/11252)
+ 複数回NULLが返されたときにパニックが発生する問題を修正しました [#11226](https://github.com/pingcap/tidb/pull/11226)
+ `RAND`関数を使用する際の非スレッドセーフな`rand.Rand`によって発生するデータ競合の問題を修正しました [#11169](https://github.com/pingcap/tidb/pull/11169)
+ `oom-action="cancel"`が構成されている場合に、ステートメントの実行がキャンセルされないまましきい値を超えたSQLステートメントのメモリ使用量がしきい値を超える場合に、実行がキャンセルされず、返された結果が不正確になる問題を修正しました [#11004](https://github.com/pingcap/tidb/pull/11004)
+ MemTrackerのメモリ使用量が正しくクリアされなかったために、`SHOW PROCESSLIST`がメモリ使用量を`0`と表示しなかった問題を修正しました [#10970](https://github.com/pingcap/tidb/pull/10970)
+ 整数と非整数を比較した結果が一部で正しくない問題を修正しました [#11194](https://github.com/pingcap/tidb/pull/11194)
+ テーブルパーティション上のクエリに明示的トランザクションの述語を含む場合に、クエリ結果が正しくない問題を修正しました [#11196](https://github.com/pingcap/tidb/pull/11196)
+ `infoHandle`が`NULL`である可能性があるために発生するDDLジョブのパニック問題を修正しました [#11022](https://github.com/pingcap/tidb/pull/11022)
+ ネストされた集計クエリを実行する際に、サブクエリでクエリされた列が参照されず、誤って削除された結果が返される問題を修正しました [#11020](https://github.com/pingcap/tidb/pull/11020)
+ `Sleep`関数が`KILL`ステートメントに適時に応答しない問題を修正しました [#11028](https://github.com/pingcap/tidb/pull/11028)
+ `SHOW PROCESSLIST`コマンドで表示される`DB`および`INFO`列がMySQLと互換性がないために`0`のメモリ使用量を示す問題を修正しました [#11003](https://github.com/pingcap/tidb/pull/11003)
+ `skip-grant-table=true`が構成されているときに`FLUSH PRIVILEGES`ステートメントによってシステムパニックが発生する問題を修正しました [#11027](https://github.com/pingcap/tidb/pull/11027)
+ `UNSIGNED`整数でテーブルプライマリーキーがあり、`FAST ANALYZE`によって収集されたプライマリーキー統計が正しくない問題を修正しました [#11099](https://github.com/pingcap/tidb/pull/11099)
+ 一部の場合に`FAST ANALYZE`ステートメントによって`無効なキー`エラーが報告される問題を修正しました [#11098](https://github.com/pingcap/tidb/pull/11098)
+`CURRENT_TIMESTAMP`を列のデフォルト値として使用し、浮動小数点精度が指定された場合に、`SHOW CREATE TABLE`ステートメントに表示される精度が不完全である問題を修正しました [#11088](https://github.com/pingcap/tidb/pull/11088)
+ ウィンドウ関数がエラーを報告したときに関数名が小文字でない問題を修正しました。これによりMySQLとの互換性が向上しました [#11118](https://github.com/pingcap/tidb/pull/11118)
+ バックグラウンドスレッドのTiKV Client Batch gRPCがパニックを起こした後、TiKVに接続できず、サービスを提供できないという問題を修正しました [#11101](https://github.com/pingcap/tidb/pull/11101)
+ 文字列の浅いコピーによって`SetVar`によって変数が誤って設定される問題を修正しました [#11044](https://github.com/pingcap/tidb/pull/11044)
+ テーブルパーティションで`INSERT … ON DUPLICATE`ステートメントを適用する際に、実行が失敗してエラーが報告される問題を修正しました [#11231](https://github.com/pingcap/tidb/pull/11231)

+ 悲観的ロック（実験的な機能）
    - 悲観的ロックを使用してポイントクエリを実行し、返されたデータが空の場合に無効なロックによって誤った結果が返される問題を修正しました [#10976](https://github.com/pingcap/tidb/pull/10976)
    - 悲観的ロックを使用する際の`SELECT … FOR UPDATE`がクエリ内の悲観的ロックに正しいTSOを使用していなかった場合にクエリ結果が正しくない問題を修正しました [#11015](https://github.com/pingcap/tidb/pull/11015)
    - 楽観的トランザクションが悲観的ロックと出会ったときに即時の競合検出から待機への検出動作を変更し、ロックの競合を悪化させないようにしました [#11051](https://github.com/pingcap/tidb/pull/11051)

## TiKV

- blobファイルのサイズの統計情報を統計情報に追加しました[#5060](https://github.com/tikv/tikv/pull/5060)
- プロセスが終了する際にメモリリソースが誤ってクリアされたためにコアダンプが発生する問題を修正しました [#5053](https://github.com/tikv/tikv/pull/5053)
- Titanエンジンに関連する全監視メトリクスを追加しました [#4772](https://github.com/tikv/tikv/pull/4772), [#4836](https://github.com/tikv/tikv/pull/4836)
- Titanのオープンファイルハンドルの数をカウントする際に、正確なファイルハンドルの統計がなされないためにファイルハンドルが利用できなくなる問題を避けるために、特定のCFでTitanエンジンを有効にするかどうかを決定する`blob_run_mode`を設定しました [#5026](https://github.com/tikv/tikv/pull/5026)
- 読み取り操作が悲観的トランザクションのコミット情報を取得できなくなる問題を修正しました [#5067](https://github.com/tikv/tikv/pull/5067)
- Titanエンジンの実行モードを制御する`blob-run-mode`構成パラメータを追加しました。その値は`normal`、`read-only`、または`fallback`であることができます [#4865](https://github.com/tikv/tikv/pull/4865)
- デッドロックの検出のパフォーマンスを改善しました [#5089](https://github.com/tikv/tikv/pull/5089)

## PD

- PDがホットリージョンをスケジュールする際に自動的にスケジューリング制限が0に調整される問題を修正しました [#1552](https://github.com/pingcap/pd/pull/1552)
- etcdの`enable-grpc-gateway`設定オプションを追加して、gRPCゲートウェイ機能を有効にします[#1596](https://github.com/pingcap/pd/pull/1596)
- `store-balance-rate`、`hot-region-schedule-limit`などのスケジューラ構成に関連する統計情報を追加します[#1601](https://github.com/pingcap/pd/pull/1601)
- ホットリージョンのスケジューリング戦略を最適化し、スケジューリング中にレプリカが不足しているリージョンをスキップして、同じIDCに複数のレプリカがスケジュールされるのを防ぎます[#1609](https://github.com/pingcap/pd/pull/1609)
- リージョンのマージ処理ロジックを最適化し、サイズが小さいリージョンのマージを優先してスピードアップするようにサポートします[#1613](https://github.com/pingcap/pd/pull/1613)
- 一度にホットリージョンのスケジューリングのデフォルト制限を64に調整して、システムリソースを占有し、パフォーマンスに影響を与える過剰なスケジューリングタスクを防ぎます[#1616](https://github.com/pingcap/pd/pull/1616)
- リージョンのスケジューリング戦略を最適化し、`Pending`ステータスのリージョンのスケジューリングを高い優先度でサポートします[#1617](https://github.com/pingcap/pd/pull/1617)
- `random-merge`および`admin-merge-region`オペレーターを追加できない問題を修正します[#1634](https://github.com/pingcap/pd/pull/1634)
- ログのリージョンキーのフォーマットを16進数表記に調整して、表示しやすくします[#1639](https://github.com/pingcap/pd/pull/1639)

## ツール

TiDB Binlog

- Pump GC戦略を最適化し、未消費のbinlogが長時間占有されないように、未消費のbinlogをクリーンアップできる制限を削除します[#646](https://github.com/pingcap/tidb-binlog/pull/646)

TiDB Lightning

- SQLダンプで指定された列名が小文字でない場合に発生するインポートエラーを修正します[#210](https://github.com/pingcap/tidb-lightning/pull/210)

## TiDB Ansible

- ansibleコマンドとその`jmespath`、`jinja2`依存パッケージに対する事前チェック機能を追加します[#803](https://github.com/pingcap/tidb-ansible/pull/803), [#813](https://github.com/pingcap/tidb-ansible/pull/813)
- Pumpにおいて、利用可能なディスク容量がパラメータ値未満になったときにPumpでbinlogファイルの書き込みを停止する`stop-write-at-available-space`パラメータ(デフォルトで10 GiB)を追加します[#806](https://github.com/pingcap/tidb-ansible/pull/806)
- TiKVモニタリング情報のI/O監視項目を更新し、新バージョンの監視コンポーネントと互換性があるようにします[#820](https://github.com/pingcap/tidb-ansible/pull/820)
- PDモニタリング情報を更新し、ディスクレイテンシーのディスクパフォーマンスダッシュボードで空白となる異常を修正します[#817](https://github.com/pingcap/tidb-ansible/pull/817)
- TiKVの詳細ダッシュボードにTitanの監視項目を追加します[#824](https://github.com/pingcap/tidb-ansible/pull/824)