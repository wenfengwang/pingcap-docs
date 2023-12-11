---
title: SQL式を使用したDMLのフィルタリング
aliases: ['/tidb/dev/feature-expression-filter/']
---

# SQL式を使用したDMLのフィルタリング

## 概要

増分データ移行の過程で、[Filter Binlog Events](/filter-binlog-event.md)機能を使用して特定の種類のBinlogイベントをフィルタリングすることができます。たとえば、アーカイブまたは監査の目的で、データをダウンストリームに移行する際に`DELETE`イベントをフィルタリングすることができます。ただし、Binlogイベントフィルタは`DELETE`イベントの特定の行をフィルタリングすることができません。

上記の問題を解決するために、DMはv2.0.5から`binlog value filter`を使用して増分移行中にデータをフィルタリングすることをサポートしています。 DMがサポートする`ROW`フォーマットのbinlogには、binlogイベントのすべての列の値が含まれています。これらの値に基づいてSQL式を構成することができます。 SQL式が行の変更を`TRUE`と評価する場合、DMは行の変更をダウンストリームに移行しません。

詳細な操作と実装については、[SQL式を使用したDMLイベントのフィルタリング](/filter-dml-event.md)を参照してください。