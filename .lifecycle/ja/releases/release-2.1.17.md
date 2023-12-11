---
title: TiDB 2.1.17のリリースノート
aliases: ['/docs/dev/releases/release-2.1.17/','/docs/dev/releases/2.1.17/']
---

# TiDB 2.1.17のリリースノート

リリース日: 2019年9月11日

TiDB バージョン: 2.1.17

TiDB Ansible バージョン: 2.1.17

+ 新機能
    - TiDBの`SHOW TABLE REGIONS`構文に`WHERE`句を追加
    - TiKVとPDに`config-check`機能を追加して構成項目をチェック
    - pd-ctlに`tombstone`記録をクリアする`remove-tombstone`コマンドを追加
    - Reparoにリカバリ速度を制御する`worker-count`および`txn-batch`構成項目を追加

+ 改善
    - PDのスケジューリングプロセスを最適化し、積極的な演算子のプッシュをサポート
    - TiKVの起動プロセスを最適化して、ノードの再起動によるジッターを削減

+ 動作の変更
    - TiDBスロークエリログの`start ts`を最初の実行時刻に変更
    - スロークエリログの利便性を向上させるために、TiDBスロークエリログの`Index_ids`フィールドを`Index_names`フィールドに置き換える
    - `SPLIT TABLE`構文で扱える最大のリージョン数を変更するために、TiDBの構成ファイルに`split-region-max-num`パラメータを追加し、デフォルトの構成で1,000から10,000に増やす

## TiDB

+ SQLオプティマイザ
    - `EvalSubquery`ビルド`Executor`中にエラーが発生した場合にエラーメッセージが正しく返されない問題を修正 [#11811](https://github.com/pingcap/tidb/pull/11811)
    - インデックスルックアップ結合において、外部テーブルの行数が単一バッチよりも大きい場合にクエリ結果が正しくない可能性がある問題を修正し、`UnionScan`を`IndexJoin`のサブノードとして展開可能にすることで機能範囲を拡大 [#11843](https://github.com/pingcap/tidb/pull/11843)
    - 統計フィードバックプロセス中に不正なキー（`invalid encoded key flag 252`など）が発生する可能性がある状況で、`SHOW STAT_BUCKETS`構文でこれらの不正なキーの表示を追加 [#12098](https://github.com/pingcap/tidb/pull/12098)
+ SQL実行エンジン
    - `CAST`関数が数値値を最初に`UINT`に変換することで生じるいくつかの不正確な結果（`select cast(13835058000000000000 as double)`など）を修正 [#11712](https://github.com/pingcap/tidb/pull/11712)
    - `DIV`計算の除数が10進数であり、この計算に負の値が含まれる場合に計算結果が正確でない可能性がある問題を修正 [#11812](https://github.com/pingcap/tidb/pull/11812)
    - `SELECT`/ `EXPLAIN` 文を実行する際に一部の文字列が`INT`型に変換されることで生じるMySQL非互換性問題を修正するために`ConvertStrToIntStrict`関数を追加 [#11892](https://github.com/pingcap/tidb/pull/11892)
    - `EXPLAIN ... FOR CONNECTION`を使用する際に`stmtCtx`の構成が誤っていたことで`Explain`結果が正確でない可能性がある問題を修正 [#11978](https://github.com/pingcap/tidb/pull/11978)
    - 整数の結果がオーバーフローすると非10進数結果が生じるため、`unaryMinus`関数によって返される結果がMySQLと互換性がない問題を修正 [#11990](https://github.com/pingcap/tidb/pull/11990)
    - `LOAD DATA` 文が実行される際のカウント順が原因で`last_insert_id()`が正確でない可能性がある問題を修正 [#11994](https://github.com/pingcap/tidb/pull/11994)
    - ユーザーが明示的に暗黙的な方法で自動増分カラムデータを書き込む際に、`last_insert_id()`が正確でない可能性がある問題を修正 [#12001](https://github.com/pingcap/tidb/pull/12001)
    - `JSON_UNQUOTE`関数に対する過度な引用符のバグを修正: 二重引用符(`"`)で囲まれた値のみが引用符解除されるべきである。例えば、“`SELECT JSON_UNQUOTE("\\\\")`”の結果は“`\\`”であるべき（変更なし）[#12096](https://github.com/pingcap/tidb/pull/12096)
+ サーバー
    - TiDBトランザクションを再試行する際に、スロークエリログに記録される`start ts`を最初の実行時刻に変更 [#11878](https://github.com/pingcap/tidb/pull/11878)
    - ロッキングの解決コストを削減するために、`LockResolver`にトランザクションのキー数を追加してキー数を減らすことでリージョン全体のスキャン操作を回避
    - スロークエリログの`succ`フィールド値が正確でない可能性がある問題を修正
    - `slow query`ログ内の`Index_ids`フィールドを`Index_names`フィールドに置き換えて利便性を向上させる
    - `-`が含まれる`Duration`がTiDBによってEOFエラーにパースされ、接続が切断される問題を修正 (例: `select time(‘--’)`)
    - `RegionCache`内の無効なリージョンをより迅速に削除して、このリージョンに送信されるリクエスト数を削減
    - `oom-action = "cancel"`が使用され、`oom`が`Insert Into … Select` 構文で発生した場合の`OOM`パニック問題を不正に処理することで切断問題を修正 [#12126](https://github.com/pingcap/tidb/pull/12126)
+ DDL
    - `tikvSnapshot`のリバーススキャンインターフェースを追加し、DDL履歴ジョブを効率的にクエリするために追加された。このインターフェースを使用することで`ADMIN SHOW DDL JOBS`の実行時間が大幅に短縮される [#11789](https://github.com/pingcap/tidb/pull/11789)
    - `CREATE TABLE ... PRE_SPLIT_REGION` 構文を改善: `PRE_SPLIT_REGION = N`の場合、事前分割リージョンの数を2^(N-1) から2^Nに変更 [#11797](https://github.com/pingcap/tidb/pull/11797/files)
    - `Add Index` 操作のバックグラウンドワーカースレッドのデフォルトパラメータ値を減らし、オンラインワークロードに大きな影響を与えるのを避ける
    - `SPLIT TABLE ... REGIONS N`を使用してリージョンを分割する際の挙動を改善: N個のデータリージョンと1つのインデックスリージョンを生成
    - `SPLIT TABLE` 構文で扱える最大のリージョン数を調整可能にするために、構成ファイルに`split-region-max-num`パラメータ（デフォルト10000）を追加
    - システムがバイナリログを書き込む際にダウンストリームMySQLが`CREATE TABLE`句をパースできない問題を`PRE_SPLIT_REGIONS`のコメントアウトが原因で発生する問題を修正
    - `SHOW TABLE … REGIONS`と`SHOW TABLE .. INDEX … REGIONS`に`WHERE`サブ句を追加
+ モニター
    - `connection_transient_failure_count`モニタリングメトリックを追加して`tikvclient`のgRPC接続エラーをカウント
- いくつかの場合において TiKV が異常終了する問題を解決する [#5441](https://github.com/tikv/tikv/pull/5441)

## PD

- PD に `config-check` オプションを追加し、PD 構成項目が有効かどうかを確認する [#1725](https://github.com/pingcap/pd/pull/1725)
- pd-ctl に `remove-tombstone` コマンドを追加して、tombstone ストアレコードをクリアするサポートを追加 [#1705](https://github.com/pingcap/pd/pull/1705)
- スケジューリングを加速するために、オペレーターを能動的にプッシュするサポートを追加 [#1686](https://github.com/pingcap/pd/pull/1686)

## ツール

+ TiDB Binlog
    - Reparo に `worker-count` と `txn-batch` 構成項目を追加し、回復スピードを制御する [#746](https://github.com/pingcap/tidb-binlog/pull/746)
    - Drainer のメモリ使用量を最適化し、並列実行効率を改善する [#735](https://github.com/pingcap/tidb-binlog/pull/735)
    - いくつかの場合において Pump が正常に終了できないバグを修正する [#739](https://github.com/pingcap/tidb-binlog/pull/739)
    - Pump の `LevelDB` の処理ロジックを最適化し、GC の実行効率を改善する [#720](https://github.com/pingcap/tidb-binlog/pull/720)
+ TiDB Lightning
    - チェックポイントからデータを再インポートすることによって起動せる可能性のある tidb-lightning のクラッシュを修正する [#239](https://github.com/pingcap/tidb-lightning/pull/239)

## TiDB Ansible

- Spark のバージョンを 2.4.3 に更新し、Spark 2.4.3 と互換性のある TiSpark バージョンを 2.2.0 に更新する [#914](https://github.com/pingcap/tidb-ansible/pull/914), [#919](https://github.com/pingcap/tidb-ansible/pull/927)
- リモートマシンのパスワードが期限切れの場合に待機時間が長い問題を修正する [#937](https://github.com/pingcap/tidb-ansible/pull/937), [#948](https://github.com/pingcap/tidb-ansible/pull/948)