---
title: TiFlashのアップグレードガイド
summary: TiFlashをアップグレードする際の注意事項を学ぶ
aliases: ['/tidb/dev/tiflash-620-upgrade-guide']
---

# TiFlashのアップグレードガイド

このドキュメントでは、TiFlashをアップグレードする際に学ぶ必要がある機能の変更と推奨されるアクションについて説明します。

標準のアップグレードプロセスについて学ぶには、以下のドキュメントを参照してください：

- [TiUPを使用したTiDBのアップグレード](/upgrade-tidb-using-tiup.md)
- [Kubernetes上のTiDBのアップグレード](https://docs.pingcap.com/tidb-in-kubernetes/stable/upgrade-a-tidb-cluster)

> **注:**
>
> - [FastScan](/tiflash/use-fastscan.md) は実験機能としてv6.2.0で導入され、v7.0.0で一般利用可能（GA）になりました。これにより、強力なデータ整合性のコストを伴う効率的なクエリパフォーマンスが提供されます。
>
> - TiDBをメジャーバージョン間でアップグレードすることはお勧めしません。たとえば、v4.xからv6.xに移行する場合、v4.xからまずv5.xに、それからv6.xにアップグレードする必要があります。
>
> - v4.xはライフサイクルの終了が近づいています。可能な限り早くv5.xまたはそれ以降にアップグレードすることをお勧めします。詳細については、[TiDBリリースサポートポリシー](https://en.pingcap.com/tidb-release-support-policy/)を参照してください。
>
> - PingCAPはv6.0などのLTSでないバージョン向けにバグ修正を提供していません。できるだけ早くv6.1およびそれ以降のLTSバージョンにアップグレードすることをお勧めします。
>
> - TiFlashをv5.3.0より前のバージョンからv5.3.0またはそれ以降にアップグレードする場合、TiFlashを停止してからアップグレードする必要があります。次の手順に従って、他のコンポーネントに影響を与えることなくTiFlashをアップグレードできます：

>     - TiFlashインスタンスを停止します: `tiup cluster stop <cluster-name> -R tiflash`
>     - TiDBクラスタを再起動せずにアップグレードします（ファイルのみを更新します）: `tiup cluster upgrade <cluster-name> <version> --offline`，例： `tiup cluster upgrade <cluster-name> v5.3.0 --offline`
>     - TiDBクラスタをリロードします: `tiup cluster reload <cluster-name>`。リロード後、TiFlashインスタンスが起動され、手動で起動する必要はありません。

## v6.1へのアップグレード

TiFlashをv5.xまたはv6.0からv6.1にアップグレードする際には、TiFlash Proxyとダイナミックプルーニングの機能変更に注意する必要があります。

### TiFlash Proxy

TiFlash Proxyはv6.1.0でアップグレードされました（TiKV v6.0.0と同期）。新しいバージョンでは、RocksDBバージョンがアップグレードされています。TiFlashをv6.1にアップグレードすると、データ形式が自動的に新しいバージョンに変換されます。

通常のアップグレードでは、データの変換にはリスクが伴いません。ただし、特別なシナリオ（例: テストまたは検証シナリオ）でTiFlashをv6.1から以前のバージョンにダウングレードする必要がある場合、以前のバージョンは新しいRocksDB構成を解析できない可能性があります。その結果、TiFlashの再起動に失敗します。アップグレードプロセスを十分にテストして検証し、緊急時の対策を準備することをお勧めします。

**テストまたは他の特別なシナリオでTiFlashをダウングレードするためのワークアラウンド**

対象のTiFlashノードを強制的にスケーリングダウンし、再びTiKVからデータをレプリケートすることができます。詳細な手順については、[TiFlashクラスタのスケーリングダウン](/scale-tidb-using-tiup.md#scale-in-a-tiflash-cluster)を参照してください。

### ダイナミックプルーニング

将来的に[ダイナミックプルーニングモード](/partitioned-table.md#dynamic-pruning-mode)を有効にしない場合は、このセクションをスキップできます。

- 新しくインストールされたTiDB v6.1.0: ダイナミックプルーニングはデフォルトで有効になっています。

- TiDB v6.0以前: ダイナミックプルーニングはデフォルトで無効になっています。アップグレード後のダイナミックプルーニングの設定は、前のバージョンの設定を継承します。つまり、アップグレード後、ダイナミックプルーニングは自動的に有効になりません（または無効になりません）。

    アップグレード後、ダイナミックプルーニングを有効にするには、`tidb_partition_prune_mode`を`dynamic`に設定し、パーティションテーブルのGlobalStatsを手動で更新する必要があります。詳細については、[ダイナミックプルーニングモード](/partitioned-table.md#dynamic-pruning-mode)をご覧ください。

## v6.2へのアップグレード

TiDB v6.2では、TiFlashはデータストレージフォーマットをV3バージョンにアップグレードします。そのため、v5.xまたはv6.0からv6.2にTiFlashをアップグレードする際は、[TiFlash Proxy](#tiflash-proxy)および[ダイナミックプルーニング](#dynamic-pruning)の機能変更に注意する必要があります。

### PageStorage

デフォルトで、TiFlash v6.2.0ではPageStorage V3バージョン [`format_version = 4`](/tiflash/tiflash-configuration.md#configure-the-tiflashtoml-file)を使用します。この新しいデータ形式は、ピーク時の書き込みI/Oトラフィックを大幅に削減します。高い更新トラフィックと高い並行性または重いクエリを必要とするシナリオでは、TiFlashのデータGCによる過剰なCPU使用を効果的に軽減します。同時に、以前のストレージ形式と比較して、V3バージョンはスペース拡張とリソース消費を大幅に削減します。

- v6.2.0にアップグレードした後、新しいデータが既存のTiFlashノードに書き込まれると、以前のデータが徐々に新しい形式に変換されます。
- ただし、アップグレード中に既存のデータを完全に新しい形式に変換することはできません。変換にはある程度のシステムオーバーヘッドがかかります（サービスには影響がありませんが、引き続き注意が必要です）。アップグレード後、データを新しい形式に変換するために[`Compact`コマンド](/sql-statements/sql-statement-alter-table-compact.md)を実行することをお勧めします。手順は以下のとおりです：

    1. 各TiFlashレプリカを含むテーブルについて、以下のコマンドを実行します：

        ```sql
        ALTER TABLE <table_name> COMPACT tiflash replica;
        ```

    2. TiFlashノードを再起動します。

テーブルが古いデータ形式を使用しているかどうかはGrafanaで確認できます：**TiFlash-Summary** > **Storage Pool** > **Storage Pool Run Mode**。

- Only V2: PageStorage V2を使用しているテーブルの数（パーティションを含む）
- Only V3: PageStorage V3を使用しているテーブルの数（パーティションを含む）
- Mix Mode: PageStorage V2からPageStorage V3にデータ形式が変換されたテーブルの数（パーティションを含む）

**テストまたは他の特別なシナリオでTiFlashをダウングレードするためのワークアラウンド**

対象のTiFlashノードを強制的にスケーリングダウンし、再びTiKVからデータをレプリケートすることができます。詳細な手順については、[TiFlashクラスタのスケーリングダウン](/scale-tidb-using-tiup.md#scale-in-a-tiflash-cluster)を参照してください。

## v6.1からv6.2へのアップグレード

TiFlashをv6.1からv6.2にアップグレードする際には、データストレージ形式の変更に注意する必要があります。詳細については、[PageStorage](#pagestorage)を参照してください。

## `storage.format_version = 5`が構成されたv6.xまたはv7.xからv7.3へのアップグレード

v7.3から、TiFlashは新しいDTFileバージョン、DTFile V3（実験的）を導入しました。この新しいDTFileバージョンでは、複数の小さなファイルを1つの大きなファイルにマージしてファイルの総数を減らすことができます。v7.3では、デフォルトのDTFileバージョンはまだV2です。V3を使用するには、[TiFlash構成パラメータ](/tiflash/tiflash-configuration.md) `storage.format_version = 5`を設定できます。設定後、TiFlashはV2のDTFileを読み込むことができ、その後のデータ圧縮中に既存のV2 DTFileを徐々にV3 DTFileに書き換えます。

TiFlashをv7.3にアップグレードし、TiFlashを以前のバージョンに戻す必要がある場合、DTToolをオフラインで使用してV3 DTFileをV2 DTFileに書き換えることができます。詳細については、[DTToolマイグレーションツール](/tiflash/tiflash-command-line-flags.md#dttool-migrate)を参照してください。

## v6.xまたはv7.xからv7.4以降へのアップグレード

v7.4から、データ圧縮中に発生する読み取りおよび書き込みの増幅を減少させるために、TiFlashはPageStorage V3のデータ圧縮ロジックを最適化し、一部の基礎ストレージファイル名が変更されます。そのため、v7.4またはそれ以降にアップグレードした後、元のバージョンに直接ダウングレードすることはサポートされません。

**テストまたは他の特別なシナリオでTiFlashをダウングレードするためのワークアラウンド**

テストまたは他の特別なシナリオでTiFlashをダウングレードするためには、対象のTiFlashノードを強制的にスケーリングダウンし、再びTiKVからデータをレプリケートすることができます。詳細な手順については、[TiFlashクラスタのスケーリングダウン](/scale-tidb-using-tiup.md#scale-in-a-tiflash-cluster)を参照してください。