---
title: TiDB 5.3.1 リリースノート
---

# TiDB 5.3.1 リリースノート

リリース日: 2022年3月3日

TiDB バージョン: 5.3.1

## 互換性の変更

- ツール

    + TiDB Lightning

        - `regionMaxKeyCount` のデフォルト値を 1_440_000 から 1_280_000 に変更し、データのインポート後に空のリージョンが多くなるのを避けるようにしました [#30018](https://github.com/pingcap/tidb/issues/30018)

## 改善点

- TiDB

    - ユーザーログインモードのマッピングロジックを最適化し、よりMySQL互換性のあるログを生成するようにしました [#30450](https://github.com/pingcap/tidb/issues/32648)

- TiKV

    - TiCDC のリカバリ時間を短縮するために、Resolve Locks ステップを必要なリージョンの数を減らしました [#11993](https://github.com/tikv/tikv/issues/11993)
    - Raft ログの GC プロセスを高速化するために、GC 実行時の書き込みバッチサイズを増やしました [#11404](https://github.com/tikv/tikv/issues/11404)
    - proc ファイルシステム（procfs）を v0.12.0 にアップデートしました [#11702](https://github.com/tikv/tikv/issues/11702)

- PD

    - `DR_STATE` ファイルのコンテンツ形式を最適化しました [#4341](https://github.com/tikv/pd/issues/4341)

- ツール

    - TiCDC

        - Kafka プロデューサーの構成パラメータを外部から設定可能にするために公開しました [#4385](https://github.com/pingcap/tiflow/issues/4385)
        - S3 をバックエンドストレージとして使用する場合に、TiCDC 起動時に事前クリーンアッププロセスを追加しました [#3878](https://github.com/pingcap/tiflow/issues/3878)
        - 証明書名が指定されていない場合も TiCDC クライアントが動作するようにしました [#3627](https://github.com/pingcap/tiflow/issues/3627)
        - 予期しないチェックポイントタイムスタンプの進行を避けるために、テーブルごとにシンクチェックポイントを管理するようにしました [#3545](https://github.com/pingcap/tiflow/issues/3545)
        - changefeed を再起動する際の指数バックオフメカニズムを追加しました [#3329](https://github.com/pingcap/tiflow/issues/3329)
        - Kafka Sink の `partition-num` のデフォルト値を 3 に変更し、TiCDC がメッセージを Kafka パーティション全体に均等に分散するようにしました [#3337](https://github.com/pingcap/tiflow/issues/3337)
        - "EventFeed retry rate limited" ログの数を減らしました [#4006](https://github.com/pingcap/tiflow/issues/4006)
        - `max-message-bytes` のデフォルト値を 10M に設定しました [#4041](https://github.com/pingcap/tiflow/issues/4041)
        - `no owner alert`、`mounter row`、`table sink total row`、`buffer sink total row` などの Prometheus および Grafana の監視メトリクスおよびアラートを追加しました [#4054](https://github.com/pingcap/tiflow/issues/4054) [#1606](https://github.com/pingcap/tiflow/issues/1606)
        - TiKV ストアがダウンした際の KV クライアントの復旧時間を短縮しました [#3191](https://github.com/pingcap/tiflow/issues/3191)

    - TiDB Lightning

        - ローカルディスクスペースのチェックに失敗した場合に、よりユーザーフレンドリーな出力メッセージを改善しました [#30395](https://github.com/pingcap/tidb/issues/30395)

## バグ修正

- TiDB

    - TiDB における `date_format` が MySQL非互換な方法で `'\n'` を処理する問題を修正しました [#32232](https://github.com/pingcap/tidb/issues/32232)
    - `alter column set default` が誤ってテーブルスキーマを更新する問題を修正しました [#31074](https://github.com/pingcap/tidb/issues/31074)
    - `tidb_restricted_read_only` が有効化されている場合に `tidb_super_read_only` が自動的に有効化されない問題を修正しました [#31745](https://github.com/pingcap/tidb/issues/31745)
    - コレーションを用いた `greatest` または `least` 関数が誤った結果を返す問題を修正しました [#31789](https://github.com/pingcap/tidb/issues/31789)
    - クエリを実行する際に MPP タスクリストが空のエラーが発生する問題を修正しました [#31636](https://github.com/pingcap/tidb/issues/31636)
    - 内部ワーカーがパニックを起こすことによるインデックスジョインの誤った結果を修正しました [#31494](https://github.com/pingcap/tidb/issues/31494)
    - カラムの型を `FLOAT` から `DOUBLE` に変更した後のクエリ結果が誤っていた問題を修正しました [#31372](https://github.com/pingcap/tidb/issues/31372)
    - インデックスルックアップ結合を使用したクエリの実行中に `invalid transaction` エラーが発生する問題を修正しました [#30468](https://github.com/pingcap/tidb/issues/30468)
    - `Order By` の最適化によるクエリ結果の誤りを修正しました [#30271](https://github.com/pingcap/tidb/issues/30271)
    - `MaxDays` および `MaxBackups` の設定が遅いログに反映されない問題を修正しました [#25716](https://github.com/pingcap/tidb/issues/25716)
    - `INSERT ... SELECT ... ON DUPLICATE KEY UPDATE` ステートメントを実行する際にパニックが発生する問題を修正しました [#28078](https://github.com/pingcap/tidb/issues/28078)

- TiKV

    - ピアステータスが `Applying` の場合にスナップショットファイルを削除することで発生するパニックの問題を修正しました [#11746](https://github.com/tikv/tikv/issues/11746)
    - フローコントロールが有効化され、`level0_slowdown_trigger` が明示的に設定されている場合の QPS 低下の問題を修正しました [#11424](https://github.com/tikv/tikv/issues/11424)
    - cgroup コントローラーがマウントされていない場合に発生するパニックの問題を修正しました [#11569](https://github.com/tikv/tikv/issues/11569)
    - TiKV の動作を停止した後に Resolved TS の待ち時間が増加する問題を修正しました [#11351](https://github.com/tikv/tikv/issues/11351)
    - GC ワーカーが多忙な場合にデータ範囲を削除できない（`unsafe_destroy_range` を実行できない）問題を修正しました [#11903](https://github.com/tikv/tikv/issues/11903)
    - ピアを破棄すると高待ち時間が発生する問題を修正しました [#10210](https://github.com/tikv/tikv/issues/10210)
    - 領域が空の場合に `any_value` 関数が誤った結果を返すバグを修正しました [#11735](https://github.com/tikv/tikv/issues/11735)
    - 未初期化のレプリカを削除することで古いレプリカが再作成される問題を修正しました [#10533](https://github.com/tikv/tikv/issues/10533)
    - `Prepare Merge` 後に新しい選挙が終了し、孤立したピアに通知されないままでメタデータが破損する問題を修正しました [#11526](https://github.com/tikv/tikv/issues/11526)
    - コルーチンが速すぎて発生する時折のデッドロック問題を修正しました [#11549](https://github.com/tikv/tikv/issues/11549)
    - ダウンしている TiKV ノードによって解決されたタイムスタンプが遅れる問題を修正しました [#11351](https://github.com/tikv/tikv/issues/11351)
    - Raft クライアントの実装においてバッチメッセージが大きすぎる問題を修正しました [#9714](https://github.com/tikv/tikv/issues/9714)
    - 極端な条件下で Region のマージ、ConfChange、および Snapshot が同時に発生するとパニックが発生する問題を修正しました [#11475](https://github.com/tikv/tikv/issues/11475)
    - TiKV が逆テーブルスキャンを行う際にメモリロックを検出できない問題を修正しました [#11440](https://github.com/tikv/tikv/issues/11440)
    - ディスク容量がいっぱいの状態で RocksDB がフラッシュまたは圧縮することが原因でパニックが発生する問題を修正しました [#11224](https://github.com/tikv/tikv/issues/11224)
    - tikv-ctl が正しい関連情報を返せない問題を修正しました [#11393](https://github.com/tikv/tikv/issues/11393)
    - TiKV メトリックにおける by-instance gRPC リクエストの平均待ち時間が正確でない問題を修正しました [#11299](https://github.com/tikv/tikv/issues/11299)

- PD

    - 特定の場合にスケジューリングプロセスが不要な JointConsensus ステップを持つ問題を修正しました [#4362](https://github.com/tikv/pd/issues/4362)
    - 投票者を直接降格する場合にスケジューリングが実行されない問題を修正しました [#4444](https://github.com/tikv/pd/issues/4444)
    - レプリカのレプリケーションモードの構成を更新する際に発生するデータ競合の問題を修正しました [#4325](https://github.com/tikv/pd/issues/4325)
- 特定のケースでリードロックが解除されないバグを修正 [#4354](https://github.com/tikv/pd/issues/4354)
    - ホットスポット統計から冷たいホットスポットデータを削除できない問題を修正 [#4390](https://github.com/tikv/pd/issues/4390)

- TiFlash

    - `cast(arg as decimal(x,y))` が入力引数 `arg` が `decimal(x,y)` の範囲を超えた場合に間違った結果を返す問題を修正
    - `max_memory_usage` と `max_memory_usage_for_all_queries` が有効な場合に TiFlash クラッシュ問題を修正
    - `cast(string as real)` が間違った結果を返す問題を修正
    - `cast(string as decimal)` が間違った結果を返す問題を修正
    - プライマリキーカラムをより大きな int データ型に変更した後に潜在的なデータ不整合を修正
    - ステートメント内の `select (arg0, arg1) in (x,y)` のような `in` が複数の引数を持っている場合に `in` が間違った結果を返すバグを修正
    - MPP クエリが停止した際に TiFlash がパニックする可能性がある問題を修正
    - 入力引数に先行ゼロが含まれる場合に `str_to_date` が間違った結果を返す問題を修正
    - フィルタが `where <string>` 形式の場合にクエリが間違った結果を返す問題を修正
    - 入力引数 `string` が `%Y-%m-%d\n%H:%i:%s` 形式の場合に `cast(string as datetime)` が間違った結果を返す問題を修正

- ツール

    - バックアップおよびリストア（BR）

        - リストア操作が完了した後に領域が不均等になる可能性がある問題を修正 [#31034](https://github.com/pingcap/tidb/issues/31034)

    - TiCDC

        - 長い varchar が `Column length too big` エラーを報告するバグを修正 [#4637](https://github.com/pingcap/tiflow/issues/4637)
        - PD リーダーが停止した際に TiCDC ノードが異常終了するバグを修正 [#4248](https://github.com/pingcap/tiflow/issues/4248)
        - セーフモードにおける更新ステートメントの実行エラーが DM-worker パニックの原因となる問題を修正 [#4317](https://github.com/pingcap/tiflow/issues/4317)
        - TiKV クライアントのキャッシュされた領域メトリクスが負の値になる可能性がある問題を修正 [#4300](https://github.com/pingcap/tiflow/issues/4300)
        - 必要なプロセッサ情報が存在しない場合に HTTP API がパニックするバグを修正 [#3840](https://github.com/pingcap/tiflow/issues/3840)
        - 一時停止した changefeed を削除する際にリドゥログが削除されないバグを修正 [#4740](https://github.com/pingcap/tiflow/issues/4740)
        - コンテナ環境における OOM を修正 [#1798](https://github.com/pingcap/tiflow/issues/1798)
        - ローダー上で `query-status` コマンドに対して誤った進捗が返される問題を修正 [#3252](https://github.com/pingcap/tiflow/issues/3252)
        - クラスタ内の異なるバージョンの TiCDC ノードが存在する場合に HTTP API が機能しない問題を修正 [#3483](https://github.com/pingcap/tiflow/issues/3483)
        - TiCDC Redo Log が構成された場合に TiCDC が異常終了する問題を修正 [#3523](https://github.com/pingcap/tiflow/issues/3523)
        - デフォルト値をレプリケートできない問題を修正 [#3793](https://github.com/pingcap/tiflow/issues/3793)
        - `batch-replace-enable` が無効になっている場合に MySQL sink が重複した `replace` SQL ステートメントを生成するバグを修正 [#4501](https://github.com/pingcap/tiflow/issues/4501)
        - ステータスをクエリングしていない限り、syncer メトリクスが更新されない問題を修正 [#4281](https://github.com/pingcap/tiflow/issues/4281)
        - `mq sink write row` に監視データが存在しない問題を修正 [#3431](https://github.com/pingcap/tiflow/issues/3431)
        - `min.insync.replicas` が `replication-factor` よりも小さい場合にレプリケーションが実行されない問題を修正 [#3994](https://github.com/pingcap/tiflow/issues/3994)
        - レプリケーションタスクが削除された際に発生する潜在的なパニック問題を修正 [#3128](https://github.com/pingcap/tiflow/issues/3128)
        - デッドロックが原因でレプリケーションタスクがスタックする可能性がある問題を修正 [#4055](https://github.com/pingcap/tiflow/issues/4055)
        - 手動で etcd 内のタスクステータスをクリーニングした際に TiCDC がパニックする問題を修正 [#2980](https://github.com/pingcap/tiflow/issues/2980)
        - DDL ステートメント内の特別なコメントがレプリケーションタスクを停止させる問題を修正 [#3755](https://github.com/pingcap/tiflow/issues/3755)
        - `config.Metadata.Timeout` の構成が正しくないことでサービスが開始できなくなる問題を修正 [#3352](https://github.com/pingcap/tiflow/issues/3352)
        - 一部の RHEL リリースでタイムゾーンの問題が原因でサービスを開始できない問題を修正 [#3584](https://github.com/pingcap/tiflow/issues/3584)
        - クラスタのアップグレード後に `stopped` changefeed が自動的に再開する問題を修正 [#3473](https://github.com/pingcap/tiflow/issues/3473)
        - ミドルウェアプロトコル（Canal および Maxwell）で `enable-old-value` 構成項目が自動的に `true` に設定されないバグを修正 [#3676](https://github.com/pingcap/tiflow/issues/3676)
        - Avro sink が JSON タイプの列の解析をサポートしていない問題を修正 [#3624](https://github.com/pingcap/tiflow/issues/3624)
        - changefeed チェックポイントラグにおける負の値のエラーを修正 [#3010](https://github.com/pingcap/tiflow/issues/3010)

    - TiDB データ移行（DM）

        - DM-master と DM-worker を特定の順序で再起動した後に DM-master でのリレーのステータスが間違っているバグを修正 [#3478](https://github.com/pingcap/tiflow/issues/3478)
        - 再起動後に DM-worker が起動しないバグを修正 [#3344](https://github.com/pingcap/tiflow/issues/3344)
        - PARTITION DDL の実行に長時間かかる場合に DM タスクが失敗するバグを修正 [#3854](https://github.com/pingcap/tiflow/issues/3854)
        - MySQL 8.0 が上流にある場合に DM が `invalid sequence` を報告するバグを修正 [#3847](https://github.com/pingcap/tiflow/issues/3847)
        - DM がより細かい再試行を行う際にデータ損失が発生する問題を修正 [#3487](https://github.com/pingcap/tiflow/issues/3487)
        - `CREATE VIEW` ステートメントがデータレプリケーションを中断する問題を修正 [#4173](https://github.com/pingcap/tiflow/issues/4173)
        - DDL ステートメントがスキップされた後にスキーマをリセットする必要がある問題を修正 [#4177](https://github.com/pingcap/tiflow/issues/4177)

    - TiDB Lightning

        - いくつかのインポートタスクにソースファイルが含まれていない場合に TiDB Lightning がメタデータスキーマを削除しないバグを修正 [#28144](https://github.com/pingcap/tidb/issues/28144)
        - ストレージ URL プレフィックスが "gs://xxx" ではなく "gcs://xxx" の場合に TiDB Lightning がエラーを返さないバグを修正 [#32742](https://github.com/pingcap/tidb/issues/32742)
        - `--log-file="-"` を設定しても TiDB Lightning が標準出力にログを出力しないバグを修正 [#29876](https://github.com/pingcap/tidb/issues/29876)
        - S3 ストレージパスが存在しない場合に TiDB Lightning がエラーを報告しないバグを修正 [#30709](https://github.com/pingcap/tidb/issues/30709)