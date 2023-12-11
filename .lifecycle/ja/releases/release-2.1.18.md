---
title: TiDB 2.1.18 リリースノート
aliases: ['/docs/dev/releases/release-2.1.18/','/docs/dev/releases/2.1.18/']
---

# TiDB 2.1.18 リリースノート

リリース日: 2019年11月4日

TiDB バージョン: 2.1.18

TiDB Ansible バージョン: 2.1.18

## TiDB

+ SQL オプティマイザ
    - フィードバックによって区切られた際に無効なクエリ範囲が表示される問題を修正 [#12172](https://github.com/pingcap/tidb/pull/12172)
    - ポイント取得プランにおいて特権チェックが間違っている問題を修正 [#12341](https://github.com/pingcap/tidb/pull/12341)
    - `select ... limit ... offset …` ステートメントの実行パフォーマンスを最適化し、`Limit` 演算子を `IndexLookUpReader` の実行ロジックに配下げることをサポート [#12380](https://github.com/pingcap/tidb/pull/12380)
    - `ORDER BY`、`GROUP BY`、`LIMIT OFFSET` でパラメータを使用することをサポート [#12514](https://github.com/pingcap/tidb/pull/12514)
    - パーティションテーブル上の `IndexJoin` が誤った結果を返す問題を修正 [#12713](https://github.com/pingcap/tidb/pull/12713)
    - TiDB の `str_to_date` 関数が MySQL と異なる結果を返す問題を修正 [#12757](https://github.com/pingcap/tidb/pull/12757)
    - クエリ条件に `cast` 関数が含まれる場合に、アウタージョイントが内部結合に誤って変換される問題を修正 [#12791](https://github.com/pingcap/tidb/pull/12791)
    - `AntiSemiJoin` の結合条件における不正確な式の渡し方を修正 [#12800](https://github.com/pingcap/tidb/pull/12800)
+ SQL エンジン
    - 時刻の不正確な丸めを修正（例：`2019-09-11 11:17:47.999999666` は `2019-09-11 11:17:48` に丸められるべき） [#12259](https://github.com/pingcap/tidb/pull/12259)
    - `PREPARE` ステートメントの `sql_type` による期間がモニタリングレコードに表示されない問題を修正 [#12329](https://github.com/pingcap/tidb/pull/12329)
    - `from_unixtime` 関数がヌルを処理する際に発生するパニックの問題を修正 [#12572](https://github.com/pingcap/tidb/pull/12572)
    - `YEAR` 型に無効な値が挿入された場合、結果が `0000` ではなく `NULL` になる互換性の問題を修正 [#12744](https://github.com/pingcap/tidb/pull/12744)
    - 暗黙的に割り当てられた `AutoIncrement` 列の挙動を改善し、複数の `AutoIncrement` ID が単一の `Insert` ステートメントで暗黙的に割り当てられた場合、TiDB は割り当てられた値の連続性を保証する。この改善により、JDBC の `getGeneratedKeys()` メソッドがどんなシナリオでも正しい結果を取得することを保証 [#12619](https://github.com/pingcap/tidb/pull/12619)
    - `HashAgg` が `Apply` の子ノードとして機能する際に、クエリが停止する問題を修正 [#12769](https://github.com/pingcap/tidb/pull/12769)
    - 型変換に関する `AND` および `OR` の論理式が不正確な結果を返す問題を修正 [#12813](https://github.com/pingcap/tidb/pull/12813)
+ サーバ
    - `SLEEP()` 関数が `KILL TIDB QUERY` ステートメントに対して無効である問題を修正 [#12159](https://github.com/pingcap/tidb/pull/12159)
    - `AUTO_INCREMENT` が誤って `MAX int64` と `MAX uint64` を割り当てた場合にエラーが報告されない問題を修正 [#12210](https://github.com/pingcap/tidb/pull/12210)
    - ログレベルが `ERROR` の場合にスローダイクエリログが記録されない問題を修正 [#12373](https://github.com/pingcap/tidb/pull/12373)
    - TiDB がスキーマ変更と対応する変更されたテーブル情報を100から1024回までキャッシュする回数を調整し、`tidb_max_delta_schema_count` システム変数を使用して変更をサポート [#12515](https://github.com/pingcap/tidb/pull/12515)
    - SQL統計をより正確にするために、クエリの開始時間を「実行開始」から「コンパイル開始」に変更 [#12638](https://github.com/pingcap/tidb/pull/12638)
    - TiDB ログにおいて `set session autocommit` の記録を追加 [#12568](https://github.com/pingcap/tidb/pull/12568)
    - プラン実行中にリセットされるのを防ぐために、`SessionVars` に SQLクエリの開始時間を記録 [#12676](https://github.com/pingcap/tidb/pull/12676)
    - `ORDER BY`、`GROUP BY`、`LIMIT OFFSET` に `?` プレースホルダをサポート [#12514](https://github.com/pingcap/tidb/pull/12514)
    - 前回のステートメントが `COMMIT` の場合に、前回のステートメントを出力するためにスローダイクエリログに `Prev_stmt` フィールドを追加 [#12724](https://github.com/pingcap/tidb/pull/12724)
    - 明示的にコミットされたトランザクションで `COMMIT` が失敗した場合に、`COMMIT` 前の最後のステートメントをログに記録 [#12747](https://github.com/pingcap/tidb/pull/12747)
    - TiDB サーバーが SQL ステートメントを実行するときに前回のステートメントの保存方法を最適化して、性能を向上させるためのパニック問題を修正 [#12751](https://github.com/pingcap/tidb/pull/12751)
    - `skip-grant-table=true` 構成の下で `FLUSH PRIVILEGES` ステートメントによって発生するパニック問題を修正 [#12816](https://github.com/pingcap/tidb/pull/12816)
    - 短時間に多くの書き込み要求がある場合に性能ボトルネックを回避するために、自動IDの適用のデフォルトの最小ステップを `1000` から `30000` に増やす [#12891](https://github.com/pingcap/tidb/pull/12891)
    - TiDB がパニックを起こした際に失敗した `Prepared` ステートメントをエラーログに印刷しない問題を修正 [#12954](https://github.com/pingcap/tidb/pull/12954)
    - スローダイクエリログにおける `COM_STMT_FETCH` 時間の記録が MySQL と一貫していない問題を修正 [#12953](https://github.com/pingcap/tidb/pull/12953)
    - 書き込み競合のエラーメッセージにエラーコードを追加し、原因を素早く特定できるようにするための改善を追加 [#12878](https://github.com/pingcap/tidb/pull/12878)
+ DDL
    - 列の `AUTO INCREMENT` 属性をデフォルトで削除できないようにする。この属性を削除する必要がある場合は、`tidb_allow_remove_auto_inc` 変数の値を変更してください。詳細は [System Variables](/system-variables.md#tidb_allow_remove_auto_inc-new-in-v2118-and-v304) を参照 [#12146](https://github.com/pingcap/tidb/pull/12146)
    - `Create Table` ステートメントでユニークインデックスを作成する際に複数の `unique` をサポート [#12469](https://github.com/pingcap/tidb/pull/12469)
    - 構築されたテーブルのスキーマにスキーマがない場合、外部キー制約に対する互換性の問題を修正し、`No Database selected` エラーを返すのではなく、作成されたテーブルのスキーマを使用するように [#12678](https://github.com/pingcap/tidb/pull/12678)
    - `ADMIN CANCEL DDL JOBS` を実行すると `invalid list index` エラーが報告される問題を修正 [#12681](https://github.com/pingcap/tidb/pull/12681)
+ モニター
    - 以前に記録されていなかったバックオフ時間（例：コミット時のバックオフ時間など）を補完するために、バックオフ監視のためのタイプを追加 [#12326](https://github.com/pingcap/tidb/pull/12326)
    - `Add Index` 操作の進行状況を監視する新しいメトリクスを追加 [#12389](https://github.com/pingcap/tidb/pull/12389)

## PD

- pd-ctl の `--help` コマンド出力を改善 [#1772](https://github.com/pingcap/pd/pull/1772)

## ツール

+ TiDB Binlog
    - `ALTER DATABASE` 関連の DDL 操作が Drainer の異常終了を引き起こす問題を修正 [#770](https://github.com/pingcap/tidb-binlog/pull/770)
    - レプリケーション効率を改善するために、コミットバイナリログのトランザクションステータス情報のクエリをサポートする [#761](https://github.com/pingcap/tidb-binlog/pull/761)
    - Drainerの `start_ts` がPumpの最大 `commit_ts` よりも大きい場合にPumpがパニックする問題を修正する [#759](https://github.com/pingcap/tidb-binlog/pull/759)

## TiDB Ansible

- TiDB Binlog に "キューサイズ" と "クエリヒストグラム" の2つのモニタリングアイテムを追加する [#952](https://github.com/pingcap/tidb-ansible/pull/952)
- TiDBのアラートルールを更新する [#961](https://github.com/pingcap/tidb-ansible/pull/961)
- デプロイメントおよびアップグレード前に構成ファイルをチェックする [#973](https://github.com/pingcap/tidb-ansible/pull/973)
- TiDBでインデックス速度をモニタリングするための新しいメトリクスを追加する [#987](https://github.com/pingcap/tidb-ansible/pull/987)
- TiDB Binlog モニタリングダッシュボードをGrafana v4.6.3と互換性があるように更新する [#993](https://github.com/pingcap/tidb-ansible/pull/993)