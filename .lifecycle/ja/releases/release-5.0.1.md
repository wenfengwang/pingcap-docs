---
title: TiDB 5.0.1 リリースノート
---

# TiDB 5.0.1 リリースノート

リリース日: 2021年4月24日

TiDB バージョン: 5.0.1

## 互換性の変更

- `committer-concurrency` 設定項目のデフォルト値は `16` から `128` に変更されました。

## 改善点

+ TiDB

    - 組み込み関数 `VITESS_HASH()` をサポート [#23915](https://github.com/pingcap/tidb/pull/23915)

+ TiKV

    - リージョンスナップショットの圧縮に `zstd` を使用するようになりました [#10005](https://github.com/tikv/tikv/pull/10005)

+ PD

    - 異様なストアにより適切に満たすために、リージョンスコア計算機を変更しました [#3605](https://github.com/pingcap/pd/pull/3605)
    - `scatter region` スケジューラを追加した後に予期しない統計が発生するのを回避しました [#3602](https://github.com/pingcap/pd/pull/3602)

+ ツール

    + バックアップ & リストア (BR)

        - サマリーログから誤解を招く情報を削除しました [#1009](https://github.com/pingcap/br/pull/1009)

## バグ修正

+ TiDB

    - プロジェクトの結果が空の場合にプロジェクトの除外の実行結果が誤っている問題を修正しました [#24093](https://github.com/pingcap/tidb/pull/24093)
    - 特定のケースでカラムが `NULL` 値を含む場合にクエリ結果が誤っている問題を修正しました [#24063](https://github.com/pingcap/tidb/pull/24063)
    - スキャンに仮想カラムが含まれる場合に MPP プランの生成を禁止するようにしました [#24058](https://github.com/pingcap/tidb/pull/24058)
    - プランキャッシュで `PointGet` と `TableDual` が誤って再利用される問題を修正しました [#24043](https://github.com/pingcap/tidb/pull/24043)
    - クラスター化されたインデックスに対して `IndexMerge` プランを最適化する際にオプティマイザーがエラーを発生させる問題を修正しました [#24042](https://github.com/pingcap/tidb/pull/24042)
    - BIT型の型推論エラーを修正しました [#24027](https://github.com/pingcap/tidb/pull/24027)
    - `PointGet` オペレータが存在する場合に一部のオプティマイザーヒントが効果を発揮しない問題を修正しました [#23685](https://github.com/pingcap/tidb/pull/23685)
    - エラー発生時に DDL操作がロールバックに失敗する可能性のある問題を修正しました [#24080](https://github.com/pingcap/tidb/pull/24080)
    - バイナリリテラル定数のインデックス範囲が誤って構築される問題を修正しました [#24041](https://github.com/pingcap/tidb/pull/24041)
    - 特定の場合において `IN` 句の結果が誤っている可能性のある問題を修正しました [#24023](https://github.com/pingcap/tidb/pull/24023)
    - 一部の文字列関数の誤った結果を修正しました [#23879](https://github.com/pingcap/tidb/pull/23879)
    - `REPLACE` 操作を実行するためにユーザーはテーブルに対して `INSERT` と `DELETE` の権限を両方とも必要とするようにしました [#23939](https://github.com/pingcap/tidb/pull/23939)
    - ポイントクエリの実行においてパフォーマンスの低下を修正しました [#24070](https://github.com/pingcap/tidb/pull/24070)
    - バイナリとバイトを誤って比較することで誤った `TableDual` プランを修正しました [#23918](https://github.com/pingcap/tidb/pull/23918)

+ TiKV

    - コプロセッサが `IN` 式で符号付きまたは符号なし整数型を適切に処理できない問題を修正しました [#10018](https://github.com/tikv/tikv/pull/10018)
    - SSTファイルをバッチ取り込みした後に多くの空のリージョンが発生する問題を修正しました [#10015](https://github.com/tikv/tikv/pull/10015)
    - `cast_string_as_time` の入力が無効なUTF-8バイトの場合に発生する潜在的なパニックを修正しました [#9995](https://github.com/tikv/tikv/pull/9995)
    - ファイルディレクトリが損傷した後に TiKV が起動できなくなるバグを修正しました [#9992](https://github.com/tikv/tikv/pull/9992)

+ TiFlash

    - ストレージエンジンが一部の範囲のデータを削除できない問題を修正しました
    - 時間型を整数型にキャストする際に誤った結果が発生する問題を修正しました
    - `receiver` が10秒以内に対応するタスクを見つけられないバグを修正しました
    - `cancelMPPQuery` に無効なイテレーターが存在する可能性のある問題を修正しました
    - `bitwise` オペレーターの動作が TiDB と異なる場合に発生する問題を修正しました
    - `prefix key` を使用した場合に重複する範囲によって引き起こされる警告の問題を修正しました
    - 文字列型を整数型にキャストする際に誤った結果が発生する問題を修正しました
    - 連続して高速な書き込みがTiFlashをメモリ不足に陥らせる問題を修正しました
    - 重複する列名がTiFlashがエラーを発生させる問題を修正しました
    - TiFlashがMPPプランを解析できない問題を修正しました
    - テーブルGC中にNULLポインタの例外が発生する可能性のある問題を修正しました
    - ドロップされたテーブルにデータを書き込む際にTiFlashがパニックする問題を修正しました
    - BRリストア時にデータを書き込む際にTiFlashがパニックする可能性のある問題を修正しました

+ ツール

    + TiDB Lightning

        - インポート中の進行ログで不正確なテーブル数の問題を修正しました [#1005](https://github.com/pingcap/br/pull/1005)

    + バックアップ & リストア (BR)

        - 実際のバックアップ速度が `--ratelimit` 制限を超えてしまうバグを修正しました [#1026](https://github.com/pingcap/br/pull/1026)
        - 一部のTiKVノードの障害によってバックアップが中断される問題を修正しました [#1019](https://github.com/pingcap/br/pull/1019)
        - TiDB Lightningのインポート中の進行ログで不正確なテーブル数の問題を修正しました [#1005](https://github.com/pingcap/br/pull/1005)

    + TiCDC

        - 統一ソーターの並行処理の問題を修正し、役に立たないエラーメッセージをフィルタリングしました [#1678](https://github.com/pingcap/tiflow/pull/1678)
        - 冗長なディレクトリの作成がMinIOとのレプリケーションを中断させる可能性があるバグを修正しました [#1672](https://github.com/pingcap/tiflow/pull/1672)
        - MySQL 5.7 ダウンストリームが上流TiDBと同じ動作を維持するように、`explicit_defaults_for_timestamp` セッション変数のデフォルト値を `ON` に設定しました [#1659](https://github.com/pingcap/tiflow/pull/1659)
        - `io.EOF` の不正確な処理がレプリケーションの中断を引き起こす可能性がある問題を修正しました [#1648](https://github.com/pingcap/tiflow/pull/1648)
        - TiKV CDCエンドポイントCPUメトリクスがTiCDCダッシュボードで正しくない問題を修正しました [#1645](https://github.com/pingcap/tiflow/pull/1645)
        - 特定の場合においてレプリケーションがブロックされるのを回避するために `defaultBufferChanSize` を増やしました [#1632](https://github.com/pingcap/tiflow/pull/1632)