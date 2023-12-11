---
title: TiDB 2.0.5 リリースノート
aliases: ['/docs/dev/releases/release-2.0.5/','/docs/dev/releases/205/']
---

# TiDB 2.0.5 リリースノート

2018年7月6日、TiDB 2.0.5 がリリースされました。TiDB 2.0.4 と比較して、このリリースではシステムの互換性と安定性に大幅な改善がされました。

## TiDB

- 新機能
    - `tidb_disable_txn_auto_retry` システム変数を追加し、トランザクションの自動リトライを無効にすることができます [#6877](https://github.com/pingcap/tidb/pull/6877)
- 改善点
    - `Selection` のコスト計算を最適化して、結果をより正確にする [#6989](https://github.com/pingcap/tidb/pull/6989)
    - ユニークインデックスまたはプライマリキーと完全に一致するクエリ条件をクエリパスとして直接選択 [#6966](https://github.com/pingcap/tidb/pull/6966)
    - サービスの起動に失敗した場合、必要なクリーンアップを実行する [#6964](https://github.com/pingcap/tidb/pull/6964)
    - `Load Data` ステートメントで `\N` を NULL として処理する [#6962](https://github.com/pingcap/tidb/pull/6962)
    - CBO のコード構造を最適化する [#6953](https://github.com/pingcap/tidb/pull/6953)
    - サービスの起動時に監視メトリクスを早めに報告する [#6931](https://github.com/pingcap/tidb/pull/6931)
    - SQL ステートメント内の改行を削除し、ユーザー情報を追加して、遅いクエリのフォーマットを最適化する [#6920](https://github.com/pingcap/tidb/pull/6920)
    - コメントで複数のアスタリスクをサポートする [#6858](https://github.com/pingcap/tidb/pull/6858)
- バグ修正
    - `KILL QUERY` が常に SUPER 権限を要求する問題を修正する [#7003](https://github.com/pingcap/tidb/pull/7003)
    - ユーザー数が1024を超えると、ユーザーがログインに失敗する問題を修正する [#6986](https://github.com/pingcap/tidb/pull/6986)
    - 符号なし `float`/`double` データの挿入に関する問題を修正する [#6940](https://github.com/pingcap/tidb/pull/6940)
    - `COM_FIELD_LIST` コマンドの互換性を修正し、一部のMariaDBクライアントでパニックが発生する問題を解決する [#6929](https://github.com/pingcap/tidb/pull/6929)
    - `CREATE TABLE IF NOT EXISTS LIKE` の動作に関する問題を修正する [#6928](https://github.com/pingcap/tidb/pull/6928)
    - TopN pushdown のプロセスで発生する問題を修正する [#6923](https://github.com/pingcap/tidb/pull/6923)
    - `Add Index` の実行中にエラーが発生した場合の、現在処理中の行のIDレコードの問題を修正する [#6903](https://github.com/pingcap/tidb/pull/6903)

## PD

- 一部シナリオでレプリカの移行がTiKVディスクスペースを消費する問題を修正する
- `AdjacentRegionScheduler` によって引き起こされるクラッシュの問題を修正する

## TiKV

- 10進数演算での潜在的なオーバーフロー問題を修正する
- 統合のプロセスで発生するダーティリードの問題を修正する 
