---
title: TiDB 5.4.1 リリースノート
---

# TiDB 5.4.1 リリースノート

リリース日: 2022年5月13日

TiDB バージョン: 5.4.1

## 互換性の変更

TiDB v5.4.1 では、製品設計に互換性の変更はありません。ただし、このリリースでのバグ修正によって、互換性の変更が生じる可能性があります。詳細については、[Bug Fixes](#bug-fixes) を参照してください。

## 改善点

+ TiDB

    - `_tidb_rowid` カラムを読むクエリに対して、PointGet プランを使用するサポート [#31543](https://github.com/pingcap/tidb/issues/31543)
    - `Apply` 演算子のためにより多くのログとメトリクスを追加し、それが並列であるかどうかを示すサポート [#33887](https://github.com/pingcap/tidb/issues/33887)
    - 統計情報を収集するために使用される Anaylze Version 2 における `TopN` プルーニングロジックを改善 [#34256](https://github.com/pingcap/tidb/issues/34256)
    - Grafana ダッシュボードで複数の Kubernetes クラスタを表示するサポート [#32593](https://github.com/pingcap/tidb/issues/32593)

+ TiKV

    - Grafana ダッシュボードで複数の Kubernetes クラスタを表示するサポート [#12104](https://github.com/tikv/tikv/issues/12104)

+ PD

    - Grafana ダッシュボードで複数の Kubernetes クラスタを表示するサポート [#4673](https://github.com/tikv/pd/issues/4673)

+ TiFlash

    - Grafana ダッシュボードで複数の Kubernetes クラスタを表示するサポート [#4129](https://github.com/pingcap/tiflash/issues/4129)

+ Tools

    + TiCDC

        - Grafana ダッシュボードで複数の Kubernetes クラスタをサポートする [#4665](https://github.com/pingcap/tiflow/issues/4665)
        - TiCDC での Kafka プロデューサの設定パラメータを公開して、TiCDC で構成可能にするサポート [#4385](https://github.com/pingcap/tiflow/issues/4385)

    + TiDB データ移行（DM）

        - DM-worker の作業ディレクトリを `/tmp` ではなく使用して内部ファイルを書き込み、タスクが停止した後にディレクトリをクリーニングするサポート [#4107](https://github.com/pingcap/tiflow/issues/4107)

## バグ修正

+ TiDB

    - TiDB において、`date_format` が MySQL と互換性のない方法で `'\n'` を処理する問題を修正 [#32232](https://github.com/pingcap/tidb/issues/32232)
    - TiDB が誤ったエンコーディングにより誤ったデータを書き込む問題を修正 [#32302](https://github.com/pingcap/tidb/issues/32302)
    - 特定のケースで Merge Join 演算子が誤った結果を返す問題を修正 [#33042](https://github.com/pingcap/tidb/issues/33042)
    - 相関サブクエリが定数を返す場合、TiDB が誤った結果を返す問題を修正 [#32089](https://github.com/pingcap/tidb/issues/32089)
    - TiFlash がまだ空の範囲のテーブルを読み取るサポートをしていないにもかかわらず、TiFlash を使用して空の範囲のテーブルをスキャンする際に TiDB が誤った結果を取得する問題を修正 [#33083](https://github.com/pingcap/tidb/issues/33083)
    - TiDB で新しい照合順序が有効になっている場合、`ENUM` や `SET` カラムの `MAX` または `MIN` 関数が誤った結果を返す問題を修正 [#31638](https://github.com/pingcap/tidb/issues/31638)
    - クエリにエラーが報告されると CTE がブロックされる可能性があるバグを修正 [#31302](https://github.com/pingcap/tidb/issues/31302)
    - Enum 値に対する Nulleq 関数の不正な範囲計算結果を修正 [#32428](https://github.com/pingcap/tidb/issues/32428)
    - ChunkRPC を使用してデータをエクスポートする際の TiDB OOM を修正 [#31981](https://github.com/pingcap/tidb/issues/31981) [#30880](https://github.com/pingcap/tidb/issues/30880)
    - `tidb_restricted_read_only` が有効になっている場合に `tidb_super_read_only` が自動的に有効にならないバグを修正 [#31745](https://github.com/pingcap/tidb/issues/31745)
    - 照合順序を使用した `greatest` または `least` 関数が誤った結果を返す問題を修正 [#31789](https://github.com/pingcap/tidb/issues/31789)
    - データがエスケープ文字で壊れている場合に load data パニックが発生する問題を修正 [#31589](https://github.com/pingcap/tidb/issues/31589)
    - インデックスルックアップ結合を使用してクエリを実行する際に `invalid transaction` エラーが発生する問題を修正 [#30468](https://github.com/pingcap/tidb/issues/30468)
    - `left join` を使用して複数のテーブルのデータを削除する場合の誤った結果を修正 [#31321](https://github.com/pingcap/tidb/issues/31321)
    - TiDB が TiFlash に重複したタスクをディスパッチするバグを修正 [#32814](https://github.com/pingcap/tidb/issues/32814)
    - v4.0 からアップグレードされたクラスタで `all` 権限を付与する際に失敗する問題を修正 [#33588](https://github.com/pingcap/tidb/issues/33588)
    - MySQL バイナリプロトコルでのテーブルスキーマ変更後にプリペアドステートメントを実行する際にセッションパニックが発生する問題を修正 [#33509](https://github.com/pingcap/tidb/issues/33509)
    - `tidb_enable_vectorized_expression` が有効になっている状態で `compress()` 式を含む SQL ステートメントを実行した際に失敗する問題を修正 [#33397](https://github.com/pingcap/tidb/issues/33397)
    - `reArrangeFallback` 関数による高い CPU 使用率の問題を修正 [#30353](https://github.com/pingcap/tidb/issues/30353)
    - 新しいパーティションが追加されるとテーブル属性がインデックスされず、パーティションが変更されるとテーブル範囲情報が更新されない問題を修正 [#33929](https://github.com/pingcap/tidb/issues/33929)
    - 初期化中のテーブルの `TopN` 統計情報が正しくソートされないバグを修正 [#34216](https://github.com/pingcap/tidb/issues/34216)
    - `INFORMATION_SCHEMA.ATTRIBUTES` テーブルから読み取る際に、特定できないテーブル属性をスキップして `invalid transaction` エラーが発生する問題を修正 [#33665](https://github.com/pingcap/tidb/issues/33665)
    - `order` プロパティが存在する場合にも `@@tidb_enable_parallel_apply` が設定されている場合に `Apply` 演算子が並列化されないバグを修正 [#34237](https://github.com/pingcap/tidb/issues/34237)
    - `sql_mode` が `NO_ZERO_DATE` に設定されている場合に `datetime` カラムに `'0000-00-00 00:00:00'` が挿入されるバグを修正 [#34099](https://github.com/pingcap/tidb/issues/34099)
    - `INFORMATION_SCHEMA.CLUSTER_SLOW_QUERY` テーブルを問い合わせる際に TiDB サーバーがメモリ不足になる問題を修正。この問題は Grafana ダッシュボードで遅いクエリをチェックする際に発生する可能性があります [#33893](https://github.com/pingcap/tidb/issues/33893)
    - ロックに遭遇した際にトランザクションがすぐに戻らない場合に `NOWAIT` ステートメントで失敗するバグを修正 [#32754](https://github.com/pingcap/tidb/issues/32754)
    - `GBK` 文字セットおよび `gbk_bin` 照合規則を使用してテーブルを作成する際に失敗するバグを修正 [#31308](https://github.com/pingcap/tidb/issues/31308)
    - `enable-new-charset` が `on` に設定されている場合に、`GBK` 文字セットのテーブルを照合規則とともに作成すると「Unknown character set」エラーが発生するバグを修正 [#31297](https://github.com/pingcap/tidb/issues/31297)

+ TiKV

    - 無効なターゲット Region のマージにより TiKV がパニックを起こし、ピアが予期せず破壊される問題を修正 [#12232](https://github.com/tikv/tikv/issues/12232)
    - 古いメッセージが TiKV がパニックを起こす原因となるバグを修正 [#12023](https://github.com/tikv/tikv/issues/12023)
    - メモリメトリクスのオーバーフローにより断続的なパケットロストとメモリ不足（OOM）の問題を修正 [#12160](https://github.com/tikv/tikv/issues/12160)
    - Ubuntu 18.04 での TiKV プロファイリング実行時に発生する潜在的なパニック問題を修正 [#9765](https://github.com/tikv/tikv/issues/9765)
    - レプリカリードが線形性を侵害する可能性があるバグを修正 [#12109](https://github.com/tikv/tikv/issues/12109)
    - マージングされる Region に初期化されずに破壊されたピアでターゲットピアが置換された際に TiKV がパニックする問題を修正 [#12048](https://github.com/tikv/tikv/issues/12048)
    - TiKV
        - TiKVが2年以上実行された後、パニックする可能性があるバグを修正する[#11940](https://github.com/tikv/tikv/issues/11940)
        - `Resolve Locks`ステップで必要なリージョンの数を減らすことにより、TiCDCのリカバリ時間を短縮する[#11993](https://github.com/tikv/tikv/issues/11993)
        - ピアのステータスが`Applying`のときにスナップショットファイルを削除することで発生するパニックの問題を修正する[#11746](https://github.com/tikv/tikv/issues/11746)
        - ピアを破棄することで遅延が発生する可能性がある問題を修正する[#10210](https://github.com/tikv/tikv/issues/10210)
        - リソースメータリングの無効なアサーションによって発生するパニックの問題を修正する[#12234](https://github.com/tikv/tikv/issues/12234)
        - 特定のケースでスロースコア計算が不正確である問題を修正する[#12254](https://github.com/tikv/tikv/issues/12254)
        - `resolved_ts`モジュールによって発生するOOMの問題を修正し、メトリクスを追加する[#12159](https://github.com/tikv/tikv/issues/12159)
        - ネットワークが貧弱な場合、正常に確定された楽観的なトランザクションが`Write Conflict`エラーを報告する問題を修正する[#34066](https://github.com/pingcap/tidb/issues/34066)
        - ネットワークが悪い状態でレプリカ読み取りが有効になっている場合に発生するTiKVパニックの問題を修正する[#12046](https://github.com/tikv/tikv/issues/12046)

    - PD
        - `dr-autosync`の`Duration`フィールドを動的に構成できないバグを修正する[#4651](https://github.com/tikv/pd/issues/4651)
        - 大容量のストア（たとえば2T）が存在する場合、完全に割り当てられた小さなストアが検出されず、バランスオペレータが生成されない問題を修正する[#4805](https://github.com/tikv/pd/issues/4805)
        - ラベル分布がメトリクスに残留ラベルを持っている問題を修正する[#4825](https://github.com/tikv/pd/issues/4825)

    - TiFlash
        - TLSが有効になっている場合に発生するパニックの問題を修正する[#4196](https://github.com/pingcap/tiflash/issues/4196)
        - 遅れているリージョンのピアでリージョンがマージされることによって発生する可能性のあるメタデータの破損を修正する[#4437](https://github.com/pingcap/tiflash/issues/4437)
        - `JOIN`を含むクエリがエラーが発生した場合にフリーズする可能性がある問題を修正する[#4195](https://github.com/pingcap/tiflash/issues/4195)
        - MPPタスクが永久にスレッドをリークする可能性があるバグを修正する[#4238](https://github.com/pingcap/tiflash/issues/4238)
        - `FLOAT`を`DECIMAL`にキャストする際に発生するオーバーフローの問題を修正する[#3998](https://github.com/pingcap/tiflash/issues/3998)
        - 期限切れのデータが遅くリサイクルされる問題を修正する[#4146](https://github.com/pingcap/tiflash/issues/4146)
        - ローカルトンネルが有効になっている場合、キャンセルされたMPPクエリがタスクが永久にハングする可能性のあるバグを修正する[#4229](https://github.com/pingcap/tiflash/issues/4229)
        - クエリがキャンセルされた場合にメモリリークが発生する問題を修正する[#4098](https://github.com/pingcap/tiflash/issues/4098)
        - `DATETIME`を`DECIMAL`にキャストする際に発生する誤った結果の問題を修正する[#4151](https://github.com/pingcap/tiflash/issues/4151)
        - 複数のDDL操作と同時に`Snapshot`が適用された場合にTiFlashがパニックする可能性がある問題を修正する[#4072](https://github.com/pingcap/tiflash/issues/4072)
        - 無効なストレージディレクトリ構成が予期しない動作を引き起こすバグを修正する[#4093](https://github.com/pingcap/tiflash/issues/4093)
        - 一部の例外が適切に処理されていない問題を修正する[#4101](https://github.com/pingcap/tiflash/issues/4101)
        - `INT`を`DECIMAL`にキャストすることでオーバーフローを引き起こす可能性がある問題を修正する[#3920](https://github.com/pingcap/tiflash/issues/3920)
        - 多値式で`IN`の結果が正しくない問題を修正する[#4016](https://github.com/pingcap/tiflash/issues/4016)
        - 日付形式が改行文字を無効な区切り文字として認識する問題を修正する[#4036](https://github.com/pingcap/tiflash/issues/4036)
        - 多くの削除操作が行われたテーブルで列が追加された後にクエリエラーが発生する可能性のある問題を修正する[#3967](https://github.com/pingcap/tiflash/issues/3967)
        - メモリ制限が有効になっている場合に発生するTiFlashのパニックの問題を修正する[#3902](https://github.com/pingcap/tiflash/issues/3902)
        - DTFilesでデータの破損の可能性がある問題を修正する[#4778](https://github.com/pingcap/tiflash/issues/4778)
        - 多くの削除操作が行われたテーブルでクエリする際に潜在的なエラーが発生する可能性のある問題を修正する[#4747](https://github.com/pingcap/tiflash/issues/4747)
        - TiFlashがランダムに多くの"Keepalive watchdog fired"エラーを報告するバグを修正する[#4192](https://github.com/pingcap/tiflash/issues/4192)
        - いくつかの例外の適切な処理がされていない問題を修正する[#4414](https://github.com/pingcap/tiflash/issues/4414)
        - GC後に空のセグメントをマージできないバグを修正する[#4511](https://github.com/pingcap/tiflash/issues/4511)

    - Tools
        - Backup & Restore (BR)
            - バックアップの再試行中に暗号化情報が失われるとリストア操作が失敗するバグを修正する[#32423](https://github.com/pingcap/tidb/issues/32423)
            - BRがRawKVのバックアップに失敗する問題を修正する[#32607](https://github.com/pingcap/tidb/issues/32607)
            - 増分リストア後にテーブルにレコードを挿入する際に重複した主キーが発生するバグを修正する[#33596](https://github.com/pingcap/tidb/issues/33596)
            - 空のクエリによるDDLジョブによって誤ってBR増分リストアがエラーを返す問題を修正する[#33322](https://github.com/pingcap/tidb/issues/33322)
            - リストア操作が終了した後にリージョンが均等に分布されない可能性がある問題を修正する[#31034](https://github.com/pingcap/tidb/issues/31034)
            - リストア中にリージョンが一貫性がない場合、BRが十分にリトライしない問題を修正する[#33419](https://github.com/pingcap/tidb/issues/33419)
            - 小さなファイルの結合が有効になっている場合、BRまたはTiDB Lightningが異常終了した後にスケジューラが再開しない問題を修正する[#33801](https://github.com/pingcap/tidb/issues/33801)
            - BRが異常終了した後にスケジューラが再開しない問題を修正する[#33546](https://github.com/pingcap/tidb/issues/33546)

        - TiCDC
            - オーナーが変更されたことによる不正確なメトリクスを修正する[#4774](https://github.com/pingcap/tiflow/issues/4774)
            - `Canal-JSON`が`nil`をサポートしていないために発生するTiCDCのパニックの問題を修正する[#4736](https://github.com/pingcap/tiflow/issues/4736)
            - Unified Sorterで使用されるworkerpoolの安定性の問題を修正する[#4447](https://github.com/pingcap/tiflow/issues/4447)
            - 特定のケースでシーケンスが誤ってレプリケーションされる問題を修正する[#4563](https://github.com/pingcap/tiflow/issues/4552)
            - `Canal-JSON`が`string`を誤って処理することで発生するTiCDCのパニックの問題を修正する[#4635](https://github.com/pingcap/tiflow/issues/4635)
            - PDリーダーが停止された場合にTiCDCノードが異常終了するバグを修正する[#4248](https://github.com/pingcap/tiflow/issues/4248)
            - `batch-replace-enable`が無効になっている場合にMySQLシンクが重複した`replace` SQLステートメントを生成する問題を修正する[#4501](https://github.com/pingcap/tiflow/issues/4501)
            - `rename tables`DDLによって構築エラーを修正する[#5059](https://github.com/pingcap/tiflow/issues/5059)
            - オーナーが変更された場合や新しいスケジューラが有効化された場合にレプリケーションが詰まる可能性がある問題を修正する[#4963](https://github.com/pingcap/tiflow/issues/4963)
            - 新しいスケジューラが有効になった場合にErrProcessorDuplicateOperationsエラーが報告される問題を修正する[#4769](https://github.com/pingcap/tiflow/issues/4769)
            - TLSが有効になった後に`--pd`で最初のPDセットが利用できない場合にTiCDCが起動しない問題を修正する[#4777](https://github.com/pingcap/tiflow/issues/4777)
            - テーブルがスケジュールされている間にチェックポイントのメトリクスが欠落している問題を修正する[#4714](https://github.com/pingcap/tiflow/issues/4714)

    - TiDB Lightning
- チェックサムエラー "GC 寿命がトランザクションの期間よりも短い" を修正します [#32733](https://github.com/pingcap/tidb/issues/32733)
        - TiDB Lightning が空のテーブルをチェックできないときにスタックする問題を修正します [#31797](https://github.com/pingcap/tidb/issues/31797)
        - 一部のインポートタスクにソースファイルが含まれていない場合に、TiDB Lightning がメタデータスキーマを削除しないバグを修正します [#28144](https://github.com/pingcap/tidb/issues/28144)
        - プリチェックがローカルディスクリソースとクラスタの可用性をチェックしない問題を修正します [#34213](https://github.com/pingcap/tidb/issues/34213)

+ TiDB データ移行（DM）

    - ログに何百もの "checkpoint has no change, skip sync flush checkpoint" が表示され、レプリケーションが非常に遅い問題を修正します [#4619](https://github.com/pingcap/tiflow/issues/4619)
    - 長い varchar が `Column length too big` エラーを報告するバグを修正します [#4637](https://github.com/pingcap/tiflow/issues/4637)
    - 安全モードでの更新文の実行エラーが DM-worker のパニックを引き起こす場合がある問題を修正します [#4317](https://github.com/pingcap/tiflow/issues/4317)
    - 下流でのフィルターされた DDL の手動実行がタスクの再開失敗を引き起こす可能性がある問題を修正します [#5272](https://github.com/pingcap/tiflow/issues/5272)
    - 上流で binlog が有効になっていない場合に `query-status` コマンドでデータが返されないバグを修正します [#5121](https://github.com/pingcap/tiflow/issues/5121)
    - `SHOW CREATE TABLE` ステートメントによって返されたインデックスで主キーが最初でない場合に、DM worker がパニックする問題を修正します [#5159](https://github.com/pingcap/tiflow/issues/5159)
    - GTID が有効になっている場合やタスクが自動的に再開される場合に CPU 使用率が上昇し、多くのログが出力される問題を修正します [#5063](https://github.com/pingcap/tiflow/issues/5063)