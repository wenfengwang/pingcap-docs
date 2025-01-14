---
title: TiDB ダッシュボード インスタンスプロファイリング - 連続プロファイリング
summary: TiDB、TiKV および PD からパフォーマンスデータを連続的に収集して MTTR を短縮する方法について学びます。

# TiDB ダッシュボード インスタンスプロファイリング - 連続プロファイリング

> **注意:**
>
> この機能はデータベースの専門家向けに設計されています。非専門家のユーザーにとっては、PingCAP テクニカルサポートのガイダンスのもとでこの機能を使用することをお勧めします。

連続プロファイリングを使用すると、各 TiDB、TiKV、PD インスタンスからパフォーマンスデータを**連続的に**収集できます。収集されたパフォーマンスデータは FlameGraph または DAG として視覚化できます。

これらのパフォーマンスデータにより、専門家はインスタンスの CPU およびメモリなどのリソース消費の詳細を分析し、高い CPU 負荷、高いメモリ使用量、プロセスの停滞などの複雑なパフォーマンスの問題をいつでも特定できます。再現できない問題に対しても、その瞬間に収集された過去のパフォーマンスデータを表示することで問題に深く迫ることができます。これにより、MTTR を効果的に短縮できます。

## 手動プロファイリングとの比較

連続プロファイリングは[手動プロファイリング](/dashboard/dashboard-profiling.md)の強化機能です。両者は各インスタンスの異なる種類のパフォーマンスデータを収集し、分析するために使用できます。両者の違いは次のとおりです。

- 手動プロファイリングは、有効化時に短い時間（たとえば、30秒）だけパフォーマンスデータを収集しますが、連続プロファイリングは有効化されている間、常にデータを連続的に収集します。
- 手動プロファイリングは、現在発生している問題のみを分析するために使用できますが、連続プロファイリングは現在の問題だけでなく、過去の問題も分析することができます。
- 手動プロファイリングは特定のインスタンスの特定のパフォーマンスデータを収集することができますが、連続プロファイリングはすべてのインスタンスのすべてのパフォーマンスデータを収集します。
- 連続プロファイリングはより多くのパフォーマンスデータを保存するため、より多くのディスク容量を必要とします。

## サポートされているパフォーマンスデータ

[手動プロファイリング](/dashboard/dashboard-profiling.md#supported-performance-data)で収集されるすべてのパフォーマンスデータが収集されます。

- CPU: TiDB、TiKV、TiFlash、および PD インスタンスの各内部機能の CPU オーバーヘッド

- Heap: TiDB、TiKV、および PD インスタンスの各内部機能のメモリ消費

- Mutex: TiDB および PD インスタンスのミューテックスの競合状態

- Goroutine: TiDB および PD インスタンスのすべてのゴルーチンの実行状態と呼び出しスタック

## ページへのアクセス

次のいずれかの方法で連続プロファイリングページにアクセスできます。

* TiDB ダッシュボードにログインした後、左側のナビゲーションメニューで **Advanced Debugging** > **Profiling Instances** > **連続プロファイリング** をクリックします。

  ![ページにアクセス](/media/dashboard/dashboard-conprof-access.png)

* ブラウザで <http://127.0.0.1:2379/dashboard/#/continuous_profiling> を訪れます。`127.0.0.1:2379` を実際の PD インスタンスのアドレスとポートに置き換えます。

## 連続プロファイリングを有効にする

> **注意:**
>
> 連続プロファイリングを使用するには、クラスタが最新バージョンの TiUP（v1.9.0 またはそれ以上）または TiDB Operator（v1.3.0 またはそれ以上）によってデプロイまたはアップグレードされている必要があります。クラスタが以前のバージョンの TiUP または TiDB Operator を使用してアップグレードされた場合は、手順については[FAQ](/dashboard/dashboard-faq.md#a-required-component-ngmonitoring-is-not-started-error-is-shown)を参照してください。

連続プロファイリングを有効にした後、ウェブページを常にアクティブにしておかなくても、バックグラウンドでパフォーマンスデータを連続的に収集できます。収集されたデータは一定期間保持され、期限が切れたデータは自動的にクリアされます。

この機能を有効にするには：

1. [連続プロファイリングページ](#access-the-page)を訪れます。
2. **設定を開く** をクリックします。右側の **設定** 領域で、**機能を有効にする** をスイッチオンにし、必要に応じて **データの保持期間** のデフォルト値を変更します。
3. **保存** をクリックします。

![機能を有効にする](/media/dashboard/dashboard-conprof-start.png)

## 現在のパフォーマンスデータを表示する

連続プロファイリングが有効になっているクラスタでは、手動プロファイリングは開始できません。現在の瞬間のパフォーマンスデータを表示するには、最新のプロファイリング結果をクリックするだけです。

## 過去のパフォーマンスデータを表示する

一覧ページでは、この機能を有効にしてから収集されたすべてのパフォーマンスデータが表示されます。

![履歴結果](/media/dashboard/dashboard-conprof-history.png)

## パフォーマンスデータをダウンロードする

プロファイリング結果ページで、右上隅の **プロファイリング結果をダウンロード** をクリックして、すべてのプロファイリング結果をダウンロードできます。

![プロファイリング結果をダウンロード](/media/dashboard/dashboard-conprof-download.png)

また、テーブル内の個々のインスタンスをクリックしてそのプロファイリング結果を表示したり、... 上にカーソルを合わせて生データをダウンロードしたりすることもできます。

![プロファイリング結果を表示](/media/dashboard/dashboard-conprof-single.png)

## 連続プロファイリングを無効にする

1. [連続プロファイリングページ](#access-the-page)を訪れます。
2. 右上隅の歯車アイコンをクリックして設定ページを開きます。**機能を有効にする** をオフにします。
3. **保存** をクリックします。
4. ポップアップしたダイアログボックスで **無効にする** をクリックします。

![機能を無効にする](/media/dashboard/dashboard-conprof-stop.png)

## よくある質問

**1. 連続プロファイリングを有効にできず、UI に "required component NgMonitoring is not started" が表示されます。**

[TiDB ダッシュボード FAQ](/dashboard/dashboard-faq.md#a-required-component-ngmonitoring-is-not-started-error-is-shown)を参照してください。

**2. 連続プロファイリングを有効にした後、パフォーマンスに影響が出ますか？**

ベンチマークによると、この機能を有効にした場合の平均パフォーマンス影響は1%未満です。

**3. この機能のステータスはどうですか？**

この機能は現在一般提供（GA）されており、本番環境で使用することができます。