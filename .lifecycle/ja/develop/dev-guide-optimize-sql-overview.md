---
title: SQL パフォーマンスの最適化概要
summary: TiDB アプリケーション開発者向けの SQL パフォーマンスチューニングの概要を提供します。
---

# SQL パフォーマンスの最適化概要

このドキュメントでは、TiDB における SQL ステートメントのパフォーマンスを最適化する方法について紹介します。良好なパフォーマンスを得るためには、以下の点に注意できます:

* SQL パフォーマンスチューニング
* スキーマ設計：アプリケーションのワークロードパターンに基づいて、トランザクションの競合やホットスポットを避けるためにテーブルのスキーマを変更する必要がある場合があります。

## SQL パフォーマンスチューニング

良好な SQL ステートメントのパフォーマンスを得るためには、以下のガイドラインに従うことができます:

* 可能な限り少ない行をスキャンします。必要なデータのみをスキャンし、余分なデータをスキャンしないようにします。
* 適切なインデックスを使用します。SQL の `WHERE` 句で使用されている列に対応するインデックスがあることを確認します。そうでない場合、ステートメントにはフルテーブルスキャンが伴い、結果としてパフォーマンスが低下します。
* 適切な結合タイプを使用します。クエリに関わるテーブルの相対的なサイズに基づいて適切な結合タイプを選択することが重要です。通常、TiDB のコストベースの最適化プログラムが最適な結合タイプを選択します。ただし稀に、手動でより適切な結合タイプを指定する必要がある場合があります。
* 適切なストレージエンジンを使用します。ハイブリッド OLTP および OLAP ワークロードの場合、TiFlash エンジンが推奨されます。詳細については、[HTAP クエリ](/develop/dev-guide-hybrid-oltp-and-olap-queries.md)を参照してください。

## スキーマ設計

[SQL パフォーマンスチューニング](#sql-performance-tuning)を行ってもアプリケーションのパフォーマンスが改善しない場合は、スキーマ設計およびデータアクセスパターンを確認して、次の問題を回避する必要があります:

<CustomContent platform="tidb">

* トランザクションの競合。トランザクションの競合を診断および解決する方法については、[ロックの競合のトラブルシューティング](/troubleshoot-lock-conflicts.md)を参照してください。
* ホットスポット。ホットスポットを診断および解決する方法については、[ホットスポットのトラブルシューティング](/troubleshoot-hot-spot-issues.md)を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

* トランザクションの競合。トランザクションの競合を診断および解決する方法については、[ロックの競合のトラブルシューティング](https://docs.pingcap.com/tidb/stable/troubleshoot-lock-conflicts)を参照してください。
* ホットスポット。ホットスポットを診断および解決する方法については、[ホットスポットのトラブルシューティング](https://docs.pingcap.com/tidb/stable/troubleshoot-hot-spot-issues)を参照してください。

</CustomContent>

### 関連項目

<CustomContent platform="tidb">

* [SQL パフォーマンスチューニング](/sql-tuning-overview.md)

</CustomContent>

<CustomContent platform="tidb-cloud">

* [SQL パフォーマンスチューニング](/tidb-cloud/tidb-cloud-sql-tuning-overview.md)

</CustomContent>