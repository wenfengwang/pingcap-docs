---
title: TiDB専用クラスターに接続する
summary: 異なる方法を使用してTiDB専用クラスターに接続する方法について学びます。

# TiDB専用クラスターに接続する

このドキュメントでは、TiDB専用クラスターに接続する方法について紹介します。

> **ヒント:**
>
> TiDBサーバーレスクラスターに接続する方法については、[TiDBサーバーレスクラスターに接続する](/tidb-cloud/connect-to-tidb-cluster-serverless.md)を参照してください。

TiDB専用クラスターがTiDB Cloud上に作成された後、以下のいずれかの方法で接続できます:

- [標準接続を介して接続する](/tidb-cloud/connect-via-standard-connection.md)

    標準接続はトラフィックフィルター付きのパブリックエンドポイントを公開し、そのためラップトップからSQLクライアントを使用してTiDBクラスターに接続できます。 TLSを使用してTiDBクラスターに接続することもでき、これによりアプリケーションからTiDBクラスターにデータを安全に送信することができます。

- [AWSを使用したプライベートエンドポイントを介して接続する](/tidb-cloud/set-up-private-endpoint-connections.md) (推奨)

    AWSでホストされるTiDB専用クラスターの場合、プライベートエンドポイント接続はVPC内のSQLクライアントがAWS PrivateLink経由でサービスに安全にアクセスできるようにプライベートエンドポイントを提供し、簡略化されたネットワーク管理でデータベースサービスに安全且つ一方通行にアクセスできるようにします。

- [Google Cloud上でのプライベートエンドポイントを介して接続する](/tidb-cloud/set-up-private-endpoint-connections-on-google-cloud.md) (推奨)

    Google CloudでホストされるTiDB専用クラスターの場合、プライベートエンドポイント接続は、VPC内のSQLクライアントがGoogle Cloud Private Service Connect経由でサービスに安全にアクセスできるようにプライベートエンドポイントを提供し、簡略化されたネットワーク管理でデータベースサービスに安全且つ一方通行にアクセスできるようにします。

- [VPCピアリングを介して接続する](/tidb-cloud/set-up-vpc-peering-connections.md)

    より低いレイテンシとより高いセキュリティを求める場合は、VPCピアリングを設定し、対応するクラウドプロバイダー内のVMインスタンスを使用してプライベートエンドポイント経由で接続できます。

- [Chat2Queryを介して接続する (β)](/tidb-cloud/explore-data-with-chat2query.md)

    > **注意:**
    >
    > [TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスターでChat2Queryを使用する場合は、[TiDB Cloudサポート](/tidb-cloud/tidb-cloud-support.md)に連絡してください。

    TiDB Cloudは人工知能（AI）によって動作します。クラスターがAWSにホストされ、クラスターのTiDBバージョンがv6.5.0以降の場合、[TiDB Cloudコンソール](https://tidbcloud.com/)内のAI搭載SQLエディタ「Chat2Query (β)」を使用して、データ価値を最大化できます。

    Chat2Queryでは、単に`--`を入力し、指示に続けてAIによってSQLクエリを自動生成させるか、SQLクエリを手動で記述して、ターミナルを使用せずにデータベースに対してSQLクエリを実行できます。クエリの結果は直感的にテーブルで表示され、クエリログを簡単に確認できます。

- [SQL Shellを介して接続する](/tidb-cloud/connect-via-sql-shell.md): TiDB SQLを試して、MySQLとの互換性を素早くテストしたり、ユーザー権限を管理したりするために使用します。

## 次は

TiDBクラスターに正常に接続した後は、[TiDBでSQLステートメントを探索する](/basic-sql-operations.md)ことができます。