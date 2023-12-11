---
title: TiDB 4.0.12リリースノート
---

# TiDB 4.0.12リリースノート

リリース日：2021年4月2日

TiDB バージョン：4.0.12

## 新機能

+ TiFlash

    - `tiflash replica`のオンラインローリングアップデートの状態をチェックするためのツールを追加

## 改善点

+ TiDB

    - `batch cop`モードの`EXPLAIN`文の出力情報を改善[#23164](https://github.com/pingcap/tidb/pull/23164)
    - `EXPLAIN`文の出力でストレージレイヤーにプッシュできない式に対する警告情報を追加[#23020](https://github.com/pingcap/tidb/pull/23020)
    - DDLパッケージコードの一部を`Execute`/`ExecRestricted`からセーフAPIに移行(2) [#22935](https://github.com/pingcap/tidb/pull/22935)
    - DDLパッケージコードの一部を`Execute`/`ExecRestricted`からセーフAPIに移行(1) [#22929](https://github.com/pingcap/tidb/pull/22929)
    - 遅いログに`optimization-time`と`wait-TS-time`を追加[#22918](https://github.com/pingcap/tidb/pull/22918)
    - `infoschema.partitions`テーブルから`partition_id`を問い合わせるサポートを追加[#22489](https://github.com/pingcap/tidb/pull/22489)
    - SQL文の実行計画がヒントと一致するかどうかをユーザーに知らせるために`last_plan_from_binding`を追加[#21430](https://github.com/pingcap/tidb/pull/21430)
    - `pre-split`オプションなしで切り詰められたテーブルを散らす[#22872](https://github.com/pingcap/tidb/pull/22872)
    - `str_to_date`式のための3つのフォーマット指定子を追加[#22812](https://github.com/pingcap/tidb/pull/22812)
    - メトリクス監視で`PREPARE`実行の失敗を`Failed Query OPM`として記録[#22672](https://github.com/pingcap/tidb/pull/22672)
    - `tidb_snapshot`が設定されている場合に`PREPARE`実行のエラーを報告しないようにする[#22641](https://github.com/pingcap/tidb/pull/22641)

+ TiKV

    - 短期間に大量の再接続を防ぐ[#9879](https://github.com/tikv/tikv/pull/9879)
    - 多数の墓石があるシナリオでの書き込み操作とバッチ取得を最適化[#9729](https://github.com/tikv/tikv/pull/9729)
    - `leader-transfer-max-log-lag`のデフォルト値を`128`に変更してリーダー転送の成功率を上げる[#9605](https://github.com/tikv/tikv/pull/9605)

+ PD

    - `pending-peers`または`down-peers`が変更されたときにのみリージョンキャッシュを更新することで、ハートビートの更新圧力を軽減[#3471](https://github.com/pingcap/pd/pull/3471)
    - `split-cache`内のリージョンがマージの対象にならないようにすることで、マージの圧力を減らす[#3459](https://github.com/pingcap/pd/pull/3459)

+ TiFlash

    - 構成ファイルを最適化し、不要なアイテムを削除
    - TiFlashバイナリファイルのサイズを縮小
    - メモリ使用量を減らすために適応的な積極的なGC戦略を使用

+ ツール

    + TiCDC

        - `start-ts`または`checkpoint-ts`の1日前のタイムスタンプでchagefeedを作成または再開する際に二重確認を追加[#1497](https://github.com/pingcap/tiflow/pull/1497)
        - 古い値機能のGrafanaパネルを追加[#1571](https://github.com/pingcap/tiflow/pull/1571)

    + バックアップ＆リストア（BR）

        - `HTTP_PROXY`と`HTTPS_PROXY`の環境変数をログに記録[#827](https://github.com/pingcap/br/pull/827)
        - 多くのテーブルがある場合のバックアップ性能を向上[#745](https://github.com/pingcap/br/pull/745)
        - サービスセーフポイントの確認に失敗した場合はエラーを報告[#826](https://github.com/pingcap/br/pull/826)
        - `backupmeta`に`cluster_version`と`br_version`情報を追加[#803](https://github.com/pingcap/br/pull/803)
        - 外部ストレージエラーのためのリトライを追加してバックアップの成功率を上げる[#851](https://github.com/pingcap/br/pull/851)
        - バックアップ時のメモリ使用量を減らす[#886](https://github.com/pingcap/br/pull/886)

    + TiDB Lightning

        - 予期せぬエラーを防ぐためにTiDBクラスターバージョンをチェックするようにTiDB Lightningを実行する[#787](https://github.com/pingcap/br/pull/787)
        - `cancel`エラーに遭遇した場合、TiDB Lightningを速やかに失敗させる[#867](https://github.com/pingcap/br/pull/867)
        - メモリ使用量とパフォーマンスのバランスを取るために`batch split region`を並列で実行するための構成アイテム`engine-mem-cache-size`および`local-writer-mem-cache-size`を追加[#866](https://github.com/pingcap/br/pull/866)
        - TiDB LightningのLocal-backendでインポート速度を上げるために`batch split region`を並列で実行する[#868](https://github.com/pingcap/br/pull/868)
        - S3ストレージからデータをインポートする際、TiDB Lightningは`PREPARE`実行に`s3:ListBucket`権限を必要としなくなる[#919](https://github.com/pingcap/br/pull/919)
        - チェックポイントから再開する際にTiDB Lightningが元のエンジンを引き続き使用する[#924](https://github.com/pingcap/br/pull/924)

## バグ修正

+ TiDB

    - セッション変数が16進数リテラルの場合に`get`変数式が誤った値を返す問題を修正[#23372](https://github.com/pingcap/tidb/pull/23372)
    - `Enum`または`Set`タイプの高速な実行計画を作成する際に誤った照合が使用される問題を修正[#23292](https://github.com/pingcap/tidb/pull/23292)
    - `is-null`と併用した場合に`nullif`式の誤った結果が得られる可能性がある問題を修正[#23279](https://github.com/pingcap/tidb/pull/23279)
    - 自動解析が時間範囲外でトリガーされる問題を修正[#23219](https://github.com/pingcap/tidb/pull/23219)
    - `point get`プランで`CAST`関数がエラーを無視する可能性がある問題を修正[#23211](https://github.com/pingcap/tidb/pull/23211)
    - `CurrentDB`が空の場合にSPMが効果を発揮しないバグを修正[#23209](https://github.com/pingcap/tidb/pull/23209)
    - IndexMergeプランのための誤ったテーブルフィルターの問題を修正[#23165](https://github.com/pingcap/tidb/pull/23165)
    - `NULL`定数の戻り型に予期しない`NotNullFlag`が含まれる問題を修正[#23135](https://github.com/pingcap/tidb/pull/23135)
    - テキストタイプで照合が適切に処理されない可能性があるバグを修正[#23092](https://github.com/pingcap/tidb/pull/23092)
    - 範囲パーティションが`IN`式を誤って処理する問題を修正[#23074](https://github.com/pingcap/tidb/pull/23074)
    - TiKVストアを墓石としてマークした後、同じIPアドレスとポートの異なるStoreIDで新しいTiKVストアを起動すると`StoreNotMatch`エラーが返り続けるバグを修正[#23071](https://github.com/pingcap/tidb/pull/23071)
    - `INT`型を`NULL`と比較した場合に`YEAR`と比較するときに`INT`型を調整しないように修正[#22844](https://github.com/pingcap/tidb/pull/22844)
    - `auto_random`列を持つテーブルでデータをロードする際の切断の問題を修正[#22736](https://github.com/pingcap/tidb/pull/22736)
    - DDL操作がパニックを起こした場合のDDLハングオーバーの問題を修正[#23297](https://github.com/pingcap/tidb/pull/23297)
    - `YEAR`列を`NULL`と比較したときのインデックススキャンの誤ったキーレンジの問題を修正[#23104](https://github.com/pingcap/tidb/pull/23104)
    - 正常に作成されたビューを使用できなくなる問題を修正[#23083](https://github.com/pingcap/tidb/pull/23083)

+ TiKV
- `IN`式が符号なし/符号付き整数を適切に処理しない問題を修正する[#9850](https://github.com/tikv/tikv/pull/9850) 
- インジェスト操作が再入不可である問題を修正する[#9779](https://github.com/tikv/tikv/pull/9779) 
- TiKVコプロセッサでJSONを文字列に変換する際にスペースが抜けている問題を修正する[#9666](https://github.com/tikv/tikv/pull/9666) 

+ PD
    - ストアがラベルを欠いている場合に孤立レベルが間違っているバグを修正する[#3474](https://github.com/pingcap/pd/pull/3474) 

+ TiFlash 
    - デフォルト値の`binary`型列に先頭または末尾にゼロバイトが含まれている場合に実行結果が正しくない問題を修正する
    - データベースの名前に特殊文字が含まれるとTiFlashがスキーマを同期できないバグを修正する
    - decimal値を使用した`IN`式の処理時に正しくない結果が出てしまう問題を修正する
    - Grafanaに表示されるオープンされたファイルのカウントのメトリックが高いバグを修正する
    - TiFlashが`Timestamp`リテラルをサポートしていないバグを修正する
    - `FROM_UNIXTIME`式の処理中に応答しない可能性がある問題を修正する
    - 文字列を整数にキャストする際に正しくない結果が出てしまう問題を修正する
    - `like`関数が正しくない結果を返す可能性があるバグを修正する

+ Tools 
    + TiCDC
        - `resolved ts`イベントの順序がおかしい問題を修正する[#1464](https://github.com/pingcap/tiflow/pull/1464) 
        - ネットワークの問題による誤ったテーブルスケジューリングによるデータ損失の問題を修正する[#1508](https://github.com/pingcap/tiflow/pull/1508) 
        - プロセッサの停止後にリソースが適時に解放されないバグを修正する[#1547](https://github.com/pingcap/tiflow/pull/1547) 
        - トランザクションカウンターが正しく更新されないバグを修正する、これによりデータベース接続リークが発生する可能性がある[#1524](https://github.com/pingcap/tiflow/pull/1524) 
        - PDにジッターがある場合に複数のオーナーが共存し、これがテーブルの不足を引き起こす可能性がある問題を修正する[#1540](https://github.com/pingcap/tiflow/pull/1540) 

    + Backup & Restore (BR)
        - S3ストレージの`WalkDir`が対象パスがバケット名の場合に`nil`を返すバグを修正する[#733](https://github.com/pingcap/br/pull/733) 
        - `status`ポートがTLSで提供されないバグを修正する[#839](https://github.com/pingcap/br/pull/839) 

    + TiDB Lightning
        - TiKV Importerがすでに存在するファイルを無視する可能性があるエラーを修正する[#848](https://github.com/pingcap/br/pull/848) 
        - TiDB Lightningが誤ったタイムスタンプを使用し、誤ったデータを読み込む可能性があるバグを修正する[#850](https://github.com/pingcap/br/pull/850) 
        - TiDB Lightningの予期しない終了が破損したチェックポイントファイルを引き起こす可能性があるバグを修正する[#889](https://github.com/pingcap/br/pull/889) 
        - `cancel`エラーが無視されることにより発生する可能性のあるデータエラーの問題を修正する[#874](https://github.com/pingcap/br/pull/874)