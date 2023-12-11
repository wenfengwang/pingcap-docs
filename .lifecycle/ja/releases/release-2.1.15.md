---
title: TiDB 2.1.15のリリースノート
aliases: ['/docs/dev/releases/release-2.1.15/','/docs/dev/releases/2.1.15/']
---

# TiDB 2.1.15リリースノート

リリース日: 2019年7月16日

TiDBバージョン: 2.1.15

TiDB Ansibleバージョン: 2.1.15

## TiDB

+ `DATE_ADD`関数がマイクロ秒の処理時に間違った結果を返す問題を修正 [#11289](https://github.com/pingcap/tidb/pull/11289)
+ 空の値が含まれる文字列カラムと`FLOAT`または`INT`が比較された際にエラーが報告される問題を修正 [#11279](https://github.com/pingcap/tidb/pull/11279)
+ パラメーターが`NULL`の場合に`INSERT`関数が正しく`NULL`値を返さない問題を修正 [#11249](https://github.com/pingcap/tidb/pull/11249)
+ 非文字列型および`0`長さのカラムのインデクシング時にエラーが発生する問題を修正 [#11215](https://github.com/pingcap/tidb/pull/11215)
+ SQLステートメントを使用してテーブルのRegion分布をクエリするための`SHOW TABLE REGIONS`ステートメントを追加 [#11238](https://github.com/pingcap/tidb/pull/11238)
+ `UPDATE … SELECT`ステートメントを使用した際にエラーが報告される問題を修正。`SELECT`サブクエリ内の最適化ルールにより、プロジェクションの消去が使用されていたため [#11254](https://github.com/pingcap/tidb/pull/11254)
+ プラグインを動的に有効または無効にするための`ADMIN PLUGINS ENABLE`/`ADMIN PLUGINS DISABLE` SQLステートメントを追加 [#11189](https://github.com/pingcap/tidb/pull/11189)
+ オーディットプラグインにセッション接続情報を追加 [#11189](https://github.com/pingcap/tidb/pull/11189)
+ ポイントクエリ時に複数回列が問い合わせられ、返された結果が`NULL`の際に発生するパニック問題を修正 [#11227](https://github.com/pingcap/tidb/pull/11227)
+ テーブル作成時にテーブルRegionを乱雑化するための`tidb_scatter_region`構成項目を追加 [#11213](https://github.com/pingcap/tidb/pull/11213)
+ `RAND`関数を使用する際に非スレッドセーフな`rand.Rand`によって引き起こされるデータ競合問題を修正 [#11170](https://github.com/pingcap/tidb/pull/11170)
+ 整数と非整数の比較結果が一部のケースで不正確である問題を修正 [#11191](https://github.com/pingcap/tidb/pull/11191)
+ データベースまたはテーブルの照合順序を変更するサポートを追加。ただし、データベース/テーブルの文字セットはUTF-8またはutf8mb4である必要がある [#11085](https://github.com/pingcap/tidb/pull/11085)
+ `SHOW CREATE TABLE`ステートメントによって示される精度が、デフォルト値として`CURRENT_TIMESTAMP`が使用され、浮動小数点精度が指定された場合に不完全である問題を修正 [#11087](https://github.com/pingcap/tidb/pull/11087)

## TiKV

+ ログ形式を統一 [#5083](https://github.com/tikv/tikv/pull/5083)
+ 極端なケースでRegionのおおよそのサイズまたはキーの精度を向上させ、スケジューリングの精度を向上させるために改善 [#5085](https://github.com/tikv/tikv/pull/5085)

## PD

+ ログ形式を統一 [#1625](https://github.com/pingcap/pd/pull/1625)

## Tool

TiDB Binlog

+ ポンプGC戦略を最適化し、未利用のバイナリログを長期間占有しないように制限を解除し、リソースが長期間占有されないようにするために改善 [#663](https://github.com/pingcap/tidb-binlog/pull/663)

TiDB Lightning

+ SQLダンプで指定された列名が小文字でない場合に発生するインポートエラーを修正 [#210](https://github.com/pingcap/tidb-lightning/pull/210)

## TiDB Ansible

+ SQLステートメントの解析とコンパイルにかかる時間をモニタリングするためにTiDBダッシュボードに`parse duration`および`compile duration`の監視項目を追加 [#815](https://github.com/pingcap/tidb-ansible/pull/815)