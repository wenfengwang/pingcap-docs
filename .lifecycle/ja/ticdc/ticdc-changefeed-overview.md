---
title: Changefeedの概要
summary: 基本的な概念、ステートの定義、およびチェンジフィードのステート転送を学ぶ
---

# Changefeedの概要

ChangefeedはTiCDCの複製タスクであり、TiDBクラスター内の指定されたテーブルのデータ変更ログを指定された下流にレプリケートします。TiCDCクラスターで複数のChangefeedを実行および管理できます。

## Changefeedステート転送

複製タスクのステートは、そのタスクの実行状態を表します。TiCDCの実行中には、複製タスクがエラーで失敗したり、手動でポーズしたり、再開したり、指定された`TargetTs`に到達したりすることがあります。これらの振る舞いにより、複製タスクのステートが変わる可能性があります。このセクションでは、TiCDC複製タスクのステートとその間の転送関係について述べます。

![TiCDCステート転送](/media/ticdc/ticdc-changefeed-state-transfer.png)

前述のステート転送図のステートについては次のように説明できます：

- `Normal`: 複製タスクは正常に実行され、チェックポイントTSが正常に進行します。
- `Stopped`: ユーザーがChangefeedを手動で一時停止したため、複製タスクが停止されました。このステートのChangefeedはGC操作をブロックします。
- `Warning`: 複製タスクがエラーを返しました。回復可能なエラーのため、複製が継続できません。このステートのChangefeedは`Normal`にステートが変わるまで継続的に再開を試みます。最大再試行時間は30分です。それを超えると、Changefeedは失敗状態になります。このステートのChangefeedはGC操作をブロックします。
- `Finished`: 複製タスクが完了し、指定された`TargetTs`に到達しました。このステートのChangefeedはGC操作をブロックしません。
- `Failed`: 複製タスクが失敗しました。回復不能なエラーのため、複製タスクが再開できず、回復できません。このステートのChangefeedは失敗を処理するための十分な時間を与えるため、GC操作をブロックします。ブロック期間は`gc-ttl`パラメータで指定され、デフォルト値は24時間です。

> **ノート:**
>
> Changefeedが`ErrGCTTLExceeded`、`ErrSnapshotLostByGC`、または`ErrStartTsBeforeGC`のエラーコードでエラーに遭遇した場合、GC操作をブロックしません。

前述のステート転送図の数字は次のとおりです。

- ① `changefeed pause`コマンドを実行します。
- ② `changefeed resume`コマンドを実行して複製タスクを再開します。
- ③ `changefeed`操作中に回復可能なエラーが発生し、操作が自動的に再試行されます。
- ④ Changefeedの自動再試行が成功し、`checkpoint-ts`が引き続き進行します。
- ⑤ Changefeedの自動再試行が30分を超えて失敗しました。Changefeedは失敗状態になります。この時点でChangefeedは`gc-ttl`で指定された期間にわたり上流のGCをブロックし続けます。
- ⑥ Changefeedが回復不能なエラーに遭遇し、直接失敗状態になりました。この時点でChangefeedは`gc-ttl`で指定された期間にわたり上流のGCをブロックし続けます。
- ⑦ Changefeedの複製進行が`target-ts`で設定された値に達し、複製が完了しました。
- ⑧ Changefeedが`gc-ttl`で指定された値よりも長い期間一時停止されたため、GCの進行エラーに遭遇し、再開できません。

## Changefeedの操作

TiCDCクラスターやその複製タスクは、コマンドラインツール`cdc cli`を使用して管理できます。詳細については、[Manage TiCDC changefeeds](/ticdc/ticdc-manage-changefeed.md)を参照してください。

また、TiCDCクラスターとその複製タスクを管理するには、HTTPインターフェース（TiCDCオープンAPI機能）を使用することもできます。詳細については、[TiCDC OpenAPI](/ticdc/ticdc-open-api.md)を参照してください。

TiCDCをTiUPを使用して展開した場合、`tiup ctl:v<CLUSTER_VERSION> cdc`コマンドを実行して`cdc cli`を起動できます。`v<CLUSTER_VERSION>`をTiCDCクラスターバージョン（例: `v7.4.0`）に置き換えることもできます。また、直接`cdc cli`を実行することもできます。