---
title: TiDB Cloud APIの概要
summary: TiDB Cloud APIとは、その機能、およびTiDB Cloudクラスタを管理するAPIの使用方法について学びます。

# TiDB Cloud APIの概要 <span style="color: #fff; background-color: #00bfff; border-radius: 4px; font-size: 0.5em; vertical-align: middle; margin-left: 16px; padding: 0 2px;">ベータ</span>

> **注意:**
>
> [TiDB Cloud API](https://docs.pingcap.com/tidbcloud/api/v1beta) はベータ版です。

TiDB Cloud APIは、TiDB Cloud内の管理対象オブジェクトをプログラムで管理するための[RESTインターフェース](https://en.wikipedia.org/wiki/Representational_state_transfer)を提供します。このAPIを通じて、プロジェクト、クラスタ、バックアップ、リストア、およびインポートなどのリソースを自動的かつ効率的に管理できます。

このAPIには、以下の特長があります:

- **JSONエンティティ。** すべてのエンティティはJSONで表されます。
- **HTTPSのみ。** APIはHTTPS経由でのみアクセスできます。これにより、ネットワークを介して送信されるすべてのデータがTLSで暗号化されます。
- **鍵ベースのアクセスおよびダイジェスト認証。** TiDB Cloud APIにアクセスする前に、APIキーを生成する必要があります。すべてのリクエストは[HTTP Digest Authentication](https://en.wikipedia.org/wiki/Digest_access_authentication)を介して認証されます。これにより、APIキーがネットワークを介して送信されることはありません。

TiDB Cloud APIの使用を開始するには、[TiDB Cloud APIドキュメント](https://docs.pingcap.com/tidbcloud/api/v1beta)の以下のリソースを参照してください:

- [はじめる](https://docs.pingcap.com/tidbcloud/api/v1beta#section/Get-Started)
- [認証](https://docs.pingcap.com/tidbcloud/api/v1beta#section/Authentication)
- [レート制限](https://docs.pingcap.com/tidbcloud/api/v1beta#section/Rate-Limiting)
- [APIフルリファレンス](https://docs.pingcap.com/tidbcloud/api/v1beta#tag/Project)
- [変更履歴](https://docs.pingcap.com/tidbcloud/api/v1beta#section/API-Changelog)