---
title: TiDBバックエンドタスク分散実行フレームワーク
summary: TiDBバックエンドタスク分散実行フレームワークのユースケース、制限、使用方法、および実装原則の学習
---

# TiDBバックエンドタスク分散実行フレームワーク

<CustomContent platform="tidb-cloud">

> **注意:**
>
> 現在、この機能はTiDB専用クラスターにのみ適用されます。TiDBサーバーレスクラスターでは使用できません。

</CustomContent>

TiDBは優れたスケーラビリティと弾力性を持つ計算ストレージ分離アーキテクチャを採用しています。バージョンv7.1.0からTiDBはバックエンドタスク分散実行フレームワークを導入し、分散アーキテクチャのリソース利点をさらに活用します。このフレームワークの目標は、すべてのバックエンドタスクの統一されたスケジューリングおよび分散実行を実装し、すべてのバックエンドタスクに対する統一されたリソース管理機能を提供することで、リソース使用のユーザーの期待により適したものにすることです。

この文書では、TiDBバックエンドタスク分散実行フレームワークのユースケース、制限、使用方法、および実装原則について説明します。

> **注意:**
>
> このフレームワークはSQLクエリの分散実行をサポートしていません。

## ユースケースと制限

<CustomContent platform="tidb">

データベース管理システムでは、コアのトランザクション処理（TP）および分析処理（AP）のワークロードに加えて、DDL操作、IMPORT INTO、TTL、Analyze、およびBackup/Restoreなどの重要なタスクがあります。これらは**バックエンドタスク**と呼ばれ、通常はデータベースオブジェクト（テーブル）内の大量のデータを処理する必要があるため、通常は次の特性を持ちます：

</CustomContent>

<CustomContent platform="tidb-cloud">

データベース管理システムでは、コアのトランザクション処理（TP）および分析処理（AP）のワークロードに加えて、DDL操作、TTL、Analyze、およびBackup/Restoreなどの重要なタスクがあります。これらは**バックエンドタスク**と呼ばれ、通常はデータベースオブジェクト（テーブル）内の大量のデータを処理する必要があるため、通常は次の特性を持ちます：

</CustomContent>

- スキーマまたはデータベースオブジェクト（テーブル）のすべてのデータを処理する必要があります。
- 定期的に実行する必要がある場合がありますが、頻度は低いです。
- リソースが適切に制御されないと、TPおよびAPタスクに影響を与え、データベースサービスの品質が低下するおそれがあります。

TiDBバックエンドタスク分散実行フレームワークを有効にすると、上記の問題を解決し、以下の3つの利点があります：

- このフレームワークは高いスケーラビリティ、高い可用性、高いパフォーマンスに統一された能力を提供します。
- このフレームワークはバックエンドタスクの分散実行をサポートし、TiDBクラスター全体の利用可能なコンピューティングリソースを柔軟にスケジューリングし、TiDBクラスターのコンピューティングリソースをより効果的に利用します。
- このフレームワークは、バックエンドタスクの全体と個々のリソース使用に統一された管理機能を提供します。

現在、TiDB Self-Hostedでは、TiDBバックエンドタスク分散実行フレームワークは`ADD INDEX`および`IMPORT INTO`ステートメントの分散実行をサポートしています。TiDB Cloudでは`IMPORT INTO`ステートメントは適用されません。

- `ADD INDEX`は、インデックスを作成するためのDDLステートメントです。例：

    ```sql
    ALTER TABLE t1 ADD INDEX idx1(c1);
    CREATE INDEX idx1 ON table t1(c1);
    ```

- `IMPORT INTO`は、`CSV`、`SQL`、および`PARQUET`などの形式のデータを空のテーブルにインポートするために使用されます。詳細は[`IMPORT INTO`](https://docs.pingcap.com/tidb/v7.2/sql-statement-import-into)を参照してください。

## 前提条件

分散フレームワークを使用する前に、[Fast Online DDL](/system-variables.md#tidb_ddl_enable_fast_reorg-new-in-v630)モードを有効にする必要があります。

<CustomContent platform="tidb">

1. 以下のFast Online DDLに関連するシステム変数を調整します：

    * [`tidb_ddl_enable_fast_reorg`](/system-variables.md#tidb_ddl_enable_fast_reorg-new-in-v630)：Fast Online DDLモードを有効にします。デフォルトでTiDB v6.5.0以降で有効になっています。
    * [`tidb_ddl_disk_quota`](/system-variables.md#tidb_ddl_disk_quota-new-in-v630)：Fast Online DDLモードで使用できるローカルディスクの最大クォータを制御します。

2. 以下のFast Online DDLに関連する設定項目を調整します：

    * [`temp-dir`](/tidb-configuration-file.md#temp-dir-new-in-v630)：Fast Online DDLモードで使用できるローカルディスクパスを指定します。

> **注意:**
>
> TiDBをv6.5.0以降にアップグレードする前に、TiDBの[`temp-dir`](/tidb-configuration-file.md#temp-dir-new-in-v630)パスが正しくSSDディスクにマウントされているか確認することをお勧めします。実行中のシステムユーザーがこのディレクトリに対して読み書き権限を持っていることを確認してください。そうでない場合、DDL操作に予測できない問題が発生する可能性があります。このパスはTiDBの設定項目であり、TiDBの再起動後に有効になります。したがって、アップグレード前にこの設定項目を事前に設定することで、別の再起動を回避できます。

</CustomContent>

<CustomContent platform="tidb-cloud">

Fast Online DDLに関連する次のシステム変数を調整します：

* [`tidb_ddl_enable_fast_reorg`](/system-variables.md#tidb_ddl_enable_fast_reorg-new-in-v630)：Fast Online DDLモードを有効にします。デフォルトでTiDB v6.5.0以降で有効になっています。
* [`tidb_ddl_disk_quota`](/system-variables.md#tidb_ddl_disk_quota-new-in-v630)：Fast Online DDLモードで使用できるローカルディスクの最大クォータを制御します。

</CustomContent>

## 使用方法

1. 分散フレームワークを有効にするには、[`tidb_enable_dist_task`](/system-variables.md#tidb_enable_dist_task-new-in-v710)の値を`ON`に設定します：

    ```sql
    SET GLOBAL tidb_enable_dist_task = ON;
    ```

    <CustomContent platform="tidb">

    バックグラウンドタスクが実行されると、フレームワークでサポートされるステートメント（[`ADD INDEX`](/sql-statements/sql-statement-add-index.md)や[`IMPORT INTO`](/sql-statements/sql-statement-import-into.md)など）が分散形式で実行されます。すべてのTiDBノードでバックグラウンドタスクがデフォルトで実行されます。

    </CustomContent>

    <CustomContent platform="tidb-cloud">

    バックグラウンドタスクが実行されると、フレームワークでサポートされるステートメント（[`ADD INDEX`](/sql-statements/sql-statement-add-index.md)など）が分散形式で実行されます。すべてのTiDBノードでバックグラウンドタスクがデフォルトで実行されます。

    </CustomContent>

2. DDLタスクの分散実行に影響を与える可能性がある以下のシステム変数を必要に応じて調整します：

    * [`tidb_ddl_reorg_worker_cnt`](/system-variables.md#tidb_ddl_reorg_worker_cnt)：デフォルト値は`4`。推奨する最大値は`16`です。
    * [`tidb_ddl_reorg_priority`](/system-variables.md#tidb_ddl_reorg_priority)
    * [`tidb_ddl_error_count_limit`](/system-variables.md#tidb_ddl_error_count_limit)
    * [`tidb_ddl_reorg_batch_size`](/system-variables.md#tidb_ddl_reorg_batch_size)：デフォルト値を使用します。推奨する最大値は`1024`です。

3. v7.4.0からは、必要に応じてバックグラウンドタスクを実行するTiDBノードの数を調整できます。TiDBクラスターを展開した後、クラスター内の各TiDBノードに対してインスタンスレベルのシステム変数[`tidb_service_scope`](/system-variables.md#tidb_service_scope-new-in-v740)を設定できます。TiDBノードの`tidb_service_scope`が`background`に設定されている場合、TiDBノードはバックグラウンドタスクを実行できます。デフォルト値""の場合、TiDBノードはバックグラウンドタスクを実行できません。クラスター内のどのTiDBノードにも`tidb_service_scope`が設定されていない場合、TiDB分散実行フレームワークはデフォルトですべてのTiDBノードにバックグラウンドタスクをスケジュールします。
    > **注意:**
    >
    > - 分散タスクの実行中、一部のTiDBノードがオフラインの場合、分散タスクは`tidb_service_scope`に従ってサブタスクを動的に割り当てるのではなく、利用可能なTiDBノードにサブタスクをランダムに割り当てます。
    > - 分散タスクの実行中、`tidb_service_scope`設定の変更は現在のタスクには影響しませんが、次のタスクから適用されます。

> **ヒント:**
>
> `ADD INDEX`ステートメントの分散実行には、`tidb_ddl_reorg_worker_cnt`の設定のみが必要です。

## 実装原則

TiDBバックエンドタスク分散実行フレームワークのアーキテクチャは次のとおりです：

![TiDBバックエンドタスク分散実行フレームワークのアーキテクチャ](/media/dist-task/dist-task-architect.jpg)

前述の図に示すように、分散フレームワークでのバックエンドタスクの実行は主に以下のモジュールによって処理されます：

- ディスパッチャ：各タスクの分散実行プランを生成し、実行プロセスを管理し、タスクのステータスを変換し、ランタイムタスク情報を収集・フィードバックします。
- スケジューラ：TiDBノード全体でのバックエンドタスクの実行効率を向上するために、分散タスクの実行をTiDBノード間でレプリケートします。
- サブタスクエグゼキューター: 分散サブタスクの実行者。また、サブタスクエグゼキューターはサブタスクの実行ステータスをスケジューラーに返し、スケジューラーは一元的な方法でサブタスクの実行ステータスを更新します。
- リソースプール: 上記のモジュールのコンピューティングリソースをプールし、リソースの使用および管理を量化する基盤を提供します。

## 関連情報

<CustomContent platform="tidb">

* [DDLステートメントの実行原則とベストプラクティス](/ddl-introduction.md)

</CustomContent>
<CustomContent platform="tidb-cloud">

* [DDLステートメントの実行原則とベストプラクティス](https://docs.pingcap.com/tidb/stable/ddl-introduction)

</CustomContent>