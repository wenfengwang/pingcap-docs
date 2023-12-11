---
title: TiDB バイナリログ エラー処理
summary: TiDB バイナリログのエラー処理方法を学びます。
aliases: ['/docs/dev/tidb-binlog/handle-tidb-binlog-errors/','/docs/dev/reference/tidb-binlog/troubleshoot/error-handling/']
---

# TiDB バイナリログ エラー処理

この文書では、TiDB バイナリログの使用時に遭遇する可能性のある一般的なエラーとその解決策について説明します。

## Drainer が Kafka にデータを複製する際に `kafka server: Message was too large, server rejected it to avoid allocation error` が返される

原因: TiDB で大規模なトランザクションを実行すると、Kafka のメッセージサイズの制限を超える可能性があります。

解決策: 以下に示すように、Kafka の構成パラメータを調整します:

{{< copyable "" >}}

```
message.max.bytes=1073741824
replica.fetch.max.bytes=1073741824
fetch.message.max.bytes=1073741824
```

## Pump が `no space left on device` エラーを返す

原因: Pump がバイナリログデータを正常に書き込むための十分なローカルディスク容量がないためです。

解決策: ディスク容量を片付けてから Pump を再起動します。

## Pump を起動すると `fail to notify all living drainer` が返される

原因: Pump を起動すると、`online` 状態のすべての Drainer ノードに通知します。Drainer への通知に失敗するとこのエラーログが出力されます。

解決策: [binlogctl ツール](/tidb-binlog/binlog-control.md) を使用して、各 Drainer ノードが正常かどうかを確認します。`online` 状態のすべての Drainer ノードが正常に動作していることを保証するためのものです。Drainer ノードの状態が実際の動作状況と一致していない場合は、binlogctl ツールを使用して状態を変更し、その後 Pump を再起動します。

## TiDB バイナリログ複製中にデータが失われる

TiDB インスタンス全体で TiDB バイナリログが有効になり、正常に実行されていることを確認する必要があります。クラスタのバージョンが v3.0 より新しい場合は、すべての TiDB インスタンスで `curl {TiDB_IP}:{STATUS_PORT}/info/all` コマンドを使用して TiDB バイナリログの状態を確認します。

## 上流トランザクションが大きい場合、Pump で `rpc error: code = ResourceExhausted desc = trying to send message larger than max (2191430008 vs. 2147483647)` エラーが報告される

このエラーは、TiDB が Pump に送信する gRPC メッセージがサイズ制限を超えたため発生します。Pump を起動する際に `max-message-size` を指定することで、Pump が許可する gRPC メッセージの最大サイズを調整できます。

## Drainer が出力するファイル形式の増分データの掃除機構はありますか？データは削除されますか？

- Drainer v3.0.x では、ファイル形式の増分データの掃除機構はありません。
- v4.0.x バージョンでは、時間ベースのデータクリーニング機構があります。詳細は、[Drainer の `retention-time` 構成項目](https://github.com/pingcap/tidb-binlog/blob/v4.0.9/cmd/drainer/drainer.toml#L153) を参照してください。