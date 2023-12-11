---
title: TiDB 5.2.4リリースノート
category: リリース

# TiDB 5.2.4リリースノート

リリース日: 2022年4月26日

TiDBバージョン: 5.2.4

## 互換性に関する変更

+ TiDB

    - システム変数[`tidb_analyze_version`](/system-variables.md#tidb_analyze_version-new-in-v510)のデフォルト値を`2`から`1`に変更 [#31748](https://github.com/pingcap/tidb/issues/31748)

+ TiKV

    - 不要なRaftログをコンパクト化するための時間間隔(`"2s"`がデフォルト)を制御する[`raft-log-compact-sync-interval`](https://docs.pingcap.com/tidb/v5.2/tikv-configuration-file#raft-log-compact-sync-interval-new-in-v524)を追加 [#11404](https://github.com/tikv/tikv/issues/11404)
    - [`raft-log-gc-tick-interval`](/tikv-configuration-file.md#raft-log-gc-tick-interval)のデフォルト値を`"10s"`から`"3s"`に変更 [#11404](https://github.com/tikv/tikv/issues/11404)
    - [`storage.flow-control.enable`](/tikv-configuration-file.md#enable)が`true`に設定されている場合、[`storage.flow-control.hard-pending-compaction-bytes-limit`](/tikv-configuration-file.md#hard-pending-compaction-bytes-limit)の値が[`rocksdb.(defaultcf|writecf|lockcf).hard-pending-compaction-bytes-limit`](/tikv-configuration-file.md#hard-pending-compaction-bytes-limit-1)の値を上書きするように変更 [#11424](https://github.com/tikv/tikv/issues/11424)

+ ツール

    + TiDB Lightning

        - データインポート後の空のリージョンが多すぎる問題を回避するために、`regionMaxKeyCount`のデフォルト値を1_440_000から1_280_000に変更 [#30018](https://github.com/pingcap/tidb/issues/30018)

## 改善点

+ TiKV

    - ラテンシージッターを減らすために、リーダーシップをCDCオブザーバーに移管する [#12111](https://github.com/tikv/tikv/issues/12111)
    - Resolve Locksステップが必要とするリージョンの数を減らすことで、TiCDCのリカバリ時間を短縮する [#11993](https://github.com/tikv/tikv/issues/11993)
    - proc filesystem (procfs)をv0.12.0にアップデート [#11702](https://github.com/tikv/tikv/issues/11702)
    - Raftログに対するGCプロセスのスピードを向上させるため、GC実行時のラフトログへの書き込みバッチサイズを増やす [#11404](https://github.com/tikv/tikv/issues/11404)
    - SSTファイルの挿入速度を上げるために、検証プロセスを`Apply`スレッドプールから`Import`スレッドプールに移動することで実現 [#11239](https://github.com/tikv/tikv/issues/11239)

+ ツール

    + TiCDC

        - TiCDCがKafkaパーティション全体にメッセージを均等に分散するように、Kafka Sinkの`partition-num`のデフォルト値を3に変更 [#3337](https://github.com/pingcap/tiflow/issues/3337)
        - TiKVストアがダウンしている場合に、KVクライアントが回復する時間を短縮する [#3191](https://github.com/pingcap/tiflow/issues/3191)
        - Grafanaに`Lag analyze`パネルを追加する [#4891](https://github.com/pingcap/tiflow/issues/4891)
        - Kafkaプロデューサーの構成パラメータを公開し、TiCDCで構成可能にする [#4385](https://github.com/pingcap/tiflow/issues/4385)
        - チェンジフィードを再起動するための指数バックオフメカニズムを追加する [#3329](https://github.com/pingcap/tiflow/issues/3329)
        - "EventFeed retry rate limited"ログのカウントを減らす [#4006](https://github.com/pingcap/tiflow/issues/4006)
        - `max-message-bytes`のデフォルト値を10Mに設定する [#4041](https://github.com/pingcap/tiflow/issues/4041)
        - `no owner alert`、`mounter row`、`table sink total row`、`buffer sink total row`など、PrometheusとGrafanaの監視メトリクスとアラートを追加する [#4054](https://github.com/pingcap/tiflow/issues/4054) [#1606](https://github.com/pingcap/tiflow/issues/1606)
        - Grafanaダッシュボードで複数のKubernetesクラスタをサポートする [#4665](https://github.com/pingcap/tiflow/issues/4665)
        - `changefeed checkpoint`の監視メトリクスに追いつきETA（到着予定時刻）を追加する [#5232](https://github.com/pingcap/tiflow/issues/5232)

## バグ修正

+ TiDB

    - Enum値に対するNulleq関数の誤った範囲計算結果を修正する [#32428](https://github.com/pingcap/tidb/issues/32428)
    - INDEX HASH JOINが`send on closed channel`エラーを返す問題を修正する [#31129](https://github.com/pingcap/tidb/issues/31129)
    - 並列カラム型の変更によるスキーマとデータの不整合を修正する [#31048](https://github.com/pingcap/tidb/issues/31048)
    - 楽観的なトランザクションモードでの潜在的なデータインデックスの不一致の問題を修正する [#30410](https://github.com/pingcap/tidb/issues/30410)
    - JSON型のカラムが`CHAR`型のカラムと結合する際にSQL操作がキャンセルされる問題を修正する [#29401](https://github.com/pingcap/tidb/issues/29401)
    - トランザクションの使用有無によってウィンドウ関数が異なる結果を返す問題を修正する [#29947](https://github.com/pingcap/tidb/issues/29947)
    - SQLステートメントにナチュラルジョインが含まれると、`Column 'col_name' in field list is ambiguous`エラーが予期せず報告される問題を修正する [#25041](https://github.com/pingcap/tidb/issues/25041)
    - `DECIMAL`を`STRING`にキャストする際の長さ情報が誤っている問題を修正する [#29417](https://github.com/pingcap/tidb/issues/29417)
    - `tidb_enable_vectorized_expression`の値（`on`または`off`に設定）によって`GREATEST`関数が一貫した結果を返さない問題を修正する [#29434](https://github.com/pingcap/tidb/issues/29434)
    - `left join`で複数のテーブルのデータを削除する際の誤った結果を修正する問題を修正する [#31321](https://github.com/pingcap/tidb/issues/31321)
    - TiDBがTiFlashに重複したタスクをディスパッチする可能性のあるバグを修正する [#32814](https://github.com/pingcap/tidb/issues/32814)
    - クエリの実行時にMPPタスクリストが空のエラーを修正する [#31636](https://github.com/pingcap/tidb/issues/31636)
    - `INDEX JOIN`による誤ったクエリ結果を修正する問題を修正する [#31494](https://github.com/pingcap/tidb/issues/31494)
    - `INSERT ... SELECT ... ON DUPLICATE KEY UPDATE`ステートメントの実行時にパニックが発生する問題を修正する [#28078](https://github.com/pingcap/tidb/issues/28078)
    - `Order By`の最適化によって誤ったクエリ結果が発生する問題を修正する [#30271](https://github.com/pingcap/tidb/issues/30271)
    - `ENUM`型のカラムに対する`JOIN`の際に誤った結果が返る問題を修正する [#27831](https://github.com/pingcap/tidb/issues/27831)
    - `ENUM`データ型に`CASE WHEN`関数を使用した際にパニックが発生する問題を修正する [#29357](https://github.com/pingcap/tidb/issues/29357)
    - ベクトル化された式で`microsecond`関数の誤った結果を修正する問題を修正する [#29244](https://github.com/pingcap/tidb/issues/29244)
    - ウィンドウ関数がエラーを報告せずにTiDBをパニックさせる問題を修正する [#30326](https://github.com/pingcap/tidb/issues/30326)
    - `Merge Join`演算子が特定のケースで誤った結果を返す問題を修正する [#33042](https://github.com/pingcap/tidb/issues/33042)
    - 相関サブクエリが定数を返す際にTiDBが誤った結果を返す問題を修正する [#32089](https://github.com/pingcap/tidb/issues/32089)
    - `ENUM`または`SET`カラムのエンコーディングが誤っているため、TiDBが誤ったデータを書き込む問題を修正する [#32302](https://github.com/pingcap/tidb/issues/32302)
    - 新しい照合順序がTiDBで有効になっている場合、`ENUM`または`SET`カラムの`MAX`または`MIN`関数が誤った結果を返す問題を修正する [#31638](https://github.com/pingcap/tidb/issues/31638)
    - `IndexHashJoin`演算子が正常に終了しない問題を修正する [#31062](https://github.com/pingcap/tidb/issues/31062)
    - テーブルに仮想列がある場合、TiDBが誤ったデータを読み取る可能性がある問題を修正 [#30965](https://github.com/pingcap/tidb/issues/30965)
    - ログレベルの設定がスロークエリログに反映されない問題を修正 [#30309](https://github.com/pingcap/tidb/issues/30309)
    - 一部のケースでパーティションテーブルがインデックスを完全に使用できない問題を修正 [#33966](https://github.com/pingcap/tidb/issues/33966)
    - TiDBのバックグラウンドHTTPサービスが正常に終了せず、クラスターが異常な状態になる可能性がある問題を修正 [#30571](https://github.com/pingcap/tidb/issues/30571)
    - TiDBが予期せず多くの認証失敗ログを出力する可能性がある問題を修正 [#29709](https://github.com/pingcap/tidb/issues/29709)
    - システム変数 `max_allowed_packet` が効果を発揮しない問題を修正 [#31422](https://github.com/pingcap/tidb/issues/31422)
    - `REPLACE` 文が自動IDが範囲外の場合に他の行を誤って変更してしまう問題を修正 [#29483](https://github.com/pingcap/tidb/issues/29483)
    - スロークエリログが正常に出力されず、多くのメモリを消費する可能性がある問題を修正 [#32656](https://github.com/pingcap/tidb/issues/32656)
    - NATURAL JOIN の結果が予期しない列を含む可能性がある問題を修正 [#24981](https://github.com/pingcap/tidb/issues/29481)
    - `ORDER BY` と `LIMIT` を1つの文で一緒に使用し、プレフィックス列インデックスを使用してデータをクエリする際、誤った結果が出力される可能性がある問題を修正 [#29711](https://github.com/pingcap/tidb/issues/29711)
    - DOUBLE型の自動増分列が楽観的トランザクションのリトライ時に変更される可能性がある問題を修正 [#29892](https://github.com/pingcap/tidb/issues/29892)
    - STR_TO_DATE 関数がマイクロ秒部の前ゼロを正しく処理できない問題を修正 [#30078](https://github.com/pingcap/tidb/issues/30078)
    - TiDBが TiFlash を使用して空の範囲のテーブルをスキャンする際に、間違った結果を取得する可能性がある問題を修正。ただし、TiFlashは現在空の範囲のテーブルの読み取りをサポートしていない [#33083](https://github.com/pingcap/tidb/issues/33083)

+ TiKV

    - 廃れたメッセージが TiKV でパニックを引き起こすバグを修正 [#12023](https://github.com/tikv/tikv/issues/12023)
    - メモリメトリクスのオーバーフローによる断続的なパケットロスとメモリ不足 (OOM) の問題を修正 [#12160](https://github.com/tikv/tikv/issues/12160)
    - Ubuntu 18.04 での TiKV プロファイリング時に発生する潜在的なパニックの問題を修正 [#9765](https://github.com/tikv/tikv/issues/9765)
    - tikv-ctl が誤った文字列一致により誤った結果を返す問題を修正 [#12329](https://github.com/tikv/tikv/issues/12329)
    - レプリカ読み取りが直線性を侵害する可能性があるバグを修正 [#12109](https://github.com/tikv/tikv/issues/12109)
    - TiKV が2年以上実行されている場合にパニックする可能性があるバグを修正 [#11940](https://github.com/tikv/tikv/issues/11940)
    - フローコントロールが有効で `level0_slowdown_trigger` が明示的に設定されているときの QPS 低下の問題を修正 [#11424](https://github.com/tikv/tikv/issues/11424)
    - cgroupコントローラーがマウントされていない場合に発生するパニックの問題を修正 [#11569](https://github.com/tikv/tikv/issues/11569)
    - 遅れているリージョンピアでのリージョンマージによるメタデータの破損の可能性を修正 [#11526](https://github.com/tikv/tikv/issues/11526)
    - TiKV の操作を停止した後、解決済みTSのレイテンシーが増加する問題を修正 [#11351](https://github.com/tikv/tikv/issues/11351)
    - 極端な条件下でリージョンマージ、ConfChange、およびSnapshotが同時に発生したときに発生するパニックの問題を修正 [#11475](https://github.com/tikv/tikv/issues/11475)
    - tikv-ctl が正しいリージョン関連情報を返せないバグを修正 [#11393](https://github.com/tikv/tikv/issues/11393)
    - DECIMALで割った結果がゼロの場合に負の符号が表示されるバグを修正 [#29586](https://github.com/pingcap/tidb/issues/29586)
    - 悲観的トランザクションモードでの事前書き込みリクエストを再試行することが稀にデータ不整合のリスクを引き起こす可能性がある問題を修正 [#11187](https://github.com/tikv/tikv/issues/11187)
    - 統計スレッドのモニタリングデータによるメモリリークを修正 [#11195](https://github.com/tikv/tikv/issues/11195)
    - インスタンス別gRPCリクエストの平均遅延がTiKVメトリクスで不正確である問題を修正 [#11299](https://github.com/tikv/tikv/issues/11299)
    - Peerステータスが `Applying` の場合にスナップショットファイルを削除するときに発生するパニックの問題を修正 [#11746](https://github.com/tikv/tikv/issues/11746)
    - GCワーカーがビジーなときにデータの範囲を削除できない (内部コマンド `unsafe_destroy_range` が実行される) 問題を修正 [#11903](https://github.com/tikv/tikv/issues/11903)
    - 初期化されていないレプリカを削除すると、古いレプリカが再作成される可能性がある問題を修正 [#10533](https://github.com/tikv/tikv/issues/10533)
    - TiKV がメモリロックを検出できない問題を修正し、逆テーブルスキャンを実行する場合の問題を修正 [#11440](https://github.com/tikv/tikv/issues/11440)
    - コルーチンが速すぎるために時折発生するデッドロックの問題を修正 [#11549](https://github.com/tikv/tikv/issues/11549)
    - ターゲットリージョンが無効な場合にTiKVが予期せずパニックし、ピアを破壊する問題を修正 [#10210](https://github.com/tikv/tikv/issues/10210)
    - リージョンをマージする際に、初期化されていないピアに置き換えられたターゲットピアが存在する場合にTiKVがパニックする問題を修正 [#12232](https://github.com/tikv/tikv/issues/12232)
    - スナップショット適用が中止されたときに発生するTiKVのパニックの問題を修正 [#11618](https://github.com/tikv/tikv/issues/11618)
    - オペレータ実行に失敗した際に、TiKVが送信されるスナップショットの数を正しく計算できないバグを修正 [#11341](https://github.com/tikv/tikv/issues/11341)

+ PD

    - リージョンスキャッタラースケジューリングが一部のピアを失う問題を修正 [#4565](https://github.com/tikv/pd/issues/4565)
    - コールドスポットデータがホットスポット統計情報から削除されない問題を修正 [#4390](https://github.com/tikv/pd/issues/4390)

+ TiFlash

    - MPPタスクが永続スレッドをリークする可能性があるバグを修正 [#4238](https://github.com/pingcap/tiflash/issues/4238)
    - マルチバリューエクスプレッションにおける`IN`の結果が正しくない問題を修正 [#4016](https://github.com/pingcap/tiflash/issues/4016)
    - 日付フォーマットが `'\n'` を無効な区切り文字として識別する問題を修正 [#4036](https://github.com/pingcap/tiflash/issues/4036)
    - 読み取りワークロードが重い条件下で列を追加した後にクエリエラーが発生する可能性がある問題を修正 [#3967](https://github.com/pingcap/tiflash/issues/3967)
    - 無効なストレージディレクトリ構成によって予期しない動作を引き起こすバグを修正 [#4093](https://github.com/pingcap/tiflash/issues/4093)
    - 一部の例外を適切に処理しないバグを修正 [#4101](https://github.com/pingcap/tiflash/issues/4101)
    - `STR_TO_DATE()` 関数がマイクロ秒を解析する際に、先頭のゼロを誤って処理する問題を修正 [#3557](https://github.com/pingcap/tiflash/issues/3557)
    - `INT` を `DECIMAL` にキャストする際にオーバーフローが発生する問題を修正 [#3920](https://github.com/pingcap/tiflash/issues/3920)
    - `DATETIME` を `DECIMAL` にキャストする際に誤った結果が発生する問題を修正 [#4151](https://github.com/pingcap/tiflash/issues/4151)
    - `FLOAT` を `DECIMAL` にキャストする際にオーバーフローが発生する問題を修正 [#3998](https://github.com/pingcap/tiflash/issues/3998)
    - `CastStringAsReal` の動作がTiFlashとTiDB、TiKVで一貫しないことがある問題を修正 [#3475](https://github.com/pingcap/tiflash/issues/3475)
- `CastStringAsDecimal`の挙動がTiFlashとTiDBまたはTiKVで不一致である問題を修正 [#3619](https://github.com/pingcap/tiflash/issues/3619)
    - TiFlashが再起動後に`EstablishMPPConnection`エラーを返す可能性のある問題を修正 [#3615](https://github.com/pingcap/tiflash/issues/3615)
    - TiFlashのレプリカ数を0に設定した後に旧データを回収できない問題を修正 [#3659](https://github.com/pingcap/tiflash/issues/3659)
    - プライマリキーが`handle`である場合にプライマリキーカラムを拡張するとデータの不整合が生じる可能性のある問題を修正 [#3569](https://github.com/pingcap/tiflash/issues/3569)
    - SQL文に極端に長いネストされた式が含まれる場合にパースエラーが発生する可能性のある問題を修正 [#3354](https://github.com/pingcap/tiflash/issues/3354)
    - クエリに`where <string>`句が含まれる場合に誤った結果が返される可能性のある問題を修正 [#3447](https://github.com/pingcap/tiflash/issues/3447)
    - `new_collations_enabled_on_first_bootstrap`が有効な場合に誤った結果が返される可能性のある問題を修正 [#3388](https://github.com/pingcap/tiflash/issues/3388), [#3391](https://github.com/pingcap/tiflash/issues/3391)
    - TLSが有効な場合に発生するパニックの問題を修正 [#4196](https://github.com/pingcap/tiflash/issues/4196)
    - メモリ制限が有効な場合に発生するパニックの問題を修正 [#3902](https://github.com/pingcap/tiflash/issues/3902)
    - MPPクエリの停止時にTiFlashが時々クラッシュする問題を修正 [#3401](https://github.com/pingcap/tiflash/issues/3401)
    - `Unexpected type of column: Nullable(Nothing)`の予期しないエラーを修正 [#3351](https://github.com/pingcap/tiflash/issues/3351)
    - 遅れたRegionピアでの領域マージによるメタデータの破損を修正 [#4437](https://github.com/pingcap/tiflash/issues/4437)
    - `JOIN`を含むクエリがエラーが発生した場合にハングする可能性のある問題を修正 [#4195](https://github.com/pingcap/tiflash/issues/4195)
    - 正しく実行計画がないためにMPPクエリで誤った結果が返される可能性のある問題を修正 [#3389](https://github.com/pingcap/tiflash/issues/3389)

+ ツール

    + Backup & Restore (BR)

        - BRがRawKVをバックアップできない問題を修正 [#32607](https://github.com/pingcap/tidb/issues/32607)

    + TiCDC

        - デフォルト値がレプリケートできない問題を修正 [#3793](https://github.com/pingcap/tiflow/issues/3793)
        - 一部のケースでシーケンスが誤ってレプリケートされる不具合を修正 [#4563](https://github.com/pingcap/tiflow/issues/4552)
        - PDリーダーが停止された場合にTiCDCノードが異常終了するバグを修正 [#4248](https://github.com/pingcap/tiflow/issues/4248)
        - `batch-replace-enable`が無効な場合にMySQLシンクが重複した`replace` SQL文を生成する不具合を修正 [#4501](https://github.com/pingcap/tiflow/issues/4501)
        - デフォルトカラム値を出力する際に発生するパニックおよびデータの不整合の問題を修正 [#3929](https://github.com/pingcap/tiflow/issues/3929)
        - `mq sink write row`にモニタリングデータがない問題を修正 [#3431](https://github.com/pingcap/tiflow/issues/3431)
        - `min.insync.replicas`が`replication-factor`よりも小さい場合にレプリケーションが実行されない問題を修正 [#3994](https://github.com/pingcap/tiflow/issues/3994)
        - レプリケーションタスクが削除された際に発生する潜在的なパニック問題を修正 [#3128](https://github.com/pingcap/tiflow/issues/3128)
        - 必要なプロセッサ情報が存在しない場合にHTTP APIがパニックする不具合を修正 [#3840](https://github.com/pingcap/tiflow/issues/3840)
        - 不正確なチェックポイントによる潜在的なデータ損失の問題を修正 [#3545](https://github.com/pingcap/tiflow/issues/3545)
        - デッドロックがレプリケーションタスクをスタックさせる可能性のある問題を修正 [#4055](https://github.com/pingcap/tiflow/issues/4055)
        - etcdでタスク状態を手動でクリーニングする際にTiCDCでパニックが発生する問題を修正 [#2980](https://github.com/pingcap/tiflow/issues/2980)
        - DDL文中の特殊コメントによってレプリケーションタスクが停止する問題を修正 [#3755](https://github.com/pingcap/tiflow/issues/3755)
        - `config.Metadata.Timeout`の設定ミスによってレプリケーションが停止する問題を修正 [#3352](https://github.com/pingcap/tiflow/issues/3352)
        - RHELリリースのタイムゾーンの問題によってサービスを開始できない問題を修正 [#3584](https://github.com/pingcap/tiflow/issues/3584)
        - クラスターをアップグレードした後に`stopped`チェンジフィードが自動的に再開する問題を修正 [#3473](https://github.com/pingcap/tiflow/issues/3473)
        - MySQLシンクのデッドロックによって過度な警告が発生する問題を修正 [#2706](https://github.com/pingcap/tiflow/issues/2706)
        - CanalおよびMaxwellプロトコルで`enable-old-value`構成項目が自動的に`true`に設定されないバグを修正 [#3676](https://github.com/pingcap/tiflow/issues/3676)
        - AvroシンクがJSONタイプのカラムのパースをサポートしていない問題を修正 [#3624](https://github.com/pingcap/tiflow/issues/3624)
        - changefeedのチェックポイントラグに負の値エラーが発生する問題を修正 [#3010](https://github.com/pingcap/tiflow/issues/3010)
        - コンテナ環境でのOOMの問題を修正 [#1798](https://github.com/pingcap/tiflow/issues/1798)
        - DDL処理後のメモリリークの問題を修正 [#3174](https://github.com/pingcap/tiflow/issues/3174)
        - 同一ノードで表が繰り返しスケジュールされるとchangefeedがスタックする問題を修正 [#4464](https://github.com/pingcap/tiflow/issues/4464)
        - PDノードが異常な場合にオープンAPI経由でステータスを問い合わせる際にブロックされる不具合を修正 [#4778](https://github.com/pingcap/tiflow/issues/4778)
        - オーナー変更によって発生する誤ったメトリクスの問題を修正 [#4774](https://github.com/pingcap/tiflow/issues/4774)
        - Unified Sorterで使用されるworkerpoolの安定性の問題を修正 [#4447](https://github.com/pingcap/tiflow/issues/4447)
        - `cached region`モニタリングメトリックが負の値である問題を修正 [#4300](https://github.com/pingcap/tiflow/issues/4300)

    + TiDB Lightning

        - TiDB Lightningが`mysql.tidb`テーブルへのアクセス権限を持っていない場合に誤ったインポート結果が発生する問題を修正 [#31088](https://github.com/pingcap/tidb/issues/31088)
        - "GC life time is shorter than transaction duration"というチェックサムエラーを修正 [#32733](https://github.com/pingcap/tidb/issues/32733)
        - 一部のインポートタスクがソースファイルを含まない場合にTiDB Lightningがメタデータスキーマを削除しない問題を修正 [#28144](https://github.com/pingcap/tidb/issues/28144)
        - S3ストレージパスが存在しない場合にTiDB Lightningがエラーを報告しない問題を修正 [#28031](https://github.com/pingcap/tidb/issues/28031) [#30709](https://github.com/pingcap/tidb/issues/30709)
        - GCS上で1000以上のキーを反復処理する際に発生するエラーを修正 [#30377](https://github.com/pingcap/tidb/issues/30377)