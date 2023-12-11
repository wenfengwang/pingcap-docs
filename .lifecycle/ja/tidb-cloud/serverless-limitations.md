---
title: TiDBサーバーレスの制限とクォータ
summary: TiDBサーバーレスの制限について学びます。
aliases: ['/tidbcloud/serverless-tier-limitations']
---

# TiDBサーバーレスの制限とクォータ

<!-- markdownlint-disable MD026 -->

TiDBサーバーレスは、TiDBがサポートするほとんどすべてのワークロードと連携しますが、TiDBセルフホストまたはTiDB専用クラスターとTiDBサーバーレスクラスターとの間にはいくつかの機能の違いがあります。このドキュメントではTiDBサーバーレスの制限について説明します。

TiDBサーバーレスとTiDB専用との間の機能の隙間を常に埋めています。これらの機能や機能が必要な場合は、[TiDB専用](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)をご利用いただくか、[お問い合わせ](https://www.pingcap.com/contact-us/?from=en)して機能のリクエストをしてください。

## 制限

### 監査ログ

- [データベース監査ログ](/tidb-cloud/tidb-cloud-auditing.md)は現在利用できません。

### 接続

- [Public Endpoint](/tidb-cloud/connect-via-standard-connection-serverless.md)と[Private Endpoint](/tidb-cloud/set-up-private-endpoint-connections-serverless.md)のみを使用できます。[VPC Peering](/tidb-cloud/set-up-vpc-peering-connections.md)を使用してTiDBサーバーレスクラスターに接続することはできません。
- [IPアクセスリスト](/tidb-cloud/configure-ip-access-list.md)はサポートされていません。

### 暗号化

- TiDBサーバーレスクラスターに格納されているデータは、クラスターを管理しているクラウドプロバイダーによって提供される暗号化ツールを使用して暗号化されます。ただし、TiDBサーバーレスはインフラストラクチャレベルの暗号化を超えてディスク上のデータを保護するための追加のオプション的な手段を提供していません。
- [顧客管理型暗号化キー（CMEK）](/tidb-cloud/tidb-cloud-encrypt-cmek.md)は現在利用できません。

### メンテナンスウィンドウ

- [メンテナンスウィンドウ](/tidb-cloud/configure-maintenance-window.md)は現在利用できません。

### 監視と診断

- [サードパーティの監視統合](/tidb-cloud/third-party-monitoring-integrations.md)は現在利用できません。
- [組み込みのアラート](/tidb-cloud/monitor-built-in-alerting.md)は現在利用できません。
- [Key Visualizer](/tidb-cloud/tune-performance.md#key-visualizer)は現在利用できません。
- [Index Insight](/tidb-cloud/tune-performance.md#index-insight-beta)は現在利用できません。
- [クラスターイベント](/tidb-cloud/tidb-cloud-events.md)は現在利用できません。

### セルフサービスのアップグレード

- TiDBサーバーレスはTiDBの完全管理された展開です。TiDBサーバーレスのメジャーやマイナーバージョンのアップグレードはTiDBクラウドによって処理されるため、ユーザーによって開始されることはありません。

### ストリームデータ

- [Changefeed](/tidb-cloud/changefeed-overview.md)は現在TiDBサーバーレスに対応していません。
- [データ移行](/tidb-cloud/migrate-from-mysql-using-data-migration.md)は現在TiDBサーバーレスに対応していません。

### その他

- [Time to live (TTL)](/time-to-live.md)は現在利用できません。
- トランザクションは30分を超えることはできません。
- SQLの制限についての詳細は[有限SQL機能](/tidb-cloud/limited-sql-features.md)を参照してください。

## 使用クォータ

TiDB Cloudの各組織では、デフォルトで最大5つのTiDBサーバーレスクラスターを作成できます。さらに多くのTiDBサーバーレスクラスターを作成するには、クレジットカードを追加して使用の[上限を設定](/tidb-cloud/tidb-cloud-glossary.md#spending-limit)する必要があります。

組織内の最初の5つのTiDBサーバーレスクラスターについて、TiDB Cloudはそれぞれ以下の無料利用クォータを提供します。

- 行ベースのストレージ: 5 GiB
- [リクエストユニット（RUs）](/tidb-cloud/tidb-cloud-glossary.md#request-unit): 1か月につき5,000万RUs

リクエストユニット（RU）は、クエリやトランザクションのリソース消費を追跡するための単位です。これは、データベースで特定の要求を処理するために必要なコンピューティングリソースを見積もるためのメトリックです。リクエストユニットはまた、TiDB Cloudサーバーレスサービスの請求単位でもあります。

クラスターの無料クォータが達成されると、このクラスター上での読み取りと書き込みの操作が制限されます。これは、[クォータを増やす](/tidb-cloud/manage-serverless-spend-limit.md#update-spending-limit)か、使用量が新しい月の開始時にリセットされるまで続きます。

さまざまなリソースのRU消費（読み取り、書き込み、SQL CPU、ネットワーク出力など）、価格の詳細、および制限情報について学習するには、[TiDBサーバーレスの価格詳細](https://www.pingcap.com/tidb-cloud-serverless-pricing-details)を参照してください。

追加のクォータを持つTiDBサーバーレスクラスターを作成したい場合、クラスター作成ページで使用上限を編集できます。詳細については、[TiDBサーバーレスクラスターの作成](/tidb-cloud/create-tidb-cluster-serverless.md)を参照してください。

TiDBサーバーレスを作成した後は、クラスターの概要ページで使用上限を確認して編集することができます。詳細については、[TiDBサーバーレスクラスターの使用上限の管理](/tidb-cloud/manage-serverless-spend-limit.md)を参照してください。