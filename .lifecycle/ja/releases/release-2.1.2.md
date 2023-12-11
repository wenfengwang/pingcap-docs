---
title: TiDB 2.1.2 リリースノート
aliases: ['/docs/dev/releases/release-2.1.2/','/docs/dev/releases/2.1.2/']
---

# TiDB 2.1.2 リリースノート

2018年12月22日、TiDB 2.1.2 がリリースされました。それに伴い、対応する TiDB Ansible 2.1.2 もリリースされました。このリリースは、TiDB 2.1.1 と比較して、システムの互換性と安定性が大幅に向上しています。

## TiDB

- TiDB を Kafka バージョンの TiDB Binlog と互換性があるようにする [#8747](https://github.com/pingcap/tidb/pull/8747)
- ローリングアップデート中の TiDB の終了メカニズムを改善する [#8707](https://github.com/pingcap/tidb/pull/8707)
- 一部のケースで生成された列にインデックスを追加することによって発生するパニックの問題を修正する [#8676](https://github.com/pingcap/tidb/pull/8676)
- SQL ステートメントに`TIDB_SMJ Hint` が存在する場合に、オプティマイザが最適なクエリプランを見つけることができない問題を修正する [#8729](https://github.com/pingcap/tidb/pull/8729)
- 一部のケースで`AntiSemiJoin` が誤った結果を返す問題を修正する [#8730](https://github.com/pingcap/tidb/pull/8730)
- `utf8` 文字セットの有効な文字のチェックを改善する [#8754](https://github.com/pingcap/tidb/pull/8754)
- トランザクション内で書き込み操作が読み込み操作の前に実行された場合に、時刻の型のフィールドが誤った結果を返す可能性のある問題を修正する [#8746](https://github.com/pingcap/tidb/pull/8746)

## PD

- リージョンのマージに関するリージョン情報の更新問題を修正する [#1377](https://github.com/pingcap/pd/pull/1377)

## TiKV

- `DAY` (`d`) の単位での構成フォーマットをサポートし、構成の互換性の問題を修正する [#3931](https://github.com/tikv/tikv/pull/3931)
- `Approximate Size Split` によって引き起こされる可能性のあるパニックの問題を修正する [#3942](https://github.com/tikv/tikv/pull/3942)
- 2 つのリージョンマージに関する問題を修正する [#3822](https://github.com/tikv/tikv/pull/3822), [#3873](https://github.com/tikv/tikv/pull/3873)

## Tools

+ TiDB Lightning
    - Lightning がサポートする最小のクラスターバージョンを TiDB 2.1.0 にする
    - Lightning で解析された`JSON`データを含むファイルのコンテンツエラーを修正する [#144](https://github.com/pingcap/tidb-tools/issues/144)
    - チェックポイントを使用して Lightning を再起動した後に`Too many open engines`が発生する問題を修正する
+ TiDB Binlog
    - Drainer が Kafka にデータを書き込むいくつかのボトルネックを除去する
    - TiDB Binlog の Kafka バージョンをサポートする