---
title: TiDB 3.1.1リリースノート
aliases: ['/docs/dev/releases/release-3.1.1/','/docs/dev/releases/3.1.1/']
---

# TiDB 3.1.1リリースノート

リリース日: 2020年4月30日

TiDBバージョン: 3.1.1

TiDB Ansibleバージョン: 3.1.1

## 新機能

+ TiDB

    - `auto_rand_base`に対するテーブルオプションを追加しました [#16812](https://github.com/pingcap/tidb/pull/16812)
    - `Feature ID`コメントを追加しました：SQLステートメントの特殊なコメント内では、登録されたステートメント断片のみがパーサによって解析されます。それ以外の場合、ステートメントは無視されます [#16155](https://github.com/pingcap/tidb/pull/16155)

+ TiFlash

    - 単一の読み取りリクエストのディスクI/Oを減らすために`handle`および`version`列をキャッシュします
    - GrafanaにDeltaTreeエンジンの読み書きワークロードに関連するグラフィックスを追加します
    - `Chunk`コーデックの10進数データエンコーディングを最適化します
    - TiFlashが低いワークロード時に開いているファイルディスクリプタの数を減らします

## バグ修正

+ TiDB

    - インスタンスレベルでの分離読み取り設定が効果を発揮しない問題を修正し、TiDBアップグレード後も分離読み取り設定が誤って保持される問題を修正しました [#16482](https://github.com/pingcap/tidb/pull/16482) [#16802](https://github.com/pingcap/tidb/pull/16802)
    - ハッシュパーティションテーブルでのパーティション選択構文の修正を行い、`partition (P0)`などの構文に対してエラーが報告されないようにしました [#16076](https://github.com/pingcap/tidb/pull/16076)
    - `UPDATE` SQLステートメントがビューからのみクエリを行い、ビューを更新しない場合でも、更新ステートメントがエラーを報告する問題を修正しました [#16789](https://github.com/pingcap/tidb/pull/16789)
    - 入れ子クエリから`not not`を削除することによって発生する誤った結果の問題を修正しました [#16423](https://github.com/pingcap/tidb/pull/16423)

+ TiFlash

    - 異常な状態にあるリージョンからデータを読み取る際にエラーが発生する問題を修正しました
    - TiFlash内のテーブル名のマッピングを修正し、`recover table`/`flashback table`を正しくサポートします
    - テーブル名をリネームする際に発生する潜在的なデータ損失の問題を修正するためにストレージパスを修正しました
    - オンライン更新シナリオでの読み取りモードを修正し、読み取り性能を向上させました
    - データベース/テーブル名に特殊文字が含まれる場合に、TiFlashがアップグレード後に正常に起動しない問題を修正しました

+ ツール

    - バックアップ＆リストア（BR）

        * `auto_random`属性を持つテーブルを復元した後、データを挿入すると重複エントリエラーが発生する問題を修正しました [#241](https://github.com/pingcap/br/issues/241)