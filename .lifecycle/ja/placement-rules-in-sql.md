---
title: SQLにおける配置ルール
summary: SQLステートメントを使用してテーブルやパーティションの配置をスケジュールする方法について学びます。
---

# SQLにおける配置ルール

SQLにおける配置ルールは、SQLステートメントを使用して、TiKVクラスタ内でデータをどこに格納するかを指定できる機能です。この機能により、クラスタ、データベース、テーブル、またはパーティションのデータを特定の地域、データセンター、ラック、またはホストにスケジュールできます。

この機能を使用すると、次のようなユースケースを実現できます:

- 複数のデータセンターにデータを展開し、高可用性戦略を最適化するためのルールを構成します。
- 異なるアプリケーションからの複数のデータベースをマージし、物理的に異なるユーザーのデータを分離して、インスタンス内の異なるユーザーの分離要件を満たします。
- 重要なデータのレプリカ数を増やして、アプリケーションの可用性とデータの信頼性を向上させます。

> **注意:**
>
> この機能は[TiDB Serverless](https://docs.pingcap.com/tidbcloud/select-cluster-tier#tidb-serverless)クラスタでは利用できません。

## 概要

SQLにおける配置ルールの機能を使用すると、次のように、コースから細かい粒度で異なるレベルのデータのために配置ポリシーを作成し、設定できます:

| レベル          | 説明                                                                                         |
|----------------|-------------------------------------------------------------------------------------------|
| クラスタ        | デフォルトでは、TiDBはクラスタのために3つのレプリカのポリシーを構成します。クラスタ全体に対してグローバルな配置ポリシーを構成できます。詳細は[クラスタ全体にレプリカ数を指定する](#specify-the-number-of-replicas-globally-for-a-cluster)を参照してください。 |
| データベース    | 特定のデータベースのために配置ポリシーを構成できます。詳細は[データベースのためにデフォルトの配置ポリシーを指定する](#specify-a-default-placement-policy-for-a-database)を参照してください。 |
| テーブル        | 特定のテーブルのために配置ポリシーを構成できます。詳細は[テーブルのために配置ポリシーを指定する](#specify-a-placement-policy-for-a-table)を参照してください。 |
| パーティション  | テーブル内の異なる行に対してパーティションを作成し、それぞれのパーティションに対して個別に配置ポリシーを構成できます。詳細は[パーティションされたテーブルのために配置ポリシーを指定する](#specify-a-placement-policy-for-a-partitioned-table)を参照してください。 |

> **ヒント:**
>
> *SQLにおける配置ルール*の実装は、PDの*配置ルール機能*に依存しています。詳細については、[配置ルールの構成](https://docs.pingcap.com/zh/tidb/stable/configure-placement-rules)を参照してください。SQLにおける配置ルールのコンテキストでは、*配置ルール*は他のオブジェクトに付与された*配置ポリシー*、またはTiDBからPDに送信されたルールを指す場合があります。

## 制限事項

- メンテナンスを簡素化するために、クラスタ内の配置ポリシーの数を10個以下に制限することを推奨します。
- 配置ポリシーがアタッチされているテーブルやパーティションの総数を10,000個以下に制限することを推奨します。多くのテーブルやパーティションにポリシーをアタッチすると、PD上での演算負荷が増加し、サービスのパフォーマンスに影響を与える可能性があります。
- 他の複雑な配置ポリシーを使用するのではなく、このドキュメントで提供されている例に従ってSQLにおける配置ルールの機能を使用することを推奨します。

## 前提条件

配置ポリシーは、TiKVノードのラベルの構成に依存しています。たとえば、`PRIMARY_REGION`の配置オプションは、TiKVの`region`ラベルに依存しています。

<CustomContent platform="tidb">

配置ポリシーを作成する際、ポリシーで指定されたラベルの存在をTiDBがチェックしません。代わりに、ポリシーをアタッチする際にチェックを実行します。したがって、配置ポリシーをアタッチする前に、各TiKVノードが正しいラベルで構成されていることを確認してください。TiDB Self-Hostedクラスターの構成方法は次のとおりです:

```
tikv-server --labels region=<region>,zone=<zone>,host=<host>
```

詳細な構成方法については、次の例を参照してください:

| デプロイメント方法 | 例 |
| --- | --- |
| 手動デプロイ | [トポロジーラベルによるレプリカのスケジュール](/schedule-replicas-by-topology-labels.md) |
| TiUPを使用したデプロイ | [地理的に分散したデプロイメントトポロジー](/geo-distributed-deployment-topology.md) |
| TiDB Operatorを使用したデプロイ | [Kubernetes内でのTiDBクラスターの構成](https://docs.pingcap.com/tidb-in-kubernetes/stable/configure-a-tidb-cluster#high-data-high-availability) |

> **注意:**
>
> TiDB Dedicatedクラスターでは、TiKVノードにラベルが自動的に構成されているため、これらのラベルの構成手順をスキップすることができます。

</CustomContent>

<CustomContent platform="tidb-cloud">

TiDB Dedicatedクラスターでは、TiKVノードのラベルは自動的に構成されます。

</CustomContent>

現在のTiKVクラスターで利用可能なすべてのラベルを表示するには、[`SHOW PLACEMENT LABELS`](/sql-statements/sql-statement-show-placement-labels.md)ステートメントを使用できます:

```sql
SHOW PLACEMENT LABELS;
+--------+----------------+
| Key    | Values         |
+--------+----------------+
| disk   | ["ssd"]        |
| region | ["us-east-1"]  |
| zone   | ["us-east-1a"] |
+--------+----------------+
3 rows in set (0.00 sec)
```

## 使用法

このセクションでは、SQLステートメントを使用して配置ポリシーを作成、アタッチ、表示、変更、削除する方法について説明します。

### 配置ポリシーの作成とアタッチ

1. 配置ポリシーを作成するには、[`CREATE PLACEMENT POLICY`](/sql-statements/sql-statement-create-placement-policy.md)ステートメントを使用します:

    ```sql
    CREATE PLACEMENT POLICY myplacementpolicy PRIMARY_REGION="us-east-1" REGIONS="us-east-1,us-west-1";
    ```

    このステートメントでは:

    - `PRIMARY_REGION="us-east-1"`オプションは、Raft Leaderを`region`ラベルが`us-east-1`のノードに配置することを意味します。
    - `REGIONS="us-east-1,us-west-1"`オプションは、Raft Followerを`region`ラベルが`us-east-1`のノードおよび`region`ラベルが`us-west-1`のノードに配置することを意味します。

    より多くの構成可能な配置オプションとその意味については、[配置オプション](#placement-option-reference)を参照してください。

2. テーブルまたはパーティション化されたテーブルに配置ポリシーをアタッチするには、そのテーブルまたはパーティション化されたテーブルの`CREATE TABLE`または`ALTER TABLE`ステートメントを使用して、そのテーブルまたはパーティション化されたテーブルの配置ポリシーを指定します:

    ```sql
    CREATE TABLE t1 (a INT) PLACEMENT POLICY=myplacementpolicy;
    CREATE TABLE t2 (a INT);
    ALTER TABLE t2 PLACEMENT POLICY=myplacementpolicy;
    ```

   `PLACEMENT POLICY`はデータベーススキーマに関連付けられておらず、グローバルスコープでアタッチできます。したがって、`CREATE TABLE`を使用して配置ポリシーを指定する際には、追加の特権が必要ありません。

### 配置ポリシーの表示

- 既存の配置ポリシーを表示するには、[`SHOW CREATE PLACEMENT POLICY`](/sql-statements/sql-statement-show-create-placement-policy.md)ステートメントを使用できます:

    ```sql
    SHOW CREATE PLACEMENT POLICY myplacementpolicy\G
    *************************** 1. row ***************************
           Policy: myplacementpolicy
    Create Policy: CREATE PLACEMENT POLICY myplacementpolicy PRIMARY_REGION="us-east-1" REGIONS="us-east-1,us-west-1"
    1 row in set (0.00 sec)
    ```

- 特定のテーブルにアタッチされた配置ポリシーを表示するには、[`SHOW CREATE TABLE`](/sql-statements/sql-statement-show-create-table.md)ステートメントを使用できます:

    ```sql
    SHOW CREATE TABLE t1\G
    *************************** 1. row ***************************
           Table: t1
    Create Table: CREATE TABLE `t1` (
      `a` int(11) DEFAULT NULL
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin /*T![placement] PLACEMENT POLICY=`myplacementpolicy` */
    1 row in set (0.00 sec)
    ```

- クラスター内の配置ポリシーの定義を表示するには、[`INFORMATION_SCHEMA.PLACEMENT_POLICIES`](/information-schema/information-schema-placement-policies.md)システムテーブルをクエリできます:

    ```sql
    SELECT * FROM information_schema.placement_policies\G
    ***************************[ 1. row ]***************************
    POLICY_ID            | 1
    CATALOG_NAME         | def
    POLICY_NAME          | p1
    PRIMARY_REGION       | us-east-1
    REGIONS              | us-east-1,us-west-1
    CONSTRAINTS          |
    LEADER_CONSTRAINTS   |
    FOLLOWER_CONSTRAINTS |
    LEARNER_CONSTRAINTS  |
    SCHEDULE             |
    FOLLOWERS            | 4
    LEARNERS             | 0
    1 row in set
    ```

- クラスター内で配置ポリシーがアタッチされているすべてのテーブルを表示するには、`information_schema.tables`システムテーブルの`tidb_placement_policy_name`列をクエリできます:

    ```sql
    SELECT * FROM information_schema.tables WHERE tidb_placement_policy_name IS NOT NULL;
    ```

- クラスター内で配置ポリシーがアタッチされているすべてのパーティションを表示するには、`information_schema.partitions`システムテーブルの`tidb_placement_policy_name`列をクエリできます:

    ```sql
    SELECT * FROM information_schema.partitions WHERE tidb_placement_policy_name IS NOT NULL;
    ```
- すべてのオブジェクトに添付された配置ポリシーは*非同期*に適用されます。配置ポリシーのスケジューリングの進捗状況を確認するには、 [`SHOW PLACEMENT`](/sql-statements/sql-statement-show-placement.md) ステートメントを使用できます：

    ```sql
    SHOW PLACEMENT;
    ```

### 配置ポリシーの変更

配置ポリシーを変更するには、 [`ALTER PLACEMENT POLICY`](/sql-statements/sql-statement-alter-placement-policy.md) ステートメントを使用できます。変更は、対応するポリシーで添付されたすべてのオブジェクトに適用されます。

```sql
ALTER PLACEMENT POLICY myplacementpolicy FOLLOWERS=4;
```

このステートメントにおいて、`FOLLOWERS=4` オプションは、データの5つのレプリカ（4つのフォロワーと1つのリーダー）を設定することを意味します。より詳細な配置オプションとその意味については、[配置オプションリファレンス](#配置オプションリファレンス)を参照してください。

### 配置ポリシーの削除

テーブルやパーティションに添付されていないポリシーを削除するには、[`DROP PLACEMENT POLICY`](/sql-statements/sql-statement-drop-placement-policy.md) ステートメントを使用できます：

```sql
DROP PLACEMENT POLICY myplacementpolicy;
```

## 配置オプションリファレンス

配置ポリシーを作成または修正する際には、必要に応じて配置オプションを構成できます。

> **注意：**
>
>`PRIMARY_REGION`、`REGIONS`、 `SCHEDULE` オプションは `CONSTRAINTS` オプションと一緒に指定できません。そうするとエラーが発生します。

### 通常の配置オプション

通常の配置オプションは、データ配置の基本的な要件を満たすことができます。

| オプション名                | 説明                                                                                    |
|----------------------------|------------------------------------------------------------------------------------------------|
| `PRIMARY_REGION`           | このオプションの値と一致する `region` ラベルを持つノードに Raft リーダーを配置するよう指定します。     |
| `REGIONS`                  | このオプションの値と一致する `region` ラベルを持つノードに Raft フォロワーを配置するよう指定します。 |
| `SCHEDULE`                 | フォロワーの配置のスケジューリング戦略を指定します。値のオプションは `EVEN`（デフォルト）または `MAJORITY_IN_PRIMARY` です。 |
| `FOLLOWERS`                | フォロワーの数を指定します。例えば、`FOLLOWERS=2` はデータの3つのレプリカ（2つのフォロワーと1つのリーダー）を意味します。 |

### 高度な配置オプション

高度な構成オプションは、複雑なシナリオのデータ配置の要件を満たすためにより柔軟な構成オプションを提供します。ただし、高度なオプションの構成は通常のオプションよりも複雑であり、クラスタのトポロジと TiDB のデータシャーディングについて深い理解が必要です。

| オプション名                | 説明                                                                                    |
| --------------| ------------ |
| `CONSTRAINTS`              | すべてのロールに適用される制約のリスト。例えば、`CONSTRAINTS="[+disk=ssd]"` です。 |
| `LEADER_CONSTRAINTS`       | リーダーにのみ適用される制約のリスト。                                      |
| `FOLLOWER_CONSTRAINTS`     | フォロワーにのみ適用される制約のリスト。                                   |
| `LEARNER_CONSTRAINTS`      | レプリカにのみ適用される制約のリスト。                                     |
| `LEARNERS`                 | レプリカの数。 |
| `SURVIVAL_PREFERENCE`      | ラベルの障害耐性レベルにしたがってレプリカの配置優先度を指定します。例えば、`SURVIVAL_PREFERENCE="[region, zone, host]"` です。 |

### CONSTRAINTSのフォーマット

`CONSTRAINTS`、`FOLLOWER_CONSTRAINTS`、`LEARNER_CONSTRAINTS` 配置オプションは、以下のいずれかの形式で構成できます：

| CONSTRAINTS フォーマット | 説明 |
|----------------------------|-----------------------------------------------------------------------------------------------------------|
| リスト形式  | 指定する制約がすべてのレプリカに適用される場合は、キー＝値のリスト形式を使用できます。各キーは `+` または `-` で始まります。例：<br/><ul><li>`[+region=us-east-1]` は `region` ラベルが `us-east-1` であるノードにデータを配置します。</li><li>`[+region=us-east-1,-type=fault]` は `region` ラベルが `us-east-1` であるノードかつ `type` ラベルが `fault` でないノードにデータを配置します。</li></ul><br/>  |
| 辞書形式 | 異なる制約に対して異なる数のレプリカを指定する必要がある場合は、辞書形式を使用できます。例えば：<br/><ul><li>`FOLLOWER_CONSTRAINTS="{+region=us-east-1: 1,+region=us-east-2: 1,+region=us-west-1: 1}";` は `us-east-1`、`us-east-2`、`us-west-1` にそれぞれ 1 つのフォロワーを配置します。</li><li>`FOLLOWER_CONSTRAINTS='{"+region=us-east-1,+type=scale-node": 1,"+region=us-west-1": 1}';` は `us-east-1` には `scale-node` ラベルがあり、`us-west-1` には 1 つフォロワーを配置します。</li></ul>辞書形式は各キーが `+` または `-` で始まり、特別な `#reject-leader` 属性を構成できます。例えば、`FOLLOWER_CONSTRAINTS='{"+region=us-east-1":1, "+region=us-east-2": 2, "+region=us-west-1,#reject-leader": 1}'` は、`us-west-1` で選出されたリーダーは災害復旧時に可能な限り追放されます。|

> **注意：**
>
> - `LEADER_CONSTRAINTS` 配置オプションはリスト形式のみをサポートします。
> - リスト形式と辞書形式は YAML パーサーに基づいていますが、一部のケースでは YAML 構文が誤って解釈されることがあります。例えば、`:}""`（`:` の後にスペースがない）は不意に `'{"+region=east:1": null, "+region=west:2": null}'` として解釈される可能性がありますが、`: 1,+region=west: 2`（`:` の後にスペースあり）は `'{"+region=east": 1, "+region=west": 2}'` と正しく解釈されます。そのため、`:` の後にスペースを加えることをお勧めします。

## 基本的な例

### クラスタ全体でレプリカの数を指定する

クラスタが初期化された後、デフォルトのレプリカ数は `3` です。クラスタにより多くのレプリカが必要な場合は、配置ポリシーを構成して、[`ALTER RANGE`](/sql-statements/sql-statement-alter-range.md) を使用してクラスタレベルでポリシーを適用できます。例えば：

```sql
CREATE PLACEMENT POLICY five_replicas FOLLOWERS=4;
ALTER RANGE global PLACEMENT POLICY five_replicas;
```

TiDB はリーダーの数をデフォルトで `1` にするため、`five replicas` は `4` つのフォロワーと `1` つのリーダーを意味します。

### データベースのデフォルト配置ポリシーを指定する

データベースのデフォルト配置ポリシーを指定できます。これは、データベースのデフォルト文字セットまたは照合順序を設定するのと同様に機能します。データベースのテーブルやパーティションに他の配置ポリシーが指定されていない場合、データベースの配置ポリシーがそのテーブルとパーティションに適用されます。例えば：

```sql
CREATE PLACEMENT POLICY p1 PRIMARY_REGION="us-east-1" REGIONS="us-east-1,us-east-2";  -- 配置ポリシーを作成

CREATE PLACEMENT POLICY p2 FOLLOWERS=4;

CREATE PLACEMENT POLICY p3 FOLLOWERS=2;

CREATE TABLE t1 (a INT);  -- 配置ポリシーを指定せずにテーブル t1 を作成

ALTER DATABASE test PLACEMENT POLICY=p2;  -- データベースのデフォルト配置ポリシーを p2 に変更し、既存のテーブル t1 には適用されません。

CREATE TABLE t2 (a INT);  -- テーブル t2 を作成。デフォルト配置ポリシー p2 が t2 に適用されます。

CREATE TABLE t3 (a INT) PLACEMENT POLICY=p1;  -- デフォルトの配置ルールが指定されたため、デフォルト配置ポリシー p2 はテーブル t3 に適用されません。

ALTER DATABASE test PLACEMENT POLICY=p3;  -- データベースのデフォルトポリシーを再度変更し、既存のテーブルには適用されません。

CREATE TABLE t4 (a INT);  -- デフォルト配置ポリシー p3 が t4 に適用されます。

ALTER PLACEMENT POLICY p3 FOLLOWERS=3; -- `FOLLOWERS=3` はポリシー p3 に添付されたテーブル（つまり table t4 ）に適用されます。
```

表からパーティションへのポリシーの継承は、前述の例のポリシーの継承とは異なります。テーブルのデフォルトポリシーを変更すると、新しいポリシーはそのテーブル内のパーティションにも適用されます。ただし、テーブルがポリシーを指定せずに作成された場合にデータベースからポリシーを継承します。データベースからポリシーを継承するテーブルが一度存在すると、データベースのデフォルトポリシーを変更してもそのテーブルには適用されません。

### テーブルに配置ポリシーを指定する

テーブルにデフォルト配置ポリシーを指定できます。例えば：

```sql
CREATE PLACEMENT POLICY five_replicas FOLLOWERS=4;

CREATE TABLE t (a INT) PLACEMENT POLICY=five_replicas;  -- テーブル t を作成し、配置ポリシー 'five_replicas' を添付します。
```
```sql
ALTER TABLE t PLACEMENT POLICY=default; -- テーブルtから配置ポリシー'five_replicas'を削除し、配置ポリシーをデフォルトにリセットします。

### パーティションテーブル用の配置ポリシーを指定する

パーティションテーブルまたはパーティションに配置ポリシーを指定することもできます。たとえば：

```sql
CREATE PLACEMENT POLICY storageforhisotrydata CONSTRAINTS="[+node=history]";
CREATE PLACEMENT POLICY storagefornewdata CONSTRAINTS="[+node=new]";
CREATE PLACEMENT POLICY companystandardpolicy CONSTRAINTS="";

CREATE TABLE t1 (id INT, name VARCHAR(50), purchased DATE)
PLACEMENT POLICY=companystandardpolicy
PARTITION BY RANGE( YEAR(purchased) ) (
  PARTITION p0 VALUES LESS THAN (2000) PLACEMENT POLICY=storageforhisotrydata,
  PARTITION p1 VALUES LESS THAN (2005),
  PARTITION p2 VALUES LESS THAN (2010),
  PARTITION p3 VALUES LESS THAN (2015),
  PARTITION p4 VALUES LESS THAN MAXVALUE PLACEMENT POLICY=storagefornewdata
);
```

テーブル内のパーティションに配置ポリシーが指定されていない場合、パーティションは（あれば）テーブルからポリシーを継承しようとします。前述の例では：

- `p0` パーティションには `storageforhisotrydata` ポリシーが適用されます。
- `p4` パーティションには `storagefornewdata` ポリシーが適用されます。
- `p1`、`p2`、および `p3` パーティションには、テーブル `t1` から継承された `companystandardpolicy` 配置ポリシーが適用されます。
- テーブル `t1` に配置ポリシーが指定されていない場合、`p1`、`p2`、および `p3` パーティションはデータベースデフォルトポリシーまたはグローバルデフォルトポリシーを継承します。

これらのパーティションに配置ポリシーが追加された後、次の例のように特定のパーティションの配置ポリシーを変更できます：

```sql
ALTER TABLE t1 PARTITION p1 PLACEMENT POLICY=storageforhisotrydata;
```

## 高可用性の例

次のトポロジーでクラスターがあると仮定し、各リージョンにTiKVノードが3つずつ配布されているとします。各リージョンには、3つの利用可能なゾーンが含まれています：

```sql
SELECT store_id,address,label from INFORMATION_SCHEMA.TIKV_STORE_STATUS;
+----------+-----------------+--------------------------------------------------------------------------------------------------------------------------+
| store_id | address         | label                                                                                                                    |
+----------+-----------------+--------------------------------------------------------------------------------------------------------------------------+
|        1 | 127.0.0.1:20163 | [{"key": "region", "value": "us-east-1"}, {"key": "zone", "value": "us-east-1a"}, {"key": "host", "value": "host1"}]     |
|        2 | 127.0.0.1:20162 | [{"key": "region", "value": "us-east-1"}, {"key": "zone", "value": "us-east-1b"}, {"key": "host", "value": "host2"}]     |
|        3 | 127.0.0.1:20164 | [{"key": "region", "value": "us-east-1"}, {"key": "zone", "value": "us-east-1c"}, {"key": "host", "value": "host3"}]     |
|        4 | 127.0.0.1:20160 | [{"key": "region", "value": "us-east-2"}, {"key": "zone", "value": "us-east-2a"}, {"key": "host", "value": "host4"}]     |
|        5 | 127.0.0.1:20161 | [{"key": "region", "value": "us-east-2"}, {"key": "zone", "value": "us-east-2b"}, {"key": "host", "value": "host5"}]     |
|        6 | 127.0.0.1:20165 | [{"key": "region", "value": "us-east-2"}, {"key": "zone", "value": "us-east-2c"}, {"key": "host", "value": "host6"}]     |
|        7 | 127.0.0.1:20166 | [{"key": "region", "value": "us-west-1"}, {"key": "zone", "value": "us-west-1a"}, {"key": "host", "value": "host7"}]     |
|        8 | 127.0.0.1:20167 | [{"key": "region", "value": "us-west-1"}, {"key": "zone", "value": "us-west-1b"}, {"key": "host", "value": "host8"}]     |
|        9 | 127.0.0.1:20168 | [{"key": "region", "value": "us-west-1"}, {"key": "zone", "value": "us-west-1c"}, {"key": "host", "value": "host9"}]     |
+----------+-----------------+--------------------------------------------------------------------------------------------------------------------------+
```

### サバイバル優先度を指定する

データの正確な分布に特に関心がなく、災害復旧要件を優先させたい場合は、データのサバイバル優先度を指定するために `SURVIVAL_PREFERENCES` オプションを使用できます。

前述の例のように、TiDBクラスターが3つのリージョンに分散し、それぞれのリージョンに3つのゾーンが含まれるとします。このクラスターの配置ポリシーを作成する際、次のように `SURVIVAL_PREFERENCES` を構成したと仮定します：

```sql
CREATE PLACEMENT POLICY multiaz SURVIVAL_PREFERENCES="[region, zone, host]";
CREATE PLACEMENT POLICY singleaz CONSTRAINTS="[+region=us-east-1]" SURVIVAL_PREFERENCES="[zone]";
```

配置ポリシーを作成した後は、必要に応じて対応するテーブルにそれらをアタッチできます：

- `multiaz` 配置ポリシーでアタッチされたテーブルの場合、データは異なるリージョンに3つの複製が配置され、データの隔離のクロスリージョンの生存目標を満たすことを優先し、次にクロスゾーンの生存目標を満たし、最後にクロスホストの生存目標を満たします。
- `singleaz` 配置ポリシーでアタッチされたテーブルの場合、データはまず `us-east-1` リージョンに3つの複製が配置され、次にデータの隔離のクロスゾーンの生存目標を満たします。

<CustomContent platform="tidb">

> **注記：**
>
> `SURVIVAL_PREFERENCES` は PD の `location-labels` と同等です。詳細については、[トポロジーラベルによるレプリカのスケジューリング](/schedule-replicas-by-topology-labels.md) を参照してください。

</CustomContent>

<CustomContent platform="tidb-cloud">

> **注記：**
>
> `SURVIVAL_PREFERENCES` は PD の `location-labels` と同等です。詳細については、[トポロジーラベルによるレプリカのスケジューリング](https://docs.pingcap.com/tidb/stable/schedule-replicas-by-topology-labels) を参照してください。

</CustomContent>

### 複数のデータセンターにわたって2:2:1の割合で5つのレプリカが分散されたクラスターを指定する

特定のデータ分布（たとえば、比率2:2:1の5つのレプリカ分布）が必要な場合は、`CONSTRAINTS` を構成して異なる制約の数を指定することで、異なるレプリカ数を指定できます。[ディクショナリ形式](#constraints-formats)で次のように構成できます：

```sql
CREATE PLACEMENT POLICY `deploy221` CONSTRAINTS='{"+region=us-east-1":2, "+region=us-east-2": 2, "+region=us-west-1": 1}';

ALTER RANGE global PLACEMENT POLICY = "deploy221";

SHOW PLACEMENT;
+-------------------+---------------------------------------------------------------------------------------------+------------------+
| Target            | Placement                                                                                   | Scheduling_State |
+-------------------+---------------------------------------------------------------------------------------------+------------------+
| POLICY deploy221  | CONSTRAINTS="{\"+region=us-east-1\":2, \"+region=us-east-2\": 2, \"+region=us-west-1\": 1}" | NULL             |
| RANGE TiDB_GLOBAL | CONSTRAINTS="{\"+region=us-east-1\":2, \"+region=us-east-2\": 2, \"+region=us-west-1\": 1}" | SCHEDULED        |
+-------------------+---------------------------------------------------------------------------------------------+------------------+
```

グローバルな `deploy221` 配置ポリシーがクラスターに設定された後、TiDBはこのポリシーに従ってデータを配置します：`us-east-1` リージョンに2つの複製が配置され、`us-east-2` リージョンに2つの複製が配置され、`us-west-1` リージョンに1つの複製が配置されます。

### LeadersとFollowersの分布を指定する

制約または `PRIMARY_REGION` を使用して、LeadersとFollowersの特定の分布を指定できます。

#### 制約を使用する

特定のRaft Leadersのノード間での分布に特定の要件がある場合は、次のステートメントを使用して配置ポリシーを指定できます：

```sql
CREATE PLACEMENT POLICY deploy221_primary_east1 LEADER_CONSTRAINTS="[+region=us-east-1]" FOLLOWER_CONSTRAINTS='{"+region=us-east-1": 1, "+region=us-east-2": 2, "+region=us-west-1: 1}';
```
```
トリプルクォートで囲まれた最初の行はソースドキュメントで、その後に日本語に翻訳された文書が続きます。

```
      + {R}
      + {R}
    + {R}
  + {R}
```
```
      + {T}
      + {T}
    + {T}
  + {T}
```