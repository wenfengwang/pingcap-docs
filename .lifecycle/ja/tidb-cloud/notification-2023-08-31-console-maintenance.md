---
title: 2023-08-31 TiDBクラウドコンソールメンテナンス通知
summary: 2023年8月31日のTiDBクラウドコンソールメンテナンスの詳細について、メンテナンスウィンドウ、理由、および影響について学びます。

# [2023-08-31] TiDBクラウドコンソールメンテナンス通知

この通知では、2023年8月31日の[TiDB Cloud console](https://tidbcloud.com/)メンテナンスについて知っておく必要がある詳細について説明します。

## メンテナンスウィンドウ

- 日付: 2023年08月31日
- 開始時間: 8:00 (UTC+0)
- 終了時間: 10:00 (UTC+0)
- 所要時間: 約2時間

> **注意:**
>
> 現在、ユーザーはTiDBクラウドコンソールのメンテナンスタイミングを変更することはできませんので、事前に計画を立てる必要があります。

## メンテナンス理由

TiDBクラウドコンソールのメタデータサービスをアップグレードして、パフォーマンスと効率を向上させています。この改善は、高品質のサービスを提供するための取り組みの一環として、すべてのユーザーにより良いエクスペリエンスを提供することを目的としています。

## 影響

メンテナンスウィンドウ中に、TiDBクラウドコンソールUIおよびAPIに関連する機能において一時的な障害が発生する可能性があります。ただし、TiDBクラスターはデータの読み書きに関して通常通りの運用を維持し、オンラインビジネスには悪影響を及ぼさないようにしています。

### TiDBクラウドコンソールUIの影響を受ける機能

- クラスターレベル
    - クラスター管理
        - クラスターの作成
        - クラスターの削除
        - クラスターのスケール
        - クラスターの一時停止または再開
        - クラスターパスワードの変更
        - クラスタートラフィックフィルターの変更
    - インポート
        - インポートジョブの作成
    - データ移行
        - 移行ジョブの作成
    - Changefeed
        - Changefeedジョブの作成
    - バックアップ
        - マニュアルバックアップジョブの作成
        - 自動バックアップジョブ
    - リストア
        - リストアジョブの作成
    - データベース監査ログ
        - 接続テスト
        - アクセスレコードの追加または削除
        - データベース監査ログの有効化または無効化
        - データベース監査ログの再起動
- プロジェクトレベル
    - ネットワークアクセス
        - プライベートエンドポイントの作成
        - プライベートエンドポイントの削除
        - VPC Peeringの追加
        - VPC Peeringの削除
    - メンテナンス
        - メンテナンスウィンドウの変更
        - タスクの延期
    - リサイクルビン
        - クラスターの削除
        - バックアップの削除
        - クラスターのリストア

### TiDBクラウドAPIの影響を受ける機能

- クラスター管理
    - [CreateCluster](https://docs.pingcap.com/tidbcloud/api/v1beta#tag/Cluster/operation/CreateCluster)
    - [DeleteCluster](https://docs.pingcap.com/tidbcloud/api/v1beta#tag/Cluster/operation/DeleteCluster)
    - [UpdateCluster](https://docs.pingcap.com/tidbcloud/api/v1beta#tag/Cluster/operation/UpdateCluster)
    - [CreateAwsCmek](https://docs.pingcap.com/tidbcloud/api/v1beta#tag/Cluster/operation/CreateAwsCmek)
- バックアップ
    - [CreateBackup](https://docs.pingcap.com/tidbcloud/api/v1beta#tag/Backup/operation/CreateBackup)
    - [DeleteBackup](https://docs.pingcap.com/tidbcloud/api/v1beta#tag/Backup/operation/DeleteBackup)
- リストア
    - [CreateRestoreTask](https://docs.pingcap.com/tidbcloud/api/v1beta#tag/Restore/operation/CreateRestoreTask)
- インポート
    - [CreateImportTask](https://docs.pingcap.com/tidbcloud/api/v1beta#tag/Import/operation/CreateImportTask)
    - [UpdateImportTask](https://docs.pingcap.com/tidbcloud/api/v1beta#tag/Import/operation/UpdateImportTask)

## 完了と再開

メンテナンスが正常に完了すると、影響を受ける機能が再開され、さらなる良好なエクスペリエンスが提供されます。

## サポートを受ける

ご質問やサポートが必要な場合は、[サポートチーム](/tidb-cloud/tidb-cloud-support.md)にお問い合わせください。お客様のご要望にお応えし、必要なガイダンスを提供するためにこちらにいます。