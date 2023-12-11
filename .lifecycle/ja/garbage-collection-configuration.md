---
title: ガベージコレクションの構成
summary: GCの構成パラメータについて学びます。
aliases: ['/docs/dev/garbage-collection-configuration/','/docs/dev/reference/garbage-collection/configuration/']
---

# ガベージコレクションの構成

次のシステム変数を使用して、ガベージコレクション（GC）を構成できます。

* [`tidb_gc_enable`](/system-variables.md#tidb_gc_enable-new-in-v50): TiKVのガベージコレクションを有効にするかどうかを制御します。
* [`tidb_gc_run_interval`](/system-variables.md#tidb_gc_run_interval-new-in-v50): GCの間隔を指定します。
* [`tidb_gc_life_time`](/system-variables.md#tidb_gc_life_time-new-in-v50): 各GCでデータが保持される期間を指定します。
* [`tidb_gc_concurrency`](/system-variables.md#tidb_gc_concurrency-new-in-v50): GCの[ロックの解決](/garbage-collection-overview.md#resolve-locks)ステップでのスレッド数を指定します。
* [`tidb_gc_scan_lock_mode`](/system-variables.md#tidb_gc_scan_lock_mode-new-in-v50): GCの[ロックの解決](/garbage-collection-overview.md#resolve-locks)ステップでのロックのスキャン方法を指定します。
* [`tidb_gc_max_wait_time`](/system-variables.md#tidb_gc_max_wait_time-new-in-v610): アクティブなトランザクションがGCセーフポイントをブロックする最大時間を指定します。

システム変数の値の変更方法の詳細については、[システム変数](/system-variables.md)を参照してください。

## GC I/O 制限

<CustomContent platform="tidb-cloud">

> **注意:**
>
> このセクションはTiDB Self-Hostedにのみ適用されます。TiDB CloudにはデフォルトでGC I/O制限がありません。

</CustomContent>

TiKVはGC I/O制限をサポートしています。`gc.max-write-bytes-per-sec`を構成して、GCワーカーが1秒あたりの書き込みを制限し、通常のリクエストへの影響を軽減できます。

`0`はこの機能を無効にします。

この構成は、tikv-ctlを使用して動的に変更できます:

{{< copyable "shell-regular" >}}

```bash
tikv-ctl --host=ip:port modify-tikv-config -n gc.max-write-bytes-per-sec -v 10MB
```

## TiDB 5.0の変更

TiDBの以前のリリースでは、ガベージコレクションは`mysql.tidb`システムテーブルを介して構成されていました。このテーブルへの変更は引き続きサポートされていますが、提供されたシステム変数を使用することが推奨されています。これにより構成の変更が検証され、予期せぬ動作が防がれます（[#20655](https://github.com/pingcap/tidb/issues/20655)）。

`CENTRAL`ガベージコレクションモードはもはやサポートされていません。代わりに`DISTRIBUTED`GCモード（TiDB 3.0以降のデフォルト）が自動的に使用されます。このモードでは、TiDBは各TiKVリージョンにリクエストを送信してガベージコレクションをトリガーする必要がなくなるため、より効率的です。

以前のリリースでの変更については、左側のメニューの_TIDBバージョンセレクタ_を使用して、以前のバージョンのこのドキュメントを参照してください。

## TiDB 6.1.0の変更

TiDB v6.1.0以前では、TiDBのトランザクションはGCセーフポイントに影響を与えませんでした。v6.1.0以降、TiDBはトランザクションのstartTSを考慮してGCセーフポイントを計算し、アクセスされるデータが消去された問題を解決しました。トランザクションが長すぎると、セーフポイントが長時間ブロックされ、アプリケーションのパフォーマンスに影響する可能性があります。

TiDB v6.1.0では、システム変数[`tidb_gc_max_wait_time`](/system-variables.md#tidb_gc_max_wait_time-new-in-v610)が導入され、アクティブなトランザクションがGCセーフポイントをブロックする最大時間を制御します。値を超えると、GCセーフポイントは強制的に前進します。

### コンパクションフィルターにおけるGC

`DISTRIBUTED` GCモードをベースにしたコンパクションフィルターにおけるGCのメカニズムは、別個のGCワーカースレッドではなく、RocksDBのコンパクションプロセスを使用して実行されます。この新しいGCメカニズムにより、GCによる余分なディスク読み込みを回避できます。また、不要なデータをクリアした後、大量の墓石印を残さずに、連続スキャンパフォーマンスを低下させることがあります。

<CustomContent platform="tidb-cloud">

> **注意:**
>
> TiDB Self-Hostedにのみ適用される、TiKV構成の変更例です。TiDB Cloudでは、コンパクションフィルターにおけるGCのメカニズムはデフォルトで有効です。

</CustomContent>

以下の例は、TiKV構成ファイルでメカニズムを有効にする方法を示しています:

{{< copyable "" >}}

```toml
[gc]
enable-compaction-filter = true
```

このGCメカニズムは、構成を動的に変更することで有効にすることもできます。次の例を参照してください:

{{< copyable "sql" >}}

```sql
show config where type = 'tikv' and name like '%enable-compaction-filter%';
```

```sql
+------+-------------------+-----------------------------+-------+
| Type | Instance          | Name                        | Value |
+------+-------------------+-----------------------------+-------+
| tikv | 172.16.5.37:20163 | gc.enable-compaction-filter | false |
| tikv | 172.16.5.36:20163 | gc.enable-compaction-filter | false |
| tikv | 172.16.5.35:20163 | gc.enable-compaction-filter | false |
+------+-------------------+-----------------------------+-------+
```

{{< copyable "sql" >}}

```sql
set config tikv gc.enable-compaction-filter = true;
show config where type = 'tikv' and name like '%enable-compaction-filter%';
```

```sql
+------+-------------------+-----------------------------+-------+
| Type | Instance          | Name                        | Value |
+------+-------------------+-----------------------------+-------+
| tikv | 172.16.5.37:20163 | gc.enable-compaction-filter | true  |
| tikv | 172.16.5.36:20163 | gc.enable-compaction-filter | true  |
| tikv | 172.16.5.35:20163 | gc.enable-compaction-filter | true  |
+------+-------------------+-----------------------------+-------+
```