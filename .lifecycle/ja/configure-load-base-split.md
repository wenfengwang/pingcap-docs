---
title: Load Base Split
summary: Load Base Split の機能を学ぶ。
aliases: ['/docs/dev/configure-load-base-split/']
---

# Load Base Split

Load Base Split は TiDB 4.0 で導入された新機能です。これは、小さなテーブルに対するフルテーブルスキャンなど、リージョン間のアクセスの不均衡によって引き起こされるホットスポットの問題を解決することを目的としています。

## シナリオ

TiDBでは、特定のノードに負荷が集中すると、ホットスポットが簡単に生成されます。PDは、性能を向上させるために、ホットスポットを可能な限り均等にすべてのノードに分散するようスケジュールを行います。

ただし、PDのスケジューリングの最小単位はリージョンです。クラスタ内のホットスポットの数がノードの数よりも少ない場合、または数個のホットスポットが他のリージョンよりもはるかに多くの負荷を持つ場合、PDはホットスポットを別のノードに移動することしかできず、クラスタ全体で負荷を共有することはできません。

このようなシナリオは、主に読み取りリクエストが多いワークロードでよく起こり、そのようなワークロードには、小さなテーブルに対するフルテーブルスキャンやインデックス検索、あるいは特定のフィールドへの頻繁なアクセスが含まれます。

この問題の以前の解決策は、1つまたは複数のホットスポットリージョンを分割するコマンドを手動で実行することでしたが、このアプローチには次の2つの問題があります。

- リージョンを均等に分割することが常に最善の選択肢であるとは限らず、リクエストがわずかなキーに集中している場合、均等に分割した後もホットスポットが1つのリージョンに残る可能性があり、目標を達成するために均等な分割を複数回行う必要があります。
- 人の介入が及びやすくもないことです。

## 実装の原則

Load Base Split は統計情報に基づいてリージョンを自動的に分割します。それは、10秒間にわたって読み込み負荷または CPU 使用率が一貫してしきい値を超えるリージョンを特定し、これらのリージョンを適切な位置で分割します。分割位置を選択する際、Load Base Split は、分割後の両リージョンのアクセス負荷を均衡させ、リージョン間のアクセスを避けるようにします。

Load Base Split によって分割されたリージョンは、迅速にマージされません。一方で、PDの `MergeChecker` はホットスポットのリージョンをスキップします。また、PD はヒートビート情報の `QPS` に基づいて２つのリージョンをマージするかどうかも判断し、高い `QPS` を持つ２つのリージョンのマージを避けるようにします。

## 使用法

Load Base Split 機能は、現在、次のパラメータによって制御されています。

- [`split.qps-threshold`](/tikv-configuration-file.md#qps-threshold): リージョンがホットスポットとして特定される QPS 閾値。`region-split-size` が 4 GB 未満の場合、デフォルト値は`3000`  になります。それ以上の場合はデフォルト値は `7000` になります。
- [`split.byte-threshold`](/tikv-configuration-file.md#byte-threshold-new-in-v50): (v5.0 で導入) リージョンがホットスポットとして特定されるトラフィック閾値。単位はバイトです。`region-split-size` が 4 GB 未満の場合、デフォルト値は 30 MiB/秒になります。それ以上の場合はデフォルト値は 100 MiB/秒です。
- [`split.region-cpu-overload-threshold-ratio`](/tikv-configuration-file.md#region-cpu-overload-threshold-ratio-new-in-v620): (v6.2.0 で導入) リージョンがホットスポットとして特定される CPU 使用率の閾値（読み取りスレッドプールの CPU 時間の割合）。`region-split-size` が 4 GB 未満の場合、デフォルト値は `0.25` になります。それ以上の場合はデフォルト値は `0.75` になります。

リージョンが以下のいずれかの条件を10秒間連続で満たす場合、TiKV はリージョンを分割しようとします。

- リージョンの読み取りリクエストの合計が `split.qps-threshold` を超える。
- トラフィックが `split.byte-threshold` を超える。
- ユニファイドリードプールの CPU 使用率が `split.region-cpu-overload-threshold-ratio` を超える。

Load Base Split はデフォルトで有効ですが、パラメータはかなり高い値に設定されています。この機能を無効にする場合は、`split.qps-threshold` と `split.byte-threshold` を十分に高く設定し、同時に `split.region-cpu-overload-threshold-ratio` を `0` に設定してください。

パラメータを変更するには、次のいずれかの方法を選択してください。

- SQL 文を使用します:

    ```sql
    # QPS 閾値を 1500 に設定
    SET config tikv split.qps-threshold=1500;
    # トラフィック閾値を 15 MiB (15 * 1024 * 1024) に設定
    SET config tikv split.byte-threshold=15728640;
    # CPU 使用率の閾値を 50% に設定
    SET config tikv split.region-cpu-overload-threshold-ratio=0.5;
    ```

- TiKV を使用します:

    {{< copyable "shell-regular" >}}

    ```shell
    curl -X POST "http://ip:status_port/config" -H "accept: application/json" -d '{"split.qps-threshold":"1500"}'
    curl -X POST "http://ip:status_port/config" -H "accept: application/json" -d '{"split.byte-threshold":"15728640"}'
    curl -X POST "http://ip:status_port/config" -H "accept: application/json" -d '{"split.region-cpu-overload-threshold-ratio":"0.5"}'
    ```

それに応じて、以下のいずれかの方法を使用して構成を表示できます。

- SQL 文を使用します:
  
    {{< copyable "sql" >}}
  
    ```sql
    show config where type='tikv' and name like '%split.qps-threshold%';
    ```

- TiKV を使用します:

    {{< copyable "shell-regular" >}}

    ```shell
    curl "http://ip:status_port/config"
    ```

> **注意:**
>
> v4.0.0-rc.2 以降、SQL 文を使用して構成を変更および表示することが可能です。