---
title: TiDB 4.0.13 リリースノート
---

# TiDB 4.0.13 リリースノート

リリース日: 2021年5月28日

TiDB バージョン: 4.0.13

## 新機能

+ TiDB

    - `AUTO_INCREMENT`カラムを`AUTO_RANDOM`に変更することをサポート [#24608](https://github.com/pingcap/tidb/pull/24608)
    - クライアントに返されたエラーを追跡するための`infoschema.client_errors_summary`テーブルを追加 [#23267](https://github.com/pingcap/tidb/pull/23267)

## 改善点

+ TiDB

    - キャッシュされた統計情報が最新の場合は`mysql.stats_histograms`テーブルを頻繁に読み込まないようにし、高いCPU使用率を回避する [#24352](https://github.com/pingcap/tidb/pull/24352)

+ TiKV

    - `store used size`の計算プロセスをより正確に行うように改善 [#9904](https://github.com/tikv/tikv/pull/9904)
    - `EpochNotMatch`メッセージでより多くのリージョンを設定し、リージョンのミスを減らすように改善 [#9731](https://github.com/tikv/tikv/pull/9731)
    - 長期間実行されたクラスターで蓄積されたメモリを解放する処理を高速化する [#10035](https://github.com/tikv/tikv/pull/10035)

+ PD

    - TSO処理時間のメトリクスを最適化し、PD側でTSO処理時間が長すぎるかどうかをユーザーが判断できるように改善 [#3524](https://github.com/pingcap/pd/pull/3524)
    - ダッシュボードバージョンをv2021.03.12.1に更新する [#3469](https://github.com/pingcap/pd/pull/3469)

+ TiFlash

    - アーカイブされたデータを自動的にクリアしてディスク容量を開放する

+ Tools

    + バックアップ & リストア (BR)

        - `mysql`スキーマで作成されたユーザーテーブルのバックアップをサポート [#1077](https://github.com/pingcap/br/pull/1077)
        - `checkVersion`を更新してクラスターデータとバックアップデータをチェックするように改善 [#1090](https://github.com/pingcap/br/pull/1090)
        - バックアップ時に一部のTiKVノードの障害を許容するように改善 [#1062](https://github.com/pingcap/br/pull/1062)

    + TiCDC

        - メモリオーバーフロー(OOM)を回避するためにプロセッサフローコントロールを実装する [#1751](https://github.com/pingcap/tiflow/pull/1751)
        - Unified Sorterにおける古い一時ファイルをクリーンアップし、複数の`cdc server`インスタンスが同じ`sort-dir`ディレクトリを共有するのを防ぐようにサポートする [#1741](https://github.com/pingcap/tiflow/pull/1741)
        - failpoint用のHTTPハンドラを追加する [#1732](https://github.com/pingcap/tiflow/pull/1732)

## バグ修正

+ TiDB

    - サブクエリを使用して生成された列を更新する`UPDATE`ステートメントでパニックが発生する問題を修正 [#24658](https://github.com/pingcap/tidb/pull/24658)
    - マルチカラムインデックスを使用してデータを読み取る際に重複したクエリ結果が発生する問題を修正 [#24634](https://github.com/pingcap/tidb/pull/24634)
    - `DIV`式で`BIT`型の定数を除数として使用した場合に間違ったクエリ結果が発生する問題を修正 [#24266](https://github.com/pingcap/tidb/pull/24266)
    - DDLステートメントでデフォルトの列値を設定する際に`NO_ZERO_IN_DATE` SQLモードが効果を持たない問題を修正 [#24185](https://github.com/pingcap/tidb/pull/24185)
    - `BIT`タイプの列と`INTEGER`タイプの列の間で`UNION`を使用した際に間違ったクエリ結果が返される問題を修正 [#24026](https://github.com/pingcap/tidb/pull/24026)
    - `BINARY`型と`CHAR`型を比較する際に誤って`TableDual`プランが作成される問題を修正 [#23917](https://github.com/pingcap/tidb/pull/23917)
    - `insert ignore on duplicate`ステートメントが予期せずテーブルレコードを削除する問題を修正 [#23825](https://github.com/pingcap/tidb/pull/23825)
    - `Audit`プラグインがTiDBをパニックさせる問題を修正 [#23819](https://github.com/pingcap/tidb/pull/23819)
    - `HashJoin`演算子が間違った整列処理を行う問題を修正 [#23812](https://github.com/pingcap/tidb/pull/23812)
    - `batch_point_get`が悲観的トランザクションで異常な値を誤って処理することで切断が発生する問題を修正 [#23778](https://github.com/pingcap/tidb/pull/23778)
    - `tidb_row_format_version`構成値を`1`に設定し、`enable_new_collation`値を`true`に設定した場合に不一致なインデックスが発生する問題を修正 [#23772](https://github.com/pingcap/tidb/pull/23772)
    - `INTEGER`型列を`STRING`定数値と比較した際にエラーが発生する問題を修正 [#23705](https://github.com/pingcap/tidb/pull/23705)
    - `BIT`型列を`approx_percent`関数に渡すとエラーが発生する問題を修正 [#23702](https://github.com/pingcap/tidb/pull/23702)
    - TiFlashバッチリクエストを実行する際に誤って`TiKV server timeout`エラーが報告される問題を修正 [#23700](https://github.com/pingcap/tidb/pull/23700)
    - 接頭辞列インデックスにおいて`IndexJoin`演算子が間違った結果を返す問題を修正 [#23691](https://github.com/pingcap/tidb/pull/23691)
    - `BINARY`タイプ列の整列が適切に処理されないことでクエリの誤った結果が発生する問題を修正 [#23598](https://github.com/pingcap/tidb/pull/23598)
    - `HAVING`句を含む結合クエリで`UPDATE`ステートメントを実行する際にクエリパニックが発生する問題を修正 [#23575](https://github.com/pingcap/tidb/pull/23575)
    - 比較式で`NULL`定数を使用した際にTiFlashが誤った結果を返す問題を修正 [#23474](https://github.com/pingcap/tidb/pull/23474)
    - `YEAR`タイプ列を`STRING`定数と比較した際に誤った結果が返る問題を修正 [#23335](https://github.com/pingcap/tidb/pull/23335)
    - `session.group_concat_max_len`が小さすぎると`group_concat`がパニックする問題を修正 [#23257](https://github.com/pingcap/tidb/pull/23257)
    - `TIME`タイプ列に対して`BETWEEN`式を使用した際に誤った結果が返る問題を修正 [#23233](https://github.com/pingcap/tidb/pull/23233)
    - `DELETE`ステートメントで特権チェックが正しく行われない問題を修正 [#23215](https://github.com/pingcap/tidb/pull/23215)
    - `DECIMAL`型列に不正な文字列を挿入した際にエラーが報告されない問題を修正 [#23196](https://github.com/pingcap/tidb/pull/23196)
    - `DECIMAL`型列にデータを挿入した際にパースエラーが発生する問題を修正 [#23152](https://github.com/pingcap/tidb/pull/23152)
    - `USE_INDEX_MERGE`ヒントの効果がない問題を修正 [#22924](https://github.com/pingcap/tidb/pull/22924)
    - `ENUM`または`SET`列を`WHERE`句でフィルターとして使用した際に誤った結果が返る問題を修正 [#22814](https://github.com/pingcap/tidb/pull/22814)
    - クラスターインデックスと新しい整列を同時に使用した際に誤った結果が返る問題を修正 [#21408](https://github.com/pingcap/tidb/pull/21408)
    - `ANALYZE`を`enable_new_collation`を有効にして実行した際にパニックが発生する問題を修正 [#21299](https://github.com/pingcap/tidb/pull/21299)
    - SQLビューがSQL DEFINERに関連付けられたデフォルトのロールを正しく処理しない問題を修正 [#24531](https://github.com/pingcap/tidb/pull/24531)
    - DDLジョブのキャンセルがスタックしてしまう問題を修正 [#24445](https://github.com/pingcap/tidb/pull/24445)
    - `concat`関数が整列を誤って処理する問題を修正 [#24300](https://github.com/pingcap/tidb/pull/24300)
- `SELECT`フィールドに`IN`サブクエリがあり、サブクエリの外側に`NULL`タプルが含まれている場合に、クエリが誤った結果を返すバグを修正[#24022](https://github.com/pingcap/tidb/pull/24022)
- `TableScan`が降順の場合に最適化プログラムがTiFlashを誤って選択するバグを修正[#23974](https://github.com/pingcap/tidb/pull/23974)
- `point_get`プランがMySQLと一貫性のないカラム名を返すバグを修正[#23970](https://github.com/pingcap/tidb/pull/23970)
- 大文字の名前を持つデータベースで`show table status`ステートメントを実行すると、誤った結果が返される問題を修正[#23958](https://github.com/pingcap/tidb/pull/23958)
- テーブルに対して同時に`INSERT`と`DELETE`の権限を持たないユーザーが`REPLACE`操作を実行できるバグを修正[#23938](https://github.com/pingcap/tidb/pull/23938)
- カラムの集合や操作の結果が間違っている問題を修正。これは照合が誤って処理されるためです[#23878](https://github.com/pingcap/tidb/pull/23878)
- `RANGE`パーティションを持つテーブルでクエリを実行するとパニックが発生する問題を修正[#23689](https://github.com/pingcap/tidb/pull/23689)
- 以前のバージョンのクラスターでは`tidb_enable_table_partition`変数が`false`に設定されている場合、パーティションを含むテーブルが非パーティション化されたテーブルとして処理される問題を修正。この場合、後のバージョンへのクラスターのアップグレード後、このテーブルに`batch point get`クエリを実行すると接続がパニックすることがあります[#23682](https://github.com/pingcap/tidb/pull/23682)
- TiDBがTCPおよびUNIXソケットでリスンするように構成されている場合、TCP接続経由のリモートホストが接続の正当性を正しく検証されない問題を修正[#23513](https://github.com/pingcap/tidb/pull/23513)
- デフォルト以外の照合によってクエリ結果が誤る問題を修正[#22923](https://github.com/pingcap/tidb/pull/22923)
- Grafanaの**Coprocessor Cache**パネルが機能しないバグを修正[#22617](https://github.com/pingcap/tidb/pull/22617)
- オプティマイザが統計キャッシュにアクセスする際に発生するエラーを修正[#22565](https://github.com/pingcap/tidb/pull/22565)

+ TiKV

    - `file_dict`ファイルがディスクに完全に書き込まれていない場合にTiKVが起動しないバグを修正[#9963](https://github.com/tikv/tikv/pull/9963)
    - デフォルトでTiCDCのスキャン速度を128MB/sで制限するよう修正[#9983](https://github.com/tikv/tikv/pull/9983)
    - TiCDCの初期スキャンのメモリ使用量を削減するよう修正[#10133](https://github.com/tikv/tikv/pull/10133)
    - TiCDCのスキャン速度にバックプレッシャーをサポートするよう修正[#10142](https://github.com/tikv/tikv/pull/10142)
    - 不要な読み取りを避けることでTiCDCの古い値を取得する際の潜在的なOOM問題を修正[#10031](https://github.com/tikv/tikv/pull/10031)
    - 古い値の読み取りによってTiCDCのOOM問題を修正[#10197](https://github.com/tikv/tikv/pull/10197)
    - S3ストレージのタイムアウトメカニズムを追加し、クライアントが応答せずにハングすることを避けるよう修正[#10132](https://github.com/tikv/tikv/pull/10132)

+ TiFlash

    - `delta-merge-tasks`の数がPrometheusに報告されない問題を修正
    - `Segment Split`中に発生するTiFlashのパニック問題を修正
    - Grafanaの`Region write Duration (write blocks)`パネルが誤った位置に表示される問題を修正
    - ストレージエンジンがデータを削除できない潜在的な問題を修正
    - `TIME`型を`INTEGER`型にキャストする際に誤った結果が発生する問題を修正
    - `bitwise`演算子の動作がTiDBと異なるバグを修正
    - `STRING`型を`INTEGER`型にキャストする際に誤った結果が発生する問題を修正
    - 連続して高速な書き込みがTiFlashをメモリ不足にさせる可能性があるバグを修正
    - テーブルGC中にヌルポインタの例外が発生する可能性がある潜在的な問題を修正
    - 削除されたテーブルへのデータ書き込み中に発生するTiFlashのパニック問題を修正
    - BRのリストア中に発生するTiFlashのパニック問題を修正
    - 一般的なCI照合を使用する場合、一部の文字のウェイトが誤るバグを修正
    - 削除済みテーブルでデータが失われる潜在的な問題を修正
    - ゼロバイトを含む文字列を比較する際に誤った結果が発生する問題を修正
    - 入力列にヌル定数が含まれる場合に論理関数が誤った結果を返す問題を修正
    - 論理関数が数値型のみを受け入れる問題を修正
    - タイムスタンプの値が`1970-01-01`でタイムゾーンオフセットが負の場合に誤った結果が返る問題を修正
    - `Decimal256`のハッシュ値が安定しない問題を修正

+ ツール

    + TiCDC

        - ソーターの入力チャネルがブロックされている場合に発生するフロー制御によるデッドロック問題を修正[#1779](https://github.com/pingcap/tiflow/pull/1779)
        - TiKV GCのセーフポイントがTiCDC changefeedのチェックポイントのステーガネーションによってブロックされる問題を修正[#1756](https://github.com/pingcap/tiflow/pull/1756)
        - MySQLへのデータレプリケーション時に`explicit_defaults_for_timestamp`の更新が`SUPER`権限を必要とするようになっていたことを元に戻す問題を修正[#1749](https://github.com/pingcap/tiflow/pull/1749)

    + TiDB Lightning

        - オートコミットが無効になっている場合にTiDB LightningのTiDBバックエンドがデータをロードできないバグを修正