---
title: TiDB ダッシュボードの展開
summary: TiDB ダッシュボードの展開方法を学ぶ。
aliases: ['/docs/dev/dashboard/dashboard-ops-deploy/']
---

# TiDB ダッシュボードの展開

TiDB ダッシュボード UI は、v4.0 以降の PD コンポーネントに組み込まれており、追加の展開は必要ありません。標準の TiDB クラスタを展開するだけで、TiDB ダッシュボードが利用できます。

> **注意:**
>
> TiDB v6.5.0（およびそれ以降）および TiDB Operator v1.4.0（およびそれ以降）は、Kubernetes 上で独立した Pod として TiDB ダッシュボードを展開できます。詳細については、[TiDB オペレーターに独立して TiDB ダッシュボードを展開する](https://docs.pingcap.com/tidb-in-kubernetes/dev/get-started#deploy-tidb-dashboard-independently) を参照してください。

標準の TiDB クラスタを展開する方法については、次のドキュメントを参照してください:

+ [TiDB データベースプラットフォームのクイックスタートガイド](/quick-start-with-tidb.md)
+ [本番環境での TiDB の展開](/production-deployment-using-tiup.md)
+ [Kubernetes 環境での展開](https://docs.pingcap.com/tidb-in-kubernetes/stable/access-dashboard)

> **注意:**
>
> TiDB v4.0 より前の TiDB クラスタには TiDB ダッシュボードを展開できません。

## 複数の PD インスタンスでの展開

クラスタに複数の PD インスタンスが展開されている場合、これらのインスタンスのうちの 1 つだけが TiDB ダッシュボードを提供します。

PD インスタンスが初めて実行されると、自動的に互いに交渉して TiDB ダッシュボードを提供するインスタンスを選択します。TiDB ダッシュボードは他の PD インスタンスで実行されません。PD インスタンスが再起動されたり新しい PD インスタンスが追加された場合でも、TiDB ダッシュボードは選択された PD インスタンスによって常に提供されます。ただし、TiDB ダッシュボードを提供していた PD インスタンスがクラスタから削除される（スケールインされる）と、再交渉が行われます。この交渉プロセスはユーザーの介入は必要ありません。

TiDB ダッシュボードを提供していない PD インスタンスにアクセスすると、ブラウザが自動的にリダイレクトされて TiDB ダッシュボードを提供している PD インスタンスへのアクセスを案内します。これにより、正常にサービスにアクセスできます。このプロセスは以下の図に示されています。

![プロセスの模式図](/media/dashboard/dashboard-ops-multiple-pd.png)

> **注意:**
>
> TiDB ダッシュボードを提供している PD インスタンスは PD リーダーであるとは限りません。

### 実際に TiDB ダッシュボードを提供している PD インスタンスを確認する

TiUP を使用して展開された稼働中のクラスタの場合、`tiup cluster display` コマンドを使用して TiDB ダッシュボードを提供している PD インスタンスを確認できます。`CLUSTER_NAME` をクラスタ名に置き換えてください。

{{< copyable "shell-regular" >}}

```bash
tiup cluster display CLUSTER_NAME --dashboard
```

サンプル出力は以下のようになります:

```bash
http://192.168.0.123:2379/dashboard/
```

> **注意:**
>
> この機能は、`tiup cluster` デプロイツールの後のバージョン（v1.0.3 以降）でのみ利用可能です。
>
> <details>
> <summary>TiUP Cluster をアップグレードする</summary>
>
> ```bash
> tiup update --self
> tiup update cluster --force
> ```
>
> </details>

### 別の PD インスタンスに切り替えて TiDB ダッシュボードを提供する

TiUP を使用して展開された稼働中のクラスタの場合、`tiup ctl:v<CLUSTER_VERSION> pd` コマンドを使用して TiDB ダッシュボードを提供する PD インスタンスを変更するか、無効になった場合に TiDB ダッシュボードを提供するための新しい PD インスタンスを再指定できます。

{{< copyable "shell-regular" >}}

```bash
tiup ctl:v<CLUSTER_VERSION> pd -u http://127.0.0.1:2379 config set dashboard-address http://9.9.9.9:2379
```

上記のコマンドでは、次のことを置き換えます:

- `127.0.0.1:2379` を任意の PD インスタンスの IP とポートに置き換えます。
- `9.9.9.9:2379` を TiDB ダッシュボードのサービスを実行する新しい PD インスタンスの IP とポートに置き換えます。

変更が適用されているかどうかを確認するには `tiup cluster display` コマンドを使用できます（`CLUSTER_NAME` をクラスタ名に置き換えてください）:

{{< copyable "shell-regular" >}}

```bash
tiup cluster display CLUSTER_NAME --dashboard
```

> **警告:**
>
> TiDB ダッシュボードを実行するインスタンスを変更すると、前の TiDB ダッシュボードインスタンスに格納されているローカルデータが失われます。これには Key Visualize の履歴や検索履歴も含まれます。

## TiDB ダッシュボードの無効化

TiUP を使用して展開された稼働中のクラスタの場合、`tiup ctl:v<CLUSTER_VERSION> pd` コマンドを使用してすべての PD インスタンスで TiDB ダッシュボードを無効にします（`127.0.0.1:2379` を任意の PD インスタンスの IP とポートに置き換えます）:

{{< copyable "shell-regular" >}}

```bash
tiup ctl:v<CLUSTER_VERSION> pd -u http://127.0.0.1:2379 config set dashboard-address none
```

TiDB ダッシュボードを無効化した後、TiDB ダッシュボードを提供する PD インスタンスを確認することは失敗します:

```
Error: TiDB Dashboard is disabled
```

ブラウザを介して任意の PD インスタンスの TiDB ダッシュボードアドレスにアクセスしても以下のエラーが発生します:

```
Dashboard is not started.
```

## TiDB ダッシュボードの再有効化

TiUP を使用して展開された稼働中のクラスタの場合、`tiup ctl:v<CLUSTER_VERSION> pd` コマンドを使用して PD に TiDB ダッシュボードを再度有効化するよう要求します（`127.0.0.1:2379` を任意の PD インスタンスの IP とポートに置き換えます）:

{{< copyable "shell-regular" >}}

```bash
tiup ctl:v<CLUSTER_VERSION> pd -u http://127.0.0.1:2379 config set dashboard-address auto
```

上記のコマンドを実行した後、`tiup cluster display` コマンドを使用して PD によって自動的に交渉された TiDB ダッシュボードインスタンスアドレスを表示できます（`CLUSTER_NAME` をクラスタ名に置き換えてください）:

{{< copyable "shell-regular" >}}

```bash
tiup cluster display CLUSTER_NAME --dashboard
```

また、前の TiDB ダッシュボードインスタンスに格納されているローカルデータが失われる可能性があることに注意して、TiDB ダッシュボードを手動で再有効化することもできます。詳細は[別の PD インスタンスに切り替えて TiDB ダッシュボードを提供する](#switch-to-another-pd-instance-to-serve-tidb-dashboard) を参照してください。

> **警告:**
>
> 新たに有効化した TiDB ダッシュボードインスタンスが以前のインスタンスと異なる場合、Key Visualize の履歴や検索履歴など、前の TiDB ダッシュボードインスタンスに格納されていたローカルデータが失われる可能性があります。

## 次の手順

- TiDB ダッシュボード UI にアクセスしてログインする方法については、[TiDB ダッシュボードへのアクセス](/dashboard/dashboard-access.md) を参照してください。

- ファイアウォールの構成など、TiDB ダッシュボードのセキュリティを強化する方法については、[TiDB ダッシュボードのセキュリティ](/dashboard/dashboard-ops-security.md) を参照してください。