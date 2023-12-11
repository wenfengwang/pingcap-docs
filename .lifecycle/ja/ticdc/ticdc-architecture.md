---
title: TiCDCのアーキテクチャと原則
summary: TiCDCのアーキテクチャと動作原則を学びます。

# TiCDCのアーキテクチャと原則

## TiCDCのアーキテクチャ

TiCDCクラスタは複数のTiCDCノードで構成され、分散型で状態を持たないアーキテクチャを使用します。TiCDCおよびそのコンポーネントの設計は次のようになっています。

![TiCDCのアーキテクチャ](/media/ticdc/ticdc-architecture-1.jpg)

## TiCDCのコンポーネント

上記の図では、TiCDCクラスタは複数のノードでTiCDCインスタンスを実行しています。各TiCDCインスタンスにはCaptureプロセスがあります。Captureプロセスのうち1つがオーナーCaptureとして選出され、ワークロードのスケジューリング、DDLステートメントの複製、管理タスクを担当します。

各Captureプロセスには、上流のTiDBのテーブルからデータを複製するための1つまたは複数のProcessorスレッドが含まれています。TiCDCではテーブルがデータ複製の最小単位であるため、Processorは複数のテーブルパイプラインで構成されています。

各パイプラインには次のコンポーネントが含まれています: Puller、Sorter、Mounter、Sink。

![TiCDCのアーキテクチャ](/media/ticdc/ticdc-architecture-2.jpg)

これらのコンポーネントはお互いに直列に作動し、データ複製プロセス（データの取得、整列、読み込み、上流から下流へのデータ複製）を完了します。これらのコンポーネントは次のように説明されています。

- Puller: TiKVノードからDDLおよび行の変更を取得します。
- Sorter: TiKVノードから受信した変更をタイムスタンプの昇順で整列します。
- Mounter: スキーマ情報に基づいて変更をTiCDCシンクが処理できる形式に変換します。
- Sink: 変更を下流システムに複製します。

高可用性を実現するために、各TiCDCクラスタは複数のTiCDCノードで実行されます。これらのノードは定期的にPD内のetcdクラスタに状態を報告し、ノードの1つをTiCDCクラスタの所有者として選出します。所有者ノードはetcdに保存された状態に基づいてデータをスケジュールし、スケジュール結果をetcdに書き込みます。Processorはetcd内の状態に従ってタスクを完了します。もしProcessorを実行しているノードが失敗した場合、クラスタは他のノードにテーブルをスケジュールします。所有者ノードが失敗した場合、他のノードのCaptureプロセスは新しい所有者を選出します。以下の図を参照してください。

![TiCDCのアーキテクチャ](/media/ticdc/ticdc-architecture-3.PNG)

## Changefeedとタスク

TiCDCのChangefeedとタスクは2つの論理的な概念です。具体的な説明は次のとおりです。

- Changefeed: レプリケーションタスクを表します。複製するテーブルの情報と下流に関する情報を含みます。
- Task: TiCDCがレプリケーションタスクを受信すると、このタスクを複数のサブタスクに分割します。このようなサブタスクをタスクと呼びます。これらのタスクはTiCDCノードのCaptureプロセスによって処理されます。

例：

```
cdc cli changefeed create --server="http://127.0.0.1:8300" --sink-uri="kafka://127.0.0.1:9092/cdc-test?kafka-version=2.4.0&partition-num=6&max-message-bytes=67108864&replication-factor=1"
cat changefeed.toml
......
[sink]
dispatchers = [
    {matcher = ['test1.tab1', 'test2.tab2'], topic = "{schema}_{table}"},
    {matcher = ['test3.tab3', 'test4.tab4'], topic = "{schema}_{table}"},
]
```

前述の`cdc cli changefeed create`コマンドの詳細なパラメータについては、[TiCDC Changefeed Configuration Parameters](/ticdc/ticdc-changefeed-config.md) を参照してください。

前述の`cdc cli changefeed create`コマンドは、`test1.tab1`、`test1.tab2`、`test3.tab3`、および`test4.tab4`をKafkaクラスタに複製するchangefeedタスクを作成します。TiCDCがこのコマンドを受信した後の処理フローは次のとおりです。

1. TiCDCはこのタスクを所有者Captureプロセスに送信します。
2. 所有者Captureプロセスはこのchangefeedタスクに関する情報をPD内のetcdに保存します。
3. 所有者Captureプロセスはchangefeedタスクを複数のタスクに分割し、他のCaptureプロセスに完了するタスクを通知します。
4. CaptureプロセスはTiKVノードからデータを取得し、データを処理して複製を完了します。

以下は、ChangefeedとTaskが含まれるTiCDCのアーキテクチャ図です。

![TiCDCのアーキテクチャ](/media/ticdc/ticdc-architecture-6.jpg)

上記の図では、Changefeedが4つのテーブルを下流に複製するように作成されています。このChangefeedは3つのTaskに分かれ、それぞれがTiCDCクラスタ内の3つのCaptureプロセスに送信されます。TiCDCがデータを処理した後、データは下流システムに複製されます。

TiCDCは、データをMySQL、TiDB、およびKafkaデータベースに複製できます。前述の図はChangefeedレベルでのデータ転送プロセスのみを示しています。次のセクションでは、表`table1`を複製するTask1を使用して、TiCDCがデータをどのように処理するかについて詳細に説明します。

![TiCDCのアーキテクチャ](/media/ticdc/ticdc-architecture-5.jpg)

1. データプッシュ：データが変更されると、TiKVはデータをPullerモジュールにプッシュします。
2. 増分データスキャン：Pullerモジュールは、受信したデータの変更が連続していない場合にTiKVからデータを取得します。
3. データ整列：SorterモジュールはTiKVから受信したデータをタイムスタンプに基づいて整列し、整列されたデータをMounterモジュールに送信します。
4. データロード：Mounterモジュールはデータ変更を受信した後、TiCDCシンクが理解できる形式でデータをロードします。
5. データ複製：Sinkモジュールはデータ変更を下流に複製します。

TiCDCの上流はトランザクションをサポートする分散関係データベースであるTiDBです。TiCDCがデータを複製する際には、データの一貫性と複数のテーブルを複製する際のトランザクションの一貫性を確保する必要がありますが、これは大きな課題です。次のセクションでは、TiCDCがこの課題に対処するために使用する主要な技術と概念について紹介します。

## TiCDCの主要な概念

下流のリレーショナルデータベースでは、TiCDCは単一の表でのトランザクションの一貫性と複数の表での最終的なトランザクションの一貫性を確保します。さらに、TiCDCは上流のTiDBクラスタで発生したすべてのデータ変更を少なくとも1回下流に複製することを保証します。

### アーキテクチャに関連する概念

- Capture: TiCDCノードを実行するプロセスです。複数のCaptureプロセスがTiCDCクラスタを構成します。各Captureプロセスは、TiKV内のデータ変更の複製、データの受信およびアクティブなデータのプル、下流へのデータ複製を担当します。
- Capture Owner: 複数のCaptureプロセスのうちのオーナーCaptureです。TiCDCクラスタ内には1つだけ所有者の役割が存在します。Capture Ownerはクラスタ内でのデータのスケジューリングを担当します。
- Processor: Capture内の論理的なスレッドです。各Processorは同じレプリケーションストリーム内の1つまたは複数のテーブルのデータの処理を担当します。キャプチャノードで複数のProcessorを実行できます。
- Changefeed: 上流のTiDBクラスタから下流システムにデータを複製するタスクです。Changefeedには複数のタスクが含まれ、各タスクはCaptureノードによって処理されます。

### タイムスタンプに関連する概念

TiCDCはデータ複製の状態を示す一連のタイムスタンプ（TS）を導入しています。これらのタイムスタンプはデータが下流に少なくとも1回複製され、データの一貫性が保証されるように使用されます。

#### ResolvedTS

このタイムスタンプはTiKVとTiCDCの両方に存在します。

- TiKV内のResolvedTS: リージョンリーダー内の最も古いトランザクションの開始時間を表します。つまり、`ResolvedTS` = max(`ResolvedTS`, min(`StartTS`))です。TiDBクラスタには複数のTiKVノードが含まれているため、すべてのTiKVノード上の全リージョンリーダーの最小ResolvedTSはグローバルResolvedTSと呼ばれます。TiDBクラスタは、グローバルResolvedTSより前に行われたすべてのトランザクションを確定させます。それ以前の未確定のトランザクションがないと仮定しても構いません。

- TiCDC内のResolvedTS:

    - テーブルのResolvedTS: 各テーブルには、テーブルレベルのResolvedTSがあり、このタイムスタンプは、そのテーブルに対応するすべてのリージョンの最小ResolvedTSと同じです。
    - グローバルResolvedTS: すべてのTiCDCノードのすべてのProcessorの最小ResolvedTSです。各TiCDCノードには1つ以上のProcessorがあり、各Processorは複数のテーブルパイプラインに対応しています。

    TiKVによって送信されるResolvedTSは、`<resolvedTS: timestamp>`の形式の特別なイベントです。通常、ResolvedTSは次の制約を満たします。

    ```
    table ResolvedTS >= global ResolvedTS
    ```

#### CheckpointTS

このタイムスタンプはTiCDCのみに存在します。これは、このタイムスタンプより前に発生したデータ変更が下流システムに複製されていることを示します。

- テーブルのCheckpointTS: TiCDCはテーブルごとにデータを複製するため、テーブルCheckpointTSはテーブルレベルで、このタイムスタンプより前のすべてのデータ変更が複製されたことを示します。
- ProcessorのCheckpointTS: Processor上の最小のテーブルCheckpointTSを示します。
- グローバルCheckpointTS: すべてのProcessorの中で最小のCheckpointTSを示します。

一般的には、CheckpointTSは次の制約を満たします。

```
table CheckpointTS >= global CheckpointTS
```

TiCDCは下流にグローバルResolvedTSよりも小さいデータのみを複製するため、完全な制約は次のとおりです。

```
```
table ResolvedTS >= global ResolvedTS >= table CheckpointTS >= global CheckpointTS
```

データの変更とトランザクションがコミットされると、TiKVノードのResolvedTSは引き続き進行し、TiCDCノードのPullerモジュールはTiKVからプッシュされたデータを受信し続けます。Pullerモジュールは受信したデータの変更に基づいて増分データをスキャンするかどうかも判断します。これにより、すべてのデータ変更がTiCDCノードに送信されることが保証されます。

Sorterモジュールは、Pullerモジュールが受信したデータをタイムスタンプ順に昇順でソートします。このプロセスにより、テーブルレベルでデータの整合性が確保されます。次に、Mounterモジュールは、上流からのデータ変更をSinkモジュールが消費できる形式に組み立て、Sinkモジュールに送信します。Sinkモジュールは、CheckpointTSとResolvedTS間のデータ変更をタイムスタンプ順に下流に複製し、下流がデータ変更を受信した後にcheckpointTSを進めます。

前述のセクションでは、DMLステートメントのデータ変更のみがカバーされており、DDLステートメントは含まれていません。次のセクションでは、DDLステートメントに関連するタイムスタンプについて説明します。

#### バリアTS

バリアTSは、DDL変更イベントまたはSyncpointが使用された場合に生成されます。

- DDL変更イベント：バリアTSは、DDLステートメントの前のすべての変更が下流に複製されることを保証します。このDDLステートメントが実行および複製された後、TiCDCは他のデータ変更の複製を開始します。DDLステートメントはCapture Ownerによって処理されるため、DDLステートメントに対応するバリアTSは所有者ノードだけによって生成されます。
- Syncpoint：TiCDCのSyncpoint機能を有効にすると、指定した`sync-point-interval`に従ってTiCDCによってバリアTSが生成されます。このバリアTSより前のすべてのテーブル変更が複製されると、TiCDCは現在のglobal CheckpointTSを、下流のtsMapに記録するテーブルの主要なTSとして挿入します。その後、TiCDCはデータ複製を継続します。

バリアTSが生成されると、TiCDCはこのバリアTSより前に発生したデータ変更のみが下流に複製されることを保証します。これらのデータ変更が下流に複製される前には、複製タスクは進みません。所有者TiCDCは、global CheckpointTSとBarrier TSを継続的に比較することで、すべての対象データが複製されたかどうかを確認します。global CheckpointTSがBarrier TSと等しい場合、TiCDCは指定された操作（DDLステートメントの実行や、global CheckpointTSの下流への記録など）を行った後に複製を継続します。それ以外の場合は、バリアTSより前に発生したすべてのデータ変更が下流に複製されるのを待機します。

## 主要プロセス

本セクションでは、TiCDCの主要プロセスについて説明し、その動作原理を理解するのに役立ちます。

次のプロセスはTiCDC内でのみ実行され、ユーザーには透過的です。そのため、TiCDCノードのどこで起動しているかは気にする必要はありません。

### TiCDCの開始

- オーナーではないTiCDCノードの場合、次のように機能します：

    1. Captureプロセスを開始します。
    2. Processorを開始します。
    3. オーナーによって実行されたタスクスケジューリングコマンドを受信します。
    4. スケジューリングコマンドに従ってtablePipelineを開始または停止します。

- オーナーTiCDCノードの場合、次のように機能します：

    1. Captureプロセスを開始します。
    2. ノードがオーナーに選出され、対応するスレッドが開始されます。
    3. changefeed情報を読み込みます。
    4. changefeed管理プロセスを開始します。
    5. changefeed構成および最新のCheckpointTSに基づいて、TiKVのスキーマ情報を読み込み、複製するテーブルを決定します。
    6. 各Processorが現在複製しているテーブルのリストを読み込み、追加するテーブルを配布します。
    7. 複製進捗を更新します。

### TiCDCの停止

通常、TiCDCノードを停止する必要がある場合は、アップグレードを行ったり、いくつかの計画されたメンテナンス操作を実行したりします。TiCDCノードを停止するプロセスは次のとおりです：

1. ノードは自身を停止するコマンドを受け取ります。
2. ノードはサービスステータスを利用できない状態に設定します。
3. ノードは新しい複製タスクを受け取らなくなります。
4. ノードはオーナーノードに通知し、自身のデータ複製タスクを他のノードに移管します。
5. ノードは複製タスクが他のノードに移管された後に停止します。
```