---
title: TiDB 4.0.10リリースノート
---

# TiDB 4.0.10リリースノート

リリース日: 2021年1月15日

TiDBバージョン: 4.0.10

## 新機能

+ PD

    - `enable-redact-log`構成項目を追加し、ログからユーザーデータをマスキングする[#3266](https://github.com/pingcap/pd/pull/3266)

+ TiFlash

    - `security.redact_info_log`構成項目を追加し、ログからユーザーデータをマスキングする

## 改良点

+ TiDB

    - `txn-entry-size-limit`を使用してトランザクションのキー値エントリのサイズ制限を設定可能にする[#21843](https://github.com/pingcap/tidb/pull/21843)

+ PD

    - `store-state-filter`メトリクスを最適化し、より多くの情報を表示する[#3100](https://github.com/tikv/pd/pull/3100)
    - `go.etcd.io/bbolt`依存関係をv1.3.5にアップグレードする[#3331](https://github.com/tikv/pd/pull/3331)

+ ツール

    + TiCDC

        - `maxwell`プロトコルの古い値機能を有効にする[#1144](https://github.com/pingcap/tiflow/pull/1144)
        - 統一されたソータ機能をデフォルトで有効にする[#1230](https://github.com/pingcap/tiflow/pull/1230)

    + Dumpling

        - ダンプ中の認識されていない引数の確認と現在の進行状況の表示をサポートする[#228](https://github.com/pingcap/dumpling/pull/228)

    + TiDB Lightning

        - S3から読み取り時に発生するエラーのリトライをサポートする[#533](https://github.com/pingcap/tidb-lightning/pull/533)

## バグ修正

+ TiDB

    - バッチクライアントタイムアウトを引き起こす可能性のある並行バグを修正する[#22336](https://github.com/pingcap/tidb/pull/22336)
    - 並行ベースラインキャプチャによって重複したバインディングが発生する問題を修正する[#22295](https://github.com/pingcap/tidb/pull/22295)
    - ログレベルが'debug'に設定されている場合にSQLステートメントにバインドされているベースラインキャプチャが動作するよう修正する[#22293](https://github.com/pingcap/tidb/pull/22293)
    - リージョンマージが発生した場合に正しくGCロックを解放する[#22267](https://github.com/pingcap/tidb/pull/22267)
    - `datetime`タイプのユーザ変数に対して正しい値を返す[#22143](https://github.com/pingcap/tidb/pull/22143)
    - 複数のテーブルフィルタがある場合にインデックスマージの使用する問題を修正する[#22124](https://github.com/pingcap/tidb/pull/22124)
    - `prepare`プランキャッシュによってTiFlashで発生する`wrong precision`問題を修正する[#21960](https://github.com/pingcap/tidb/pull/21960)
    - スキーマ変更によって引き起こされる不正確な結果の問題を修正する[#21596](https://github.com/pingcap/tidb/pull/21596)
    - `ALTER TABLE`で不必要な列フラグ変更を回避する[#21474](https://github.com/pingcap/tidb/pull/21474)
    - オプティマイザヒントで使用されるクエリブロックのテーブルエイリアスに対してデータベース名を設定する[#21380](https://github.com/pingcap/tidb/pull/21380)
    - `IndexHashJoin`と`IndexMergeJoin`用の適切なオプティマイザヒントを生成する[#21020](https://github.com/pingcap/tidb/pull/21020)

+ TiKV

    - 準備とピア間の間違ったマッピングを修正する[#9409](https://github.com/tikv/tikv/pull/9409)
    - `security.redact-info-log`が`true`に設定されているときに一部のログがマスキングされない問題を修正する[#9314](https://github.com/tikv/tikv/pull/9314)

+ PD

    - ID割り当てがモノトニックでない問題を修正する[#3308](https://github.com/tikv/pd/pull/3308) [#3323](https://github.com/tikv/pd/pull/3323)
    - 特定の場合にPDクライアントがブロックされる可能性がある問題を修正する[#3285](https://github.com/pingcap/pd/pull/3285)

+ TiFlash

    - TiFlashが古いバージョンのTiDBスキーマを処理できない問題を修正する
    - RedHatシステム上の`cpu_time`の不正な処理によってTiFlashが起動しない問題を修正する
    - `path_realtime_mode`が`true`に設定されているときにTiFlashが開始しない問題を修正する
    - 3つのパラメータを使用して`substr`関数を呼び出すと不正確な結果が発生する問題を修正する
    - 変更が損失なしである場合でも、TiFlashが`Enum`タイプの変更をサポートしない問題を修正する

+ ツール

    + TiCDC

        - `maxwell`プロトコルの問題を修正する。`base64`データ出力およびTSOをUnixタイムスタンプに出力する問題を含む[#1173](https://github.com/pingcap/tiflow/pull/1173)
        - 古いメタデータが新しく作成したchangefeedを異常にする可能性のあるバグを修正する[#1184](https://github.com/pingcap/tiflow/pull/1184)
        - 閉じられた通知機能でレシーバを作成する問題を修正する[#1199](https://github.com/pingcap/tiflow/pull/1199)
        - TiCDCオーナーがetcdウォッチクライアントで過剰なメモリを消費する可能性のあるバグを修正する[#1227](https://github.com/pingcap/tiflow/pull/1227)
        - `max-batch-size`が効果を発揮しない問題を修正する[#1253](https://github.com/pingcap/tiflow/pull/1253)
        - キャプチャ情報が構築される前に古いタスクのクリーンアップを行う問題を修正する[#1280](https://github.com/pingcap/tiflow/pull/1280)
        - `rollback`がMySQLシンクで呼ばれていないためにdb connの再利用がブロックされる問題を修正する[#1285](https://github.com/pingcap/tiflow/pull/1285)

    + Dumpling

        - デフォルトの動作を設定することで、TiDBのメモリ枠を超えることを回避する[#233](https://github.com/pingcap/dumpling/pull/233)

    + バックアップ＆リストア（BR）

        - BR v4.0.8でバックアップされたファイルをBR v4.0.9で復元できない問題を修正する[#688](https://github.com/pingcap/br/pull/688)
        - GCSストレージURLにプレフィックスがない場合にBRがパニックを起こす問題を修正する[#673](https://github.com/pingcap/br/pull/673)
        - BRのOOMを回避するためにバックアップ統計をデフォルトで無効にする[#693](https://github.com/pingcap/br/pull/693)

    + TiDB Binlog

        - `AMEND TRANSACTION`機能が有効な場合、Drainerが生成するSQLステートメントに適切なスキーマバージョンを選択できない問題を修正する[#1033](https://github.com/pingcap/tidb-binlog/pull/1033)

    + TiDB Lightning

        - リージョンが誤ってエンコードされたために分割されないバグを修正する[#531](https://github.com/pingcap/tidb-lightning/pull/531)
        - 複数のテーブルが作成される場合に`CREATE TABLE`の失敗が失われる問題を修正する[#530](https://github.com/pingcap/tidb-lightning/pull/530)
        - TiDBバックエンドを使用する際に`column count mismatch`の問題を修正する[#535](https://github.com/pingcap/tidb-lightning/pull/535)