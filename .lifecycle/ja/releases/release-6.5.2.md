---
title: TiDB 6.5.2 リリースノート
summary: TiDB 6.5.2 における互換性の変更、改善、およびバグ修正について学びます。

# TiDB 6.5.2 リリースノート

リリース日: 2023年4月21日

TiDB バージョン: 6.5.2

クイックアクセス: [クイックスタート](https://docs.pingcap.com/tidb/v6.5/quick-start-with-tidb) | [本番展開](https://docs.pingcap.com/tidb/v6.5/production-deployment-using-tiup) | [インストールパッケージ](https://www.pingcap.com/download/?version=v6.5.2#version-list)

## 互換性の変更

- TiCDC が Avro で `FLOAT` データのエンコーディングが不正である問題を修正 [#8490](https://github.com/pingcap/tiflow/issues/8490) @[3AceShowHand](https://github.com/3AceShowHand)

    TiCDC クラスタを v6.5.2 またはそれ以降の v6.5.x バージョンにアップグレードする際、Avro を使用して複製されたテーブルに `FLOAT` データ型が含まれている場合、アップグレード前に Confluent Schema Registry の互換性ポリシーを `None` に手動で調整する必要があります。これにより changefeed がスキーマを正常に更新できるようにします。そうしないと、アップグレード後、changefeed はスキーマを更新できずにエラー状態になります。

- ストレージサービスにパーティションテーブルのレプリケーション中のデータ損失の潜在的な問題を修正するため、TiCDC [`sink.enable-partition-separator`](/ticdc/ticdc-changefeed-config.md#changefeed-configuration-parameters) 構成項目のデフォルト値を `false` から `true` に変更しました。つまり、テーブルのパーティションはデフォルトで別々のディレクトリに格納されます。データ損失の問題を回避するために、値を `true` のままにすることを推奨します。[#8724](https://github.com/pingcap/tiflow/issues/8724) @[CharlesCheung96](https://github.com/CharlesCheung96)

## 改善

+ TiDB

    - `BatchPointGet` の実行計画をキャッシュするサポートを追加 [#42125](https://github.com/pingcap/tidb/issues/42125) @[qw4990](https://github.com/qw4990)
    - Index Join に対するさらに多くの SQL フォーマットをサポート [#40505](https://github.com/pingcap/tidb/issues/40505) @[Yisaer](https://github.com/Yisaer)
    - 一部の Index Merge リーダーのログレベルを `"info"` から `"debug"` に変更 [#41949](https://github.com/pingcap/tidb/issues/41949) @[yibin87](https://github.com/yibin87)
    - Limit に Range パーティションテーブルでの `distsql_concurrency` 設定を最適化し、クエリ遅延を減らす[#41480](https://github.com/pingcap/tidb/issues/41480) @[you06](https://github.com/you06)

+ TiFlash

    - TiFlash 読み取り中のタスクスケジューリングの CPU 消費を削減 [#6495](https://github.com/pingcap/tiflash/issues/6495) @[JinheLin](https://github.com/JinheLin)
    - デフォルト構成を使用した BR および TiDB Lightning から TiFlash へのデータインポートのパフォーマンスを改善 [#7272](https://github.com/pingcap/tiflash/issues/7272) @[breezewish](https://github.com/breezewish)

+ ツール

    + TiCDC

        - TiCDC Open API v2.0 をリリース [#8743](https://github.com/pingcap/tiflow/issues/8743) @[sdojjy](https://github.com/sdojjy)
        - OOM 問題を防ぐために `gomemlimit` を導入 [#8675](https://github.com/pingcap/tiflow/issues/8675) @[amyangfei](https://github.com/amyangfei)
        - `UPDATE` 文のバッチ実行を含むシナリオでのレプリケーションパフォーマンスを最適化するためにマルチステートメントアプローチをサポート [#8057](https://github.com/pingcap/tiflow/issues/8057) @[amyangfei](https://github.com/amyangfei)
        - 災害復旧シナリオで RTO を減らし、スループットを向上させるためにリドーアプライヤでトランザクションの分割をサポート [#8318](https://github.com/pingcap/tiflow/issues/8318) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - リドーログでの DDL イベントの適用をサポート [#8361](https://github.com/pingcap/tiflow/issues/8361) @[CharlesCheung96](https://github.com/CharlesCheung96)

    + TiDB Lightning

        - BOM ヘッダーを持つ CSV データファイルのインポートをサポート [#40744](https://github.com/pingcap/tidb/issues/40744) @[dsdashun](https://github.com/dsdashun)

## バグ修正

+ TiDB
    - キャッシュテーブルに新しい列が追加された後、その値が列のデフォルト値ではなく `NULL` になる問題を修正 [#42928](https://github.com/pingcap/tidb/issues/42928) @[lqs](https://github.com/lqs)
    - 多くのパーティションおよび TiFlash レプリカを持つパーティションテーブルに対する `TRUNCATE TABLE` の実行中に発生する書き込み競合による DDL リトライの問題を修正 [#42940](https://github.com/pingcap/tidb/issues/42940) @[mjonss](https://github.com/mjonss)
    - `DROP TABLE` 操作が実行されている際の `ADMIN SHOW DDL JOBS` 結果にテーブル名が抜けている問題を修正 [#42268](https://github.com/pingcap/tidb/issues/42268) @[tiancaiamao](https://github.com/tiancaiamao)
    - cgroup 情報を読み取る際にエラーが発生し、TiDB サーバーが起動できない問題を修正。エラーメッセージ: "can't read file memory.stat from cgroup v1: open /sys/memory.stat no such file or directory" [#42659](https://github.com/pingcap/tidb/issues/42659) @[hawkingrei](https://github.com/hawkingrei)
    - DDL データのバックフィル実行時にトランザクション内で頻繁な書き込み競合が発生する問題を修正 [#24427](https://github.com/pingcap/tidb/issues/24427) @[mjonss](https://github.com/mjonss)
    - 実行計画の生成時に整合性のない InfoSchema が取得された際に TiDB サーバーがパニックを引き起こす問題を修正 [#41622](https://github.com/pingcap/tidb/issues/41622) @[tiancaiamao](https://github.com/tiancaiamao)
    - DDL を使用して浮動小数点型の長さを変更せず、小数点以下の桁数を減らす際に、古いデータが変わらない問題を修正 [#41281](https://github.com/pingcap/tidb/issues/41281) @[zimulala](https://github.com/zimulala)
    - トランザクション内で `PointUpdate` を実行した後、`SELECT` 文の結果が正しくない問題を修正 [#28011](https://github.com/pingcap/tidb/issues/28011) @[zyguan](https://github.com/zyguan)
    - Cursor Fetch を使用し、Execute、Fetch、Close の間に他のステートメントを実行した場合、Fetch と Close コマンドが間違った結果を返すか、TiDB がパニックを引き起こす可能性がある問題を修正 [#40094](https://github.com/pingcap/tidb/issues/40094) @[YangKeao](https://github.com/YangKeao)
    - `INSERT IGNORE` および `REPLACE` ステートメントが値を変更しないキーをロックしない問題を修正 [#42121](https://github.com/pingcap/tidb/issues/42121) @[zyguan](https://github.com/zyguan)
    - 生成された列に対して TiFlash がエラーを報告する問題を修正 [#40663](https://github.com/pingcap/tidb/issues/40663) @[guo-shaoge](https://github.com/guo-shaoge)
    - 単一の SQL ステートメントに異なるパーティションテーブルが含まれる場合、TiDB が誤った結果を生成する問題を修正 [#42135](https://github.com/pingcap/tidb/issues/42135) @[mjonss](https://github.com/mjonss)
    - プリペアドプランキャッシュが有効な場合にフルインデックススキャンがエラーを発生させる問題を修正 [#42150](https://github.com/pingcap/tidb/issues/42150) @[fzzf678](https://github.com/fzzf678)
    - プリペアドプランキャッシュが有効な場合に IndexMerge が誤った結果を生成する問題を修正 [#41828](https://github.com/pingcap/tidb/issues/41828) @[qw4990](https://github.com/qw4990)
    - `max_prepared_stmt_count` の設定が効果を持たない問題を修正 [#39735](https://github.com/pingcap/tidb/issues/39735) @[xuyifangreeneyes](https://github.com/xuyifangreeneyes)
    - 準備計画キャッシュを有効にした場合、IndexMergeが誤った結果を生成する可能性の問題を修正 [#41828](https://github.com/pingcap/tidb/issues/41828) @[qw4990](https://github.com/qw4990) @[XuHuaiyu](https://github.com/XuHuaiyu)
    - パーティションテーブルの動的トリミングモードでIndex Joinがパニックを引き起こす可能性の問題を修正 [#40596](https://github.com/pingcap/tidb/issues/40596) @[tiancaiamao](https://github.com/tiancaiamao)

+ TiKV

    - TiKVがcgrouppathを処理する際に、`:`文字を正しく解釈しない問題を修正 [#14538](https://github.com/tikv/tikv/issues/14538) @[SpadeA-Tang](https://github.com/SpadeA-Tang)

+ PD

    - PDが意図しない領域に複数のLearnerを追加する可能性の問題を修正 [#5786](https://github.com/tikv/pd/issues/5786) @[HunDunDM](https://github.com/HunDunDM)
    - プレースメントルールを切り替えた際に、Leaderの不均等な配布を引き起こす可能性の問題を修正 [#6195](https://github.com/tikv/pd/issues/6195) @[bufferflies](https://github.com/bufferflies)

+ TiFlash

    - TiFlashが生成されたカラムを認識できない問題を修正 [#6801](https://github.com/pingcap/tiflash/issues/6801) @[guo-shaoge](https://github.com/guo-shaoge)
    - Decimal divisionが特定のケースで最後の桁を適切に切り上げない問題を修正 [#7022](https://github.com/pingcap/tiflash/issues/7022) @[LittleFall](https://github.com/LittleFall)
    - Decimalキャストが特定のケースで適切に切り上げられない問題を修正 [#6994](https://github.com/pingcap/tiflash/issues/6994) @[windtalker](https://github.com/windtalker)
    - 新しい照合順序を有効にした後に、TopN/Sort演算子が誤った結果を生成する問題を修正 [#6807](https://github.com/pingcap/tiflash/issues/6807) @[xzhangxian1008](https://github.com/xzhangxian1008)
    - TiFlashプロセスの故障がTiCDCとの互換性の問題で発生する可能性の問題を修正 [#7212](https://github.com/pingcap/tiflash/issues/7212) @[hongyunyan](https://github.com/hongyunyan)

+ ツール

    + Backup & Restore (BR)

        - TiDBクラスターでPITRバックアップタスクがない場合に、`resolve lock`の頻度が高すぎる問題を修正 [#40759](https://github.com/pingcap/tidb/issues/40759) @[joccau](https://github.com/joccau)
        - PITRリカバリプロセス中の分割リージョンのリトライ待ち時間が不十分な問題を修正 [#42001](https://github.com/pingcap/tidb/issues/42001) @[joccau](https://github.com/joccau)

    + TiCDC

        - TiCDCがデータをオブジェクトストレージにレプリケートする際に、パーティションセパレータが機能しない問題を修正 [#8581](https://github.com/pingcap/tiflow/issues/8581) @[CharlesCheung96](https://github.com/CharlesCheung96) @[hi-rustin](https://github.com/hi-rustin)
        - TiCDCがデータをオブジェクトストレージにレプリケートする際に、テーブルスケジューリングがデータ損失を引き起こす可能性の問題を修正 [#8256](https://github.com/pingcap/tiflow/issues/8256) @[zhaoxinyu](https://github.com/zhaoxinyu)
        - 進行中のDDLステートメントによるレプリケーションのスタックの問題を修正 [#8662](https://github.com/pingcap/tiflow/issues/8662) @[hicqu](https://github.com/hicqu)
        - TiCDCスケーリングがデータ損失を引き起こす可能性がある問題を修正 [#8666](https://github.com/pingcap/tiflow/issues/8666) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - `db sorter`のメモリ使用量が`cgroup memory limit`によって制御されない問題を修正 [#8588](https://github.com/pingcap/tiflow/issues/8588) @[amyangfei](https://github.com/amyangfei)
        - Redoログの適用中に特定のケースでデータ損失が発生する可能性の問題を修正 [#8591](https://github.com/pingcap/tiflow/issues/8591) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - データレプリケーション中の`UPDATE`と`INSERT`ステートメントの不順が`Duplicate entry`エラーを引き起こす可能性の問題を修正 [#8597](https://github.com/pingcap/tiflow/issues/8597) @[sdojjy](https://github.com/sdojjy)
        - PDとTiCDCの間のネットワーク分離によってTiCDCサービスの異常終了が発生する可能性の問題を修正 [#8562](https://github.com/pingcap/tiflow/issues/8562) @[overvenus](https://github.com/overvenus)
        - Kubernetes上でTiCDCクラスターの優れたアップグレードが失敗する問題を修正 [#8484](https://github.com/pingcap/tiflow/issues/8484) @[overvenus](https://github.com/overvenus)
        - すべてのダウンストリームKafkaサーバーが利用できない場合にTiCDCサーバーがパニックする問題を修正 [#8523](https://github.com/pingcap/tiflow/issues/8523) @[3AceShowHand](https://github.com/3AceShowHand)
        - changefeedの再起動がデータ損失を引き起こすか、チェックポイントが前進できない可能性の問題を修正 [#8242](https://github.com/pingcap/tiflow/issues/8242) @[overvenus](https://github.com/overvenus)