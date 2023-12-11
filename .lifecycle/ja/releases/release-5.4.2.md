---
title: TiDB 5.4.2リリースノート
---

# TiDB 5.4.2リリースノート

リリース日: 2022年7月8日

TiDBバージョン: 5.4.2

> **警告:**
>
> v5.4.2の使用は推奨されません。なぜなら、このバージョンに既知のバグがあるからです。詳細は[#12934](https://github.com/tikv/tikv/issues/12934)を参照してください。このバグはv5.4.3で修正されています。[v5.4.3](/releases/release-5.4.3.md)の使用を推奨します。

## 改善点

+ TiDB

    - 可用性を向上させるために、健全でないTiKVノードにリクエストを送信しないようにする[#34906](https://github.com/pingcap/tidb/issues/34906)

+ TiKV

    - 可用性を向上させるために、各アップデートごとにTLS証明書を自動的にリロードする[#12546](https://github.com/tikv/tikv/issues/12546)
    - ヘルスチェックを改善し、利用できないRaftstoreを検出し、TiKVクライアントがリージョンキャッシュを適切なタイミングで更新できるようにする[#12398](https://github.com/tikv/tikv/issues/12398)
    - レプリカをCDCオブザーバに移譲して、レイテンシージッターを減少させる[#12111](https://github.com/tikv/tikv/issues/12111)

+ PD

    - デフォルトでswaggerサーバーをコンパイルしないようにする[#4932](https://github.com/tikv/pd/issues/4932)

+ ツール

    + TiDB Lightning

        - Scatter Regionプロセスの安定性を向上させるために、Scatter Regionをバッチモードに最適化する[#33618](https://github.com/pingcap/tidb/issues/33618)

## 不具合の修正

+ TiDB

    - バイナリプロトコルでキャッシュされた誤ったTableDualプランの問題を修正する[#34690](https://github.com/pingcap/tidb/issues/34690) [#34678](https://github.com/pingcap/tidb/issues/34678)
    - EqualAllケースでTiFlashの`firstrow`集約関数のヌルフラグが誤って推論される問題を修正する[#34584](https://github.com/pingcap/tidb/issues/34584)
    - プランナーがTiFlashに対して誤った2段階の集約プランを生成する問題を修正する[#34682](https://github.com/pingcap/tidb/issues/34682)
    - `tidb_opt_agg_push_down`および`tidb_enforce_mpp`が有効になっている場合に生じるプランナーの誤った動作を修正する[#34465](https://github.com/pingcap/tidb/issues/34465)
    - プランキャッシュが削除されたときに誤って使用されるメモリ使用量の値を修正する[#34613](https://github.com/pingcap/tidb/issues/34613)
    - `LOAD DATA`ステートメントにおいて、列リストが機能しない問題を修正する[#35198](https://github.com/pingcap/tidb/issues/35198)
    - 悲観的なトランザクションで`WriteConflict`エラーが報告されないように修正する[#11612](https://github.com/tikv/tikv/issues/11612)
    - リージョンエラーやネットワークの問題が発生したときに、事前書き込みリクエストが冪等性でない場合がある問題を修正する[#34875](https://github.com/pingcap/tidb/issues/34875)
    - 非同期コミットトランザクションがアトミック性を満たさない場合がある問題を修正する[#33641](https://github.com/pingcap/tidb/issues/33641)
    - ネットワークの接続問題が発生した場合、TiDBが切断されたセッションによって保持されたリソースを常に正しく解放しないことがある問題を修正する。これにより、オープントランザクションがロールバックされ、その他の関連するリソースが解放されるようになりました。[#34722](https://github.com/pingcap/tidb/issues/34722)
    - CTEを使用している場合に、TiDBがビューをクエリする際に`references invalid table`エラーが誤って報告される問題を修正する[#33965](https://github.com/pingcap/tidb/issues/33965)
    - `fatal error: concurrent map read and map write`エラーによってパニックが発生する問題を修正する[#35340](https://github.com/pingcap/tidb/issues/35340)

+ TiKV

    - `max_sample_size`が`0`に設定されている場合に統計情報を解析する際にパニックが発生する問題を修正する[#11192](https://github.com/tikv/tikv/issues/11192)
    - TiKVを終了する際に誤ってTiKVパニックを報告する可能性がある問題を修正する[#12231](https://github.com/tikv/tikv/issues/12231)
    - リージョンマージプロセスでスナップショットによってログを追いつかせるときにパニックが発生する可能性がある問題を修正する[#12663](https://github.com/tikv/tikv/issues/12663)
    - ピアが分割および同時に破壊されるときにパニックが発生する可能性がある問題を修正する[#12825](https://github.com/tikv/tikv/issues/12825)
    - PDクライアントがエラーに遭遇したときに頻繁な再接続が発生する問題を修正する[#12345](https://github.com/tikv/tikv/issues/12345)
    - `DATETIME`値が分数および`Z`を含む場合に発生する時刻解析エラーの問題を修正する[#12739](https://github.com/tikv/tikv/issues/12739)
    - 空の文字列に対する型変換を行う際にTiKVがパニックする問題を修正する[#12673](https://github.com/tikv/tikv/issues/12673)
    - 悲観的トランザクションにおいて非同期コミット時に重複したコミットレコードが発生する可能性がある問題を修正する[#12615](https://github.com/tikv/tikv/issues/12615)
    - Follower Readを使用する際にTiKVが`invalid store ID 0`エラーを報告する問題を修正する[#12478](https://github.com/tikv/tikv/issues/12478)
    - ピアの破壊とバッチリージョンの分割の競合によってTiKVがパニックする問題を修正する[#12368](https://github.com/tikv/tikv/issues/12368)
    - 不正な文字列マッチによってtikv-ctlが正しくない結果を返す問題を修正する[#12329](https://github.com/tikv/tikv/issues/12329)
    - AUFS上でTiKVを起動できない問題を修正する[#12543](https://github.com/tikv/tikv/issues/12543)

+ PD

    - `not leader`の間違ったステータスコードを修正する[#4797](https://github.com/tikv/pd/issues/4797)
    - ホットリージョンにリーダーがいない場合にPDがパニックする問題を修正する[#5005](https://github.com/tikv/pd/issues/5005)
    - PDリーダー転送後にスケジューリングが即座に開始できない問題を修正する[#4769](https://github.com/tikv/pd/issues/4769)
    - 一部の特殊なケースでTSOフォールバックのバグを修正する[#4884](https://github.com/tikv/pd/issues/4884)

+ TiFlash

    - クラスターインデックスを持つテーブルの列を削除した後にTiFlashがクラッシュする問題を修正する[#5154](https://github.com/pingcap/tiflash/issues/5154)
    - 大量のINSERTおよびDELETE操作の後にデータの不整合性が発生する可能性がある問題を修正する[#4956](https://github.com/pingcap/tiflash/issues/4956)
    - 一部のケースで誤った10進数比較結果が生成される問題を修正する[#4512](https://github.com/pingcap/tiflash/issues/4512)

+ ツール

    + バックアップ＆リストア(BR)

        - RawKVモードでBRが`ErrRestoreTableIDMismatch`を報告するバグを修正する[#35279](https://github.com/pingcap/tidb/issues/35279)
        - ファイル保存時にエラーが発生した場合にBRがリトライしないバグを修正する[#34865](https://github.com/pingcap/tidb/issues/34865)
        - BRの実行中にパニックが発生する問題を修正する[#34956](https://github.com/pingcap/tidb/issues/34956)
        - BRがS3内部エラーを処理できない問題を修正する[#34350](https://github.com/pingcap/tidb/issues/34350)
        - リストア操作で回復不能なエラーが発生した場合にBRがスタックする問題を修正する[#33200](https://github.com/pingcap/tidb/issues/33200)

    + TiCDC

        - 特殊な増分スキャンシナリオで発生するデータ損失を修正する[#5468](https://github.com/pingcap/tiflow/issues/5468)
        - リドゥログマネージャーがログを書き込む前にログをフラッシュするバグを修正する[#5486](https://github.com/pingcap/tiflow/issues/5486)
        - リゾルブされたtsがredoライターによってメンテナンスされていないテーブルがある場合に移動しすぎる問題を修正する[#5486](https://github.com/pingcap/tiflow/issues/5486)
        - ファイル名の競合がデータ損失の原因になる可能性がある問題を修正する[#5486](https://github.com/pingcap/tiflow/issues/5486)
        - リージョンリーダーが欠落し、リトライが上限を超えるとレプリケーションが中断する問題を修正する[#5230](https://github.com/pingcap/tiflow/issues/5230)
        - MySQL Sinkが誤ったcheckpointTsを保存する可能性があるバグを修正する[#5107](https://github.com/pingcap/tiflow/issues/5107)
```
        - HTTPサーバーでgoroutineのリークを引き起こす可能性があるバグを修正しました[#5303](https://github.com/pingcap/tiflow/issues/5303)
        - メタリージョンの変更が遅延の増加につながる問題を修正しました[#4756](https://github.com/pingcap/tiflow/issues/4756) [#4762](https://github.com/pingcap/tiflow/issues/4762)

    + TiDBデータマイグレーション（DM）

        - タスクが自動的に再開されると、DMがより多くのディスクスペースを占有してしまう問題を修正しました[#5344](https://github.com/pingcap/tiflow/issues/5344)
        - `case-sensitive: true`が設定されていない場合に、大文字のテーブルをレプリケートできない問題を修正しました[#5255](https://github.com/pingcap/tiflow/issues/5255)
```