---
title: TiDB 6.1.6 リリースノート
summary: TiDB 6.1.6 における互換性の変更、改善、およびバグ修正について学びます。

# TiDB 6.1.6 リリースノート

リリース日: 2023年4月12日

TiDB バージョン: 6.1.6

クイックアクセス: [クイックスタート](https://docs.pingcap.com/tidb/v6.1/quick-start-with-tidb) | [プロダクションデプロイ](https://docs.pingcap.com/tidb/v6.1/production-deployment-using-tiup) | [インストールパッケージ](https://www.pingcap.com/download/?version=v6.1.6#version-list)

## 互換性の変更

- TiCDC は Avro の FLOAT データのエンコーディングが正しくない問題を修正 [#8490](https://github.com/pingcap/tiflow/issues/8490) @[3AceShowHand](https://github.com/3AceShowHand)

    TiCDC クラスタを v6.1.6 または後続の v6.1.x バージョンにアップグレードする際、Avro を使用して複製されたテーブルに `FLOAT` データ型が含まれている場合は、アップグレード前に Confluent Schema Registry の互換性ポリシーを `None` に手動で調整する必要があります。そうすることで、changefeed がスキーマを正常に更新できるようになります。それ以外の場合、アップグレード後に changefeed はスキーマを更新できなくなり、エラー状態になります。

## 改善

+ TiDB

    - `BatchPointGet` の実行プランを Prepared Plan Cache でキャッシュする機能をサポート [#42125](https://github.com/pingcap/tidb/issues/42125) @[qw4990](https://github.com/qw4990)
    - インデックス結合のためのさらなる SQL フォーマットをサポート [#40505](https://github.com/pingcap/tidb/issues/40505) @[Yisaer](https://github.com/Yisaer)

+ TiKV

    - 1コア未満のCPUで TiKV を起動する機能をサポート [#13586](https://github.com/tikv/tikv/issues/13586) [#13752](https://github.com/tikv/tikv/issues/13752) [#14017](https://github.com/tikv/tikv/issues/14017) @[andreid-db](https://github.com/andreid-db) @[andreid-db](https://github.com/andreid-db)

## バグ修正

+ TiDB

    - `INSERT` 文に対して `ignore_plan_cache` ヒントが機能しない可能性がある問題を修正 [#40079](https://github.com/pingcap/tidb/issues/40079) [#39717](https://github.com/pingcap/tidb/issues/39717) @[qw4990](https://github.com/qw4990)
    - `indexMerge` でエラーが発生した後 TiDB がパニックする可能性がある問題を修正 [#41047](https://github.com/pingcap/tidb/issues/41047) [#40877](https://github.com/pingcap/tidb/issues/40877) @[guo-shaoge](https://github.com/guo-shaoge) @[windtalker](https://github.com/windtalker)
    - 誤った結果が返される可能性がある問題を修正し、TopN 演算子が仮想列を誤って TiKV または TiFlash に押し込んだ場合 [#41355](https://github.com/pingcap/tidb/issues/41355) @[Dousir9](https://github.com/Dousir9)
    - `Prepare` または `Execute` を使用して一部の仮想テーブルをクエリする際に、大量のリージョンが存在するがテーブル ID が押し込まれない場合に PD OOM 問題が発生する問題を修正 [#39605](https://github.com/pingcap/tidb/issues/39605) @[djshow832](https://github.com/djshow832)
    - `int_col in (decimal...)` の条件を処理する際に Plan Cache が FullScan プランをキャッシュし、誤った結果を返す問題を修正 [#40224](https://github.com/pingcap/tidb/issues/40224) @[qw4990](https://github.com/qw4990)
    - SET 型の列に対して IndexMerge プランが誤った範囲を生成する問題を修正 [#41273](https://github.com/pingcap/tidb/issues/41273) [#41293](https://github.com/pingcap/tidb/issues/41293) @[time-and-fate](https://github.com/time-and-fate)
    - 正の `TINYINT`/`SMALLINT`/`INT` 値と `DECIMAL`/`FLOAT`/`DOUBLE` 値よりも小さい値を比較する際に、間違った結果が生じる可能性がある問題を修正 [#41736](https://github.com/pingcap/tidb/issues/41736) @[LittleFall](https://github.com/LittleFall)
    - クエリを介して `INFORMATION_SCHEMA.CLUSTER_SLOW_QUERY` テーブルを確認する際に、TiDB サーバーがメモリ不足になる可能性がある問題を修正。この問題は Grafana ダッシュボードで遅いクエリをチェックする際に発生する可能性があります [#33893](https://github.com/pingcap/tidb/issues/33893) @[crazycs520](https://github.com/crazycs520)
    - range パーティションが複数の `MAXVALUE` パーティションを許可する問題を修正 [#36329](https://github.com/pingcap/tidb/issues/36329) @[u5surf](https://github.com/u5surf)
    - Plan Cache が Shuffle 演算子をキャッシュし、誤った結果を返す問題を修正 [#38335](https://github.com/pingcap/tidb/issues/38335) @[qw4990](https://github.com/qw4990)
    - タイムゾーンのデータ競合がデータ-インデックスの不整合を引き起こす可能性がある問題を修正 [#40710](https://github.com/pingcap/tidb/issues/40710) @[wjhuang2016](https://github.com/wjhuang2016)
    - `indexMerge` でゴルーチンリークが発生する可能性がある問題を修正 [#41545](https://github.com/pingcap/tidb/issues/41545) [#41605](https://github.com/pingcap/tidb/issues/41605) @[guo-shaoge](https://github.com/guo-shaoge) @[guo-shaoge](https://github.com/guo-shaoge)
    - トランザクション内で `Cursor Fetch` を使用して他のステートメントを実行する場合に、Fetch および Close コマンドが誤った結果を返すか TiDB をパニックさせる可能性がある問題を修正 [#40094](https://github.com/pingcap/tidb/issues/40094) @[YangKeao](https://github.com/YangKeao)
    - DDL を使用して浮動小数点型の長さを変更せずに小数点以下の桁数を減らす場合に、古いデータが変わらない問題を修正 [#41281](https://github.com/pingcap/tidb/issues/41281) @[zimulala](https://github.com/zimulala)
    - `information_schema.columns` テーブルを結合することで TiDB がパニックする問題を修正 [#32459](https://github.com/pingcap/tidb/issues/32459) @[tangenta](https://github.com/tangenta)
    - 実行プランの生成時に一貫しない InfoSchema を取得することで TiDB がパニックする問題を修正 [#41622](https://github.com/pingcap/tidb/issues/41622) @[tiancaiamao](https://github.com/tiancaiamao)
    - 実行中に TiFlash が生成された列に対してエラーを報告する問題を修正 [#40663](https://github.com/pingcap/tidb/issues/40663) @[guo-shaoge](https://github.com/guo-shaoge)
    - 単一の SQL ステートメント内で異なるパーティション化されたテーブルが現れると TiDB が誤った結果を返す可能性がある問題を修正 [#42135](https://github.com/pingcap/tidb/issues/42135) @[mjonss](https://github.com/mjonss)
    - Plan Cache が Shuffle 演算子をキャッシュし、誤った結果を返す可能性がある問題を修正 [#38335](https://github.com/pingcap/tidb/issues/38335) @[qw4990](https://github.com/qw4990) @[fzzf678](https://github.com/fzzf678)
    - `SET` 型の列を含むテーブルを Index Merge を使用して読み取る際に、誤った結果を出す可能性がある問題を修正 [#41293](https://github.com/pingcap/tidb/issues/41293) @[time-and-fate](https://github.com/time-and-fate)
    - 準備済みプランキャッシュが有効な場合にフルインデックススキャンがエラーを引き起こす可能性がある問題を修正 [#42150](https://github.com/pingcap/tidb/issues/42150) @[fzzf678](https://github.com/fzzf678)
    - トランザクション内で `PointUpdate` を実行した後、`SELECT` ステートメントに対して誤った結果が返される可能性がある問題を修正 [#28011](https://github.com/pingcap/tidb/issues/28011) @[zyguan](https://github.com/zyguan)
- メモリーリークやパフォーマンスの低下を避けるために、期限切れのリージョンキャッシュを定期的にクリアします [#40461](https://github.com/pingcap/tidb/issues/40461) @[sticnarf](https://github.com/sticnarf) @[zyguan](https://github.com/zyguan)
- `INSERT IGNORE`および`REPLACE`ステートメントが値を変更しないキーをロックしない問題を修正 [#42121](https://github.com/pingcap/tidb/issues/42121) @[zyguan](https://github.com/zyguan)

+ TiKV

    - `const Enum`タイプを他のタイプにキャストする際に発生するエラーを修正 [#14156](https://github.com/tikv/tikv/issues/14156) @[wshwsh12](https://github.com/wshwsh12)
    - CPUクォータ制限の問題を修正 [#13084](https://github.com/tikv/tikv/issues/13084) @[BornChanger](https://github.com/BornChanger)
    - 不正なスナップショット最終インデックスの問題を修正 [#12618](https://github.com/tikv/tikv/issues/12618) @[LintianShi](https://github.com/LintianShi)

+ PD

    - リージョンスキャッタがリーダーの均等な分布を引き起こす可能性のある問題を修正 [#6017](https://github.com/tikv/pd/issues/6017) @[HunDunDM](https://github.com/HunDunDM)
    - オンライン不安定回復のタイムアウトメカニズムが機能しない問題を修正 [#6107](https://github.com/tikv/pd/issues/6107) @[v01dstar](https://github.com/v01dstar)

+ TiFlash

    - セミジョインがカルテシアン積を計算する際に過剰なメモリを使用していた問題を修正 [#6730](https://github.com/pingcap/tiflash/issues/6730) @[gengliqi](https://github.com/gengliqi)
    - TiFlashログ検索が遅すぎる問題を修正 [#6829](https://github.com/pingcap/tiflash/issues/6829) @[hehechen](https://github.com/hehechen)
    - 新しい照合順序を有効にした後、TopN/ソート演算子が正しくない結果を生成する問題を修正 [#6807](https://github.com/pingcap/tiflash/issues/6807) @[xzhangxian1008](https://github.com/xzhangxian1008)
    - 特定のケースでDecimalのキャストが誤って切り上げる問題を修正 [#6994](https://github.com/pingcap/tiflash/issues/6994) @[windtalker](https://github.com/windtalker)
    - 生成列をTiFlashが認識できない問題を修正 [#6801](https://github.com/pingcap/tiflash/issues/6801) @[guo-shaoge](https://github.com/guo-shaoge)
    - 特定のケースでDecimalの除算が最後の桁を切り上げない問題を修正 [#7022](https://github.com/pingcap/tiflash/issues/7022) @[LittleFall](https://github.com/LittleFall)

+ ツール

    + TiCDC

        - データレプリケーション中の`UPDATE`および`INSERT`ステートメントの順序が不整合になると、`Duplicate entry`エラーを引き起こす問題を修正 [#8597](https://github.com/pingcap/tiflow/issues/8597) @[sdojjy](https://github.com/sdojjy)
        - PDとTiCDCの間のネットワーク分離によって引き起こされるTiCDCサービスの異常終了問題を修正 [#8562](https://github.com/pingcap/tiflow/issues/8562) @[overvenus](https://github.com/overvenus)
        - `CHARACTER SET`が非NULLのユニークインデックスを持つカラムで指定されている場合、TiDBまたはMySQLシンクにデータを複製する際のデータの不整合を修正 [#8420](https://github.com/pingcap/tiflow/issues/8420) @[zhaoxinyu](https://github.com/zhaoxinyu)
        - `db sorter`のメモリ使用量が`cgroup memory limit`によって制御されない問題を修正 [#8588](https://github.com/pingcap/tiflow/issues/8588) @[amyangfei](https://github.com/amyangfei)
        - 無効な入力に対する`cdc cli`のエラーメッセージを最適化 [#7903](https://github.com/pingcap/tiflow/issues/7903) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - S3ストレージの障害に耐えられるリドーログの許容期間が不十分な問題を修正 [#8089](https://github.com/pingcap/tiflow/issues/8089) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - PDが異常な場合に変更フィードを一時停止すると、間違ったステータスが発生する問題を修正 [#8330](https://github.com/pingcap/tiflow/issues/8330) @[sdojjy](https://github.com/sdojjy)

    + TiDB Lightning

        - 競合解決ロジック（`duplicate-resolution`）が不一致のチェックサムにつながる可能性がある問題を修正 [#40657](https://github.com/pingcap/tidb/issues/40657) @[gozssky](https://github.com/gozssky)
        - リージョンの分割フェーズでTiDB Lightningがパニックを引き起こす問題を修正 [#40934](https://github.com/pingcap/tidb/issues/40934) @[lance6716](https://github.com/lance6716)
        - ローカルバックエンドモードでデータをインポートする際、インポート対象テーブルの複合プライマリキーに`auto_random`カラムがあり、ソースデータでカラムの値が指定されていない場合、対象カラムが自動的にデータを生成しない問題を修正 [#41454](https://github.com/pingcap/tidb/issues/41454) @[D3Hunter](https://github.com/D3Hunter)
        - パラレルインポート中にすべてのTiDB Lightningインスタンスの最後以外がローカルで重複するレコードをエンカウンターした場合、競合解決が誤ってスキップされる可能性がある問題を修正 [#40923](https://github.com/pingcap/tidb/issues/40923) @[lichunzhu](https://github.com/lichunzhu)