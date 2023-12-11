---
title: TiDB ダッシュボードの遅いクエリーページ
summary: TiDB ダッシュボードの遅いクエリーページの使用方法
aliases: ['/docs/dev/dashboard/dashboard-slow-query/']
---

# TiDB ダッシュボードの遅いクエリーページ

TiDB ダッシュボードの遅いクエリーページでは、クラスタ内のすべての遅いクエリを検索して表示することができます。

デフォルトでは、実行時間が300ミリ秒を超えるSQLクエリは遅いクエリと見なされます。これらのクエリは[遅いクエリログ](/identify-slow-queries.md)に記録され、TiDBダッシュボードを介して検索できます。遅いクエリのしきい値は、 [`tidb_slow_log_threshold`](/system-variables.md#tidb_slow_log_threshold) セッション変数または [`instance.tidb_slow_log_threshold`](/tidb-configuration-file.md#tidb_slow_log_threshold) TiDBパラメータを調整することができます。

> **注意:**
>
> 遅いクエリログが無効になっている場合、この機能は利用できません。遅いクエリログはデフォルトで有効になっており、システム変数 [`tidb_enable_slow_log`](/system-variables.md#tidb_enable_slow_log) を使用して有効または無効にできます。

## ページへのアクセス

遅いクエリーページにアクセスするために、次の2つの方法のいずれかを使用できます:

* TiDBダッシュボードにログインした後、左のナビゲーションメニューで**遅いクエリ**をクリックします。

* ブラウザで <http://127.0.0.1:2379/dashboard/#/slow_query> を訪問します。`127.0.0.1:2379` を実際のPDアドレスとポートに置換します。

遅いクエリページに表示されるすべてのデータは、TiDBの遅いクエリシステムテーブルと遅いクエリログから取得されます。詳細については、[遅いクエリログ](/identify-slow-queries.md)を参照してください。

### フィルタの変更

時間範囲、関連データベース、SQLキーワード、SQLの種類、表示する遅いクエリの数に基づいて遅いクエリをフィルタリングすることができます。以下の画像では、過去30分間の100件の遅いクエリがデフォルトで表示されています。

![リストのフィルターの変更](/media/dashboard/dashboard-slow-queries-list1-v620.png)

### 追加のカラムの表示

ページ上で **Columns** をクリックすると、追加のカラムを選択できます。カラム名の右側にある **(i)** アイコンにマウスを移動させると、そのカラムの説明が表示されます:

![追加のカラムの表示](/media/dashboard/dashboard-slow-queries-list2-v620.png)

### 遅いクエリのローカルエクスポート

ページの右上隅で ☰ (**More**) をクリックし、**Export** オプションを表示できます。**Export** をクリックすると、TiDBダッシュボードは現在のリスト内の遅いクエリをCSVファイルとしてエクスポートします。

![遅いクエリのローカルエクスポート](/media/dashboard/dashboard-slow-queries-export-v651.png)

### カラムでのソート

デフォルトでは、リストは **終了時間** の降順でソートされています。カラムの見出しをクリックして、そのカラムでソートするかソート順序を切り替えることができます:

![ソートの基準の変更](/media/dashboard/dashboard-slow-queries-list3-v620.png)

## 実行詳細の表示

リスト内の任意のアイテムをクリックすると、遅いクエリの実行情報が詳細に表示されます。表示される情報には以下が含まれます:

- クエリ: SQLステートメントのテキスト (以下の図のarea 1)
- プラン: 遅いクエリの実行計画 (以下の図のarea 2)
- その他のソートされたSQL実行情報 (以下の図のarea 3)

![実行詳細の表示](/media/dashboard/dashboard-slow-queries-detail1-v620.png)

### SQL

**Expand** ボタンをクリックすると、アイテムの詳細情報を表示できます。**Copy** ボタンをクリックすると、詳細情報をクリップボードにコピーできます。

### 実行計画

TiDBダッシュボードでは、テーブル、テキスト、グラフの3つの方法で実行計画を表示できます。実行計画の読み方については、 [クエリ実行計画の理解](/explain-overview.md) を参照してください。

#### テーブル形式の実行計画

テーブル形式では、実行計画の詳細情報が表示され、異常なオペレータのメトリックを素早く特定し、異なるオペレータの状態を比較するのに役立ちます。以下の図は、テーブル形式の実行計画を示しています:

![テーブル形式の実行計画](/media/dashboard/dashboard-table-plan.png)

テーブル形式はテキスト形式と同様の情報を表示しますが、ユーザーフレンドリーな相互作用が提供されます:

- カラム幅を自由に調整できます。
- コンテンツがカラム幅を超えると、自動的に切り捨てられ、完全な情報のためのツールチップが表示されます。
- 実行計画が大きい場合、ローカルで分析するためにテキストファイルとしてダウンロードできます。
- カラムピッカーを使用して、カラムの非表示や管理ができます。

![テーブル形式の実行計画 - カラムピッカー](/media/dashboard/dashboard-table-plan-columnpicker.png)

#### グラフ形式の実行計画

グラフ形式は、複雑なSQLステートメントの実行計画ツリーを表示するのに適しており、各オペレータとそれに対応するコンテンツを詳しく理解するのに役立ちます。以下の図は、グラフ形式の実行計画を示しています:

![グラフ形式の実行計画](/media/dashboard/dashboard-visual-plan-2.png)

- グラフは左から右、上から下に実行を示しています。
- 上位のノードは親オペレータであり、下位のノードは子オペレータです。
- タイトルバーの色は、オペレータが実行されるコンポーネントを示しています: 黄色はTiDB、青色はTiKV、ピンク色はTiFlashです。
- タイトルバーにはオペレータ名が表示され、それ以下に表示されているテキストがオペレータの基本情報です。

ノード領域をクリックすると、詳細なオペレータ情報が右側のサイドバーに表示されます。

![グラフ形式の実行計画 - サイドバー](/media/dashboard/dashboard-visual-plan-popup.png)

### SQL実行の詳細

SQLステートメントの基本情報、実行時間、Coprocessorの読み取り、トランザクション、遅いクエリに関する情報については、詳細情報を表示するために各タブのタイトルをクリックできます。

![異なる実行情報の表示](/media/dashboard/dashboard-slow-queries-detail2-v620.png)

#### 基本タブ

SQL実行の基本情報には、テーブル名、インデックス名、実行回数、および合計待ち時間が含まれます。 **説明** 列には、各フィールドの詳細な説明が提供されます。

![基本情報](/media/dashboard/dashboard-slow-queries-detail-plans-basic.png)

#### 時間タブ

**Time** タブをクリックし、実行計画の各段階がどのくらい続くかを確認できます。

> **注意:**
>
> 1つのSQLステートメント内で並列でいくつかの操作が実行される場合、各段階の累積時間は実際のSQLステートメントの実行時間を超える場合があります。

![実行時間](/media/dashboard/dashboard-slow-queries-detail-plans-time.png)

#### Coprocessorタブ

**Coprocessor** タブをクリックし、Coprocessorの読み取り関連情報を確認できます。

![Coprocessorの読み取り](/media/dashboard/dashboard-slow-queries-detail-plans-cop-read.png)

#### トランザクションタブ

**Transaction** タブをクリックし、実行計画とトランザクションに関連する情報（書き込みキーの平均数、書き込みキーの最大数など）を確認できます。

![Transaction](/media/dashboard/dashboard-slow-queries-detail-plans-transaction.png)