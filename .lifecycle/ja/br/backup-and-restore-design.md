---
title: TiDB Backup & Restore Architecture Overview
summary: TiDBのバックアップとリストア機能のアーキテクチャ設計について学びます。

# TiDB Backup & Restore Architecture Overview

[TiDB Backup & Restore Overview](/br/backup-and-restore-overview.md) に記載されているように、TiDBは複数のタイプのクラスタデータのバックアップとリストアをサポートしています。これらの機能には、Backup & Restore (BR) および TiDB Operator を使用し、TiKV ノードからデータをバックアップしたり、TiKV ノードにデータをリストアするタスクを作成することができます。

各バックアップおよびリストア機能のアーキテクチャの詳細については、次のドキュメントを参照してください。

- フルデータのバックアップとリストア

    - [スナップショットデータのバックアップ](/br/br-snapshot-architecture.md#process-of-backup)
    - [スナップショットバックアップデータのリストア](/br/br-snapshot-architecture.md#process-of-restore)

- データ変更ログのバックアップ

    - [ログバックアップ: KVデータ変更のバックアップ](/br/br-log-architecture.md#process-of-log-backup)

- 時間指定のリカバリ（PITR）

    - [PITR](/br/br-log-architecture.md#process-of-pitr)