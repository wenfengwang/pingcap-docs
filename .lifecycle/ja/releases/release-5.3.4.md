---
title: TiDB 5.3.4 リリースノート
---

# TiDB 5.3.4 リリースノート

リリース日: 2022年11月24日

TiDB バージョン: 5.3.4

## 改善点

+ TiKV

    - 更新ごとにTLS証明書を自動で再ロードして可用性を向上させる[#12546](https://github.com/tikv/tikv/issues/12546)

## バグ修正

+ TiDB

    - リージョンがマージされる際にリージョンキャッシュが適切にクリーンアップされない問題を修正[#37141](https://github.com/pingcap/tidb/issues/37141)
    - `ENUM`または`SET`カラムのエンコーディングが間違っているためにTiDBが誤ったデータを書き込む問題を修正[#32302](https://github.com/pingcap/tidb/issues/32302)
    - データベースレベルの権限が適切にクリーンアップされない問題を修正[#38363](https://github.com/pingcap/tidb/issues/38363)
    - `mysql.tables_priv`テーブルに`grantor`フィールドが欠落している問題を修正[#38293](https://github.com/pingcap/tidb/issues/38293)
    - アイドル接続で`KILL TIDB`が即座に有効にならない問題を修正[#24031](https://github.com/pingcap/tidb/issues/24031)
    - `date_add`および`date_sub`の戻り値の違いを修正(TiDBとMySQLの間)[#36394](https://github.com/pingcap/tidb/issues/36394), [#27573](https://github.com/pingcap/tidb/issues/27573)
    - パーサがテーブルオプションを復元する際に不正な`INSERT_METHOD`値を修正[#38368](https://github.com/pingcap/tidb/issues/38368)
    - MySQLクライアント(v5.1またはそれ以前)がTiDBサーバに接続した際に認証に失敗する問題を修正[#29725](https://github.com/pingcap/tidb/issues/29725)
    - 符号なしの`BIGINT`引数を渡した場合の`GREATEST`および`LEAST`の誤った結果を修正[#30101](https://github.com/pingcap/tidb/issues/30101)
    - TiDBにおける`ifnull(time(3))`の結果がMySQLと異なる問題を修正[#29498](https://github.com/pingcap/tidb/issues/29498)
    - `avg()`関数がTiFlashからクエリされた際に`ERROR 1105 (HY000): other error for mpp stream: Could not convert to the target type - -value is out of range.`が返される問題を修正[#29952](https://github.com/pingcap/tidb/issues/29952)
    - `HashJoinExec`を使用した際に`ERROR 1105 (HY000): close of nil channel`が返される問題を修正[#30289](https://github.com/pingcap/tidb/issues/30289)
    - 論理演算をクエリした際にTiKVとTiFlashが異なる結果を返す問題を修正[#37258](https://github.com/pingcap/tidb/issues/37258)
    - DMLエグゼキューターを含む`EXPLAIN ANALYZE`ステートメントがトランザクションのコミットが完了する前に結果を返す問題を修正[#37373](https://github.com/pingcap/tidb/issues/37373)
    - 多くのリージョンをマージした後、リージョンキャッシュが適切にクリアされない問題を修正[#37174](https://github.com/pingcap/tidb/issues/37174)
    - 特定のシナリオで`EXECUTE`ステートメントが予期しないエラーをスローする問題を修正[#37187](https://github.com/pingcap/tidb/issues/37187)
    - `ORDER BY`句に相関副問い合わせが含まれる場合に`GROUP CONCAT`が失敗する問題を修正[#18216](https://github.com/pingcap/tidb/issues/18216)
    - DecimalおよびRealの長さと幅が誤って設定された場合に、プランキャッシュを使用した際に誤った結果が返される問題を修正[#29565](https://github.com/pingcap/tidb/issues/29565)

+ PD

    - PDがダッシュボードプロキシリクエストを正しく処理できない問題を修正[#5321](https://github.com/tikv/pd/issues/5321)
    - 特定のシナリオでTiFlashのレプリカが正しく作成されない問題を修正[#5401](https://github.com/tikv/pd/issues/5401)
    - 不正確なストリームタイムアウトを修正し、リーダー切り替えを加速する[#5207](https://github.com/tikv/pd/issues/5207)

+ TiFlash

    - 引数の型がUInt8の場合に論理演算子が誤った結果を返す問題を修正[#6127](https://github.com/pingcap/tiflash/issues/6127)
    - 整数のデフォルト値として`0.0`が使用されるとTiFlashブートストラップが失敗する問題を修正(例: `` `i` int(11) NOT NULL DEFAULT '0.0'``)[#3157](https://github.com/pingcap/tiflash/issues/3157)

+ Tools

    + Dumpling

        - `--compress`オプションとS3出力ディレクトリが同時に設定されている場合にDumplingがデータをダンプできない問題を修正[#30534](https://github.com/pingcap/tidb/issues/30534)

    + TiCDC

        - Changefeedの状態が正しくないため、MySQL関連のエラーが時間内にオーナーに報告されない問題を修正[#6698](https://github.com/pingcap/tiflow/issues/6698)