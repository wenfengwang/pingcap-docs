# TiDB 5.3.2 リリースノート

リリース日: 2022年6月29日

TiDB バージョン: 5.3.2

> **警告:**
> 
> v5.3.2の使用はお勧めしません。このバージョンには既知のバグがあります。詳細については、[#12934](https://github.com/tikv/tikv/issues/12934) を参照してください。このバグはv5.3.3で修正されています。[v5.3.3](/releases/release-5.3.3.md) の使用をお勧めします。

## 互換性の変更

+ TiDB

    - auto ID が範囲外の場合に `REPLACE` ステートメントが他の行を誤って変更する問題を修正 [#29483](https://github.com/pingcap/tidb/issues/29483)

+ PD

    - デフォルトで Swagger サーバのコンパイルを無効にする [#4932](https://github.com/tikv/pd/issues/4932)

## 改善点

+ TiKV

    - Raft クライアントによるシステムコールの削減と CPU の効率化のための改善 [#11309](https://github.com/tikv/tikv/issues/11309)
    - 利用不可な Raftstore を検出するためのヘルスチェックの改善により、TiKV クライアントがリージョンキャッシュをタイムリーに更新できるようにする [#12398](https://github.com/tikv/tikv/issues/12398)
    - レイテンシのジッタを減らすために CDC オブザーバにリーダーシップを転送する改善 [#12111](https://github.com/tikv/tikv/issues/12111)
    - Raft ログのガベージコレクションモジュールのパフォーマンス問題を特定するために、より多くのメトリクスを追加する [#11374](https://github.com/tikv/tikv/issues/11374)

+ ツール

    + TiDB データ移行（DM）

        - DM ワーカーの作業ディレクトリを `/tmp` ではなく使用して内部ファイルを書き込み、タスクが停止された後にディレクトリをクリーニングする機能をサポート [#4107](https://github.com/pingcap/tiflow/issues/4107)

    + TiDB Lightning

        - Scatter Region をバッチモードに最適化して、Scatter Region プロセスの安定性を向上させる [#33618](https://github.com/pingcap/tidb/issues/33618)

## バグ修正

+ TiDB

    - Amazon S3 が圧縮されたデータのサイズを正しく計算できない問題を修正 [#30534](https://github.com/pingcap/tidb/issues/30534)
    - 楽観的トランザクションモードでポテンシャルなデータインデックスの不整合の問題を修正 [#30410](https://github.com/pingcap/tidb/issues/30410)
    - JSON 型の列が `CHAR` 型の列に結合する場合に SQL 操作がキャンセルされる問題を修正 [#29401](https://github.com/pingcap/tidb/issues/29401)
    - ネットワークの接続問題が発生した場合、以前は TiDB が切断されたセッションが保持するリソースを正しく解放しないことがありました。この問題は修正され、オープンされたトランザクションをロールバックし、その他関連するリソースを解放できるようになりました [#34722](https://github.com/pingcap/tidb/issues/34722)
    - TiDB ビンログが有効になっている場合に重複した値を挿入すると `data and columnID count not match` エラーが発生する問題を修正 [#33608](https://github.com/pingcap/tidb/issues/33608)
    - Plan Cache が RC 分離レベルで開始されている場合にクエリ結果が誤っている可能性がある問題を修正 [#34447](https://github.com/pingcap/tidb/issues/34447)
    - MySQL バイナリプロトコルでテーブルスキーマが変更された後にプリペアドステートメントを実行するとセッションパニックが発生する問題を修正 [#33509](https://github.com/pingcap/tidb/issues/33509)
    - 新しいパーティションが追加されたときにテーブル属性がインデックス化されない問題と、パーティションが変更されたときにテーブル範囲情報が更新されない問題を修正 [#33929](https://github.com/pingcap/tidb/issues/33929)
    - `INFORMATION_SCHEMA.CLUSTER_SLOW_QUERY` テーブルがクエリされると TiDB サーバーがメモリ不足になる可能性がある問題を修正。この問題は Grafana ダッシュボードで遅いクエリを確認する場合に発生する可能性があります [#33893](https://github.com/pingcap/tidb/issues/33893)
    - クラスターの PD ノードが置換された後、一部の DDL ステートメントが一定期間スタックする可能性がある問題を修正 [#33908](https://github.com/pingcap/tidb/issues/33908)
    - v4.0 からアップグレードされたクラスターで `all` 権限を付与する際に失敗する可能性がある問題を修正 [#33588](https://github.com/pingcap/tidb/issues/33588)
    - `left join` を使用して複数のテーブルのデータを削除すると誤った結果が得られる可能性がある問題を修正 [#31321](https://github.com/pingcap/tidb/issues/31321)
    - TiDB が TiFlash に重複したタスクを送信する可能性があるバグを修正 [#32814](https://github.com/pingcap/tidb/issues/32814)
    - バックグラウンド HTTP サービスが正常に終了せず、クラスターが異常な状態になる場合がある問題を修正 [#30571](https://github.com/pingcap/tidb/issues/30571)
    - `fatal error: concurrent map read and map write` エラーによってパニックが発生する問題を修正 [#35340](https://github.com/pingcap/tidb/issues/35340)

+ TiKV

    - PD クライアントがエラーに遭遇したときに頻繁に再接続する問題を修正 [#12345](https://github.com/tikv/tikv/issues/12345)
    - `DATETIME` 値が小数点および `Z` を含む場合の時刻の解析エラーを修正 [#12739](https://github.com/tikv/tikv/issues/12739)
    - 空の文字列の型変換を行う際に TiKV がパニックする問題を修正 [#12673](https://github.com/tikv/tikv/issues/12673)
    - 非同期コミットが有効になっているときの悲観的トランザクションで可能な重複コミットレコードの修正 [#12615](https://github.com/tikv/tikv/issues/12615)
    - Follower Read を使用したときに TiKV が `invalid store ID 0` エラーを報告するバグを修正 [#12478](https://github.com/tikv/tikv/issues/12478)
    - ピアの破棄とリージョンのバッチ分割の競合によって TiKV がパニックを起こす問題を修正 [#12368](https://github.com/tikv/tikv/issues/12368)
    - ネットワークが不安定な場合に正常にコミットされた楽観的トランザクションが `Write Conflict` エラーを報告する問題を修正 [#34066](https://github.com/pingcap/tidb/issues/34066)
    - 無効なターゲットリージョンをマージすると TiKV がパニックする問題を修正 [#12232](https://github.com/tikv/tikv/issues/12232)
    - 陳腐なメッセージが TiKV をパニックさせる原因となるバグを修正 [#12023](https://github.com/tikv/tikv/issues/12023)
    - メモリメトリクスのオーバーフローによる一時的なパケット損失とメモリの枯渇に起因する問題を修正 [#12160](https://github.com/tikv/tikv/issues/12160)
    - Ubuntu 18.04 で TiKV がプロファイリングを実行する際に発生する可能性のあるパニック問題を修正 [#9765](https://github.com/tikv/tikv/issues/9765)
    - 間違った文字列一致によって tikv-ctl が誤った結果を返すバグを修正 [#12329](https://github.com/tikv/tikv/issues/12329)
    - レプリカ読み取りが直線性を満たさない可能性のあるバグを修正 [#12109](https://github.com/tikv/tikv/issues/12109)
    - マージされるリージョンに初期化されていないピアが置き換えられると TiKV がパニックする問題を修正 [#12048](https://github.com/tikv/tikv/issues/12048)
    - 2年以上実行されている場合に TiKV がパニックする可能性があるバグを修正 [#11940](https://github.com/tikv/tikv/issues/11940)

+ PD

    - ホットリージョンにリーダーが存在しない場合に PD がパニックする問題を修正 [#5005](https://github.com/tikv/pd/issues/5005)
    - PD リーダーのトランスファー直後にスケジューリングがすぐに開始できない問題を修正 [#4769](https://github.com/tikv/pd/issues/4769)
    - PD リーダーのトランスファー後に削除されたトゥームストーンストアが再度現れる問題を修正​​ [#4941](https://github.com/tikv/pd/issues/4941)
    - 一部の隅ケースでの TSO のフォールバックのバグを修正 [#4884](https://github.com/tikv/pd/issues/4884)
    - 大容量のストア（例: 2T）が存在する場合、完全に割り当てられた小容量のストアを検出できず、バランスオペレータが生成されない問題を修正 [#4805](https://github.com/tikv/pd/issues/4805)
    - `SchedulerMaxWaitingOperator` が `1` に設定されているときにスケジューラが機能しない問題を修正 [#4946](https://github.com/tikv/pd/issues/4946)
    - ラベル分布がメトリクスに残留ラベルを持っている問題を修正 [#4825](https://github.com/tikv/pd/issues/4825)

+ TiFlash
- 無効なストレージディレクトリ構成が予期しない動作を引き起こすバグを修正する[#4093](https://github.com/pingcap/tiflash/issues/4093)
- `NOT NULL`列を追加したときに報告される`TiFlash_schema_error`を修正する[#4596](https://github.com/pingcap/tiflash/issues/4596)
- `commit state jump backward`エラーによる繰り返しクラッシュを修正する[#2576](https://github.com/pingcap/tiflash/issues/2576)
- 多くのINSERTおよびDELETE操作後にデータの整合性が損なわれる可能性があるバグを修正する[#4956](https://github.com/pingcap/tiflash/issues/4956)
- ローカルトンネルが有効になっている場合、MPPクエリのキャンセルがタスクが永遠に停滞する原因となるバグを修正する[#4229](https://github.com/pingcap/tiflash/issues/4229)
- TiFlashがリモート読み取りを行う際にTiFlashのバージョンの整合性に関連した誤った報告を修正する[#3713](https://github.com/pingcap/tiflash/issues/3713)
- ランダムなgRPC keepaliveタイムアウトによってMPPクエリが失敗する可能性があるバグを修正する[#4662](https://github.com/pingcap/tiflash/issues/4662)
- 交換受信機でリトライがある場合にMPPクエリが永遠に停滞する可能性があるバグを修正する[#3444](https://github.com/pingcap/tiflash/issues/3444)
- `DATETIME`を`DECIMAL`にキャストする際に発生する誤った結果を修正する[#4151](https://github.com/pingcap/tiflash/issues/4151)
- `FLOAT`を`DECIMAL`にキャストする際に発生するオーバーフローを修正する[#3998](https://github.com/pingcap/tiflash/issues/3998)
- 空の文字列で`json_length`を呼び出した場合に発生する`index out of bounds`エラーを修正する[#2705](https://github.com/pingcap/tiflash/issues/2705)
- 特定の角のケースで誤ったDECIMAL比較の結果を修正する[#4512](https://github.com/pingcap/tiflash/issues/4512)
- ジョイン構築ステージでクエリが失敗した場合にMPPクエリが永遠に停滞する可能性があるバグを修正する[#4195](https://github.com/pingcap/tiflash/issues/4195)
- クエリに`where <string>`句が含まれる場合に誤った結果が発生する可能性がある問題を修正する[#3447](https://github.com/pingcap/tiflash/issues/3447)
- TiFlashとTiDBまたはTiKVで`CastStringAsReal`の振る舞いが一貫していない問題を修正する[#3475](https://github.com/pingcap/tiflash/issues/3475)
- 文字列を日時にキャストする際に`microsecond`が不正な値になる問題を修正する[#3556](https://github.com/pingcap/tiflash/issues/3556)
- 多くの削除操作を行ったテーブルのクエリ時に潜在的なエラーを修正する[#4747](https://github.com/pingcap/tiflash/issues/4747)
- TiFlashがランダムに多くの"Keepalive watchdog fired"エラーを報告するバグを修正する[#4192](https://github.com/pingcap/tiflash/issues/4192)
- TiFlashノードに残ったいかなるリージョン範囲にも一致しないデータを修正するバグを修正する[#4414](https://github.com/pingcap/tiflash/issues/4414)
- MPPタスクが永遠にスレッドをリークするバグを修正する[#4238](https://github.com/pingcap/tiflash/issues/4238)
- GC後に空のセグメントをマージできないバグを修正する[#4511](https://github.com/pingcap/tiflash/issues/4511)
- TLSが有効になっている際に発生するパニックの問題を修正する[#4196](https://github.com/pingcap/tiflash/issues/4196)
- 期限切れのデータが遅くリサイクルされる問題を修正する[#4146](https://github.com/pingcap/tiflash/issues/4146)
- 無効なストレージディレクトリ構成が予期しない動作を引き起こすバグを修正する[#4093](https://github.com/pingcap/tiflash/issues/4093)
- 一部の例外が適切に処理されないバグを修正する[#4101](https://github.com/pingcap/tiflash/issues/4101)
- 大量の読み取りワークロード下で列を追加した後にクエリエラーが発生する可能性があるバグを修正する[#3967](https://github.com/pingcap/tiflash/issues/3967)
- `STR_TO_DATE()`関数がミリ秒を解析する際に先頭のゼロを正しく扱わないバグを修正する[#3557](https://github.com/pingcap/tiflash/issues/3557)

+ ツール

    + バックアップ＆リストア（BR）

        - 増分リストア後にテーブルにレコードを挿入した際の重複主キーを修正する[#33596](https://github.com/pingcap/tidb/issues/33596)
        - BRまたはTiDB Lightningが異常終了した後にスケジューラが再開されない問題を修正する[#33546](https://github.com/pingcap/tidb/issues/33546)
        - 空のクエリを持つDDLジョブにより誤ってBR増分リストアがエラーを返す問題を修正する[#33322](https://github.com/pingcap/tidb/issues/33322)
        - リージョンが復元中に一貫していない場合に、BRが十分なリトライを行わない問題を修正する[#33419](https://github.com/pingcap/tidb/issues/33419)
        - 復元操作中にいくつかの回復不能なエラーが発生した際にBRがスタックする問題を修正する[#33200](https://github.com/pingcap/tidb/issues/33200)
        - BRがRawKVのバックアップに失敗する問題を修正する[#32607](https://github.com/pingcap/tidb/issues/32607)
        - BRがS3内部エラーを処理できない問題を修正する[#34350](https://github.com/pingcap/tidb/issues/34350)

    + TiCDC

        - オーナーの変更により不正確なメトリックが発生する問題を修正する[#4774](https://github.com/pingcap/tiflow/issues/4774)
        - redoログマネージャがログを書き込む前にログをフラッシュするバグを修正する[#5486](https://github.com/pingcap/tiflow/issues/5486)
        - redoライターがメンテナンスされていないテーブルが存在する場合に解決済タイムスタンプが速すぎる問題を修正する[#5486](https://github.com/pingcap/tiflow/issues/5486)
        - ファイル名の衝突がデータ損失を引き起こす可能性がある問題を修正するためにredoログファイル名にUUID接尾辞を追加する[#5486](https://github.com/pingcap/tiflow/issues/5486)
        - MySQL Sinkが誤ったcheckpointTsを保存する問題を修正する[#5107](https://github.com/pingcap/tiflow/issues/5107)
        - アップグレード後にTiCDCクラスターがパニックに陥る可能性がある問題を修正する[#5266](https://github.com/pingcap/tiflow/issues/5266)
        - テーブルが同一ノードで繰り返しスケジュールされるとchangefeedが停滞する問題を修正する[#4464](https://github.com/pingcap/tiflow/issues/4464)
        - TLSを有効にした後に最初のPDセットが利用できない場合にTiCDCが起動しない問題を修正する[#4777](https://github.com/pingcap/tiflow/issues/4777)
        - PDノードが異常な場合にオープンAPI経由でステータスを問い合わせるとブロックされる場合がある問題を修正する[#4778](https://github.com/pingcap/tiflow/issues/4778)
        - Unified Sorterで使用されるworkerpoolの安定性問題を修正する[#4447](https://github.com/pingcap/tiflow/issues/4447)
        - 特定のケースでシーケンスが誤ってレプリケートされるバグを修正する[#4563](https://github.com/pingcap/tiflow/issues/4552)

    + TiDBデータ移行（DM）

        - タスクが自動的に再開された後にDMがより多くのディスクスペースを占有する問題を修正する[#3734](https://github.com/pingcap/tiflow/issues/3734) [#5344](https://github.com/pingcap/tiflow/issues/5344)
        - `case-sensitive: true`が設定されていない場合、大文字のテーブルがレプリケートされない場合がある問題を修正する[#5255](https://github.com/pingcap/tiflow/issues/5255)
        - 下流でフィルタされたDDLを手動実行した場合にタスクの再開に失敗することがある問題を修正する[#5272](https://github.com/pingcap/tiflow/issues/5272)
        - `SHOW CREATE TABLE`ステートメントでインデックスにおいて主キーが最初でない場合にDMワーカーパニックが発生する問題を修正する[#5159](https://github.com/pingcap/tiflow/issues/5159)
        - GTIDが有効な場合やタスクが自動的に再開された場合にCPU使用率が上昇し大量のログが出力される問題を修正する[#5063](https://github.com/pingcap/tiflow/issues/5063)
        - DMマスターが再起動された後にリレーログが無効になる問題を修正する[#4803](https://github.com/pingcap/tiflow/issues/4803)

    + TiDB Lightning

        - `auto_increment`列の範囲外データによってローカルバックエンドのインポートが失敗する問題を修正する[#27937](https://github.com/pingcap/tidb/issues/27937)
        - プレチェックがローカルディスクリソースとクラスタの可用性をチェックしない問題を修正する[#34213](https://github.com/pingcap/tidb/issues/34213)
      - チェックサムエラー "GCライフタイムがトランザクションの期間よりも短い" を修正します [#32733](https://github.com/pingcap/tidb/issues/32733)