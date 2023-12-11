---
title: 標準接続を使用してTiDB Dedicatedに接続する
summary: TiDB Cloudクラスタに標準接続で接続する方法について学びます。

# 標準接続を使用してTiDB Dedicatedに接続する

このドキュメントは、標準接続を使用してTiDB Dedicatedクラスタに接続する方法について説明します。標準接続はトラフィックフィルタを公開エンドポイントで公開し、そのため、SQLクライアントを使用してノートパソコンからTiDB Dedicatedクラスタに接続できます。

> **ヒント:**
>
> TiDB Serverlessクラスタに標準接続で接続する方法については、「[公開エンドポイントを使用してTiDB Serverlessに接続する](/tidb-cloud/connect-via-standard-connection-serverless.md)」を参照してください。

TiDB Dedicatedクラスタに標準接続で接続するには、次の手順を実行します。

1. 対象クラスタの概要ページを開きます。

    1. [TiDB Cloudコンソール](https://tidbcloud.com/)にログインし、プロジェクトの[**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動します。

        > **ヒント:**
        >
        > 複数のプロジェクトを所有している場合、左下隅の<MDSvgIcon name="icon-left-projects" />をクリックして、別のプロジェクトに切り替えることができます。

    2. 対象クラスタの名前をクリックして、その概要ページに移動します。

2. 右上隅の**接続**をクリックします。接続ダイアログが表示されます。

3. クラスタのトラフィックフィルタを作成します。トラフィックフィルタはSQLクライアントを使用してTiDB Cloudにアクセスすることが許可されているIPとCIDRアドレスの一覧です。

    トラフィックフィルタがすでに設定されている場合は、次のサブ手順をスキップします。トラフィックフィルタが空の場合は、次のサブ手順を実行して追加します。

    1. いずれかのボタンをクリックして、簡単にルールを追加します。

        - **現在のIPアドレスを追加**
        - **どこからでもアクセスを許可**

    2. 追加されたIPアドレスまたはCIDR範囲にオプションの説明を入力します。

    3. 変更を確認するために**フィルタを作成**をクリックします。

4. ダイアログの**ステップ2: TiDBクラスタCAをダウンロード**の下で、TiDBクラスタのTLS接続のために**TiDBクラスタCAをダウンロード**をクリックします。TiDBクラスタCAはデフォルトでTLS 1.2バージョンをサポートしています。

    > **注意:**
    >
    > - TiDBクラスタCAはTiDB Dedicatedクラスタでのみ利用できます。
    > - 現在、TiDB Cloudはこれらの接続方法に関する接続文字列とサンプルコードの提供のみを行っています: MySQL、MyCLI、JDBC、Python、Go、およびNode.js。

5. ダイアログの**ステップ3: SQLクライアントで接続**の下で、お好みの接続方法のタブをクリックし、そのタブの接続文字列とサンプルコードを参照してクラスタに接続します。

    ダウンロードしたCAファイルのパスを接続文字列の`--ssl-ca`オプションの引数として使用する必要があることに注意してください。

## 次の手順

TiDBクラスタに正常に接続できたら、[TiDBでSQL文を探索](/basic-sql-operations.md)できます。