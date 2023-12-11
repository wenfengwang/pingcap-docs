---
title: タイタンの構成
summary: タイタンの構成方法を学びます。

# タイタンの構成

このドキュメントでは、対応する設定項目を使用して[Titan](/storage-engine/titan-overview.md)を有効または無効にする方法と、関連するパラメータとレベルマージの機能について紹介します。

## タイタンの有効化

TitanはRocksDBと互換性があり、そのため既存のRocksDBを使用するTiKVインスタンスで直接Titanを有効にできます。以下のいずれかの方法でTitanを有効にできます。

+ 方法1：TiUPを使用してクラスタを展開した場合、`tiup cluster edit-config ${cluster-name}`コマンドを実行し、TiKV構成ファイルを次の例のように編集できます：

    {{< copyable "shell-regular" >}}

    ```shell
      tikv:
        rocksdb.titan.enabled: true
    ```

    構成を再読み込みし、TiKVは動的にローリングリスタートされます：

    {{< copyable "shell-regular" >}}

    ```shell
    tiup cluster reload ${cluster-name} -R tikv
    ```

    詳細なコマンドについては、「[TiUPを使用して構成を変更する](/maintain-tidb-using-tiup.md#modify-the-configuration)」を参照してください。

+ 方法2：TiKV構成ファイルを直接編集してTitanを有効にします（**本番環境ではお勧めしません**）。

    {{< copyable "" >}}

    ``` toml
    [rocksdb.titan]
    enabled = true
    ```

Titanが有効になると、RocksDBに保存されている既存のデータはすぐにTitanエンジンに移動されるわけではありません。新しいデータがTiKVの前景に書き込まれ、RocksDBがコンパクションを実行すると、値は徐々にキーから分離され、Titanに書き込まれます。**TiKVの詳細** -> **Titan kv** -> **blob file size**パネルで、Titanに保存されているデータのサイズを確認できます。

書き込みプロセスを高速化するには、tikv-ctlを使用してTiKVクラスタ全体のデータを手動でコンパクトしてください。詳細については、「[manual compaction](/tikv-control.md#compact-data-of-the-whole-tikv-cluster-manually)」を参照してください。

> **注：**
>
> Titanが無効になると、RocksDBはTitanに移行されたデータを読み取れません。Titanが既に有効なTiKVインスタンスでTitanを誤って無効にした場合（`rocksdb.titan.enabled`を`false`に間違えて設定した場合）、TiKVは起動に失敗し、「You have disabled titan when its data directory is not empty」というエラーがTiKVログに表示されます。Titanを正しく無効にするには、[Titanの無効化](#disable-titan)を参照してください。

## パラメータ

TiUPを使用してTitan関連のパラメータを調整するには、「[構成の変更](/maintain-tidb-using-tiup.md#modify-the-configuration)」を参照してください。

+ Titan GCスレッド数。

    **TiKVの詳細** -> **Thread CPU** -> **RocksDB CPU**パネルで、Titan GCスレッドが長時間完全に稼働している場合、Titan GCスレッドプールのサイズを増やすことを検討してください。

    {{< copyable "" >}}

    ```toml
    [rocksdb.titan]
    max-background-gc = 1
    ```

+ 値のサイズ閾値。

    前景に書き込まれる値のサイズが閾値より小さい場合、この値はRocksDBに保存され、それ以上の場合はTitanのblobファイルに保存されます。値のサイズの分布に基づいて、閾値を上げるとより多くの値がRocksDBに保存され、TiKVは小さな値を読む際に性能が向上します。閾値を下げると、より多くの値がTitanに移動するため、RocksDBのコンパクションがさらに減少します。

    ```toml
    [rocksdb.defaultcf.titan]
    min-blob-size = "1KB"
    ```

+ Titanで値を圧縮するためのアルゴリズム（値単位）。

    ```toml
    [rocksdb.defaultcf.titan]
    blob-file-compression = "lz4"
    ```

+ Titanの値キャッシュのサイズ。

    より大きなキャッシュサイズはTitanの読み取り性能が向上します。ただし、キャッシュサイズが大きすぎるとメモリ不足（OOM）が発生します。データベースが安定して稼働している場合は、`storage.block-cache.capacity`の値をブロックキャッシュサイズを除いたストアサイズに設定し、`blob-cache-size`を`メモリサイズ * 50% - ブロックキャッシュサイズ`に設定することが推奨されます。これにより、ブロックキャッシュが十分に大きい場合にTitanのブロックキャッシュサイズを最大にします。

    ```toml
    [rocksdb.defaultcf.titan]
    blob-cache-size = 0
    ```

+ ブロブファイル内の破棄可能データ（対応するキーが更新または削除されたデータ）の割合が以下の閾値を超えると、Titan GCがトリガーされます。

    ```toml
    discardable-ratio = 0.5
    ```

    Titanはこのブロブファイルの有用なデータを別のファイルに書き込む場合、`discardable-ratio`の値を使用して書き込み増幅とスペース増幅の上限を見積もることができます（圧縮が無効と仮定した場合）。

    書き込み増幅の上限 = 1 / discardable_ratio

    スペース増幅の上限 = 1 / (1 - discardable_ratio)

    上記の2つの式からわかるように、`discardable_ratio`の値を下げるとスペース増幅が減少しますが、TitanでGCが頻繁に行われるようになります。一方、値を上げるとTitanのGCが減少し、対応するI/O帯域幅およびCPU消費が減少しますが、ディスク使用量が増加します。

+ 以下のオプションによってRocksDBコンパクションのI/Oレートが制限されます。ピーク時のトラフィックでRocksDBコンパクションのI/O帯域幅およびCPU消費を制限することで、前景の書き込みと読み取り性能に対する影響を減らすことができます。

    Titanが有効な場合、このオプションはRocksDBコンパクションとTitan GCの合計I/Oレートを制限します。RocksDBコンパクションおよびTitan GCのI/OおよびCPU消費が大きすぎる場合は、適切な値に設定してください。これはディスクのI/O帯域幅と実際の書き込みトラフィックに基づいて決定します。

    ```toml
    [rocksdb]
    rate-bytes-per-sec = 0
    ```

## タイタンの無効化

Titanを無効化するには、`rocksdb.defaultcf.titan.blob-run-mode`オプションを設定できます。`blob-run-mode`のオプション値は次の通りです：

- オプションが`normal`に設定されている場合、Titanは通常通り読み取りおよび書き込みを行います。
- オプションが`read-only`に設定されている場合、すべての新しく書き込まれた値は値のサイズに関係なくRocksDBに書き込まれます。
- オプションが`fallback`に設定されている場合、すべての新しく書き込まれた値は値のサイズに関係なくRocksDBに書き込まれます。また、Titanのblobファイルに保存されているすべてのコンパクションされた値が自動的にRocksDBに移動されます。

既存および将来のすべてのデータでTitanを完全に無効にするには、以下の手順に従うことができます：

1. Titanを無効にしたいTiKVノードの構成を更新します。2つの方法で構成を更新できます：

    + `tiup cluster edit-config`を実行し、構成ファイルを編集し、`tiup cluster reload -R tikv`を実行します。
    + 手動で構成ファイルを更新し、TiKVを再起動します。

    ```toml
    [rocksdb.defaultcf.titan]
    blob-run-mode = "fallback"
    discardable-ratio = 1.0
    ```

2. tikv-ctlを使用して完全なコンパクションを実行します。このプロセスでは、大量のI/OおよびCPUリソースが消費されます。

    ```bash
    tikv-ctl --pd <PD_ADDR> compact-cluster --bottommost force
    ```

3. コンパクションが完了するまで、**TiKV-Details**/**Titan - kv**の**Blob file count**メトリクスが`0`になるまで待機してください。

4. これらのTiKVノードの構成を更新し、Titanを無効にします。

    ```toml
    [rocksdb.titan]
    enabled = false
    ```

## レベルマージ（実験的）

TiKV 4.0では、[Level Merge](/storage-engine/titan-overview.md#level-merge)という新しいアルゴリズムが導入され、範囲クエリのパフォーマンスを向上させ、Titan GCが前景の書き込み操作に与える影響を減らします。次のオプションを使用してLevel Mergeを有効にできます：

```toml
[rocksdb.defaultcf.titan]
level-merge = true
```

Level Mergeを有効にすると、以下のような利点があります：

- Titanの範囲クエリのパフォーマンスが大幅に向上します。
- Titan GCが前景の書き込み操作に与える影響が軽減され、書き込みパフォーマンスが向上します。
- Titanのスペース増幅が低減し、デフォルトの構成よりもディスク使用量が削減されます。

したがって、Level Mergeを有効にすると、書き込み増幅はTitanよりもわずかに高くなりますが、ネイティブのRocksDBよりも低くなります。