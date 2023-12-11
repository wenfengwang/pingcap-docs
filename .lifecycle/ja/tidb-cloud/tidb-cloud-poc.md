---
title: TiDB CloudでProof of Concept (PoC)を実行する
summary: TiDB CloudでProof of Concept (PoC)を実行する方法について学びます。
---

# TiDB CloudでProof of Concept (PoC)を実行する

TiDB Cloudは、完全に管理されたクラウドデータベースでTiDBの優れた機能をすべて提供するデータベースサービス（DBaaS）製品です。データベースの複雑さではなく、アプリケーションに集中できるようにサポートします。現在、TiDB CloudはAmazon Web Services（AWS）とGoogle Cloudの両方で利用可能です。

PoCを実行することは、TiDB Cloudがビジネスニーズに最適かどうかを判断するための最良の方法です。また、TiDB Cloudの主な機能に短期間で慣れることができます。パフォーマンステストを実行することで、TiDB Cloudでのワークロードの効率的な実行が可能かどうかを確認できます。また、データの移行と構成の適応に必要な作業を評価することもできます。

このドキュメントでは、典型的なPoC手順を説明し、TiDB CloudのPoCを迅速に完了するのに役立ちます。これは、TiDBの専門家や大規模な顧客ベースによって検証されたベストプラクティスです。

PoCに興味がある場合は、開始する前に<a href="mailto:tidbcloud-support@pingcap.com">PingCAP</a>にお問い合わせください。サポートチームがテスト計画を作成し、PoC手順をスムーズに進行させるお手伝いをします。

または、[TiDB Serverlessを作成して](/tidb-cloud/tidb-cloud-quickstart.md#step-1-create-a-tidb-cluster)、TiDB Cloudを素早く評価することができます。ただし、TiDB Serverlessにはいくつかの[特別な条件](/tidb-cloud/select-cluster-tier.md#tidb-serverless-special-terms-and-conditions)があります。

## PoC手順の概要

PoCの目的は、TiDB Cloudがビジネス要件を満たしているかどうかをテストすることです。典型的なPoCは通常14日間続き、その間にPoCを完了することが期待されます。

典型的なTiDB CloudのPoCは、次の手順で構成されています。

1. 成功基準を定義し、テスト計画を作成する
2. ワークロードの特性を特定する
3. PoC用にTiDB Dedicatedクラスターをサインアップして作成する
4. スキーマとSQLを適応する
5. データをインポートする
6. ワークロードを実行し、結果を評価する
7. より多くの機能を探索する
8. 環境をクリーンアップし、PoCを完了する

## ステップ1. 成功基準を定義し、テスト計画を作成する

PoCでTiDB Cloudを評価する際には、ビジネスニーズに基づいて興味の対象と対応する技術的評価基準を決定し、PoCの期待と目標を明確にすることをお勧めします。明確で測定可能な技術基準と詳細なテスト計画は、重要な側面に焦点を当て、ビジネスレベルの要件をカバーし、最終的にPoC手順を通じて回答を得るのに役立ちます。

PoCの目標を特定するのに次の質問を使用してください。

- ワークロードのシナリオは何ですか？
- ビジネスのデータセットサイズやワークロードは何ですか？成長率は？
- ビジネス要件には、ビジネスの重要なスループットやレイテンシー要件が含まれますか？
- 予定された計画停止または非計画のダウンタイムの最小許容値を含め、可用性と安定性の要件は何ですか？
- オペレーショナル効率のための必要なメトリクスは何ですか？それらをどのように測定しますか？
- ワークロードのセキュリティおよびコンプライアンス要件は何ですか？

成功基準やテスト計画の詳細については、<a href="mailto:tidbcloud-support@pingcap.com">PingCAP</a>にお問い合わせください。

## ステップ2. ワークロードの特性を特定する

TiDB Cloudは、高い可用性と強い一貫性が必要なさまざまなユースケースに適しています。[TiDB概要](https://docs.pingcap.com/tidb/stable/overview)には、主な機能とシナリオがリストされています。これらがビジネスシナリオに適用されるかどうかを確認できます。

- 水平方向の拡大または縮小
- 金融機関レベルの高可用性
- リアルタイムのHTAP
- MySQL 5.7プロトコルとMySQLエコシステムとの互換性

また、分析処理の高速化を支援する列指向ストレージエンジンである[TiFlash](https://docs.pingcap.com/tidb/stable/tiflash-overview)を使用することに興味があるかもしれません。PoC中はいつでもTiFlash機能を使用できます。

## ステップ3. TiDB Dedicatedクラスターをサインアップして作成する

PoC用に[TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated)クラスターを作成するには、次の手順に従います。

1. 次のいずれかの方法でPoC申請フォームに記入します。

    - PingCAPのウェブサイトで[Pocを申し込む](https://pingcap.com/apply-for-poc/)ページにアクセスして、申請フォームに記入します。
    - [TiDB Cloudコンソール](https://tidbcloud.com/)で、右下の **?** をクリックし、**Contact Sales** をクリックしてから、**Apply for PoC** を選択して申請フォームに記入します。

    フォームを送信すると、TiDB Cloudサポートチームが申請を確認し、承認された場合はアカウントにクレジットを転送します。また、PoC手順をスムーズに実行するためにPingCAPのサポートエンジニアに連絡して協力を依頼することもできます。

2. PoC用に[TiDB Dedicatedクラスターを作成する](/tidb-cloud/create-tidb-cluster.md)に関する情報を参照します。

クラスターサイズを計画するために容量プランニングを推奨します。TiDB、TiKV、またはTiFlashノードの見積り数から開始し、後でクラスターをスケーリングアウトしてパフォーマンス要件を満たすことができます。詳細については、次のドキュメントを参照するか、サポートチームに相談してください。

- 見積り方法の詳細については、[TiDBのサイズを見積もる](/tidb-cloud/size-your-cluster.md)をご覧ください。
- TiDB Dedicatedクラスターの構成については、[TiDB Dedicatedクラスターを作成する](/tidb-cloud/create-tidb-cluster.md)をご覧ください。それぞれ、TiDB、TiKV、およびTiFlash（オプション）のクラスターサイズを設定します。
- PoCクレジットの消費を効果的に計画し最適化する方法については、このドキュメントの[FAQ](#faq)をご覧ください。
- スケーリングの詳細については、[TiDBクラスターのスケーリング](/tidb-cloud/scale-tidb-cluster.md)をご覧ください。

専用のPoCクラスターが作成されたら、データを読み込んで一連のテストを実行する準備ができます。TiDBクラスターに接続する方法については、[TiDB Dedicatedクラスターに接続する](/tidb-cloud/connect-to-tidb-cluster.md)を参照してください。

新しく作成されたクラスターの場合、次の構成に注意してください。

- デフォルトのタイムゾーン（ダッシュボードの**作成時間**列）はUTCです。[組織のタイムゾーンを設定する](/tidb-cloud/manage-user-access.md#set-the-time-zone-for-your-organization)の手順に従って、現地のタイムゾーンに変更できます。
- 新しいクラスターのデフォルトのバックアップ設定は、毎日のフルデータベースバックアップです。好みのバックアップ時間を指定したり、データを手動でバックアップしたりできます。デフォルトのバックアップ時間と詳細については、[TiDBクラスターのデータのバックアップとリストア](/tidb-cloud/backup-and-restore.md#backup)をご覧ください。

## ステップ4. スキーマとSQLを適応する

次に、テーブルやインデックスなどのデータベーススキーマをTiDBクラスターにロードできます。

PoCクレジットの量が限られているため、クレジットの価値を最大化するために、[TiDB Serverlessクラスター](/tidb-cloud/select-cluster-tier.md#tidb-serverless)を作成して、TiDB Cloudでの互換性テストや予備分析を実行することをお勧めします。

TiDB CloudはMySQL 5.7と高い互換性があります。MySQL互換であるか、MySQLと互換性があるように適応できるデータであれば、データをTiDBに直接インポートできます。

互換性に関する詳細については、次のドキュメントを参照してください。

- [TiDBとMySQLの互換性](https://docs.pingcap.com/tidb/stable/mysql-compatibility)
- [MySQLとは異なるTiDBの機能](https://docs.pingcap.com/tidb/stable/mysql-compatibility#features-that-are-different-from-mysql)
- [TiDBのキーワードと予約語](https://docs.pingcap.com/tidb/stable/keywords)
- [TiDBの制限事項](https://docs.pingcap.com/tidb/stable/tidb-limitations)

以下はいくつかのベストプラクティスです。

- スキーマ設定に効率が悪い箇所がないか確認します。
- 不要なインデックスを削除します。
- 効果的なパーティション分割のためのパーティションポリシーを計画します。
- タイムスタンプなどのインデックスによって[ホットスポット問題](https://docs.pingcap.com/tidb/stable/troubleshoot-hot-spot-issues#identify-hotspot-issues)が引き起こされないようにします。
- [SHARD_ROW_ID_BITS](https://docs.pingcap.com/tidb/stable/shard-row-id-bits)や[AUTO_RANDOM](https://docs.pingcap.com/tidb/stable/auto-random)を使用することによって、[ホットスポット問題](https://docs.pingcap.com/tidb/stable/troubleshoot-hot-spot-issues#identify-hotspot-issues)を避けます。

SQLステートメントについては、データソースの互換性レベルに応じて適応が必要になる場合があります。

質問がある場合は、[PingCAP](/tidb-cloud/tidb-cloud-support.md)にご相談ください。

## ステップ5. データをインポートする
```markdown
小規模のデータセットをインポートして素早く実施可能性をテストするか、大規模なデータセットをインポートして TiDB データ移行ツールのスループットをテストできます。TiDB はサンプルデータを提供していますが、実際のビジネスでの実稼働負荷でのテストを強くお勧めします。

以下のさまざまな形式で TiDB Cloud にデータをインポートできます：

- [データ移行を使用して MySQL 互換データベースを TiDB Cloud に移行](/tidb-cloud/migrate-from-mysql-using-data-migration.md)
- [ローカルファイルを TiDB Cloud にインポート](/tidb-cloud/tidb-cloud-import-local-files.md)
- [SQL ファイル形式のサンプルデータをインポート](/tidb-cloud/import-sample-data.md)
- [Amazon S3 または GCS から CSV ファイルをインポート](/tidb-cloud/import-csv-files.md)
- [Apache Parquet ファイルをインポート](/tidb-cloud/import-parquet-files.md)

> **注記:**
>
> **インポート** ページでのデータインポートによる追加の請求は発生しません。

## ステップ 6. ワークロードを実行して結果を評価する

これで環境が作成され、スキーマが適応され、データがインポートされました。ワークロードをテストする時がきました。

ワークロードをテストする前に、必要に応じてマニュアル バックアップを実行し、必要に応じてデータベースを元の状態に復元できるようにします。詳細については、[TiDB クラスターデータのバックアップとリストア](/tidb-cloud/backup-and-restore.md#backup) を参照してください。

ワークロードを開始した後、次の方法でシステムを観察できます：

- クラスター概要ページで、クラスターの一般的に使用されるメトリクスを確認できます。これには、Total QPS、Latency、Connections、TiFlash Request QPS、TiFlash Request Duration、TiFlash Storage Size、TiKV Storage Size、TiDB CPU、TiKV CPU、TiKV IO Read、TiKV IO Write などが含まれます。[TiDB クラスターの監視](/tidb-cloud/monitor-tidb-cluster.md) を参照してください。
- **診断 > ステートメント** に移動し、SQL の実行を観察し、システムテーブルのクエリなしでパフォーマンスの問題を簡単に特定できます。[ステートメント分析](/tidb-cloud/tune-performance.md) を参照してください。
- **診断 > Key Visualizer** に移動し、TiDB データのアクセスパターンとデータのホットスポットを表示できます。[Key Visualizer](/tidb-cloud/tune-performance.md#key-visualizer) を参照してください。
- これらのメトリクスを独自の Datadog および Prometheus に統合することもできます。[サードパーティ監視統合](/tidb-cloud/third-party-monitoring-integrations.md) を参照してください。

これでテスト結果の評価の時間です。

より正確な評価を行うために、テストの前にメトリクスのベースラインを決定し、各実行のテスト結果を適切に記録してください。結果を分析することで、TiDB Cloud がアプリケーションに適しているかどうかを判断できます。また、これらの結果はシステムの実行状況を示し、メトリクスに応じてシステムを調整できます。たとえば：

- システムのパフォーマンスが要件を満たしているかどうかを評価します。合計 QPS と遅延をチェックしてください。システムのパフォーマンスが満足のいくものでない場合、次のようにパフォーマンスを調整できます。

    - ネットワークの遅延を監視および最適化します。
    - SQL パフォーマンスを調査および最適化します。
    - ホットスポットの問題を監視および[解決します](https://docs.pingcap.com/tidb/dev/troubleshoot-hot-spot-issues#troubleshoot-hotspot-issues)。

- ストレージサイズと CPU 使用率を評価し、必要に応じて TiDB クラスターをスケール アウトまたはスケールインします。スケーリングの詳細については、[FAQ](#faq) セクションを参照してください。

次のはパフォーマンス調整のためのヒントです：

- 書き込みパフォーマンスの向上

    - TiDB クラスターをスケールアウトすることで書き込みスループットを向上させます ([TiDB クラスターをスケール](/tidb-cloud/scale-tidb-cluster.md) 参照)。
    - [楽観的トランザクションモデル](https://docs.pingcap.com/tidb/stable/optimistic-transaction#tidb-optimistic-transaction-model) を使用することでロックの競合を減らします。

- クエリパフォーマンスの向上

    - **診断 > ステートメント** ページで SQL 実行プランを確認します。
    - **ダッシュボード > Key Visualizer** ページでホットスポットの問題を確認します。
    - **概要 > 容量メトリクス** ページで TiDB クラスターが容量不足になっていないかを監視します。
    - TiFlash 機能を使用して、分析処理を最適化します。[HTAP クラスターを使用](/tiflash/tiflash-overview.md) を参照してください。

## ステップ 7. より多くの機能を探索する

ワークロードテストが完了したら、アップグレードやバックアップなどの機能をさらに探索できます。

- アップグレード

    TiDB Cloud では定期的に TiDB クラスターをアップグレードします。また、クラスターのアップグレードをリクエストするためのサポートチケットを提出することもできます。[TiDB クラスターをアップグレード](/tidb-cloud/upgrade-tidb-cluster.md) を参照してください。

- バックアップ

    ベンダーロックインを避けるために、デイリーフルバックアップを使用してデータを新しいクラスターに移行し、[Dumpling](https://docs.pingcap.com/tidb/stable/dumpling-overview) を使用してデータをエクスポートできます。詳細については、[TiDB 専用データのバックアップとリストア](/tidb-cloud/backup-and-restore.md#backup) および [TiDB 専用データのバックアップとリストア](/tidb-cloud/backup-and-restore-serverless.md#backup) を参照してください。

## ステップ 8. 環境を片付けて PoC を完了する

TiDB Cloud を実際のワークロードでテストし、テスト結果を取得した後、PoC の完全なサイクルが完了しました。これらの結果は TiDB Cloud が期待に応えるかどうかを判断するのに役立ちます。また、TiDB Cloud のベストプラクティスを蓄積することができます。

新しい一連の展開やテスト、TiDB Cloud が提供するその他のノードストレージサイズでの展開など、TiDB Cloud をより大規模に試してみたい場合は、[TiDB 専用](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスターを作成して TiDB Cloud のフルアクセスを取得できます。

クレジットが不足して PoC を続行したい場合は、[TiDB Cloud サポート](/tidb-cloud/tidb-cloud-support.md) に連絡してください。

PoC を終了し、テスト環境をいつでも削除できます。詳細については、[TiDB クラスターを削除](/tidb-cloud/delete-tidb-cluster.md) を参照してください。

サポートチームへのフィードバックは、[TiDB Cloud フィードバックフォーム](https://www.surveymonkey.com/r/L3VVW8R) に記入することで大歓迎です。たとえば、PoC プロセス、機能リクエスト、製品の改善方法などをお知らせください。

## FAQ

### 1. データのバックアップとリストアにはどれくらい時間がかかりますか？

TiDB Cloud は、自動バックアップとマニュアルバックアップの 2 種類のデータベースバックアップを提供します。どちらの方法もフルデータベースをバックアップします。

データのバックアップおよびリストアにかかる時間は、テーブルの数、ミラーコピーの数、および CPU による負荷レベルによって異なります。1 つの単一 TiKV ノードでのバックアップおよびリストアの速度は、およそ 50 MB/s です。

データベースのバックアップとリストアの操作は通常 CPU に負荷がかかり、追加の CPU リソースが常に必要です。このような環境では、QPS およびトランザクションの遅延に影響を及ぼす可能性があります (CPU 負荷に応じて 10% から 50%)。

### 2. スケールアウトとスケールインはいつ行う必要がありますか？

次の点に注意してスケーリングについて検討してください：

- ピーク時またはデータのインポート時に、ダッシュボードの容量メトリクスが上限に達したことがわかる場合 ([TiDB クラスターの監視](/tidb-cloud/monitor-tidb-cluster.md) を参照)、クラスターをスケールアウトする必要があります。
- リソースの使用率が一貫して低い場合、たとえば CPU 使用率が 10% から 20% の場合、リソースを節約するためにクラスターをスケールインできます。

クラスターを自分で拡張できますが、クラスターをスケールインするには、[TiDB Cloud サポート](/tidb-cloud/tidb-cloud-support.md) に連絡してサポートを依頼する必要があります。スケーリングの詳細については、[TiDB クラスターのスケール](/tidb-cloud/scale-tidb-cluster.md) を参照してください。進行状況を追跡するためにサポートチームとコンタクトを取ることができます。データ再バランスによるパフォーマンスへの影響を避けるために、テストを開始する前にスケーリング操作が完了するのを待たなければなりません。

### 3. PoC クレジットを最大限に活用するためにはどうすればよいですか？

PoC の承認がされたら、アカウントにクレジットが付与されます。通常、クレジットは 14 日間の PoC に十分な量が付与されます。クレジットはノードの種類と数によって時間単位で請求されます。詳細については、[TiDB Cloud 請求](/tidb-cloud/tidb-cloud-billing.md#credits) を参照してください。

PoC で使用可能なクレジットの残高を確認するには、対象プロジェクトの [**クラスター**](https://tidbcloud.com/console/clusters) ページに移動し、次のスクリーンショットに示すようにします。

![TiDB Cloud PoC クレジット](/media/tidb-cloud/poc-points.png)

または、TiDB Cloud コンソールの左下隅にある <MDSvgIcon name="icon-top-organization" /> をクリックし、**請求** をクリックし、クレジットの詳細ページを表示します。

未使用のクレジットが残っている場合は、これらのクレジットを PoC プロセスが完了するまでの間、TiDB クラスター料金を支払うために使用することができます。

PoC プロセスが完了しても未使用のクレジットがある場合、これらのクレジットを有効期限が切れるまで使用して、TiDB クラスター料金を支払うことができます。
```
```
### 4. Can I take more than 2 weeks to complete a PoC?

If you want to extend the PoC trial period or are running out of credits, [contact PingCAP](https://www.pingcap.com/contact-us/) for help.

### 5. I'm stuck with a technical problem. How do I get help for my PoC?

You can always [contact TiDB Cloud Support](/tidb-cloud/tidb-cloud-support.md) for help.
```