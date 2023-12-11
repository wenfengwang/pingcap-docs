---
title: TiDB 6.1.2リリースノート
---

# TiDB 6.1.2リリースノート

リリース日: 2022年10月24日

TiDBバージョン: 6.1.2

クイックアクセス: [クイックスタート](https://docs.pingcap.com/tidb/v6.1/quick-start-with-tidb) | [本番デプロイ](https://docs.pingcap.com/tidb/v6.1/production-deployment-using-tiup) | [インストールパッケージ](https://www.pingcap.com/download/?version=v6.1.2#version-list)

## 改善点

+ TiDB

    - 1つのテーブルで配置ルールとTiFlashレプリカを同時に設定することを許可 [#37171](https://github.com/pingcap/tidb/issues/37171) @[lcwangchao](https://github.com/lcwangchao)

+ TiKV

    - Raftstoreが1つのピアが到達不能になった後に多くのメッセージをブロードキャストするのを避けるために`unreachable_backoff`アイテムの構成をサポートする [#13054](https://github.com/tikv/tikv/issues/13054) @[5kbpers](https://github.com/5kbpers)
    - RocksDBの書き込みスタール設定をフローコントロール閾値よりも小さい値に構成することをサポートする [#13467](https://github.com/tikv/tikv/issues/13467) @[tabokie](https://github.com/tabokie)

+ ツール

    + TiDB Lightning

        - チェックサム中の再試行可能なエラーを追加し、頑健性を向上させる [#37690](https://github.com/pingcap/tidb/issues/37690) @[D3Hunter](https://github.com/D3Hunter)

    + TiCDC

        - リージョンワーカーの性能を向上させ、バッチで解決TSを処理することでパフォーマンスを向上させる [#7078](https://github.com/pingcap/tiflow/issues/7078) @[sdojjy](https://github.com/sdojjy)

## バグ修正

+ TiDB

    - データベースレベルの権限が誤ってクリーンアップされる問題を修正 [#38363](https://github.com/pingcap/tidb/issues/38363) @[dveeden](https://github.com/dveeden)
    - `SHOW CREATE PLACEMENT POLICY`の出力が正しくない問題を修正 [#37526](https://github.com/pingcap/tidb/issues/37526) @[xhebox](https://github.com/xhebox)
    - 1つのPDノードがダウンしたとき、他のPDノードへの再試行がないため`information_schema.TIKV_REGION_STATUS`のクエリが失敗する問題を修正 [#35708](https://github.com/pingcap/tidb/issues/35708) @[tangenta](https://github.com/tangenta)
    - `UNION`演算子が予期しない空の結果を返す可能性がある問題を修正 [#36903](https://github.com/pingcap/tidb/issues/36903) @[tiancaiamao](https://github.com/tiancaiamao)
    - TiFlashのパーティション化されたテーブルでダイナミックモードを有効にしたときに発生する誤った結果を修正 [#37254](https://github.com/pingcap/tidb/issues/37254) @[wshwsh12](https://github.com/wshwsh12)
    - リージョンがマージされるとリージョンキャッシュがタイムリーにクリーンアップされない問題を修正 [#37141](https://github.com/pingcap/tidb/issues/37141) @[sticnarf](https://github.com/sticnarf)
    - KVクライアントが不要なpingメッセージを送信する問題を修正 [#36861](https://github.com/pingcap/tidb/issues/36861) @[jackysp](https://github.com/jackysp)
    - DML実行者を含む`EXPLAIN ANALYZE`文がトランザクションのコミットが完了する前に結果を返す可能性がある問題を修正 [#37373](https://github.com/pingcap/tidb/issues/37373) @[cfzjywxk](https://github.com/cfzjywxk)
    - `ORDER BY`句が相関サブクエリを含む場合に`GROUP CONCAT`が失敗する可能性がある問題を修正 [#18216](https://github.com/pingcap/tidb/issues/18216) @[winoros](https://github.com/winoros)
    - `UPDATE`文に共通のテーブル式（CTE）が含まれる場合に`Can't find column`が報告される問題を修正 [#35758](https://github.com/pingcap/tidb/issues/35758) @[AilinKid](https://github.com/AilinKid)
    - 特定のシナリオで`EXECUTE`が予期しないエラーをスローする可能性がある問題を修正 [#37187](https://github.com/pingcap/tidb/issues/37187) @[Reminiscent](https://github.com/Reminiscent)

+ TiKV

    - バッチスナップショットがリージョンを跨いで行われることによりスナップショットデータが不完全になる可能性がある問題を修正 [#13553](https://github.com/tikv/tikv/issues/13553) @[SpadeA-Tang](https://github.com/SpadeA-Tang)
    - フローコントロールが有効になっており`level0_slowdown_trigger`が明示的に設定されているときにQPSが低下する問題を修正 [#11424](https://github.com/tikv/tikv/issues/11424) @[Connor1996](https://github.com/Connor1996)
    - TiKVがWebアイデンティティプロバイダからエラーを取得してデフォルトプロバイダに戻る際に許可が拒否される問題を修正 [#13122](https://github.com/tikv/tikv/issues/13122) @[3pointer](https://github.com/3pointer)
    - TiKVインスタンスが孤立したネットワーク環境にいるときに数分間TiKVサービスが利用できなくなる問題を修正 [#12966](https://github.com/tikv/tikv/issues/12966) @[cosven](https://github.com/cosven)

+ PD

    - リージョンツリーの統計が不正確になる可能性がある問題を修正 [#5318](https://github.com/tikv/pd/issues/5318) @[rleungx](https://github.com/rleungx)
    - TiFlashの学習レプリカが作成されない可能性がある問題を修正 [#5401](https://github.com/tikv/pd/issues/5401) @[HunDunDM](https://github.com/HunDunDM)
    - PDがダッシュボードプロキシリクエストを正しく処理できない問題を修正 [#5321](https://github.com/tikv/pd/issues/5321) @[HunDunDM](https://github.com/HunDunDM)
    - 不健康なリージョンがPDをパニック状態にさせる可能性がある問題を修正 [#5491](https://github.com/tikv/pd/issues/5491) @[nolouch](https://github.com/nolouch)

+ TiFlash

    - バルク書き込み後にI/O LimiterがクエリリクエストのI/Oスループットを誤って制限し、クエリのパフォーマンスが低下する可能性がある問題を修正 [#5801](https://github.com/pingcap/tiflash/issues/5801) @[JinheLin](https://github.com/JinheLin)
    - クエリがキャンセルされたときにウィンドウ関数がTiFlashをクラッシュさせる可能性がある問題を修正 [#5814](https://github.com/pingcap/tiflash/issues/5814) @[SeaRise](https://github.com/SeaRise)
    - `NULL`値を含む列で主キーを作成した後にパニックが発生する問題を修正 [#5859](https://github.com/pingcap/tiflash/issues/5859) @[JaySon-Huang](https://github.com/JaySon-Huang)

+ ツール

    + TiDB Lightning

        - 無効なメトリックカウンタによってTiDB Lightningがパニックを引き起こす問題を修正 [#37338](https://github.com/pingcap/tidb/issues/37338) @[D3Hunter](https://github.com/D3Hunter)

    + TiDBデータ移行（DM）

        - DMタスクが同期ユニットに入り中断されたときに上流テーブル構造情報が失われる問題を修正 [#7159](https://github.com/pingcap/tiflow/issues/7159) @[lance6716](https://github.com/lance6716)
        - チェックポイントを保存する際にSQLステートメントを分割して大規模トランザクションエラーを修正 [#5010](https://github.com/pingcap/tiflow/issues/5010) @[lance6716](https://github.com/lance6716)
        - DMプリチェックが`INFORMATION_SCHEMA`の`SELECT`権限を必要とする問題を修正 [#7317](https://github.com/pingcap/tiflow/issues/7317) @[lance6716](https://github.com/lance6716)
        - DMワーカーが高速/完全バリデータでDMタスクを実行した後にデッドロックエラーをトリガーする問題を修正 [#7241](https://github.com/pingcap/tiflow/issues/7241) @[buchuitoudegou](https://github.com/buchuitoudegou)
        - DMが`Specified key was too long`エラーを報告する問題を修正 [#5315](https://github.com/pingcap/tiflow/issues/5315) @[lance6716](https://github.com/lance6716)
        - ラテン1データがレプリケーション中に破損する可能性がある問題を修正 [#7028](https://github.com/pingcap/tiflow/issues/7028) @[lance6716](https://github.com/lance6716)

    + TiCDC
- CDCサーバーが完全に起動する前にHTTPリクエストを受信した場合にCDCサーバーがパニックする可能性がある問題を修正する[#6838](https://github.com/pingcap/tiflow/issues/6838) @[asddongmen](https://github.com/asddongmen)
        - アップグレード中のログフラッディングの問題を修正する[#7235](https://github.com/pingcap/tiflow/issues/7235) @[hi-rustin](https://github.com/hi-rustin)
        - チェンジフィードのリドゥログファイルが誤って削除される可能性がある問題を修正する[#6413](https://github.com/pingcap/tiflow/issues/6413) @[hi-rustin](https://github.com/hi-rustin)
        - etcdトランザクションで多数の操作がコミットされた場合にTiCDCが利用できなくなる可能性がある問題を修正する[#7131](https://github.com/pingcap/tiflow/issues/7131) @[hi-rustin](https://github.com/hi-rustin)
        - リドゥログで再帰的でないDDLステートメントが2回実行されるとデータの不整合が発生する可能性がある問題を修正する[#6927](https://github.com/pingcap/tiflow/issues/6927) @[hicqu](https://github.com/hicqu)

    + バックアップとリストア（BR）

        - 復元中に並行性が設定されすぎて領域のバランスが崩れる問題を修正する[#37549](https://github.com/pingcap/tidb/issues/37549) @[3pointer](https://github.com/3pointer)
        - 外部ストレージの認証キーに特殊文字が存在する場合にバックアップとリストアの失敗につながる可能性がある問題を修正する[#37469](https://github.com/pingcap/tidb/issues/37469) @[MoCuishle28](https://github.com/MoCuishle28)