---
title: TiDB 3.0.0-rc.3 リリースノート
aliases: ['/docs/dev/releases/release-3.0.0-rc.3/','/docs/dev/releases/3.0.0-rc.3/']
---

# TiDB 3.0.0-rc.3 リリースノート

リリース日: 2019年6月21日

TiDB バージョン: 3.0.0-rc.3

TiDB Ansible バージョン: 3.0.0-rc.3

## 概要

2019年6月21日、TiDB 3.0.0-rc.3 がリリースされました。対応する TiDB Ansible バージョンは 3.0.0-rc.3 です。このリリースは、TiDB 3.0.0-rc.2 と比較して、安定性、利便性、機能、SQL オプティマイザ、統計情報、実行エンジンが大幅に改善されています。

## TiDB

+ SQL オプティマイザ
    - 仮想生成列の統計情報収集機能を削除しました [#10629](https://github.com/pingcap/tidb/pull/10629)
    - ポイントクエリ中の主キー定数のオーバーフロー問題を修正しました [#10699](https://github.com/pingcap/tidb/pull/10699)
    - `fast analyze` で初期化されていない情報を使用するとパニックが発生する問題を修正しました [#10691](https://github.com/pingcap/tidb/pull/10691)
    - `prepare` を使用して `create view` ステートメントを実行すると、誤った列情報によりパニックが発生する問題を修正しました [#10713](https://github.com/pingcap/tidb/pull/10713)
    - ウィンドウ関数を処理する際に列情報がクローンされない問題を修正しました [#10720](https://github.com/pingcap/tidb/pull/10720)
    - インデックス結合の内部テーブル選択の選択率の見積もりが正しくない問題を修正しました [#10854](https://github.com/pingcap/tidb/pull/10854)
    - `stats-lease` 変数の値が 0 の場合に統計情報の自動読み込みをサポートしました [#10811](https://github.com/pingcap/tidb/pull/10811)

+ 実行エンジン
    - `StreamAggExec` で `Close` 関数を呼び出す際にリソースが正しく解放されない問題を修正しました [#10636](https://github.com/pingcap/tidb/pull/10636)
    - パーティションテーブルの `show create table` ステートメントの実行結果における `table_option` と `partition_options` の順序が正しくない問題を修正しました [#10689](https://github.com/pingcap/tidb/pull/10689)
    - `admin show ddl jobs` のパフォーマンスを向上させるため、逆順でデータをスキャンすることをサポートしました [#10687](https://github.com/pingcap/tidb/pull/10687)
    - RBAC における `show grants` ステートメントの結果が MySQL のそれと互換性がない問題を修正しました（このステートメントに `current_user` フィールドがある場合） [#10684](https://github.com/pingcap/tidb/pull/10684)
    - UUID が複数のノードで重複した値を生成する可能性がある問題を修正しました [#10712](https://github.com/pingcap/tidb/pull/10712)
    - `explain` において `show view` 権限が考慮されていない問題を修正しました [#10635](https://github.com/pingcap/tidb/pull/10635)
    - テーブル Region を手動で分割してホットスポットの問題を緩和するための `split table region` ステートメントを追加しました [#10765](https://github.com/pingcap/tidb/pull/10765)
    - インデックス Region を手動で分割してホットスポットの問題を緩和するための `split index region` ステートメントを追加しました [#10764](https://github.com/pingcap/tidb/pull/10764)
    - `create user`、`grant`、または `revoke` などの複数のステートメントを連続して実行する際の実行問題を修正しました [#10737](https://github.com/pingcap/tidb/pull/10737)
    - Coprocessor への式のプッシュダウンを禁止するためのブロックリストを追加しました [#10791](https://github.com/pingcap/tidb/pull/10791)
    - クエリがメモリ構成の制限を超えた場合に `expensive query` ログを出力する機能を追加しました [#10849](https://github.com/pingcap/tidb/pull/10849)
    - 変更されたバインディング実行プランの更新時間を制御するための `bind-info-lease` 構成項目を追加しました [#10727](https://github.com/pingcap/tidb/pull/10727)
    - `execdetails.ExecDetails` ポインタの失敗により、高同時実行シナリオにおける Coprocessor リソースの迅速な解放による OOM 問題を修正しました [#10832](https://github.com/pingcap/tidb/pull/10832)
    - 一部の場合に `kill` ステートメントによってパニックが発生する問題を修正しました [#10876](https://github.com/pingcap/tidb/pull/10876)

+ サーバー
    - GC を修復する際に goroutine がリークする可能性がある問題を修正しました [#10683](https://github.com/pingcap/tidb/pull/10683)
    - 遅いクエリで `host` 情報を表示する機能をサポートしました [#10693](https://github.com/pingcap/tidb/pull/10693)
    - TiKV とやり取りするアイドルリンクを再利用する機能をサポートしました [#10632](https://github.com/pingcap/tidb/pull/10632)
    - RBAC で `skip-grant-table` オプションを有効にするサポートを修正しました [#10738](https://github.com/pingcap/tidb/pull/10738)
    - `pessimistic-txn` 構成が無効になる問題を修正しました [#10825](https://github.com/pingcap/tidb/pull/10825)
    - アクティブにキャンセルされた ticlient リクエストが再試行される問題を修正しました [#10850](https://github.com/pingcap/tidb/pull/10850)
    - 悲観的トランザクションが楽観的トランザクションと競合する場合のパフォーマンスを向上させました [#10881](https://github.com/pingcap/tidb/pull/10881)

+ DDL
    - `alter table` を使用して文字セットを変更すると `blob` タイプが変更される問題を修正しました [#10698](https://github.com/pingcap/tidb/pull/10698)
    - `AUTO_INCREMENT` 属性を含む列で `SHARD_ROW_ID_BITS` を使用して行 ID を散開する機能を追加し、ホットスポットの問題を緩和しました [#10794](https://github.com/pingcap/tidb/pull/10794)
    - `alter table` ステートメントを使用してストアド生成列を追加することを禁止しました [#10808](https://github.com/pingcap/tidb/pull/10808)
    - DDL メタデータの無効な生存時間を最適化し、クラスタのアップグレード後の期間を短縮しました [#10795](https://github.com/pingcap/tidb/pull/10795)

## PD

- 一方向マージのみを許可するための `enable-two-way-merge` 構成項目を追加しました [#1583](https://github.com/pingcap/pd/pull/1583)
- リージョンスキャッタリングが制限メカニズムによって制限されないように、`AddLightLearner` と `AddLightPeer` のスケジューリング操作を追加しました [#1563](https://github.com/pingcap/pd/pull/1563)
- システムが起動された際にデータに1つのレプリカのみが存在する可能性があるため、信頼性が不十分な問題を修正しました [#1581](https://github.com/pingcap/pd/pull/1581)
- 構成チェックロジックを最適化し、構成項目エラーを回避しました [#1585](https://github.com/pingcap/pd/pull/1585)
- `store-balance-rate` 構成の定義を1分あたりに生成されるバランス演算子の数の上限に調整しました [#1591](https://github.com/pingcap/pd/pull/1591)
- ストアがスケジュールされた操作を生成できなくなる可能性がある問題を修正しました [#1590](https://github.com/pingcap/pd/pull/1590)

## TiKV

+ エンジン
    - システムに不完全なスナップショットが生成される問題を修正しました（イテレータがステータスをチェックしないため） [#4936](https://github.com/tikv/tikv/pull/4936)
    - 異常な状態でのシステムの電源障害後にスナップショットを受信してからデータをディスクにフラッシュする遅延によるデータ損失問題を修正しました [#4850](https://github.com/tikv/tikv/pull/4850)

+ サーバー
    - `block-size` 構成の有効性をチェックする機能を追加しました [#4928](https://github.com/tikv/tikv/pull/4928)
    - `READ_INDEX` 関連の監視メトリクスを追加しました [#4830](https://github.com/tikv/tikv/pull/4830)
    - GC ワーカー関連の監視メトリクスを追加しました [#4922](https://github.com/tikv/tikv/pull/4922)

+ Raftstore
    - ローカルリーダーのキャッシュが正しくクリアされない問題を修正しました [#4778](https://github.com/tikv/tikv/pull/4778)
    - リーダーの転送および `conf` の変更時にリクエストの遅延が増加する可能性がある問題を修正しました [#4734](https://github.com/tikv/tikv/pull/4734)
- スタールなコマンドが誤って報告される問題を修正する [#4682](https://github.com/tikv/tikv/pull/4682)
    - コマンドが長時間保留される可能性がある問題を修正する [#4810](https://github.com/tikv/tikv/pull/4810)
    - スナップショットファイルをディスクに同期する遅延によって引き起こされる停電後のファイルの損傷を修正する問題を修正する [#4807](https://github.com/tikv/tikv/pull/4807), [#4850](https://github.com/tikv/tikv/pull/4850)

+ Coprocessor
    - ベクトル計算でのTop-Nをサポートする [#4827](https://github.com/tikv/tikv/pull/4827)
    - ベクトル計算での`Stream`集計をサポートする [#4786](https://github.com/tikv/tikv/pull/4786)
    - ベクトル計算での`AVG`集約関数をサポートする [#4777](https://github.com/tikv/tikv/pull/4777)
    - ベクトル計算での`First`集約関数をサポートする [#4771](https://github.com/tikv/tikv/pull/4771)
    - ベクトル計算での`SUM`集約関数をサポートする [#4797](https://github.com/tikv/tikv/pull/4797)
    - ベクトル計算での`MAX`/`MIN`集約関数をサポートする [#4837](https://github.com/tikv/tikv/pull/4837)
    - ベクトル計算での`Like`式をサポートする [#4747](https://github.com/tikv/tikv/pull/4747)
    - ベクトル計算での`MultiplyDecimal`式をサポートする [#4849](https://github.com/tikv/tikv/pull/4849 )
    - ベクトル計算での`BitAnd`/`BitOr`/`BitXor`式をサポートする [#4724](https://github.com/tikv/tikv/pull/4724)
    - ベクトル計算での`UnaryNot`式をサポートする [#4808](https://github.com/tikv/tikv/pull/4808)

+ トランザクション
    - 悲観的トランザクションにおける非悲観的ロック競合によって発生するエラーを修正する問題を修正する [#4801](https://github.com/tikv/tikv/pull/4801), [#4883](https://github.com/tikv/tikv/pull/4883)
    - 悲観的トランザクションを有効にした後の楽観的トランザクションに対する不必要な計算を削減してパフォーマンスを改善する [#4813](https://github.com/tikv/tikv/pull/4813)
    - デッドロックの状況において全体のトランザクションにロールバックが必要なくなるように単一ステートメントのロールバック機能を追加する [#4848](https://github.com/tikv/tikv/pull/4848)
    - 悲観的トランザクション関連の監視項目を追加する [#4852](https://github.com/tikv/tikv/pull/4852)
    - 激しい競合が存在する場合に性能を向上させるために`ResolveLockLite`コマンドを使用して軽量ロックを解決する機能をサポートする [#4882](https://github.com/tikv/tikv/pull/4882)

+ tikv-ctl
    - より多くの異常条件をチェックするための`bad-regions`コマンドを追加する [#4862](https://github.com/tikv/tikv/pull/4862)
    - `tombstone`コマンドを強制的に実行する機能を追加する [#4862](https://github.com/tikv/tikv/pull/4862)

+ その他
    - `dist_release`コンパイルコマンドを追加する [#4841](https://github.com/tikv/tikv/pull/4841)

## ツール

+ TiDB Binlog
    - Pumpがデータ書き込みに失敗した場合にその返された値をチェックしなかったことによる誤ったオフセットの問題を修正する [#640](https://github.com/pingcap/tidb-binlog/pull/640)
    - コンテナ環境でブリッジモードをサポートするためにDrainerに`advertise-addr`構成を追加する [#634](https://github.com/pingcap/tidb-binlog/pull/634)
    - トランザクションの状態をクエリする速度を向上させるためにPumpに`GetMvccByEncodeKey`関数を追加する [#632](https://github.com/pingcap/tidb-binlog/pull/632)

## TiDB Ansible

- クラスターの最大QPS値を予測するための監視項目を追加する（デフォルトでは非表示） [#f5cfa4d](https://github.com/pingcap/tidb-ansible/commit/f5cfa4d903bbcd77e01eddc8d31eabb6e6157f73)