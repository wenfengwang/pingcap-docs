---
title: パブリックエンドポイントを介したTiDB Serverlessへの接続
summary: TiDB Serverlessクラスターへのパブリックエンドポイントを介した接続方法について学びます。

# パブリックエンドポイントを介したTiDB Serverlessへの接続

このドキュメントでは、TiDB Serverlessクラスターへのパブリックエンドポイントを介した接続方法について説明します。パブリックエンドポイントを使用すると、ラップトップからSQLクライアントを使用してTiDB Serverlessクラスターに接続できます。

> **ヒント：**
>
> TiDB Dedicatedクラスターへのパブリックエンドポイントを介した接続方法については、[標準接続を介したTiDB Dedicatedへの接続](/tidb-cloud/connect-via-standard-connection.md)を参照してください。

TiDB Serverlessクラスターへのパブリックエンドポイントを介した接続には、以下の手順を実行してください：

1. [**クラスター**](https://tidbcloud.com/console/clusters) ページに移動し、次に、対象のクラスター名をクリックしてその概要ページに移動します。

2. 右上隅にある **Connect** をクリックします。接続ダイアログが表示されます。

3. ダイアログで、エンドポイントタイプを `Public` としてデフォルト設定のままにし、希望の接続方法とオペレーティングシステムを選択して、対応する接続文字列を取得します。

    > **注意：**
    >
    > - エンドポイントタイプを `Public` のままにすることは、標準TLS接続を介した接続を意味します。詳細については、[TiDB ServerlessへのTLS接続](/tidb-cloud/secure-connections-to-serverless-clusters.md)を参照してください。
    > - **エンドポイントタイプ** ドロップダウンリストで **Private** を選択すると、プライベートエンドポイントを介した接続を意味します。詳細については、[プライベートエンドポイントを介したTiDB Serverlessへの接続](/tidb-cloud/set-up-private-endpoint-connections-serverless.md)を参照してください。

4. まだパスワードを設定していない場合は、**Create password** をクリックしてランダムなパスワードを生成します。生成されたパスワードは再表示されないため、安全な場所に保存してください。

5. 接続文字列を使用してクラスターに接続します。

    > **注意：**
    >
    > TiDB Serverlessクラスターに接続する際は、ユーザー名にクラスターのプレフィックスを含め、名前を引用符で囲む必要があります。詳細については、[ユーザー名のプレフィックス](/tidb-cloud/select-cluster-tier.md#user-name-prefix)を参照してください。

## 次の手順

TiDBクラスターに正常に接続した後は、[TiDBでSQLステートメントを探索](/basic-sql-operations.md)することができます。