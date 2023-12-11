---
title: 最小デプロイメントトポロジー
summary: TiDBクラスターの最小デプロイメントトポロジーを学ぶ。
aliases: ['/docs/dev/minimal-deployment-topology/']
---

# 最小デプロイメントトポロジー

このドキュメントではTiDBクラスターの最小デプロイメントトポロジーについて説明します。

## トポロジー情報

| インスタンス | 数 | 物理マシン構成 | IP | 構成 |
| :-- | :-- | :-- | :-- | :-- |
| TiDB | 2 | 16 VCore 32 GiB <br/> 100 GiBのストレージ | 10.0.1.1 <br/> 10.0.1.2 | デフォルトポート <br/> グローバルディレクトリ構成 |
| PD | 3 | 4 VCore 8 GiB <br/> 100 GiBのストレージ |10.0.1.4 <br/> 10.0.1.5 <br/> 10.0.1.6 | デフォルトポート <br/> グローバルディレクトリ構成 |
| TiKV | 3 | 16 VCore 32 GiB <br/> 2 TiB（NVMe SSD）のストレージ | 10.0.1.7 <br/> 10.0.1.8 <br/> 10.0.1.9 | デフォルトポート <br/> グローバルディレクトリ構成 |
| 監視 & Grafana | 1 | 4 VCore 8 GiB <br/> 500 GiB（SSD）のストレージ | 10.0.1.10 | デフォルトポート <br/> グローバルディレクトリ構成 |

### トポロジーテンプレート

- [最小トポロジー用のシンプルテンプレート](https://github.com/pingcap/docs/blob/master/config-templates/simple-mini.yaml)
- [最小トポロジー用の複雑なテンプレート](https://github.com/pingcap/docs/blob/master/config-templates/complex-mini.yaml)

上記TiDBクラスタートポロジーファイル内の構成項目の詳細な説明については、[TiUPを使用してTiDBを展開するためのトポロジー構成ファイル](/tiup/tiup-cluster-topology-reference.md)を参照してください。

> **注意:**
>
> - 構成ファイルで「tidb」ユーザを手動で作成する必要はありません。TiUPクラスターコンポーネントは自動的にターゲットマシン上に「tidb」ユーザを作成します。ユーザをカスタマイズするか、制御マシンとユーザを一貫させることができます。
> - デプロイディレクトリを相対パスとして構成する場合、クラスターはユーザーのホームディレクトリに展開されます。