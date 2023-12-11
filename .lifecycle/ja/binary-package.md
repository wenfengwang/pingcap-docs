---
title：TiDBのインストールパッケージ
summary：TiDBのインストールパッケージと含まれる特定のコンポーネントについて学ぶ。

# TiDBのインストールパッケージ

[TiUPをオフラインでデプロイ](/production-deployment-using-tiup.md#deploy-tiup-offline)する前に、TiDBのバイナリパッケージを[公式ダウンロードページ](https://en.pingcap.com/download/)からダウンロードする必要があります。

TiDBのバイナリパッケージは、amd64およびarm64アーキテクチャで利用できます。いずれのアーキテクチャでも、TiDBは2つのバイナリパッケージを提供します：`TiDB-community-server`と`TiDB-community-toolkit`。

`TiDB-community-server`パッケージには次の内容が含まれます。

| コンテンツ | 変更履歴 |
|---|---|
| tidb-{version}-linux-{arch}.tar.gz |  |
| tikv-{version}-linux-{arch}.tar.gz |  |
| tiflash-{version}-linux-{arch}.tar.gz |  |
| pd-{version}-linux-{arch}.tar.gz |  |
| ctl-{version}-linux-{arch}.tar.gz |  |
| grafana-{version}-linux-{arch}.tar.gz |  |
| alertmanager-{version}-linux-{arch}.tar.gz |  |
| blackbox_exporter-{version}-linux-{arch}.tar.gz |  |
| prometheus-{version}-linux-{arch}.tar.gz |  |
| node_exporter-{version}-linux-{arch}.tar.gz |  |
| tiup-linux-{arch}.tar.gz |  |
| tiup-{version}-linux-{arch}.tar.gz |  |
| local_install.sh |  |
| cluster-{version}-linux-{arch}.tar.gz |  |
| insight-{version}-linux-{arch}.tar.gz |  |
| diag-{version}-linux-{arch}.tar.gz | v6.0.0での新機能 |
| influxdb-{version}-linux-{arch}.tar.gz |  |
| playground-{version}-linux-{arch}.tar.gz |  |

> **注意:**
>
> `{version}`は、インストールしているコンポーネントやサーバのバージョンに依存します。`{arch}`は、`amd64`または`arm64`のシステムのアーキテクチャに依存します。

`TiDB-community-toolkit`パッケージには次の内容が含まれます。

| コンテンツ | 変更履歴 |
|---|---|
| pd-recover-{version}-linux-{arch}.tar.gz |  |
| etcdctl | v6.0.0での新機能 |
| tiup-linux-{arch}.tar.gz |  |
| tiup-{version}-linux-{arch}.tar.gz |  |
| tidb-lightning-{version}-linux-{arch}.tar.gz |  |
| tidb-lightning-ctl |  |
| dumpling-{version}-linux-{arch}.tar.gz |  |
| cdc-{version}-linux-{arch}.tar.gz |  |
| dm-{version}-linux-{arch}.tar.gz |  |
| dm-worker-{version}-linux-{arch}.tar.gz |  |
| dm-master-{version}-linux-{arch}.tar.gz |  |
| dmctl-{version}-linux-{arch}.tar.gz |  |
| br-{version}-linux-{arch}.tar.gz |  |
| package-{version}-linux-{arch}.tar.gz |  |
| bench-{version}-linux-{arch}.tar.gz |  |
| errdoc-{version}-linux-{arch}.tar.gz |  |
| dba-{version}-linux-{arch}.tar.gz |  |
| PCC-{version}-linux-{arch}.tar.gz |  |
| pump-{version}-linux-{arch}.tar.gz |  |
| drainer-{version}-linux-{arch}.tar.gz |  |
| binlogctl | v6.0.0での新機能 |
| sync_diff_inspector |  |
| reparo |  |
| arbiter |  |
| server-{version}-linux-{arch}.tar.gz | v6.2.0での新機能 |
| grafana-{version}-linux-{arch}.tar.gz | v6.2.0での新機能 |
| alertmanager-{version}-linux-{arch}.tar.gz | v6.2.0での新機能 |
| prometheus-{version}-linux-{arch}.tar.gz | v6.2.0での新機能 |
| blackbox_exporter-{version}-linux-{arch}.tar.gz | v6.2.0での新機能 |
| node_exporter-{version}-linux-{arch}.tar.gz | v6.2.0での新機能 |

> **注意:**
>
> `{version}`は、インストールしているツールのバージョンに依存します。`{arch}`は、`amd64`または`arm64`のシステムのアーキテクチャに依存します。

## 関連情報

[TiUPをオフラインでデプロイ](/production-deployment-using-tiup.md#deploy-tiup-offline)