---
title: TiDB 2.1.16 リリースノート
aliases: ['/docs/dev/releases/release-2.1.16/','/docs/dev/releases/2.1.16/']
---

# TiDB 2.1.16 リリースノート

リリース日: 2019年8月15日

TiDB バージョン: 2.1.16

TiDB Ansible バージョン: 2.1.16

## TiDB

+ SQL オプティマイザ
    - 時間列の等しい条件に対して行数の推定が正確でない問題を修正 [#11526](https://github.com/pingcap/tidb/pull/11526)
    - `TIDB_INLJ` ヒントが効果がないか、指定されたテーブルに効果がある問題を修正 [#11361](https://github.com/pingcap/tidb/pull/11361)
    - 外部結合からアンチジョインへのクエリ内の `NOT EXISTS` の実装を変更し、より最適化された実行プランを見つけるために [#11291](https://github.com/pingcap/tidb/pull/11291)
    - `SHOW` 文内でサブクエリをサポートし、`SHOW COLUMNS FROM tbl WHERE FIELDS IN (SELECT 'a')` などの構文を可能にする [#11461](https://github.com/pingcap/tidb/pull/11461)
    - 定数折りたたみ最適化によって `SELECT … CASE WHEN … ELSE NULL ...` クエリが不正な結果を返す問題を修正 [#11441](https://github.com/pingcap/tidb/pull/11441)
+ SQL 実行エンジン
    - `INTERVAL` が負の場合に `DATE_ADD` 関数が誤った結果を返す問題を修正 [#11616](https://github.com/pingcap/tidb/pull/11616)
    - `FLOAT`、`DOUBLE`、または `DECIMAL` 型の引数を受け入れる際に型変換を誤って行い、`DATE_ADD` 関数が誤った結果を返す問題を修正 [#11628](https://github.com/pingcap/tidb/pull/11628)
    - CAST(JSON AS SIGNED) がオーバーフローした際に誤ったエラーメッセージを表示する問題を修正 [#11562](https://github.com/pingcap/tidb/pull/11562)
    - クエリ内のエグゼキュータを閉じる際、1 つの子ノードが閉じられないとエラーが返される問題を修正 [#11598](https://github.com/pingcap/tidb/pull/11598)
    - タイムアウト前にリージョン分散のスケジューリングが完了していない場合に、エラーではなく分割されたテーブルを返す `SPLIT TABLE` ステートメントをサポート [#11487](https://github.com/pingcap/tidb/pull/11487)
    - MySQL との互換性を持たせるために `REGEXP BINARY` 関数を大文字小文字を区別するよう修正 [#11505](https://github.com/pingcap/tidb/pull/11505)
    - `DATE_ADD`/`DATE_SUB` の結果で `YEAR` の値が 0 未満または 65535 より大きい場合に `NULL` が正しく返らない問題を修正 [#11477](https://github.com/pingcap/tidb/pull/11477)
    - 遅いクエリテーブルに実行が成功したかを示す `Succ` フィールドを追加 [#11412](https://github.com/pingcap/tidb/pull/11421)
    - SQL ステートメント内で現在のタイムスタンプを複数回取得する問題を修正し、MySQL との互換性を修正 [#11392](https://github.com/pingcap/tidb/pull/11392)
    - AUTO_INCREMENT 列が FLOAT または DOUBLE 型を扱わない問題を修正 [#11389](https://github.com/pingcap/tidb/pull/11389)
    - `CONVERT_TZ` 関数が無効な引数を受け入れた場合に `NULL` が正しく返らない問題を修正 [#11357](https://github.com/pingcap/tidb/pull/11357)
    - `PARTITION BY LIST` ステートメントでエラーが報告される問題を修正 (現在、構文のみをサポート; TiDB がステートメントを実行すると、通常のテーブルが作成され、プロンプトメッセージが提供される) [#11236](https://github.com/pingcap/tidb/pull/11236)
    - 小数点以下が多い場合に MySQL と一致しない `Mod(%)`、`Multiple(*)`、および `Minus(-)` 演算が不一貫な `0` 結果を返す問題を修正 (例: `select 0.000 % 0.11234500000000000000`) [#11353](https://github.com/pingcap/tidb/pull/11353)
+ サーバー
    - `OnInit` がコールバックされるときにプラグインが `NULL` ドメインを取得する問題を修正 [#11426](https://github.com/pingcap/tidb/pull/11426)
    - スキーマ内のテーブル情報が削除された後も HTTP インターフェースを通じて取得できる問題を修正した後に生成されたテーブル情報を [#11586](https://github.com/pingcap/tidb/pull/11586)
+ DDL
    - この操作によって自動増分列の誤った結果が生じることを避けるために、自動増分列のインデックスを削除することを禁止する [#11402](https://github.com/pingcap/tidb/pull/11402)
    - 異なる文字セットと照合順序を使用してテーブルを作成し変更する際に、列の文字セットが正しくない問題を修正 [#11423](https://github.com/pingcap/tidb/pull/11423)
    - `alter table ... set default...` とこの列を修正する他の DDL ステートメントが並行して実行されると、列のスキーマが誤ってしまう問題を修正 [#11374](https://github.com/pingcap/tidb/pull/11374)
    - 生成列 A が生成列 B に依存し、A がインデックスの作成に使用される場合にデータがバックフィルされない問題を修正 [#11538](https://github.com/pingcap/tidb/pull/11538)
    - `ADMIN CHECK TABLE` 操作の処理を高速化 [#11538](https://github.com/pingcap/tidb/pull/11676)

## TiKV

+ クライアントがクローズされている TiKV リージョンにアクセスした際にエラーメッセージを返すサポートを追加 [#4820](https://github.com/tikv/tikv/pull/4820)
+ 逆向きの `raw_scan` および `raw_batch_scan` インタフェースのサポートを追加 [#5148](https://github.com/tikv/tikv/pull/5148)

## Tools

+ TiDB ビンロッグ
    - トランザクション内の一部ステートメントの実行をスキップするための `ignore-txn-commit-ts` 設定項目を Drainer に追加 [#697](https://github.com/pingcap/tidb-binlog/pull/697)
    - 起動時に構成項目の確認を行い、無効な構成項目に遭遇した場合は Pump および Drainer の実行を停止し、エラーメッセージを返す構成項目を追加 [#708](https://github.com/pingcap/tidb-binlog/pull/708)
    - Drainer でのノード ID を指定するための `node-id` 設定を追加 [#706](https://github.com/pingcap/tidb-binlog/pull/706)
+ TiDB Lightning
    - `tikv_gc_life_time` が同時に 2 つのチェックサムが実行されているときに元の値に戻らない問題を修正 [#224](https://github.com/pingcap/tidb-lightning/pull/224)

## TiDB Ansible

+ Spark に `log4j` 構成ファイルを追加 [#842](https://github.com/pingcap/tidb-ansible/pull/842)
+ tispark jar パッケージを v2.1.2 に更新 [#863](https://github.com/pingcap/tidb-ansible/pull/863)
+ TiDB Binlog が Kafka または ZooKeeper を使用する際に Prometheus 構成ファイルが間違った形式で生成される問題を修正 [#845](https://github.com/pingcap/tidb-ansible/pull/845)
+ `rolling_update.yml` 操作を実行する際に PD がリーダーを切り替えられないバグを修正 [#888](https://github.com/pingcap/tidb-ansible/pull/888)
+ PD ノードをローリングアップデートするロジックを最適化し、Follower を先にアップグレードし、その後リーダーをアップグレードすることで安定性を向上させる [#895](https://github.com/pingcap/tidb-ansible/pull/895)