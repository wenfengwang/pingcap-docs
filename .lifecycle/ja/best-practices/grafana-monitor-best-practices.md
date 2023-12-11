---
title: Grafanaを使用してTiDBを監視するための最良の方法
summary: TiDBを監視するためのGrafanaを効果的に使用するための7つのヒントを学ぶ
aliases: ['/docs/dev/best-practices/grafana-monitor-best-practices/','/docs/dev/reference/best-practices/grafana-monitor/']
---

# Grafanaを使用してTiDBを監視するためのベストプラクティス

[TiUPを使用してTiDBクラスタを展開](/production-deployment-using-tiup.md)し、トポロジ設定にGrafanaとPrometheusを追加した場合、TiDBクラスタには同時に[Grafana + Prometheus監視プラットフォーム](/tidb-monitoring-framework.md)が展開されます。これにより、TiDBクラスタ内のさまざまなコンポーネントやマシンのメトリクスを収集および表示するためのドキュメントが提供されます。このドキュメントでは、Grafanaを使用したTiDBの監視のためのベストプラクティスについて説明します。これにより、メトリクスを使用してTiDBクラスタの状態を分析し、問題を診断するのに役立ちます。

## 監視アーキテクチャ

[Prometheus](https://prometheus.io/)は多次元データモデルと柔軟なクエリ言語を備えた時系列データベースです。[Grafana](https://grafana.com/)はメトリクスを分析および視覚化するためのオープンソースの監視システムです。

![TiDBクラスタの監視アーキテクチャ](/media/prometheus-in-tidb.png)

TiDB 2.1.3以降のバージョンでは、TiDBの監視がプル方式をサポートしています。この調整には、次の利点があります。

- Prometheusを移行する必要がある場合、TiDBクラスタ全体を再起動する必要がありません。以前の調整では、目標アドレスを更新するためにクラスタ全体を再起動する必要がありました。
- 単一障害点の監視を防ぐために、高可用性でないGrafana + Prometheus監視プラットフォームを2つ別々に展開できます。
- 単一障害点となりうるPushgatewayが取り除かれます。

## 監視データのソースと表示

TiDBの3つのコアコンポーネント（TiDBサーバー、TiKVサーバー、PDサーバー）は、HTTPインターフェースを介してメトリクスを取得します。これらのメトリクスはプログラムコードから収集され、次のポートを介して取得されます。

| コンポーネント  | ポート  |
| :---------- |:----- |
| TiDBサーバー | 10080 |
| TiKVサーバー | 20181 |
| PDサーバー   | 2379  |

以下のコマンドを実行して、HTTPインターフェースを介してSQLステートメントのQPSを確認します。TiDBサーバーを例に挙げます:

{{< copyable "shell-regular" >}}

```bash
curl http://__tidb_ip__:10080/metrics |grep tidb_executor_statement_total
```

```
# 異なる種類のSQLステートメントのリアルタイムQPSを確認します。以下の数値は、カウンタータイプの累積値（指数表記）です。
tidb_executor_statement_total{type="Delete"} 520197
tidb_executor_statement_total{type="Explain"} 1
tidb_executor_statement_total{type="Insert"} 7.20799402e+08
tidb_executor_statement_total{type="Select"} 2.64983586e+08
tidb_executor_statement_total{type="Set"} 2.399075e+06
tidb_executor_statement_total{type="Show"} 500531
tidb_executor_statement_total{type="Use"} 466016
```

上記のデータはPrometheusに保存され、Grafanaで表示されます。パネルを右クリックし、次に以下の図に示すように**Edit**ボタン（または<kbd>E</kbd>キー）をクリックします:

![メトリクスタブの編集エントリ](/media/best-practices/metric-board-edit-entry.png)

**Edit**ボタンをクリックすると、Metricsタブに`tidb_executor_statement_total`メトリック名が表示されたクエリ式が表示されます。パネル上のいくつかのアイテムの意味は次のとおりです:

- `rate[1m]`: 1分間の成長率。カウンタータイプのデータにのみ使用できます。
- `sum`: 値の合計。
- `by type`: 元のメトリックの値を種類別にグループ化します。
- `Legend format`: メトリック名のフォーマット。
- `Resolution`: ステップ幅はデフォルトで15秒です。解像度は複数のピクセルに1つのデータポイントを生成するかどうかを意味します。

**Metrics**タブのクエリ式は次のとおりです:

![Metricsタブのクエリ式](/media/best-practices/metric-board-expression.jpeg)

Prometheusは多くのクエリ式と関数をサポートしています。詳細については、[Prometheus公式ウェブサイト](https://prometheus.io/docs/prometheus/latest/querying)を参照してください。

## Grafanaのヒント

このセクションでは、TiDBのメトリクスを効果的に監視および分析するためのGrafanaの7つのヒントを紹介します。

### ヒント1: すべての次元を確認し、クエリ式を編集する

[監視データのソースと表示](#source-and-display-of-monitoring-data)セクションで示されている例では、データは種類別にグループ化されています。他の次元でグループ化できるかどうかを素早く確認したい場合は、次の方法を使用できます: **クエリ式では、計算を行わずにメトリック名のみを残し、`Legend format`フィールドを空白のままにします**。これにより、元のメトリクスが表示されます。例えば、次の図では3つの次元（`instance`、`job`、`type`）があることがわかります:

![クエリ式を編集し、すべての次元を確認](/media/best-practices/edit-expression-check-dimensions.jpg)

その後、`type`の後に`instance`次元を追加し、`Legend format`フィールドに`{{instance}}`を追加してクエリ式を修正できます。これにより、各TiDBサーバーで実行される異なる種類のSQLステートメントのQPSを確認できます:

![クエリ式にインスタンス次元を追加](/media/best-practices/add-instance-dimension.jpeg)

### ヒント2: Y軸のスケールを切り替える

クエリの実行時間を例に取ると、Y軸はデフォルトで2進対数スケール（log<sub>2</sub>n）になっており、表示のギャップが狭まります。変更を強調するために、これを線形スケールに切り替えることができます。次の2つの図を比較すると、表示の違いが簡単にわかり、SQLステートメントの実行が遅くなった時点を特定できます。

もちろん、線形スケールはすべての状況に適しているわけではありません。例えば、1か月間の実行時間のパフォーマンストレンドを観察する場合、線形スケールではノイズが発生する可能性があり、観察が難しくなります。

Y軸はデフォルトで2進対数スケールを使用します:

![Y軸は2進対数スケールを使用します](/media/best-practices/default-axes-scale.jpg)

Y軸を線形スケールに切り替えます:

![線形スケールに切り替える](/media/best-practices/axes-scale-linear.jpg)

> **ヒント:**
>
> ヒント2とヒント1を組み合わせると、`sql_type`次元を見つけて、`SELECT`ステートメントまたは`UPDATE`ステートメントが遅いかどうかを分析したり、遅いSQLステートメントのインスタンスを特定したりできます。

### ヒント3: Y軸の基準を変更して変更を強調する

線形スケールに切り替えても、トレンドが見えない場合があります。たとえば、次の図では、クラスタのスケールを変更した後の`ストアサイズ`のリアルタイムな変化を観察したい場合、大きな基準値のため、小さな変化が見えません。このような場合は、Y軸の基準を`0`から`auto`に変更して、上部を拡大することができます。以下の2つの図を確認すると、データ移行が開始されていることがわかります。

基準値はデフォルトで`0`です:

![基準値はデフォルトで`0`](/media/best-practices/default-y-min.jpeg)

基準値を`auto`に変更します:

![基照準を`auto`に変更します](/media/best-practices/y-min-auto.jpg)

### ヒント4: 共有クロスヘアまたはツールチップを使用する

**Settings**パネルには、**Graph Tooltip**パネルオプションがあり、デフォルトでは**Default**に設定されています。

![グラフィックプレゼンテーションツール](/media/best-practices/graph-tooltip.jpeg)

それぞれ**Shared crosshair**と**Shared Tooltip**を使用して効果をテストできます。そのため、2つのメトリクスの相関関係を確認する際に便利な、スケールがリンクされた表示となります。

グラフィックプレゼンテーションツールを**Shared crosshair**に設定:

![グラフィックプレゼンテーションツールを**Shared crosshair**に設定](/media/best-practices/graph-tooltip-shared-crosshair.jpeg)

グラフィックプレゼンテーションツールを**Shared Tooltip**に設定:

![グラフィックプレゼンテーションツールを**Shared Tooltip**に設定](/media/best-practices/graph-tooltip-shared-tooltip.jpg)

### ヒント5: `IPアドレス:ポート番号`を入力して過去のメトリクスを確認する

PDのダッシュボードは、現在のリーダーのメトリクスのみを表示します。過去のPDリーダーの状態を確認したい場合で、`instance`フィールドのドロップダウンリストに存在しない場合は、`IPアドレス:2379`を手動で入力してリーダーのデータを確認できます。

![過去のメトリクスを確認する](/media/best-practices/manually-input-check-metric.jpeg)

### ヒント6: `Avg`関数を使用する

通常、デフォルトでは`Max`と`Current`の関数のみが凡例で利用できます。メトリクスが大きく変動する場合は、`Avg`関数などの他のサマリー関数を凡例に追加して、時間の経過に伴う全体のトレンドを確認できます。

`Avg`関数などのサマリー関数を追加します:

![`Avg`などのサマリー関数を追加](/media/best-practices/add-avg-function.jpeg)

その後、全体的なトレンドを確認します:

![平均関数を追加して全体のトレンドをチェックします](/media/best-practices/add-avg-function-check-trend.jpg)

### Tip 7: PrometheusのAPIを使用してクエリ式の結果を取得する

GrafanaはPrometheusのAPIを使用してデータを取得し、このAPIを使用して情報を取得することができます。さらに、以下のような使い方もできます。

- クラスターのサイズや状態などの情報を自動的に取得します。
- 報告書用に式をわずかに変更して情報を提供し、例えば、1日あたりの合計QPSの数、1日あたりの最大QPSのピーク値、および1日あたりの応答時間をカウントします。
- 重要な指標の定期的な健康診断を実行します。

PrometheusのAPIは以下のように表示されます：

![PrometheusのAPI](/media/best-practices/prometheus-api-interface.jpg)

{{< copyable "shell-regular" >}}

```bash
curl -u user:pass 'http://__grafana_ip__:3000/api/datasources/proxy/1/api/v1/query_range?query=sum(tikv_engine_size_bytes%7Binstancexxxxxxxxx20181%22%7D)%20by%20(instance)&start=1565879269&end=1565882869&step=30' |python -m json.tool
```

```
{
    "data": {
        "result": [
            {
                "metric": {
                    "instance": "xxxxxxxxxx:20181"
                },
                "values": [
                    [
                        1565879269,
                        "1006046235280"
                    ],
                    [
                        1565879299,
                        "1006057877794"
                    ],
                    [
                        1565879329,
                        "1006021550039"
                    ],
                    [
                        1565879359,
                        "1006021550039"
                    ],
                    [
                        1565882869,
                        "1006132630123"
                    ]
                ]
            }
        ],
        "resultType": "matrix"
    },
    "status": "success"
}
```

## サマリー

Grafana + Prometheusモニタリングプラットフォームは非常に強力なツールです。これをうまく活用することで、TiDBクラスターの状態を分析するための多くの時間を節約し、効率を向上させることができます。さらに重要なのは、問題の診断を支援することができます。このツールは特に大量のデータがある場合にTiDBクラスターの運用とメンテナンスに非常に役立ちます。