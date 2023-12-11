---
title: TiDB 2.1.4のリリースノート
aliases: ['/docs/dev/releases/release-2.1.4/','/docs/dev/releases/2.1.4/']
---

# TiDB 2.1.4リリースノート

2019年2月15日、TiDB 2.1.4がリリースされました。それに伴い、TiDB Ansible 2.1.4もリリースされました。このリリースでは、TiDB 2.1.3に比べて、安定性、SQLオプティマイザ、統計情報、実行エンジンが大幅に改善されました。

## TiDB

+ SQLオプティマイザ/実行エンジン
    - `VALUES` 関数がFLOAT型を適切に処理しない問題を修正しました [#9223](https://github.com/pingcap/tidb/pull/9223)
    - 一部の場合において、FloatをStringにキャストする際の誤った結果が修正されました [#9227](https://github.com/pingcap/tidb/pull/9227)
    - 一部の場合において、`FORMAT` 関数の誤った結果が修正されました [#9235](https://github.com/pingcap/tidb/pull/9235)
    - 一部の場合において、Joinクエリの処理中にpanicが発生する問題が修正されました [#9264](https://github.com/pingcap/tidb/pull/9264)
    - `VALUES` 関数がENUM型を適切に処理しない問題が修正されました [#9280](https://github.com/pingcap/tidb/pull/9280)
    - `DATE_ADD`/`DATE_SUB` の誤った結果が一部の場合において修正されました [#9284](https://github.com/pingcap/tidb/pull/9284)
+ サーバ
    - 「権限の再読み込みに成功」のログを最適化し、DEBUGレベルに変更しました [#9274](https://github.com/pingcap/tidb/pull/9274)
+ DDL
    - `tidb_ddl_reorg_worker_cnt` と `tidb_ddl_reorg_batch_size` をグローバル変数に変更しました [#9134](https://github.com/pingcap/tidb/pull/9134)
    - 異常な条件下で生成列にインデックスを追加することによって発生するバグが修正されました [#9289](https://github.com/pingcap/tidb/pull/9289)

## TiKV

- TiKVを終了する際の重複書き込みの問題が修正されました [#4146](https://github.com/tikv/tikv/pull/4146)
- 一部の場合において、イベントリスナーの異常な結果の問題が修正されました [#4132](https://github.com/tikv/tikv/pull/4132)

## ツール

+ Lightning
    - メモリ使用量が最適化されました [#107](https://github.com/pingcap/tidb-lightning/pull/107), [#108](https://github.com/pingcap/tidb-lightning/pull/108)
    - ダンプファイルのチャンク分割が削除され、余分なダンプファイルの解析を回避するようになりました [#109](https://github.com/pingcap/tidb-lightning/pull/109)
    - 多くのキャッシュミスによって引き起こされるパフォーマンスの劣化を避けるために、ダンプファイルの読み取りのI/O並行性が制限されるようになりました [#110](https://github.com/pingcap/tidb-lightning/pull/110)
    - 個々のテーブルのデータをバッチでインポートすることで、インポートの安定性が向上するようになりました [#110](https://github.com/pingcap/tidb-lightning/pull/113)
    - TiKVでインポートモードで自動コンパクションが有効になるようになりました [#4199](https://github.com/tikv/tikv/pull/4199)
    - インポーターディスク領域を過度に消費するのを避けるために、インポートエンジンの数が制限されるようになりました [#119](https://github.com/pingcap/tidb-lightning/pull/119)
+ Sync-diff-inspectorでTiDBの統計情報を使用してチャンクを分割するサポートが追加されました [#197](https://github.com/pingcap/tidb-tools/pull/197)