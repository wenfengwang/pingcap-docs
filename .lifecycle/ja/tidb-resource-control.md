---
title: リソース制御を使用してリソースの分離を実現する
summary: リソース制御機能を使用してアプリケーションのリソースを制御およびスケジュールする方法を学びます。
---

# リソース制御を使用してリソースの分離を実現する

> **注意:**
>
> この機能は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスターでは利用できません。

クラスター管理者として、リソース制御機能を使用してリソースグループを作成し、リソースグループのクォータを設定し、ユーザーをそれらのグループにバインドすることができます。

TiDBのリソース制御機能には、TiDB層でのフロー制御機能と、TiKV層での優先度スケジューリング機能の2つのリソース管理機能があります。これらの2つの機能は別々にまたは同時に有効にすることができます。詳細については、[リソース制御のパラメータ](#parameters-for-resource-control)を参照してください。これにより、TiDB層はリソースグループのクォータに基づいてユーザーの読み書きリクエストのフローを制御し、TiKV層は読み書きクォータにマップされた優先度に基づいてリクエストをスケジュールできます。これにより、アプリケーションのリソースの分離を確保し、サービス品質（QoS）の要件を満たすことができます。

- TiDBフロー制御: TiDBフロー制御は[トークンバケットアルゴリズム](https://en.wikipedia.org/wiki/Token_bucket)を使用します。バケットにトークンが十分にない場合、およびリソースグループが`BURSTABLE`オプションを指定していない場合、リソースグループへのリクエストはトークンバケットがトークンを補充してリトライするのを待機します。タイムアウトのため、リトライは失敗する可能性があります。

- TiKVスケジューリング: 必要に応じて絶対優先度[`(PRIORITY)`](/information-schema/information-schema-resource-groups.md#examples)を設定できます。さまざまなリソースは、`PRIORITY`の設定に従ってスケジュールされます。高い`PRIORITY`を持つタスクが最初にスケジュールされます。絶対優先度を設定しない場合、TiKVは各リソースグループの`RU_PER_SEC`の値を使用して読み書きリクエストの優先度を決定します。優先度に基づいて、ストレージ層は優先度キューを使用してリクエストをスケジュールおよび処理します。

v7.4.0からはじめて、リソース制御機能はTiFlashリソースの制御をサポートします。その原則はTiDBフロー制御およびTiKVスケジューリングと類似しています。

<CustomContent platform="tidb">

- TiFlashフロー制御: [TiFlashパイプライン実行モデル](/tiflash/tiflash-pipeline-model.md)を使用すると、TiFlashは異なるクエリのCPU消費をより正確に取得し、[リクエストユニット（RU）](#what-is-request-unit-ru)に変換して差し引きます。トラフィック制御にはトークンバケットアルゴリズムが使用されます。
- TiFlashスケジューリング: システムリソースが不足している場合、TiFlashはリソースグループ間のパイプラインタスクを優先度に基づいてスケジュールします。具体的なロジックは、まずTiFlashがリソースグループの`PRIORITY`を評価し、次にCPU使用量と`RU_PER_SEC`を考慮します。その結果、`rg1`と`rg2`が同じ`PRIORITY`を持つが、`rg2`の`RU_PER_SEC`が`rg1`の2倍である場合、`rg2`のCPU使用量は`rg1`の2倍になります。

</CustomContent>

<CustomContent platform="tidb-cloud">

- TiFlashフロー制御: [TiFlashパイプライン実行モデル](http://docs.pingcap.com/tidb/dev/tiflash-pipeline-model)を使用すると、TiFlashは異なるクエリのCPU消費をより正確に取得し、[リクエストユニット（RU）](#what-is-request-unit-ru)に変換して差し引きます。トラフィック制御にはトークンバケットアルゴリズムが使用されます。
- TiFlashスケジューリング: システムリソースが不足している場合、TiFlashはリソースグループ間のパイプラインタスクを優先度に基づいてスケジュールします。具体的なロジックは、まずTiFlashがリソースグループの`PRIORITY`を評価し、次にCPU使用量と`RU_PER_SEC`を考慮します。その結果、`rg1`と`rg2`が同じ`PRIORITY`を持つが、`rg2`の`RU_PER_SEC`が`rg1`の2倍である場合、`rg2`のCPU使用量は`rg1`の2倍になります。

</CustomContent>

## リソース制御のシナリオ

リソース制御機能の導入は、TiDBにとって重要な節目です。これにより、分散データベースクラスターを複数の論理ユニットに分割できます。個々のユニットがリソースを過度に使用しても、他のユニットが必要とするリソースを圧迫することはありません。

この機能を使用すると、次のことができます:

- 異なるシステムからの複数の小規模および中規模のアプリケーションを単一のTiDBクラスターに組み合わせることができます。アプリケーションのワークロードが増加しても、他のアプリケーションの通常の動作に影響を与えません。システムのワークロードが低い場合、クォータを超えた場合でも、ビジーなアプリケーションに必要なシステムリソースを割り当てることができ、リソースの最大利用を実現できます。
- すべてのテスト環境を単一のTiDBクラスターに組み合わせるか、より多くのリソースを消費するバッチタスクを単一のリソースグループにグループ化するかを選択できます。これにより、ハードウェアの利用率が向上し、運用コストが削減される一方で、重要なアプリケーションが常に必要なリソースを確保できます。
- システムに異なるワークロードがある場合、異なるワークロードを別々のリソースグループに配置することができます。リソース制御機能を使用することで、トランザクションアプリケーションの応答時間がデータ分析やバッチアプリケーションによって影響を受けないようにできます。
- クラスターに予期しないSQLのパフォーマンスの問題が発生した場合、SQLバインディングとリソースグループを使用してSQLステートメントのリソース消費を一時的に制限することができます。

また、リソース制御機能を理性的に活用することで、クラスターの数を減らし、運用・保守の難しさを軽減し、管理コストを節約することができます。

## 制限事項

リソース制御によって追加のスケジューリングオーバーヘッドが発生します。そのため、この機能を有効にするとわずかなパフォーマンスの低下（5%未満）が発生する可能性があります。

## リクエストユニット（RU）とは

リクエストユニット（RU）は、TiDBのシステムリソースに対する統一された抽象単位であり、現在はCPU、IOPS、およびIOバンド幅のメトリクスを含んでいます。これは、データベースへの単一リクエストによって消費されるリソース量を示すために使用されます。リクエストによって消費されるRUの量は、操作のタイプやクエリされるまたは変更されるデータの量など、さまざまな要因に依存します。現在、RUには次の表に示すリソースの消費統計が含まれています:

<table>
    <thead>
        <tr>
            <th>リソースタイプ</th>
            <th>RU消費</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan="3">読み取り</td>
            <td>2つのストレージ読み取りバッチが1 RUを消費</td>
        </tr>
        <tr>
            <td>8つのストレージ読み取りリクエストが1 RUを消費</td>
        </tr>
        <tr>
            <td>64 KiBの読み取りリクエストペイロードが1 RUを消費</td>
        </tr>
        <tr>
            <td rowspan="3">書き込み</td>
            <td>1つのストレージ書き込みバッチが各レプリカにつき1 RUを消費</td>
        </tr>
        <tr>
            <td>1つのストレージ書き込みリクエストが1 RUを消費</td>
        </tr>
        <tr>
            <td>1 KiBの書き込みリクエストペイロードが1 RUを消費</td>
        </tr>
        <tr>
            <td>SQL CPU</td>
            <td>3ミリ秒が1 RUを消費</td>
        </tr>
    </tbody>
</table>

現在、TiFlashのリソース制御は、クエリのパイプラインタスクの実行によって消費されるSQL CPUのみを考慮しています。

> **注意:**
>
> - 各書き込み操作は最終的にすべてのレプリカに複製されます（デフォルトでは、TiKVには3つのレプリカがあります）。各レプリケーション操作は異なる書き込み操作と見なされます。
> - ユーザーが実行するクエリに加えて、RUは自動統計収集などのバックグラウンドタスクによって消費されることがあります。
> - 前述の表は、TiDB Self-HostedクラスターのRUの計算に関連するリソースのみを一覧表示しており、ネットワークおよびストレージの消費は除外しています。TiDB Serverless RUsの場合は、[TiDB Serverlessの価格詳細](https://www.pingcap.com/tidb-cloud-serverless-pricing-details/)を参照してください。

## SQLステートメントのRU消費を見積もる

[`EXPLAIN ANALYZE`](/sql-statements/sql-statement-explain-analyze.md#ru-request-unit-consumption)ステートメントを使用して、SQLの実行中に消費されるRUの量を取得できます。RUの量はキャッシュ（たとえば、[コプロセッサキャッシュ](/coprocessor-cache.md)）に影響を受けます。同じSQLが複数回実行される場合、各実行で消費されるRUの量は異なる場合があります。RUの値は各実行の正確な値を表すものではなく、見積もりのための参考になります。

## リソース制御のパラメータ

リソース制御機能には、次のシステム変数またはパラメータが導入されています:

* TiDB: [`tidb_enable_resource_control`](/system-variables.md#tidb_enable_resource_control-new-in-v660)システム変数を使用して、リソースグループのフロー制御を有効にするかどうかを制御できます。

<CustomContent platform="tidb">

* TiKV: [`resource-control.enabled`](/tikv-configuration-file.md#resource-control)パラメータを使用して、リソースグループに基づいたリクエストスケジューリングの使用の有無を制御できます。
* TiFlash: [`tidb_enable_resource_control`](/system-variables.md#tidb_enable_resource_control-new-in-v660) システム変数と [`enable_resource_control`](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file) 構成項目（v7.4.0 で導入）を使用して、TiFlash リソース制御を有効にするかどうかを制御できます。

</CustomContent>

<CustomContent platform="tidb-cloud">

* TiKV: TiDB 自己ホスト型では、`resource-control.enabled` パラメータを使用してリクエストスケジューリングをリソースグループクォータに基づいて使用するかどうかを制御できます。TiDB Cloud の場合、`resource-control.enabled` パラメータの値はデフォルトで `true` であり、動的な変更はサポートされていません。
* TiFlash: TiDB 自己ホスト型では、`tidb_enable_resource_control` システム変数と `enable_resource_control` 構成項目（v7.4.0 で導入）を使用して、TiFlash リソース制御を有効にするかどうかを制御できます。

</CustomContent>

TiDB v7.0.0 からは、`tidb_enable_resource_control` と `resource-control.enabled` がデフォルトで有効になります。これら2つのパラメータの組み合わせの結果は、次の表に示されています。

| `resource-control.enabled`  | `tidb_enable_resource_control`= ON   | `tidb_enable_resource_control`= OFF  |
|:----------------------------|:-------------------------------------|:-------------------------------------|
| `resource-control.enabled`= true  |  フロー制御とスケジューリング（推奨） | 無効な組み合わせ  |
| `resource-control.enabled`= false |  フロー制御のみ（非推奨）                 | この機能は無効です。     |

<CustomContent platform="tidb">

v7.4.0 から、TiFlash 構成項目 `enable_resource_control` はデフォルトで有効になります。これは `tidb_enable_resource_control` と連携して TiFlash リソース制御機能を制御します。`enable_resource_control` が有効な場合、TiFlash は [パイプライン実行モデル](/tiflash/tiflash-pipeline-model.md) を使用します。

</CustomContent>

<CustomContent platform="tidb-cloud">

v7.4.0 から、TiFlash 構成項目 `enable_resource_control` はデフォルトで有効になります。これは `tidb_enable_resource_control` と連携して TiFlash リソース制御機能を制御します。`enable_resource_control` が有効な場合、TiFlash は [パイプライン実行モデル](http://docs.pingcap.com/tidb/dev/tiflash-pipeline-model) を使用します。

</CustomContent>

リソース制御メカニズムおよびパラメータの詳細については、[TiDB でのグローバルリソース制御](https://github.com/pingcap/tidb/blob/master/docs/design/2022-11-25-global-resource-control.md) および [TiFlash のリソース制御](https://github.com/pingcap/tiflash/blob/master/docs/design/2023-09-21-tiflash-resource-control.md) を参照してください。

## リソース制御の使用方法

このセクションでは、リソースグループの管理および各リソースグループのリソース割り当ての制御にリソース制御機能を使用する方法について説明します。

### クラスター容量の見積もり

<CustomContent platform="tidb">

リソースプランニングの前に、クラスターの全体的な容量を把握する必要があります。TiDB は、クラスター容量を見積もるための [`CALIBRATE RESOURCE`](/sql-statements/sql-statement-calibrate-resource.md) ステートメントを提供します。次の方法のいずれかを使用できます。

- [実際のワークロードに基づく容量の見積もり](/sql-statements/sql-statement-calibrate-resource.md#estimate-capacity-based-on-actual-workload)
- [ハードウェアデプロイメントに基づく容量の見積もり](/sql-statements/sql-statement-calibrate-resource.md#estimate-capacity-based-on-hardware-deployment)

TiDB ダッシュボードの [リソースマネージャーページ](/dashboard/dashboard-resource-manager.md) でクラスターの容量を確認できます。詳細については、[`CALIBRATE RESOURCE`](/sql-statements/sql-statement-calibrate-resource.md#methods-for-estimating-capacity) を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

TiDB 自己ホスト型では、[`CALIBRATE RESOURCE`](https://docs.pingcap.com/zh/tidb/stable/sql-statement-calibrate-resource) ステートメントを使用して、クラスター容量を見積もることができます。

TiDB Cloud の場合、[`CALIBRATE RESOURCE`](https://docs.pingcap.com/zh/tidb/stable/sql-statement-calibrate-resource) ステートメントは適用されません。

</CustomContent>

### リソースグループの管理

リソースグループを作成、変更、または削除するには、`SUPER` 権限または `RESOURCE_GROUP_ADMIN` 権限が必要です。

[`CREATE RESOURCE GROUP`](/sql-statements/sql-statement-create-resource-group.md) を使用して、クラスターのためのリソースグループを作成できます。

既存のリソースグループについては、[`ALTER RESOURCE GROUP`](/sql-statements/sql-statement-alter-resource-group.md) を使用してリソースグループの `RU_PER_SEC` オプション（秒あたりの RU バックフィルレート）を変更できます。リソースグループの変更は即座に有効になります。

[`DROP RESOURCE GROUP`](/sql-statements/sql-statement-drop-resource-group.md) を使用してリソースグループを削除できます。

### リソースグループの作成

以下は、リソースグループを作成する方法の例です。

1. `rg1` というリソースグループを作成します。リソース制限は 1 秒あたり 500 RU であり、このリソースグループ内のアプリケーションがリソースを超過しても構いません。

    ```sql
    CREATE RESOURCE GROUP IF NOT EXISTS rg1 RU_PER_SEC = 500 BURSTABLE;
    ```

2. `rg2` というリソースグループを作成します。RU バックフィルレートは 1 秒あたり 600 RU であり、このリソースグループ内のアプリケーションがリソースを超過してはいけません。

    ```sql
    CREATE RESOURCE GROUP IF NOT EXISTS rg2 RU_PER_SEC = 600;
    ```

3. `rg3` というリソースグループを作成します。絶対優先度を `HIGH` に設定します。絶対優先度は現在 `LOW|MEDIUM|HIGH` をサポートしています。デフォルト値は `MEDIUM` です。

    ```sql
    CREATE RESOURCE GROUP IF NOT EXISTS rg3 RU_PER_SEC = 100 PRIORITY = HIGH;
    ```

### リソースグループのバインド

TiDB は、次の 3 つのレベルのリソースグループ設定をサポートしています。

- ユーザーレベル。[`CREATE USER`](/sql-statements/sql-statement-create-user.md) または [`ALTER USER`](/sql-statements/sql-statement-alter-user.md#modify-the-resource-group-bound-to-the-user) ステートメントを使用して、ユーザーを特定のリソースグループにバインドできます。ユーザーがリソースグループにバインドされると、そのユーザーによって作成されたセッションは自動的に対応するリソースグループにバインドされます。
- セッションレベル。[`SET RESOURCE GROUP`](/sql-statements/sql-statement-set-resource-group.md) を使用して、現在のセッションのリソースグループを設定します。
- ステートメントレベル。[`RESOURCE_GROUP()`](/optimizer-hints.md#resource_groupresource_group_name) オプティマイザヒントを使用して、現在のステートメントのリソースグループを設定します。

#### ユーザーをリソースグループにバインド

次の例では、ユーザー `usr1` を作成し、そのユーザーをリソースグループ `rg1` にバインドします。`rg1` は [リソースグループの作成](#create-a-resource-group) の例で作成されたリソースグループです。

```sql
CREATE USER 'usr1'@'%' IDENTIFIED BY '123' RESOURCE GROUP rg1;
```

次の例では、`ALTER USER` を使用してユーザー `usr2` をリソースグループ `rg2` にバインドします。`rg2` は [リソースグループの作成](#create-a-resource-group) の例で作成されたリソースグループです。

```sql
ALTER USER usr2 RESOURCE GROUP rg2;
```

ユーザーをバインドした後、新たに作成されたセッションのリソース消費は指定されたクォータ（リクエストユニット、RU）によって制御されます。システムのワークロードが比較的高く、余剰容量がない場合、`usr2` のリソース消費率は厳格に制御され、クォータを超えないようになります。`usr1` は `BURSTABLE` で構成された `rg1` にバインドされているため、`usr1` の消費率はクォータを超えることが許可されています。

リソースグループのリソースが不足すると多くのリクエストがある場合、クライアントのリクエストは待機します。待機時間が長すぎる場合、リクエストはエラーを報告します。

> **注意:**
>
> - `CREATE USER` または `ALTER USER` を使用してユーザーをリソースグループにバインドすると、ユーザーの既存のセッションには影響しませんが、ユーザーの新しいセッションには影響します。
> - TiDB は、クラスターの初期化中に `default` リソースグループを自動的に作成します。このリソースグループの `RU_PER_SEC` のデフォルト値は `UNLIMITED`（`INT` 型の最大値である `2147483647` と同等）であり、`BURSTABLE` モードです。リソースグループにバインドされていないステートメントは自動的にこのリソースグループにバインドされます。このリソースグループは削除をサポートしていませんが、その RU の構成を変更できます。

ユーザーをリソースグループからバインド解除するには、単に次のようにデフォルトグループに再度バインドすればよいです。

```sql
ALTER USER 'usr3'@'%' RESOURCE GROUP `default`;
```

詳細については、[`ALTER USER ... RESOURCE GROUP`](/sql-statements/sql-statement-alter-user.md#modify-the-resource-group-bound-to-the-user) を参照してください。
#### 現在のセッションをリソースグループにバインドする

セッションをリソースグループにバインドすることで、対応するセッションのリソース使用量（RU）が指定された使用量で制限されます。

以下の例では、現在のセッションをリソースグループ `rg1` にバインドしています。

```sql
SET RESOURCE GROUP rg1;
```

#### 現在のステートメントをリソースグループにバインドする

SQLステートメントに [`RESOURCE_GROUP(resource_group_name)`](/optimizer-hints.md#resource_groupresource_group_name) ヒントを追加することで、ステートメントをバインドするリソースグループを指定できます。このヒントは `SELECT`、`INSERT`、`UPDATE`、`DELETE` ステートメントをサポートしています。

以下の例では、現在のステートメントをリソースグループ `rg1` にバインドしています。

```sql
SELECT /*+ RESOURCE_GROUP(rg1) */ * FROM t limit 10;
```

### 予想以上のリソースを消費するクエリを管理する（暴走クエリ）

> **警告:**
>
> この機能は実験的なものです。本番環境で使用しないことをお勧めします。この機能は予告なく変更または削除される場合があります。バグを見つけた場合は、GitHub 上の [issue](https://github.com/pingcap/tidb/issues) で報告できます。

暴走クエリとは、予想以上に多くの時間やリソースを消費するクエリのことです。**暴走クエリ** という用語は、暴走クエリの管理機能を指すために以下で使用されています。

- v7.2.0 から、リソース制御機能には暴走クエリの管理が導入されています。リソースグループに暴走クエリを特定するための基準を設定し、それらがリソースを使い果たし、他のクエリに影響を与えるのを防ぐために自動的にアクションを取ることができます。[`CREATE RESOURCE GROUP`](/sql-statements/sql-statement-create-resource-group.md) または [`ALTER RESOURCE GROUP`](/sql-statements/sql-statement-alter-resource-group.md) で `QUERY_LIMIT` フィールドをリソースグループに含めることで、リソースグループの暴走クエリを管理できます。
- v7.3.0 から、リソース制御機能には暴走ウォッチのマニュアル管理が導入され、指定されたSQLステートメントまたはダイジェストでの暴走クエリを迅速に特定することが可能です。`QUERY WATCH` ステートメントを実行してリソースグループの暴走ウォッチリストを手動で管理できます。

#### `QUERY_LIMIT` パラメータ

サポートされる条件設定:

- `EXEC_ELAPSED`: クエリの実行時間がこの制限を超えると、そのクエリは暴走クエリとして特定されます。

サポートされる操作（`ACTION`）:

- `DRYRUN`: 何も行いません。暴走クエリのレコードは追加されます。これは主に、条件設定が合理的かどうかを観察するために使用されます。
- `COOLDOWN`: クエリの実行優先度が最低レベルまで低下します。このクエリは最低優先度で継続して実行され、他の操作のリソースを占有しません。
- `KILL`: 特定されたクエリは自動的に終了され、エラー `Query execution was interrupted, identified as runaway query` が報告されます。

システムリソースを使い果たすような多くの同時暴走クエリを避けるために、リソース制御機能はクイック特定メカニズムを導入し、暴走クエリを迅速に特定し分離できるようにしています。`WATCH` 句を介してこの機能を使用できます。クエリが暴走クエリとして特定されると、このメカニズムはクエリのマッチング特徴（`WATCH` の後のパラメータで定義される）を抽出します。次の一定期間（`DURATION` で定義される）中、暴走クエリのマッチング特徴がウォッチリストに追加され、TiDB インスタンスはウォッチリストでクエリをマッチングし、対応するアクションに従ってクエリを直ちに暴走クエリと単独化します。`KILL` 操作はクエリを終了し、エラー `Quarantined and interrupted because of being in runaway watch list` を報告します。

`WATCH` でクイック特定を行うためのマッチ方法には3つの方法があります：

- `EXACT` は、まったく同じSQL文のみが迅速に特定されます。
- `SIMILAR` は、同一のパターンを持つすべてのSQL文がプランダイジェストでマッチングされ、リテラル値は無視されます。
- `PLAN` は、同一のパターンを持つすべてのSQL文がプランダイジェストでマッチングされます。

`WATCH` の`DURATION` オプションは既定では無限です。

ウォッチアイテムが追加された後は、`QUERY_LIMIT` 構成が変更または削除された場合でも、マッチング特徴や `ACTION` は変更または削除されません。`QUERY WATCH REMOVE` を使用してウォッチアイテムを削除できます。

`QUERY_LIMIT` のパラメータは以下の通りです：

| パラメータ          | 説明            | 注釈                                  |
|---------------|--------------|--------------------------------------|
| `EXEC_ELAPSED`  | クエリの実行時間がこの値を超えると、それが暴走クエリとして特定されます | `EXEC_ELAPSED ='60s'` は、クエリが実行に60秒以上かかる場合、それを暴走クエリとして特定します。 |
| `ACTION`    | 暴走クエリが特定された場合の実行する処置 | `DRYRUN`、`COOLDOWN`、`KILL` の選択肢があります。 |
| `WATCH`   | 特定する暴走クエリを迅速にマッチさせます。一定期間内に同じまたは類似したクエリが再度出会われると、対応するアクションが即座に実行されます。 | オプションです。例: `WATCH=SIMILAR DURATION '60s'`、`WATCH=EXACT DURATION '1m'`、`WATCH=PLAN`。 |

#### 例

1. リソースグループ `rg1` を毎秒500 RUのクォータとともに作成し、クエリが60秒を超えるとその優先度を下げる暴走クエリを定義する。

    ```sql
    CREATE RESOURCE GROUP IF NOT EXISTS rg1 RU_PER_SEC = 500 QUERY_LIMIT=(EXEC_ELAPSED='60s', ACTION=COOLDOWN);
    ```

2. `rg1` リソースグループを変更して暴走クエリを終了し、同じパターンのクエリが次の10分間で即座に暴走クエリとしてマークされるようにします。

    ```sql
    ALTER RESOURCE GROUP rg1 QUERY_LIMIT=(EXEC_ELAPSED='60s', ACTION=KILL, WATCH=SIMILAR DURATION='10m');
    ```

3. `rg1` リソースグループを変更して暴走クエリのチェックをキャンセルします。

    ```sql
    ALTER RESOURCE GROUP rg1 QUERY_LIMIT=NULL;
    ```

#### `QUERY WATCH` パラメータ

`QUERY WATCH` のシノプシスについての詳細情報については、[`QUERY WATCH`](/sql-statements/sql-statement-query-watch.md) を参照してください。

パラメータは以下の通りです：

- `RESOURCE GROUP` はリソースグループを指定します。このステートメントによって追加される暴走クエリのマッチング特徴は、リソースグループのウォッチリストに追加されます。このパラメータは省略することができます。省略された場合は、`default` リソースグループに適用されます。
- `ACTION` の意味は `QUERY LIMIT` と同じです。このパラメータは省略することができます。省略された場合、特定後の対応アクションはリソースグループの `QUERY LIMIT` で構成された `ACTION` を採用し、`QUERY LIMIT` 構成が変更されてもアクションは変更されません。リソースグループで `ACTION` が構成されていない場合はエラーが報告されます。
- `QueryWatchTextOption` パラメータには 3 つのオプションがあります: `SQL DIGEST`、`PLAN DIGEST`、`SQL TEXT`。
    - `SQL DIGEST` は `SIMILAR` と同じです。以下のパラメータは文字列、ユーザー定義変数、または他の文字列の結果を生成する式を受け入れます。文字列の長さは64でなければならず、これは TiDB での Digest の定義と同じです。
    - `PLAN DIGEST` は `PLAN` と同じです。以下のパラメータは Digest 文字列です。
    - `SQL TEXT` は入力SQLを生の文字列 (`EXACT`) として一致させるか、それを `SQL DIGEST` (`SIMILAR`) または `PLAN DIGEST` (`PLAN`) に解析してコンパイルします。これは次のパラメータに依存します。

- デフォルトリソースグループに対して、暴走クエリウォッチリストに一致する特徴を追加します（デフォルトリソースグループに事前に `QUERY LIMIT` を設定する必要があります）。

    ```sql
    QUERY WATCH ADD ACTION KILL SQL TEXT EXACT TO 'select * from test.t2';
    ```

- SQLをSQLダイジェストに解析して `rg1` リソースグループの暴走クエリウォッチリストに一致する特徴を追加します。`ACTION` が指定されていない場合、`rg1` リソースグループで既に構成されている `ACTION` オプションが使用されます。

    ```sql
    QUERY WATCH ADD RESOURCE GROUP rg1 SQL TEXT SIMILAR TO 'select * from test.t2';
    ```

- `PLAN DIGEST` を使用して `rg1` リソースグループの暴走クエリウォッチリストに一致する特徴を追加します。

    ```sql
    QUERY WATCH ADD RESOURCE GROUP rg1 ACTION KILL PLAN DIGEST 'd08bc323a934c39dc41948b0a073725be3398479b6fa4f6dd1db2a9b115f7f57';
    ```

- `INFORMATION_SCHEMA.RUNAWAY_WATCHES` をクエリしてウォッチアイテムIDを取得し、ウォッチアイテムを削除します。

    ```sql
    SELECT * from information_schema.runaway_watches ORDER BY id;
    ```

    ```sql
    *************************** 1. row ***************************
                    ID: 20003
    RESOURCE_GROUP_NAME: rg2
            START_TIME: 2023-07-28 13:06:08
            END_TIME: UNLIMITED
                WATCH: Similar
            WATCH_TEXT: 5b7fd445c5756a16f910192ad449c02348656a5e9d2aa61615e6049afbc4a82e
                SOURCE: 127.0.0.1:4000
    ```
                アクション: Kill
    (0.00 秒で) 1 行がセットになりました

    ```sql
    QUERY WATCH REMOVE 20003;
    ```

#### 監視可能性

以下のシステムテーブルと `INFORMATION_SCHEMA` から、runaway クエリに関する情報を取得できます。

+ `mysql.tidb_runaway_queries` テーブルには、過去 7 日間に識別されたすべての runaway クエリの履歴レコードが含まれています。次の行を例として挙げます。

    ```sql
    MySQL [(なし)]> SELECT * FROM mysql.tidb_runaway_queries LIMIT 1\G;
    *************************** 1. 行 ***************************
    resource_group_name: rg1
                   time: 2023-06-16 17:40:22
             match_type: identify
                 action: kill
           original_sql: select * from sbtest.sbtest1
            plan_digest: 5b7d445c5756a16f910192ad449c02348656a5e9d2aa61615e6049afbc4a82e
            tidb_server: 127.0.0.1:4000
    ```

    上記の出力では、`match_type` は runaway クエリの識別方法を示しています。値は次のいずれかです。

    - `identify`: runaway クエリの条件に一致します。
    - `watch`: watch リストのクイック識別ルールに一致します。

+ `information_schema.runaway_watches` テーブルには、runaway クエリのクイック識別ルールのレコードが含まれています。詳細は [`RUNAWAY_WATCHES`](/information-schema/information-schema-runaway-watches.md) を参照してください。

### バックグラウンドタスクの管理

> **警告:**
>
> この機能は実験的なものです。本番環境で使用することはお勧めしません。この機能は予告なく変更または削除される可能性があります。バグを見つけた場合は、GitHub で[問題](https://docs.pingcap.com/tidb/stable/support)を報告できます。

データバックアップや自動統計収集などのバックグラウンドタスクは、優先度は低いが多くのリソースを消費します。これらのタスクは通常、定期的または不定期にトリガーされます。実行中は多くのリソースを消費するため、オンラインの高優先度タスクのパフォーマンスに影響を与えます。

TiDB 7.4.0 からは、TiDB リソース制御機能がバックグラウンドタスクの管理をサポートしています。タスクがバックグラウンドタスクとしてマークされると、TiKV はこのタイプのタスクによって他の前面タスクのパフォーマンスに影響が出ないようにリソース使用量を動的に制限します。TiKV はすべての前面タスクによってリアルタイムで消費される CPU および IO リソースをモニタリングし、インスタンスの総リソース制限に基づいてバックグラウンドタスクが使用できるリソース閾値を計算します。実行中の間、すべてのバックグラウンドタスクはこの閾値によって制限されます。

#### `BACKGROUND` パラメータ

`TASK_TYPES`: バックグラウンドタスクとして管理する必要のあるタスクの種類を指定します。複数のタスクタイプをカンマ (`,`) で区切って使用します。

TiDB は以下の種類のバックグラウンドタスクをサポートしています:

<CustomContent platform="tidb">

- `lightning`: [TiDB Lightning](/tidb-lightning/tidb-lightning-overview.md) を使用してインポートタスクを実行します。TiDB Lightning の物理的および論理的なインポートモードの両方がサポートされます。
- `br`: [BR](/br/backup-and-restore-overview.md) を使用してバックアップおよびリストアタスクを実行します。PITR はサポートされていません。
- `ddl`: Reorg DDL のバッチデータ書き戻しフェーズのリソース使用状況を制御します。
- `stats`: TiDB によって手動で実行されるまたは自動的にトリガーされる[統計情報の収集](/statistics.md#collect-statistics)タスク。

</CustomContent>

<CustomContent platform="tidb-cloud">

- `lightning`: [TiDB Lightning](https://docs.pingcap.com/tidb/stable/tidb-lightning-overview) を使用してインポートタスクを実行します。TiDB Lightning の物理的および論理的なインポートモードの両方がサポートされます。
- `br`: [BR](https://docs.pingcap.com/tidb/stable/backup-and-restore-overview) を使用してバックアップおよびリストアタスクを実行します。PITR はサポートされていません。
- `ddl`: Reorg DDL のバッチデータ書き戻しフェーズのリソース使用状況を制御します。
- `stats`: TiDB によって手動で実行されるまたは自動的にトリガーされる[統計情報の収集](/statistics.md#collect-statistics)タスク。

</CustomContent>

デフォルトでは、バックグラウンドタスクとしてマークされたタスクのタイプは空であり、バックグラウンドタスクの管理は無効になっています。このデフォルトの動作は、TiDB v7.4.0 より前のバージョンと同じです。バックグラウンドタスクを管理するためには、`default` リソースグループのバックグラウンドタスクタイプを手動で変更する必要があります。

#### 例

1. `rg1` リソースグループを作成し、`br` および `stats` をバックグラウンドタスクとして設定します。

    ```sql
    CREATE RESOURCE GROUP IF NOT EXISTS rg1 RU_PER_SEC = 500 BACKGROUND=(TASK_TYPES='br,stats');
    ```

2. `rg1` リソースグループを変更し、`br` および `ddl` をバックグラウンドタスクとして設定します。

    ```sql
    ALTER RESOURCE GROUP rg1 BACKGROUND=(TASK_TYPES='br,ddl');
    ```

3. `rg1` リソースグループのバックグラウンドタスクをデフォルト値に復元します。この場合、バックグラウンドタスクのタイプは `default` リソースグループの構成に従います。

    ```sql
    ALTER RESOURCE GROUP rg1 BACKGROUND=NULL;
    ```

4. `rg1` リソースグループを変更し、バックグラウンドタスクのタイプを空に設定します。この場合、このリソースグループのすべてのタスクはバックグラウンドタスクとして扱われません。

    ```sql
    ALTER RESOURCE GROUP rg1 BACKGROUND=(TASK_TYPES="");
    ```

## リソース制御の無効化

<CustomContent platform="tidb">

1. リソース制御機能を無効にするには、次の文を実行します。

    ```sql
    SET GLOBAL tidb_enable_resource_control = 'OFF';
    ```

2. リソースグループの RU に基づくスケジューリングを無効にするには、次の TiKV パラメータ [`resource-control.enabled`](/tikv-configuration-file.md#resource-control) を `false` に設定します。

3. TiFlash のリソース制御を無効にするには、次の TiFlash 構成項目 [`enable_resource_control`](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file) を `false` に設定します。

</CustomContent>

<CustomContent platform="tidb-cloud">

1. リソース制御機能を無効にするには、次の文を実行します。

    ```sql
    SET GLOBAL tidb_enable_resource_control = 'OFF';
    ```

2. TiDB Self-Hosted の場合、リソースグループクォータに基づくリクエストのスケジューリングを制御するための `resource-control.enabled` パラメータを使用できます。TiDB Cloud の場合、`resource-control.enabled` パラメータの値はデフォルトで `true` であり、動的な変更はサポートされていません。TiDB Dedicated クラスタの場合は、無効にする必要がある場合は、[TiDB Cloud サポート](/tidb-cloud/tidb-cloud-support.md)にお問い合わせください。

3. TiDB Self-Hosted の場合、`enable_resource_control` 構成項目を使用して、TiFlash リソース制御を有効にするかどうかを制御できます。TiDB Cloud の場合、`enable_resource_control` パラメータの値はデフォルトで `true` であり、動的な変更はサポートされていません。TiDB Dedicated クラスタの場合は、無効にする必要がある場合は、[TiDB Cloud サポート](/tidb-cloud/tidb-cloud-support.md)にお問い合わせください。

</CustomContent>

## 監視メトリクスとチャート

<CustomContent platform="tidb">

TiDB はリソース制御に関するランタイム情報を定期的に収集し、Grafana の **TiDB** > **Resource Control** ダッシュボードでメトリクスの視覚化チャートを提供しています。メトリクスの詳細については、[TiDB 重要な監視メトリクス](/grafana-tidb-dashboard.md) の **Resource Control** セクションを参照してください。

TiKV は異なるリソースグループからのリクエスト QPS を記録します。詳細については、[TiKV 監視メトリクスの詳細](/grafana-tikv-dashboard.md#grpc)を参照してください。

TiDB Dashboard の現在の [`RESOURCE_GROUPS`](/information-schema/information-schema-resource-groups.md) テーブルで、リソースグループのデータを表示できます。詳細については [Resource Manager ページ](/dashboard/dashboard-resource-manager.md) を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **注意:**
>
> このセクションは TiDB Self-Hosted にのみ適用されます。現在、TiDB Cloud ではリソース制御メトリクスは提供されていません。

TiDB はリソース制御に関するランタイム情報を定期的に収集し、Grafana の **TiDB** > **Resource Control** ダッシュボードでメトリクスの視覚化チャートを提供しています。

TiKV は異なるリソースグループからのリクエスト QPS を Grafana の **TiKV** ダッシュボードで記録しています。

</CustomContent>

## ツールの互換性

リソース制御機能は、データのインポート、エクスポート、およびその他のレプリケーションツールの通常の使用に影韓を与えません。BR、TiDB Lightning、および TiCDC では、リソース制御に関連する DDL 操作の処理は現在サポートされておらず、リソース消費もリソース制御によって制限されません。

## FAQ

1. リソースグループを使用したくない場合、リソース制御を無効にする必要がありますか？

    いいえ。リソースグループを指定しないユーザは、無制限のリソースを持つ `default` リソースグループにバインドされます。すべてのユーザが `default` リソースグループに属する場合、リソース割り当て方法はリソース制御が無効になった場合と同じです。

2. データベースユーザは複数のリソースグループにバインドできますか？

      + {R}
      + {R}
    + {R}
  + {R}