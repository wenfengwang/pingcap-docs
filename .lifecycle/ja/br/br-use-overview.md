---
title: TiDB バックアップとリストアの使用概要
summary: バックアップとリストアツールの展開方法と、TiDB クラスタのバックアップとリストアの使用方法について学びます。
aliases: ['/tidb/dev/br-deployment/']
---

# TiDB バックアップとリストアの使用概要

この文書では、TiDB バックアップとリストア機能のベストプラクティスについて説明します。 これには、バックアップ方法の選択、バックアップデータの管理方法、およびバックアップとリストアツールのインストールと展開方法が含まれます。

## 推奨される手法

TiDB バックアップとリストア機能を使用する前に、推奨されるバックアップとリストアのソリューションを理解することをお勧めします。

### データのバックアップ方法

**TiDB は 2 種類のバックアップを提供しています。どちらを使用すべきですか？** フルバックアップには、クラスタの完全なデータが特定の時点で含まれます。ログバックアップには、TiDB に書き込まれたデータ変更が含まれます。推奨されるのは、両方の種類のバックアップを同時に使用することです。

- **[ログバックアップの開始](/br/br-pitr-guide.md#start-log-backup)**: `br log start` コマンドを実行して、ログバックアップタスクを開始します。その後、タスクがすべての TiKV ノードで定期的に小さなバッチで TiDB データの変更を指定されたストレージにバックアップします。
- 定期的に [スナップショット（完全）バックアップ](/br/br-snapshot-guide.md#back-up-cluster-snapshots) を実行します: `br backup full` コマンドを実行して、クラスタのスナップショットを指定されたストレージにバックアップします。例えば、毎日午前0時にクラスタのスナップショットをバックアップします。

### バックアップデータの管理方法

BR は基本的なバックアップとリストア機能のみを提供し、バックアップ管理をサポートしていません。したがって、バックアップデータの管理方法を自分で決定する必要があります。以下の質問が関わってくる可能性があります。

* どのバックアップストレージシステムを選択すべきか？
* バックアップタスク中にバックアップデータを配置するディレクトリはどこですか？
* 完全バックアップデータとログバックアップデータのディレクトリをどのように整理すべきか？
* ストレージシステム内の過去のバックアップデータをどのように扱うべきか？

以下のセクションでは、これらの質問について順番に回答します。

**バックアップストレージシステムの選択**

Amazon S3、Google Cloud Storage（GCS）、またはAzure Blob Storage にバックアップデータを格納することが推奨されています。これらのシステムを使用すると、バックアップ容量と帯域幅の割り当てについて心配する必要はありません。

TiDB クラスタが自己構築のデータセンターに展開されている場合、以下の手法が推奨されます：

* バックアップストレージシステムとして [MinIO](https://docs.min.io/docs/minio-quickstart-guide.html) を構築し、S3 プロトコルを使用して MinIO にデータをバックアップします。
* Network File System（NFS、NASなど）ディスクを br コマンドラインツールおよびすべての TiKV インスタンスにマウントし、対応する NFS ディレクトリにバックアップデータを書き込むために POSIX ファイルシステムインターフェースを使用します。

> **注意:**
>
> NFS またはAmazon S3、GCS、Azure Blob Storage プロトコルをサポートするストレージシステムを選択しない場合、バックアップされたデータが各 TiKV ノードで生成されます。**これは BR の推奨される使用方法ではない**ため、バックアップデータの収集がデータの冗長性や運用および保守の問題を引き起こす可能性があります。

**バックアップデータディレクトリの整理**

* スナップショットバックアップとログバックアップを統合管理するために、同じディレクトリにスナップショットバックアップとログバックアップを格納します。例：`backup-${cluster-id}`。
* 各スナップショットバックアップをバックアップ日を含むディレクトリに格納します。例：`backup-${cluster-id}/fullbackup-202209081330`。
* ログバックアップは固定ディレクトリに格納します。例：`backup-${cluster-id}/logbackup`。ログバックアッププログラムは、毎日新しいサブディレクトリを`logbackup`ディレクトリの下に作成して、当日にバックアップされたデータを区別します。

**過去のバックアップデータの処理**

各バックアップデータにライフサイクル（例：7日）を設定する必要があると仮定します。このようなライフサイクルは**バックアップ保持期間**と呼ばれ、バックアップチュートリアルにも記載されます。

* PITR を実行するためには、リストアポイント前の完全バックアップおよび完全バックアップとリストアポイント間のログバックアップをリストアする必要があります。したがって、**完全なスナップショットの前にログバックアップのみを削除することをお勧めします**。バックアップ保持期間を超えるログバックアップについては、`br log truncate` コマンドを使用して指定された時刻より前のバックアップを削除できます。
* 保持期間を超えるバックアップデータについては、バックアップディレクトリを削除またはアーカイブにできます。

### データのリストア方法

- 完全バックアップデータのみをリストアする場合は、指定されたバックアップの完全なリストアを実行するために `br restore` を使用できます。
- ログバックアップを開始し、定期的に完全バックアップを実行している場合、`br restore point` コマンドを実行して、バックアップ保持期間内の任意の時点にデータをリストアできます。

## BR の展開と使用

BR を展開する前に、以下の要件を満たしていることを確認してください：

- BR、TiKV ノード、およびバックアップストレージシステムがバックアップ速度よりも大きなネットワーク帯域幅を提供します。ターゲットクラスタが特に大きい場合、バックアップおよびリストア速度の閾値はバックアップネットワークの帯域幅によって制限されます。
- バックアップストレージシステムが十分な読み書きパフォーマンス（IOPS）を提供します。そうでないと、バックアップまたはリストア中にパフォーマンスボトルネックになる可能性があります。
- TiKV ノードには、バックアップ用に少なくとも 2 つの追加 CPU コアと高性能ディスクが必要です。そうでないと、クラスタで実行されているサービスに影響を与える可能性があります。
- BR は、8 つ以上のコアと 16 GiB 以上のメモリを搭載したノードで実行する必要があります。

コマンドラインツール、SQL コマンドの実行、および TiDB Operator を使用するなど、複数の方法でバックアップとリストア機能を使用できます。以下のセクションでは、これらの 3 つの方法について詳しく説明します。

### br コマンドラインツールを使用する（推奨）

TiDB は、br コマンドラインツールを使用してバックアップとリストアをサポートしています。

* `tiup install br` コマンドを実行して、[TiUP オンラインを使用して br コマンドラインツールをインストール](/migration-tools.md#install-tools-using-tiup)します。
* バックアップとリストアデータのバックアップとリストアデータの使用方法の詳細については、次の文書を参照してください：

    * [TiDB スナップショットバックアップとリストアガイド](/br/br-snapshot-guide.md)
    * [TiDB ログバックアップと PITR ガイド](/br/br-pitr-guide.md)
    * [TiDB バックアップとリストアのユースケース](/br/backup-and-restore-use-cases.md)

### SQL ステートメントを使用する

TiDB は、SQL ステートメントを使用して完全なバックアップとリストアをサポートしています。

- [`BACKUP`](/sql-statements/sql-statement-backup.md): 完全なスナップショットデータをバックアップします。
- [`RESTORE`](/sql-statements/sql-statement-restore.md): スナップショットバックアップデータをリストアします。
- [`SHOW BACKUPS|RESTORES`](/sql-statements/sql-statement-show-backups.md): バックアップとリストアの進行状況を表示します。

### Kubernetes 上の TiDB Operator を使用する

Kubernetes 上では、TiDB Operator を使用して、TiDB クラスタデータをAmazon S3、GCS、またはAzure Blob Storageにバックアップし、そのようなシステムでのバックアップデータからデータをリストアできます。詳細については、[TiDB Operator を使用したデータのバックアップとリストア](https://docs.pingcap.com/tidb-in-kubernetes/stable/backup-restore-overview)を参照してください。

## 関連項目

- [TiDB バックアップとリストアの概要](/br/backup-and-restore-overview.md)
- [TiDB バックアップとリストアのアーキテクチャ](/br/backup-and-restore-design.md)