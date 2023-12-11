---
title: TiDB アーキテクチャFAQ
summary: TiDBに関連する最もよくある質問（FAQ）について学びます。
aliases: ['/docs/dev/faq/tidb-faq/', '/docs/dev/faq/tidb/', '/docs/dev/tiflash/tiflash-faq/', '/docs/dev/reference/tiflash/faq/', '/tidb/dev/tiflash-faq']
---

# TiDB アーキテクチャFAQ

<!-- markdownlint-disable MD026 -->

このドキュメントは、TiDBに関する最もよくある質問をリストアップしています。

## TiDBの紹介とアーキテクチャ

### TiDBとは何ですか？

[TiDB](https://github.com/pingcap/tidb) は、ハイブリッドトランザクショナルおよび分析処理（HTAP）ワークロードをサポートするオープンソースの分散SQLデータベースです。MySQLに対応しており、水平スケーラビリティ、強力な整合性、高可用性を備えています。TiDBの目標は、ユーザーにOLTP（オンライントランザクション処理）、OLAP（オンライン分析処理）、およびHTAPサービスをカバーするワンストップのデータベースソリューションを提供することです。TiDBは、大規模なデータと高可用性、強力な整合性が必要なさまざまなユースケースに適しています。

### TiDBのアーキテクチャは何ですか？

TiDBクラスタには、TiDBサーバー、PD（配置ドライバー）サーバー、およびTiKVサーバーの3つのコンポーネントがあります。詳細については、[TiDBアーキテクチャ](/tidb-architecture.md)、[TiDBストレージ](/tidb-storage.md)、[TiDB計算](/tidb-computing.md)、および[TiDBスケジューリング](/tidb-scheduling.md)を参照してください。

### TiDBはMySQLベースですか？

いいえ。TiDBはMySQLの構文とプロトコルをサポートしていますが、PingCAP, Inc.によって開発および維持されている新しいオープンソースデータベースです。

### TiDB、TiKV、およびPD（配置ドライバー）のそれぞれの責任は何ですか？

- TiDBはSQL計算レイヤーとして機能し、主にSQLの解析、クエリプランの指定、および実行コードの生成に責任を持ちます。
- TiKVは分散型Key-Valueストレージエンジンとして機能し、実際のデータを保存するために使用されます。要するに、TiKVはTiDBのストレージエンジンです。
- PDはTiDBのクラスタマネージャとして機能し、TiKVメタデータを管理し、タイムスタンプを割り当て、データの配置と負荷分散のための決定を行います。

### TiDBは簡単に使用できますか？

はい、簡単に使用できます。すべての必要なサービスが開始されると、TiDBをほとんどの場合、MySQLサーバーのように簡単に使用できます。一般的に、お使いのアプリケーションのコードを一行も変更することなく、MySQLをTiDBで置き換えてアプリケーションに組み込むことができます。また、一般的なMySQL管理ツールを使用してTiDBを管理することもできます。

### TiDBはMySQLと互換性がありますか？

現時点では、TiDBはMySQL 5.7の構文の大部分をサポートしていますが、トリガーやストアドプロシージャ、ユーザー定義関数はサポートしていません。詳細については、[MySQLとの互換性](/mysql-compatibility.md)を参照してください。

### TiDBは分散トランザクションをサポートしていますか？

はい。 TiDBは、クラスタ全体でトランザクションを分散処理します。これは、単一の場所にある数個のノードであるか、複数のデータセンターにまたがる多数のノードであろうともです。TiDBのトランザクションモデルは、GoogleのPercolatorから着想を得たもので、主に2フェーズコミットプロトコルにいくつかの実用的な最適化が施されています。このモデルでは、各トランザクションに対して単調増加するタイムスタンプを割り当て、衝突を検出できるようにするためのタイムスタンプアロケータが使用されています。 [PD](/tidb-architecture.md#placement-driver-pd-server) は、TiDBクラスタでのタイムスタンプアロケータとして機能します。

### TiDBを使用して、どのプログラミング言語を使用できますか？

MySQLクライアントまたはドライバがサポートするすべての言語を使用できます。

### TiDBで他のKey-Valueストレージエンジンを使用できますか？

はい。TiKVに加えて、TiDBはUniStoreやMockTiKVなどのスタンドアロンのストレージエンジンをサポートしています。注意：後のTiDBリリースでは、MockTiKVはサポートされなくなる可能性があります。

すべてのTiDBサポートストレージエンジンを確認するには、次のコマンドを使用してください：

{{< copyable "shell-regular" >}}

```shell
./bin/tidb-server -h
```

以下は出力例です：

```shell
Usage of ./bin/tidb-server:
  -L string
        ログレベル: info, debug, warn, error, fatal (default "info")
  -P string
        tidbサーバーポート (default "4000")
  -V    バージョン情報を出力して終了する (default false)
.........
  -store string
        登録されたストアの名前、[tikv, mocktikv, unistore] (default "unistore")
  ......
```

### TiDBドキュメント以外に、TiDBの知識を獲得するための他の方法はありますか？

- [TiDBドキュメント](https://docs.pingcap.com/): TiDB関連知識を得るための最も重要でタイムリーな方法。
- [TiDBブログ](https://www.pingcap.com/blog/): 技術記事、製品に関する情報、事例研究を学ぶ。
- [PingCAP Education](https://www.pingcap.com/education/?from=en): オンラインコースや認定プログラムを受講する。

### TiDBのユーザー名の長さ制限は何ですか？

最大32文字です。

### TiDBの列数および行サイズの制限は何ですか？

- TiDBの最大列数はデフォルトで1017です。4096まで数を増やすことができます。
- 単一行の最大サイズはデフォルトで6 MBです。数を増やして最大120 MBまで増やすことができます。

詳細については、[TiDBの制限事項](/tidb-limitations.md)を参照してください。

### TiDBはXAをサポートしていますか？

いいえ。TiDBのJDBCドライバはMySQL Connector/Jです。Atomikosを使用する場合は、データソースを`type="com.mysql.jdbc.jdbc2.optional.MysqlXADataSource"`に設定します。 TiDBはMySQL JDBC XADataSourceとの接続をサポートしていません。 MySQL JDBC XADataSource は、MySQL向けにDMLを使用して`redo`ログを変更するなどの操作に対して機能します。

Atomikosの2つのデータソースを構成した後、JDBCドライバーをXAに設定します。 AtomikosがTMとRM（DB）を運用するとき、AtomikosはJDBCレイヤーにXAを含むコマンドを送信します。 MySQLの場合、JDBCレイヤーでXAが有効になっている場合、JDBCはInnoDBに対して`redo`ログを含む一連のXAロジック操作を送信します。これには、2フェーズコミットを行うためにDMLを使用するなどが含まれます。これは、現在のTiDBバージョンでは、上位アプリケーションレイヤのJTA/XAをサポートしておらず、Atomikosによって送信されたXA操作を解析していません。

単独のデータベースとして、MySQLはXAを使用してデータベース間のトランザクションを実行できます。一方、TiDBはGoogle Percolatorトランザクションモデルを使用し、そのパフォーマンスの安定性がXAよりも高いため、JTA/XAをサポートしておらず、TiDBにはXAをサポートする必要がありません。

### TiDBは、列指向ストレージエンジン（TiFlash）に対する高並行の`INSERT`または`UPDATE`操作をどのように処理し、パフォーマンスを低下させずにサポートしていますか？

- [TiFlash](/tiflash/tiflash-overview.md) は、列指向エンジンの変更を処理するためにDeltaTreeという特別な構造を導入しています。
- TiFlashはRaftグループで学習者の役割を果たしているため、ログのコミットまたは書き込みの投票を行いません。これは、DML操作がTiFlashの確認を待つ必要がないことを意味し、それがTiFlashがOLTPのパフォーマンスを低下させることがない理由です。また、TiFlashとTiKVは別々のインスタンスで動作するため、お互いに影響を与えません。

### TiFlashはどのような整合性を提供しますか？

TiFlashはデフォルトで強力なデータ整合性を維持します。 raft学習者プロセスがデータを更新します。また、トランザクション内のデータがクエリのデータと完全に整合していることを保証するために、TSOのチェックが行われます。詳細については、[非同期レプリケーション](/tiflash/tiflash-overview.md#asynchronous-replication)および[整合性](/tiflash/tiflash-overview.md#consistency)を参照してください。

## TiDBテクニック

### データストレージのTiKV

[TiDB内部（I）- データストレージ](https://www.pingcap.com/blog/tidb-internal-data-storage/?from=en)を参照。

### データ計算のTiDB

[TiDB内部（II）- 計算](https://www.pingcap.com/blog/tidb-internal-computing/?from=en)を参照。

### スケジューリングのPD

[TiDB内部（III）- スケジューリング](https://www.pingcap.com/blog/tidb-internal-scheduling/?from=en)を参照。