---
title: TiDB ダッシュボードのインスタンスプロファイリング - マニュアルプロファイリング
summary: 複雑な問題を分析するためのパフォーマンスデータの収集方法を学びます。
aliases: ['/docs/dev/dashboard/dashboard-profiling/']
---

# TiDB ダッシュボードのインスタンスプロファイリング - マニュアルプロファイリング

> **注意:**
>
> この機能はデータベースの専門家を対象としています。専門外のユーザーは、PingCAP のテクニカルサポートの指導のもとでこの機能を使用することをお勧めします。

マニュアルプロファイリングを使用すると、TiDB、TiKV、PD、および TiFlash の各インスタンスに対して、1回のクリックで現在のパフォーマンスデータを即座に収集することができます。収集されたパフォーマンスデータは、FlameGraph または DAG として可視化することができます。

これらのパフォーマンスデータにより、専門家は、インスタンスの CPU およびメモリなどの現在のリソース消費の詳細を分析し、高い CPU オーバーヘッド、高いメモリ使用量、およびプロセスの停止などの複雑な現在のパフォーマンスの問題を特定するのに役立ちます。

プロファイリングを開始すると、TiDB ダッシュボードは、一定期間（デフォルトで 30 秒）の間、現在のパフォーマンスデータを収集します。したがって、この機能はクラスタが現在直面している問題を分析するためにのみ使用でき、過去の問題にはほとんど影響を与えません。いつでもパフォーマンスデータを収集して分析するには、[連続的なプロファイリング](/dashboard/continuous-profiling.md)を参照してください。

## サポートされているパフォーマンスデータ

現在サポートされている次のパフォーマンスデータ:

- CPU: TiDB、TiKV、PD、TiFlash インスタンスの各内部関数の CPU オーバーヘッド

  > TiKV および TiFlash インスタンスの CPU オーバーヘッドは ARM アーキテクチャでは現在サポートされていません。

- Heap: TiDB、TiKV、PD インスタンスの各内部関数のメモリ消費

- Mutex: TiDB および PD インスタンスのミューテックス競合状態

- Goroutine: TiDB および PD インスタンスのすべてのゴルーチンの実行状態とコールスタック

## ページへのアクセス

次のいずれかの方法を使用して、インスタンスのプロファイリングページにアクセスできます:

* TiDB ダッシュボードにログインした後、左側のナビゲーションメニューで **Advanced Debugging** > **Profiling Instances** > **Manual Profiling** をクリックします。

  ![インスタンスのプロファイリングページにアクセス](/media/dashboard/dashboard-profiling-access.png)

* ブラウザで <http://127.0.0.1:2379/dashboard/#/instance_profiling> を開きます。`127.0.0.1:2379` を実際の PD インスタンスのアドレスとポートに置き換えます。

## プロファイリングの開始

インスタンスのプロファイリングページで、少なくとも1つのターゲットインスタンスを選択し、**Start Profiling** をクリックしてインスタンスのプロファイリングを開始します。

![インスタンスのプロファイリングの開始](/media/dashboard/dashboard-profiling-start.png)
  
プロファイリングを開始する前に、プロファイリングの期間を変更することができます。この期間は、デフォルトで30秒かかるプロファイリングに必要な時間によって決まります。

マニュアルプロファイリングは、[連続的なプロファイリング](/dashboard/continuous-profiling.md)が有効になっているクラスタで開始することはできません。現在の瞬間のパフォーマンスデータを表示するには、[連続的なプロファイリングページ](/dashboard/continuous-profiling.md#access-the-page)で最新のプロファイリング結果をクリックします。

## プロファイリングの状態を表示

プロファイリングが開始されると、リアルタイムでプロファイリングの状態と進捗状況を表示できます。

![プロファイリングの詳細](/media/dashboard/dashboard-profiling-view-progress.png)

プロファイリングはバックグラウンドで実行されます。ページをリフレッシュしたり、現在のページから離れても、実行中のプロファイリングタスクは停止しません。

## パフォーマンスデータのダウンロード

すべてのインスタンスのプロファイリングが完了すると、右上隅の **Download Profiling Result** をクリックして、すべてのパフォーマンスデータをダウンロードできます。

![プロファイリング結果のダウンロード](/media/dashboard/dashboard-profiling-download.png)

また、テーブル内の個々のインスタンスをクリックしてプロファイリング結果を表示することもできます。または、... 上にカーソルを合わせて生データをダウンロードすることもできます。

![単一のインスタンス結果](/media/dashboard/dashboard-profiling-view-single.png)

## プロファイリング履歴の表示

オンデマンドプロファイリングの履歴がページにリストされます。行をクリックして詳細を表示します。

![プロファイリングの履歴を表示](/media/dashboard/dashboard-profiling-history.png)

プロファイリングの状態ページの詳細な操作については、[プロファイリングの状態の表示](#view-profiling-status)を参照してください。