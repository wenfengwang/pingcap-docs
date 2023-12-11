---
title: データ移行におけるDMLレプリケーションメカニズム
summary: DMのコア処理ユニットであるSyncがDMLステートメントをレプリケートするメカニズム
---

# データ移行におけるDMLレプリケーションメカニズム

この文書は、DM内のコア処理ユニットであるSyncが、データソースまたはリレーログから読み取ったDMLステートメントを処理する仕組みを紹介します。DMにおけるDMLイベントの完全な処理フローについて詳しく説明し、binlogの読み取り、フィルタリング、ルーティング、変換、最適化、および実行のロジックを含む内容です。また、DMLの最適化ロジックと実行ロジックについても詳細に説明します。

## DML処理フロー

Syncユニットは、以下のようにDMLステートメントを処理します:

1. MySQL、MariaDB、またはリレーログからbinlogイベントを読み取ります。
2. データソースから読み取ったbinlogイベントを変換します:

    1. [Binlogフィルタ](/dm/dm-binlog-event-filter.md): `filters`で設定されたbinlogの式に従ってbinlogイベントをフィルタリングします。
    2. [テーブルルーティング](/dm/dm-table-routing.md): "database/table"ルーティングルールに従って "database/table" 名を変換します。`routes` で設定されます。
    3. [Expression filter](/filter-dml-event.md): `expression-filter` で設定されたSQLの式に従ってbinlogイベントをフィルタリングします。

3. DML実行プランを最適化します:

    1. [Compactor](#compactor): 同じレコード（同じプライマリキーを持つ）に対する複数の操作を1つの操作にマージします。`syncer.compact`によって有効になります。
    2. [Causality](#causality): 異なるレコード（異なるプライマリキーを持つ）の競合を検出し、レプリケーションの並列性を向上させます。
    3. [Merger](#merger): 複数のbinlogイベントを1つのDMLステートメントにマージし、`syncer.multiple-rows`によって有効になります。

4. DMLを下流で実行します。
5. 定期的にbinlogの位置またはGTIDをチェックポイントに保存します。

![DML処理ロジック](/media/dm/dm-dml-replication-logic.png)

## DML最適化ロジック

Syncユニットは、Compactor、Causality、そしてMergerという3つのステップでDML最適化ロジックを実装しています。

### Compactor

上流のbinlogレコードに基づき、DMはレコードの変更を収集し、それを下流にレプリケートします。上流で同じレコードに対して短時間に複数の変更がある場合、Compactorによってこれらを1つの変更に圧縮し、下流の負荷を減らしスループットを向上させることができます。例:

```
INSERT + UPDATE => INSERT
INSERT + DELETE => DELETE
UPDATE + UPDATE => UPDATE
UPDATE + DELETE => DELETE
DELETE + INSERT => UPDATE
```

Compactor機能はデフォルトで無効です。有効にするには、レプリケーションタスクの`sync`構成モジュールで `syncer.compact` を `true` に設定します:

```yaml
syncers:                            # Sync処理ユニットの構成パラメータ
  global:                           # 構成名
    ...                              # 他の構成は省略
    compact: true
```

### Causality

MySQL binlogの順次レプリケーションモデルでは、binlogイベントがbinlogの順にレプリケートされる必要があります。このレプリケーションモデルは、高いQPSと低いレプリケーション遅延の要件を満たすことができません。また、binlogに関与するすべての操作が競合しているわけではないため、順次レプリケーションはそれらの場合に必要ありません。

DMは、競合検出を通じて順次に実行する必要のあるbinlogを認識し、それらのbinlogを順次実行しつつ、他のbinlogの並列性を最大限に確保します。これにより、binlogのパフォーマンスが向上します。

Causalityは、各DMLを分類し、互いに関連するDMLをグループ化するために、Union-Findアルゴリズムに似たアルゴリズムを採用しています。

### Merger

MySQL binlogプロトコルによると、各binlogは1行のデータの変更操作に対応しています。Mergerにより、DMは複数のbinlogを1つのDMLにマージし、これを下流で実行します。これによりネットワークの相互作用を減らすことができます。例:

```
  INSERT tb(a,b) VALUES(1,1);
+ INSERT tb(a,b) VALUES(2,2);
= INSERT tb(a,b) VALUES(1,1),(2,2);
  UPDATE tb SET a=1, b=1 WHERE a=1;
+ UPDATE tb SET a=2, b=2 WHERE a=2;
= INSERT tb(a,b) VALUES(1,1),(2,2) ON DUPLICATE UPDATE a=VALUES(a), b=VALUES(b)
  DELETE tb WHERE a=1
+ DELETE tb WHERE a=2
= DELETE tb WHERE (a) IN (1),(2);
```

Merger機能はデフォルトで無効です。有効にするには、レプリケーションタスクの`sync`構成モジュールで `syncer.multiple-rows` を `true` に設定します:

```yaml
syncers:                            # Sync処理ユニットの構成パラメータ
  global:                           # 構成名
    ...                              # 他の構成は省略
    multiple-rows: true
```

## DML実行ロジック

SyncユニットがDMLを最適化した後、実行ロジックを実行します。

### DML生成

DMは、上流と下流のスキーマ情報を記録する組み込みスキーマトラッカーを持っています:

* DMがDDLステートメントを受信すると、DMは内部スキーマトラッカーのテーブルスキーマを更新します。
* DMがDMLステートメントを受信すると、DMはスキーマトラッカーのテーブルスキーマに従って対応するDMLを生成します。

DMLの生成ロジックは次のとおりです:

1. Syncユニットは上流の初期テーブル構造を記録します:
    * フルデータ移行の上流タスクまたはインクリメンタルタスクを開始する際、Syncは **上流のフルデータ移行中にエクスポートされたテーブル構造** を上流の初期テーブル構造として使用します。
    * インクリメンタルタスクを開始する際、MySQL binlogはテーブル構造情報を記録しないため、Syncは上流の初期テーブル構造として **下流の対応する表のテーブル構造** を使用します。
2. ユーザーの上流と下流のテーブル構造が一致しないことがあります。たとえば、下流には上流よりも追加のカラムがあったり、上流と下流のプライマリキーが一致しない場合があります。したがって、データのレプリケーションの正確さを確保するために、DMは **下流の対応するテーブルのプライマリキーおよびユニークキー情報を記録** します。
3. DMはDMLを生成します:
    * スキーマトラッカーに記録された **上流のテーブル構造** を使用して、DMLステートメントの列名を生成します。
    * binlogに記録された **列の値** を使用して、DMLステートメントの列値を生成します。
    * スキーマトラッカーに記録された **下流のプライマリキーまたはユニークキー** を使用して、DMLステートメントの `WHERE` 条件を生成します。テーブル構造にユニークキーがない場合、DMは `WHERE` 条件としてbinlogに記録されたすべての列の値を使用します。

### Worker count

Causalityは、競合検出によりbinlogを複数のグループに分割し、それらを下流で並行して実行できるようにします。DMは `worker-count` を設定することで並列性を制御します。下流のTiDBのCPU使用率が高くない場合、並列性を増やすことでデータのレプリケーションのスループットを効果的に向上させることができます。

[`syncer.worker-count` 構成項目](/dm/dm-tune-configuration.md#worker-count)を変更することで、DMLの並列移行スレッドの数を調整できます。

### Batch

DMは複数のDMLを1つのトランザクションにまとめて下流で実行します。DMLワーカーがDMLを受信すると、そのDMLをキャッシュに追加します。キャッシュ内のDMLの数があらかじめ設定されたしきい値に達したり、 DMLワーカーが長時間DMLを受信しなかった場合、DMLワーカーはキャッシュ内のDMLを下流で実行します。

[`syncer.batch` 構成項目](/dm/dm-tune-configuration.md#batch)を変更することで、トランザクションに含まれるDMLの数を調整できます。

### チェックポイント

DMLの実行とチェックポイントの更新はアトミックではありません。

DMでは、デフォルトで30秒ごとにチェックポイントを更新します。複数のDMLワーカープロセスが存在するため、チェックポイントプロセスはすべてのDMLワーカーの最も早いレプリケーション進捗のbinlog位置を計算し、この位置を現在のレプリケーショントチェックポイントとして使用します。この位置よりも前のすべてのbinlogは、下流に正常に実行されることが保証されます。

<!--Checkpointメカニズムの詳細については、Checkpoint /dm/dm-checkpoint.md を参照してください。-->

## メモ

### トランザクションの一貫性

DMは行レベルでデータをレプリケートし、トランザクションの一貫性を保証しません。DMでは、上流のトランザクションが複数の行に分割され、異なるDMLワーカーに分散されて並行的に実行されます。したがって、DMレプリケーションタスクがエラーを報告して一時停止した場合、またはユーザーがタスクを手動で一時停止した場合、下流は中間状態になる可能性があります。つまり、上流トランザクションのDMLステートメントが下流に部分的にレプリケートされる可能性があり、これにより下流が不整合な状態になる可能性があります。

タスクを一時停止する際に下流ができるだけ一貫した状態にするために、DM v5.3.0から、DMはタスクを一時停止する前に上流からのすべてのトランザクションが下流にレプリケートされることを保証するために、10秒間待機します。ただし、10秒以内にトランザクションが下流にレプリケートされない場合、下流はまだ不整合な状態になる可能性があります。

### セーフモード
```
The operation of DML execution and checkpoint update is not atomic, and the operation of checkpoint update and writing data to the downstream is also not atomic. When DM exits abnormally, the checkpoint might only record a recovery point before the exit time. Therefore, when the task is restarted, DM might write the same data multiple times, which means that DM actually provides the "at least once processing" logic, and the same data might be processed more than once.

To make sure the data is reentrant, DM enters the safe mode when it restarts from an abnormal exit. <!--For the specific logic, refer to [DM Safe Mode](/dm/dm-safe-mode.md).-->

When the safe mode is enabled, to make sure that data can be processed multiple times, DM performs the following conversions:

* Rewrite the `INSERT` statement of the upstream to the `REPLACE` statement.
* Rewrite the `UPDATE` statement of the upstream to the `DELETE` + `REPLACE` statement.

### Exactly-once processing

Currently, DM only guarantees eventual consistency and does not support "exactly-once processing" and "keeping the original order of transactions".
```