---
title: クラスタセキュリティ設定の構成
summary: クラスタに接続するためのルートパスワードと許可されたIPアドレスを構成する方法を学びます。

# クラスタセキュリティ設定の構成

TiDB専用クラスタでは、クラスタに接続するためのルートパスワードと許可されたIPアドレスを構成することができます。

> **注意:**
>
> TiDB Serverlessクラスタの場合、このドキュメントは適用されません。[TiDB ServerlessへのTLS接続](/tidb-cloud/secure-connections-to-serverless-clusters.md)を参照してください。

1. TiDB Cloudコンソールで、プロジェクトの[**クラスタ**](https://tidbcloud.com/console/clusters)ページに移動します。

    > **ヒント:**
    >
    > 複数のプロジェクトがある場合は、左下隅の<MDSvgIcon name="icon-left-projects" />をクリックして、別のプロジェクトに切り替えることができます。

2. 対象のクラスタの行で、**...**をクリックし、**セキュリティ設定**を選択します。
3. **セキュリティ設定**ダイアログで、ルートパスワードと許可されたIPアドレスを構成します。

    クラスタがどのIPアドレスからでもアクセス可能になるようにする場合は、**どこからでもアクセスを許可**をクリックします。

4. **適用**をクリックします。

> **ヒント:**
>
> クラスタの概要ページを表示している場合は、ページの右上隅にある**...**をクリックし、**セキュリティ設定**を選択して、これらの設定も構成できます。