---
title: オンラインセーフリカバリ
summary: オンラインセーフリカバリの使用方法を学ぶ。

# オンラインセーフリカバリ

> **警告:**
>
> オンラインセーフリカバリは損失が発生する場合があります。この機能を使用すると、データやデータインデックスの整合性が保証されなくなる可能性があります。

TiKV上で永続的に損傷した複製により、データの一部が読み取れなくなり書き込みできなくなった場合、オンラインセーフリカバリ機能を使用して損失が発生するリカバリ操作を実行できます。

## 機能の説明

TiDBでは、ユーザーが定義したレプリカルールに従って、同じデータが同時に複数のストアに格納されることがあります。これにより、単一または数少ないストアが一時的にオフラインまたは損傷しても、データが読み取り可能で書き込み可能であることが保証されます。ただし、一定期間にわたってほとんどまたはすべてのリージョンのレプリカがオフラインになると、そのリージョンは一時的に利用できず、読み取りや書き込みができなくなります。

あるデータ範囲の複数のレプリカが永続的な損傷（ディスク損傷など）のような問題に遭遇し、これらの問題がストアをオフラインのままにしている場合、このデータ範囲は一時的に利用できなくなります。クラスタを再利用し、データの巻き戻しやデータ損失を受け入れ、理論的には、失敗したレプリカを手動でグループから削除し、再度多数のレプリカを形成することで、（ステールまたは空の）このデータ範囲をアプリケーション層のサービスによって再び読み取り書き込みできるようにできます。

この場合、損失を許容するデータを持つストアが永続的に損傷している場合、オンラインセーフリカバリを使用することで損失が発生するリカバリ操作を実行できます。この機能を使用すると、PDは自動的にリージョンのスケジューリング（分割および統合を含む）を一時停止し、すべてのストアからデータシャードのメタデータを収集し、その後、グローバルな視点でリアルタイムかつ完全なリカバリプランを生成します。その後、PDはプランをすべての生存しているストアに配布して、それらにデータリカバリタスクを実行させます。また、データリカバリプランが配布されると、PDは定期的にリカバリの進捗を監視し、必要に応じてプランを再送します。

## ユーザーシナリオ

オンラインセーフリカバリ機能は、次のシナリオに適しています:

* 永続的な損傷のため、アプリケーションサービスのデータが読み取れず書き込めなくなっている。
* データ損失を受け入れ、影響を受けたデータが読み取り可能で書き込み可能であることが必要。
* 一括のオンラインデータリカバリ操作を実行したい。

## 使用方法

### 前提条件

オンラインセーフリカバリを使用する前に、次の要件を満たしていることを確認してください:

* オフラインのストアは実際に一部のデータを利用できない状態にしている。
* オフラインのストアは自動的に回復または再起動されない。

### ステップ1. 回復できないストアを指定する

自動リカバリをトリガーするには、PD Controlを使用して[`unsafe remove-failed-stores <store_id>[,<store_id>,...]`](/pd-control.md#unsafe-remove-failed-stores-store-ids--show)を実行し、回復できない**すべて**のTiKVノードをカンマで区切って指定してください。

{{< copyable "shell-regular" >}}

```bash
pd-ctl -u <pd_addr> unsafe remove-failed-stores <store_id1,store_id2,...>
```

コマンドが`Success`を返すと、PDコントロールはタスクをPDに正常に登録しました。これはリクエストが受け付けられたことを意味するため、リカバリは成功したことを意味するものではありません。リカバリタスクはバックグラウンドで実行されます。リカバリの進捗状況を確認するには、[`show`](#ステップ2-リカバリ進捗を確認し完了を待つ)を使用してください。

コマンドが`Failed`を返すと、PD ControlはPDにタスクを登録できませんでした。可能なエラーは次のとおりです:

- `unsafe recovery is running`: すでに実行中のリカバリタスクがあります。
- `invalid input store x doesn't exist`: 指定したストアIDが存在しません。
- `invalid input store x is up and connected`: 指定されたストアIDはまだ正常であり、回復すべきではありません。

復旧タスクの最長許容時間を指定するには、`--timeout <seconds>`オプションを使用してください。このオプションが指定されていない場合、デフォルトで最長の継続時間は5分です。タイムアウトが発生すると、リカバリが中断され、エラーが返されます。

> **注意:**
>
> - このコマンドはすべてのピアから情報を収集する必要があるため、メモリ使用量が増加することがあります（推定100,000ピアで500 MiBのメモリを使用すると見積もられます）。
> - コマンドが実行中にPDが再起動すると、リカバリが中断され、タスクを再トリガする必要があります。
> - コマンドが実行中の場合、指定されたストアはTombstoneステータスに設定され、これらのストアを再起動できません。
> - コマンドが実行中の場合、すべてのスケジューリングタスクや分割/統合は一時停止され、リカバリが正常または失敗した後に自動的に再開されます。

### ステップ2. リカバリ進捗を確認し完了を待つ

上記のストア削除コマンドが正常に実行されると、PD Controlを使用して[`unsafe remove-failed-stores show`](/pd-control.md#config-show--set-option-value--placement-rules)を実行することで、削除の進捗状況を確認できます。

{{< copyable "shell-regular" >}}

```bash
pd-ctl -u <pd_addr> unsafe remove-failed-stores show
```

リカバリプロセスには複数の段階があります:

- `collect report`: PDがTiKVからレポートを収集し、グローバル情報を取得する初期段階。
- `tombstone tiflash learner`: 非健康なリージョンの中で、他の健全なピアよりも新しいTiFlashラーナを削除して、その極端な状況と潜在的なパニックを防ぐ。
- `force leader for commit merge`: 特別な段階です。未完了のマージを強制的にリーダーにさせることで、マージを実行していないリージョンに対して実行します。
- `force leader`: 非健康なリージョンに対して残りの健全なピアの間でRaftリーダーを指定します。
- `demote failed voter`: リージョンの失敗した投票者を学習者に降格し、その後リージョンは通常通りRaftリーダーを選択できます。
- `create empty region`: キー範囲のスペースを埋めるために、空のリージョンを作成します。一部のリージョンのすべてのレプリカを持つストアが損傷した場合に解決するためです。

上記の各段階は、情報、時間、および詳細なリカバリプランを含むJSON形式で出力されます。例:

```json
[
    {
        "info": "Unsafe recovery enters collect report stage: failed stores 4, 5, 6",
        "time": "......"
    },
    {
        "info": "Unsafe recovery enters force leader stage",
        "time": "......",
        "actions": {
            "store 1": [
                "force leader on regions: 1001, 1002"
            ],
            "store 2": [
                "force leader on regions: 1003"
            ]
        }
    },
    {
        "info": "Unsafe recovery enters demote failed voter stage",
        "time": "......",
        "actions": {
            "store 1": [
                "region 1001 demotes peers { id:101 store_id:4 }, { id:102 store_id:5 }",
                "region 1002 demotes peers { id:103 store_id:5 }, { id:104 store_id:6 }",
            ],
            "store 2": [
                "region 1003 demotes peers { id:105 store_id:4 }, { id:106 store_id:6 }",
            ]
        }
    },
    {
        "info": "Collecting reports from alive stores(1/3)",
        "time": "......",
        "details": [
            "Stores that have not dispatched plan: ",
            "Stores that have reported to PD: 4",
            "Stores that have not reported to PD: 5, 6",
        ]
    }
]
```

PDがリカバリプランを正常に配布すると、TiKVが実行結果を報告するのを待ちます。上記の出力の最後の段階である`alive storesからのレポートの収集`で分かるように、この部分の出力にはPDがリカバリプランを配布し、TiKVからのレポートを受け取る詳細なステータスが表示されます。

リカバリ全体のプロセスは複数の段階を経ており、1つの段階が複数回再試行される場合があります。通常、推定所要時間は3から10回のストアハートビート（ストアハートビートの1回の期間はデフォルトで10秒です）です。リカバリが完了すると、コマンド出力の最後の段階には`"Unsafe recovery finished"`、影響を受けたリージョンが属するテーブルID（ない場合やRawKVが使用される場合、出力にテーブルIDが表示されません）、および影響を受けたSQLメタリージョンが表示されます。例:

```json
{
    "info": "Unsafe recovery finished",
    "time": "......",
    "details": [
        "Affected table ids: 64, 27",
        "Affected meta regions: 1001",
    ]
}
```

影響を受けたテーブルIDを取得した後は、`INFORMATION_SCHEMA.TABLES`をクエリして影響を受けたテーブル名を表示できます。

```sql
SELECT TABLE_SCHEMA, TABLE_NAME, TIDB_TABLE_ID FROM INFORMATION_SCHEMA.TABLES WHERE TIDB_TABLE_ID IN (64, 27);
```

> **注意:**
>
> - リカバリ操作により、いくつかの失敗した投票者が失敗した学習者に変更されます。その後、PDスケジューリングはこれらの失敗した学習者を削除するために時間がかかります。
> - できるだけ早く新しいストアを追加することをお勧めします。

タスク中にエラーが発生した場合、出力の最後の段階に`"Unsafe recovery failed"`とエラーメッセージが表示されます。例:

```json
{
    "info": "Unsafe recovery failed: <error>",
    "time": "......"
}
```

### ステップ3. データとインデックスの整合性を確認する（RawKVの場合は必要ありません）

> **注意:**
>
> データは読み書きできるが、データが失われないという意味ではありません。

回復が完了すると、データとインデックスが不整合になる可能性があります。影響を受けるテーブルのデータとインデックスの整合性をチェックするには、SQLコマンド[`ADMIN CHECK`](/sql-statements/sql-statement-admin-check-table-index.md)を使用してください。

```sql
ADMIN CHECK TABLE テーブル名;
```

インデックスが不整合なら、古いインデックスの名前を変更し、新しいインデックスを作成し、古いインデックスを削除することで、インデックスの不整合を修正できます。

1. 古いインデックスの名前を変更します：

    ```sql
    ALTER TABLE テーブル名 RENAME INDEX インデックス名 TO インデックス名_lame_duck;
    ```

2. 新しいインデックスを作成します：

    ```sql
    ALTER TABLE テーブル名 ADD INDEX インデックス名 (カラム名);
    ```

3. 古いインデックスを削除します：

    ```sql
    ALTER TABLE テーブル名 DROP INDEX インデックス名_lame_duck;
    ```

### ステップ4: 回復不能なストアの削除（オプション）

<SimpleTab>
<div label="TiUPを使用してデプロイされたストア">

1. 回復不能なノードを削除します：

    ```bash
    tiup cluster scale-in <クラスター名> -N <ホスト> --force
    ```

2. Tombstoneノードをクリーンアップします：

    ```bash
    tiup cluster prune <クラスター名>
    ```

</div>
<div label="TiDB Operatorを使用してデプロイされたストア">

1. `PersistentVolumeClaim`を削除します。

    {{< copyable "shell-regular" >}}

    ```bash
    kubectl delete -n ${namespace} pvc ${pvc_name} --wait=false
    ```

2. TiKV Podを削除し、新しく作成されたTiKV Podsがクラスターに参加するのを待ちます。

    {{< copyable "shell-regular" >}}

    ```bash
    kubectl delete -n ${namespace} pod ${pod_name}
    ```

</div>
</SimpleTab>