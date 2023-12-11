---
title: TiDB 6.1.4のリリースノート
summary: TiDB 6.1.4の新機能、互換性の変更、改善、およびバグ修正についての情報をご覧ください。

# TiDB 6.1.4のリリースノート

リリース日: 2023年2月8日

TiDBバージョン: 6.1.4

クイックアクセス: [クイックスタート](https://docs.pingcap.com/tidb/v6.1/quick-start-with-tidb) | [本番環境への展開](https://docs.pingcap.com/tidb/v6.1/production-deployment-using-tiup) | [インストールパッケージ](https://www.pingcap.com/download/?version=v6.1.4#version-list)

## 互換性の変更

- TiDB

    - パーティション化されたテーブルでの列の型の変更をサポートしなくなりました。潜在的な正確性の問題のためです [#40620](https://github.com/pingcap/tidb/issues/40620) @[mjonss](https://github.com/mjonss)

## 改善

- TiFlash

    - 高い更新スループットのワークロード下で、TiFlashインスタンスのIOPSを最大95%、書き込み増幅率を最大65%削減しました [#6460](https://github.com/pingcap/tiflash/issues/6460) @[flowbehappy](https://github.com/flowbehappy)

- ツール

    - TiCDC

        - DMLバッチ操作モードを追加し、SQLステートメントがバッチで生成されるときのスループットを向上させました [#7653](https://github.com/pingcap/tiflow/issues/7653) @[asddongmen](https://github.com/asddongmen)
        - RedoログをGCSまたはAzure互換のオブジェクトストレージに保存することをサポートしました [#7987](https://github.com/pingcap/tiflow/issues/7987) @[CharlesCheung96](https://github.com/CharlesCheung96)

    - TiDB Lightning

        - `clusterResourceCheckItem`および`emptyRegionCheckItem`の事前チェック項目の重要度を`Critical`から`Warning`に変更しました [#37654](https://github.com/pingcap/tidb/issues/37654) @[niubell](https://github.com/niubell)

## バグ修正

+ TiDB

    - テーブルを作成する際に、デフォルト値と列の型が一貫しておらず、自動的に修正されない問題を修正しました [#34881](https://github.com/pingcap/tidb/issues/34881) @[Lloyd-Pottiger](https://github.com/Lloyd-Pottiger) @[mjonss](https://github.com/mjonss)
    - `LazyTxn.LockKeys`関数のデータ競合の問題を修正しました [#40355](https://github.com/pingcap/tidb/issues/40355) @[HuSharp](https://github.com/HuSharp)
    - 長時間のセッション接続で`INSERT`または`REPLACE`ステートメントがパニックする可能性がある問題を修正しました [#40351](https://github.com/pingcap/tidb/issues/40351) @[fanrenhoo](https://github.com/fanrenhoo)
    - "カーソル読み取り"メソッドを使用してデータを読み取る際に、GCのためエラーが発生する可能性がある問題を修正しました [#39447](https://github.com/pingcap/tidb/issues/39447) @[zyguan](https://github.com/zyguan)
    - [`pessimistic-auto-commit`](/tidb-configuration-file.md#pessimistic-auto-commit-new-in-v600)構成項目がポイント取得クエリに対して効果がない問題を修正しました [#39928](https://github.com/pingcap/tidb/issues/39928) @[zyguan](https://github.com/zyguan)
    - `INFORMATION_SCHEMA.TIKV_REGION_STATUS`テーブルをクエリすると不正な結果が返る問題を修正しました [#37436](https://github.com/pingcap/tidb/issues/37436) @[zimulala](https://github.com/zimulala)
    - 一部のパターンでの`IN`および`NOT IN`サブクエリが`Can't find column`エラーを報告する問題を修正しました [#37032](https://github.com/pingcap/tidb/issues/37032) @[AilinKid](https://github.com/AilinKid) @[lance6716](https://github.com/lance6716)

- PD

    - PDが予期しなくリージョンに複数のLearnerを追加する問題を修正しました [#5786](https://github.com/tikv/pd/issues/5786) @[HunDunDM](https://github.com/HunDunDM)

+ TiKV

    - 複数の`cgroup`および`mountinfo`レコードがある場合にGitpodでTiDBの起動に失敗する問題を修正しました [#13660](https://github.com/tikv/tikv/issues/13660) @[tabokie](https://github.com/tabokie)
    - `tikv-ctl`が`reset-to-version`コマンドを実行した際に予期せず終了する問題を修正しました [#13829](https://github.com/tikv/tikv/issues/13829) @[tabokie](https://github.com/tabokie)
    - TiKVが誤って`PessimisticLockNotFound`エラーを報告する問題を修正しました [#13425](https://github.com/tikv/tikv/issues/13425) @[sticnarf](https://github.com/sticnarf)
    - 1回の書き込みでのサイズが2 GiBを超えるとTiKVがパニックする問題を修正しました [#13848](https://github.com/tikv/tikv/issues/13848) @[YuJuncen](https://github.com/YuJuncen)
    - 失敗した悲観的DMLの実行後にTiDBとTiKVのネットワーク障害によるデータ不整合の問題を修正しました [#14038](https://github.com/tikv/tikv/issues/14038) @[MyonKeminta](https://github.com/MyonKeminta)
    - 新しい照合順序が有効でない場合、`LIKE`演算子の`_`が非ASCII文字に一致しない問題を修正しました [#13769](https://github.com/tikv/tikv/issues/13769) @[YangKeao](https://github.com/YangKeao) @[tonyxuqqi](https://github.com/tonyxuqqi)

+ TiFlash

    - TiFlashのグローバルロックが長時間ブロックされる問題を修正しました [#6418](https://github.com/pingcap/tiflash/issues/6418) @[SeaRise](https://github.com/SeaRise)
    - 高いスループットの書き込みがOOMを引き起こす問題を修正しました [#6407](https://github.com/pingcap/tiflash/issues/6407) @[JaySon-Huang](https://github.com/JaySon-Huang)

+ ツール

    + バックアップ＆リストア（BR）

        - リージョンサイズを取得に失敗してリストアが中断する問題を修正しました [#36053](https://github.com/pingcap/tidb/issues/36053) @[YuJuncen](https://github.com/YuJuncen)
        - BRが`backupmeta`ファイルをデバッグする際にパニックが発生する問題を修正しました [#40878](https://github.com/pingcap/tidb/issues/40878) @[MoCuishle28](https://github.com/MoCuishle28)

    + TiCDC

        - TiCDCが過剰な数のテーブルをレプリケートする際にチェックポイントが前進しない問題を修正しました [#8004](https://github.com/pingcap/tiflow/issues/8004) @[asddongmen](https://github.com/asddongmen)
        - `transaction_atomicity`および`protocol`を構成ファイル経由で更新できない問題を修正しました [#7935](https://github.com/pingcap/tiflow/issues/7935) @[CharlesCheung96](https://github.com/CharlesCheung96)
        - TiFlashのバージョンがTiCDCのバージョンよりも新しい場合にTiCDCが誤ってエラーを報告する問題を修正しました [#7744](https://github.com/pingcap/tiflow/issues/7744) @[overvenus](https://github.com/overvenus)
        - TiCDCが大きなトランザクションをレプリケートする際にOOMが発生する問題を修正しました [#7913](https://github.com/pingcap/tiflow/issues/7913) @[overvenus](https://github.com/overvenus)
        - 大きなトランザクションを分割せずにデータをレプリケートする際にコンテキストの締め切り時刻が超過する問題を修正しました [#7982](https://github.com/pingcap/tiflow/issues/7982) @[hi-rustin](https://github.com/hi-rustin)
        - `changefeed query`の結果で`sasl-password`がマスクされない問題を修正しました [#7182](https://github.com/pingcap/tiflow/issues/7182) @[dveeden](https://github.com/dveeden)
        - ユーザーがレプリケーションタスクを素早く削除してから同じタスク名で別のタスクを作成するとデータが失われる問題を修正しました [#7657](https://github.com/pingcap/tiflow/issues/7657) @[overvenus](https://github.com/overvenus)

    + TiDBデータ移行（DM）
        - 「SHOW GRANTS」で下流データベース名にワイルドカード（"*"）が含まれている場合に、DMが事前チェック中にエラーを発生させる可能性があるバグを修正 [#7645](https://github.com/pingcap/tiflow/issues/7645) @[lance6716](https://github.com/lance6716)
        - binlogクエリイベントの「COMMIT」によってDMが過剰なログを出力する問題を修正 [#7525](https://github.com/pingcap/tiflow/issues/7525) @[liumengya94](https://github.com/liumengya94)
        - SSLの設定で`ssl-ca`のみが構成されている場合に、DMタスクが開始に失敗する問題を修正 [#7941](https://github.com/pingcap/tiflow/issues/7941) @[liumengya94](https://github.com/liumengya94)
        - 1つのテーブルで「update」と「non-update」の両方の式フィルタが指定されている場合に、すべての`UPDATE`ステートメントがスキップされるバグを修正 [#7831](https://github.com/pingcap/tiflow/issues/7831) @[lance6716](https://github.com/lance6716)
        - テーブルごとに「update-old-value-expr」と「update-new-value-expr」のうち1つだけが設定されている場合に、フィルタルールが効かないかDMがパニックになるバグを修正 [#7774](https://github.com/pingcap/tiflow/issues/7774) @[lance6716](https://github.com/lance6716)

    + TiDB ライトニング

        - TiDBライトニングが巨大なソースデータファイルをインポートする際のメモリリーク問題を修正 [#39331](https://github.com/pingcap/tidb/issues/39331) @[dsdashun](https://github.com/dsdashun)
        - 以前の失敗したインポートによって残された不良データをTiDBライトニングの事前チェックが見つけられない問題を修正 [#39477](https://github.com/pingcap/tidb/issues/39477) @[dsdashun](https://github.com/dsdashun)