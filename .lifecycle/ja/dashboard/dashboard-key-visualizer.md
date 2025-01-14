---
title: キービジュアライザーページ
summary: トラフィックを監視するためのキービジュアライザーの使用方法を学ぶ
aliases: ['/docs/dev/dashboard/dashboard-key-visualizer/','/docs/dev/key-visualizer-monitoring-tool/']
---

# キービジュアライザーページ

TiDBダッシュボードのキービジュアライザーページは、TiDBの使用状況を分析し、トラフィックのホットスポットをトラブルシューティングするために使用されます。このページは、TiDBクラスターのトラフィックを時間経過で視覚的に表示します。

## キービジュアライザーページへのアクセス

以下の2つの方法のいずれかを使用してキービジュアライザーページにアクセスできます:

* TiDBダッシュボードにログインした後、左のナビゲーションメニューで **キービジュアライザー** をクリックします。

    ![キービジュアライザーへのアクセス](/media/dashboard/dashboard-keyviz-access-v650.png)

* ブラウザで <http://127.0.0.1:2379/dashboard/#/keyviz> を訪れます。`127.0.0.1:2379` を実際のPDインスタンスのアドレスとポートに置き換えてください。

## インターフェースのデモンストレーション

次の画像はキービジュアライザーページのデモンストレーションです:

![キービジュアライザーページ](/media/dashboard/dashboard-keyviz-overview.png)

前述のインターフェースから、以下のオブジェクトが見えます:

+ 時間経過に伴う全体のトラフィックの変化を示す大きなヒートマップ。
+ 特定の座標点の詳細情報。
+ ヒートマップの左側にあるテーブルとインデックスの情報。

## 基本的な概念

この章では、キービジュアライザーに関連する基本的な概念を紹介します。

### リージョン

TiDBクラスターでは、格納されているデータはTiKVインスタンスに分散されています。論理的に、TiKVは巨大で整然としたキー・値マップです。キー・値スペース全体は多くのセグメントに分割され、各セグメントは隣接したキーから構成されます。このようなセグメントを `リージョン` と呼びます。

リージョンの詳細な説明については、[TiDBインターナル（I）- データストレージ](https://en.pingcap.com/blog/tidb-internal-data-storage/)を参照してください。

### ホットスポット

TiDBデータベースを使用する際、高トラフィックがデータの小さな範囲に集中する典型的なホットスポット問題が発生します。連続したデータ範囲はしばしば同じTiKVインスタンスで処理されるため、ホットスポットの発生したTiKVインスタンスは全体のアプリケーションのパフォーマンスのボトルネックとなります。ホットスポットの問題は、次のシナリオでよく発生します:

+ `AUTO_INCREMENT`プライマリキーを持つテーブルに隣接したデータを書き込むと、このテーブルでホットスポットの問題が発生します。
+ テーブルの時間インデックスに隣接した時間データを書き込むと、テーブルインデックスでホットスポットの問題が発生します。

ホットスポットについての詳細は、[高並行書き込みベストプラクティス](/best-practices/high-concurrency-best-practices.md#hotspot-causes)を参照してください。

### ヒートマップ

ヒートマップはキービジュアライザーのコアパートであり、その指標の時間経過を示します。ヒートマップのX軸は時間を示し、Y軸は連続したリージョンを示し、これによりTiDBクラスターのすべてのスキーマとテーブルをカバーするキー範囲に基づいています。

ヒートマップの寒色は、その期間の間のリージョンの低い読み書きトラフィックを示します。暖かい（明るい）色はより高いトラフィックを示します。

### リージョンの圧縮

TiDBクラスターには何十万ものリージョンがある可能性があります。これほど多くのリージョンを画面に表示するのは難しいため、これらのリージョンは各ヒートマップで1,500の連続した範囲、それぞれを `バケツ` と呼ばれるものに圧縮されます。ヒートマップでは、より注目されるようなホットなインスタンスでは、低いトラフィックの多くのリージョンを1つのバケツに圧縮し、また、より高いトラフィックのリージョンも1つのバケツに表示します。

## キービジュアライザーの使用

このセクションでは、キービジュアライザーの使用方法について紹介します。

### 設定

初めてキービジュアライザーページを使用するには、**設定** ページでこの機能を手動で有効にする必要があります。ページのガイドに従って **設定を開く** をクリックして設定ページを開きます:

![機能が無効](/media/dashboard/dashboard-keyviz-not-enabled.png)

この機能を有効にした後、右上隅の **設定** アイコンをクリックして設定ページを開くことができます:

![設定アイコン](/media/dashboard/dashboard-keyviz-settings-button.png)

設定ページは以下のように表示されます:

![設定ページ](/media/dashboard/dashboard-keyviz-settings.png)

スイッチを介してデータ収集を開始するかどうかを設定し、**保存** をクリックして有効にします。機能を有効にした後、ツールバーが使用可能になります:

![ツールバー](/media/dashboard/dashboard-keyviz-toolbar.png)

この機能を有効にした後、データの収集がバックエンドで行われています。まもなくヒートマップを表示することができます。

### 特定の時間帯またはリージョン範囲を観察する

キービジュアライザーを開くと、デフォルトで最近の6時間のデータベース全体のヒートマップが表示されます。このヒートマップでは、右側（現在の時間）に近づくほど、各バケツの時間間隔が短くなります。特定の時間帯や特定のリージョン範囲を観察したい場合は、詳細を確認するためにズームインできます。具体的な手順は次の通りです:

1. ヒートマップ内を上下にスクロールします。
2. 次のボタンのいずれかをクリックして範囲を選択し、ズームインします。
    + **選択してズーム** ボタンをクリックします。その後、このボタンをクリックして範囲を選択してズームインします。

    ![選択ボックス](/media/dashboard/dashboard-keyviz-select-zoom.gif)

    + **リセット** ボタンをクリックしてリージョン範囲をデータベース全体にリセットします。
    + 前述のインターフェースの `6時間` の位置にある **時間の選択ボックス** をクリックして、再び観察する時間帯を選択します。

> **注意:**
>
> 上記の手順2に従うと、ヒートマップが再描画されます。これは元のヒートマップとは大きく異なる場合があります。この違いは正常です。なぜなら、より詳細に観察すると、リージョンの圧縮の粒度が変わったり、特定範囲の `hot` の基準が変わったりしたからです。

### 明るさの調整

ヒートマップでは、異なる明るさの色を使用してバケツのトラフィックを示します。ヒートマップの寒色は、その期間の間のリージョンの低い読み書きトラフィックを示します。暖かい（明るい）色はより高いトラフィックを示します。色があまりにも寒いか暑すぎる場合、細かく観察することが難しいです。このような状況では、**明るさ** ボタンをクリックして、ページの明るさを調整することができます。

> **注意:**
>
> キービジュアライザーが特定のエリアのヒートマップを表示する際、このエリアのトラフィックに基づいて冷たくて暑い基準を定義します。エリア全体のトラフィックが比較的均一な場合、値が低い場合でも、大きな明るいエリアが表示される可能性があります。分析にその値を含めることを忘れないでください。

### メトリクスの選択

![メトリクスの選択](/media/dashboard/dashboard-keyviz-select-type.png)

**メトリクスの選択ボックス** （前述のインターフェースで `Write (bytes)` の位置にある）で興味のあるメトリクスを選択することができます:

* `Read (bytes)`: 読み込みトラフィック。
* `Write (bytes)`: 書き込みトラフィック。
* `Read (keys)`: 読み込まれた行の数。
* `Write (keys)`: 書き込まれた行の数。
* `すべて`: 読み込みと書き込みトラフィックの合計。

### リフレッシュと自動リフレッシュ

現在の時間を基準にヒートマップを再取得するには、**リフレッシュ** ボタンをクリックします。データベースのトラフィック分布をリアルタイムで観察する必要がある場合は、**リフレッシュ** ボタンの右側の下矢印をクリックし、ヒートマップをこの間隔で自動的にリフレッシュする固定時間間隔を選択します。

> **注意:**
>
> 時間範囲やリージョン範囲を調整した場合、自動リフレッシュは無効になります。

### バケツの詳細を確認

興味のあるバケツにマウスを重ねると、このリージョン範囲の詳細情報が表示されます。以下の画像はこの情報の例です:

![バケツの詳細](/media/dashboard/dashboard-keyviz-tooltip.png)

この情報をコピーしたい場合は、バケツをクリックします。その後、関連する詳細が一時的に固定されたページに表示されます。情報をクリックすると、クリップボードにコピーされます。

![バケツの詳細をコピー](/media/dashboard/dashboard-keyviz-tooltip-copy.png)

## 一般的なヒートマップのタイプ

この章では、キービジュアライザーで見られる4つの一般的なヒートマップタイプを表示および解釈します。

### 均等に分散されたワークロード

![均等](/media/dashboard/dashboard-keyviz-well-dist.png)

上のヒートマップでは、明るく暗い色が微細な混合になっています。これは読み込みまたは書き込みが時間とキー範囲にわたって均等に分散されていることを示しています。ワークロードはすべてのノードに均等に分散され、これは分散データベースにとって理想的です。

### 定期的な読み込みと書き込み

![定期的](/media/dashboard/dashboard-keyviz-period.png)

上のヒートマップでは、X軸（時間）に沿って交互に明るさと暗さがありますが、Y軸（リージョン）に沿って明るさは比較的均等です。これは読み込みと書き込みが定期的に変化していることを示し、これは定期的なスケジュールされたタスクのシナリオで発生する可能性があります。たとえば、ビッグデータプラットフォームはTiDBからデータを定期的に抽出します。このようなシナリオでは、ピーク時の利用でリソースが十分かどうかに注意する必要があります。

### 集中的な読み込みまたは書き込み

![集中的](/media/dashboard/dashboard-keyviz-continue.png)
上記のヒートマップでは、いくつかの明るいラインが見られます。Y軸沿いには、明るいラインの周囲の縁が暗く、これは明るいラインに対応する地域が高い読み取りおよび書き込みトラフィックを持っていることを示しています。トラフィックの分布がアプリケーションの期待通りかどうかを観察できます。たとえば、すべてのサービスがユーザーテーブルに関連付けられている場合、ユーザーテーブルの全体的なトラフィックが高いため、ヒートマップに明るいラインが表示されることは理にかなっています。

さらに、明るいラインの高さ（Y軸に沿った厚さ）が重要です。TiKVには独自の地域ベースのホットスポットバランシングメカニズムがありますので、ホットスポットに関与する地域が多いほど、すべてのTiKVインスタンス間でトラフィックをバランス良く分散させるのに役立ちます。より太く、より明るいラインほど、ホットスポットがより分散しており、TiKVがより良く使用されています。より細く、明るいラインが少ないほど、ホットスポットがより集中しており、TiKVのホットスポットの問題がより顕著で、手動の介入が必要となるかもしれません。

### 連続した読み取りまたは書き込み

![連続した](/media/dashboard/dashboard-keyviz-sequential.png)

上記のヒートマップでは、明るいラインが見られます。これはデータの読み取りまたは書き込みが連続していることを意味します。連続したデータの読み取りまたは書き込みの典型的なシナリオには、データのインポートやテーブルおよびインデックスのスキャンがあります。たとえば、自動増分IDを持つテーブルにデータを連続して書き込む場合です。

明るい領域の地域は、読み取りおよび書き込みトラフィックのホットスポットであり、しばしば全クラスターのパフォーマンスボトルネックとなります。この状況では、アプリケーションの主キーを再調整する必要があります。これにより、地域を可能な限り分散させて圧力を複数の地域に広げることができます。また、低ピーク時にアプリケーションタスクをスケジュールすることもできます。

> **注意:**
>
> このセクションでは一般的なヒートマップのみを示しています。 Key Visualizer は実際の状況に基づいてクラスタ全体のすべてのスキーマやテーブルのヒートマップを表示するため、異なるエリアで異なるヒートマップの種類を見たり、複数のヒートマップの結果が混在していることがあります。実際の状況に基づいてヒートマップを使用してください。

## ホットスポットの問題に対処

TiDBには一般的なホットスポットの問題を緩和するためのいくつかの組み込み機能があります。詳細については、「[高並行書き込みのベストプラクティス](/best-practices/high-concurrency-best-practices.md)」を参照してください。