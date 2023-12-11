---
title： TiDB 4.0 RC.1 リリースノート
aliases: ['/docs/dev/releases/release-4.0.0-rc.1/', '/docs/dev/releases/4.0.0-rc.1/']
---


# TiDB 4.0 RC.1 リリースノート

リリース日: 2020年4月28日

TiDB バージョン: 4.0.0-rc.1

## 互換性の変更

+ TiKV

   - ヒベルネートリージョン機能をデフォルトで無効にする[#7618](https://github.com/tikv/tikv/pull/7618)

+ TiDB Binlog

   - DrainerでのシーケンスDDL操作のサポート[#950](https://github.com/pingcap/tidb-binlog/pull/950)

## 重要なバグ修正

+ TiDB

   - `MemBuffer` がチェックされていないため、明示的なトランザクション内で `INSERT ... ON DUPLICATE UPDATE` 文が複数行で誤って実行される問題を修正しました[#16689](https://github.com/pingcap/tidb/pull/16689)
   - 複数の行で重複するキーをロックする際のデータ不整合を修正しました[#16769](https://github.com/pingcap/tidb/pull/16769)
   - TiDB インスタンス間の非スーパーバッチアイドル接続のリサイクル時に発生するパニックを修正しました[#16303](https://github.com/pingcap/tidb/pull/16303)

+ TiKV

    - TiDB からのプローブリクエストによって発生するデッドロックの問題を修正しました[#7540](https://github.com/tikv/tikv/pull/7540)
    - トランザクションの最小コミットタイムスタンプがオーバーフローする可能性があり、データの正確性に影響を与える問題を修正しました[#7638](https://github.com/tikv/tikv/pull/7638)

+ TiFlash

    - 複数のデータパスが構成されている場合に `rename table` 操作によって発生するデータ損失の問題を修正
    - 複数のデータパスが設定された場合にマージリージョンからデータを読み取る際にエラーが発生する問題を修正
    - 異常な状態のリージョンからデータを読み取る際にエラーが発生する問題を修正
    - `recover table`/`flashback table` を正しくサポートするために TiFlash でのテーブル名のマッピングを変更
    - テーブルの名称を変更する際に発生する潜在的なデータ損失の問題を修正するためにストレージパスを変更
    - スーパーバッチが有効な場合に TiDB が発生する潜在的なパニックを修正
    - オンライン更新シナリオでの読み取りモードを変更して読み取り性能を改善

+ TiCDC

    - TiCDC で管理されているスキーマが読み取りと書き込みのタイミングの問題を正しく処理できないことによりレプリケーションが失敗する問題を修正[#438](https://github.com/pingcap/tiflow/pull/438) [#450](https://github.com/pingcap/tiflow/pull/450) [#478](https://github.com/pingcap/tiflow/pull/478) [#496](https://github.com/pingcap/tiflow/pull/496)
    - 一部の TiKV の異常が発生した場合に、TiKV クライアントが内部リソースを正しく管理できない問題を修正[#499](https://github.com/pingcap/tiflow/pull/499) [#492](https://github.com/pingcap/tiflow/pull/492)
    - メタデータが正しくクリーンアップされずに TiCDC ノードに異常なまま残る問題を修正[#488](https://github.com/pingcap/tiflow/pull/488) [#504](https://github.com/pingcap/tiflow/pull/504)
    - Prewrite イベントの繰り返し送信を正しく処理できない TiKV クライアントの問題を修正[#446](https://github.com/pingcap/tiflow/pull/446)
    - 初期化前に受信した冗長な Prewrite イベントを正しく処理できない TiKV クライアントの問題を修正[#448](https://github.com/pingcap/tiflow/pull/448)

+ Backup & Restore (BR)

    - チェックサムが無効になっている場合にチェックサムがまだ実行される問題を修正[#223](https://github.com/pingcap/br/pull/223)
    - `auto-random` または `alter-pk` が TiDB で有効になっている場合に増分レプリケーションが失敗する問題を修正[#230](https://github.com/pingcap/br/pull/230) [#231](https://github.com/pingcap/br/pull/231)

## 新機能

+ TiDB

    - TiFlash に対して Coprocessor リクエストをバッチで送信するサポートを追加[#16226](https://github.com/pingcap/tidb/pull/16226)
    - デフォルトで Coprocessor キャッシュ機能を有効にする[#16710](https://github.com/pingcap/tidb/pull/16710)
    - SQL ステートメントの特別なコメントの登録セクションのみをパースする機能を追加[#16157](https://github.com/pingcap/tidb/pull/16157)
    - PD と TiKV インスタンスの構成を表示するための `SHOW CONFIG` 構文のサポートを追加[#16475](https://github.com/pingcap/tidb/pull/16475)

+ TiKV

    - データを S3 にバックアップする際のサーバーサイド暗号化にユーザー所有の KMS キーを使用するサポートを追加[#7630](https://github.com/tikv/tikv/pull/7630)
    - 負荷ベースの `split region` 操作のサポートを追加[#7623](https://github.com/tikv/tikv/pull/7623)
    - 一般名を検証するサポートを追加[#7468](https://github.com/tikv/tikv/pull/7468)
    - 同じアドレスにバインドされた複数の TiKV インスタンスの開始を回避するためのファイルロックのチェックを追加[#7447](https://github.com/tikv/tikv/pull/7447)
    - 暗号化ストレージで AWS KMS をサポート[#7465](https://github.com/tikv/tikv/pull/7465)

+ PD

    - 他のコンポーネントが自身のコンポーネント構成を制御できるようにするために `config manager` を削除[#2349](https://github.com/pingcap/pd/pull/2349)

+ TiFlash

    - DeltaTree エンジンの読み取りおよび書き込みワークロードに関連するメトリクスレポートを追加
    - 単一の読み取りまたは書き込みリクエストのディスク I/O を減らすために `handle` および `version` カラムをキャッシュ
    - `fromUnixTime` および `dateFormat` 関数をプッシュダウンするサポートを追加
    - 最初のディスクに対するグローバルな状態を評価し、この評価をレポート
    - DeltaTree エンジンの読み取りおよび書き込みワークロードに関連する Grafana でのグラフィックを追加
    - `Chunk` コーデックでの10進数データのエンコードを最適化
    - 診断（SQL 診断）の gRPC API を実装し、`INFORMATION_SCHEMA.CLUSTER_INFO` などのシステムテーブルの問合せをサポート

+ TiCDC

    - Kafka sink モジュールでメッセージをバッチで送信するサポートを追加[#426](https://github.com/pingcap/tiflow/pull/426)
    - プロセッサでファイルのソートをサポートする機能を追加[#477](https://github.com/pingcap/tiflow/pull/477)
    - 自動 `resolve lock` をサポートする機能を追加[#459](https://github.com/pingcap/tiflow/pull/459)
    - TiCDC サービスGCセーフポイントの自動更新機能を追加[#487](https://github.com/pingcap/tiflow/pull/487)
    - データレプリケーションのためのタイムゾーン設定を追加[#498](https://github.com/pingcap/tiflow/pull/498)

+ Backup and Restore (BR)

    - ストレージ URL で S3/GCS を構成するためのサポートを追加[#246](https://github.com/pingcap/br/pull/246)

## バグ修正

+ TiDB

    - 列が符号なしで定義されているためシステムテーブルで負の数が正しく表示されない問題を修正[#16004](https://github.com/pingcap/tidb/pull/16004)
    - `use_index_merge` ヒントに無効なインデックス名が含まれている場合に警告を追加[#15960](https://github.com/pingcap/tidb/pull/15960)
    - 同じ一時ディレクトリを共有する複数の TiDB サーバーインスタンスを禁止するためのパニックを修正[#16026](https://github.com/pingcap/tidb/pull/16026)
    - プランキャッシュが有効な状態で `explain for connection` の実行中に発生するパニックを修正[#16285](https://github.com/pingcap/tidb/pull/16285)
    - `tidb_capture_plan_baselines` システム変数の結果が正しく表示されない問題を修正[#16048](https://github.com/pingcap/tidb/pull/16048)
    - `prepare` ステートメントの `group by` 句が誤って解析される問題を修正[#16377](https://github.com/pingcap/tidb/pull/16377)
    - `analyze primary key` ステートメントの実行中に発生するパニックを修正[#16081](https://github.com/pingcap/tidb/pull/16081)
    - `cluster_info` システムテーブルの TiFlash ストア情報が誤っていた問題を修正[#16024](https://github.com/pingcap/tidb/pull/16024)
    - インデックスマージプロセス中に発生する可能性のあるパニックを修正[#16360](https://github.com/pingcap/tidb/pull/16360)
    - インデックスマージリーダーが生成された列を読み取る際に誤った結果が発生する可能性のある問題を修正[#16359](https://github.com/pingcap/tidb/pull/16359)
- `show create table` ステートメントにおいて、デフォルトのシーケンス値の表示が正しくされない問題を修正しました [#16526](https://github.com/pingcap/tidb/pull/16526)
- シーケンスが主キーのデフォルト値として使用されるために `not-null` エラーが返される問題を修正しました [#16510](https://github.com/pingcap/tidb/pull/16510)
- TiKV が `StaleCommand` エラーを継続的に返すと、ブロックされた SQL 実行に対してエラーが報告されない問題を修正しました [#16530](https://github.com/pingcap/tidb/pull/16530)
- データベースを作成する際に `COLLATE` のみを指定するとエラーが報告される問題を修正し、`SHOW CREATE DATABASE` の結果に不足していた `COLLATE` 部分を追加しました [#16540](https://github.com/pingcap/tidb/pull/16540)
- プランキャッシュが有効な場合にパーティションのプルーニングに失敗する問題を修正しました [#16723](https://github.com/pingcap/tidb/pull/16723)
- `PointGet` がオーバーフローを処理する際に間違った結果を返すバグを修正しました [#16755](https://github.com/pingcap/tidb/pull/16755)
- `slow_query` システムテーブルを等しい時間の値でクエリする際に間違った結果が返される問題を修正しました [#16806](https://github.com/pingcap/tidb/pull/16806)

+ TiKV

    - OpenSSL のセキュリティ問題 (CVE-2020-1967) を対処しました [#7622](https://github.com/tikv/tikv/pull/7622)
    - 楽観的トランザクションに大量の書き込み競合が存在する場合に、`BatchRollback` によって書き込まれたロールバックレコードを保護しないようにし、パフォーマンスを改善しました [#7604](https://github.com/tikv/tikv/pull/7604)
    - 重いロック競合ワークロードにおいて、不要なトランザクションの再試行とパフォーマンスの低下が発生するのを避け、トランザクションの不要な起床を修正しました [#7551](https://github.com/tikv/tikv/pull/7551)
    - マルチタイムマージングにおいて、リージョンがスタックしてしまう問題を修正しました [#7518](https://github.com/tikv/tikv/pull/7518)
    - リーダーを削除してもラーナーが削除されない問題を修正しました [#7518](https://github.com/tikv/tikv/pull/7518)
    - Raft-rs においてフォロワーリードがパニックを引き起こす可能性がある問題を修正しました [#7408](https://github.com/tikv/tikv/pull/7408)
    - SQL 操作が `group by constant` エラーのために失敗する可能性があるバグを修正しました [#7383](https://github.com/tikv/tikv/pull/7383)
    - 対応する主キーのロックが悲観的ロックである場合、楽観的ロックが読み取りをブロックする可能性がある問題を修正しました [#7328](https://github.com/tikv/tikv/pull/7328)

+ PD

    - TLS 検証において一部の API が失敗する可能性がある問題を修正しました [#2363](https://github.com/pingcap/pd/pull/2363)
    - 構成 API が接頭辞を持つ構成項目を受け入れない可能性がある問題を修正しました [#2354](https://github.com/pingcap/pd/pull/2354)
    - スケジューラが見つからない場合に `500` エラーが返される問題を修正しました [#2328](https://github.com/pingcap/pd/pull/2328)
    - `scheduler config balance-hot-region-scheduler list` コマンドに対して `404` エラーが返される問題を修正しました [#2321](https://github.com/pingcap/pd/pull/2321)

+ TiFlash

    - ストレージエンジンの粗い粒度インデックス最適化を無効にしました
    - リージョンのロックを解決する際に一部のロックをスキップする必要がある場合に例外がスローされる問題を修正しました
    - Coprocessor 統計情報を収集する際にヌルポインタ例外 (NPE) がスローされる問題を修正しました
    - リージョンの分割/リージョンのマージのプロセスが正常に行われることを保証するために、リージョンメタのチェックを修正しました
    - Coprocessor レスポンスのサイズが見積もられていないために gRPC のメッセージサイズが上限を超える問題を修正しました
    - TiFlash における `AdminCmdType::Split` コマンドの処理を修正しました