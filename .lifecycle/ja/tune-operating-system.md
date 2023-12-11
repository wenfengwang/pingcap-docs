---
title: オペレーティングシステムのパフォーマンスチューニング
summary: オペレーティングシステムのパラメータをチューニングする方法を学びます。
aliases: ['/docs/dev/tune-operating-system/']
---

# オペレーティングシステムのパフォーマンスチューニング

このドキュメントでは、CentOS 7の各サブシステムをチューニングする方法について紹介します。

> **注意:**
>
> + CentOS 7オペレーティングシステムのデフォルト設定は、中程度のワークロードで実行されるほとんどのサービスに適しています。特定のサブシステムのパフォーマンスを調整すると、他のサブシステムに悪影響を与える可能性があります。そのため、システムのチューニングを行う前に、ユーザーデータと設定情報をすべてバックアップしてください。
> + プロダクション環境に適用する前に、テスト環境で変更内容を十分にテストしてください。

## パフォーマンス分析方法

システムのチューニングは、システムのパフォーマンス分析の結果に基づいて行う必要があります。このセクションでは、パフォーマンス分析のための一般的な方法をリストします。

### 60秒で分析する

[*Linux Performance Analysis in 60,000 Milliseconds*](http://www.brendangregg.com/Articles/Netflix_Linux_Perf_Analysis_60s.pdf) は、著者の Brendan Gregg と Netflix パフォーマンスエンジニアリングチームによって発行されたものです。使用されているツールはすべて公式のリリースから入手できます。以下のリスト項目の出力を分析して、一般的なパフォーマンスの問題をトラブルシューティングできます。

+ `uptime`
+ `dmesg | tail`
+ `vmstat 1`
+ `mpstat -P ALL 1`
+ `pidstat 1`
+ `iostat -xz 1`
+ `free -m`
+ `sar -n DEV 1`
+ `sar -n TCP,ETCP 1`
+ `top`

詳細な使用方法については、対応する`man`の説明を参照してください。

### perf

perf はLinuxカーネルによって提供される重要なパフォーマンス分析ツールであり、ハードウェアレベル（CPU/PMU、パフォーマンスモニタユニット）の機能とソフトウェア機能（ソフトウェアカウンタ、トレースポイント）をカバーしています。詳細な使用方法については、[perf Examples](http://www.brendangregg.com/perf.html#Background) を参照してください。

### BCC/bpftrace

CentOS 7.6から、LinuxカーネルはBerkeley Packet Filter（BPF）をサポートしています。したがって、[60秒で分析する](#60秒で分析する)の結果に基づいて、適切なツールを選択して詳細な分析を行うことができます。BPFはperf / ftraceと比較してプログラム可能でパフォーマンスオーバーヘッドが小さいです。また、BPFはkprobeよりも高いセキュリティを提供し、プロダクション環境に適しています。BCCツールキットの詳細な使用方法については、[BPF Compiler Collection（BCC）](https://github.com/iovisor/bcc/blob/master/README.md) を参照してください。

## パフォーマンスチューニング

このセクションでは、分類されたカーネルサブシステムに基づいたパフォーマンスチューニングを紹介します。

### CPU - 周波数スケーリング

`cpufreq`は、CPUの周波数を動的に調整するモジュールです。5つのモードをサポートしています。サービスのパフォーマンスを確保するために、パフォーマンスモードを選択し、CPUの周波数を最高サポートされる動作周波数で固定します。この操作のコマンドは、`cpupower frequency-set --governor performance`です。

### CPU - 割り込みアフィニティ

- `irqbalance`サービスを介して自動的なバランスを行うことができます。
- 手動でバランスを行う場合：
    - 割り込みをバランスする必要があるデバイスを特定します。CentOS 7.5以降、特定のデバイスとそのドライバ（たとえば `be2iscsi`ドライバやNVMeの設定を使用するデバイスなど）には、システムが最適な割り込みアフィニティを自動的に設定します。そのようなデバイスでは、このようなデバイスの割り込みアフィニティを手動で設定することはできなくなりました。
    - 他のデバイスの場合は、チップのマニュアルを確認して、これらのデバイスが割り込みを分散させるかどうかを確認します。
        - 分散しない場合、これらのデバイスのすべての割り込みは同じCPUにルーティングされ、変更することはできません。
        - 分散する場合、`smp_affinity`マスクを計算し、対応する設定ファイルを設定します。詳細については、[カーネルのドキュメント](https://www.kernel.org/doc/Documentation/IRQ-affinity.txt)を参照してください。

### NUMA CPUバインディング

可能な限り Non-Uniform Memory Access（NUMA）ノード間のメモリアクセスを避けるために、スレッド/プロセスを特定のCPUコアにバインドすることで、CPUのアフィニティを設定することができます。一般的なプログラムの場合、CPUバインディングには`numactl`コマンドを使用できます。詳細な使用方法については、Linuxのマニュアルページを参照してください。ネットワークインターフェースカード（NIC）の割り込みについては、[ネットワークチューニング](#ネットワークチューニング)を参照してください。

### メモリ - 透過的な巨大ページ（THP）

データベースアプリケーションでは、THP（Transparent Huge Page）を使用することは**推奨されません**。なぜなら、データベースは通常、連続したメモリアクセスパターンではなく疎なメモリアクセスパターンを持つためです。メモリの高レベルの断片化が深刻な場合、THPページの割り当て時にレイテンシが高くなります。THP用のダイレクトコンパクションが有効になっている場合、CPU使用率が急増します。そのため、THPを無効にすることをおすすめします。

```shell
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag
```

### メモリ - 仮想メモリパラメータ

- `dirty_ratio`パーセンテージ比率。総合的なディープページキャッシュの量がシステムメモリのこのパーセンテージ比率に達すると、システムは`pdflush`操作を使用してディープページキャッシュをディスクに書き込み始めます。`dirty_ratio`のデフォルト値は20%であり、通常は調整の必要はありません。NVMeデバイスなどの高性能SSDの場合、この値を下げることでメモリリクレームの効率が向上します。
- `dirty_background_ratio`パーセンテージ比率。総合的なディープページキャッシュの量がシステムメモリのこのパーセンテージ比率に達すると、システムはバックグラウンドでディープページキャッシュをディスクに書き込み始めます。`dirty_background_ratio`のデフォルト値は10%であり、通常は調整の必要はありません。NVMeデバイスなどの高性能SSDの場合、より低い値を設定することでメモリリクレームの効率が向上します。

### ストレージとファイルシステム

I/Oスタックリンクは長く、ファイルシステムレイヤ、ブロックデバイスレイヤ、およびドライバレイヤを含みます。

#### I/Oスケジューラ

I/Oスケジューラは、ストレージデバイスでのI/O操作がいつ実行され、どれだけの時間実行されるかを決定します。I/Oエレベーターとも呼ばれます。SSDデバイスの場合、I/Oスケジューリングポリシーを`noop`に設定することを推奨します。

```shell
echo noop > /sys/block/${SSD_DEV_NAME}/queue/scheduler
```

#### フォーマットパラメータ - ブロックサイズ

ブロックはファイルシステムの作業単位です。ブロックサイズは、1つのブロックに格納できるデータの量を決定し、したがって1回の書き込みまたは読み取りごとに書き込まれるまたは読み取られる最小データ量を決定します。

デフォルトのブロックサイズはほとんどのシナリオに適しています。ただし、ブロックサイズ（または複数のブロックのサイズ）が通常の読み書き時のデータ量と同じかわずかに大きい場合、ファイルシステムのパフォーマンスが向上し、データのストレージ効率が高くなります。小さなファイルはまだ1つのブロックを使用します。ファイルは複数のブロックに分散されることがありますが、これによりランタイムオーバーヘッドが増加します。

`mkfs`コマンドを使用してデバイスをフォーマットする場合は、ファイルシステムオプションの一部としてブロックサイズを指定します。ブロックサイズを指定するパラメータは、ファイルシステムによって異なります。詳細については、[mkfs.ext4のmanページ](https://manpages.debian.org/testing/e2fsprogs/mkfs.ext4.8.ja.html)などを参照してください。

#### `mount`パラメータ

`mount`コマンドで`noatime`オプションを有効にすると、ファイルが読み取られる際にメタデータの更新が無効になります。`nodiratime`の動作が有効な場合、ディレクトリが読み取られる際にメタデータの更新が無効になります。

### ネットワークチューニング

ネットワークサブシステムは、多くの異なる部分からなる繊細な接続で構成されています。CentOS 7のネットワークサブシステムは、ほとんどのワークロードに最適なパフォーマンスを提供するように設計されており、これらのワークロードのパフォーマンスを自動的に最適化します。そのため、通常はネットワークのパフォーマンスを手動で調整する必要はありません。

ネットワークの問題は通常、ハードウェアまたは関連するデバイスの問題によって引き起こされます。そのため、プロトコルスタックのチューニングを行う前に、ハードウェアの問題が原因でないかを確認してください。

ネットワークスタックは主に自己最適化していますが、ネットワークパケット処理の以下の側面はボトルネックとなり、パフォーマンスに影響を与える可能性があります。

- NICハードウェアキャッシュ：ハードウェアレベルでのパケットロスを正確に観察するには、`ethtool -S ${NIC_DEV_NAME}`コマンドを使用して`drops`フィールドを観察します。パケットロスが発生する場合、ハード/ソフト割り込みの処理速度がNICの受信速度に追い付いていない可能性があります。受信バッファサイズが上限よりも小さい場合、パケットロスを避けるためにRXバッファを増やすこともできます。クエリコマンドは：`ethtool -g ${NIC_DEV_NAME}`であり、変更コマンドは`ethtool -G ${NIC_DEV_NAME}`です。
- ハードウェア割り込み：NICがReceive-Side Scaling（RSS、またはマルチNIC受信）機能をサポートしている場合、`/proc/interrupts`のNIC割り込みを観察します。割り込みが不均一である場合は、[CPU - 周波数スケーリング](#CPU---周波数スケーリング)、[CPU - 割り込みアフィニティ](#CPU---割り込みアフィニティ)、および[Numa CPUバインディング](#NUMA CPUバインディング)を参照してください。NICがRSSをサポートしていない場合、またはRSSの数が物理CPUコアの数よりもはるかに少ない場合は、受信パケットステアリング（RSSのソフトウェア実装と見なされる）およびRPS拡張の受信フローステアリング（RFS）を構成します。詳細な構成については、[カーネルのドキュメント](https://www.kernel.org/doc/Documentation/networking/scaling.txt)を参照してください。
- ソフトウェア割り込み: `/proc/net/softnet_stat` の監視を行います。3番目の列以外の値が増加している場合は、`net.core.netdev_budget` または `net.core.dev_weight` の値を適切に調整して `softirq` によるCPU時間を増やします。さらに、CPU使用率をチェックして、CPUを頻繁に使用しているタスクやそれらを最適化できるかを検討する必要があります。

- アプリケーションソケットの受信キュー: `ss -nmp` の`Resv-q` 列を監視します。キューが一杯の場合は、アプリケーションソケットのキャッシュサイズを増やすか、自動キャッシュ調整メソッドを使用することを検討してください。さらに、アプリケーション層のアーキテクチャを最適化し、ソケットの読み取り間隔を短縮できるか検討してください。

- イーサネットフロー制御: NIC とスイッチがフロー制御機能をサポートしている場合、この機能を使用して、NIC キュー内のデータを処理するための時間を確保し、NIC バッファのオーバーフロー問題を回避することができます。

- 割り込みの統合: 頻繁なハードウェア割り込みはシステムのパフォーマンスを低下させ、遅れたハードウェア割り込みはパケットロスを引き起こします。新しいNICは、割り込み統合機能をサポートしており、ドライバがハードウェア割り込みの数を自動的に調整することができます。`ethtool -c ${NIC_DEV_NAME}` を実行してこの機能をチェックし、`ethtool -C ${NIC_DEV_NAME}`を実行して有効にします。適応モードでは、NIC は割り込みの統合を自動的に調整します。このモードでは、ドライバがトラフィックモードとカーネル受信モードをチェックし、パケットロスを防ぐためにリアルタイムで統合設定を評価します。異なるブランドのNICにはさまざまな機能とデフォルトの設定があります。詳細については、NIC のマニュアルを参照してください。

- アダプターキュー: プロトコルスタックを処理する前に、カーネルはこのキューを使用してNICで受信したデータをバッファリングし、各CPUにはバックログキューがあります。このキューにキャッシュできるパケットの最大数は `netdev_max_backlog` です。 `/proc/net/softnet_stat` の2番目の列を監視します。行の2番目の列が増加し続けると、CPU [行-1] のキューが一杯でデータパケットが失われていることを意味します。この問題を解決するために、`net.core.netdev_max_backlog` の値を倍にして続けます。

- 送信キュー: 送信キューの長さは、送信前にキューに並べられるパケット数を決定します。デフォルト値は `1000` であり、これは 10 Gbps に十分です。しかし、`ip -s link` の出力から TX エラーの値を観測した場合には、値を倍にしてみてください： `ip link set dev ${NIC_DEV_NAME} txqueuelen 2000`。

- ドライバ: NIC ドライバは通常、チューニングパラメータを提供しています。デバイスのハードウェアマニュアルおよびそのドライバのドキュメントを参照してください。