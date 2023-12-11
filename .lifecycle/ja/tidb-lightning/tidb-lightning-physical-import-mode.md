---
title: 物理インポートモード
summary: TiDB Lightningの物理インポートモードについて学びます。

# 物理インポートモード

物理インポートモードは、SQLインターフェースを経由せずにデータをTiKVノードにキーバリューペアとして直接挿入する、効率的で高速なインポートモードです。物理インポートモードを使用すると、一つのLightningのインスタンスで最大10 TiBのデータをインポートできます。サポートされるインポートデータの量は、理論的にはLightningのインスタンス数が増えるにつれて増加します。[Lightningに基づく並列インポート](/tidb-lightning/tidb-lightning-distributed-import.md) によって、ユーザー検証済みで最大50 TiBのデータを効果的に取り扱えることが確認されています。

物理インポートモードを使用する前に、[要件と制限事項](#requirements-and-restrictions) をご確認ください。

物理インポートモードのバックエンドは `local` です。`tidb-lightning.toml` で修正できます:

 ```toml
 [tikv-importer]
 # インポートモードを "local" に設定して物理インポートモードを使用します。
 backend = "local"
 ```

## 実装

1. データをインポートする前に、TiDB Lightning は自動的にTiKVノードを "インポートモード" に切り替え、書き込みの性能を向上させ、自動圧縮を停止します。TiDB Lightning は TiDB Lightning バージョンに応じてグローバルスケジューリングを一時停止するかどうかを判断します。

    - v7.1.0 からは、TiDB Lightningパラメータ [`pause-pd-scheduler-scope`](/tidb-lightning/tidb-lightning-configuration.md) を使用して、一時停止スケジューリングの範囲を制御できます。
    - v6.2.0 から v7.0.0のTiDB Lightningバージョンでは、一時停止のグローバルスケジューリングの振る舞いは、TiDBクラスタのバージョンに依存します。TiDBクラスタがv6.1.0以上の場合、TiDB Lightning はターゲットテーブルデータを保存するリージョンの全体スケジューリングを一時停止します。インポートが完了すると、TiDB Lightning はスケジューリングを回復させます。その他のバージョンの場合、TiDB Lightning はグローバルスケジューリングを一時停止します。
    - TiDB Lightningが v6.2.0 未満の場合、TiDB Lightning はグローバルスケジューリングを一時停止します。

2. TiDB Lightning は対象データベースにテーブルスキーマを作成し、メタデータを取得します。

    `add-index-by-sql` を `true` に設定した場合、`tidb-lightning` はSQLインターフェースを介してインデックスを追加し、データをインポートする前に対象テーブルからすべてのセカンダリインデックスを削除します。デフォルト値は `false` で、これは以前のバージョンと一貫しています。

3. 各テーブルは複数の連続した **ブロック** に分割されます。これにより、TiDB Lightning は大きなテーブル（200 GB以上）からデータを並列にインポートできます。

4. TiDB Lightning は各ブロックに対して "エンジンファイル" を準備し、キーバリューペアを処理します。TiDB Lightning はSQLダンプを並列に読み取り、データソースをTiDBと同じエンコーディングでキーバリューペアに変換し、キーバリューペアをソートしてローカルの一時ストレージファイルに書き込みます。

5. エンジンファイルが書き込まれると、TiDB Lightning は対象のTiKVクラスタ上でデータを分割しスケジュールし、TiKVクラスタにデータをインポートします。

    エンジンファイルには2つの種類のエンジンが含まれます: **データエンジン** と **インデックスエンジン**。各エンジンはキーバリューペアのタイプに対応しており、行データとセカンダリインデックスです。通常、行データはデータソース内で完全に順序付けされており、セカンダリインデックスは順序がありません。したがって、データエンジンファイルは対応するブロックが書き込まれるとすぐにインポートされ、すべてのインデックスエンジンファイルはテーブル全体がエンコードされた後にのみインポートされます。

    `tidb-lightning` が SQLインターフェースを介してインデックスを追加する場合（つまり `add-index-by-sql` を `true` に設定した場合）、インデックスエンジンはデータを書き込まないため、ステップ2で対象テーブルのセカンダリインデックスが既に削除されているためです。

6. すべてのエンジンファイルがインポートされた後、TiDB Lightning はローカルデータソースとダウンストリームクラスタ間のチェックサムを比較し、インポートされたデータが破損していないことを確認します。その後、TiDB Lightning はステップ2で以前に削除したセカンダリインデックスを追加したり、新しいデータを解析して将来の操作を最適化します（`ANALYZE`）。同時に、将来の競合を防ぐために `AUTO_INCREMENT` の値を調整します。

    オートインクリメントIDは、行数の上限値とテーブルデータファイルの合計サイズに比例して推定されます。したがって、オートインクリメントIDは通常、実際の行数よりも大きくなります。これは、オートインクリメントIDが [必ずしも連続的でない](/mysql-compatibility.md#auto-increment-id) ためです。

7. すべてのステップが完了すると、TiDB Lightning は自動的にTiKVノードを "通常モード" に切り替えます。グローバルスケジューリングが一時停止されている場合、TiDB Lightning はグローバルスケジューリングも回復します。その後、TiDBクラスタは通常通りサービスを提供できます。

## 要件と制限事項

### 環境要件

**オペレーティングシステム**:

新しい CentOS 7 インスタンスを使用することをお勧めします。ローカルホストまたはクラウド上の仮想マシンに展開できます。デフォルトでは TiDB Lightning は必要なCPUリソースを消費するため、専用サーバーに展開することをお勧めします。これができない場合は、他のTiDBコンポーネント（例: tikv-server）と一緒に単一のサーバーに展開し、その後`region-concurrency` を構成して TiDB Lightning からのCPU使用量を制限できます。通常、CPUの論理数の75%にサイズを設定できます。

**メモリとCPU**:

性能を向上させるために、32コア以上のCPUと64 GiB以上のメモリを割り当てることをお勧めします。

> **注記:**
>
> 大量のデータをインポートする場合、1つの同時インポートがおよそ2 GiBのメモリを消費することがあります。合計メモリ使用量は `region-concurrency * 2 GiB` になります。デフォルトでは `region-concurrency` は論理CPUの数と同じです。メモリサイズ（GiB）がCPUの2倍未満の場合、もしくはインポート中にOOMが発生した場合、OOMを回避するために `region-concurrency` を減らすことができます。

**ストレージ**:
`sorted-kv-dir` 設定項目は、ソートされたキーバリューファイルの一時ストレージディレクトリを指定します。ディレクトリは空である必要があり、ストレージスペースはインポートするデータセットのサイズよりも大きい必要があります。インポートパフォーマンスを向上させるためには、`data-source-dir` とは異なるディレクトリを使用し、ディレクトリのフラッシュストレージおよび専用I/Oを使用することをお勧めします。

**ネットワーク**:
10Gbpsのイーサネットカードを使用することをお勧めします。

### バージョン要件

- TiDB Lightning >= v4.0.3。
- TiDB >= v4.0.0。

### 制限事項

- 本番のTiDBクラスタにデータを直接インポートするために物理インポートモードを使用しないでください。重大なパフォーマンスへの影響があります。必要な場合は、[インポート中のテーブルレベルでスケジューリングを一時停止する](/tidb-lightning/tidb-lightning-physical-import-mode-usage.md#scope-of-pausing-scheduling-during-import) を参照してください。
- TiDBクラスタが遅延に敏感なアプリケーションと低い並行性を持つ場合、物理インポートモードを使用してデータをクラスタにインポートしないでください。このモードはオンラインアプリケーションに大きな影響を与える可能性があります。
- 複数のTiDB Lightningインスタンスを使用してデータを同じTiDBクラスタにインポートしないでください。代わりに[並列インポート](/tidb-lightning/tidb-lightning-distributed-import.md) を使用してください。
- 複数のTiDB Lightningを使用してデータを同じ対象クラスタにインポートする場合は、複数のインポートモードを混在させないでください。つまり、物理インポートモードと論理インポートモードを同時に使用しないでください。
- データをインポートする過程で、対象テーブルでDDLおよびDML操作を実行しないでください。さもないと、インポートが失敗するかデータが不整合になります。同時に、読み取り操作を実行することはお勧めしません。読み取ったデータが不整合である可能性があるためです。インポート操作が完了した後に読み取りおよび書き込み操作を実行できます。
- 一つのLightningプロセスで最大10 TBの単一テーブルをインポートできます。並列インポートは最大で10のLightningインスタンスを使用できます。

### 他のコンポーネントとの使用方法のヒント

- TiDB LightningをTiFlashと共に使用する場合は、次の点に注意してください:

    - テーブルのTiFlashレプリカを作成しているかどうかにかかわらず、TiDB Lightningを使用してデータをテーブルにインポートできます。ただし、通常のインポートより時間がかかる場合があります。インポート時間は、TiDB Lightningが展開されたサーバーのネットワーク帯域幅、TiFlashノードのCPUおよびディスク負荷、TiFlashレプリカの数に影響を受けます。

- TiDB Lightningの文字セット:

    - v5.4.0 よりも古い TiDB Lightning は `charset=GBK` のテーブルをインポートできません。

- TiDB LightningをTiCDCと共に使用する場合は、次の点に注意してください:

    - TiCDC は物理インポートモードで挿入されたデータを取得できません。