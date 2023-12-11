---
title: SQL論理最適化
---

# SQL論理最適化

この章では、TiDBが最終クエリプランを生成する方法を理解するのに役立ついくつかの主要な論理書き換えについて説明します。たとえば、TiDBで `select * from t where t.a in (select t1.a from t1 where t1.b=t.b)` クエリを実行すると、 `IN` サブクエリ `t.a in (select t1.a from t1 where t1.b=t.b)` は存在しないことがわかります。これはTiDBがここでいくつかの書き換えを行ったためです。

この章では、次の主要な書き換えについて紹介します:

- [サブクエリ関連の最適化](/subquery-optimization.md)
- [列プルーニング](/column-pruning.md)
- [相関サブクエリのデコレーション](/correlated-subquery-optimization.md)
- [Max/Minの除外](/max-min-eliminate.md)
- [述語のプッシュダウン](/predicate-push-down.md)
- [パーティションプルーニング](/partition-pruning.md)
- [TopNおよびLimit演算子のプッシュダウン](/topn-limit-push-down.md)
- [結合の並び替え](/join-reorder.md)