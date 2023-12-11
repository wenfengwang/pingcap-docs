---
title: TiFlash パイプライン実行モデル
summary: TiFlashのパイプライン実行モデルについて学びます。

# TiFlash パイプライン実行モデル

このドキュメントは、TiFlashのパイプライン実行モデルを紹介します。

v7.2.0から、TiFlashは新しい実行モデルであるパイプライン実行モデルをサポートしています。

- v7.2.0とv7.3.0向け: パイプライン実行モデルは実験的な機能であり、[`tidb_enable_tiflash_pipeline_model`](https://docs.pingcap.com/tidb/v7.2/system-variables#tidb_enable_tiflash_pipeline_model-introduced-since-v720)で制御されます。
- v7.4.0以降: パイプライン実行モデルは一般に利用可能となります。TiFlashの内部機能であり、TiFlashリソース制御と密接に統合されています。TiFlashリソース制御を有効にすると、パイプライン実行モデルも自動的に有効になります。TiFlashリソース制御の使用方法については、[リソース制御を使用してリソース分離を実現する](/tidb-resource-control.md#parameters-for-resource-control)を参照してください。また、v7.4.0からは、システム変数 `tidb_enable_tiflash_pipeline_model` は非推奨となります。

論文[Morsel-Driven Parallelism: A NUMA-Aware Query Evaluation Framework for the Many-Core Age](https://dl.acm.org/doi/10.1145/2588555.2610507)の影響を受けて、TiFlashパイプライン実行モデルは伝統的なスレッドスケジューリングモデルとは異なる、細かいタスクスケジューリングモデルを提供します。これにより、オペレーティングシステムのスレッドアプリケーションにおけるoverhead が削減され、細かいスケジューリングメカニズムが提供されます。

## 設計と実装

元のTiFlashストリームモデルはスレッドスケジューリング実行モデルです。各クエリは独立していくつかのスレッドを実行するように申請します。

スレッドスケジューリングモデルには以下の2つの欠点があります:

- 高並列のシナリオでは、多くのスレッドが多数のコンテキストスイッチを引き起こし、高いスレッドスケジューリングコストにつながります。
- スレッドスケジューリングモデルでは、クエリのリソース使用量を正確に測定することも、細かいリソース制御もできません。

新しいパイプライン実行モデルは以下の最適化を行います:

- クエリは複数のパイプラインに分割され、順次実行されます。各パイプラインでは、できる限りデータブロックをキャッシュに保持して、より良い時間的局所性を実現し、全体の実行プロセスの効率を向上させます。
- オペレーティングシステムのネイティブスレッドスケジューリングモデルを取り除き、より細かいスケジューリングメカニズムを実装するために、各パイプラインをいくつかのタスクにインスタンス化し、タスクスケジューリングモデルを使用します。同時に、固定スレッドプールを使用して、オペレーティングシステムのスレッドスケジューリングのオーバーヘッドを削減します。

パイプライン実行モデルのアーキテクチャは以下のようになっています:

![TiFlash パイプライン実行モデルの設計](/media/tiflash/tiflash-pipeline-model.png)

前の図に示すように、パイプライン実行モデルには以下の2つの主要コンポーネントがあります: パイプラインクエリ実行器とタスクスケジューラ。

- パイプラインクエリ実行器

    パイプラインクエリ実行器は、TiDBノードから送信されたクエリリクエストを、パイプライン有向非循環グラフ(DAG)に変換します。

    クエリ内のパイプラインブレーカーオペレータを見つけ、パイプラインブレーカーに応じてクエリを複数のパイプラインに分割します。その後、パイプラインを依存関係に従ってDAGに組み立てます。

    パイプラインブレーカーは、一時停止/ブロッキングロジックを持つオペレータです。このタイプのオペレータは、上流オペレータからデータブロックを連続して受け取り、すべてのデータブロックが受信された後に下流オペレータに処理結果を返します。このタイプのオペレータはデータ処理パイプラインを中断するため、パイプラインブレーカーと呼ばれます。その一例として、集約オペレータがあります。これは、上流オペレータのすべてのデータをハッシュテーブルに書き込み、ハッシュテーブル内のデータを計算して下流オペレータに結果を返します。

    クエリがパイプラインDAGに変換されると、パイプラインクエリ実行器は依存関係に従って各パイプラインを順次実行します。クエリの並行性に応じて、パイプラインは複数のタスクにインスタンス化され、それらはタスクスケジューラに提出されます。

- タスクスケジューラ

    タスクスケジューラは、パイプラインクエリ実行器によって提出されたタスクを実行します。タスクは異なる実行ロジックに応じて、タスクスケジューラ内の異なるコンポーネント間で動的に切り替わります。

    - CPUタスクスレッドプール

        タスク内のCPU集中型計算ロジック（データのフィルタリングや関数計算など）を実行します。

    - IOタスクスレッドプール

        タスク内のIO集中型計算ロジック（中間結果のディスクへの書き込みなど）を実行します。

    - ウェイトリアクター

        タスク内のウェイトロジック（ネットワークレイヤーが計算レイヤーにデータパケットを転送するための待機など）を実行します。