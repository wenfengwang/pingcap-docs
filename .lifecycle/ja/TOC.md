```markdown
      + [Docs Home](https://docs.pingcap.com/)
      + About TiDB
        + [TiDB Introduction](/overview.md)
        + [TiDB 7.4 Release Notes](/releases/release-7.4.0.md)
        + [Features](/basic-features.md)
        + [MySQL Compatibility](/mysql-compatibility.md)
        + [TiDB Limitations](/tidb-limitations.md)
        + [Credits](/credits.md)
        + [Roadmap](/tidb-roadmap.md)
      + Quick Start
        + [Try Out TiDB](/quick-start-with-tidb.md)
        + [Try Out HTAP](/quick-start-with-htap.md)
        + [Learn TiDB SQL](/basic-sql-operations.md)
        + [Learn HTAP](/explore-htap.md)
        + [Import Example Database](/import-example-data.md)
      + Develop
        + [Overview](/develop/dev-guide-overview.md)
        + Quick Start
          + [Build a TiDB Serverless Cluster](/develop/dev-guide-build-cluster-in-cloud.md)
          + [CRUD SQL in TiDB](/develop/dev-guide-tidb-crud-sql.md)
        + Example Applications
          + Java
            + [JDBC](/develop/dev-guide-sample-application-java-jdbc.md)
            + [MyBatis](/develop/dev-guide-sample-application-java-mybatis.md)
            + [Hibernate](/develop/dev-guide-sample-application-java-hibernate.md)
            + [Spring Boot](/develop/dev-guide-sample-application-java-spring-boot.md)
          + Go
            + [Go-MySQL-Driver](/develop/dev-guide-sample-application-golang-sql-driver.md)
            + [GORM](/develop/dev-guide-sample-application-golang-gorm.md)
          + Python
            + [mysqlclient](/develop/dev-guide-sample-application-python-mysqlclient.md)
            + [MySQL Connector/Python](/develop/dev-guide-sample-application-python-mysql-connector.md)
            + [PyMySQL](/develop/dev-guide-sample-application-python-pymysql.md)
            + [SQLAlchemy](/develop/dev-guide-sample-application-python-sqlalchemy.md)
            + [peewee](/develop/dev-guide-sample-application-python-peewee.md)
            + [Django](/develop/dev-guide-sample-application-python-django.md)
          + Node.js
            + [node-mysql2](/develop/dev-guide-sample-application-nodejs-mysql2.md)
            + [mysql.js](/develop/dev-guide-sample-application-nodejs-mysqljs.md)
            + [Prisma](/develop/dev-guide-sample-application-nodejs-prisma.md)
            + [Sequelize](/develop/dev-guide-sample-application-nodejs-sequelize.md)
            + [TypeORM](/develop/dev-guide-sample-application-nodejs-typeorm.md)
            + [Next.js](/develop/dev-guide-sample-application-nextjs.md)
            + [AWS Lambda](/develop/dev-guide-sample-application-aws-lambda.md)
          + Ruby
            + [mysql2](/develop/dev-guide-sample-application-ruby-mysql2.md)
            + [Rails](/develop/dev-guide-sample-application-ruby-rails.md)
        + Connect to TiDB
          + GUI Database Tools
            + [JetBrains DataGrip](/develop/dev-guide-gui-datagrip.md)
            + [DBeaver](/develop/dev-guide-gui-dbeaver.md)
            + [VS Code](/develop/dev-guide-gui-vscode-sqltools.md)
          + [Choose Driver or ORM](/develop/dev-guide-choose-driver-or-orm.md)
          + [Connect to TiDB](/develop/dev-guide-connect-to-tidb.md)
          + [Connection Pools and Connection Parameters](/develop/dev-guide-connection-parameters.md)
        + Design Database Schema
          + [Overview](/develop/dev-guide-schema-design-overview.md)
          + [Create a Database](/develop/dev-guide-create-database.md)
          + [Create a Table](/develop/dev-guide-create-table.md)
          + [Create a Secondary Index](/develop/dev-guide-create-secondary-indexes.md)
        + Write Data
          + [Insert Data](/develop/dev-guide-insert-data.md)
          + [Update Data](/develop/dev-guide-update-data.md)
          + [Delete Data](/develop/dev-guide-delete-data.md)
          + [Periodically Delete Data Using Time to Live](/time-to-live.md)
          + [Prepared Statements](/develop/dev-guide-prepared-statement.md)
        + Read Data
          + [Query Data from a Single Table](/develop/dev-guide-get-data-from-single-table.md)
          + [Multi-table Join Queries](/develop/dev-guide-join-tables.md)
          + [Subquery](/develop/dev-guide-use-subqueries.md)
          + [Paginate Results](/develop/dev-guide-paginate-results.md)
          + [Views](/develop/dev-guide-use-views.md)
          + [Temporary Tables](/develop/dev-guide-use-temporary-tables.md)
          + [Common Table Expression](/develop/dev-guide-use-common-table-expression.md)
          + Read Replica Data
            + [Follower Read](/develop/dev-guide-use-follower-read.md)
            + [Stale Read](/develop/dev-guide-use-stale-read.md)
          + [HTAP Queries](/develop/dev-guide-hybrid-oltp-and-olap-queries.md)
        + Transaction
          + [Overview](/develop/dev-guide-transaction-overview.md)
          + [Optimistic and Pessimistic Transactions](/develop/dev-guide-optimistic-and-pessimistic-transaction.md)
          + [Transaction Restraints](/develop/dev-guide-transaction-restraints.md)
          + [Handle Transaction Errors](/develop/dev-guide-transaction-troubleshoot.md)
        + Optimize
          + [Overview](/develop/dev-guide-optimize-sql-overview.md)
          + [SQL Performance Tuning](/develop/dev-guide-optimize-sql.md)
          + [Best Practices for Performance Tuning](/develop/dev-guide-optimize-sql-best-practices.md)
          + [Best Practices for Indexing](/develop/dev-guide-index-best-practice.md)
          + Other Optimization Methods
            + [Avoid Implicit Type Conversions](/develop/dev-guide-implicit-type-conversion.md)
            + [Unique Serial Number Generation](/develop/dev-guide-unique-serial-number-generation.md)
        + Troubleshoot
          + [SQL or Transaction Issues](/develop/dev-guide-troubleshoot-overview.md)
          + [Unstable Result Set](/develop/dev-guide-unstable-result-set.md)
          + [Timeouts](/develop/dev-guide-timeouts-in-tidb.md)
        + Reference
          + [Bookshop Example Application](/develop/dev-guide-bookshop-schema-design.md)
          + Guidelines
            + [Object Naming Convention](/develop/dev-guide-object-naming-guidelines.md)
            + [SQL Development Specifications](/develop/dev-guide-sql-development-specification.md)
        + Cloud Native Development Environment
          + [Gitpod](/develop/dev-guide-playground-gitpod.md)
        + Third-Party Support
          + [Third-Party Tools Supported by TiDB](/develop/dev-guide-third-party-support.md)
          + [Known Incompatibility Issues with Third-Party Tools](/develop/dev-guide-third-party-tools-compatibility.md)
          + [ProxySQL Integration Guide](/develop/dev-guide-proxysql-integration.md)
          + [Amazon AppFlow Integration Guide](/develop/dev-guide-aws-appflow-integration.md)
      + Deploy
        + [Software and Hardware Requirements](/hardware-and-software-requirements.md)
        + [Environment Configuration Checklist](/check-before-deployment.md)
        + Plan Cluster Topology
          + [Minimal Topology](/minimal-deployment-topology.md)
          + [TiFlash Topology](/tiflash-deployment-topology.md)
          + [TiCDC Topology](/ticdc-deployment-topology.md)
          + [TiDB Binlog Topology](/tidb-binlog-deployment-topology.md)
          + [TiSpark Topology](/tispark-deployment-topology.md)
          + [Cross-DC Topology](/geo-distributed-deployment-topology.md)
          + [Hybrid Topology](/hybrid-deployment-topology.md)
        + Install and Start
          + [Use TiUP](/production-deployment-using-tiup.md)
          + [Deploy on Kubernetes](/tidb-in-kubernetes.md)
        + [Verify Cluster Status](/post-installation-check.md)
        + Test Cluster Performance
          + [Test TiDB Using Sysbench](/benchmark/benchmark-tidb-using-sysbench.md)
          + [Test TiDB Using TPC-C](/benchmark/benchmark-tidb-using-tpcc.md)
          + [Test TiDB Using CH-benCHmark](/benchmark/benchmark-tidb-using-ch.md)
      + Migrate
        + [Overview](/migration-overview.md)
        + [Migration Tools](/migration-tools.md)
        + [Import Best Practices](/tidb-lightning/data-import-best-practices.md)
        + Migration Scenarios
          + [Migrate from Aurora](/migrate-aurora-to-tidb.md)
          + [Migrate Small Datasets from MySQL](/migrate-small-mysql-to-tidb.md)
          + [Migrate Large Datasets from MySQL](/migrate-large-mysql-to-tidb.md)
          + [Migrate and Merge MySQL Shards of Small Datasets](/migrate-small-mysql-shards-to-tidb.md)
```
- [大規模なデータセットのMySQLシャードの移行と統合](/migrate-large-mysql-shards-to-tidb.md)
  - [CSVファイルからの移行](/migrate-from-csv-files-to-tidb.md)
  - [SQLファイルからの移行](/migrate-from-sql-files-to-tidb.md)
  - [Parquetファイルからの移行](/migrate-from-parquet-files-to-tidb.md)
  - [TiDBクラスターから別のTiDBクラスターへの移行](/migrate-from-tidb-to-tidb.md)
  - [TiDBからMySQL互換のデータベースへの移行](/migrate-from-tidb-to-mysql.md)
- 高度な移行
  - [gh-ostまたはpt-oscを使用した連続レプリケーション](/migrate-with-pt-ghost.md)
  - [カラムが追加されたダウンストリームテーブルへの移行](/migrate-with-more-columns-downstream.md)
  - [Binlogイベントのフィルタリング](/filter-binlog-event.md)
  - [SQL式を使用したDMLイベントのフィルタリング](/filter-dml-event.md)
- 統合
  - [概要](/integration-overview.md)
  - 統合シナリオ
    - [ConfluentおよびSnowflakeとの統合](/ticdc/integrate-confluent-using-ticdc.md)
    - [Apache KafkaおよびApache Flinkとの統合](/replicate-data-to-kafka.md)
- 維持
  - アップグレード
    - [TiUPの使用](/upgrade-tidb-using-tiup.md)
    - [TiDB Operatorの使用](https://docs.pingcap.com/tidb-in-kubernetes/stable/upgrade-a-tidb-cluster)
    - [TiDBスムーズなアップグレード](/smooth-upgrade-tidb.md)
    - [TiFlashのアップグレードガイド](/tiflash-upgrade-guide.md)
  - スケール
    - [TiUPの使用（推奨）](/scale-tidb-using-tiup.md)
    - [TiDB Operatorの使用](https://docs.pingcap.com/tidb-in-kubernetes/stable/scale-a-tidb-cluster)
  - バックアップとリストア
    - [概要](/br/backup-and-restore-overview.md)
    - アーキテクチャ
      - [アーキテクチャの概要](/br/backup-and-restore-design.md)
      - [スナップショットバックアップとリストアのアーキテクチャ](/br/br-snapshot-architecture.md)
      - [ログバックアップとPITRアーキテクチャ](/br/br-log-architecture.md)
    - BRの使用
      - [概要](/br/br-use-overview.md)
      - [スナップショットバックアップとリストアガイド](/br/br-snapshot-guide.md)
      - [ログバックアップとPITRガイド](/br/br-pitr-guide.md)
      - [使用事例](/br/backup-and-restore-use-cases.md)
      - [バックアップストレージ](/br/backup-and-restore-storages.md)
    - BR CLIマニュアル
      - [概要](/br/use-br-command-line-tool.md)
      - [スナッショットバックアップとリストアコマンドマニュアル](/br/br-snapshot-manual.md)
      - [ログバックアップとPITRコマンドマニュアル](/br/br-pitr-manual.md)
    - リファレンス
      - BRの機能
        - [バックアップ自動チューニング](/br/br-auto-tune.md)
        - [バッチ作成テーブル](/br/br-batch-create-table.md)
        - [チェックポイントバックアップ](/br/br-checkpoint-backup.md)
        - [チェックポイントリストア](/br/br-checkpoint-restore.md)
      - [DumplingおよびTiDB Lightningを使用したデータのバックアップとリストア](/backup-and-restore-using-dumpling-lightning.md)
      - [RawKVのバックアップとリストア](/br/rawkv-backup-and-restore.md)
      - [増分バックアップとリストア](/br/br-incremental-guide.md)
  - クラスターの障害復旧（DR）
    - [DRソリューションの概要](/dr-solution-introduction.md)
    - [プライマリ-セカンダリDR](/dr-secondary-cluster.md)
    - [マルチレプリカクラスターDR](/dr-multi-replica.md)
    - [BRベースのDR](/dr-backup-restore.md)
    - [リソース制御](/tidb-resource-control.md)
    - [タイムゾーンの設定](/configure-time-zone.md)
    - [デイリーチェックリスト](/daily-check.md)
    - [TiFlashの維持](/tiflash/maintain-tiflash.md)
    - [TiUPを使用したTiDBのメンテナンス](/maintain-tidb-using-tiup.md)
    - [動的構成の変更](/dynamic-config.md)
    - [オンライン非安全リカバリ](/online-unsafe-recovery.md)
    - [プライマリとセカンダリクラスター間のデータのレプリケーション](/replicate-between-primary-and-secondary-clusters.md)
- 監視とアラート
  - [監視フレームワークの概要](/tidb-monitoring-framework.md)
  - [監視API](/tidb-monitoring-api.md)
  - [監視サービスのデプロイ](/deploy-monitoring-services.md)
  - [監視サービスのアップグレード](/upgrade-monitoring-services.md)
  - [Grafanaスナップショットのエクスポート](/exporting-grafana-snapshots.md)
  - [TiDBクラスターのアラートルール](/alert-rules.md)
  - [TiFlashのアラートルール](/tiflash/tiflash-alert-rules.md)
  - [監視サーバーのカスタマイズ設定](/tiup/customized-montior-in-tiup-environment.md)
  - [BRの監視とアラート](/br/br-monitoring-and-alert.md)
- トラブルシューティング
  - イシューサマリー
    - [TiDBトラブルシューティングマップ](/tidb-troubleshooting-map.md)
    - [TiDBクラスターセットアップのトラブルシューティング](/troubleshoot-tidb-cluster.md)
    - [TiFlashのトラブルシューティング](/tiflash/troubleshoot-tiflash.md)
  - イシューシナリオ
    - 遅いクエリ
      - [遅いクエリの特定](/identify-slow-queries.md)
      - [遅いクエリの分析](/analyze-slow-queries.md)
    - [TiDB OOM](/troubleshoot-tidb-oom.md)
    - [ホットスポット](/troubleshoot-hot-spot-issues.md)
    - [読み取りと書き込みの遅延増加](/troubleshoot-cpu-issues.md)
    - [楽観的トランザクションでの書き込み競合](/troubleshoot-write-conflicts.md)
    - [高いディスクI/O使用率](/troubleshoot-high-disk-io.md)
    - [ロックの競合](/troubleshoot-lock-conflicts.md)
    - [データとインデックスの不整合](/troubleshoot-data-inconsistency-errors.md)
  - 診断方法
    - [SQL診断](/information-schema/information-schema-sql-diagnostics.md)
    - [ステートメントサマリーテーブル](/statement-summary-tables.md)
    - [高コストのクエリの特定](/dashboard/top-sql.md)
    - [ログを使用した高コストのクエリの特定](/identify-expensive-queries.md)
    - [クラスターの現地情報の保存と復元](/sql-plan-replayer.md)
    - [古いリードとTiKVのsafe-tsの理解](/troubleshoot-stale-read.md)
  - [サポートリソース](/support.md)
- パフォーマンスチューニング
  - チューニングガイド
    - [パフォーマンスチューニングの概要](/performance-tuning-overview.md)
    - [パフォーマンス分析とチューニング](/performance-tuning-methods.md)
    - [OLTPシナリオのパフォーマンスチューニングプラクティス](/performance-tuning-practices.md)
    - [TiFlashパフォーマンス分析方法](/tiflash-performance-tuning-methods.md)
    - [TiCDCパフォーマンス分析方法](/ticdc-performance-tuning-methods.md)
    - [遅延の内訳](/latency-breakdown.md)
    - [パブリッククラウドでのTiDBのベストプラクティス](/best-practices-on-public-cloud.md)
  - 構成チューニング
    - [オペレーティングシステムのパフォーマンスチューニング](/tune-operating-system.md)
    - [TiDBメモリのチューニング](/configure-memory-usage.md)
    - [TiKVスレッドのチューニング](/tune-tikv-thread-performance.md)
    - [TiKVメモリのチューニング](/tune-tikv-memory-performance.md)
    - [TiKVフォローリード](/follower-read.md)
    - [リージョンのパフォーマンスチューニング](/tune-region-performance.md)
    - [TiFlashのパフォーマンスチューニング](/tiflash/tune-tiflash-performance.md)
    - [Coprocessorキャッシュ](/coprocessor-cache.md)
    - ガベージコレクション（GC）
      - [概要](/garbage-collection-overview.md)
      - [構成](/garbage-collection-configuration.md)
  - SQLチューニング
    - [概要](/sql-tuning-overview.md)
    - クエリ実行プランの理解
      - [概要](/explain-overview.md)
      - [`EXPLAIN`の解説](/explain-walkthrough.md)
      - [インデックス](/explain-indexes.md)
      - [結合](/explain-joins.md)
      - [MPPクエリ](/explain-mpp.md)
      - [サブクエリ](/explain-subqueries.md)
      - [集約](/explain-aggregation.md)
      - [ビュー](/explain-views.md)
      - [パーティション](/explain-partitions.md)
      - [インデックスマージ](/explain-index-merge.md)
    - SQL最適化プロセス
      - [概要](/sql-optimization-concepts.md)
      - ロジック最適化
        - [概要](/sql-logical-optimization.md)
        - [サブクエリ関連の最適化](/subquery-optimization.md)
        - [列の剪定](/column-pruning.md)
        - [相関サブクエリの切離し](/correlated-subquery-optimization.md)
        - [Max/Min の除去](/max-min-eliminate.md)
        - [述部のプッシュダウン](/predicate-push-down.md)
        - [パーティション剪定](/partition-pruning.md)
        - [TopN および Limit のプッシュダウン](/topn-limit-push-down.md)
        - [結合の再配置](/join-reorder.md)
        - [ウィンドウ関数からの TopN または Limit の導出](/derive-topn-from-window.md)
      - 物理最適化
        - [概要](/sql-physical-optimization.md)
        - [インデックスの選択](/choose-index.md)
        - [統計情報](/statistics.md)
        - [拡張統計情報](/extended-statistics.md)
        - [適切でないインデックスの解決](/wrong-index-solution.md)
        - [重複の最適化](/agg-distinct-optimization.md)
        - [コストモデル](/cost-model.md)
        - [実行時フィルタ](/runtime-filter.md)
      - [プリペアド実行計画キャッシュ](/sql-prepared-plan-cache.md)
      - [非プリペアド実行計画キャッシュ](/sql-non-prepared-plan-cache.md)
    - 実行計画の制御
      - [概要](/control-execution-plan.md)
      - [最適化ヒント](/optimizer-hints.md)
      - [SQL 実行計画管理](/sql-plan-management.md)
      - [最適化ルールと式プッシュダウンのブロックリスト](/blocklist-control-plan.md)
      - [最適化の修正コントロール](/optimizer-fix-controls.md)
- チュートリアル
  - [1リージョン内の複数の可用性ゾーンの展開](/multi-data-centers-in-one-city-deployment.md)
  - [2リージョン内の3つの可用性ゾーンの展開](/three-data-centers-in-two-cities-deployment.md)
  - [1リージョン内の2つの可用性ゾーンの展開](/two-data-centers-in-one-city-deployment.md)
  - 過去のデータの読み取り
    - ステール読み取りの使用（推奨）
      - [ステール読み取りの使用シナリオ](/stale-read.md)
      - [`As OF TIMESTAMP` を使用したステール読み取りの実行](/as-of-timestamp.md)
      - [`tidb_read_staleness` を使用したステール読み取りの実行](/tidb-read-staleness.md)
      - [`tidb_external_ts` を使用したステール読み取りの実行](/tidb-external-ts.md)
    - [`tidb_snapshot` システム変数の使用](/read-historical-data.md)
  - ベストプラクティス
    - [TiDB の使用](/best-practices/tidb-best-practices.md)
    - [Java アプリケーションの開発](/best-practices/java-app-best-practices.md)
    - [HAProxy の使用](/best-practices/haproxy-best-practices.md)
    - [高並列書き込み](/best-practices/high-concurrency-best-practices.md)
    - [Grafana モニタリング](/best-practices/grafana-monitor-best-practices.md)
    - [PD のスケジューリング](/best-practices/pd-scheduling-best-practices.md)
    - [大量リージョンの TiKV パフォーマンスチューニング](/best-practices/massive-regions-best-practices.md)
    - [3ノードのハイブリッド展開](/best-practices/three-nodes-hybrid-deployment.md)
    - [3データセンター展開でのローカル読み取り](/best-practices/three-dc-local-read.md)
    - [UUID の使用](/best-practices/uuid.md)
    - [読み取り専用ストレージノード](/best-practices/readonly-nodes.md)
  - [配置ルールの使用](/configure-placement-rules.md)
  - [ロードベース分割の使用](/configure-load-base-split.md)
  - [ストアリミットの使用](/configure-store-limit.md)
  - [DDL 実行原則とベストプラクティス](/ddl-introduction.md)
- TiDB ツール
  - [概要](/ecosystem-tool-user-guide.md)
  - [ユースケース](/ecosystem-tool-user-case.md)
  - [ダウンロード](/download-ecosystem-tools.md)
  - TiUP
    - [ドキュメントマップ](/tiup/tiup-documentation-guide.md)
    - [概要](/tiup/tiup-overview.md)
    - [用語と概念](/tiup/tiup-terminology-and-concepts.md)
    - [TiUP コンポーネントの管理](/tiup/tiup-component-management.md)
    - [FAQ](/tiup/tiup-faq.md)
    - [トラブルシューティングガイド](/tiup/tiup-troubleshooting-guide.md)
    - コマンドリファレンス
      - [概要](/tiup/tiup-reference.md)
      - TiUP コマンド
        - [tiup clean](/tiup/tiup-command-clean.md)
        - [tiup completion](/tiup/tiup-command-completion.md)
        - [tiup env](/tiup/tiup-command-env.md)
        - [tiup help](/tiup/tiup-command-help.md)
        - [tiup install](/tiup/tiup-command-install.md)
        - [tiup list](/tiup/tiup-command-list.md)
        - tiup mirror
          - [概要](/tiup/tiup-command-mirror.md)
          - [tiup mirror clone](/tiup/tiup-command-mirror-clone.md)
          - [tiup mirror genkey](/tiup/tiup-command-mirror-genkey.md)
          - [tiup mirror grant](/tiup/tiup-command-mirror-grant.md)
          - [tiup mirror init](/tiup/tiup-command-mirror-init.md)
          - [tiup mirror merge](/tiup/tiup-command-mirror-merge.md)
          - [tiup mirror modify](/tiup/tiup-command-mirror-modify.md)
          - [tiup mirror publish](/tiup/tiup-command-mirror-publish.md)
          - [tiup mirror rotate](/tiup/tiup-command-mirror-rotate.md)
          - [tiup mirror set](/tiup/tiup-command-mirror-set.md)
          - [tiup mirror sign](/tiup/tiup-command-mirror-sign.md)
        - [tiup status](/tiup/tiup-command-status.md)
        - [tiup telemetry](/tiup/tiup-command-telemetry.md)
        - [tiup uninstall](/tiup/tiup-command-uninstall.md)
        - [tiup update](/tiup/tiup-command-update.md)
      - TiUP Cluster コマンド
        - [概要](/tiup/tiup-component-cluster.md)
        - [tiup cluster audit](/tiup/tiup-component-cluster-audit.md)
        - [tiup cluster audit cleanup](/tiup/tiup-component-cluster-audit-cleanup.md)
        - [tiup cluster check](/tiup/tiup-component-cluster-check.md)
        - [tiup cluster clean](/tiup/tiup-component-cluster-clean.md)
        - [tiup cluster deploy](/tiup/tiup-component-cluster-deploy.md)
        - [tiup cluster destroy](/tiup/tiup-component-cluster-destroy.md)
        - [tiup cluster disable](/tiup/tiup-component-cluster-disable.md)
        - [tiup cluster display](/tiup/tiup-component-cluster-display.md)
        - [tiup cluster edit-config](/tiup/tiup-component-cluster-edit-config.md)
        - [tiup cluster enable](/tiup/tiup-component-cluster-enable.md)
        - [tiup cluster help](/tiup/tiup-component-cluster-help.md)
        - [tiup cluster import](/tiup/tiup-component-cluster-import.md)
        - [tiup cluster list](/tiup/tiup-component-cluster-list.md)
        - [tiup cluster meta backup](/tiup/tiup-component-cluster-meta-backup.md)
        - [tiup cluster meta restore](/tiup/tiup-component-cluster-meta-restore.md)
        - [tiup cluster patch](/tiup/tiup-component-cluster-patch.md)
        - [tiup cluster prune](/tiup/tiup-component-cluster-prune.md)
        - [tiup cluster reload](/tiup/tiup-component-cluster-reload.md)
        - [tiup cluster rename](/tiup/tiup-component-cluster-rename.md)
        - [tiup cluster replay](/tiup/tiup-component-cluster-replay.md)
        - [tiup cluster restart](/tiup/tiup-component-cluster-restart.md)
        - [tiup cluster scale-in](/tiup/tiup-component-cluster-scale-in.md)
        - [tiup cluster scale-out](/tiup/tiup-component-cluster-scale-out.md)
        - [tiup cluster start](/tiup/tiup-component-cluster-start.md)
        - [tiup cluster stop](/tiup/tiup-component-cluster-stop.md)
        - [tiup cluster template](/tiup/tiup-component-cluster-template.md)
        - [tiup cluster upgrade](/tiup/tiup-component-cluster-upgrade.md)
      - TiUP DM コマンド
        - [概要](/tiup/tiup-component-dm.md)
        - [tiup dm audit](/tiup/tiup-component-dm-audit.md)
        - [tiup dm deploy](/tiup/tiup-component-dm-deploy.md)
        - [tiup dm destroy](/tiup/tiup-component-dm-destroy.md)
        - [tiup dm disable](/tiup/tiup-component-dm-disable.md)
        - [tiup dm display](/tiup/tiup-component-dm-display.md)
        - [tiup dm edit-config](/tiup/tiup-component-dm-edit-config.md)
        - [tiup dm enable](/tiup/tiup-component-dm-enable.md)
        - [tiup dm help](/tiup/tiup-component-dm-help.md)
        - [tiup dm import](/tiup/tiup-component-dm-import.md)
        - [tiup dm list](/tiup/tiup-component-dm-list.md)
        - [tiup dm patch](/tiup/tiup-component-dm-patch.md)
        - [tiup dm prune](/tiup/tiup-component-dm-prune.md)
        - [tiup dm reload](/tiup/tiup-component-dm-reload.md)
        - [tiup dm replay](/tiup/tiup-component-dm-replay.md)
        - [tiup dm restart](/tiup/tiup-component-dm-restart.md)
        - [tiup dm scale-in](/tiup/tiup-component-dm-scale-in.md)
        - [tiup dm scale-out](/tiup/tiup-component-dm-scale-out.md)
        - [tiup dm start](/tiup/tiup-component-dm-start.md)
        - [tiup dm stop](/tiup/tiup-component-dm-stop.md)
        - [tiup dm template](/tiup/tiup-component-dm-template.md)
        - [tiup dm upgrade](/tiup/tiup-component-dm-upgrade.md)
    - [TiDB クラスタ トポロジ リファレンス](/tiup/tiup-cluster-topology-reference.md)
    - [DM クラスタ トポロジ リファレンス](/tiup/tiup-dm-topology-reference.md)
    - [ミラー リファレンス ガイド](/tiup/tiup-mirror-reference.md)
    - TiUP コンポーネント
      - [tiup-playground](/tiup/tiup-playground.md)
      - [tiup-cluster](/tiup/tiup-cluster.md)
      - [tiup-mirror](/tiup/tiup-mirror.md)
      - [tiup-bench](/tiup/tiup-bench.md)
  - [TiDB オペレータ](/tidb-operator-overview.md)
  - TiDB データ マイグレーション
    - [TiDB データマイグレーションについて](/dm/dm-overview.md)
    - [アーキテクチャ](/dm/dm-arch.md)
    - [クイックスタート](/dm/quick-start-with-dm.md)
    - [ベストプラクティス](/dm/dm-best-practices.md)
    - DM クラスタをデプロイする
      - [ハードウェアとソフトウェアの要件](/dm/dm-hardware-and-software-requirements.md)
      - [TiUP の使用 (推奨)](/dm/deploy-a-dm-cluster-using-tiup.md)
      - [TiUP オフラインの使用](/dm/deploy-a-dm-cluster-using-tiup-offline.md)
      - [バイナリの使用](/dm/deploy-a-dm-cluster-using-binary.md)
      - [Kubernetes の使用](https://docs.pingcap.com/tidb-in-kubernetes/dev/deploy-tidb-dm)
    - チュートリアル
      - [データソースを作成する](/dm/quick-start-create-source.md)
      - [データソースの管理](/dm/dm-manage-source.md)
      - [タスクの構成](/dm/dm-task-configuration-guide.md)
      - [シャードマージ](/dm/dm-shard-merge.md)
      - [テーブルルーティング](/dm/dm-table-routing.md)
      - [ブロックおよび許可リスト](/dm/dm-block-allow-table-lists.md)
      - [Binlog イベントフィルター](/dm/dm-binlog-event-filter.md)
      - [SQL 式を使用した DML のフィルタリング](/dm/feature-expression-filter.md)
      - [オンライン DDL ツールのサポート](/dm/dm-online-ddl-tool-support.md)
      - データマイグレーションタスクの管理
        - [タスクの事前チェック](/dm/dm-precheck.md)
        - [タスクの作成](/dm/dm-create-task.md)
        - [ステータスのクエリ](/dm/dm-query-status.md)
        - [タスクの一時停止](/dm/dm-pause-task.md)
        - [タスクの再開](/dm/dm-resume-task.md)
        - [タスクの停止](/dm/dm-stop-task.md)
    - 高度なチュートリアル
      - シャードテーブルからデータを統合および移行する
        - [概要](/dm/feature-shard-merge.md)
        - [悲観的モード](/dm/feature-shard-merge-pessimistic.md)
        - [楽観的モード](/dm/feature-shard-merge-optimistic.md)
        - [シャーディング DDL ロックを手動で処理する](/dm/manually-handling-sharding-ddl-locks.md)
      - [GH-ost/PT-osc を使用する MySQL データベースからの移行](/dm/feature-online-ddl.md)
      - [より多くの列を持つダウンストリーム TiDB テーブルへのデータ移行](/migrate-with-more-columns-downstream.md)
      - [連続データバリデーション](/dm/dm-continuous-data-validation.md)
    - メンテナンス
      - クラスタのアップグレード
        - [TiUP を使用して DM クラスタを保守する (推奨)](/dm/maintain-dm-using-tiup.md)
        - [v1.0.x から v2.0+ への手動アップグレード](/dm/manually-upgrade-dm-1.0-to-2.0.md)
      - ツール
        - [WebUI を使用して管理](/dm/dm-webui-guide.md)
        - [dmctl を使用した管理](/dm/dmctl-introduction.md)
      - パフォーマンスチューニング
        - [ベンチマーク](/dm/dm-benchmark-v5.4.0.md)
        - [構成の最適化](/dm/dm-tune-configuration.md)
        - [DM のパフォーマンステスト](/dm/dm-performance-test.md)
        - [パフォーマンスの問題を処理する](/dm/dm-handle-performance-issues.md)
      - データソースの管理
        - [移行対象となる MySQL インスタンスの切り替え](/dm/usage-scenario-master-slave-switch.md)
      - タスクの管理
        - [失敗した DDL ステートメントを処理する](/dm/handle-failed-ddl-statements.md)
        - [移行対象テーブルのスキーマを管理する](/dm/dm-manage-schema.md)
      - [クラスタのデータソースおよびタスク構成のエクスポートとインポート](/dm/dm-export-import-config.md)
      - [アラートの処理](/dm/dm-handle-alerts.md)
      - [デイリーチェック](/dm/dm-daily-check.md)
    - リファレンス
      - アーキテクチャ
        - [DM-worker](/dm/dm-worker-intro.md)
        - [セーフモード](/dm/dm-safe-mode.md)
        - [リレーログ](/dm/relay-log.md)
        - [DDL 処理](/dm/dm-ddl-compatible.md)
      - メカニズム
        - [DML レプリケーションメカニズム](/dm/dm-replication-logic.md)
      - コマンドライン
        - [DM-master および DM-worker](/dm/dm-command-line-flags.md)
      - 設定ファイル
        - [概要](/dm/dm-config-overview.md)
        - [アップストリームデータベースの設定](/dm/dm-source-configuration-file.md)
        - [タスクの設定](/dm/task-configuration-file-full.md)
        - [DM-master の設定](/dm/dm-master-configuration-file.md)
        - [DM-worker の設定](/dm/dm-worker-configuration-file.md)
        - [テーブルセレクタ](/dm/table-selector.md)
      - [OpenAPI](/dm/dm-open-api.md)
      - [互換性カタログ](/dm/dm-compatibility-catalog.md)
      - セキュリティ
        - [DM 接続を TLS で有効にする](/dm/dm-enable-tls.md)
        - [自己署名証明書の生成](/dm/dm-generate-self-signed-certificates.md)
      - 監視とアラート
        - [監視メトリクス](/dm/monitor-a-dm-cluster.md)
        - [アラートルール](/dm/dm-alert-rules.md)
      - [エラーコード](/dm/dm-error-handling.md#handle-common-errors)
      - [用語集](/dm/dm-glossary.md)
      - サンプル
        - [DM を使用してデータを移行する](/dm/migrate-data-using-dm.md)
        - [データマイグレーションタスクを作成する](/dm/quick-start-create-task.md)
        - [シャードマージシナリオでのデータ移行のベストプラクティス](/dm/shard-merge-best-practices.md)
      - トラブルシューティング
        - [よくある質問](/dm/dm-faq.md)
        - [エラーの処理](/dm/dm-error-handling.md)
      - [リリースノート](/dm/dm-release-notes.md)
  - TiDB Lightning
    - [概要](/tidb-lightning/tidb-lightning-overview.md)
- [はじめに](/get-started-with-tidb-lightning.md)
- [TiDB Lightningをデプロイする](/tidb-lightning/deploy-tidb-lightning.md)
- [データベースへの要件](/tidb-lightning/tidb-lightning-requirements.md)
- データソース
  - [データの一致ルール](/tidb-lightning/tidb-lightning-data-source.md)
  - [CSV](/tidb-lightning/tidb-lightning-data-source.md#csv)
  - [SQL](/tidb-lightning/tidb-lightning-data-source.md#sql)
  - [Parquet](/tidb-lightning/tidb-lightning-data-source.md#parquet)
  - [カスタマイズされたファイル](/tidb-lightning/tidb-lightning-data-source.md#match-customized-files)
- 物理的インポートモード
  - [要件と制限事項](/tidb-lightning/tidb-lightning-physical-import-mode.md)
  - [物理的インポートモードを使用する](/tidb-lightning/tidb-lightning-physical-import-mode-usage.md)
- 論理インポートモード
  - [要件と制限事項](/tidb-lightning/tidb-lightning-logical-import-mode.md)
  - [論理インポートモードの使用](/tidb-lightning/tidb-lightning-logical-import-mode-usage.md)
- [事前チェック](/tidb-lightning/tidb-lightning-prechecks.md)
- [テーブルフィルタ](/table-filter.md)
- [チェックポイント](/tidb-lightning/tidb-lightning-checkpoints.md)
- [並列でデータをインポートする](/tidb-lightning/tidb-lightning-distributed-import.md)
- [エラーの解決](/tidb-lightning/tidb-lightning-error-resolution.md)
- [トラブルシューティング](/tidb-lightning/troubleshoot-tidb-lightning.md)
- 参照
  - [設定ファイル](/tidb-lightning/tidb-lightning-configuration.md)
  - [コマンドラインフラグ](/tidb-lightning/tidb-lightning-command-line-full.md)
  - [モニタリング](/tidb-lightning/monitor-tidb-lightning.md)
  - [Webインターフェース](/tidb-lightning/tidb-lightning-web-interface.md)
  - [FAQ](/tidb-lightning/tidb-lightning-faq.md)
  - [用語集](/tidb-lightning/tidb-lightning-glossary.md)
- [Dumpling](/dumpling-overview.md)
- TiCDC
  - [概要](/ticdc/ticdc-overview.md)
  - [デプロイとメンテナンス](/ticdc/deploy-ticdc.md)
  - Changefeed
    - [概要](/ticdc/ticdc-changefeed-overview.md)
    - Changefeedの作成
      - [MySQL互換データベースにデータをレプリケート](/ticdc/ticdc-sink-to-mysql.md)
      - [Kafkaにデータをレプリケート](/ticdc/ticdc-sink-to-kafka.md)
      - [Pulsarにデータをレプリケート](/ticdc/ticdc-sink-to-pulsar.md)
      - [ストレージサービスにデータをレプリケート](/ticdc/ticdc-sink-to-cloud-storage.md)
    - [Changefeedの管理](/ticdc/ticdc-manage-changefeed.md)
    - [ログフィルタ](/ticdc/ticdc-filter.md)
    - [DDLレプリケーション](/ticdc/ticdc-ddl.md)
    - [双方向レプリケーション](/ticdc/ticdc-bidirectional-replication.md)
    - [単一行データのデータ整合性検証](/ticdc/ticdc-integrity-check.md)
    - [TiDBの上流/下流クラスターのデータ整合性検証](/ticdc/ticdc-upstream-downstream-check.md)
  - モニタリングとアラート
    - [モニタリングメトリクスの概要](/ticdc/ticdc-summary-monitor.md)
    - [モニタリングメトリクスの詳細](/ticdc/monitor-ticdc.md)
    - [アラートルール](/ticdc/ticdc-alert-rules.md)
  - 参照
    - [アーキテクチャ](/ticdc/ticdc-architecture.md)
    - [TiCDCサーバー設定](/ticdc/ticdc-server-config.md)
    - [TiCDC Changefeed設定](/ticdc/ticdc-changefeed-config.md)
    - 出力プロトコル
      - [TiCDC Avroプロトコル](/ticdc/ticdc-avro-protocol.md)
      - [TiCDC Canal-JSONプロトコル](/ticdc/ticdc-canal-json.md)
      - [TiCDC Openプロトコル](/ticdc/ticdc-open-protocol.md)
      - [TiCDC CSVプロトコル](/ticdc/ticdc-csv.md)
    - [TiCDCオープンAPI v2](/ticdc/ticdc-open-api-v2.md)
    - [TiCDCオープンAPI v1](/ticdc/ticdc-open-api.md)
    - TiCDCデータ消費
      - [AvroベースのTiCDC行データチェックサム検証](/ticdc/ticdc-avro-checksum-verification.md)
      - [ストレージシンクコンシューマーの開発ガイド](/ticdc/ticdc-storage-consumer-dev-guide.md)
    - [互換性](/ticdc/ticdc-compatibility.md)
  - [トラブルシューティング](/ticdc/troubleshoot-ticdc.md)
  - [FAQ](/ticdc/ticdc-faq.md)
  - [用語集](/ticdc/ticdc-glossary.md)
- TiDB Binlog
  - [概要](/tidb-binlog/tidb-binlog-overview.md)
  - [クイックスタート](/tidb-binlog/get-started-with-tidb-binlog.md)
  - [デプロイ](/tidb-binlog/deploy-tidb-binlog.md)
  - [メンテナンス](/tidb-binlog/maintain-tidb-binlog-cluster.md)
  - [構成](/tidb-binlog/tidb-binlog-configuration-file.md)
    - [Pump](/tidb-binlog/tidb-binlog-configuration-file.md#pump)
    - [Drainer](/tidb-binlog/tidb-binlog-configuration-file.md#drainer)
  - [アップグレード](/tidb-binlog/upgrade-tidb-binlog.md)
  - [モニタリング](/tidb-binlog/monitor-tidb-binlog-cluster.md)
  - [Reparo](/tidb-binlog/tidb-binlog-reparo.md)
  - [binlogctl](/tidb-binlog/binlog-control.md)
  - [Binlogコンシューマークライアント](/tidb-binlog/binlog-consumer-client.md)
  - [TiDB Binlogリレーログ](/tidb-binlog/tidb-binlog-relay-log.md)
  - [TiDBクラスター間の双方向レプリケーション](/tidb-binlog/bidirectional-replication-between-tidb-clusters.md)
  - [用語集](/tidb-binlog/tidb-binlog-glossary.md)
  - トラブルシューティング
    - [トラブルシューティング](/tidb-binlog/troubleshoot-tidb-binlog.md)
    - [エラーの処理](/tidb-binlog/handle-tidb-binlog-errors.md)
  - [FAQ](/tidb-binlog/tidb-binlog-faq.md)
- PingCAP Clinic診断サービス
  - [概要](/clinic/clinic-introduction.md)
  - [クイックスタート](/clinic/quick-start-with-clinic.md)
  - [PingCAP Clinicを使用したクラスタのトラブルシューティング](/clinic/clinic-user-guide-for-tiup.md)
  - [PingCAP Clinic診断データ](/clinic/clinic-data-instruction-for-tiup.md)
- TiSpark
  - [ユーザーガイド](/tispark-overview.md)
- sync-diff-inspector
  - [概要](/sync-diff-inspector/sync-diff-inspector-overview.md)
  - [異なるスキーマ/テーブル名を持つテーブルのデータチェック](/sync-diff-inspector/route-diff.md)
  - [シャーディングシナリオでのデータチェック](/sync-diff-inspector/shard-diff.md)
  - [DMレプリケーションシナリオでのデータチェック](/sync-diff-inspector/dm-diff.md)
- 参照
  - クラスターアーキテクチャ
    - [概要](/tidb-architecture.md)
    - [ストレージ](/tidb-storage.md)
    - [コンピューティング](/tidb-computing.md)
    - [スケジューリング](/tidb-scheduling.md)
    - [TSO](/tso.md)
  - ストレージエンジン - TiKV
    - [TiKV概要](/tikv-overview.md)
    - [RocksDB概要](/storage-engine/rocksdb-overview.md)
    - [Titan概要](/storage-engine/titan-overview.md)
    - [Titan構成](/storage-engine/titan-configuration.md)
    - [分割されたRaft KV](/partitioned-raft-kv.md)
  - ストレージエンジン - TiFlash
    - [概要](/tiflash/tiflash-overview.md)
    - [TiFlashレプリカの作成](/tiflash/create-tiflash-replicas.md)
- [Use TiDB to Read TiFlash Replicas](/tiflash/use-tidb-to-read-tiflash.md)
  - TiDBでTiFlashのレプリカを読む
- [Use TiSpark to Read TiFlash Replicas](/tiflash/use-tispark-to-read-tiflash.md)
  - TiSparkを使用してTiFlashのレプリカを読む
- [Use MPP Mode](/tiflash/use-tiflash-mpp-mode.md)
  - MPPモードを使用する
- [Use FastScan](/tiflash/use-fastscan.md)
  - FastScanを使用する
- [Disaggregated Storage and Compute Architecture and S3 Support](/tiflash/tiflash-disaggregated-and-s3.md)
  - 分散ストレージおよびコンピューティングアーキテクチャとS3サポートの使用
- [Supported Push-down Calculations](/tiflash/tiflash-supported-pushdown-calculations.md)
  - サポートされているプッシュダウン計算
- [TiFlash Query Result Materialization](/tiflash/tiflash-results-materialization.md)
  - TiFlashクエリ結果の素材化
- [TiFlash Late Materialization](/tiflash/tiflash-late-materialization.md)
  - TiFlashの遅い素材化
- [Spill to Disk](/tiflash/tiflash-spill-disk.md)
  - ディスクへの溢れ
- [Data Validation](/tiflash/tiflash-data-validation.md)
  - データの検証
- [Compatibility](/tiflash/tiflash-compatibility.md)
  - 互換性
- [Pipeline Execution Model](/tiflash/tiflash-pipeline-model.md)
  - パイプライン実行モデル
- [System Variables](/system-variables.md)
  - システム変数
- Configuration File Parameters
  - 設定ファイルパラメータ
    - [tidb-server](/tidb-configuration-file.md)
      - tidb-server
    - [tikv-server](/tikv-configuration-file.md)
      - tikv-server
    - [tiflash-server](/tiflash/tiflash-configuration.md)
      - tiflash-server
    - [pd-server](/pd-configuration-file.md)
      - pd-server
- CLI
  - CLI
    - [tikv-ctl](/tikv-control.md)
      - tikv-ctl
    - [pd-ctl](/pd-control.md)
      - pd-ctl
    - [tidb-ctl](/tidb-control.md)
      - tidb-ctl
    - [pd-recover](/pd-recover.md)
      - pd-recover
- Command Line Flags
  - コマンドラインフラグ
    - [tidb-server](/command-line-flags-for-tidb-configuration.md)
      - tidb-serverのコマンドラインフラグ
    - [tikv-server](/command-line-flags-for-tikv-configuration.md)
      - tikv-serverのコマンドラインフラグ
    - [tiflash-server](/tiflash/tiflash-command-line-flags.md)
      - tiflash-serverのコマンドラインフラグ
    - [pd-server](/command-line-flags-for-pd-configuration.md)
      - pd-serverのコマンドラインフラグ
- Key Monitoring Metrics
  - キー監視メトリクス
    - [Overview](/grafana-overview-dashboard.md)
      - 概要
    - [Performance Overview](/grafana-performance-overview-dashboard.md)
      - パフォーマンス概要
    - [TiDB](/grafana-tidb-dashboard.md)
      - TiDB
    - [PD](/grafana-pd-dashboard.md)
      - PD
    - [TiKV](/grafana-tikv-dashboard.md)
      - TiKV
    - [TiFlash](/tiflash/monitor-tiflash.md)
      - TiFlash
    - [TiCDC](/ticdc/monitor-ticdc.md)
      - TiCDC
    - [Resource Control](/grafana-resource-control-dashboard.md)
      - リソース制御
- Security
  - セキュリティ
    - [Enable TLS Between TiDB Clients and Servers](/enable-tls-between-clients-and-servers.md)
      - TiDBクライアントとサーバ間でTLSを有効にする
    - [Enable TLS Between TiDB Components](/enable-tls-between-components.md)
      - TiDBコンポーネント間でTLSを有効にする
    - [Generate Self-signed Certificates](/generate-self-signed-certificates.md)
      - セルフサインド証明書の生成
    - [Encryption at Rest](/encryption-at-rest.md)
      - 休止時の暗号化
    - [Enable Encryption for Disk Spill](/enable-disk-spill-encrypt.md)
      - ディスクへの溢れの暗号化を有効にする
    - [Log Redaction](/log-redaction.md)
      - ログの編集
- Privileges
  - 権限
    - [Security Compatibility with MySQL](/security-compatibility-with-mysql.md)
      - MySQLとのセキュリティ互換性
    - [Privilege Management](/privilege-management.md)
      - 権限管理
    - [User Account Management](/user-account-management.md)
      - ユーザーアカウントの管理
    - [TiDB Password Management](/password-management.md)
      - TiDBパスワード管理
    - [Role-Based Access Control](/role-based-access-control.md)
      - ロールベースのアクセス制御
    - [Certificate-Based Authentication](/certificate-authentication.md)
      - 証明書ベースの認証
- SQL
  - SQL
    - SQL Language Structure and Syntax
      - SQL言語の構造と構文
        - Attributes
          - 属性
            - [AUTO_INCREMENT](/auto-increment.md)
              - AUTO_INCREMENT
            - [AUTO_RANDOM](/auto-random.md)
              - AUTO_RANDOM
            - [SHARD_ROW_ID_BITS](/shard-row-id-bits.md)
              - SHARD_ROW_ID_BITS
        - [Literal Values](/literal-values.md)
          - リテラル値
        - [Schema Object Names](/schema-object-names.md)
          - スキーマオブジェクト名
        - [Keywords and Reserved Words](/keywords.md)
          - キーワードと予約語
        - [User-Defined Variables](/user-defined-variables.md)
          - ユーザー定義変数
        - [Expression Syntax](/expression-syntax.md)
          - 式の構文
        - [Comment Syntax](/comment-syntax.md)
          - コメントの構文
    - SQL Statements
      - SQLステートメント
        - [`ADD COLUMN`](/sql-statements/sql-statement-add-column.md)
          - `ADD COLUMN`
        - [`ADD INDEX`](/sql-statements/sql-statement-add-index.md)
          - `ADD INDEX`
        - [`ADMIN`](/sql-statements/sql-statement-admin.md)
          - `ADMIN`
        - [`ADMIN CANCEL DDL`](/sql-statements/sql-statement-admin-cancel-ddl.md)
          - `ADMIN CANCEL DDL`
        - [`ADMIN CHECKSUM TABLE`](/sql-statements/sql-statement-admin-checksum-table.md)
          - `ADMIN CHECKSUM TABLE`
        - [`ADMIN CHECK [TABLE|INDEX]`](/sql-statements/sql-statement-admin-check-table-index.md)
          - `ADMIN CHECK [TABLE|INDEX]`
        - [`ADMIN CLEANUP`](/sql-statements/sql-statement-admin-cleanup.md)
          - `ADMIN CLEANUP`
        - [`ADMIN PAUSE DDL`](/sql-statements/sql-statement-admin-pause-ddl.md)
          - `ADMIN PAUSE DDL`
        - [`ADMIN RECOVER INDEX`](/sql-statements/sql-statement-admin-recover.md)
          - `ADMIN RECOVER INDEX`
        - [`ADMIN RESUME DDL`](/sql-statements/sql-statement-admin-resume-ddl.md)
          - `ADMIN RESUME DDL`
        - [`ADMIN SHOW DDL [JOBS|JOB QUERIES]`](/sql-statements/sql-statement-admin-show-ddl.md)
          - `ADMIN SHOW DDL [JOBS|JOB QUERIES]`
        - [`ADMIN SHOW TELEMETRY`](/sql-statements/sql-statement-admin-show-telemetry.md)
          - `ADMIN SHOW TELEMETRY`
        - [`ALTER DATABASE`](/sql-statements/sql-statement-alter-database.md)
          - `ALTER DATABASE`
        - [`ALTER INDEX`](/sql-statements/sql-statement-alter-index.md)
          - `ALTER INDEX`
        - [`ALTER INSTANCE`](/sql-statements/sql-statement-alter-instance.md)
          - `ALTER INSTANCE`
        - [`ALTER PLACEMENT POLICY`](/sql-statements/sql-statement-alter-placement-policy.md)
          - `ALTER PLACEMENT POLICY`
        - [`ALTER RANGE`](/sql-statements/sql-statement-alter-range.md)
          - `ALTER RANGE`
        - [`ALTER RESOURCE GROUP`](/sql-statements/sql-statement-alter-resource-group.md)
          - `ALTER RESOURCE GROUP`
        - [`ALTER TABLE`](/sql-statements/sql-statement-alter-table.md)
          - `ALTER TABLE`
        - [`ALTER TABLE COMPACT`](/sql-statements/sql-statement-alter-table-compact.md)
          - `ALTER TABLE COMPACT`
        - [`ALTER USER`](/sql-statements/sql-statement-alter-user.md)
          - `ALTER USER`
        - [`ANALYZE TABLE`](/sql-statements/sql-statement-analyze-table.md)
          - `ANALYZE TABLE`
        - [`BACKUP`](/sql-statements/sql-statement-backup.md)
          - `BACKUP`
        - [`BATCH`](/sql-statements/sql-statement-batch.md)
          - `BATCH`
        - [`BEGIN`](/sql-statements/sql-statement-begin.md)
          - `BEGIN`
        - [`CALIBRATE RESOURCE`](/sql-statements/sql-statement-calibrate-resource.md)
          - `CALIBRATE RESOURCE`
        - [`CANCEL IMPORT JOB`](/sql-statements/sql-statement-cancel-import-job.md)
          - `CANCEL IMPORT JOB`
        - [`CHANGE COLUMN`](/sql-statements/sql-statement-change-column.md)
          - `CHANGE COLUMN`
        - [`COMMIT`](/sql-statements/sql-statement-commit.md)
          - `COMMIT`
        - [`CHANGE DRAINER`](/sql-statements/sql-statement-change-drainer.md)
          - `CHANGE DRAINER`
        - [`CHANGE PUMP`](/sql-statements/sql-statement-change-pump.md)
          - `CHANGE PUMP`
        - [`CREATE BINDING`](/sql-statements/sql-statement-create-binding.md)
          - `CREATE BINDING`
        - [`CREATE DATABASE`](/sql-statements/sql-statement-create-database.md)
          - `CREATE DATABASE`
        - [`CREATE INDEX`](/sql-statements/sql-statement-create-index.md)
          - `CREATE INDEX`
        - [`CREATE PLACEMENT POLICY`](/sql-statements/sql-statement-create-placement-policy.md)
          - `CREATE PLACEMENT POLICY`
        - [`CREATE RESOURCE GROUP`](/sql-statements/sql-statement-create-resource-group.md)
          - `CREATE RESOURCE GROUP`
        - [`CREATE ROLE`](/sql-statements/sql-statement-create-role.md)
          - `CREATE ROLE`
        - [`CREATE SEQUENCE`](/sql-statements/sql-statement-create-sequence.md)
          - `CREATE SEQUENCE`
        - [`CREATE TABLE LIKE`](/sql-statements/sql-statement-create-table-like.md)
          - `CREATE TABLE LIKE`
        - [`CREATE TABLE`](/sql-statements/sql-statement-create-table.md)
          - `CREATE TABLE`
        - [`CREATE USER`](/sql-statements/sql-statement-create-user.md)
          - `CREATE USER`
        - [`CREATE VIEW`](/sql-statements/sql-statement-create-view.md)
          - `CREATE VIEW`
        - [`DEALLOCATE`](/sql-statements/sql-statement-deallocate.md)
          - `DEALLOCATE`
        - [`DELETE`](/sql-statements/sql-statement-delete.md)
          - `DELETE`
        - [`DESC`](/sql-statements/sql-statement-desc.md)
          - `DESC`
        - [`DESCRIBE`](/sql-statements/sql-statement-describe.md)
          - `DESCRIBE`
        - [`DO`](/sql-statements/sql-statement-do.md)
          - `DO`
        - [`DROP BINDING`](/sql-statements/sql-statement-drop-binding.md)
          - `DROP BINDING`
        - [`DROP COLUMN`](/sql-statements/sql-statement-drop-column.md)
          - `DROP COLUMN`
        - [`DROP DATABASE`](/sql-statements/sql-statement-drop-database.md)
          - `DROP DATABASE`
        - [`DROP INDEX`](/sql-statements/sql-statement-drop-index.md)
          - `DROP INDEX`
        - [`DROP PLACEMENT POLICY`](/sql-statements/sql-statement-drop-placement-policy.md)
          - `DROP PLACEMENT POLICY`
        - [`DROP RESOURCE GROUP`](/sql-statements/sql-statement-drop-resource-group.md)
          - `DROP RESOURCE GROUP`
        - [`DROP ROLE`](/sql-statements/sql-statement-drop-role.md)
          - `DROP ROLE`
        - [`DROP SEQUENCE`](/sql-statements/sql-statement-drop-sequence.md)
          - `DROP SEQUENCE`
        - [`DROP STATS`](/sql-statements/sql-statement-drop-stats.md)
          - `DROP STATS`
        - [`DROP TABLE`](/sql-statements/sql-statement-drop-table.md)
          - `DROP TABLE`
        - [`DROP USER`](/sql-statements/sql-statement-drop-user.md)
          - `DROP USER`
        - [`DROP VIEW`](/sql-statements/sql-statement-drop-view.md)
          - `DROP VIEW`
      - [`EXECUTE`](/sql-statements/sql-statement-execute.md)
      - [`EXPLAIN ANALYZE`](/sql-statements/sql-statement-explain-analyze.md)
      - [`EXPLAIN`](/sql-statements/sql-statement-explain.md)
      - [`FLASHBACK CLUSTER TO TIMESTAMP`](/sql-statements/sql-statement-flashback-to-timestamp.md)
      - [`FLASHBACK DATABASE`](/sql-statements/sql-statement-flashback-database.md)
      - [`FLASHBACK TABLE`](/sql-statements/sql-statement-flashback-table.md)
      - [`FLUSH PRIVILEGES`](/sql-statements/sql-statement-flush-privileges.md)
      - [`FLUSH STATUS`](/sql-statements/sql-statement-flush-status.md)
      - [`FLUSH TABLES`](/sql-statements/sql-statement-flush-tables.md)
      - [`GRANT <privileges>`](/sql-statements/sql-statement-grant-privileges.md)
      - [`GRANT <role>`](/sql-statements/sql-statement-grant-role.md)
      - [`IMPORT INTO`](/sql-statements/sql-statement-import-into.md)
      - [`INSERT`](/sql-statements/sql-statement-insert.md)
      - [`KILL`](/sql-statements/sql-statement-kill.md)
      - [`LOAD DATA`](/sql-statements/sql-statement-load-data.md)
      - [`LOAD STATS`](/sql-statements/sql-statement-load-stats.md)
      - [`LOCK STATS`](/sql-statements/sql-statement-lock-stats.md)
      - [`[LOCK|UNLOCK] TABLES`](/sql-statements/sql-statement-lock-tables-and-unlock-tables.md)
      - [`MODIFY COLUMN`](/sql-statements/sql-statement-modify-column.md)
      - [`PREPARE`](/sql-statements/sql-statement-prepare.md)
      - [`QUERY WATCH`](/sql-statements/sql-statement-query-watch.md)
      - [`RECOVER TABLE`](/sql-statements/sql-statement-recover-table.md)
      - [`RENAME USER`](/sql-statements/sql-statement-rename-user.md)
      - [`RENAME INDEX`](/sql-statements/sql-statement-rename-index.md)
      - [`RENAME TABLE`](/sql-statements/sql-statement-rename-table.md)
      - [`REPLACE`](/sql-statements/sql-statement-replace.md)
      - [`RESTORE`](/sql-statements/sql-statement-restore.md)
      - [`REVOKE <privileges>`](/sql-statements/sql-statement-revoke-privileges.md)
      - [`REVOKE <role>`](/sql-statements/sql-statement-revoke-role.md)
      - [`ROLLBACK`](/sql-statements/sql-statement-rollback.md)
      - [`SAVEPOINT`](/sql-statements/sql-statement-savepoint.md)
      - [`SELECT`](/sql-statements/sql-statement-select.md)
      - [`SET DEFAULT ROLE`](/sql-statements/sql-statement-set-default-role.md)
      - [`SET [NAMES|CHARACTER SET]`](/sql-statements/sql-statement-set-names.md)
      - [`SET PASSWORD`](/sql-statements/sql-statement-set-password.md)
      - [`SET RESOURCE GROUP`](/sql-statements/sql-statement-set-resource-group.md)
      - [`SET ROLE`](/sql-statements/sql-statement-set-role.md)
      - [`SET TRANSACTION`](/sql-statements/sql-statement-set-transaction.md)
      - [`SET <variable>`](/sql-statements/sql-statement-set-variable.md)
      - [`SHOW ANALYZE STATUS`](/sql-statements/sql-statement-show-analyze-status.md)
      - [`SHOW [BACKUPS|RESTORES]`](/sql-statements/sql-statement-show-backups.md)
      - [`SHOW BINDINGS`](/sql-statements/sql-statement-show-bindings.md)
      - [`SHOW BUILTINS`](/sql-statements/sql-statement-show-builtins.md)
      - [`SHOW CHARACTER SET`](/sql-statements/sql-statement-show-character-set.md)
      - [`SHOW COLLATION`](/sql-statements/sql-statement-show-collation.md)
      - [`SHOW COLUMNS FROM`](/sql-statements/sql-statement-show-columns-from.md)
      - [`SHOW CONFIG`](/sql-statements/sql-statement-show-config.md)
      - [`SHOW CREATE DATABASE`](/sql-statements/sql-statement-show-create-database.md)
      - [`SHOW CREATE PLACEMENT POLICY`](/sql-statements/sql-statement-show-create-placement-policy.md)
      - [`SHOW CREATE RESOURCE GROUP`](/sql-statements/sql-statement-show-create-resource-group.md)
      - [`SHOW CREATE SEQUENCE`](/sql-statements/sql-statement-show-create-sequence.md)
      - [`SHOW CREATE TABLE`](/sql-statements/sql-statement-show-create-table.md)
      - [`SHOW CREATE USER`](/sql-statements/sql-statement-show-create-user.md)  
      - [`SHOW DATABASES`](/sql-statements/sql-statement-show-databases.md)
      - [`SHOW DRAINER STATUS`](/sql-statements/sql-statement-show-drainer-status.md)
      - [`SHOW ENGINES`](/sql-statements/sql-statement-show-engines.md)
      - [`SHOW ERRORS`](/sql-statements/sql-statement-show-errors.md)
      - [`SHOW FIELDS FROM`](/sql-statements/sql-statement-show-fields-from.md)
      - [`SHOW GRANTS`](/sql-statements/sql-statement-show-grants.md)
      - [`SHOW IMPORT JOB`](/sql-statements/sql-statement-show-import-job.md)
      - [`SHOW INDEXES`](/sql-statements/sql-statement-show-indexes.md)
      - [`SHOW MASTER STATUS`](/sql-statements/sql-statement-show-master-status.md)
      - [`SHOW PLACEMENT`](/sql-statements/sql-statement-show-placement.md)
      - [`SHOW PLACEMENT FOR`](/sql-statements/sql-statement-show-placement-for.md)
      - [`SHOW PLACEMENT LABELS`](/sql-statements/sql-statement-show-placement-labels.md)
      - [`SHOW PLUGINS`](/sql-statements/sql-statement-show-plugins.md)
      - [`SHOW PRIVILEGES`](/sql-statements/sql-statement-show-privileges.md)
      - [`SHOW PROCESSSLIST`](/sql-statements/sql-statement-show-processlist.md)
      - [`SHOW PROFILES`](/sql-statements/sql-statement-show-profiles.md)
      - [`SHOW PUMP STATUS`](/sql-statements/sql-statement-show-pump-status.md)
      - [`SHOW SCHEMAS`](/sql-statements/sql-statement-show-schemas.md)
      - [`SHOW STATS_HEALTHY`](/sql-statements/sql-statement-show-stats-healthy.md)
      - [`SHOW STATS_HISTOGRAMS`](/sql-statements/sql-statement-show-histograms.md)
      - [`SHOW STATS_LOCKED`](/sql-statements/sql-statement-show-stats-locked.md)
      - [`SHOW STATS_META`](/sql-statements/sql-statement-show-stats-meta.md)
      - [`SHOW STATUS`](/sql-statements/sql-statement-show-status.md)
      - [`SHOW TABLE NEXT_ROW_ID`](/sql-statements/sql-statement-show-table-next-rowid.md)
      - [`SHOW TABLE REGIONS`](/sql-statements/sql-statement-show-table-regions.md)
      - [`SHOW TABLE STATUS`](/sql-statements/sql-statement-show-table-status.md)
      - [`SHOW TABLES`](/sql-statements/sql-statement-show-tables.md)
      - [`SHOW VARIABLES`](/sql-statements/sql-statement-show-variables.md)
      - [`SHOW WARNINGS`](/sql-statements/sql-statement-show-warnings.md)
      - [`SHUTDOWN`](/sql-statements/sql-statement-shutdown.md)
      - [`SPLIT REGION`](/sql-statements/sql-statement-split-region.md)
      - [`START TRANSACTION`](/sql-statements/sql-statement-start-transaction.md)
      - [`TABLE`](/sql-statements/sql-statement-table.md)
      - [`TRACE`](/sql-statements/sql-statement-trace.md)
      - [`TRUNCATE`](/sql-statements/sql-statement-truncate.md)
      - [`UNLOCK STATS`](/sql-statements/sql-statement-unlock-stats.md)
      - [`UPDATE`](/sql-statements/sql-statement-update.md)
      - [`USE`](/sql-statements/sql-statement-use.md)
      - [`WITH`](/sql-statements/sql-statement-with.md)
    - Data Types
      - [Overview](/data-type-overview.md)
      - [Default Values](/data-type-default-values.md)
      - [Numeric Types](/data-type-numeric.md)
      - [Date and Time Types](/data-type-date-and-time.md)
      - [String Types](/data-type-string.md)
      - [JSON Type](/data-type-json.md)
    - Functions and Operators
      - [Overview](/functions-and-operators/functions-and-operators-overview.md)
      - [Type Conversion in Expression Evaluation](/functions-and-operators/type-conversion-in-expression-evaluation.md)
      - [Operators](/functions-and-operators/operators.md)
      - [Control Flow Functions](/functions-and-operators/control-flow-functions.md)
      - [String Functions](/functions-and-operators/string-functions.md)
      - [Numeric Functions and Operators](/functions-and-operators/numeric-functions-and-operators.md)
      - [Date and Time Functions](/functions-and-operators/date-and-time-functions.md)
      - [Bit Functions and Operators](/functions-and-operators/bit-functions-and-operators.md)
      - [Cast Functions and Operators](/functions-and-operators/cast-functions-and-operators.md)
      - [暗号化および圧縮関数](/functions-and-operators/encryption-and-compression-functions.md)
      - [ロッキング関数](/functions-and-operators/locking-functions.md)
      - [情報関数](/functions-and-operators/information-functions.md)
      - [JSON 関数](/functions-and-operators/json-functions.md)
      - [集約 (GROUP BY) 関数](/functions-and-operators/aggregate-group-by-functions.md)
      - [GROUP BY 修飾子](/functions-and-operators/group-by-modifier.md)
      - [ウィンドウ関数](/functions-and-operators/window-functions.md)
      - [その他の関数](/functions-and-operators/miscellaneous-functions.md)
      - [精度数学](/functions-and-operators/precision-math.md)
      - [集合演算](/functions-and-operators/set-operators.md)
      - [プッシュダウン用式の一覧](/functions-and-operators/expressions-pushed-down.md)
      - [TiDB 固有の関数](/functions-and-operators/tidb-functions.md)
      - [Oracle と TiDB の関数および構文の比較](/oracle-functions-to-tidb.md)
    - [クラスタ化インデックス](/clustered-indexes.md)
    - [制約](/constraints.md)
    - [生成列](/generated-columns.md)
    - [SQL モード](/sql-mode.md)
    - [テーブル属性](/table-attributes.md)
    - トランザクション
      - [概要](/transaction-overview.md)
      - [分離レベル](/transaction-isolation-levels.md)
      - [楽観的トランザクション](/optimistic-transaction.md)
      - [悲観的トランザクション](/pessimistic-transaction.md)
      - [非トランザクション DML 文](/non-transactional-dml.md)
    - [ビュー](/views.md)
    - [パーティション](/partitioned-table.md)
    - [一時テーブル](/temporary-tables.md)
    - [キャッシュ テーブル](/cached-tables.md)
    - [外部キー制約](/foreign-key.md)
    - 文字セットと照合順序
      - [概要](/character-set-and-collation.md)
      - [GBK](/character-set-gbk.md)
    - [SQL における配置ルール](/placement-rules-in-sql.md)
    - システム テーブル
      - [`mysql`](/mysql-schema.md)
      - INFORMATION_SCHEMA
        - [概要](/information-schema/information-schema.md)
        - [`ANALYZE_STATUS`](/information-schema/information-schema-analyze-status.md)
        - [`CHECK_CONSTRAINTS`](/information-schema/information-schema-check-constraints.md)
        - [`CLIENT_ERRORS_SUMMARY_BY_HOST`](/information-schema/client-errors-summary-by-host.md)
        - [`CLIENT_ERRORS_SUMMARY_BY_USER`](/information-schema/client-errors-summary-by-user.md)
        - [`CLIENT_ERRORS_SUMMARY_GLOBAL`](/information-schema/client-errors-summary-global.md)
        - [`CHARACTER_SETS`](/information-schema/information-schema-character-sets.md)
        - [`CLUSTER_CONFIG`](/information-schema/information-schema-cluster-config.md)
        - [`CLUSTER_HARDWARE`](/information-schema/information-schema-cluster-hardware.md)
        - [`CLUSTER_INFO`](/information-schema/information-schema-cluster-info.md)
        - [`CLUSTER_LOAD`](/information-schema/information-schema-cluster-load.md)
        - [`CLUSTER_LOG`](/information-schema/information-schema-cluster-log.md)
        - [`CLUSTER_SYSTEMINFO`](/information-schema/information-schema-cluster-systeminfo.md)
        - [`COLLATIONS`](/information-schema/information-schema-collations.md)
        - [`COLLATION_CHARACTER_SET_APPLICABILITY`](/information-schema/information-schema-collation-character-set-applicability.md)
        - [`COLUMNS`](/information-schema/information-schema-columns.md)
        - [`DATA_LOCK_WAITS`](/information-schema/information-schema-data-lock-waits.md)
        - [`DDL_JOBS`](/information-schema/information-schema-ddl-jobs.md)
        - [`DEADLOCKS`](/information-schema/information-schema-deadlocks.md)
        - [`ENGINES`](/information-schema/information-schema-engines.md)
        - [`INSPECTION_RESULT`](/information-schema/information-schema-inspection-result.md)
        - [`INSPECTION_RULES`](/information-schema/information-schema-inspection-rules.md)
        - [`INSPECTION_SUMMARY`](/information-schema/information-schema-inspection-summary.md)
        - [`KEY_COLUMN_USAGE`](/information-schema/information-schema-key-column-usage.md)
        - [`MEMORY_USAGE`](/information-schema/information-schema-memory-usage.md)
        - [`MEMORY_USAGE_OPS_HISTORY`](/information-schema/information-schema-memory-usage-ops-history.md)
        - [`METRICS_SUMMARY`](/information-schema/information-schema-metrics-summary.md)
        - [`METRICS_TABLES`](/information-schema/information-schema-metrics-tables.md)
        - [`PARTITIONS`](/information-schema/information-schema-partitions.md)
        - [`PLACEMENT_POLICIES`](/information-schema/information-schema-placement-policies.md)
        - [`PROCESSLIST`](/information-schema/information-schema-processlist.md)
        - [`REFERENTIAL_CONSTRAINTS`](/information-schema/information-schema-referential-constraints.md)
        - [`RESOURCE_GROUPS`](/information-schema/information-schema-resource-groups.md)
        - [`RUNAWAY_WATCHES`](/information-schema/information-schema-runaway-watches.md)
        - [`SCHEMATA`](/information-schema/information-schema-schemata.md)
        - [`SEQUENCES`](/information-schema/information-schema-sequences.md)
        - [`SESSION_VARIABLES`](/information-schema/information-schema-session-variables.md)
        - [`SLOW_QUERY`](/information-schema/information-schema-slow-query.md)
        - [`STATISTICS`](/information-schema/information-schema-statistics.md)
        - [`TABLES`](/information-schema/information-schema-tables.md)
        - [`TABLE_CONSTRAINTS`](/information-schema/information-schema-table-constraints.md)
        - [`TABLE_STORAGE_STATS`](/information-schema/information-schema-table-storage-stats.md)
        - [`TIDB_HOT_REGIONS`](/information-schema/information-schema-tidb-hot-regions.md)
        - [`TIDB_HOT_REGIONS_HISTORY`](/information-schema/information-schema-tidb-hot-regions-history.md)
        - [`TIDB_INDEXES`](/information-schema/information-schema-tidb-indexes.md)
        - [`TIDB_SERVERS_INFO`](/information-schema/information-schema-tidb-servers-info.md)
        - [`TIDB_TRX`](/information-schema/information-schema-tidb-trx.md)
        - [`TIFLASH_REPLICA`](/information-schema/information-schema-tiflash-replica.md)
        - [`TIFLASH_SEGMENTS`](/information-schema/information-schema-tiflash-segments.md)
        - [`TIFLASH_TABLES`](/information-schema/information-schema-tiflash-tables.md)
        - [`TIKV_REGION_PEERS`](/information-schema/information-schema-tikv-region-peers.md)
        - [`TIKV_REGION_STATUS`](/information-schema/information-schema-tikv-region-status.md)
        - [`TIKV_STORE_STATUS`](/information-schema/information-schema-tikv-store-status.md)
        - [`USER_ATTRIBUTES`](/information-schema/information-schema-user-attributes.md)
        - [`USER_PRIVILEGES`](/information-schema/information-schema-user-privileges.md)
        - [`VARIABLES_INFO`](/information-schema/information-schema-variables-info.md)
        - [`VIEWS`](/information-schema/information-schema-views.md)
      - [`METRICS_SCHEMA`](/metrics-schema.md)
      - PERFORMANCE_SCHEMA
        - [概要](/performance-schema/performance-schema.md)
        - [`SESSION_CONNECT_ATTRS`](/performance-schema/performance-schema-session-connect-attrs.md)
    - [メタデータロック](/metadata-lock.md)
  - UI
    - TiDB ダッシュボード
      - [概要](/dashboard/dashboard-intro.md)
      - メンテナンス
        - [デプロイ](/dashboard/dashboard-ops-deploy.md)
        - [リバースプロキシ](/dashboard/dashboard-ops-reverse-proxy.md)
        - [ユーザー管理](/dashboard/dashboard-user.md)
        - [セキュリティ](/dashboard/dashboard-ops-security.md)
      - [アクセス](/dashboard/dashboard-access.md)
      - [概要ページ](/dashboard/dashboard-overview.md)
      - [クラスタ情報ページ](/dashboard/dashboard-cluster-info.md)
      - [Top SQL ページ](/dashboard/top-sql.md)
      - [キービジュアライザページ](/dashboard/dashboard-key-visualizer.md)
      - [メトリクス関連グラフ](/dashboard/dashboard-metrics-relation.md)
      - SQL ステートメントの解析
        - [SQL ステートメントページ](/dashboard/dashboard-statement-list.md)
        - [SQL 詳細ページ](/dashboard/dashboard-statement-details.md)
      - [遅いクエリページ](/dashboard/dashboard-slow-query.md)
      - クラスタの診断
        - [クラスタ診断ページへのアクセス](/dashboard/dashboard-diagnostics-access.md)
        - [診断レポートの表示](/dashboard/dashboard-diagnostics-report.md)
        - [診断の使用](/dashboard/dashboard-diagnostics-usage.md)
      - [監視ページ](/dashboard/dashboard-monitoring.md)
      - [ログ検索ページ](/dashboard/dashboard-log-search.md)
      - [リソースマネージャページ](/dashboard/dashboard-resource-manager.md)
      - インスタンスのプロファイリング
        - [手動プロファイリング](/dashboard/dashboard-profiling.md)
        - [連続プロファイリング](/dashboard/continuous-profiling.md)
      - セッション管理および構成
        - [セッションの共有](/dashboard/dashboard-session-share.md)
        - [SSOの設定](/ja/dashboard/dashboard-session-sso.md)
      - [FAQ](/ja/dashboard/dashboard-faq.md)
  - [Telemetry（テレメトリ）](/ja/telemetry.md)
  - [エラーコード](/ja/error-codes.md)
  - [テーブルフィルタ](/ja/table-filter.md)
  - [トポロジラベルによるレプリカのスケジュール](/ja/schedule-replicas-by-topology-labels.md)
  - [外部ストレージサービスのURIフォーマット](/ja/external-storage-uri.md)
  - インターナルコンポーネント
    - [TiDBバックエンドタスク分散実行フレームワーク](/ja/tidb-distributed-execution-framework.md)
    - [TiDBグローバルソート](/ja/tidb-global-sort.md)
- FAQ
  - [FAQ概要](/ja/faq/faq-overview.md)
  - [TiDB FAQ](/ja/faq/tidb-faq.md)
  - [SQL FAQ](/ja/faq/sql-faq.md)
  - [デプロイメントFAQ](/ja/faq/deploy-and-maintain-faq.md)
  - [移行FAQ](/ja/faq/migration-tidb-faq.md)
  - [アップグレードFAQ](/ja/faq/upgrade-faq.md)
  - [モニタリングFAQ](/ja/faq/monitor-faq.md)
  - [クラスタ管理FAQ](/ja/faq/manage-cluster-faq.md)
  - [高可用性FAQ](/ja/faq/high-availability-faq.md)
  - [高信頼性FAQ](/ja/faq/high-reliability-faq.md)
  - [バックアップとリストアFAQ](/ja/faq/backup-and-restore-faq.md)
- リリースノート
  - [全てのリリース](/ja/releases/release-notes.md)
  - [リリースタイムライン](/ja/releases/release-timeline.md)
  - [TiDBのバージョニング](/ja/releases/versioning.md)
  - [TiDBインストールパッケージ](/ja/binary-package.md)
  - v7.4
    - [7.4.0-DMR](/ja/releases/release-7.4.0.md)
  - v7.3
    - [7.3.0-DMR](/ja/releases/release-7.3.0.md)
  - v7.2
    - [7.2.0-DMR](/ja/releases/release-7.2.0.md)
  - v7.1
    - [7.1.2](/ja/releases/release-7.1.2.md)
    - [7.1.1](/ja/releases/release-7.1.1.md)
    - [7.1.0](/ja/releases/release-7.1.0.md)
  - v7.0
    - [7.0.0-DMR](/ja/releases/release-7.0.0.md)
  - v6.6
    - [6.6.0-DMR](/ja/releases/release-6.6.0.md)
  - v6.5
    - [6.5.5](/ja/releases/release-6.5.5.md)
    - [6.5.4](/ja/releases/release-6.5.4.md)
    - [6.5.3](/ja/releases/release-6.5.3.md)
    - [6.5.2](/ja/releases/release-6.5.2.md)
    - [6.5.1](/ja/releases/release-6.5.1.md)
    - [6.5.0](/ja/releases/release-6.5.0.md)
  - v6.4
    - [6.4.0-DMR](/ja/releases/release-6.4.0.md)
  - v6.3
    - [6.3.0-DMR](/ja/releases/release-6.3.0.md)
  - v6.2
    - [6.2.0-DMR](/ja/releases/release-6.2.0.md)
  - v6.1
    - [6.1.7](/ja/releases/release-6.1.7.md)
    - [6.1.6](/ja/releases/release-6.1.6.md)
    - [6.1.5](/ja/releases/release-6.1.5.md)
    - [6.1.4](/ja/releases/release-6.1.4.md)
    - [6.1.3](/ja/releases/release-6.1.3.md)
    - [6.1.2](/ja/releases/release-6.1.2.md)
    - [6.1.1](/ja/releases/release-6.1.1.md)
    - [6.1.0](/ja/releases/release-6.1.0.md)
  - v6.0
    - [6.0.0-DMR](/ja/releases/release-6.0.0-dmr.md)
  - v5.4
    - [5.4.3](/ja/releases/release-5.4.3.md)
    - [5.4.2](/ja/releases/release-5.4.2.md)
    - [5.4.1](/ja/releases/release-5.4.1.md)
    - [5.4.0](/ja/releases/release-5.4.0.md)
  - v5.3
    - [5.3.4](/ja/releases/release-5.3.4.md)
    - [5.3.3](/ja/releases/release-5.3.3.md)
    - [5.3.2](/ja/releases/release-5.3.2.md)
    - [5.3.1](/ja/releases/release-5.3.1.md)
    - [5.3.0](/ja/releases/release-5.3.0.md)
  - v5.2
    - [5.2.4](/ja/releases/release-5.2.4.md)
    - [5.2.3](/ja/releases/release-5.2.3.md)
    - [5.2.2](/ja/releases/release-5.2.2.md)
    - [5.2.1](/ja/releases/release-5.2.1.md)
    - [5.2.0](/ja/releases/release-5.2.0.md)
  - v5.1
    - [5.1.5](/ja/releases/release-5.1.5.md)
    - [5.1.4](/ja/releases/release-5.1.4.md)
    - [5.1.3](/ja/releases/release-5.1.3.md)
    - [5.1.2](/ja/releases/release-5.1.2.md)
    - [5.1.1](/ja/releases/release-5.1.1.md)
    - [5.1.0](/ja/releases/release-5.1.0.md)
  - v5.0
    - [5.0.6](/ja/releases/release-5.0.6.md)
    - [5.0.5](/ja/releases/release-5.0.5.md)
    - [5.0.4](/ja/releases/release-5.0.4.md)
    - [5.0.3](/ja/releases/release-5.0.3.md)
    - [5.0.2](/ja/releases/release-5.0.2.md)
    - [5.0.1](/ja/releases/release-5.0.1.md)
    - [5.0 GA](/ja/releases/release-5.0.0.md)
    - [5.0.0-rc](/ja/releases/release-5.0.0-rc.md)
  - v4.0
    - [4.0.16](/ja/releases/release-4.0.16.md)
    - [4.0.15](/ja/releases/release-4.0.15.md)
    - [4.0.14](/ja/releases/release-4.0.14.md)
    - [4.0.13](/ja/releases/release-4.0.13.md)
    - [4.0.12](/ja/releases/release-4.0.12.md)
    - [4.0.11](/ja/releases/release-4.0.11.md)
    - [4.0.10](/ja/releases/release-4.0.10.md)
    - [4.0.9](/ja/releases/release-4.0.9.md)
    - [4.0.8](/ja/releases/release-4.0.8.md)
    - [4.0.7](/ja/releases/release-4.0.7.md)
    - [4.0.6](/ja/releases/release-4.0.6.md)
    - [4.0.5](/ja/releases/release-4.0.5.md)
    - [4.0.4](/ja/releases/release-4.0.4.md)
    - [4.0.3](/ja/releases/release-4.0.3.md)
    - [4.0.2](/ja/releases/release-4.0.2.md)
    - [4.0.1](/ja/releases/release-4.0.1.md)
    - [4.0 GA](/ja/releases/release-4.0-ga.md)
- [4.0.0-rc.2](/releases/release-4.0.0-rc.2.md)
    - [4.0.0-rc.1](/releases/release-4.0.0-rc.1.md)
    - [4.0.0-rc](/releases/release-4.0.0-rc.md)
    - [4.0.0-beta.2](/releases/release-4.0.0-beta.2.md)
    - [4.0.0-beta.1](/releases/release-4.0.0-beta.1.md)
    - [4.0.0-beta](/releases/release-4.0.0-beta.md)
  - v3.1
    - [3.1.2](/releases/release-3.1.2.md)
    - [3.1.1](/releases/release-3.1.1.md)
    - [3.1.0 GA](/releases/release-3.1.0-ga.md)
    - [3.1.0-rc](/releases/release-3.1.0-rc.md)
    - [3.1.0-beta.2](/releases/release-3.1.0-beta.2.md)
    - [3.1.0-beta.1](/releases/release-3.1.0-beta.1.md)
    - [3.1.0-beta](/releases/release-3.1.0-beta.md)
  - v3.0
    - [3.0.20](/releases/release-3.0.20.md)
    - [3.0.19](/releases/release-3.0.19.md)
    - [3.0.18](/releases/release-3.0.18.md)
    - [3.0.17](/releases/release-3.0.17.md)
    - [3.0.16](/releases/release-3.0.16.md)
    - [3.0.15](/releases/release-3.0.15.md)
    - [3.0.14](/releases/release-3.0.14.md)
    - [3.0.13](/releases/release-3.0.13.md)
    - [3.0.12](/releases/release-3.0.12.md)
    - [3.0.11](/releases/release-3.0.11.md)
    - [3.0.10](/releases/release-3.0.10.md)
    - [3.0.9](/releases/release-3.0.9.md)
    - [3.0.8](/releases/release-3.0.8.md)
    - [3.0.7](/releases/release-3.0.7.md)
    - [3.0.6](/releases/release-3.0.6.md)
    - [3.0.5](/releases/release-3.0.5.md)
    - [3.0.4](/releases/release-3.0.4.md)
    - [3.0.3](/releases/release-3.0.3.md)
    - [3.0.2](/releases/release-3.0.2.md)
    - [3.0.1](/releases/release-3.0.1.md)
    - [3.0 GA](/releases/release-3.0-ga.md)
    - [3.0.0-rc.3](/releases/release-3.0.0-rc.3.md)
    - [3.0.0-rc.2](/releases/release-3.0.0-rc.2.md)
    - [3.0.0-rc.1](/releases/release-3.0.0-rc.1.md)
    - [3.0.0-beta.1](/releases/release-3.0.0-beta.1.md)
    - [3.0.0-beta](/releases/release-3.0-beta.md)
  - v2.1
    - [2.1.19](/releases/release-2.1.19.md)
    - [2.1.18](/releases/release-2.1.18.md)
    - [2.1.17](/releases/release-2.1.17.md)
    - [2.1.16](/releases/release-2.1.16.md)
    - [2.1.15](/releases/release-2.1.15.md)
    - [2.1.14](/releases/release-2.1.14.md)
    - [2.1.13](/releases/release-2.1.13.md)
    - [2.1.12](/releases/release-2.1.12.md)
    - [2.1.11](/releases/release-2.1.11.md)
    - [2.1.10](/releases/release-2.1.10.md)
    - [2.1.9](/releases/release-2.1.9.md)
    - [2.1.8](/releases/release-2.1.8.md)
    - [2.1.7](/releases/release-2.1.7.md)
    - [2.1.6](/releases/release-2.1.6.md)
    - [2.1.5](/releases/release-2.1.5.md)
    - [2.1.4](/releases/release-2.1.4.md)
    - [2.1.3](/releases/release-2.1.3.md)
    - [2.1.2](/releases/release-2.1.2.md)
    - [2.1.1](/releases/release-2.1.1.md)
    - [2.1 GA](/releases/release-2.1-ga.md)
    - [2.1 RC5](/releases/release-2.1-rc.5.md)
    - [2.1 RC4](/releases/release-2.1-rc.4.md)
    - [2.1 RC3](/releases/release-2.1-rc.3.md)
    - [2.1 RC2](/releases/release-2.1-rc.2.md)
    - [2.1 RC1](/releases/release-2.1-rc.1.md)
    - [2.1 Beta](/releases/release-2.1-beta.md)
  - v2.0
    - [2.0.11](/releases/release-2.0.11.md)
    - [2.0.10](/releases/release-2.0.10.md)
    - [2.0.9](/releases/release-2.0.9.md)
    - [2.0.8](/releases/release-2.0.8.md)
    - [2.0.7](/releases/release-2.0.7.md)
    - [2.0.6](/releases/release-2.0.6.md)
    - [2.0.5](/releases/release-2.0.5.md)
    - [2.0.4](/releases/release-2.0.4.md)
    - [2.0.3](/releases/release-2.0.3.md)
    - [2.0.2](/releases/release-2.0.2.md)
    - [2.0.1](/releases/release-2.0.1.md)
    - [2.0](/releases/release-2.0-ga.md)
    - [2.0 RC5](/releases/release-2.0-rc.5.md)
    - [2.0 RC4](/releases/release-2.0-rc.4.md)
    - [2.0 RC3](/releases/release-2.0-rc.3.md)
    - [2.0 RC1](/releases/release-2.0-rc.1.md)
    - [1.1 Beta](/releases/release-1.1-beta.md)
    - [1.1 Alpha](/releases/release-1.1-alpha.md)
  - v1.0
    - [1.0.8](/releases/release-1.0.8.md)
    - [1.0.7](/releases/release-1.0.7.md)
    - [1.0.6](/releases/release-1.0.6.md)
    - [1.0.5](/releases/release-1.0.5.md)
    - [1.0.4](/releases/release-1.0.4.md)
    - [1.0.3](/releases/release-1.0.3.md)
    - [1.0.2](/releases/release-1.0.2.md)
    - [1.0.1](/releases/release-1.0.1.md)
    - [1.0](/releases/release-1.0-ga.md)
    - [Pre-GA](/releases/release-pre-ga.md)
    - [RC4](/releases/release-rc.4.md)
    - [RC3](/releases/release-rc.3.md)
    - [RC2](/releases/release-rc.2.md)
    - [RC1](/releases/release-rc.1.md)
- [Glossary](/glossary.md)