---
title: DM における継続的なデータ検証
summary: 継続的なデータ検証の使用方法とその動作原理について学ぼう。

# DM における継続的なデータ検証

このドキュメントでは、DM における継続的なデータ検証の使用方法、動作原理、および制限について説明します。

## ユーザシナリオ

上流データベースから下流データベースにデータを段階的に移行する過程で、データの流れがデータの破損やデータの損失につながる可能性が極めて低い場合があります。クレジット業界や証券業界など、データの整合性が要求されるシナリオでは、移行後にデータの整合性を確認するためにフルデータ検証を実行できます。

しかし、段階的な移行のシナリオでは、上流と下流が常にデータを書き込んでいます。上流と下流のデータが常に変化しているため、表内のすべてのデータをフルデータ検証（たとえば [sync-diff-inspector](/sync-diff-inspector/sync-diff-inspector-overview.md) を使用）することは困難です。

段階的な移行のシナリオでは、DM における継続的なデータ検証機能を使用できます。この機能は、下流にデータが継続的に書き込まれる増分移行中にデータの整合性と一貫性を保証します。

## 継続的なデータ検証を有効にする

継続的なデータ検証を有効にするには、次のいずれかの方法を使用できます。

- タスク構成ファイルに有効にする。
- dmctl を使用して有効にする。

### 方法 1: タスク構成ファイルに有効にする

継続的なデータ検証を有効にするには、タスク構成ファイルに次の構成項目を追加します:

```yaml
# 検証が必要な上流データベースに次の構成項目を追加します:
mysql-instances:
  - source-id: "mysql1"
    block-allow-list: "bw-rule-1"
    validator-config-name: "global"
validators:
  global:
    mode: full # "fast" も許可されます。"none" は検証が実行されないデフォルトのモードです。
    worker-count: 4 # バックグラウンドでの検証ワーカーの数。デフォルト値は 4 です。
    row-error-delay: 30m # 指定された時間内に行が検証に合格しない場合、エラー行としてマークされます。デフォルト値は 30 分です。
```

構成項目は次のように説明されています:

* `mode`: 検証モード。可能な値は `none`、`full`、`fast` です。
    * `none`: デフォルト値で、検証が実行されません。
    * `full`: 変更された行と下流データベースで取得された行を比較します。
    * `fast`: 変更された行が下流データベースに存在するかどうかだけをチェックします。
* `worker-count`: バックグラウンドでの検証ワーカーの数。各ワーカーはゴールーチンです。
* `row-error-delay`: 指定された時間内に行が検証に合格しない場合、エラー行としてマークされます。デフォルト値は 30 分です。

完全な構成については、[DM アドバンスドタスク構成ファイル](/dm/task-configuration-file-full.md) を参照してください。

### 方法 2: dmctl を使用して有効にする

継続的なデータ検証を有効にするには、`dmctl validation start` コマンドを実行します:

```
Usage:
  dmctl validation start [--all-task] [task-name] [flags]

Flags:
      --all-task            すべてのタスクに適用するかどうか
  -h, --help                start のヘルプ
      --mode string         検証モードを指定します: full (デフォルト)、fast; このフラグは、検証タスクが有効になっているが現在一時停止している場合には無視されます (デフォルト "full")
      --start-time string   検証のためのバイナリログの開始時刻を指定します。例: '2021-10-21 00:01:00' または 2021-10-21T00:01:00
```

* `--mode`: 検証モードを指定します。可能な値は `fast` と `full` です。
* `--start-time`: 検証の開始時刻を指定します。形式は `2021-10-21 00:01:00` または `2021-10-21T00:01:00` に従います。
* `task`: 継続的な検証を有効にするタスクの名前を指定します。`--all-task` を使用してすべてのタスクに対して検証を有効にできます。

例:

```shell
dmctl --master-addr=127.0.0.1:8261 validation start --start-time 2021-10-21T00:01:00 --mode full my_dm_task
```

## 継続的なデータ検証の使用

継続的なデータ検証を使用する際には、dmctl を使用して検証の状態を表示し、エラー行を処理することができます。"エラー行" とは、上流と下流のデータ間で整合性がないと判断された行です。

### 検証の状態を表示する

検証の状態は次のいずれかの方法で表示できます:

方法 1: `dmctl query-status <task-name>` コマンドを実行します。継続的なデータ検証が有効になっている場合、各サブタスクの `validation` フィールドに検証結果が表示されます。例:

```json
"subTaskStatus": [
    {
        "name": "test",
        "stage": "Running",
        "unit": "Sync",
        "result": null,
        "unresolvedDDLLockID": "",
        "sync": {
            ...
        },
        "validation": {
            "task": "test", // タスク名
            "source": "mysql-01", // ソース ID
            "mode": "full", // 検証モード
            "stage": "Running", // 現在のステージ。"Running" または "Stopped".
            "validatorBinlog": "(mysql-bin.000001, 5989)", // 検証のバイナリログ位置
            "validatorBinlogGtid": "1642618e-cf65-11ec-9e3d-0242ac110002:1-30", // 検証の GTID 位置
            "cutoverBinlogPos": "", // 切り替えの指定されたバイナリログ位置
            "cutoverBinlogGTID": "1642618e-cf65-11ec-9e3d-0242ac110002:1-30", // 切り替えの指定された GTID 位置
            "result": null, // 検証が異常な場合、エラーメッセージを表示します
            "processedRowsStatus": "insert/update/delete: 0/0/0", // 処理済みバイナリログ行の統計
            "pendingRowsStatus": "insert/update/delete: 0/0/0", // まだ検証されていないか、検証に合格できなかったバイナリログ行の統計
            "errorRowsStatus": "new/ignored/resolved: 0/0/0" // エラー行の統計。次のセクションで 3 つのステータスが説明されています
        }
    }
]
```

方法 2: `dmctl validation status <taskname>` コマンドを実行します。

```
dmctl validation status [--table-stage stage] <task-name> [flags]
Flags:
  -h, --help                 status のヘルプ
      --table-stage string   ステージでテーブルをフィルタリング: running/stopped
```

上記のコマンドでは、`--table-stage` を使用して検証中のテーブルをフィルタリングしたり、検証を停止したりできます。例:

```json
{
    "result": true,
    "msg": "",
    "validators": [
        {
            "task": "test",
            "source": "mysql-01",
            "mode": "full",
            "stage": "Running",
            "validatorBinlog": "(mysql-bin.000001, 6571)",
            "validatorBinlogGtid": "",
            "cutoverBinlogPos": "(mysql-bin.000001, 6571)",
            "cutoverBinlogGTID": "",
            "result": null,
            "processedRowsStatus": "insert/update/delete: 2/0/0",
            "pendingRowsStatus": "insert/update/delete: 0/0/0",
            "errorRowsStatus": "new/ignored/resolved: 0/0/0"
        }
    ],
    "tableStatuses": [
        {
            "source": "mysql-01", // ソース ID
            "srcTable": "`db`.`test1`", // エラーメッセージ
            "dstTable": "`db`.`test1`", // 対象テーブル名
            "stage": "Running", // 検証ステータス
            "message": "" // エラーメッセージ
        }
    ]
}
```

エラー行の詳細（エラータイプやエラー発生時刻など）を表示するには、`dmctl validation show-error` コマンドを実行します:

```
Usage:
  dmctl validation show-error [--error error-state] <task-name> [flags]

Flags:
      --error string   エラーの種類をフィルタリング: all、ignored、または unprocessed (デフォルト "unprocessed")
  -h, --help           show-error のヘルプ
```

例:

```json
{
    "result": true,
    "msg": "",
    "error": [
        {
            "id": "1", // エラー行の ID。エラー行の処理に使用されます
            "source": "mysql-replica-01", // ソース ID
            "srcTable": "`validator_basic`.`test`", // エラー行のソーステーブル
            "srcData": "[0, 0]", // ソーステーブルのエラー行のデータ
            "dstTable": "`validator_basic`.`test`", // エラー行の対象テーブル
            "dstData": "[]" // 対象テーブルのエラー行のデータ
```json
[
    {
        "errorType": "Expected rows not exist",
        "status": "NewErr",
        "time": "2022-07-04 13:33:02",
        "message": ""
    }
]
```

### エラー行の処理

連続的なデータ検証でエラー行が返された場合は、エラー行を手動で処理する必要があります。

連続的なデータ検証はエラー行を見つけた際に、即座に停止しません。代わりに、エラー行を記録し、後で処理するために保存します。エラー行が処理される前には、デフォルトのステータスは「unprocessed」となります。もし、下流でエラー行を手動で修正した場合、検証は自動的に修正されたデータの最新のステータスを取得しません。エラー行は依然として「error」フィールドに記録されます。

もし、検証ステータスでエラー行を見たくない場合、またはエラー行を解決済みとしてマークしたい場合は、`validation show-error` コマンドを使用してエラー行のIDを特定し、その後与えられたエラーIDで処理することができます。

dmctl には、次の3つのエラー処理コマンドが用意されています：

- `clear-error`: エラー行をクリアします。「show-error」コマンドでエラー行が表示されなくなります。

    ```
    使用法：
      dmctl validation clear-error <task-name> <error-id|--all> [flags]

    フラグ：
          --all    すべてのエラー
      -h, --help   clear-errorのヘルプ
    ```

- `ignore-error`: エラー行を無視します。このエラー行は「ignored」としてマークされます。

    ```
    使用法：
      dmctl validation ignore-error <task-name> <error-id|--all> [flags]

    フラグ：
          --all    すべてのエラー
      -h, --help   ignore-errorのヘルプ
    ```

- `resolve-error`: エラー行は手動で処理され、「resolved」としてマークされます。

    ```
    使用法：
      dmctl validation resolve-error <task-name> <error-id|--all> [flags]

    フラグ：
          --all    すべてのエラー
      -h, --help   resolve-errorのヘルプ
    ```

## 連続的なデータ検証を停止する

連続的なデータ検証を停止するには、`validation stop` コマンドを実行してください：

```
使用法：
  dmctl validation stop [--all-task] [task-name] [flags]

フラグ：
      --all-task   すべてのタスクに適用するかどうか
  -h, --help       stopのヘルプ
```

詳細な使用法については、[`dmctl validation start`](#method-2-enable-using-dmctl)を参照してください。

## 連続的なデータ検証のカットオーバーポイントを設定する

アプリケーションを別のデータベースに切り替える前に、特定のポジションにデータがレプリケートされた直後に、データの整合性を確保するために、すぐに連続的なデータ検証を実行する必要があるかもしれません。このような場合、連続検証のためのカットオーバーポイントとして特定のポジションを設定することができます。

連続的なデータ検証のためのカットオーバーポイントを設定するには、`validation update` コマンドを使用してください：

```
使用法：
  dmctl validation update <task-name> [flags]

フラグ：
      --cutover-binlog-gtid string   検証のためのカットオーバーbinlog gtidを指定します。upstream クラスターで GTID が有効な場合にのみ有効です。例：'1642618e-cf65-11ec-9e3d-0242ac110002:1-30'
      --cutover-binlog-pos string    検証のためのカットオーバーbinlog 名を指定します。binlog 名と pos を括弧で含める必要があります。例：'(mysql-bin.000001, 5989)'
  -h, --help                         updateのヘルプ
```

* `--cutover-binlog-gtid`：検証のためのカットオーバー位置を、`1642618e-cf65-11ec-9e3d-0242ac110002:1-30` の形式で指定します。上流クラスターで GTID が有効な場合にのみ有効です。
* `--cutover-binlog-pos`：検証のためのカットオーバー位置を、`(mysql-bin.000001, 5989)` の形式で指定します。
* `task-name`：連続的なデータ検証のタスク名です。このパラメータは**必須**です。

## 実装

DMにおける連続的なデータ検証（バリデータ）のアーキテクチャは以下の通りです:

![バリデータサマリー](/media/dm/dm-validator-summary.jpeg)

連続的なデータ検証のライフサイクルは以下の通りです:

![バリデータライフサイクル](/media/dm/dm-validator-lifecycle.jpeg)

連続的なデータ検証の詳細な実装は以下の通りです：

1. バリデータは上流から binlog イベントを引き出し、変更された行を取得します：
     - バリデータはシンカーによって増分的に移行されたイベントのみをチェックします。もしイベントがシンカーによって処理されていない場合、バリデータは一時停止し、シンカーが完了するのを待ちます。
     - イベントがシンカーによって処理された場合、バリデータは次のステップに移ります。
2. バリデータは binlog イベントを解析し、ブロックおよび許可リスト、テーブルフィルタ、およびテーブルルーティングに基づいて行をフィルタリングします。その後、バリデータは変更された行をバックグラウンドで実行される検証ワーカーに提出します。
3. 検証ワーカーは、同じテーブルおよび同じ主キーに影韓する変更された行をマージして、「期限切れ」データの検証を避けます。変更された行はメモリにキャッシュされます。
4. 検証ワーカーが一定数の変更された行を蓄積した場合、または一定の時間経過した場合、検証ワーカーは主キーを使用して下流データベースをクエリし、現在のデータを取得し、変更された行と比較します。
5. 検証ワーカーはデータ検証を実行します。もし検証モードが `full` の場合、検証ワーカーは変更された行のデータを下流データベースのデータと比較します。もし検証モードが `fast` の場合、検証ワーカーは変更された行の存在のみをチェックします。
    - もし変更された行が検証に合格した場合、その行はメモリから削除されます。
    - もし変更された行が検証に合格しない場合、バリデータは即座にエラーを報告せず、行を再度検証する前に一定の時間を待ちます。
    - 特定の時間内に変更された行が検証に合格しない場合（ユーザーによって指定された時間）、バリデータはその行をエラー行としてマークし、その情報をダウンストリームのメタデータベースに書き込みます。エラー行の情報は、マイグレーションタスクをクエリすることで確認できます。詳細については [検証ステータスの表示](#view-the-validation-status) および [エラー行の処理](#handle-error-rows) を参照してください。

## 制限事項

- 検証するソーステーブルには、プライマリキーや NOT NULL のユニークキーが必要です。
- DMが上流データベースから DDL を移行する場合、以下の制限が適用されます:
    - DDL はプライマリキーを変更したり、列の順序を変更したり、既存の列を削除してはなりません。
    - テーブルをドロップしてはいけません。
- イベントをフィルタするために式を使用するタスクはサポートされていません。
- 浮動小数点数の精度は、TiDB と MySQL の間で異なります。10^-6 よりも小さい差異は等しいと見なされます。
- 次のデータ型はサポートされていません:
    - JSON
    - バイナリデータ