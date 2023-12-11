---
title: TiDBツールのダウンロード
summary: TiDBツールの公式にメンテナンスされたバージョンをダウンロードします。
aliases: ['/docs/dev/download-ecosystem-tools/','/docs/dev/reference/tools/download/']
---

# TiDBツールのダウンロード

このドキュメントでは、TiDB Toolkitのダウンロード方法について説明します。

TiDB Toolkitには、データエクスポートツールであるDumpling、データインポートツールであるTiDB Lightning、およびバックアップとリストアツールであるBRなど、よく使用されるTiDBツールが含まれています。

> **ヒント:**
>
> - デプロイ環境がインターネットにアクセスできる場合は、単一の[TiUPコマンド](/tiup/tiup-component-management.md)を使用してTiDBツールをデプロイできるため、TiDB Toolkitを別途ダウンロードする必要はありません。
> - Kubernetes上でTiDBをデプロイおよびメンテナンスする必要がある場合は、TiDB Toolkitをダウンロードする代わりに、[TiDB Operatorオフラインインストール](https://docs.pingcap.com/tidb-in-kubernetes/stable/deploy-tidb-operator#offline-installation)の手順に従ってください。

## 環境要件

- オペレーティングシステム: Linux
- アーキテクチャ: amd64またはarm64

## ダウンロードリンク

以下のリンクからTiDB Toolkitをダウンロードできます:

```
https://download.pingcap.org/tidb-community-toolkit-{version}-linux-{arch}.tar.gz
```

リンク中の`{version}`はTiDBのバージョン番号を、`{arch}`はシステムのアーキテクチャを示し、`amd64`または`arm64`のいずれかになります。例えば、`v6.2.0`の`amd64`アーキテクチャ向けのダウンロードリンクは`https://download.pingcap.org/tidb-community-toolkit-v6.2.0-linux-amd64.tar.gz`です。

> **注意:**
>
> [PD Control](/pd-control.md)ツール`pd-ctl`をダウンロードする必要がある場合は、TiDBインストールパッケージを別途`https://download.pingcap.org/tidb-community-server-{version}-linux-{arch}.tar.gz`からダウンロードしてください。

## TiDB Toolkitの説明

使用したいツールに応じて、以下のオフラインパッケージをインストールできます:

| ツール | オフラインパッケージ名 |
|:------|:----------|
| [TiUP](/tiup/tiup-overview.md)  | `tiup-linux-{arch}.tar.gz` <br/>`tiup-{tiup-version}-linux-{arch}.tar.gz` <br/>`dm-{tiup-version}-linux-{arch}.tar.gz` <br/> `server-{version}-linux-{arch}.tar.gz` |
| [Dumpling](/dumpling-overview.md)  | `dumpling-{version}-linux-{arch}.tar.gz`  |
| [TiDB Lightning](/tidb-lightning/tidb-lightning-overview.md)  | `tidb-lightning-ctl` <br/>`tidb-lightning-{version}-linux-{arch}.tar.gz`  |
| [TiDBデータ移行(DM)](/dm/dm-overview.md)  | `dm-worker-{version}-linux-{arch}.tar.gz` <br/>`dm-master-{version}-linux-{arch}.tar.gz` <br/>`dmctl-{version}-linux-{arch}.tar.gz`  |
| [TiCDC](/ticdc/ticdc-overview.md)  | `cdc-{version}-linux-{arch}.tar.gz`  |
| [TiDB Binlog](/tidb-binlog/tidb-binlog-overview.md)  | `pump-{version}-linux-{arch}.tar.gz` <br/>`drainer-{version}-linux-{arch}.tar.gz` <br/>`binlogctl` <br/>`reparo`  |
| [バックアップ＆リストア(BR)](/br/backup-and-restore-overview.md)  | `br-{version}-linux-{arch}.tar.gz`  |
| [sync-diff-inspector](/sync-diff-inspector/sync-diff-inspector-overview.md)  | `sync_diff_inspector`  |
| [PD Recover](/pd-recover.md)  | `pd-recover-{version}-linux-{arch}.tar` |

> **注意:**
>
> `{version}`はインストールするツールのバージョンに依存します。`{arch}`はシステムのアーキテクチャを示し、`amd64`または`arm64`のいずれかになります。