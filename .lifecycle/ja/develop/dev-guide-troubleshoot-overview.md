---
title: SQLまたはトランザクションの問題
summary: アプリケーション開発中に発生するSQLまたはトランザクションの問題のトラブルシューティング方法を学びます。

# SQLまたはトランザクションの問題

本書では、アプリケーション開発中に発生する問題と関連するドキュメントについて紹介します。

## SQLクエリの問題のトラブルシューティング

SQLクエリのパフォーマンスを向上させたい場合は、[SQL パフォーマンスチューニング](/develop/dev-guide-optimize-sql-overview.md) の手順に従って、フルテーブルスキャンやインデックスの不足などのパフォーマンスの問題を解決してください。

<CustomContent platform="tidb">

パフォーマンスの問題が解消されない場合は、以下のドキュメントを参照してください：

- [遅いクエリの分析](/analyze-slow-queries.md)
- [Top SQL を使用した高コストなクエリの識別](/dashboard/top-sql.md)

SQL操作に関する質問がある場合は、[SQL FAQs](/faq/sql-faq.md) を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

SQL操作に関する質問がある場合は、[SQL FAQs](https://docs.pingcap.com/tidb/stable/sql-faq) を参照してください。

</CustomContent>

## トランザクションの問題のトラブルシューティング

[トランザクションエラーの処理](/develop/dev-guide-transaction-troubleshoot.md) を参照してください。

## 関連項目

- [サポートされていない機能](/mysql-compatibility.md#unsupported-features)

<CustomContent platform="tidb">

- [クラスタ管理のFAQ](/faq/manage-cluster-faq.md)
- [TiDBのFAQ](/faq/tidb-faq.md)

</CustomContent>