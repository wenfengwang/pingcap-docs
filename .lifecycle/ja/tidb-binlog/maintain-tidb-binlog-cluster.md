---
title: TiDBビンログクラスターの操作
summary: TiDBビンログのクラスターバージョンの操作方法について学びます。
aliases: ['/docs/dev/tidb-binlog/maintain-tidb-binlog-cluster/','/docs/dev/reference/tidb-binlog/maintain/','/docs/dev/how-to/maintain/tidb-binlog/','/docs/dev/reference/tools/tidb-binlog/maintain/']
---

# TiDBビンログクラスターの操作

このドキュメントでは、以下のTiDBビンログクラスターの操作について紹介します：

+ PumpおよびDrainerノードの状態
+ PumpやDrainerプロセスの起動または終了
+ binlogctlツールを使用するか、TiDBでSQL操作を直接実行して、TiDBビンログクラスターを管理する方法

## PumpまたはDrainerの状態

PumpまたはDrainerの状態の説明：

* `online`: 正常に実行中
* `pausing`: 一時停止中
* `paused`: 停止済み
* `closing`: オフライン処理中
* `offline`: オフライン

> **注意:**
>
> PumpまたはDrainerノードの状態情報は、サービス自体によって維持され、定期的に配置ドライバー（PD）に更新されます。

## PumpまたはDrainerプロセスの起動および終了

### Pump

* 起動: Pumpノードは、`online`状態のすべてのDrainerノードに通知します。通知に成功すると、Pumpノードは自分の状態を`online`に設定します。それ以外の場合、Pumpノードはエラーを報告し、自分の状態を`paused`に設定してプロセスを終了します。
* 終了: プロセスが正常に終了する前にPumpノードは`paused`または`offline`状態に入ります。プロセスが異常終了する場合（`kill -9`コマンドによるもの、プロセスのパニック、クラッシュ）、ノードは引き続き`online`状態のままです。
    * 一時停止: `kill`コマンド（`kill -9`ではない）、<kbd>Ctrl</kbd>+<kbd>C</kbd>の押下、またはbinlogctlツールの`pause-pump`コマンドを使用して、Pumpプロセスを一時停止できます。一時停止指示を受信した後、Pumpノードは状態を`pausing`に設定し、ビンログの書き込みリクエストの受信を停止し、Drainerノードにビンログデータを提供することを停止します。すべてのスレッドが安全に終了した後、Pumpノードは状態を`paused`に更新し、プロセスを終了します。
    * オフライン: Pumpプロセスは、binlogctlツールの`offline-pump`コマンドを使用してのみ終了できます。オフライン指示を受信した後、Pumpノードは状態を`closing`に設定し、ビンログの書き込みリクエストの受信を停止します。PumpノードはDrainerノードにビンログを提供し続けますが、すべてのビンログデータがDrainerノードによって消費されるまでです。その後、Pumpノードは状態を`offline`に設定し、プロセスを終了します。

### Drainer

* 起動: Drainerノードは起動すると、自分の状態を`online`に設定し、`offline`状態でないすべてのPumpノードからビンログを引き出しようとします。ビンログを取得できない場合、継続的に試行します。
* 終了: プロセスが正常に終了する前にDrainerノードは`paused`または`offline`状態に入ります。プロセスが異常終了する場合（`kill -9`、プロセスのパニック、クラッシュによるもの）、ノードは引き続き`online`状態のままです。
    * 一時停止: `kill`コマンド（`kill -9`ではない）、<kbd>Ctrl</kbd>+<kbd>C</kbd>の押下、またはbinlogctlツールの`pause-drainer`コマンドを使用して、Drainerプロセスを一時停止できます。一時停止指示を受信した後、Drainerノードは状態を`pausing`に設定し、Pumpノードからのビンログの引き出しを停止します。すべてのスレッドが安全に終了した後、Drainerノードは状態を`paused`に更新し、プロセスを終了します。
    * オフライン: Drainerプロセスはbinlogctlツールの`offline-drainer`コマンドを使用してのみ終了できます。オフライン指示を受信した後、Drainerノードは状態を`closing`に設定し、Pumpノードからのビンログの引き出しを停止します。すべてのスレッドが安全に終了した後、Drainerノードは状態を`offline`に更新し、プロセスを終了します。

Drainerの一時停止、終了、状態の確認、変更方法についての詳細は、[binlogctlガイド](/tidb-binlog/binlog-control.md)を参照してください。

## Pump/Drainerの管理に`binlogctl`を使用

[`binlogctl`](https://github.com/pingcap/tidb-binlog/tree/master/binlogctl)は、TiDBビンログの操作ツールであり、次の機能を提供します：

* PumpまたはDrainerの状態を確認する
* PumpまたはDrainerを一時停止または終了する
* PumpまたはDrainerの異常状態を処理する

`binlogctl`の詳細な使用方法については、[binlogctl概要](/tidb-binlog/binlog-control.md)を参照してください。

## SQLステートメントを使用してPumpまたはDrainerを管理する

ビンログ関連の状態を表示または変更するには、TiDBで対応するSQLステートメントを実行します。

- ビンログが有効かどうかを確認する：

    {{< copyable "sql" >}}

    ```sql
    show variables like "log_bin";
    ```

    ```
    +---------------+-------+
    | Variable_name | Value |
    +---------------+-------+
    | log_bin       |  0    |
    +---------------+-------+
    ```
    
    Valueが`0`の場合、ビンログは有効です。Valueが`1`の場合、ビンログは無効です。

- すべてのPumpまたはDrainerノードの状態を確認する：

    {{< copyable "sql" >}}

    ```sql
    show pump status;
    ```

    ```
    +--------|----------------|--------|--------------------|---------------------|
    | NodeID |     Address    | State  |   Max_Commit_Ts    |    Update_Time      |
    +--------|----------------|--------|--------------------|---------------------|
    | pump1  | 127.0.0.1:8250 | Online | 408553768673342237 | 2019-05-01 00:00:01 |
    +--------|----------------|--------|--------------------|---------------------|
    | pump2  | 127.0.0.2:8250 | Online | 408553768673342335 | 2019-05-01 00:00:02 |
    +--------|----------------|--------|--------------------|---------------------|
    ```

    {{< copyable "sql" >}}

    ```sql
    show drainer status;
    ```

    ```
    +----------|----------------|--------|--------------------|---------------------|
    |  NodeID  |     Address    | State  |   Max_Commit_Ts    |    Update_Time      |
    +----------|----------------|--------|--------------------|---------------------|
    | drainer1 | 127.0.0.3:8249 | Online | 408553768673342532 | 2019-05-01 00:00:03 |
    +----------|----------------|--------|--------------------|---------------------|
    | drainer2 | 127.0.0.4:8249 | Online | 408553768673345531 | 2019-05-01 00:00:04 |
    +----------|----------------|--------|--------------------|---------------------|
    ```

- 異常状態のPumpまたはDrainerノードの状態を変更する

    {{< copyable "sql" >}}

    ```sql
    change pump to node_state ='paused' for node_id 'pump1';
    ```

    ```
    クエリは正常に完了しました。影響を受ける行: 0 (0.01 sec)
    ```

    {{< copyable "sql" >}}

    ```sql
    change drainer to node_state ='paused' for node_id 'drainer1';
    ```

    ```
    クエリは正常に完了しました。影響を受ける行: 0 (0.01 sec)
    ```

    上記のSQLステートメントを実行すると、`binlogctl`の`update-pump`または`update-drainer`コマンドと同じように機能します。PumpまたはDrainerノードが異常状態にある場合にのみ、上記のSQLステートメントを使用してください。

> **注意:**
>
> - ビンログが有効かどうかおよびPumpまたはDrainerの実行状態を確認することは、TiDB v2.1.7およびそれ以降のバージョンでサポートされています。
> - PumpまたはDrainerの状態を変更することは、TiDB v3.0.0-rc.1およびそれ以降のバージョンでサポートされています。この機能では、PDに保存されているPumpまたはDrainerノードの状態を変更することのみをサポートしています。ノードを一時停止または終了する場合は、`binlogctl`ツールを使用してください。