---
title: TiDB 3.0.6 リリースノート
aliases: ['/docs/dev/releases/release-3.0.6/','/docs/dev/releases/3.0.6/']
---

# TiDB 3.0.6 リリースノート

リリース日: 2019年11月28日

TiDB バージョン: 3.0.6

TiDB Ansible バージョン: 3.0.6

## TiDB

+ SQL オプティマイザー
    - ウィンドウ関数 AST が SQL テキストを復元した後に結果が正しくない問題を修正し、たとえば `over w` が誤って `over (w)` に復元される問題を修正 [#12933](https://github.com/pingcap/tidb/pull/12933)
    - `STREAM AGG()` を `doubleRead` にプッシュダウンする問題を修正 [#12690](https://github.com/pingcap/tidb/pull/12690)
    - SQL バインディングでクォートが誤って処理される問題を修正 [#13117](https://github.com/pingcap/tidb/pull/13117)
    -  `select max(_tidb_rowid) from t` シナリオを最適化して全テーブルスキャンを回避する[#13095](https://github.com/pingcap/tidb/pull/13095)
    - クエリステートメントに変数割り当て式が含まれる場合にクエリ結果が正しくない問題を修正 [#13231](https://github.com/pingcap/tidb/pull/13231)
    - `UPDATE` ステートメントにサブクエリと生成列の両方が含まれる場合の結果が正しくない問題を修正し、このステートメントに異なるソースデータベースから同じテーブル名を含む場合に `UPDATE` ステートメントの実行エラーを修正 [#13350](https://github.com/pingcap/tidb/pull/13350)
    - ポイントクエリ用に `_tidb_rowid` をサポート [#13416](https://github.com/pingcap/tidb/pull/13416)
    - パーティション化されたテーブル統計の誤った使用によって生成されたクエリ実行計画が不正確な問題を修正 [#13628](https://github.com/pingcap/tidb/pull/13628)
+ SQL 実行エンジン
    - 無効な年の値を処理する際に TiDB が MySQL と互換性がない問題を修正 [#12745](https://github.com/pingcap/tidb/pull/12745)
    - `INSERT ON DUPLICATE UPDATE` ステートメントで `Chunk` を再利用してメモリオーバーヘッドを減らす問題を修正 [#12998](https://github.com/pingcap/tidb/pull/12998)
    - `JSON_VALID` 組み込み関数のサポートを追加 [#13133](https://github.com/pingcap/tidb/pull/13133)
    - パーティション化されたテーブルで `ADMIN CHECK TABLE` を実行するサポートを追加 [#13140](https://github.com/pingcap/tidb/pull/13140)
    - 空のテーブルで `FAST ANALYZE` を実行したときのパニック問題を修正 [#13343](https://github.com/pingcap/tidb/pull/13343)
    - マルチカラムインデックスを含む空のテーブルで `FAST ANALYZE` を実行したときに生じるパニック問題を修正 [#13394](https://github.com/pingcap/tidb/pull/13394)
    - ユニークキーに等しい条件が含まれる `WHERE` 句で推定行数が1よりも大きい問題を修正 [#13382](https://github.com/pingcap/tidb/pull/13382)
    - TiDB で `Streaming` が有効になっている場合、返されるデータが重複する可能性がある問題を修正 [#13254](https://github.com/pingcap/tidb/pull/13254)
    - カウントミンスケッチから上位N値を抽出し、推定精度を向上させる問題を修正 [#13429](https://github.com/pingcap/tidb/pull/13429)
+ サーバ
    - gRPC ダイアルがタイムアウトしたときに TiKV に送信されるリクエストがすぐに失敗するよう修正 [#12926](https://github.com/pingcap/tidb/pull/12926)
    - 次の仮想テーブルを追加しました: [#13009](https://github.com/pingcap/tidb/pull/13009)
        - `performance_schema.tidb_profile_allocs`
        - `performance_schema.tidb_profile_block`
        - `performance_schema.tidb_profile_cpu`
        - `performance_schema.tidb_profile_goroutines`
    - クエリが悲観的ロック待ちをしている場合に `kill` コマンドが機能しない問題を修正 [#12989](https://github.com/pingcap/tidb/pull/12989)
    - 悲観的ロック取得が失敗し、トランザクションが単一キーを修正するだけの場合に非同期ロールバックを行わないように修正 [#12707](https://github.com/pingcap/tidb/pull/12707)
    - リージョンの分割のリクエストのレスポンスが空のときにパニック問題を修正 [#13092](https://github.com/pingcap/tidb/pull/13092)
    - `PessimisticLock` がロックエラーを返す場合に不必要なバックオフを避けるよう修正 [#13116](https://github.com/pingcap/tidb/pull/13116)
    - 識別されない構成オプションに対して警告ログを出力して TiDB の構成をチェックするための振る舞いを修正 [#13272](https://github.com/pingcap/tidb/pull/13272)
    - `/info/all` インタフェースを介してすべての TiDB ノードのバイナリログステータスを取得することをサポート [#13187](https://github.com/pingcap/tidb/pull/13187)
    - TiDB がコネクションを切断するときにゴルーチンがリークする可能性がある問題を修正 [#13251](https://github.com/pingcap/tidb/pull/13251)
    - 悲観的トランザクションに対するロック待ちタイムアウトを制御するために `innodb_lock_wait_timeout` パラメータが機能するようにする修正 [#13165](https://github.com/pingcap/tidb/pull/13165)
    - 悲観的トランザクションのクエリが殺されたときに不要なくタイムアウトを避けるために悲観的トランザクション TTL の更新を停止する修正 [#13046](https://github.com/pingcap/tidb/pull/13046)
+ DDL
    - TiDB での `SHOW CREATE VIEW` の実行結果が MySQL と一貫していない問題を修正 [#12912](https://github.com/pingcap/tidb/pull/12912)
    - `union` をベースにして `View` を作成するサポートを追加, たとえば、 `create view v as select * from t1 union select * from t2` [#12955](https://github.com/pingcap/tidb/pull/12955)
    - `slow_query` テーブルに関連するトランザクションフィールドを追加しました: [#13072](https://github.com/pingcap/tidb/pull/13072)
        - `Prewrite_time`
        - `Commit_time`
        - `Get_commit_ts_time`
        - `Commit_backoff_time`
        - `Backoff_types`
        - `Resolve_lock_time`
        - `Local_latch_wait_time`
        - `Write_key`
        - `Write_size`
        - `Prewrite_region`
        - `Txn_retry`
    - テーブルが作成され、`COLLATE` を含み、システムのデフォルト文字セットではなく表の `COLLATE` を使用する修正 [#13174](https://github.com/pingcap/tidb/pull/13174)
    - テーブルを作成する際にインデックス名の長さを制限する修正 [#13310](https://github.com/pingcap/tidb/pull/13310)
    - テーブル名の長さが確認されない問題を修正するためにテーブルが名前変更されたときの修正 [#13346](https://github.com/pingcap/tidb/pull/13346)
    - プライマリキーの追加/削除をサポートするために `alter-primary-key` 設定を追加 (デフォルトでは無効) [#13522](https://github.com/pingcap/tidb/pull/13522)

## TiKV

- `acquire_pessimistic_lock` インタフェースが誤った `txn_size` を返す問題を修正 [#5740](https://github.com/tikv/tikv/pull/5740)
- パフォーマンスへの影響を減らすためにGCワーカー毎秒の書き込みを制限する修正 [#5735](https://github.com/tikv/tikv/pull/5735)
- `lock_manager` の正確性を向上させる修正 [#5845](https://github.com/tikv/tikv/pull/5845)
- 悲観的ロックがデッドロック検出器にかかる負荷を減らすためのclean upリクエストを削減する修正 [#5965](https://github.com/tikv/tikv/pull/5965)
- 悲観的ロックのプリライトリクエストでTTLを短縮しないようにする修正 [#6056](https://github.com/tikv/tikv/pull/6056)
- Titan に対する構成チェックを追加 [#5720](https://github.com/tikv/tikv/pull/5720)
- `acquire_pessimistic_lock` インタフェースが誤った `txn_size` を返す問題を修正 [#5740](https://github.com/tikv/tikv/pull/5740)
- GC I/O の制限値を動的に変更するために tikv-ctl を使用するサポートを追加: `tikv-ctl --host=ip:port modify-tikv-config -m server -n gc.max_write_bytes_per_sec -v 10MB` [#5957](https://github.com/tikv/tikv/pull/5957)
- Titan でブロブファイルが欠落する可能性がある問題を修正 [#5968](https://github.com/tikv/tikv/pull/5968)
- Titanの`RocksDBOptions`が効果を発揮しない問題を修正 [#6009](https://github.com/tikv/tikv/pull/6009)

## PD

- 各フィルタに対して `ActOn` 次元を追加し、各スケジューラとチェッカーがフィルタの影響を受けることを示し、`disconnectFilter` と `rejectLeaderFilter` の未使用フィルタを削除 [#1911](https://github.com/pingcap/pd/pull/1911)
- PD でタイムスタンプを生成するのに 5 ミリ秒以上かかる場合、警告ログを出力する [#1867](https://github.com/pingcap/pd/pull/1867)
- クライアントに利用できないエンドポイントが渡されたとき、クライアントのログレベルを下げる [#1856](https://github.com/pingcap/pd/pull/1856)
- `region_syncer` レプリケーションプロセスで gRPC メッセージパッケージが最大サイズを超える問題を修正 [#1952](https://github.com/pingcap/pd/pull/1952)

## Tools

+ TiDB Binlog
    - Drainer が `initial-commit-ts` を "−1" に設定するとき、PD から初期レプリケーションタイムスタンプを取得する [#788](https://github.com/pingcap/tidb-binlog/pull/788)
    - Drainer の `Checkpoint` ストレージをダウンストリームから切り離し、`Checkpoint` をMySQLまたはローカルファイルに保存することをサポート [#790](https://github.com/pingcap/tidb-binlog/pull/790)
    - Replicationデータベース/テーブルのフィルタリングを設定する際に空の値を使用することで発生するDrainerのパニック問題を修正 [#801](https://github.com/pingcap/tidb-binlog/pull/801)
    - Drainerがダウンストリームにバイナリログファイルを適用できずにパニックが発生し、プロセスがデッドロック状態に陥る問題を修正 [#807](https://github.com/pingcap/tidb-binlog/pull/807)
    - gRPCの `GracefulStop` によってPumpが終了しようとするとブロックする問題を修正 [#817](https://github.com/pingcap/tidb-binlog/pull/817)
    - TiDB (v3.0.6 以降) で `DROP COLUMN` ステートメントが実行される際に、列が欠落したバイナリログを受け取った場合にDrainerが失敗する問題を修正 [#827](https://github.com/pingcap/tidb-binlog/pull/827)
+ TiDB Lightning
    - TiDBバックエンドのための `max-allowed-packet` 構成（デフォルトでは64 M）を追加 [#248](https://github.com/pingcap/tidb-lightning/pull/248)