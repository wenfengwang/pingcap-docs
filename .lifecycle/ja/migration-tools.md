---
title: TiDB移行ツールの概要
summary: TiDB移行ツールの概要を学ぶ。
---

# TiDB移行ツールの概要

TiDBには、完全なデータ移行、増分データ移行、バックアップとリストア、およびデータレプリケーションなど、さまざまなシナリオに対応した複数のデータ移行ツールが用意されています。

このドキュメントでは、これらのツールのユーザーシナリオ、サポートされている上流および下流、利点、および制限について紹介します。お客様のニーズに合わせて適切なツールを選択できます。

<!--以下の図は各移行ツールのユーザーシナリオを示しています。

!TiDB移行ツール media/migration-tools.png-->

## [TiDBデータ移行（DM）](/dm/dm-overview.md)

- **ユーザーシナリオ**: MySQL互換データベースからTiDBへのデータ移行
- **上流**: MySQL、MariaDB、Aurora
- **下流**: TiDB
- **利点**:
    - 完全なデータ移行および増分レプリケーションをサポートする便利で統一されたデータ移行タスク管理ツール
    - テーブルと操作のフィルタリングをサポート
    - シャードのマージと移行をサポート
- **制限**: データのインポート速度は、TiDB Lightningの[論理インポートモード](/tidb-lightning/tidb-lightning-logical-import-mode.md)とほぼ同じであり、TiDB Lightningの[物理インポートモード](/tidb-lightning/tidb-lightning-physical-import-mode.md)よりも低速です。そのため、1 TiB未満のサイズの完全なデータを移行する場合は、DMを使用することをお勧めします。

## [TiDB Lightning](/tidb-lightning/tidb-lightning-overview.md)

- **ユーザーシナリオ**: TiDBへの完全なデータインポート
- **上流（インポート元ファイル）**:
    - Dumplingからエクスポートされたファイル
    - Amazon AuroraやApache HiveによってエクスポートされたParquetファイル
    - CSVファイル
    - ローカルディスクまたはAmazon S3からのデータ
- **下流**: TiDB
- **利点**:
    - 大量のデータを迅速にインポートし、TiDBクラスター内の特定のテーブルを迅速に初期化することをサポート
    - インポートの進行状況を保存するためのチェックポイントをサポートし、再起動後に`tidb-lightning`が中断したところからインポートを継続することができる
    - データのフィルタリングをサポート
- **制限**:
    - データのインポートに[物理インポートモード](/tidb-lightning/tidb-lightning-physical-import-mode-usage.md)を使用する場合、インポートプロセス中にTiDBクラスターはサービスを提供できません。
    - TiDBサービスに影響を与えたくない場合は、TiDB Lightningの[論理インポートモード](/tidb-lightning/tidb-lightning-logical-import-mode-usage.md)に従ってデータをインポートしてください。

## [Dumpling](/dumpling-overview.md)

- **ユーザーシナリオ**: MySQLまたはTiDBからの完全なデータエクスポート
- **上流**: MySQL、TiDB
- **下流（出力ファイル）**: SQL、CSV
- **利点**:
    - データを簡単にフィルタリングできるテーブルフィルタリング機能をサポート
    - Amazon S3へのデータのエクスポートをサポート
- **制限**:
    - TiDB以外のデータベースにエクスポートしたデータをリストアする場合は、Dumplingの使用をお勧めします。
    - エクスポートしたデータを別のTiDBクラスターにリストアする場合は、バックアップ＆リストア（BR）の使用をお勧めします。

## [TiCDC](/ticdc/ticdc-overview.md)

- **ユーザーシナリオ**: このツールはTiKVの変更ログを取得することで実装されています。任意の上流TSOに基づいてクラスターデータを一貫した状態に復元し、他のシステムでデータ変更を購読することができます。
- **上流**: TiDB
- **下流**: TiDB、MySQL、Kafka、MQ、Confluent、Amazon S3、GCS、Azure Blob Storage、NFSなどのストレージサービス
- **利点**: TiCDCオープンプロトコルを提供
- **制限**: TiCDCは少なくとも1つの有効なインデックスを持つテーブルのみをレプリケートします。次のシナリオはサポートされていません:
    - RawKVのみを使用するTiKVクラスター。
    - TiDBでのDDL操作`CREATE SEQUENCE`および`SEQUENCE`機能。

## [バックアップ＆リストア（BR）](/br/backup-and-restore-overview.md)

- **ユーザーシナリオ**: バックアップおよびデータのリストアによって大量のTiDBクラスターデータを移行する
- **上流**: TiDB
- **下流（出力ファイル）**: SST、backup.metaファイル、backup.lockファイル
- **利点**:
    - 別のTiDBクラスターにデータを移行するために適しています
    - 災害復旧のために外部ストレージにデータをバックアップすることをサポート
- **制限**:
    - BRがデータをTiCDCまたはDrainerの上流クラスターに復元する場合、復元されたデータはTiCDCまたはDrainerによって下流にレプリケートされません。
    - BRは、`new_collations_enabled_on_first_bootstrap`値が同じクラスター間の操作のみをサポートしています。

## [sync-diff-inspector](/sync-diff-inspector/sync-diff-inspector-overview.md)

- **ユーザーシナリオ**: MySQLプロトコルでデータベースに格納されたデータを比較する
- **上流**: TiDB、MySQL
- **下流**: TiDB、MySQL
- **利点**: 少量のデータが不整合な場合のデータの修復に使用できます
- **制限**:
    - MySQLとTiDB間のデータ移行ではオンラインチェックはサポートされていません。
    - JSON、BIT、BINARY、BLOBなどのデータタイプはサポートされていません。

## TiUPを使用したツールのインストール

TiDB v4.0以降、TiUPはTiDBエコシステム内のさまざまなクラスターコンポーネントを管理するのに役立つパッケージマネージャとして機能します。今では、1つのコマンドで任意のクラスターコンポーネントを管理できます。

### ステップ1. TiUPをインストール

```shell
curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
```

グローバル環境変数を再宣言します:

```shell
source ~/.bash_profile
```

### ステップ2. コンポーネントをインストール

以下のコマンドを使用して利用可能なすべてのコンポーネントを表示できます:

```shell
tiup list
```

コマンドの出力には、利用可能なすべてのコンポーネントがリストされます:

```bash
利用可能なコンポーネント:
Name            Owner    Description
----            -----    -----------
bench           pingcap  異なるワークロードを持つベンチマークデータベース
br              pingcap  TiDB / TiKVクラスターバックアップリストアツール
cdc             pingcap  TiDBのデータ変更キャプチャツール
client          pingcap  playgroundに接続するクライアント
cluster         pingcap  本番用のTiDBクラスターをデプロイする
ctl             pingcap  TiDBコントローラースイート
dm              pingcap  データ移行プラットフォームマネージャー
dmctl           pingcap  データ移行プラットフォームのdmctlコンポーネント
errdoc          pingcap  TiDBエラーに関するドキュメント
pd-recover      pingcap  PD RecoverはPDクラスターの災害復旧ツールで、PDクラスターを正常に起動またはサービスを提供できない状態から回復するために使用されます
playground      pingcap  遊び心のあるローカルなTiDBクラスターを開始する
tidb            pingcap  MySQLプロトコルと互換性のあるオープンソースの分散HTAPデータベース
tidb-lightning  pingcap  大規模なデータの高速な完全なインポートをTiDBクラスターに行うためのツール
tiup            pingcap  ローカルシステムにTiDBプラットフォームコンポーネントをダウンロードおよびインストールするのに役立つコマンドラインコンポーネント管理ツール
```

インストールするコンポーネントを選択します:

```shell
tiup install dumpling tidb-lightning
```

> **注意:**
>
> 特定のバージョンのコンポーネントをインストールするには、`tiup install <component>[:version]`コマンドを使用してください。

### ステップ3. TiUPおよびそのコンポーネントをアップデートする（オプション）

新しいバージョンのリリースログと互換性ノートを確認することをお勧めします。

```shell
tiup update --self && tiup update dm
```

## 関連項目

- [TiUPをオフラインで展開](/production-deployment-using-tiup.md#deploy-tiup-offline)
- [バイナリでツールをダウンロードしてインストール](/download-ecosystem-tools.md)