---
title: 高同時書き込みのベストプラクティス
summary: TiDBにおける高い同時書き込み重視のワークロードに関するベストプラクティスを学びます。
aliases: ['/docs/dev/best-practices/high-concurrency-best-practices/', '/docs/dev/reference/best-practices/high-concurrency/']
---

# 高同時書き込みのベストプラクティス

このドキュメントでは、TiDBにおける高い同時書き込みのワークロードに対処するためのベストプラクティスについて説明します。これにより、アプリケーション開発の促進に役立ちます。

## ターゲットオーディエンス

このドキュメントは、TiDBの基本的な理解があることを前提としています。以下の3つのブログ記事を最初に読むことをお勧めします。これらの記事では、TiDBの基本事項と[TiDBベストプラクティス](https://en.pingcap.com/blog/tidb-best-practice/)が説明されています：

+ [データストレージ](https://en.pingcap.com/blog/tidb-internal-data-storage/)
+ [計算](https://en.pingcap.com/blog/tidb-internal-computing/)
+ [スケジューリング](https://en.pingcap.com/blog/tidb-internal-scheduling/)

## 高同時書き込み重視シナリオ

高い同時書き込みシナリオは、アプリケーションでのバッチタスクの実行時によく発生します。これにはクリアリングや決済などが含まれます。このシナリオには以下の特徴があります：

+ 膨大なデータ量
+ 短時間で過去のデータをデータベースにインポートする必要
+ 短時間でデータベースから膨大なデータを読み取る必要

これらの特徴により、TiDBには以下の課題が発生します：

+ 書き込みまたは読み取り容量を線形にスケーラブルにする必要がある
+ データが同時に大量に書き込まれてもデータベースのパフォーマンスが安定して低下しない必要がある

分散データベースの場合、すべてのノードの容量を最大限に活用し、単一のノードがボトルネックになるのを防ぐことが重要です。

## TiDBにおけるデータ分布原則

上記の課題に対処するためには、TiDBのデータセグメンテーションとスケジューリングの原則から始める必要があります。詳細については[Scheduling](https://en.pingcap.com/blog/tidb-internal-scheduling/)を参照してください。

TiDBはデータをリージョンに分割します。各リージョンにはデフォルトでサイズ制限が96Mあります。各リージョンには複数のレプリカがあり、各レプリカグループはRaftグループと呼ばれます。Raftグループでは、リージョンリーダーがデータ範囲内での読み込みと書き込みタスクを実行します（TiDBは[Follower-Read](/follower-read.md)をサポートしています）。リージョンリーダーは配置ドライバ（PD）コンポーネントによって異なる物理ノードに均等にスケジュールされ、読み取りと書き込みの圧力を分散します。

![TiDBデータ概要](/media/best-practices/tidb-data-overview.png)

理論的には、アプリケーションに書き込みホットスポットがない場合、TiDBのアーキテクチャにより、読み込みと書き込みの容量を線形にスケーラブルにするだけでなく、分散リソースを最大限に活用することができます。この観点から見ると、TiDBは特に高同時書き込み重視のシナリオに適しています。

しかし、実際の状況は理論的な仮定とは異なることが多いです。

> **注記：**
>
>  アプリケーションで書き込みホットスポットがない場合、書き込みシナリオには`AUTO_INCREMENT`主キーまたは単調増加インデックスがありません。

## ホットスポットケース

以下のケースでは、ホットスポットが生成される方法を説明します。以下の表を例として取り上げます：

{{< copyable "sql" >}}

```sql
CREATE TABLE IF NOT EXISTS TEST_HOTSPOT(
      id         BIGINT PRIMARY KEY,
      age        INT,
      user_name  VARCHAR(32),
      email      VARCHAR(128)
)
```

このテーブルの構造はシンプルです。プライマリキーとしての`id`の他に、二次インデックスは存在しません。次のステートメントを実行して、このテーブルにデータを書き込みます。`id`はランダムな番号として離散的に生成されます。

{{< copyable "sql" >}}

```sql
SET SESSION cte_max_recursion_depth = 1000000;
INSERT INTO TEST_HOTSPOT
SELECT
  n,                                       -- ID
  RAND()*80,                               -- 0から80の間の数値
  CONCAT('user-',n),
  CONCAT(
    CHAR(65 + (RAND() * 25) USING ascii),  -- 65から65+25の間の数値を文字に変換、A-Z
    '-user-',
    n,
    '@example.com'
  )
FROM
  (WITH RECURSIVE nr(n) AS 
    (SELECT 1                              -- CTEの開始を1で開始
      UNION ALL SELECT n + 1               -- ループごとにnを1増やす
      FROM nr WHERE n < 1000000            -- 1,000,000でループを停止 
    ) SELECT n FROM nr
  ) a;
```

上記のステートメントを集中的に短時間で実行し、負荷をかけます。

理論的には、上記の操作はTiDBのベストプラクティスに準拠しており、アプリケーションでホットスポットが発生することはありません。適切なマシンを使用することで、TiDBの分散容量を十分に活用できます。これが理論の最適な状況であるかどうかを確認するために、実験環境でテストが実施されます。

クラスタトポロジーでは、TiDBノード2つ、PDノード3つ、TiKVノード6つが展開されています。ベンチマークではなく、原則の理解を確認するため、QPSのパフォーマンスは無視してください。

![QPS1](/media/best-practices/QPS1.png)

クライアントは、短時間で"集中的な"書き込みリクエストを開始し、TiDBが受信するQPSが3Kです。理論的には、負荷は6つのTiKVノードに均等に分散されるはずです。しかし、各TiKVノードのCPU使用率を見ると、負荷分布は均等ではありません。`tikv-3`ノードが書き込みホットスポットになります。

![QPS2](/media/best-practices/QPS2.png)

![QPS3](/media/best-practices/QPS3.png)

[RaftストアCPU](/grafana-tikv-dashboard.md)は通常、`raftstore`スレッドのCPU使用率を表し、書き込み負荷を表します。このシナリオでは、`tikv-3`がこのRaftグループのリーダーであり、`tikv-0`および`tikv-1`がフォロワーです。他のノードの負荷はほとんどありません。

PDの監視メトリクスもホットスポットが発生したことを確認しています。

![QPS4](/media/best-practices/QPS4.png)

## ホットスポットの原因

上記のテストでは、ベストプラクティスで期待される理想的なパフォーマンスには到達しません。その理由は、TiDBではデフォルトで新しく作成されたテーブルのデータを保存するために1つのリージョンのみが分割されるためです。データ範囲は以下のようになります：

```
[CommonPrefix + TableID, CommonPrefix + TableID + 1)
```

短時間で膨大なデータが同じリージョンに連続して書き込まれるため、1つのリージョンにおける負荷が集中します。

![TiKVリージョン分割](/media/best-practices/tikv-Region-split.png)

上記の図は、リージョンの分割プロセスを示しています。TiKVにデータが連続して書き込まれると、TiKVは1つのリージョンを複数のリージョンに分割します。リージョンリーダーの選出は、リージョンリーダーが分割される元のストアで開始されるため、新しく分割された2つのリージョンのリーダーはまだ同じストアにあるかもしれません。このプロセスは新しく分割されたリージョン 2 およびリージョン 3でも発生します。これにより、書き込み負荷がTiKV-Node 1に集中します。

継続的な書き込みプロセス中に、ノード1でホットスポットが発生したことがわかると、PDは集中したリーダーを他のノードに均等に分散します。TiKVノードの数がリージョンレプリカの数よりも多い場合、TiKVはこれらのリージョンをアイドル状態のノードに移行しようとします。書き込みプロセス中のこれら2つの操作は、PDの監視メトリクスにも反映されます。

![QPS5](/media/best-practices/QPS5.png)

継続的な書き込みの一定期間後、PDはTiKVクラスタ全体を自動的にスケジュールして、負荷が均等に分散された状態にします。この時点で、クラスタ全体の容量を完全に使用することができます。

多くの場合、こうしたホットスポットの生成プロセスは正常であり、これはデータベースのリージョンのウォームアップフェーズですが、高い同時書き込み重視のシナリオではこのフェーズを避ける必要があります。

## ホットスポットの解決方法

理論的に望ましいパフォーマンスを達成するためには、リージョンを直接所望のリージョン数に分割し、これらのリージョンを事前にクラスタの他のノードにスケジュールすることで、ウォームアップフェーズをスキップできます。

v3.0.x、v2.1.13およびそれ以降のバージョンでは、TiDBは[Split Region](/sql-statements/sql-statement-split-region.md)という新しい機能をサポートしています。この新機能では、次の新しい構文が提供されます：

{{< copyable "sql" >}}

```sql
SPLIT TABLE table_name [INDEX index_name] BETWEEN (lower_value) AND (upper_value) REGIONS region_num
```

{{< copyable "sql" >}}

```sql
SPLIT TABLE table_name [INDEX index_name] BY (value_list) [, (value_list)]
```

ただし、TiDBはこの事前分割操作を自動的に実行しません。その理由は、TiDBのデータ分布に関連しています。

![テーブルリージョンの範囲](/media/best-practices/table-Region-range.png)

上記の図からわかるように、行のキーのエンコーディングルールに基づくと、`rowID`は唯一の変数部分です。TiDBでは、`rowID`は`Int64`整数です。ただし、必ずしも`Int64`整数範囲を均等に分割し、それらの範囲を異なるノードに分配する必要はありません。なぜなら、リージョンの分割は実際の状況に基づいている必要があるからです。
`rowID`の書き込みが完全に離散している場合、上記の手法はホットスポットを引き起こしません。行IDやインデックスに固定された範囲やプレフィックスがある場合(たとえば、範囲`[2000w、5000w)`にデータを離散的に挿入する場合)、ホットスポットは引き起こされません。しかし、上記の手法を使用してリージョンを分割すると、データは最初に同じリージョンに書き込まれる可能性があります。

TiDBは一般的な用途のためのデータベースであり、データの分布についての仮定は行いません。そのため、テーブルのデータを格納するために最初は1つのリージョンのみを使用し、実際のデータが挿入された後にデータの分布に応じてそのリージョンを自動的に分割します。

この状況とホットスポット問題の回避の必要性を考慮して、TiDBは高い並行性で書き込みが多いシナリオのパフォーマンスを最適化するために`Split Region`構文を提供しています。上記の場合に基づいて、`Split Region`構文を使用して今リージョンを散乱させ、負荷分散を観察します。

テストで書き込むデータは、正数の範囲内で完全に離散しているため、次のステートメントを使用してテーブルを`minInt64`から`maxInt64`の範囲内で128リージョンに事前分割できます。

{{< copyable "sql" >}}

```sql
SPLIT TABLE TEST_HOTSPOT BETWEEN (0) AND (9223372036854775807) REGIONS 128;
```

事前分割操作の後、`SHOW TABLE test_hotspot REGIONS;`ステートメントを実行してリージョンの分散状態を確認します。`SCATTERING`列の値がすべて`0`であれば、スケジューリングが成功しています。

また、次のSQLステートメントを使用してリージョンリーダーの分布を確認できます。`table_name`は実際のテーブル名に置き換える必要があります。

{{< copyable "sql" >}}

```sql
SELECT
    p.STORE_ID,
    COUNT(s.REGION_ID) PEER_COUNT
FROM
    INFORMATION_SCHEMA.TIKV_REGION_STATUS s
    JOIN INFORMATION_SCHEMA.TIKV_REGION_PEERS p ON s.REGION_ID = p.REGION_ID
WHERE
    TABLE_NAME = 'table_name'
    AND p.is_leader = 1
GROUP BY
    p.STORE_ID
ORDER BY
    PEER_COUNT DESC;
```

その後、書き込み負荷を再度操作します:

![QPS6](/media/best-practices/QPS6.png)

![QPS7](/media/best-practices/QPS7.png)

![QPS8](/media/best-practices/QPS8.png)

明らかなホットスポット問題が解決されたことがわかります。

この場合、テーブルはシンプルです。他の場合では、インデックスのホットスポット問題も考慮する必要があります。インデックスリージョンを事前分割する方法の詳細については、[Split Region](/sql-statements/sql-statement-split-region.md)を参照してください。

## 複雑なホットスポット問題

**問題1:**

テーブルに主キーがない場合、または主キーが`Int`型でなくランダムに分布する主キーIDを生成したくない場合、TiDBは主キーとして`_tidb_rowid`列を暗黙的に提供します。一般的には、`SHARD_ROW_ID_BITS`パラメータを使用しない場合、`_tidb_rowid`列の値も単調増加するため、これもホットスポットを引き起こす可能性があります。詳細については[`SHARD_ROW_ID_BITS`](/shard-row-id-bits.md)を参照してください。

この状況でホットスポット問題を回避するには、テーブルを作成する際に`SHARD_ROW_ID_BITS`および`PRE_SPLIT_REGIONS`を使用できます。`PRE_SPLIT_REGIONS`の詳細については[事前分割リージョン](/sql-statements/sql-statement-split-region.md#pre_split_regions)を参照してください。

`SHARD_ROW_ID_BITS`は、`_tidb_rowid`列で生成された行IDをランダムに分散するために使用されます。`PRE_SPLIT_REGIONS`は、テーブルが作成された後にリージョンを事前分割するために使用されます。

> **注意:**
>
> `PRE_SPLIT_REGIONS`の値は`SHARD_ROW_ID_BITS`の値以下である必要があります。

例:

{{< copyable "sql" >}}

```sql
create table t (a int, b int) SHARD_ROW_ID_BITS = 4 PRE_SPLIT_REGIONS=3;
```

- `SHARD_ROW_ID_BITS = 4`は、`tidb_rowid`の値がランダムに16個(16=2^4)の範囲に分散されることを意味します。
- `PRE_SPLIT_REGIONS=3`は、テーブルが作成された後に8(2^3)のリージョンに事前分割されることを意味します。

テーブル`t`にデータを書き込み始めると、データは事前に分割された8つのリージョンに書き込まれます。これにより、テーブル作成後に1つのリージョンだけが存在する場合に引き起こされるホットスポット問題が回避されます。

> **注意:**
>
> `tidb_scatter_region`グローバル変数は`PRE_SPLIT_REGIONS`の動作に影響を与えます。
>
> この変数は、テーブル作成後にリージョンが事前に分割され、散らばることを待つかどうかを制御します。テーブルの作成後に書き込みが集中する場合、この変数の値を`1`に設定する必要があります。そうすると、TiDBは全てのリージョンが分割され、散乱されるまでクライアントへの結果を返しません。それ以外の場合、散乱が完了する前にTiDBはデータを書き込み、書き込みパフォーマンスに大きな影響を与えます。

**問題2:**

テーブルの主キーがInteger型であり、テーブルが主キーの一意性を保証するために`AUTO_INCREMENT`を使用していても(連続して増加する必要はない)、このテーブルのホットスポットを`SHARD_ROW_ID_BITS`で分散させることはできません。なぜならTiDBは直接主キーの行値を`_tidb_rowid`として使用するからです。

このシナリオで問題を解決するには、データ挿入時に[`AUTO_RANDOM`](/auto-random.md) (列属性)で`AUTO_INCREMENT`を置き換えることができます。その後、TiDBは整数型の主キー列に値を自動的に割り当て、行IDの連続性を排除し、ホットスポットを分散します。

## パラメータ構成

v2.1では、[ラッチメカニズム](/tidb-configuration-file.md#txn-local-latches)がTiDBに導入され、書き込み衝突が頻繁に発生するシナリオでトランザクション衝突を事前に識別することを目的としています。これにより、TiDBおよびTiKVでのトランザクションコミットの再試行が減少します。一般的に、バッチタスクはTiDBにすでに格納されているデータを使用するため、トランザクションの書き込み衝突は存在しません。このような状況では、小さなオブジェクトのためのメモリ割り当てを減らすために、TiDBでラッチを無効にすることができます:

```
[txn-local-latches]
enabled = false
```