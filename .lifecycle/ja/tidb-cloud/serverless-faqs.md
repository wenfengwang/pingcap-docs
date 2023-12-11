---
title: TiDBサーバーレスFAQ
summary: TiDBサーバーレスに関連する最もよくある質問（FAQ）について学びます。
aliases: ['/tidbcloud/serverless-tier-faqs']
---

# TiDBサーバーレスFAQ

<!-- markdownlint-disable MD026 -->

このドキュメントは、TiDBサーバーレスに関する最もよくある質問をリストしています。

## 一般的なFAQ

### TiDBサーバーレスとは何ですか？

TiDBサーバーレスは、TiDBデータベースを完全なHTAP機能で提供します。それは、TiDBの完全管理型であり、自分のデータベースをすぐに利用でき、基礎となるノードを気にせずにアプリケーションの開発や実行ができ、アプリケーションのワークロードの変更に基づいて自動的にスケーリングします。

### TiDBサーバーレスをどのように始めればよいですか？

5分で始める[TiDB Cloudクイックスタート](/tidb-cloud/tidb-cloud-quickstart.md)から始めてください。

### TiDB Cloudでは、TiDBサーバーレスクラスターはいくつ作成できますか？

TiDB Cloudの各組織では、デフォルトで最大5つのTiDBサーバーレスクラスターを作成できます。さらに多くのTiDBサーバーレスクラスターを作成するには、クレジットカードを追加し、使用量に[支出制限](/tidb-cloud/tidb-cloud-glossary.md#spending-limit)を設定する必要があります。

### TiDBサーバーレスではTiDB Cloudのすべての機能が完全にサポートされていますか？

TiDB Cloudの一部の機能はTiDBサーバーレスで部分的にサポートされているか、サポートされていません。詳細については、[TiDBサーバーレスの制限とクォータ](/tidb-cloud/serverless-limitations.md)を参照してください。

### TiDBサーバーレスがAWS以外のクラウドプラットフォームで利用可能になるのはいつですか？

TiDBサーバーレスをGoogle CloudやAzureなどの他のクラウドプラットフォームに展開する取り組みを進めています。しかし、現在はすべての環境でのスムーズな機能を得ることに焦点を当てており、現時点では正確なタイムラインはありません。安心してください、TiDBサーバーレスをより多くのクラウドプラットフォームで利用できるようにするために努力しています。進行状況についてはコミュニティを更新していきます。

### TiDBサーバーレスが利用可能になる前にDeveloper Tierクラスターを作成しました。それでもクラスターを使用することはできますか？

はい、Developer Tierクラスターは自動的にTiDBサーバーレスクラスターに移行され、以前の利用に支障をきたすことなく、ユーザーエクスペリエンスが向上します。

## 請求と計量に関するFAQ

### リクエストユニットとは何ですか？

TiDBサーバーレスは使用量に応じた支払いモデルを採用しており、ストレージスペースとクラスターの使用量のみが請求されます。このモデルでは、SQLクエリ、バルク操作、バックグラウンドジョブなどのクラスターアクティビティはすべて[リクエストユニット（RUs）](/tidb-cloud/tidb-cloud-glossary.md#request-unit)で数量化されます。 RUは、クラスター上で開始されたリクエストのサイズと複雑さの抽象的な測定です。詳細については、[TiDBサーバーレスの価格の詳細](https://www.pingcap.com/tidb-cloud-serverless-pricing-details/)を参照してください。

### TiDBサーバーレスのSQLステートメントのRUコストをどのように確認できますか？

[TiDBサーバーレス](/tidb-cloud/select-cluster-tier.md#tidb-serverless)での各SQLステートメントの**総RU**および**平均RU**のコストを確認できます。この機能により、RUのコストを特定し分析することができ、オペレーションでの潜在的なコスト削減につながります。

SQLステートメントのRUの詳細を確認するには、次の手順を実行してください。

1. [TiDB Cloudコンソール](https://tidbcloud.com/)にログインし、プロジェクトの[**クラスター**](https://tidbcloud.com/console/clusters)ページに移動します。

2. [TiDBサーバーレスクラスター](https://tidbcloud.com/console/clusters)の**Diagnosis**ページに移動します。

3. **SQLステートメント**タブをクリックします。

### TiDBサーバーレスに無料プランはありますか？

組織ごとに最初の5つのTiDBサーバーレスクラスターについて、TiDB Cloudは以下の無料使用クォータを提供します。

- 行ベースのストレージ：5 GiB
- [リクエストユニット（RUs）](/tidb-cloud/tidb-cloud-glossary.md#request-unit)：1ヶ月あたり5000万RUs

無料クォータを超える使用量については課金されます。クラスターの無料クォータが達成されると、そのクラスター上での読み書き操作は[クォータの増加](/tidb-cloud/manage-serverless-spend-limit.md#update-spending-limit)または使用量が新しい月の開始時にリセットされるまで抑制されます。

詳細については、[TiDBサーバーレスの使用クォータ](/tidb-cloud/select-cluster-tier.md#usage-quota)を参照してください。

### 無料プランの制限は何ですか？

無料プランでは、クエリごとの最大RUsが実際のワークロードに基づいて秒間最大10,000 RUsに制限されます。さらに、クエリごとのメモリ割り当ては256 MiBに制限されます。クラスターのパフォーマンスを最大限に引き出したい場合は、[支出制限を増やす](/tidb-cloud/manage-serverless-spend-limit.md#update-spending-limit)ことができます。

### ワークロードに必要なRUsの数を見積もり、月次予算を立てる方法は？

個々のSQLステートメントのRU消費量を取得するには、[`EXPLAIN ANALYZE`](/sql-statements/sql-statement-explain-analyze.md#ru-request-unit-consumption)SQLステートメントを使用できます。ただし、`EXPLAIN ANALYZE`で返されるのはegress RUsを含まず、egress使用量はTiDBサーバーが認識できないゲートウェイで別途測定されます。

クラスターのRUおよびストレージの使用量を得るには、クラスター概要ページの**今月の使用量**パネルを確認します。このパネルには、過去のリソース使用データとリアルタイムのリソース使用データが表示され、クラスターのリソース消費を追跡し、適切な支出制限を見積もることができます。無料クォータが要件を満たすことができない場合は、支出制限を簡単に編集できます。詳細については、[TiDBサーバーレスクラスターの支出制限の管理](/tidb-cloud/manage-serverless-spend-limit.md)を参照してください。

### ワークロードの最適化により、RUsの消費を最小限に抑える方法は？

クエリを最適なパフォーマンスで注意深く最適化するために、[SQLパフォーマンスの最適化](/develop/dev-guide-optimize-sql-overview.md)のガイドラインに従ってください。また、egressトラフィックの量を最小限に抑えることもRUsの消費を削減するためには重要です。これは、クエリで必要な列と行のみを返すようにすることで達成できます。列と行を厳密に選択およびフィルタリングすることにより、ネットワークの利用を最適化し、ネットワークの利用を最適化し、ネットワークの利用を最適化できます。

### TiDBサーバーレスではストレージがどのようにメータされますか？

ストレージはTiDBサーバーレスクラスターで保存されているデータの量に基づいて計測され、1ヶ月ごとのGiBで測定します。これは、データの圧縮やレプリカを除いた全てのテーブルとインデックスの合計サイズをその月にデータが保存された時間の数で乗じて計算されます。

### 直ちにテーブルやデータベースを削除してもストレージ使用量サイズが変わらないのはなぜですか？

これはTiDBが一定期間、削除されたテーブルやデータベースを保持するためです。この保持期間は、これらのテーブルに依存するトランザクションが中断することなく実行されるようにします。また、保持期間は誤って削除されたテーブルやデータベースを回復することができる[`FLASHBACK TABLE`](/sql-statements/sql-statement-flashback-table.md)/[`FLASHBACK DATABASE`](/sql-statements/sql-statement-flashback-database.md)機能を実現します。

### どうしてクエリを実行していないときにもRUsが消費されるのですか？

RUsの消費は様々なシナリオで発生することがあります。1つは、テーブル間でのスキーマ変更の同期などのバックグラウンドクエリにおいてです。もう1つは、特定のWebコンソールの機能がスキーマをロードするなどのクエリを生成する場合です。これらのプロセスは、明示的なユーザートリガーがなくてもRUsを使用します。

### ワークロードが一定の状態であるときにRUsの使用量が急増するのはなぜですか？

RUの使用量の急増は、TiDBにおける必要なバックグラウンドジョブによって発生することがあります。これらのジョブは、テーブルを自動的に解析し、統計情報を再構築するなど、最適化されたクエリプランを生成するために必要です。

### クラスターが無料クォータを使い果たしたり支出制限を超えたらどうなりますか？

クラスターが無料クォータまたは支出制限に達すると、クラスターはその読み込みと書き込み操作に対して規制措置を実施します。クォータが増加されるか、使用量が新しい月の開始時にリセットされるまで、これらの操作は制限されます。詳細については、[TiDBサーバーレスの制限とクォータ](/tidb-cloud/serverless-limitations.md#usage-quota)を参照してください。

## セキュリティに関するFAQ

### TiDBサーバーレスは共有されたものなのか、専用のものなのか？

サーバーレステクノロジーはマルチテナントのために設計され、すべてのクラスターに使用されるリソースが共有されます。独立したインフラストラクチャとリソースを使用した管理されたTiDBサービスを取得するには、[TiDB専用](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)にアップグレードできます。

### TiDBサーバーレスはどのようにセキュリティを確保していますか？

- あなたの接続はTransport Layer Security（TLS）によって暗号化されます。TiDBサーバーレスへのTLS接続についての詳細については、[TLS接続をTiDBサーバーレスに使用する](/tidb-cloud/secure-connections-to-serverless-clusters.md)を参照してください。
- TiDBサーバーレスに保存されているすべての永続データは、クラスターが実行されているクラウドプロバイダのツールによって暗号化されています。

## メンテナンスに関するFAQ

### クラスターで実行されているTiDBのバージョンをアップグレードすることはできますか？

```
No. TiDB Serverless clusters are upgraded automatically as we roll out new TiDB versions on TiDB Cloud. You can see what version of TiDB your cluster is running in the [TiDB Cloud console](https://tidbcloud.com/console/clusters) or in the latest [release note](https://docs.pingcap.com/tidbcloud/tidb-cloud-release-notes). Alternatively, you can also connect to your cluster and use `SELECT version()` or `SELECT tidb_version()` to check the TiDB version.
```