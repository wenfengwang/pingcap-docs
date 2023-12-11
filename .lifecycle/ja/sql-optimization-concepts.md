---
title: SQLの最適化プロセス
summary: TiDBにおけるSQLの論理最適化と物理最適化について学びます。
aliases: ['/docs/dev/sql-optimization-concepts/','/docs/dev/reference/performance/sql-optimizer-overview/']
---

# SQLの最適化プロセス

TiDBでは、クエリを入力して最終的な実行計画に従って実行結果を取得するまでのプロセスは、次のように示されます:

![SQLの最適化プロセス](/media/sql-optimization.png)

`parser`によって元のクエリテキストを解析し、いくつかの簡単な妥当性チェックを行った後、TiDBはまずクエリにいくつかの論理的に等価な変更を加えます。詳細な変更については、[SQLの論理最適化](/sql-logical-optimization.md)を参照してください。

これらの等価な変更を通じて、このクエリは論理実行計画でより扱いやすくなります。等価な変更が完了した後、TiDBは元のクエリと同等のクエリ計画構造を取得し、その後演算子のデータ分布と特定の実行コストに基づいて最終的な実行計画を取得します。詳細については、[SQLの物理最適化](/sql-physical-optimization.md)を参照してください。

同時に、TiDBが[`PREPARE`](/sql-statements/sql-statement-prepare.md)文を実行する際、TiDBで実行計画の生成コストを削減するためにキャッシュの有効化を選択することができます。詳細については、[実行計画キャッシュ](/sql-prepared-plan-cache.md)を参照してください。