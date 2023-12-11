---
title: データマイグレーション構成ファイル概要
summary: この文書は、データマイグレーション構成ファイルの概要を示しています。
aliases: ['/docs/tidb-data-migration/dev/config-overview/']
---

# データマイグレーション構成ファイル概要

この文書は、DM（データマイグレーション）の構成ファイルの概要を示しています。

## DM プロセス構成ファイル

- `dm-master.toml`: DM-master プロセスの実行構成ファイルであり、トポロジ情報と DM-master のログを含みます。詳細については、[DM-master 構成ファイル](/dm/dm-master-configuration-file.md) を参照してください。
- `dm-worker.toml`: DM-worker プロセスの実行構成ファイルであり、トポロジ情報と DM-worker のログを含みます。詳細については、[DM-worker 構成ファイル](/dm/dm-worker-configuration-file.md) を参照してください。
- `source.yaml`: MySQL や MariaDB などのアップストリーム データベースの構成を含みます。詳細については、[アップストリーム データベース構成ファイル](/dm/dm-source-configuration-file.md) を参照してください。

## DM マイグレーションタスク構成

### データマイグレーションタスクの作成

次の手順でデータマイグレーションタスクを作成できます。

1. [dmctl を使用して DM クラスタにデータソース構成をロードします](/dm/dm-manage-source.md#operate-data-source)。
2. [タスク構成ガイド](/dm/dm-task-configuration-guide.md) の説明を参照して、構成ファイル `your_task.yaml` を作成します。
3. [dmctl を使用してデータマイグレーションタスクを作成します](/dm/dm-create-task.md)。

### 重要なコンセプト

このセクションでは、いくつかの重要なコンセプトの説明を示します。

| コンセプト  | 説明  | 構成ファイル  |
| :------ | :--------- | :------------- |
| `source-id`  | MySQL や MariaDB のインスタンス、またはプライマリセカンダリ構造を持つマイグレーショングループをユニークに表します。`source-id` の最大長は 32 です。 | `source.yaml` の `source_id`；<br/> `task.yaml` の `source-id` |
| DM-master ID | DM-master（`dm-master.toml` の `master-addr` パラメータにより）をユニークに表します。 | `dm-master.toml` の `master-addr` |
| DM-worker ID | DM-worker（`dm-worker.toml` の `worker-addr` パラメータにより）をユニークに表します。 | `dm-worker.toml` の `worker-addr` |