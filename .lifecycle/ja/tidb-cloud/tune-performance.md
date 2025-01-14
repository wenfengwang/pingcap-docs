---
title: パフォーマンスの分析とチューニング
summary: TiDB Cloud クラスターのパフォーマンスを分析およびチューニングする方法について学びます。

# パフォーマンスの分析とチューニング

TiDB Cloud では [スロークエリ](#slow-query)、[SQL ステートメントの分析](#statement-analysis)、[Key Visualizer](#key-visualizer)、および [インデックスインサイト (ベータ版)](#index-insight-beta) が利用可能で、クラスターのパフォーマンスを分析できます。

- スロークエリ を使用すると、TiDB クラスター内のすべての遅いクエリを検索して表示し、それぞれの遅いクエリのボトルネックを実行計画や SQL 実行情報などの詳細を確認することで探ることができます。

- ステートメント分析 を使用すると、ページ上で SQL 実行を直接観察し、システムテーブルをクエリすることなく簡単にパフォーマンスの問題を特定することができます。

- Key Visualizer を使用すると、TiDB のデータアクセスパターンやデータホットスポットを観察することができます。

- インデックスインサイト を使用すると、有意義で実行可能なインデックスの推奨事項が提供されます。

> **注意:**
>
> 現在、 **Key Visualizer** および **インデックスインサイト (ベータ版)** は [TiDB サーバーレス](/tidb-cloud/select-cluster-tier.md#tidb-serverless) クラスターで利用できません。

## スロークエリ

デフォルトでは、300ミリ秒以上かかる SQL クエリをスロークエリと見なします。

クラスターでスロークエリを表示するには、次の手順を実行します。

1. クラスターの **診断** ページに移動します。

2. **スロークエリ** タブをクリックします。

3. リスト内のスロークエリをクリックして、その詳細な実行情報を表示します。

4. (オプション) 対象の時間範囲、関連するデータベース、および SQL キーワードに基づいてスロークエリをフィルタリングしたり、表示するスロークエリの数を制限したりすることができます。

結果は表の形式で表示され、異なる列で結果を並べ替えることができます。

詳細については、[TiDB ダッシュボードのスロークエリ](https://docs.pingcap.com/tidb/stable/dashboard-slow-query) を参照してください。

## ステートメント分析

ステートメント分析を使用するには、次の手順を実行します。

1. クラスターの **診断** ページに移動します。

2. **SQL ステートメント** タブをクリックします。

3. 時間間隔ボックスで分析される期間を選択します。その後、この期間のすべてのデータベースの SQL ステートメントの実行統計情報を取得できます。

4. (オプション) 特定のデータベースに関心がある場合は、次のボックスで対応するスキーマを選択して結果をフィルタリングできます。

結果は表の形式で表示され、異なる列で結果を並べ替えることができます。

詳細については、[TiDB ダッシュボードのステートメント詳細](https://docs.pingcap.com/tidb/stable/dashboard-statement-details) を参照してください。

## Key Visualizer

> **注意:**
>
> Key Visualizer は [TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスターでのみ利用可能です。

キーの分析を表示するには、次の手順を実行します。

1. クラスターの **診断** ページに移動します。

2. **Key Visualizer** タブをクリックします。

**Key Visualizer** ページでは、大きなヒートマップが時間の経過とともにアクセストラフィックの変化を示します。ヒートマップの各軸に沿った平均値は下部および右側に表示されます。左側にはテーブル名、インデックス名などの情報が表示されます。

詳細については、[Key Visualizer](https://docs.pingcap.com/tidb/stable/dashboard-key-visualizer) を参照してください。

## インデックスインサイト (ベータ版)

TiDB Cloud の インデックスインサイト 機能は、効果的にインデックスが活用されていない遅いクエリのための推奨されるインデックスを提供することで、クエリのパフォーマンスを最適化する強力な機能を提供します。

> **注意:**
>
> インデックスインサイト は現在ベータ版であり、[TiDB Dedicated](/tidb-cloud/select-cluster-tier.md#tidb-dedicated) クラスターのみで利用可能です。

詳細については、[インデックスインサイト](/tidb-cloud/index-insight.md) を参照してください。