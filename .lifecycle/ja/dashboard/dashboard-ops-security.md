---
title: TiDBダッシュボードのセキュリティ保護
summary: TiDBダッシュボードのセキュリティを向上させる方法を学びます。
aliases: ['/docs/dev/dashboard/dashboard-ops-security/']
---

# TiDBダッシュボードのセキュリティ保護

TiDBダッシュボードへのアクセスにはサインインが必要ですが、TiDBダッシュボードはデフォルトで信頼されたユーザー エンティティによってアクセスされるように設計されています。TiDBダッシュボードを外部ネットワークユーザーや信頼できないユーザーに提供する場合は、以下の対策を取ることでセキュリティの脆弱性を回避できます。

## TiDBユーザーのセキュリティ強化

### TiDB `root`ユーザーの強力なパスワードを設定する

TiDBダッシュボードのアカウントシステムはTiDB SQLユーザーのものと一貫しています。デフォルトではTiDBの`root`ユーザーにはパスワードがありません。そのため、TiDBダッシュボードへのアクセスにはパスワード認証が必要ありません。これにより、悪意のある訪問者が特権を持つSQLステートメントを実行するなど、高い特権が与えられる可能性があります。

TiDB`root`ユーザーに強力なパスワードを設定することをお勧めします。詳細は[TiDBユーザーアカウントの管理](/user-account-management.md)を参照してください。または、TiDB`root`ユーザーを無効にすることもできます。

### TiDBダッシュボードのために最小特権を持つユーザーを作成する

TiDBダッシュボードのアカウントシステムはTiDB SQLのそれと一貫しています。TiDBダッシュボードにアクセスするユーザーは、TiDB SQLユーザーの権限に基づいて認証および認可されます。したがって、TiDBダッシュボードには限られた権限、または単に読み取り専用の権限が必要です。最小特権の原則に基づいてユーザーを構成し、高権限のユーザーがアクセスできないようにすることをお勧めします。

TiDBダッシュボードへのアクセスおよびサインインのために最小特権のSQLユーザーを作成することをお勧めします。これにより、高権限のユーザーがアクセスできなくなり、セキュリティが向上します。詳細は[TiDBダッシュボードユーザーの管理](/dashboard/dashboard-user.md)を参照してください。

## 信頼できないアクセスをブロックするファイアウォールの使用

> **注意：**
>
> TiDB v6.5.0（以降）およびTiDB Operator v1.4.0（以降）は、TiDBダッシュボードをKubernetes上で独立したPodとして展開することをサポートしています。TiDB Operatorを使用すると、このPodのIPアドレスにアクセスしてTiDBダッシュボードを起動できます。このポートは他の特権のあるPDのインターフェースとは通信せず、外部に提供されている場合は追加のファイアウォールは必要ありません。詳細については、「TiDB OperatorでTiDBダッシュボードを独立して展開する」を参照してください。

TiDBダッシュボードはPDクライアントポートを介してサービスを提供し、デフォルトでは<http://IP:2379/dashboard/>です。TiDBダッシュボードはアイデンティティ認証が必要ですが、PDクライアントポートにおけるPDで実行される他の特権のあるインターフェース（<http://IP:2379/pd/api/v1/members>など）のアイデンティティ認証は不要で、特権のある操作が実行される可能性があります。そのため、PDクライアントポートを外部ネットワークに直接公開するのは非常にリスクが高いです。

以下の対策を取ることをお勧めします：

+ ファイアウォールを使用して、PDコンポーネントの**どの**クライアントポートに信頼されていないネットワークからのアクセスを禁止する

    > **注意：**
    >
    > TiDB、TiKV、およびその他のコンポーネントはPDコンポーネントとの間で内部ネットワークで通信する必要があります。そうでない場合、クラスタが利用できなくなります。

+ [リバースプロキシを使用したTiDBダッシュボードへのアクセス](/dashboard/dashboard-ops-reverse-proxy.md)を参照して、別のポートでTiDBダッシュボードサービスを安全に提供する方法を学びます。

### 複数のPDインスタンスを展開する場合のTiDBダッシュボードポートの公開方法

> **警告：**
>
> このセクションでは、テスト環境専用の安全でないアクセス解決策について説明しています。本番環境でこの解決策を使用**しないで**ください。

テスト環境では、ファイアウォールを構成してTiDBダッシュボードのポートを外部からアクセス可能にする必要がある場合があります。

複数のPDインスタンスが展開されている場合、実際にTiDBダッシュボードを実行しているのは1つのPDインスタンスのみで、他のPDインスタンスにアクセスするとブラウザのリダイレクションが発生します。そのため、ファイアウォールが正しいIPアドレスで構成されていることを確認する必要があります。このメカニズムの詳細については、「複数のPDインスタンスを展開する」を参照してください。

TiUPデプロイメントツールを使用する場合は、次のコマンドを実行して実際にTiDBダッシュボードを実行しているPDインスタンスのアドレスを表示できます（`CLUSTER_NAME`はクラスタ名に置き換えてください）：

{{< copyable "shell-regular" >}}

```bash
tiup cluster display CLUSTER_NAME --dashboard
```

出力は実際のTiDBダッシュボードのアドレスです。

> **注意：**
>
> この機能は、`tiup cluster`デプロイメントツールの後のバージョン（v1.0.3以降）でのみ利用できます。
>
> <details>
> <summary>TiUP Clusterをアップグレード</summary>
>
> ```bash
> tiup update --self
> tiup update cluster --force
> ```
>
> </details>

以下はサンプルの出力です：

```bash
http://192.168.0.123:2379/dashboard/
```

この例では、ファイアウォールを`192.168.0.123`のオープンIPの`2379`ポートへの受信アクセスが構成されており、TiDBダッシュボードは<http://192.168.0.123:2379/dashboard/>を介してアクセスされます。

## TiDBダッシュボード専用のリバースプロキシ

[信頼できないアクセスをブロックするファイアウォールの使用](#信頼できないアクセスをブロックするファイアウォールの使用)で述べたように、PDクライアントポートの下で提供されるサービスは、TiDBダッシュボード（<http://IP:2379/dashboard/>にある）だけでなく、PDの他の特権のあるインターフェース（<http://IP:2379/pd/api/v1/members>など）も含まれています。したがって、リバースプロキシを使用してTiDBダッシュボードを外部ネットワークに提供する際は、外部ネットワークがリバースプロキシを介してPDの特権のあるインターフェースにアクセスすることを避けるために、ポートの下の**全て**のサービスではなく、`/dashboard`の接頭辞の**み**のサービスが提供されるようにしてください。

リバースプロキシの構成方法については、安全で推奨されるリバースプロキシの構成方法を学ぶために[リバースプロキシを使用したTiDBダッシュボードへのアクセス](/dashboard/dashboard-ops-reverse-proxy.md)を参照してください。

## リバースプロキシのためのTLSの有効化

トランスポート層のセキュリティをさらに向上させるために、リバースプロキシのためにTLSを有効にしたり、ユーザー証明書を認証するためにmTLSを導入したりすることができます。

詳細については[HTTPSサーバーの構成](http://nginx.org/en/docs/http/configuring_https_servers.html)および[HAProxy SSL Termination](https://www.haproxy.com/blog/haproxy-ssl-termination/)を参照してください。

## その他の推奨されるセキュリティ対策

- [コンポーネント間のTLS認証の有効化および保存データの暗号化](/enable-tls-between-components.md)
- [TiDBクライアントとサーバー間のTLS有効化](/enable-tls-between-clients-and-servers.md)