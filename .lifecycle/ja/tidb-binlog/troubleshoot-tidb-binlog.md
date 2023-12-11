---
title: TiDB ビンログのトラブルシューティング
summary: TiDB ビンログのトラブルシューティングプロセスを学びます。
aliases: ['/docs/dev/tidb-binlog/troubleshoot-tidb-binlog/','/docs/dev/reference/tidb-binlog/troubleshoot/binlog/','/docs/dev/how-to/troubleshoot/tidb-binlog/']
---

# TiDB ビンログのトラブルシューティング

このドキュメントでは、TiDB ビンログの問題を特定するためのトラブルシューティング方法について説明します。

TiDB ビンログの実行中にエラーが発生した場合は、次の手順を実行してトラブルシューティングを行ってください：

1. 各監視メトリクスが正常かどうかを確認してください。詳細については、[TiDB ビンログの監視](/tidb-binlog/monitor-tidb-binlog-cluster.md)を参照してください。

2. [binlogctl ツール](/tidb-binlog/binlog-control.md)を使用して、各 Pump ノードまたは Drainer ノードの状態が正常かどうかを確認してください。

3. Pump ログまたは Drainer ログに `ERROR` または `WARN` が存在するかどうかを確認してください。

上記の手順で問題を特定した後は、[FAQ](/tidb-binlog/tidb-binlog-faq.md)および[TiDB ビンログのエラーハンドリング](/tidb-binlog/handle-tidb-binlog-errors.md)を参照して解決策を見つけてください。解決策が見つからない場合や提供された解決策が役立たない場合は、ヘルプのために[issue](https://github.com/pingcap/tidb-binlog/issues)を提出してください。