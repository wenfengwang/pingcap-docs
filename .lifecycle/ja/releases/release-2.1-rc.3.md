---
title: TiDB 2.1 RC3のリリースノート
aliases: ['/docs/dev/releases/release-2.1-rc.3/','/docs/dev/releases/21rc3/']
---

# TiDB 2.1 RC3のリリースノート

2018年9月29日にTiDB 2.1 RC3がリリースされました。このリリースでは、TiDB 2.1 RC2と比べて、安定性、互換性、SQLオプティマイザー、および実行エンジンの大幅な改善が行われています。

## TiDB

+ SQLオプティマイザー
    - ステートメントが埋め込まれた`LEFT OUTER JOIN`を含む場合に不正な結果が発生する問題を修正 [#7689](https://github.com/pingcap/tidb/pull/7689)
    - `JOIN`ステートメントにおけるプレディケートプッシュダウンの最適化ルールを強化 [#7645](https://github.com/pingcap/tidb/pull/7645)
    - `UnionScan`演算子に対するプレディケートプッシュダウンの最適化ルールを修正 [#7695](https://github.com/pingcap/tidb/pull/7695)
    - `Union`演算子の重複キー特性が正しく設定されていない問題を修正 [#7680](https://github.com/pingcap/tidb/pull/7680)
    - 定数折りたたみの最適化ルールを強化 [#7696](https://github.com/pingcap/tidb/pull/7696)
    - テーブルダブルに伝播した後、フィルターがヌルであるデータソースを最適化 [#7756](https://github.com/pingcap/tidb/pull/7756)
+ SQL実行エンジン
    - トランザクション内の読み取りリクエストのパフォーマンスを最適化 [#7717](https://github.com/pingcap/tidb/pull/7717)
    - 一部の実行エンジンでのチャンクメモリの割り当てコストを最適化 [#7540](https://github.com/pingcap/tidb/pull/7540)
    - ポイントクエリがすべてNULL値を取得する列によって引き起こされる「範囲外のインデックス」パニックを修正 [#7790](https://github.com/pingcap/tidb/pull/7790)
+ サーバー
    - 設定ファイルのメモリクォータが効果を上げない問題を修正 [#7729](https://github.com/pingcap/tidb/pull/7729)
    - 各ステートメントの実行優先度を設定する`tidb_force_priority`システム変数を追加 [#7694](https://github.com/pingcap/tidb/pull/7694)
    - 遅いクエリログを取得するための`admin show slow`ステートメントのサポートを追加 [#7785](https://github.com/pingcap/tidb/pull/7785)
+ 互換性
    - `information_schema.schemata`における`charset/collation`の結果が正しくない問題を修正 [#7751](https://github.com/pingcap/tidb/pull/7751)
    - `hostname`システム変数の値が空である問題を修正 [#7750](https://github.com/pingcap/tidb/pull/7750)
+ 式
    - `AES_ENCRYPT`/`AES_DECRYPT`ビルトイン関数における`init_vecter`引数のサポートを追加 [#7425](https://github.com/pingcap/tidb/pull/7425)
    - 一部の式における`Format`の結果が正しくない問題を修正 [#7770](https://github.com/pingcap/tidb/pull/7770)
    - `JSON_LENGTH`ビルトイン関数のサポートを追加 [#7739](https://github.com/pingcap/tidb/pull/7739)
    - 符号なし整数型を10進数型にキャストした際の不正な結果の問題を修正 [#7792](https://github.com/pingcap/tidb/pull/7792)
+ DML
    - ユニークキーを更新する際に`INSERT … ON DUPLICATE KEY UPDATE`ステートメントの結果が正しくない問題を修正 [#7675](https://github.com/pingcap/tidb/pull/7675)
+ DDL
    - タイムスタンプ型の新しい列に新しいインデックスを作成する際に、インデックス値がタイムゾーン間で変換されない問題を修正 [#7724](https://github.com/pingcap/tidb/pull/7724)
    - enum型に新しい値を追加することをサポート [#7767](https://github.com/pingcap/tidb/pull/7767)
    - ネットワーク分離後のクラスターの可用性を向上させるために、etcdセッションの作成を迅速に行う機能をサポート [#7774](https://github.com/pingcap/tidb/pull/7774)

## PD

+ 新機能
    - サイズで逆順にリージョンリストを取得するAPIを追加 [#1254](https://github.com/pingcap/pd/pull/1254)
+ 改善
    - リージョンAPIでより詳細な情報を返すようにする [#1252](https://github.com/pingcap/pd/pull/1252)
+ バグ修正
    - PDがリーダを切り替えた後に`adjacent-region-scheduler`がクラッシュを引き起こす可能性がある問題を修正 [#1250](https://github.com/pingcap/pd/pull/1250)

## TiKV

+ パフォーマンス
    - コプロセッサリクエストの並行処理を最適化 [#3515](https://github.com/tikv/tikv/pull/3515)
+ 新機能
    - ログ関数のサポートを追加 [#3603](https://github.com/tikv/tikv/pull/3603)
    - `sha1`関数のサポートを追加 [#3612](https://github.com/tikv/tikv/pull/3612)
    - `truncate_int`関数のサポートを追加 [#3532](https://github.com/tikv/tikv/pull/3532)
    - `year`関数のサポートを追加 [#3622](https://github.com/tikv/tikv/pull/3622)
    - `truncate_real`関数のサポートを追加 [#3633](https://github.com/tikv/tikv/pull/3633)
+ バグ修正
    - 時間関数に関連する報告エラー動作を修正 [#3487](https://github.com/tikv/tikv/pull/3487), [#3615](https://github.com/tikv/tikv/pull/3615)
    - 文字列から解析された時間がTiDBでの時間と一貫していない問題を修正 [#3589](https://github.com/tikv/tikv/pull/3589)