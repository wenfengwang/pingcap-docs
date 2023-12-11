---
title: TiDB データ移行（DM）のベストプラクティス
summary: TiDB データ移行（DM）を使用する際のベストプラクティスについて学びます。
---

# TiDB データ移行（DM）のベストプラクティス

[PingCAP](https://github.com/pingcap/tiflow/tree/master/dm) が開発したデータ移行ツールである[TiDBデータ移行（DM）](https://github.com/pingcap/tiflow/tree/master/dm)は、MySQL 互換データベース（MySQL、Percona MySQL、MariaDB、Amazon RDS for MySQL、およびAmazon Aurora）からTiDBへの完全および増分データ移行をサポートしています。

次のシナリオでDMを使用できます。

- 単一のMySQL互換データベースインスタンスからTiDBへの完全および増分データ移行
- 小規模データセット（1 TiB未満）のMySQLシャードをTiDBに移行および統合
- ビジネスデータの中間プラットフォームやビジネスデータのリアルタイム集計などのデータハブシナリオで、DMをデータ移行のためのミドルウェアとして使用

このドキュメントでは、DMのエレガントかつ効率的な使用方法と、DMの使用時に一般的なミスを避ける方法について紹介します。

## パフォーマンスの制約

| パフォーマンス項目 | 制約 |
| ----------------- | :--------: |
| 最大作業ノード数 | 1000 |
| 最大タスク数 | 600 |
| 最大QPS | 30k QPS/ワーカー |
| 最大バイナログスループット | 20 MB/s/ワーカー |
| タスクごとのテーブル数制限 | 制限なし |

- DMは同時に1000個の作業ノードを管理でき、最大タスク数は600です。作業ノードの高可用性を確保するためには、一部の作業ノードを待機ノードとして保留する必要があります。おすすめの待機ノードの数は、実施中の移行タスクの作業ノード数の20%から50%です。
- １つの作業ノードは、理論上は最大30K QPS/ワーカーのレプリケーションQPSをサポートできます。これは異なるスキーマやワークロードによって異なります。アップストリームのバイナログの処理能力は最大20 MB/s/ワーカーです。
- DMをデータレプリケーションのミドルウェアとして長期間使用する場合は、DMコンポーネントの展開アーキテクチャを慎重に設計する必要があります。詳細については、[DM-master と DM-worker のデプロイ](#deploy-dm-master-and-dm-worker)を参照してください。

## データ移行前

データ移行前に、全体の解決策の設計が重要です。次のセクションでは、ビジネス面や実装面からのベストプラクティスとシナリオについて説明します。

### ビジネス面のベストプラクティス

複数のノードに均等に作業負荷を分散させるために、分散データベースの設計は従来のデータベースとは異なります。解決策は、移行コストを低く抑えつつ、移行後のロジックの正確さを確保する必要があります。次のセクションでは、データ移行前のベストプラクティスについて説明します。

#### スキーマ設計におけるAUTO_INCREMENTのビジネス影響

TiDBの`AUTO_INCREMENT`はMySQLの`AUTO_INCREMENT`と互換性があります。ただし、分散データベースとして、通常、TiDBには複数のコンピューティングノード（クライアントエンドのエントリ）があります。アプリケーションデータが書き込まれる際、作業負荷は均等に分散されます。これにより、テーブルに`AUTO_INCREMENT`列がある場合、その列の自動増加IDは連続しない可能性があります。詳細については、[AUTO_INCREMENT](/auto-increment.md#implementation-principles)をご覧ください。

ビジネスが自動増加IDに強く依存している場合は、[SEQUENCE関数](/sql-statements/sql-statement-create-sequence.md#sequence-function)の使用を検討してください。

#### クラスターインデックスの使用

テーブルを作成する際、主キーをクラスターインデックスまたはノンクラスターインデックスとして宣言できます。次のセクションでは、それぞれの選択肢の利点と欠点について説明します。

- クラスターインデックス

    [クラスターインデックス](/clustered-indexes.md)は、主キーをデータ保存のハンドルID（行ID）として使用します。主キーを使用したクエリは、テーブルの参照を回避でき、クエリパフォーマンスが向上します。ただし、テーブルが書き込み集中型で、主キーが [`AUTO_INCREMENT`](/auto-increment.md) を使用する場合、[書き込みホットスポットの問題](/best-practices/high-concurrency-best-practices.md#highly-concurrent-write-intensive-scenario)が発生し、クラスターの性能と単一のストレージノードのパフォーマンスボトルネックが起こる可能性があります。

- ノンクラスターインデックス + `shard row id bit`

    ノンクラスターインデックスと`shard row id bit`を使用すると、`AUTO_INCREMENT`を使用する際の書き込みホットスポット問題を回避できます。ただし、このシナリオではテーブル参照がクエリパフォーマンスに影響を与える可能性があります。

- クラスターインデックス + 外部分散IDジェネレーター

    クラスターインデックスを使用し、IDが連続するようにしたい場合は、SnowflakeアルゴリズムやLeafなどの外部分散IDジェネレーターを使用することを検討してください。アプリケーションプログラムがシーケンスIDを生成するため、ある程度連続性のあるIDを保証できます。また、クラスターインデックスを使用する利点を保持できます。ただし、アプリケーションをカスタマイズする必要があります。

- クラスターインデックス + `AUTO_RANDOM`

    この解決策は、クラスターインデックスを使用する利点を保ち、書き込みホットスポットの問題を回避できます。カスタマイズの必要が少なく、TiDBを書き込みデータベースとして切り替える際にスキーマ属性を変更できます。次に、データをソートするためにID列を使用する場合は、 [`AUTO_RANDOM`](/auto-random.md) ID列を使用し、左に5ビットシフトさせることで、クエリデータの順序を保証できます。例えば：

    ```sql
    CREATE TABLE t (a bigint PRIMARY KEY AUTO_RANDOM, b varchar(255));
    Select a, a<<5 ,b from t order by a <<5 desc
    ```

次の表は、それぞれの解決策の利点と欠点をまとめたものです。

| シナリオ | 推奨される解決策 | 利点 | 欠点 |
| :--- | :--- | :--- | :--- |
| <li>TiDBが主要で書き込み集中型データベースとして機能します。</li><li>ビジネスロジックが主キーIDの連続性に強く依存しています。</li> | ノンクラスターインデックスのテーブルを作成し、`SHARD_ROW_ID_BIT`を設定し、主キー列に`SEQUENCE`を使用します。 | データ書き込みのスループット容量を下げて、データ書き込みの連続性を保証します。 | <li>データ書き込みの性能が低下します。</li><li>主キークエリのパフォーマンスが低下します。</li> |
| <li>TiDBが主要で書き込み集中型データベースとして機能します。</li><li>ビジネスロジックが主キーIDの増加に強く依存しています。</li> | ノンクラスターインデックスのテーブルを作成し、`SHARD_ROW_ID_BIT`を設定し、アプリケーションIDジェネレーターを使用して主キーIDを生成します。 | データ書き込みのスループット性能を保証し、ビジネスデータの増加を保証しますが、連続性は保証されません。 | <li>アプリケーションのカスタマイズが必要です。</li><li>外部IDジェネレーターは時計の精度に強く依存し、障害を引き起こす可能性があります。</li> |
| <li>TiDBが主要で書き込み集中型データベースとして機能します。</li><li>ビジネスロジックが主キーIDの連続性に強く依存していません。</li> | クラスターインデックスのテーブルを作成し、主キー列に`AUTO_RANDOM`を設定します。 | <li>データ書き込みのスループット性能が制限されません。 </li><li>主キークエリのパフォーマンスが優れています。</li> | <li>主キーIDはランダムです。</li><li>書き込みスループット能力が制限されます。</li><li>ビジネスデータは挿入時刻列を使用することがおすすめです。</li><li>データを並べ替える際、主キーIDを使用する必要がある場合は、5ビット左にシフトしてクエリすることで、データの増加を保証できます。</li> |
| TiDBが読み取り専用データベースとして機能します。 | ノンクラスターインデックスのテーブルを作成し、`SHARD_ROW_ID_BIT`を設定し、主キー列をデータソースと一貫させます。 | <li>データ書き込みのホットスポットを回避できます。</li><li>カスタマイズコストが低いです。</li> | 主キークエリのパフォーマンスに影響を与えます。 |

### MySQLシャードのキーポイント

#### 分割と統合

DMを使用して、[小さなデータセットのMySQLシャードをTiDBにマイグレーションおよび統合](/migrate-small-mysql-shards-to-tidb.md)することをお勧めします。

データ統合以外にも、典型的なシナリオにデータアーカイブがあります。データは常に書き込まれ続けており、時間の経過とともに、大量のデータが徐々にホットデータからウォームデータ、さらにはコールドデータへと変化していきます。幸いにも、TiDBではさまざまな[配置ルール](/configure-placement-rules.md)をデータに設定できます。ルールの最小粒度は[パーティション](/partitioned-table.md)です。

したがって、書き込み集中型のシナリオでは、将来的にデータをアーカイブし、ホットデータとコールデータを別々のメディアに格納する必要があるかどうかを最初から評価する必要があります（TiDBはまだテーブル再構築操作をサポートしていません）。データをアーカイブする必要がある場合、マイグレーション前にパーティション化ルールを設定することで、将来的にテーブルを再作成してデータをインポートする手間を省くことができます。

#### 悲観モードと楽観モード

DMはデフォルトで悲観モードを使用しています。MySQLシャードをマイグレーションおよび統合するシナリオでは、上流シャードスキーマの変更が下流データベースへのDML書き込みをブロックすることがあります。すべてのスキーマが変更され、同じ構造になるのを待ってから、マイグレーションを続行する必要があります。
- 上流のスキーマ変更が長時間かかる場合、上流のBinlogのクリーンアップが発生する可能性があります。この問題を回避するために、リレーログを有効にできます。詳細については、[リレーログを使用する](#use-the-relay-log) を参照してください。

- 上流のスキーマ変更によるデータ書き込みのブロックを避けたい場合は、楽観的モードを使用することを検討してください。この場合、DMは上流のシャードスキーマの変更を検出しても、データ移行をブロックせずに継続します。ただし、上流と下流で互換性のない形式が検出されると、移行タスクは停止します。この問題は手動で解決する必要があります。

以下の表は、楽観的モードと悲観的モードの利点と欠点をまとめたものです。

| シナリオ | 利点 | 欠点 |
| :--- | :--- | :--- |
| 悲観的モード (デフォルト) | 下流に移行したデータに問題がないことを確実にできます。 | シャードが大量にある場合、移行タスクが長時間ブロックされることがあります。上流のBinlogがクリーンアップされている場合は、この問題が発生する可能性があります。この問題を回避するために、リレーログを有効にできます。詳細については、[リレーログを使用する](#use-the-relay-log) を参照してください。 |
| 楽観的モード| 上流のスキーマ変更がデータ移行の遅延を引き起こしません。 | このモードでは、スキーマの変更が互換性のあるものであることを確認する必要があります（増分列にデフォルト値があるかどうかをチェック）。不整合なデータが見落とされる可能性があります。詳細については、[楽観的モードでのシャードテーブルからのデータのマージと移行](/dm/feature-shard-merge-optimistic.md#restrictions) を参照してください。|

### その他の制約と影響

#### 上流と下流のデータ型

TiDBはほとんどのMySQLデータ型をサポートしていますが、一部の特別な型（`SPATIAL`など）はまだサポートされていません。データ型の互換性については、[データ型](/data-type-overview.md)を参照してください。

#### 文字セットと照合順序

TiDB v6.0.0以降、照合順序の新しいフレームワークがデフォルトで使用されています。以前のバージョンでは、`utf8_general_ci`、`utf8mb4_general_ci`、`utf8_unicode_ci`、`utf8mb4_unicode_ci`、`gbk_chinese_ci`、`gbk_bin` をサポートするために、`new_collations_enabled_on_first_bootstrap` の値を `true` に設定してクラスタを作成する必要があります。詳細については、[照合順序の新しいフレームワーク](/character-set-and-collation.md#new-framework-for-collations)を参照してください。

TiDBのデフォルトの文字セットは `utf8mb4` です。上流データベースと下流データベース、またはアプリケーションで `utf8mb4` を使用することをお勧めします。上流のデータベースで文字セットや照合順序が明示的に指定されている場合、TiDBがそれをサポートしているかどうかを確認する必要があります。

TiDB v6.0.0以降、GBKがサポートされています。詳細については、以下のドキュメントを参照してください。

- [文字セットと照合順序](/character-set-and-collation.md)
- [GBKの互換性](/character-set-gbk.md#mysql-compatibility)

### デプロイのベストプラクティス

#### DM-masterとDM-workerのデプロイ

DMはDM-masterノードとDM-workerノードで構成されています。

- DM-masterは移行タスクのメタデータを管理し、DM-workerノードをスケジュールします。これは全体のDMプラットフォームの中核です。そのため、DMプラットフォームの高い可用性を確保するために、DM-masterをクラスタとしてデプロイできます。

- DM-workerは上流と下流の移行タスクを実行します。DM-workerノードは状態を持ちません。最大で1000台のDM-workerノードをデプロイできます。DMを使用する際には、高い可用性を確保するためにいくつかのアイドル状態のDM-workerを予約することを推奨します。

#### 移行タスクの計画

MySQLシャードの移行とマージを行う際には、上流のシャードの種類に応じて移行タスクを分割することができます。たとえば、「usertable_1〜50」と「Logtable_1〜50」という2つのシャードの場合、2つの移行タスクを作成できます。これにより、移行タスクのテンプレートを簡素化し、データ移行の中断の影響を効果的に制御できます。

大規模なデータセットの移行の場合、以下の提案に従って移行タスクを分割できます:

- 上流で複数のデータベースを移行する必要がある場合、データベースの数に応じて移行タスクを分割できます。

- 上流での書き込み圧力に応じて移行タスクを分割します。つまり、上流で頻繁なDML操作を行うテーブルを別の移行タスクに分割し、頻繁でないテーブルを別の移行タスクに使用します。この方法は、特に上流のテーブルに大量のログが書き込まれている場合や、それがビジネス全体に影響を与えない場合に、移行の進行を速めることができます。ただし、このテーブルが全体のビジネスに影響を与えない場合は、この方法でも問題ありません。

移行タスクを分割することは、最終的なデータの整合性を保証することしかできません。さまざまな理由で、リアルタイムの整合性が大幅に逸脱する可能性があります。

以下の表は、異なるシナリオでのDM-masterとDM-workerの推奨デプロイプランについて説明しています。

| シナリオ | DM-masterのデプロイ | DM-workerのデプロイ |
| :--- | :--- | :--- |
| <li>データセットが小さい（1 TiB未満）</li><li>ワンタイムデータ移行</li> | DM-masterノード1台のデプロイ | 上流データソースの数に応じて1〜N台のDM-workerノードをデプロイします。通常、DM-workerノード1台を推奨します。 |
| <li>データセットが大きい（1 TiB以上）、およびMySQLシャードの移行およびマージ</li><li>ワンタイムデータ移行</li> | 長時間のデータ移行中にDMクラスタの可用性を確保するために、DM-masterノード3台をデプロイすることを推奨します。 | データソースまたは移行タスクの数に応じてDM-workerノードをデプロイします。稼働中のDM-workerノードの他に、アイドル状態のDM-workerノードを1〜3台デプロイすることを推奨します。 |
| 長期的なデータレプリケーション | DM-masterノード3台をデプロイする必要があります。DM-masterノードをクラウド上にデプロイする場合は、異なる可用性ゾーン（AZ）にデプロイするようにしてください。 | データソースまたは移行タスクの数に応じてDM-workerノードをデプロイします。実際に必要なDM-workerノードの数の1.5〜2倍をデプロイする必要があります。 |

#### 上流データソースを選択し構成する

DMは、完全なデータのバックアップを行い、並列論理バックアップ方式を使用します。MySQLのバックアップ時には、グローバルリードロック [`FLUSH TABLES WITH READ LOCK`](https://dev.mysql.com/doc/refman/8.0/en/flush.html#flush-tables-with-read-lock) を追加します。上流データベースのDMLおよびDDL操作は短時間ブロックされます。したがって、上流でバックアップデータを実行して、データソースのGTID機能を有効にすることを強く推奨します（`enable-gtid: true`）。これにより、上流の影響を避け、増分移行中のレイテンシを減少させ、上流のマスターノードに切り替えることができます。上流MySQLデータソースの切り替えの手順については、[上流MySQLインスタンス間でのDM-worker接続の切り替え](/dm/usage-scenario-master-slave-switch.md#switch-dm-worker-connection-via-virtual-ip)を参照してください。

以下に留意してください：

- 上流データベースのマスターノードでのみ完全データバックアップを実行できます。

  このシナリオでは、構成ファイルの `consistency` パラメータを `none` に設定すること（`mydumpers.global.extra-args: "--consistency none"`）で、上流のマスターノードにグローバルリードロックを追加することを回避することができます。ただし、これにより、完全バックアップのデータ整合性に影響が出る可能性があり、上流と下流でデータの整合性が損なわれる可能性があります。

- バックアップスナップショットを使用してフルデータ移行を実行する（AWS上のMySQL RDSおよびAurora RDSの移行にのみ適用）

  移行対象のデータベースがAWSのMySQL RDSまたはAurora RDSである場合、Amazon S3にあるバックアップデータを直接TiDBに移行するためにRDSスナップショットを使用できます。データの整合性を確保するために、詳細については[Migrate Data from Amazon Aurora to TiDB](/migrate-aurora-to-tidb.md)を参照してください。

### 設定の詳細

#### 大文字と小文字

TiDBのスキーマ名はデフォルトで大文字と小文字を区別しない（`lower_case_table_names:2`）です。ただし、ほとんどの上流MySQLデータベースはデフォルトで大文字小文字を区別するLinuxシステムを使用しています。この場合、上流からスキーマを正しく移行するためには、DMのタスク構成ファイルで `case-sensitive: true` を設定する必要があります。

例外的な場合として、上流に大文字の `Table` や小文字の `table` が含まれるデータベースがあるとします。その場合、次のエラーが発生します。

`ERROR 1050 (42S01): Table '{tablename}' already exists`

#### フィルタルール

データソースの構成を開始するとすぐにフィルタルールを設定することができます。詳細については、[データ移行タスクの設定ガイド](/dm/dm-task-configuration-guide.md)を参照してください。フィルタルールを設定する利点は次の通りです。

- 下流が処理する必要のあるBinlogイベントの数を減らし、移行効率を向上させます。
- 不要なリレーログのストレージを減らし、ディスクスペースを節約します。

> **注意:**
>
> MySQLシャードを移行しマージする場合、データソースでフィルタルールを設定している場合は、データソースと移行タスクとの間でルールが一致していることを確認する必要があります。一致しない場合、移行タスクが長時間にわたって増分データを受信できない問題が発生する可能性があります。

#### リレーログを使用する
```
In the MySQL master/standby mechanism, the standby node saves a copy of relay logs to ensure the reliability and efficiency of asynchronous replication. DM also supports saving a copy of relay logs on DM-worker. You can configure information such as the storage location and expiration time. This feature applies to the following scenarios:

- During full and incremental data migration, if the amount of full data is large, the entire process takes more time than the time for the upstream binlogs to be archived. It causes the incremental replication task to fail to start normally. If you enable the relay log, DM-worker will start receiving relay logs when the full migration is started. This avoids the failure of the incremental task.

- When you use DM to perform long-time data replication, sometimes the migration task is blocked for a long time due to various reasons. If you enable the relay log, you can effectively deal with the problem of upstream binlogs being recycled due to the blocking of the migration task.

There are some restrictions on using the relay log. DM supports high availability. When a DM-worker fails, it will try to promote an idle DM-worker instance to a working instance. If the upstream binlogs do not contain the necessary migration logs, it may cause interruption. You need to intervene manually to copy the relay log to the new DM-worker node as soon as possible, and modify the corresponding relay meta file. For details, see [Troubleshooting](/dm/dm-error-handling.md#the-relay-unit-throws-error-event-from--in--diff-from-passed-in-event--or-a-migration-task-is-interrupted-with-failing-to-get-or-parse-binlog-errors-like-get-binlog-error-error-1236-hy000-and-binlog-checksum-mismatch-data-may-be-corrupted-returned).

#### Use PT-osc/GH-ost in upstream

In daily MySQL operation and maintenance, usually you use tools such as PT-osc/GH-ost to change the schema online to minimize impact on the business. However, the whole process will be logged to MySQL Binlog. Migrating such data to TiDB downstream will result in a lot of unnecessary write operations, which is neither efficient nor economical.

To resolve this issue, DM supports third-party data tools such as PT-osc and GH-ost when you configure the migration task. When you use such tools, DM does not migrate redundant data and ensure data consistency. For details, see [Migrate from Databases that Use GH-ost/PT-osc](/dm/feature-online-ddl.md).

## Best practices during migration

This section introduces how to troubleshoot problems you might encounter during migration.

### Inconsistent schemas in upstream and downstream

Common errors include:

- `messages: Column count doesn't match value count: 3 (columns) vs 2 (values)`
- `Schema/Column doesn't match`

Usually such issues are caused by changed or added indexes in the downstream TiDB, or there are more columns in the downstream. When such errors occur, check whether the upstream and downstream schemas are inconsistent.

To resolve such issues, update the schema information cached in DM to be consistent with the downstream TiDB schema. For details, see [Manage Table Schemas of Tables to be Migrated](/dm/dm-manage-schema.md).

If the downstream has more columns, see [Migrate Data to a Downstream TiDB Table with More Columns](/migrate-with-more-columns-downstream.md).

### Interrupted migration task due to failed DDL

DM supports skipping or replacing DDL statements that cause a migration task to interrupt. For details, see [Handle Failed DDL Statements](/dm/handle-failed-ddl-statements.md#usage-examples).

## Data validation after data migration

It is recommended that you validate the consistency of data after data migration. TiDB provides [sync-diff-inspector](/sync-diff-inspector/sync-diff-inspector-overview.md) to help you complete the data validation.

Now sync-diff-inspector can automatically manage the table list to be checked for data consistency through DM tasks. Compared with the previous manual configuration, it is more efficient. For details, see [Data Check in the DM Replication Scenario](/sync-diff-inspector/dm-diff.md).

Since DM v6.2.0, DM supports continuous data validation for incremental replication. For details, see [Continuous Data Validation in DM](/dm/dm-continuous-data-validation.md).

## Long-term data replication

If you use DM to perform a long-term data replication task, it is necessary to back up the metadata. On the one hand, it ensures the ability to rebuild the migration cluster. On the other hand, it can implement the version control of the migration task. For details, see [Export and Import Data Sources and Task Configuration of Clusters](/dm/dm-export-import-config.md).
```