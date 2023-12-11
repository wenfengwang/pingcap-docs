# TiDB 4.0.4リリースノート

リリース日：2020年7月31日

TiDBバージョン：4.0.4

## バグ修正

+ TiDB

    - `information_schema.columns`をクエリする際にスタックする問題を修正しました [#18849](https://github.com/pingcap/tidb/pull/18849)
    - `PointGet`および`BatchPointGet`演算子が`in null`に遭遇したときに発生するエラーを修正しました [#18848](https://github.com/pingcap/tidb/pull/18848)
    - `BatchPointGet`の誤った結果を修正しました [#18815](https://github.com/pingcap/tidb/pull/18815)
    - `HashJoin`演算子が`set`または`enum`タイプに遭遇したときに発生するクエリ結果の不正を修正しました [#18859](https://github.com/pingcap/tidb/pull/18859)