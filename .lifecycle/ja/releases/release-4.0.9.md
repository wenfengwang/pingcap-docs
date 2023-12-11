---
title: TiDB 4.0.9 リリースノート
---

# TiDB 4.0.9 リリースノート

リリース日: 2020年12月21日

TiDB バージョン: 4.0.9

## 互換性変更

+ TiDB

    - `enable-streaming`構成項目を廃止 [#21055](https://github.com/pingcap/tidb/pull/21055)

+ TiKV

    - 休止中の暗号化が有効になっている場合のI/Oおよびmutex競合を削減しました。この変更は後方互換性がありません。クラスタをv4.0.9より前のバージョンにダウングレードする必要がある場合は、`security.encryption.enable-file-dictionary-log`を無効にし、ダウングレード前にTiKVを再起動する必要があります。[#9195](https://github.com/tikv/tikv/pull/9195)

## 新機能

+ TiFlash

    - ストレージエンジンの最新データを複数のディスクに保存する機能をサポート（実験的）

+ TiDB ダッシュボード

    - **SQLステートメント**ページで全てのフィールドを表示およびソートする機能をサポート [#749](https://github.com/pingcap/tidb-dashboard/pull/749)
    - トポロジーグラフのズームおよびパンをサポート [#772](https://github.com/pingcap/tidb-dashboard/pull/772)
    - **SQLステートメント**および**スロークエリ**ページでディスク使用量情報を表示する機能をサポート [#777](https://github.com/pingcap/tidb-dashboard/pull/777)
    - **SQLステートメント**および**スロークエリ**ページでリストデータのエクスポートをサポート [#778](https://github.com/pingcap/tidb-dashboard/pull/778)
    - Prometheusアドレスのカスタマイズをサポート [#808](https://github.com/pingcap/tidb-dashboard/pull/808)
    - クラスタ統計のためのページを追加 [#815](https://github.com/pingcap/tidb-dashboard/pull/815)
    - **スロークエリ**の詳細でより多くの時間関連フィールドを追加 [#810](https://github.com/pingcap/tidb-dashboard/pull/810)

## 改善点

+ TiDB

    - 等価条件を他の条件に変換する際に、ヒューリスティックに（インデックス）マージ結合を避ける [#21146](https://github.com/pingcap/tidb/pull/21146)
    - ユーザ変数の種類を区別する [#21107](https://github.com/pingcap/tidb/pull/21107)
    - 構成ファイルで`GOGC`変数を設定する機能をサポート [#20922](https://github.com/pingcap/tidb/pull/20922)
    - ダンプされたバイナリ時刻（`Timestamp`および`Datetime`）をMySQLとより互換性のあるものにする [#21135](https://github.com/pingcap/tidb/pull/21135)
    - `LOCK IN SHARE MODE`構文を使用するステートメントに対してエラーメッセージを提供する [#21005](https://github.com/pingcap/tidb/pull/21005)
    - ショートカット可能な式の定数を畳み込む際に不要な警告またはエラーを出力しないようにする [#21040](https://github.com/pingcap/tidb/pull/21040)
    - `LOAD DATA`ステートメントの準備時にエラーを発生させるようにする [#21199](https://github.com/pingcap/tidb/pull/21199)
    - 整数カラムタイプを変更する際に整数ゼロフィルサイズの属性を無視するようにする [#20986](https://github.com/pingcap/tidb/pull/20986)
    - `EXPLAIN ANALYZE`の結果でDMLステートメントの実行時情報を追加する [#21066](https://github.com/pingcap/tidb/pull/21066)
    - 単一SQLステートメント内でプライマリキーの複数更新を禁止するようにする [#21113](https://github.com/pingcap/tidb/pull/21113)
    - 接続のアイドル時間の監視メトリクスを追加する [#21301](https://github.com/pingcap/tidb/pull/21301)
    - `runtime/trace`ツールの実行中に一時的にスローログを有効にするようにする [#20578](https://github.com/pingcap/tidb/pull/20578)

+ TiKV

    - `split`コマンドのソースをトレースするタグを追加する [#8936](https://github.com/tikv/tikv/pull/8936)
    - `pessimistic-txn.pipelined`構成を動的に変更する機能をサポートする [#9100](https://github.com/tikv/tikv/pull/9100)
    - バックアップ＆リストアおよびTiDBライトニングの実行時の影響を削減する機能をサポートする [#9098](https://github.com/tikv/tikv/pull/9098)
    - SSTエラーの出現を監視するメトリクスを追加する [#9096](https://github.com/tikv/tikv/pull/9096)
    - 一部のピアがログを複製する必要がある場合にリーダーを休止状態にしないようにする [#9093](https://github.com/tikv/tikv/pull/9093)
    - パイプライン化された悲観的ロックの成功率を上げるように変更する [#9086](https://github.com/tikv/tikv/pull/9086)
    - `apply-max-batch-size`および`store-max-batch-size`のデフォルト値を`1024`に変更する [#9020](https://github.com/tikv/tikv/pull/9020)
    - `max-background-flushes`構成項目を追加する [#8947](https://github.com/tikv/tikv/pull/8947)
    - パフォーマンス向上のためにデフォルトで`force-consistency-checks`を無効にする [#9029](https://github.com/tikv/tikv/pull/9029)
    - `pd heartbeat worker`から`split check worker`へのリージョンサイズに関するクエリをオフロードする [#9185](https://github.com/tikv/tikv/pull/9185)

+ PD

    - TiKVストアが`Tombstone`になった際にTiKVクラスターバージョンを確認し、ダウングレードまたはアップグレードプロセス中に互換性のない機能を有効にしないようにする [#3213](https://github.com/pingcap/pd/pull/3213)
    - 低いバージョンのTiKVストアを`Tombstone`から`Up`へ戻すことを許可しないようにする [#3206](https://github.com/pingcap/pd/pull/3206)

+ TiDB ダッシュボード

    - **SQLステートメント**において、「展開」がクリックされた際に引き続き展開を続けるようにする [#775](https://github.com/pingcap/tidb-dashboard/pull/775)
    - **SQLステートメント**および**スロークエリ**の詳細ページを新しいウィンドウで開くようにする [#816](https://github.com/pingcap/tidb-dashboard/pull/816)
    - **スロークエリ**の詳細において時間関連フィールドの説明を改善する [#817](https://github.com/pingcap/tidb-dashboard/pull/817)
    - 詳細なエラーメッセージを表示するようにする [#794](https://github.com/pingcap/tidb-dashboard/pull/794)

+ TiFlash

    - レプリカ読み込みの待ち時間を短縮する
    - TiFlashのエラーメッセージを改良する
    - データボリュームが大きい場合のキャッシュデータのメモリ使用を制限する
    - 処理中のコプロセッサタスクの数を監視するメトリクスを追加する

+ ツール

    + バックアップ＆リストア（BR）

        - コマンドラインで曖昧な`--checksum false`引数を禁止し、正しくチェックサムを無効にしないようにする。`--checksum=false`のみが受け入れられる。[#588](https://github.com/pingcap/br/pull/588)
        - BRが誤って終了した場合にPDが元の設定を回復できるよう、一時的にPD構成を変更する機能をサポートする [#596](https://github.com/pingcap/br/pull/596)
        - リストア後にテーブルを分析する機能をサポートする [#622](https://github.com/pingcap/br/pull/622)
        - `read index not ready`および`proposal in merging mode`エラーに対するリトライをサポートする [#626](https://github.com/pingcap/br/pull/626)

    + TiCDC

        - TiKVのHibernate Region機能を有効にするためのアラートを追加する [#1120](https://github.com/pingcap/tiflow/pull/1120)
        - スキーマストレージにおけるメモリ使用を減らす [#1127](https://github.com/pingcap/tiflow/pull/1127)
        - インクリメンタルスキャンのデータサイズが大きい場合のレプリケーションを加速する統一化ソーターの機能を追加する（実験的） [#1122](https://github.com/pingcap/tiflow/pull/1122)
        - TiCDC Open Protocolメッセージでの最大メッセージサイズと最大メッセージバッチを設定する機能をサポートする（Kafkaシンクのみ） [#1079](https://github.com/pingcap/tiflow/pull/1079)

    + Dumpling

        - 失敗したチャンクでデータをダンプし直すためのリトライを追加する [#182](https://github.com/pingcap/dumpling/pull/182)
        - 同時に`-F`および`-r`引数を設定する機能をサポートする [#177](https://github.com/pingcap/dumpling/pull/177)
        - デフォルトでシステムデータベースを`--filter`で除外する機能をサポートする [#194](https://github.com/pingcap/dumpling/pull/194)
        - `--transactional-consistency`パラメータをサポートし、リトライ中にMySQL接続を再構築する機能をサポートする [#199](https://github.com/pingcap/dumpling/pull/199)
- ダンプリングによって使用される圧縮アルゴリズムを指定する`-c,--compress`パラメーターをサポートします。空の文字列は圧縮なしを意味します。[#202](https://github.com/pingcap/dumpling/pull/202)

    + TiDB Lightning

        - デフォルトですべてのシステムスキーマをフィルタリングします[#459](https://github.com/pingcap/tidb-lightning/pull/459)
        - Local-backendまたはImporter-backend向けの自動ランダムプライマリキーのデフォルト値の設定をサポート[#457](https://github.com/pingcap/tidb-lightning/pull/457)
        - ローカルバックエンドで範囲プロパティを使用して範囲分割をより正確に行うためのサポート[#422](https://github.com/pingcap/tidb-lightning/pull/422)
        - `tikv-importer.region-split-size`、`mydumper.read-block-size`、`mydumper.batch-size`、`mydumper.max-region-size`に人間が読み取れる形式（例：「2.5 GiB」など）をサポート[#471](https://github.com/pingcap/tidb-lightning/pull/471)

    + TiDB Binlog

        - 上流のPDがダウンしている場合、またはダウンストリームへのDDLまたはDMLステートメント適用が失敗した場合に、Drainerプロセスを非ゼロコードで終了させます[#1012](https://github.com/pingcap/tidb-binlog/pull/1012)

## バグ修正

+ TiDB

    - `OR`条件を使用したときのプレフィックスインデックスによる間違った結果の問題を修正[#21287](https://github.com/pingcap/tidb/pull/21287)
    - 自動リトライが有効になっている場合にパニックを引き起こす可能性のあるバグを修正[#21285](https://github.com/pingcap/tidb/pull/21285)
    - カラムタイプに従ってパーティション定義をチェックする際に発生するバグを修正[#21273](https://github.com/pingcap/tidb/pull/21273)
    - パーティション式の値の型がパーティションカラムの型と一貫していないバグを修正[#21136](https://github.com/pingcap/tidb/pull/21136)
    - ハッシュタイプのパーティションがパーティション名の一意性をチェックしないバグを修正[#21257](https://github.com/pingcap/tidb/pull/21257)
    - `INT`でないタイプの値をハッシュパーティションテーブルに挿入した後の間違った結果を修正[#21238](https://github.com/pingcap/tidb/pull/21238)
    - `INSERT`ステートメントでインデックス結合を使用した場合に、予期しないエラーの修正問題[#21249](https://github.com/pingcap/tidb/pull/21249)
    - `BigInt`符号付き値に誤って変換される`BigInt`符号なし列の値が`CASE WHEN`演算子で返される問題を修正[#21236](https://github.com/pingcap/tidb/pull/21236)
    - インデックスハッシュ結合とインデックスマージ結合が照合を考慮しないバグを修正[#21219](https://github.com/pingcap/tidb/pull/21219)
    - `CREATE TABLE`および`SELECT`構文の中でパーティションテーブルが照合を考慮しない問題を修正[#21181](https://github.com/pingcap/tidb/pull/21181)
    - `slow_query`のクエリ結果に一部の行が抜けてしまう問題を修正[#21211](https://github.com/pingcap/tidb/pull/21211)
    - データベース名が純粋な小文字表現でない場合に、`DELETE`がデータを正しく削除しない問題を修正[#21206](https://github.com/pingcap/tidb/pull/21206)
    - DML操作後のスキーマ変更を引き起こすバグを修正[#21050](https://github.com/pingcap/tidb/pull/21050)
    - 結合を使用している際に、縮退列をクエリできないバグを修正[#21021](https://github.com/pingcap/tidb/pull/21021)
    - いくつかのセミ結合クエリの間違った結果を修正[#21019](https://github.com/pingcap/tidb/pull/21019)
    - `UPDATE`ステートメントでテーブルロックが効かない問題を修正[#21002](https://github.com/pingcap/tidb/pull/21002)
    - 再帰ビューを構築する際にスタックオーバーフローが発生する問題を修正[#21001](https://github.com/pingcap/tidb/pull/21001)
    - 外部結合でインデックスマージが使用される場合に誤った結果が得られる問題を修正[#20954](https://github.com/pingcap/tidb/pull/20954)
    - 未確定結果を失敗として取り扱う可能性のあるトランザクションの問題を修正[#20925](https://github.com/pingcap/tidb/pull/20925)
    - `EXPLAIN FOR CONNECTION`が最後のクエリプランを表示できない問題を修正[#21315](https://github.com/pingcap/tidb/pull/21315)
    - Read Committed分離レベルのトランザクションでIndex Mergeを使用した場合に結果が正しくない可能性のある問題を修正[#21253](https://github.com/pingcap/tidb/pull/21253)
    - 書き込み競合後のトランザクションリトライによる自動ID割り当ての失敗を修正[#21079](https://github.com/pingcap/tidb/pull/21079)
    - `LOAD DATA`を使用してJSONデータをTiDBに正しくインポートできない問題を修正[#21074](https://github.com/pingcap/tidb/pull/21074)
    - 新たに追加された`Enum`型列のデフォルト値が正しくない問題を修正[#20998](https://github.com/pingcap/tidb/pull/20998)
    - `adddate`関数が無効な文字を挿入する問題を修正[#21176](https://github.com/pingcap/tidb/pull/21176)
    - いくつかの状況で誤った結果を引き起こす`PointGet`プランの問題を修正[#21244](https://github.com/pingcap/tidb/pull/21244)
    - MySQLとの互換性を保つために、`ADD_DATE`関数でサマータイムの変換を無視する問題を修正[#20888](https://github.com/pingcap/tidb/pull/20888)
    - `varchar`または`char`の長さ制約を超える末尾にスペースを含む文字列を挿入できない問題を修正[#21282](https://github.com/pingcap/tidb/pull/21282)
    - `int`を`year`と比較する際に`[1, 69]`の整数を`[2001, 2069]`または`[70, 99]`の整数を`[1970, 1999]`に変換しない問題を修正[#21283](https://github.com/pingcap/tidb/pull/21283)
    - `sum()`関数のオーバーフロー結果がパニックを引き起こす問題を修正[#21272](https://github.com/pingcap/tidb/pull/21272)
    - 一意キーにロックが追加されない`DELETE`のバグを修正[#20705](https://github.com/pingcap/tidb/pull/20705)
    - スナップショットリードがロックキャッシュに影響する問題を修正[#21539](https://github.com/pingcap/tidb/pull/21539)
    - 長寿命トランザクションで多くのデータを読み込んだ後に潜在的なメモリリークが発生する問題を修正[#21129](https://github.com/pingcap/tidb/pull/21129)
    - サブクエリでテーブルエイリアスを省略すると構文エラーが返される問題を修正[#20367](https://github.com/pingcap/tidb/pull/20367)
    - クエリで`IN`関数の引数が時間型の場合、間違った結果が返される問題を修正[#21290](https://github.com/pingcap/tidb/issues/21290)

+ TiKV

    - コプロセッサが255列を超える場合に誤った結果を返す問題を修正[#9131](https://github.com/tikv/tikv/pull/9131)
    - ネットワーク分割中にリージョンマージがデータ損失を引き起こす可能性がある問題を修正[#9108](https://github.com/tikv/tikv/pull/9108)
    - `latin1`文字セットを使用した場合に`ANALYZE`ステートメントがパニックを引き起こす可能性がある問題を修正[#9082](https://github.com/tikv/tikv/pull/9082)
    - 数値型を時間型に変換したときに返される誤った結果を修正[#9031](https://github.com/tikv/tikv/pull/9031)
    - Transparent Data Encryption（TDE）が有効になっている場合に、TiDB LightningがSSTファイルをImporter-backendまたはLocal-backendに正常に取り込めない問題を修正[#8995](https://github.com/tikv/tikv/pull/8995)
    - 無効な`advertise-status-addr`値（`0.0.0.0`）の修正[#9036](https://github.com/tikv/tikv/pull/9036)
    - コミット済みトランザクションでロックされたキーが存在するとエラーが返される問題を修正[#8930](https://github.com/tikv/tikv/pull/8930)
- RocksDBキャッシュマッピングエラーがデータの破損を引き起こす問題を修正 [#9029](https://github.com/tikv/tikv/pull/9029)
    - リーダーが転送された後にFollower Readが古いデータを返す可能性があるバグを修正 [#9240](https://github.com/tikv/tikv/pull/9240)
    - Pesimmisticロックで古い値が読み取られる可能性がある問題を修正 [#9282](https://github.com/tikv/tikv/pull/9282)
    - リーダー転送後にレプリカリードが古いデータを取得する可能性があるバグを修正 [#9240](https://github.com/tikv/tikv/pull/9240)
    - プロファイリング後に`SIGPROF`を受信した際にTiKVがクラッシュする問題を修正 [#9229](https://github.com/tikv/tikv/pull/9229)

+ PD

    - プレースメントルールを使用して指定されたリーダーロールが一部のケースで効果を持たない問題を修正 [#3208](https://github.com/pingcap/pd/pull/3208)
    - `trace-region-flow`値が予期せず`false`に設定される問題を修正 [#3120](https://github.com/pingcap/pd/pull/3120)
    - 無制限のTime To Live（TTL）を持つサービスsafepointが機能しないバグを修正 [#3143](https://github.com/pingcap/pd/pull/3143)

+ TiDB Dashboard

    - 中国語で時刻の表示に関する問題を修正 [#755](https://github.com/pingcap/tidb-dashboard/pull/755)
    - ブラウザ互換性の通知が機能しないバグを修正 [#776](https://github.com/pingcap/tidb-dashboard/pull/776)
    - 特定のシナリオでトランザクション`start_ts`が誤って表示される問題を修正 [#793](https://github.com/pingcap/tidb-dashboard/pull/793)
    - 一部のSQLテキストが誤ってフォーマットされる問題を修正 [#805](https://github.com/pingcap/tidb-dashboard/pull/805)

+ TiFlash

    - `INFORMATION_SCHEMA.CLUSTER_HARDWARE`が使用されていないディスク情報を含む可能性がある問題を修正
    - Delta Cacheのメモリ使用量の見積もりが実際の使用量よりも小さい問題を修正
    - スレッド情報統計によって引き起こされるメモリリークを修正

+ Tools

    + Backup & Restore (BR)

        - S3シークレットアクセスキーに特殊文字が含まれていることによる失敗を修正 [#617](https://github.com/pingcap/br/pull/617)

    + TiCDC

        - オーナーキャンペーンキーが削除された際に複数の所有者が存在する可能性がある問題を修正 [#1104](https://github.com/pingcap/tiflow/pull/1104)
        - TiKVノードがクラッシュしたりクラッシュから回復した際にTiCDCがデータを継続してレプリケートできない可能性があるバグを修正。このバグはv4.0.8でのみ発生する。[#1198](https://github.com/pingcap/tiflow/pull/1198)
        - テーブルの初期化前にメタデータが繰り返しetcdにフラッシュされる問題を修正 [#1191](https://github.com/pingcap/tiflow/pull/1191)
        - 早期GCや`TableInfo`の更新の遅延によって起こるレプリケーションの中断問題を修正。これはスキーマストレージがTiDBテーブルをキャッシュする際にのみ発生する [#1114](https://github.com/pingcap/tiflow/pull/1114)
        - DDL操作が頻繁に行われる際にスキーマストレージが過剰なメモリを使用する問題を修正 [#1127](https://github.com/pingcap/tiflow/pull/1127)
        - チェンジフィードが一時停止または停止した際にゴルーチンがリークするバグを修正 [#1075](https://github.com/pingcap/tiflow/pull/1075)
        - ダウンストリームのKafkaでサービスまたはネットワークの揺れによるレプリケーションの中断を防ぐために、Kafkaプロデューサーでの最大リトライタイムアウトを600秒に増やす [#1118](https://github.com/pingcap/tiflow/pull/1118)
        - Kafkaのバッチサイズが効果を持たないバグを修正 [#1112](https://github.com/pingcap/tiflow/pull/1112)
        - TiCDCとPD間のネットワークに揺れがあり、一時停止されたチェンジフィードが同時に再開される場合に、一部のテーブルの行の変更が失われる問題を修正 [#1213](https://github.com/pingcap/tiflow/pull/1213)
        - TiCDCプロセスが不安定な際にTiCDCプロセスが終了する可能性があるバグを修正 [#1218](https://github.com/pingcap/tiflow/pull/1218)
        - TiCDCでPDクライアントをシングルトンで使用し、誤ってPDクライアントを閉じてレプリケーションがブロックされるバグを修正 [#1217](https://github.com/pingcap/tiflow/pull/1217)
        - TiCDCオーナーがetcdウォッチクライアントで過剰なメモリを消費するバグを修正 [#1224](https://github.com/pingcap/tiflow/pull/1224)

    + Dumpling

        - MySQLデータベースサーバーへの接続が閉じられた際にDumplingがブロックされる問題を修正 [#190](https://github.com/pingcap/dumpling/pull/190)

    + TiDB Lightning

        - 間違ったフィールド情報を使用してキーがエンコードされる問題を修正 [#437](https://github.com/pingcap/tidb-lightning/pull/437)
        - GCライフタイムTTLが効果を持たない問題を修正 [#448](https://github.com/pingcap/tidb-lightning/pull/448)
        - ローカルバックエンドモードで実行中のTiDB Lightningを手動で停止した際にパニックを引き起こす問題を修正 [#484](https://github.com/pingcap/tidb-lightning/pull/484)