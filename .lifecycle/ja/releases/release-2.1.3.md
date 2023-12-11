---
title: TiDB 2.1.3リリースノート
aliases: ['/docs/dev/releases/release-2.1.3/','/docs/dev/releases/2.1.3/']
---

# TiDB 2.1.3リリースノート

2019年1月28日、TiDB 2.1.3がリリースされました。対応するTiDB Ansible 2.1.3もリリースされました。このリリースでは、TiDB 2.1.2と比較して、システムの安定性、SQLオプティマイザ、統計情報、実行エンジンに大幅な改善がされています。

## TiDB

+ SQLオプティマイザ/エグゼキューター
    - 準備されたプランキャッシュのパニック問題をいくつかのケースで修正 [#8826](https://github.com/pingcap/tidb/pull/8826)
    - インデックスが接頭辞インデックスの場合、レンジ計算が間違っている問題を修正 [#8851](https://github.com/pingcap/tidb/pull/8851)
    - `SQL_MODE`が厳密でない場合、文字列が不正な`TIME`形式の場合、`CAST(str AS TIME(N))`がnullを返すようにする [#8966](https://github.com/pingcap/tidb/pull/8966)
    - いくつかのケースで`UPDATE`の処理中に生成されたカラムのパニック問題を修正 [#8980](https://github.com/pingcap/tidb/pull/8980)
    - いくつかのケースで統計ヒストグラムの上限オーバーフロー問題を修正 [#8989](https://github.com/pingcap/tidb/pull/8989)
    - `_tidb_rowid`構築クエリでのレンジのサポート。全体テーブルスキャンを避け、クラスター負荷を軽減するため [#9059](https://github.com/pingcap/tidb/pull/9059)
    - `CAST(AS TIME)`の精度が大きすぎる場合、エラーを返すようにする [#9058](https://github.com/pingcap/tidb/pull/9058)
    - カルテシアン積で`Sort Merge Join`を使用することを許可する [#9037](https://github.com/pingcap/tidb/pull/9037)
    - いくつかのケースで統計ワーカーがパニック後に再開できない問題を修正 [#9085](https://github.com/pingcap/tidb/pull/9085)
    - いくつかのケースで`Sort Merge Join`が間違った結果を返す問題を修正 [#9046](https://github.com/pingcap/tidb/pull/9046)
    - `CASE`句でJSON型を返すことをサポートする [#8355](https://github.com/pingcap/tidb/pull/8355)
+ サーバー
    - コメントに非TiDBヒントが存在する場合、エラーの代わりに警告を返すようにする [#8766](https://github.com/pingcap/tidb/pull/8766)
    - 設定されたTIMEZONE値の妥当性を検証する [#8879](https://github.com/pingcap/tidb/pull/8879)
    - `QueryDurationHistogram`メトリクス項目を最適化し、より多くのステートメントタイプを表示する [#8875](https://github.com/pingcap/tidb/pull/8875)
    - いくつかのケースでbigintの下限オーバーフロー問題を修正 [#8544](https://github.com/pingcap/tidb/pull/8544)
    - `ALLOW_INVALID_DATES` SQLモードをサポートする [#9110](https://github.com/pingcap/tidb/pull/9110)
+ DDL
    - `RENAME TABLE`互換性の問題を修正し、MySQLとの動作を一貫させるようにする [#8808](https://github.com/pingcap/tidb/pull/8808)
    - `ADD INDEX`の同時変更を即座に有効にするようにサポートする [#8786](https://github.com/pingcap/tidb/pull/8786)
    - いくつかのケースで`ADD COLUMN`の処理中に`UPDATE`のパニック問題を修正 [#8906](https://github.com/pingcap/tidb/pull/8906)
    - いくつかのケースでTable Partitionの作成を同時に行う問題を修正 [#8902](https://github.com/pingcap/tidb/pull/8902)
    - `utf8`文字セットを`utf8mb4`に変換するサポートを追加 [#8951](https://github.com/pingcap/tidb/pull/8951) [#9152](https://github.com/pingcap/tidb/pull/9152)
    - Shard Bitsのオーバーフロー問題を修正する [#8976](https://github.com/pingcap/tidb/pull/8976)
    - `SHOW CREATE TABLE`で列文字セットを出力するサポートを追加 [#9053](https://github.com/pingcap/tidb/pull/9053)
    - `utf8mb4`でのvarchar型列の最大長制限の問題を修正する [#8818](https://github.com/pingcap/tidb/pull/8818)
    - `ALTER TABLE TRUNCATE TABLE PARTITION`をサポートする [#9093](https://github.com/pingcap/tidb/pull/9093)
    - 文字セットが指定されていない場合に文字セットを解決する [#9147](https://github.com/pingcap/tidb/pull/9147)

## PD

- リーダー選出に関連するWatchの問題を修正する [#1396](https://github.com/pingcap/pd/pull/1396)

## TiKV

- HTTPメソッドを使用して監視情報を取得するサポートを追加する [#3855](https://github.com/tikv/tikv/pull/3855)
- `data_format`のNULL問題を修正する [#4075](https://github.com/tikv/tikv/pull/4075)
- スキャンリクエストの範囲を検証するサポートを追加する [#4124](https://github.com/tikv/tikv/pull/4124)

## Tools

+ TiDB Binlog
    - TiDBの開始時または再開時に`no available pump`の問題を修正する [#157](https://github.com/pingcap/tidb-tools/pull/158)
    - Pumpクライアントログの出力を有効にする [#165](https://github.com/pingcap/tidb-tools/pull/165)
    - ユニークキーがNULL値を含み、テーブルにプライマリキーがなく、データの不整合が発生する問題を修正