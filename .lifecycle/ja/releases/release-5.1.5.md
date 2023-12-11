---
title: TiDB 5.1.5 リリースノート
---

# TiDB 5.1.5 リリースノート

リリース日: 2022年12月28日

TiDB バージョン: 5.1.5

クイックアクセス: [クイックスタート](https://docs.pingcap.com/tidb/v5.1/quick-start-with-tidb) | [本番環境への展開](https://docs.pingcap.com/tidb/v5.1/production-deployment-using-tiup) | [インストールパッケージ](https://www.pingcap.com/download/?version=v5.1.5#version-list)

## 互換性の変更

+ PD

    - デフォルトで swagger サーバーのコンパイルを無効にする [#4932](https://github.com/tikv/pd/issues/4932)

## バグ修正

+ TiDB

    - ウィンドウ関数によって TiDB がパニックを引き起こす代わりにエラーを報告する問題を修正 [#30326](https://github.com/pingcap/tidb/issues/30326)
    - TiFlash のパーティションテーブルでダイナミックモードを有効にすると起こる間違った結果を修正 [#37254](https://github.com/pingcap/tidb/issues/37254)
    - 符号なし `BIGINT` 引数を渡すと `GREATEST` および `LEAST` の間違った結果を修正 [#30101](https://github.com/pingcap/tidb/issues/30101)
    - `left join` を使用して複数のテーブルのデータを削除する際の間違った結果を修正 [#31321](https://github.com/pingcap/tidb/issues/31321)
    - TiDB における `concat(ifnull(time(3)))` の結果が MySQL と異なる問題を修正 [#29498](https://github.com/pingcap/tidb/issues/29498)
    - `cast(integer as char) union string` を含む SQL ステートメントの結果が間違って返される問題を修正 [#29513](https://github.com/pingcap/tidb/issues/29513)
    - `LIMIT` と一緒に使用すると `INL_HASH_JOIN` がハングアップする問題を修正 [#35638](https://github.com/pingcap/tidb/issues/35638)
    - 空のデータを返すことになる場合の間違った `ANY_VALUE` の結果を修正 [#30923](https://github.com/pingcap/tidb/issues/30923)
    - 内部の Worker パニックによって引き起こされるインデックス結合の間違った結果を修正 [#31494](https://github.com/pingcap/tidb/issues/31494)
    - JSON タイプの列が `CHAR` タイプの列と結合すると SQL 操作がキャンセルされる問題を修正 [#29401](https://github.com/pingcap/tidb/issues/29401)
    - TiDB のバックグラウンド HTTP サービスが正常に終了せず、クラスターが異常な状態になる問題を修正 [#30571](https://github.com/pingcap/tidb/issues/30571)
    - 並列カラム型の変更がスキーマとデータの間の不一致を引き起こす問題を修正 [#31048](https://github.com/pingcap/tidb/issues/31048)
    - アイドル接続に対して `KILL TIDB` が直ちに効果を発揮しないバグを修正 [#24031](https://github.com/pingcap/tidb/issues/24031)
    - 任意のセッション変数を設定すると `tidb_snapshot` が機能しなくなるバグを修正 [#35515](https://github.com/pingcap/tidb/issues/35515)
    - リージョンがマージされる際にリージョンキャッシュが適切なタイミングでクリーンアップされない問題を修正 [#37141](https://github.com/pingcap/tidb/issues/37141)
    - KV クライアント内の接続配列競合によって引き起こされるパニック問題を修正 [#33773](https://github.com/pingcap/tidb/issues/33773)
    - TiDB Binlog が有効になっている場合、`ALTER SEQUENCE` ステートメントの実行が誤ったメタデータバージョンを引き起こし Drainer が終了する問題を修正 [#36276](https://github.com/pingcap/tidb/issues/36276)
    - ステートメントサマリーテーブルをクエリする際に TiDB がパニックする可能性があるバグを修正 [#35340](https://github.com/pingcap/tidb/issues/35340)
    - TiFlash で空の範囲のテーブルをスキャンする際に TiDB が誤った結果を返す問題を修正 [#33083](https://github.com/pingcap/tidb/issues/33083)
    - `HashJoinExec` を使用する際に `ERROR 1105 (HY000): close of nil channel` が返される問題を修正 [#30289](https://github.com/pingcap/tidb/issues/30289)
    - TiKV および TiFlash が論理演算をクエリする際に異なる結果を返す問題を修正 [#37258](https://github.com/pingcap/tidb/issues/37258)
    - 特定のシナリオで `EXECUTE` ステートメントが予期しないエラーをスローする可能性がある問題を修正 [#37187](https://github.com/pingcap/tidb/issues/37187)
    - `tidb_opt_agg_push_down` および `tidb_enforce_mpp` が有効になっている場合に発生するプランナーの誤った動作を修正 [#34465](https://github.com/pingcap/tidb/issues/34465)
    - `SHOW COLUMNS` ステートメントを実行する際に TiDB が coprocessor リクエストを送信する可能性があるバグを修正 [#36496](https://github.com/pingcap/tidb/issues/36496)
    - `enable-table-lock` フラグが有効になっていない場合に `lock tables` および `unlock tables` に警告を追加する [#28967](https://github.com/pingcap/tidb/issues/28967)
    - 複数の `MAXVALUE` パーティションを許可する範囲パーティションの問題を修正 [#36329](https://github.com/pingcap/tidb/issues/36329)

+ TiKV

    - `DATETIME` 値に小数点が含まれている場合の時間解析エラーを修正 [#12739](https://github.com/tikv/tikv/issues/12739)
    - レプリカリードが直線性を侵害する可能性があるバグを修正 [#12109](https://github.com/tikv/tikv/issues/12109)
    - Raftstore がビジーな場合にリージョンが重複する可能性があるバグを修正 [#13160](https://github.com/tikv/tikv/issues/13160)
    - スナップショットの適用が中止された際に TiKV がパニックする問題を修正 [#11618](https://github.com/tikv/tikv/issues/11618)
    - 2年以上実行されている場合に TiKV がパニックする可能性があるバグを修正 [#11940](https://github.com/tikv/tikv/issues/11940)
    - リージョンマージプロセスでスナップショットを介してログを追いつかせる際に TiKV がパニックする問題を修正 [#12663](https://github.com/tikv/tikv/issues/12663)
    - 空の文字列の型変換を行う際に TiKV がパニックする可能性があるバグを修正 [#12673](https://github.com/tikv/tikv/issues/12673)
    - ステールメッセージが TiKV をパニックさせる可能性があるバグを修正 [#12023](https://github.com/tikv/tikv/issues/12023)
    - ピアが分割および同時に破棄されている際にパニックが発生する可能性があるバグを修正 [#12825](https://github.com/tikv/tikv/issues/12825)
    - ターゲットピアが初期化されていない状態で置き換えられる際に TiKV がパニックする問題を修正 [#12048](https://github.com/tikv/tikv/issues/12048)
    - Follower Read を使用する際に TiKV が `invalid store ID 0` エラーを報告する問題を修正 [#12478](https://github.com/tikv/tikv/issues/12478)
    - 悲観的トランザクションで可能な重複コミットレコードを修正し、非同期コミットを有効にする [#12615](https://github.com/tikv/tikv/issues/12615)
    - 1つのピアが到達不能になった後に Raftstore が多くのメッセージをブロードキャストするのを防ぐための `unreachable_backoff` アイテムの構成をサポートする [#13054](https://github.com/tikv/tikv/issues/13054)
    - ネットワークが不安定な場合に、正常にコミットされた楽観的トランザクションが `Write Conflict` エラーを報告する問題を修正 [#34066](https://github.com/pingcap/tidb/issues/34066)
    - ダッシュボードでの `Unified Read Pool CPU` の誤った表現を修正 [#13086](https://github.com/tikv/tikv/issues/13086)

+ PD

    - PD リーダーの転送後に削除された墓石ストアが再び表示される問題を修正 [#4941](https://github.com/tikv/pd/issues/4941)
    - PD リーダーの転送後にスケジューリングが即座に開始されない問題を修正 [#4769](https://github.com/tikv/pd/issues/4769)
    - `not leader` の間違ったステータスコードを修正 [#4797](https://github.com/tikv/pd/issues/4797)
    - PD がダッシュボードプロキシリクエストを正しく処理できない問題を修正 [#5321](https://github.com/tikv/pd/issues/5321)
- 一部の特定のケースでTSOフォールバックのバグを修正する [#4884](https://github.com/tikv/pd/issues/4884)
    - 特定のシナリオでTiFlashレプリカが作成されない問題を修正する [#5401](https://github.com/tikv/pd/issues/5401)
    - メトリクスに残存ラベルが含まれるラベル分布の問題を修正する [#4825](https://github.com/tikv/pd/issues/4825)
    - 大容量のストア（例えば2T）が存在する場合、完全に割り当てられた小さいストアを検出できず、バランスオペレータが生成されない問題を修正する [#4805](https://github.com/tikv/pd/issues/4805)
    - `SchedulerMaxWaitingOperator`を`1`に設定した場合にスケジューラが機能しない問題を修正する [#4946](https://github.com/tikv/pd/issues/4946)

+ TiFlash

    - 文字列を日時にキャストする際の不正確な`microsecond`を修正する [#3556](https://github.com/pingcap/tiflash/issues/3556)
    - TLSが有効である場合に発生するパニックの問題を修正する [#4196](https://github.com/pingcap/tiflash/issues/4196)
    - 並列集計のエラーによってTiFlashがクラッシュする可能性があるバグを修正する [#5356](https://github.com/pingcap/tiflash/issues/5356)
    - `JOIN`を含むクエリがエラーが発生した場合にハングする可能性がある問題を修正する [#4195](https://github.com/pingcap/tiflash/issues/4195)
    - `OR`関数が間違った結果を返す問題を修正する [#5849](https://github.com/pingcap/tiflash/issues/5849)
    - 無効なストレージディレクトリ構成が予期しない動作を引き起こすバグを修正する [#4093](https://github.com/pingcap/tiflash/issues/4093)
    - 大量のINSERTとDELETE操作の後にデータの不整合が発生する可能性があるバグを修正する [#4956](https://github.com/pingcap/tiflash/issues/4956)
    - いくつかの領域範囲に一致しないデータがTiFlashノードに残るバグを修正する [#4414](https://github.com/pingcap/tiflash/issues/4414)
    - 高負荷のリードワークロードで列を追加した後に潜在的なクエリエラーが発生する可能性がある問題を修正する [#3967](https://github.com/pingcap/tiflash/issues/3967)
    - `commit state jump backward`エラーによって繰り返しクラッシュする可能性があるバグを修正する [#2576](https://github.com/pingcap/tiflash/issues/2576)
    - 多くの削除操作が行われたテーブルでクエリを実行する際に潜在的なエラーを修正する [#4747](https://github.com/pingcap/tiflash/issues/4747)
    - 日付形式が`''`を無効な区切り文字として認識する問題を修正する [#4036](https://github.com/pingcap/tiflash/issues/4036)
    - `DATETIME`を`DECIMAL`にキャストした際に誤った結果が返される問題を修正する [#4151](https://github.com/pingcap/tiflash/issues/4151)
    - いくつかの例外が適切に処理されていないバグを修正する [#4101](https://github.com/pingcap/tiflash/issues/4101)
    - `Prepare Merge`がraftストアのメタデータを損傷させ、TiFlashを再起動させる原因となる可能性があるバグを修正する [#3435](https://github.com/pingcap/tiflash/issues/3435)
    - MPPクエリがランダムなgRPC keepaliveタイムアウトによって失敗する可能性があるバグを修正する [#4662](https://github.com/pingcap/tiflash/issues/4662)
    - 多値式で`IN`の結果が正しくない問題を修正する [#4016](https://github.com/pingcap/tiflash/issues/4016)
    - MPPタスクが永遠にスレッドをリークする可能性があるバグを修正する [#4238](https://github.com/pingcap/tiflash/issues/4238)
    - 期限切れのデータが遅くリサイクルされる問題を修正する [#4146](https://github.com/pingcap/tiflash/issues/4146)
    - `FLOAT`を`DECIMAL`にキャストした際にオーバーフローが発生する問題を修正する [#3998](https://github.com/pingcap/tiflash/issues/3998)
    - 引数の種類がUInt8の場合に論理演算子が誤った結果を返す問題を修正する [#6127](https://github.com/pingcap/tiflash/issues/6127)
    - 空の文字列で`json_length`を呼び出した際に`index out of bounds`エラーが発生する可能性があるバグを修正する [#2705](https://github.com/pingcap/tiflash/issues/2705)
    - 特定のケースでの誤った10進数の比較結果を修正する [#4512](https://github.com/pingcap/tiflash/issues/4512)
    - `NOT NULL`の列が追加された際に`TiFlash_schema_error`が報告される問題を修正する [#4596](https://github.com/pingcap/tiflash/issues/4596)
    - 整数のデフォルト値として`0.0`が使用される際にTiFlashブートストラップが失敗する問題を修正する [#3157](https://github.com/pingcap/tiflash/issues/3157)

+ ツール

    + TiDB Binlog

        - `compressor`を`zip`に設定した場合に、DrainerがPumpに正しくリクエストを送信できない問題を修正する [#1152](https://github.com/pingcap/tidb-binlog/issues/1152)

    + バックアップとリストア (BR)

        - システムテーブルを同時にバックアップすると、テーブル名の更新に失敗するため、システムテーブルが復元できない問題を修正する [#29710](https://github.com/pingcap/tidb/issues/29710)

    + TiCDC

        - 特定の増分スキャンシナリオで発生するデータ損失を修正する [#5468](https://github.com/pingcap/tiflow/issues/5468)
        - ソーターメトリクスが存在しない問題を修正する [#5690](https://github.com/pingcap/tiflow/issues/5690)
        - DDLスキーマのバッファリング方法を最適化することで過剰なメモリ使用量を修正する [#1386](https://github.com/pingcap/tiflow/issues/1386)