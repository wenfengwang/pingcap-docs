---
title: TiDBサーバーレスクラスターへの接続
summary: 異なる方法でTiDBサーバーレスクラスターに接続する方法を学びます。
---

# TiDBサーバーレスクラスターへの接続

このドキュメントでは、TiDBサーバーレスクラスターに接続する方法を紹介します。

> **ヒント:**
>
> TiDB専用クラスターに接続する方法については、[TiDB専用クラスターに接続](/tidb-cloud/connect-to-tidb-cluster.md)を参照してください。

TiDB CloudでTiDBサーバーレスクラスターが作成された後、以下の方法のいずれかを使用して接続できます。

- [プライベートエンドポイントを介した接続](/tidb-cloud/set-up-private-endpoint-connections-serverless.md)（推奨）

    プライベートエンドポイント接続は、AWS PrivateLinkを使用してVPC内のSQLクライアントがデータベースサービスに安全にアクセスできるようにするプライベートエンドポイントを提供し、簡素化されたネットワーク管理でデータベースサービスへの高度な安全かつ一方向のアクセスを提供します。

- [パブリックエンドポイントを介した接続](/tidb-cloud/connect-via-standard-connection-serverless.md)

    標準接続はトラフィックフィルタを備えたパブリックエンドポイントを公開し、ラップトップからSQLクライアントを使用してTiDBクラスターに接続できるようにします。

    TiDBサーバーレスは[TLS接続](/tidb-cloud/secure-connections-to-serverless-clusters.md)をサポートしており、これによりアプリケーションからTiDBクラスターへのデータ伝送のセキュリティが確保されます。

- [Chat2Queryを介した接続（β版）](/tidb-cloud/explore-data-with-chat2query.md)

    TiDB Cloudは人工知能（AI）によって支えられています。[TiDB Cloudコンソール](https://tidbcloud.com/)のChat2Query（β版）を使用して、AI搭載のSQLエディタを使用してデータの価値を最大化できます。

    Chat2Queryでは、`--`と続けて指示を入力することでAIが自動的にSQLクエリを生成するか、手動でSQLクエリを記述し、ターミナルを使用せずにデータベースに対してSQLクエリを実行できます。また、直感的にテーブルにクエリ結果を表示し、クエリログを簡単に確認できます。

## 次は何ですか

TiDBクラスターに正常に接続した後、[TiDBでSQLステートメントを探索](/basic-sql-operations.md)できます。