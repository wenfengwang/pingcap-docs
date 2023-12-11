---
title: TiDB 3.0.14 リリースノート
aliases: ['/docs/dev/releases/release-3.0.14/','/docs/dev/releases/3.0.14/']
---

# TiDB 3.0.14 リリースノート

リリース日: 2020年5月9日

TiDB バージョン: 3.0.14

## 互換性の変更

+ TiDB

    - `performance_schema`および`metrics_schema`内のユーザー権限を読み取り専用から読み書き可能に調整 [#15417](https://github.com/pingcap/tidb/pull/15417)

## 重要なバグ修正

+ TiDB

    - `join`条件が`handle`属性を持つ列で複数の等価条件を持つ場合に、`index join`のクエリ結果が不正確である問題を修正 [#15734](https://github.com/pingcap/tidb/pull/15734)
    - `handle`属性を持つ列に対する`fast analyze`操作を実行する際に発生するパニックを修正 [#16079](https://github.com/pingcap/tidb/pull/16079)
    - `DDL`ステートメントを`prepare`の方法で実行した際に、`DDL job`構造体内の`query`フィールドが不正確であり、Binlogがデータレプリケーションに使用された場合に、上流と下流の間でデータの不整合を引き起こす可能性のある問題を修正 [#15443](https://github.com/pingcap/tidb/pull/15443)

+ TiKV

    - ロックのクリーンアップに関する繰り返しリクエストがトランザクションの原子性を破壊する可能性がある問題を修正 [#7388](https://github.com/tikv/tikv/pull/7388)

## 新機能

+ TiDB

    - `admin show ddl jobs`ステートメントのクエリ結果にスキーマ名列とテーブル名列を追加 [#16428](https://github.com/pingcap/tidb/pull/16428)
    - 切り捨てられたテーブルの復元をサポートするために`RECOVER TABLE`構文を強化 [#15458](https://github.com/pingcap/tidb/pull/15458)
    - `SHOW GRANTS`ステートメントの権限チェックをサポート [#16168](https://github.com/pingcap/tidb/pull/16168)
    - `LOAD DATA`ステートメントの権限チェックをサポート [#16736](https://github.com/pingcap/tidb/pull/16736)
    - パーティションキーに関連する時刻および日付に関連する関数が使用される場合のパーティションプルーニングのパフォーマンスを向上 [#15618](https://github.com/pingcap/tidb/pull/15618)
    - `dispatch error`のログレベルを`WARN`から`ERROR`に調整 [#16232](https://github.com/pingcap/tidb/pull/16232)
    - `require-secure-transport`の起動オプションをサポートし、クライアントにTLSを使用するように強制する [#15415](https://github.com/pingcap/tidb/pull/15415)
    - TLSを構成した場合に、TiDBコンポーネント間でのHTTP通信をサポート [#15419](https://github.com/pingcap/tidb/pull/15419)
    - 現在のトランザクションの`start_ts`情報を`information_schema.processlist`テーブルに追加 [#16160](https://github.com/pingcap/tidb/pull/16160)
    - クラスタ間の通信に使用されるTLS証明書情報を自動的に再読み込みする機能をサポート [#15162](https://github.com/pingcap/tidb/pull/15162)
    - パーティションプルーニングのリードパフォーマンスを向上させ、パーティション化されたテーブルの再構築を行うことで [#15628](https://github.com/pingcap/tidb/pull/15628)
    - `range`パーティションテーブルのパーティション式として`floor(unix_timestamp(a))`が使用された場合のパーティションプルーニング機能をサポート [#16521](https://github.com/pingcap/tidb/pull/16521)
    - `view`を更新せずに`view`を含む`update`ステートメントを実行を許可 [#16787](https://github.com/pingcap/tidb/pull/16787)
    - ネストされた`view`の作成を禁止 [#15424](https://github.com/pingcap/tidb/pull/15424)
    - `view`の切り捨てを禁止 [#16420](https://github.com/pingcap/tidb/pull/16420)
    - 列が`public`状態にない場合に、列の値を明示的に更新する`update`ステートメントの実行を禁止 [#15576](https://github.com/pingcap/tidb/pull/15576)
    - `status`ポートが占有されている場合にTiDBを起動することを禁止 [#15466](https://github.com/pingcap/tidb/pull/15466)
    - `current_role`関数の文字セットを`binary`から`utf8mb4`に変更 [#16083](https://github.com/pingcap/tidb/pull/16083)
    - リージョンのデータを読む際に中断シグナルを確認することで、`max-execution-time`の使いやすさを向上 [#15615](https://github.com/pingcap/tidb/pull/15615)
    - `auto_id`のキャッシュステップを明示的に設定するための`ALTER TABLE ... AUTO_ID_CACHE`構文を追加 [#16287](https://github.com/pingcap/tidb/pull/16287)

+ TiKV

    - 楽観的トランザクションで多くの競合と`BatchRollback`の条件が存在する場合のパフォーマンスを向上 [#7605](https://github.com/tikv/tikv/pull/7605)
    - 悲観的トランザクションで多くの競合が存在する場合に頻繁に覚醒する`pessimistic lock waiter`によってパフォーマンスが低下する問題を修正 [#7584](https://github.com/tikv/tikv/pull/7584)

+ Tools

    + TiDB Lightning

        - `fetch-mode`サブコマンドを使用してTiKVクラスターモードを印刷する機能をサポート [#287](https://github.com/pingcap/tidb-lightning/pull/287)

## バグ修正

+ TiDB

    - SQLモードが`ALLOW_INVALID_DATES`の場合に、`WEEKEND`関数がMySQLと互換性がない問題を修正 [#16170](https://github.com/pingcap/tidb/pull/16170)
    - インデックス列が自動増分プライマリキーを含む場合に、`DROP INDEX`ステートメントの実行に失敗する問題を修正 [#16008](https://github.com/pingcap/tidb/pull/16008)
    - ステートメントサマリ内の`TABLE_NAMES`列の値が不正確な問題を修正 [#15231](https://github.com/pingcap/tidb/pull/15231)
    - プランキャッシュが有効の場合に、一部の式が不正確な結果を持つ問題を修正 [#16184](https://github.com/pingcap/tidb/pull/16184)
    - `not`/`istrue`/`isfalse`関数の結果が不正確である問題を修正 [#15916](https://github.com/pingcap/tidb/pull/15916)
    - 冗長なインデックスを持つテーブルでの`MergeJoin`操作によって発生するパニックを修正 [#15919](https://github.com/pingcap/tidb/pull/15919)
    - 述語が外部テーブルを参照する場合にリンクを適切に単純化しない問題を修正 [#16492](https://github.com/pingcap/tidb/pull/16492)
    - `SET ROLE`ステートメントによって`CURRENT_ROLE`関数がエラーを報告する問題を修正 [#15569](https://github.com/pingcap/tidb/pull/15569)
    - `LOAD DATA`ステートメントの結果がMySQLと互換性がない問題を修正 [#16633](https://github.com/pingcap/tidb/pull/16633)
    - データベースの可視性がMySQLと互換性がない問題を修正 [#14939](https://github.com/pingcap/tidb/pull/14939)
    - `SET DEFAULT ROLE ALL`ステートメントの権限チェックが不正確な問題を修正 [#15585](https://github.com/pingcap/tidb/pull/15585)
    - プランキャッシュによって引き起こされるパーティションプルーニングの失敗の問題を修正 [#15818](https://github.com/pingcap/tidb/pull/15818)
    - テーブルに対して同時`DDL`操作を行い、ブロッキングが存在する場合にトランザクションのコミット中に`schema change`が報告される問題を修正 [#15707](https://github.com/pingcap/tidb/pull/15707)
    - `IF(not_int, *, *)`の不正確な動作を修正 [#15356](https://github.com/pingcap/tidb/pull/15356)
    - `CASE WHEN (not_int)`の不正確な動作を修正 [#15359](https://github.com/pingcap/tidb/pull/15359)
    - 現在のスキーマ内にない`view`を使用した際に`Unknown column`エラーメッセージが返される問題を修正 [#15866](https://github.com/pingcap/tidb/pull/15866)
    - 時刻文字列の解析結果がMySQLと互換性がない問題を修正 [#16242](https://github.com/pingcap/tidb/pull/16242)
```
    - `left join`で`null`カラムが右の子ノードに存在する場合の結合演算子の可能なパニックを修正する[#16528](https://github.com/pingcap/tidb/pull/16528)
    - TiKVが`StaleCommand`エラーメッセージを繰り返し返す際に、SQL実行がブロックされているときに、エラーメッセージが返されない問題を修正する[#16528](https://github.com/pingcap/tidb/pull/16528)
    - オーディットプラグインが有効な場合にポートプロービングによって発生する可能性のあるパニックを修正する[#15967](https://github.com/pingcap/tidb/pull/15967)
    - `fast analyze`がインデックスのみに対して動作した場合に発生する可能性のあるパニックを修正する[#15967](https://github.com/pingcap/tidb/pull/15967)
    - ある場合に`SELECT * FROM INFORMATION_SCHEMA.PROCESSLIST`ステートメントの実行における可能なパニックを修正する[#16309](https://github.com/pingcap/tidb/pull/16309)
    - ハッシュパーティションテーブルを作成する際にパーティション数を確認せずにメモリを割り当てることによるTiDBのOOMの問題を修正する[#16218](https://github.com/pingcap/tidb/pull/16218)
    - `information_schema.tidb_hot_table`におけるパーティションテーブルの不正確な情報の問題を修正する[#16726](https://github.com/pingcap/tidb/pull/16726)
    - ハッシュパーティションテーブルでのパーティション選択アルゴリズムが効果を発揮しない問題を修正する[#16070](https://github.com/pingcap/tidb/pull/16070)
    - MVCCシリーズのHTTP APIがパーティションテーブルをサポートしていない問題を修正する[#16191](https://github.com/pingcap/tidb/pull/16191)
    - `UNION`ステートメントのエラーハンドリングを`SELECT`ステートメントと一貫させる[#16137](https://github.com/pingcap/tidb/pull/16137)
    - `VALUES`関数のパラメータータイプが`bit(n)`の場合の動作が正しくない問題を修正する[#15486](https://github.com/pingcap/tidb/pull/15486)
    - `view`列名が長すぎる場合にTiDBの処理ロジックがMySQLと一貫していない問題を修正する。この場合、システムが自動的に短い列名を生成する[#14873](https://github.com/pingcap/tidb/pull/14873)
    - `(not not col)`が誤って`col`に最適化される問題を修正する[#16094](https://github.com/pingcap/tidb/pull/16094)
    - `IndexLookupJoin`プランによって構築された内部テーブルの`range`が正しくない問題を修正する[#15753](https://github.com/pingcap/tidb/pull/15753)
    - `only_full_group_by`が括弧を含む式を正しくチェックできない問題を修正する[#16012](https://github.com/pingcap/tidb/pull/16012)
    - `select view_name.col_name from view_name`ステートメントを実行する際にエラーが返される問題を修正する[#15572](https://github.com/pingcap/tidb/pull/15572)

+ TiKV

    - 分離回復後にノードが適切に削除されない問題を修正する[#7703](https://github.com/tikv/tikv/pull/7703)
    - リージョンマージ操作によるネットワーク分離によるデータ損失の問題を修正する[#7679](https://github.com/tikv/tikv/pull/7679)
    - 特定のケースでLearnerが適切に削除されない問題を修正する[#7598](https://github.com/tikv/tikv/pull/7598)
    - 生のキー値ペアのスキャン結果が順序外になる問題を修正する[#7597](https://github.com/tikv/tikv/pull/7597)
    - Raftメッセージのバッチが大きすぎる場合の再接続の問題を修正する[#7542](https://github.com/tikv/tikv/pull/7542)
    - 空のリクエストによるgRPCスレッドデッドロックの問題を修正する[#7538](https://github.com/tikv/tikv/pull/7538)
    - マージプロセス中のLearnerの再起動処理ロジックが不正確な問題を修正する[#7457](https://github.com/tikv/tikv/pull/7457)
    - ロックのクリーンアップに関する繰り返し要求がトランザクションの原子性を破壊する問題を修正する[#7388](https://github.com/tikv/tikv/pull/7388)
```