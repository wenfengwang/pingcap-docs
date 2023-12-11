---
title: TiDB 3.0.5のリリースノート
aliases: ['/docs/dev/releases/release-3.0.5/','/docs/dev/releases/3.0.5/']
---

# TiDB 3.0.5のリリースノート

リリース日: 2019年10月25日

TiDBのバージョン: 3.0.5

TiDB Ansibleのバージョン: 3.0.5

## TiDB

+ SQLオプティマイザ
    - Window関数における境界チェックのサポート [#12404](https://github.com/pingcap/tidb/pull/12404)
    - パーティションテーブルにおける`IndexJoin`の誤った結果を修正 [#12712](https://github.com/pingcap/tidb/pull/12712)
    - Outer joinの`Apply`演算子の上にある`ifnull`関数において誤った結果を修正 [#12694](https://github.com/pingcap/tidb/pull/12694)
    - `UPDATE`の`where`条件でのサブクエリを含む場合の更新失敗を修正 [#12597](https://github.com/pingcap/tidb/pull/12597)
    - クエリ条件に`cast`関数が含まれる場合に、アウタージョインが誤ってインナージョインに変換される問題を修正 [#12790](https://github.com/pingcap/tidb/pull/12790)
    - `AntiSemiJoin`の結合条件での不正確な式の修正 [#12799](https://github.com/pingcap/tidb/pull/12799)
    - 統計情報の浅いコピーによる統計エラーを修正 [#12817](https://github.com/pingcap/tidb/pull/12817)
    - TiDBにおける`str_to_date`関数が、日付文字列とフォーマット文字列が一致しない場合にMySQLと異なる結果を返す問題を修正 [#12725](https://github.com/pingcap/tidb/pull/12725)
+ SQL実行エンジン
    - `from_unixtime`関数がnullを処理する際のパニック問題を修正 [#12551](https://github.com/pingcap/tidb/pull/12551)
    - DDLジョブのキャンセル時に報告される`invalid list index`エラーを修正 [#12671](https://github.com/pingcap/tidb/pull/12671)
    - Window関数を使用する際に配列が範囲外になる問題を修正 [#12660](https://github.com/pingcap/tidb/pull/12660)
    - 暗黙的に割り当てられる`AutoIncrement`列の動作を改善し、MySQLのデフォルトの自動インクリメントロックモード（["連続"ロックモード](https://dev.mysql.com/doc/refman/5.7/en/innodb-auto-increment-handling.html)）と一貫性を保つようにしました。単一行の`Insert`ステートメントで複数の暗黙的に割り当てられた`AutoIncrement` IDの連続性を保証します。この改善により、JDBCの`getGeneratedKeys()`メソッドがどんなシナリオでも正しい結果を取得できるようになります。 [#12602](https://github.com/pingcap/tidb/pull/12602)
    - `HashAgg`が`Apply`の子ノードとして機能する際にクエリがハングする問題を修正 [#12766](https://github.com/pingcap/tidb/pull/12766)
    - 型変換に関する`AND`と`OR`論理式が誤った結果を返す問題を修正 [#12811](https://github.com/pingcap/tidb/pull/12811)
+ サーバ
    - 今後の大規模トランザクションのサポートを支援するために、トランザクションTTLを変更するためのインターフェース関数を実装 [#12397](https://github.com/pingcap/tidb/pull/12397)
    - ペシミスティックトランザクションをサポートするために、必要に応じてトランザクションTTLを延長する機能をサポート（最大10分間）[#12579](https://github.com/pingcap/tidb/pull/12579)
    - TiDBがスキーマの変更やそれに対応する変更されたテーブル情報を100から1024にキャッシュする回数を調整し、`tidb_max_delta_schema_count`システム変数を使用して変更をサポート [#12502](https://github.com/pingcap/tidb/pull/12502)
    - `kvrpc.Cleanup`プロトコルの振る舞いを更新し、期限切れでないトランザクションのロックをクリーンアップしないようにしました [#12417](https://github.com/pingcap/tidb/pull/12417)
    - Partitionテーブル情報を`information_schema.tables`テーブルにロギングする機能をサポート [#12631](https://github.com/pingcap/tidb/pull/12631)
    - `region-cache-ttl`を構成することでリージョンキャッシュのTTLを変更する機能をサポート [#12683](https://github.com/pingcap/tidb/pull/12683)
    - 遅いログで実行計画の圧縮エンコード情報を表示する機能をサポート。この機能はデフォルトで有効になっており、`slow-log-plan`構成または`tidb_record_plan_in_slow_log`変数を使用して制御できます。また、`tidb_decode_plan`関数を使用して遅いログ内の実行計画列のエンコード情報を実行計画情報にデコードできます。[#12808](https://github.com/pingcap/tidb/pull/12808)
    - `information_schema.processlist`テーブルにメモリ使用量情報を表示する機能をサポート [#12801](https://github.com/pingcap/tidb/pull/12801)
    - TiKVクライアントがアイドル接続を判断する際のエラーや予期しない警告が発生する可能性のあるエラーを修正 [#12846](https://github.com/pingcap/tidb/pull/12846)
    - `tikvSnapshot`が`BatchGet()`のKV結果を適切にキャッシュしなかったため、`INSERT IGNORE`ステートメントのパフォーマンスが低下した問題を修正 [#12872](https://github.com/pingcap/tidb/pull/12872)
    - 一部のKVサービスへの接続が遅かったためにTiDBの応答速度が比較的低下した問題を修正 [#12814](https://github.com/pingcap/tidb/pull/12814)
+ DDL
    - `Create Table`操作がSet列のInt型デフォルト値を正しく設定しない問題を修正 [#12267](https://github.com/pingcap/tidb/pull/12267)
    - `Create Table`ステートメントで一意なインデックスを作成する際の複数の`unique`をサポート [#12463](https://github.com/pingcap/tidb/pull/12463)
    - `Alter Table`を使用してBit型の列を追加する際に既存の行のこの列のデフォルト値を設定するとエラーが発生する問題を修正 [#12489](https://github.com/pingcap/tidb/pull/12489)
    - RangeパーティションテーブルでDateまたはDatetime型の列をパーティションキーとして使用する際に、パーティションの追加の失敗を修正 [#12815](https://github.com/pingcap/tidb/pull/12815)
    - Rangeパーティションテーブルの一意キー列セットがパーティション化列セット以上であることを確認するチェックを追加 [#12718](https://github.com/pingcap/tidb/pull/12718)
+ モニタ
    - `Transaction OPS`ダッシュボードにCommitとRollback操作のモニタリングメトリクスを追加 [#12505](https://github.com/pingcap/tidb/pull/12505)
    - `Add Index`操作の進行状況のモニタリングメトリクスを追加 [#12390](https://github.com/pingcap/tidb/pull/12390)

## TiKV

+ ストレージ
    - ペシミスティックトランザクションの新機能を追加: 期限切れのTTLを持つロックのクリーンアップインターフェースをサポートのみ [#5589](https://github.com/tikv/tikv/pull/5589)
    - トランザクションのプライマリキーのロールバックが折りたたまれる問題を修正 [#5646](https://github.com/tikv/tikv/pull/5646), [#5671](https://github.com/tikv/tikv/pull/5671)
    - ペシミスティックロックの下で、ポイントクエリが以前のバージョンのデータを返す可能性がある問題を修正 [#5634](https://github.com/tikv/tikv/pull/5634)
+ Raftstore
    - メッセージのフラッシュ操作を減らしてパフォーマンスを向上し、CPU使用率を削減するため [#5617](https://github.com/tikv/tikv/pull/5617)
    - リージョンサイズと推定キーの数を取得するコストを最適化し、ハートビートのオーバーヘッドとCPU使用率を削減するため [#5620](https://github.com/tikv/tikv/pull/5620)
    - 無効なデータを取得しようとするとRaftstoreがエラーログを出力してパニックする問題を修正 [#5643](https://github.com/tikv/tikv/pull/5643)
+ Engine
    - データの安全性を向上するためにRocksDBの`force_consistency_checks`を有効にしました [#5662](https://github.com/tikv/tikv/pull/5662)
    - Titanでの同時のフラッシュ操作がデータ損失を引き起こす可能性がある問題を修正 [#5672](https://github.com/tikv/tikv/pull/5672)
- rust-rocksdbのバージョンを更新して、TiKVのクラッシュおよび再起動がL0内部のコンパクションによって引き起こされる問題を回避します [#5710](https://github.com/tikv/tikv/pull/5710)

## PD

- Regionが占有するストレージの精度を向上させる [#1782](https://github.com/pingcap/pd/pull/1782)
- `--help`コマンドの出力を改善する [#1763](https://github.com/pingcap/pd/pull/1763)
- TLSが有効になった後にHTTPリクエストがリダイレクトに失敗する問題を修正する [#1777](https://github.com/pingcap/pd/pull/1777)
- pd-ctlが`store shows limit`コマンドを使用したときに発生するパニックの問題を修正する [#1808](https://github.com/pingcap/pd/pull/1808)
- ラベルの監視メトリクスの読みやすさを向上させ、リーダーが切り替わったときに元のリーダーの監視データをリセットして、誤ったレポートを回避するための監視データをリセットする [#1815](https://github.com/pingcap/pd/pull/1815)

## Tools

+ TiDB Binlog
    - `ALTER DATABASE`に関連するDDL操作によってDrainerが異常終了する問題を修正する [#769](https://github.com/pingcap/tidb-binlog/pull/769)
    - リプリケーション効率を向上させるためにCommit binlogのトランザクションステータス情報のクエリをサポートする [#757](https://github.com/pingcap/tidb-binlog/pull/757)
    - Drainerの`start_ts`がPumpの最大`commit_ts`よりも大きい場合に、Pumpのパニックが発生する可能性がある問題を修正する [#758](https://github.com/pingcap/tidb-binlog/pull/758)
+ TiDB Lightning
    - Loaderの完全な論理インポート機能を統合し、バックエンドモードの設定をサポートする [#221](https://github.com/pingcap/tidb-lightning/pull/221)

## TiDB Ansible

- インデックス追加速度の監視メトリクスを追加する [#986](https://github.com/pingcap/tidb-ansible/pull/986)
- ユーザーが設定する必要のないパラメータを削除し、構成ファイルの内容を簡素化する [#1043c](https://github.com/pingcap/tidb-ansible/commit/1043c3df7ddb72eb234c55858960e9fdd3830a14), [#998](https://github.com/pingcap/tidb-ansible/pull/998)
- パフォーマンスの読み取りと書き込みの監視式エラーを修正する [#e90e7](https://github.com/pingcap/tidb-ansible/commit/e90e79f5117bb89197e01b1391fd02e25d57a440)
- Raftstore CPU使用率の監視表示方法とアラームルールを更新する [#992](https://github.com/pingcap/tidb-ansible/pull/992)
- 概要監視ダッシュボードのTiKV CPU監視項目を更新して、過剰な監視内容をフィルタリングする [#1001](https://github.com/pingcap/tidb-ansible/pull/1001)