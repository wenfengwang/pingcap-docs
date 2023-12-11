---
title: TiFlashの展開トポロジー
summary: 最小限のTiDBトポロジーに基づいてTiFlashの展開トポロジーを学ぶ
aliases: ['/docs/dev/tiflash-deployment-topology/']
---

# TiFlashの展開トポロジー

このドキュメントでは、最小限のTiDBトポロジーに基づいて[TiFlash](/tiflash/tiflash-overview.md)の展開トポロジーについて説明します。

TiFlashは、列指向のストレージエンジンであり、徐々に標準的なクラスタートポロジーとなっています。リアルタイムのHTAPアプリケーションに適しています。

## トポロジー情報

| インスタンス | 数 | 物理マシンの構成 | IP | 設定 |
| :-- | :-- | :-- | :-- | :-- |
| TiDB | 3 | 16 VCore 32GB * 1 | 10.0.1.7 <br/> 10.0.1.8 <br/> 10.0.1.9 | デフォルトポート <br/> グローバルディレクトリの設定 |
| PD | 3 | 4 VCore 8GB * 1 | 10.0.1.4 <br/> 10.0.1.5 <br/> 10.0.1.6 | デフォルトポート <br/> グローバルディレクトリの設定 |
| TiKV | 3 | 16 VCore 32GB 2TB (nvme ssd) * 1 | 10.0.1.1 <br/> 10.0.1.2 <br/> 10.0.1.3 | デフォルトポート <br/> グローバルディレクトリの設定 |
| TiFlash | 1 | 32 VCore 64 GB 2TB (nvme ssd) * 1  | 10.0.1.11 | デフォルトポート <br/> グローバルディレクトリの設定 |
| 監視 & Grafana | 1 | 4 VCore 8GB * 1 500GB (ssd) | 10.0.1.10 | デフォルトポート <br/> グローバルディレクトリの設定 |

### トポロジーテンプレート

- [TiFlashトポロジーのためのシンプルなテンプレート](https://github.com/pingcap/docs/blob/master/config-templates/simple-tiflash.yaml)
- [TiFlashトポロジーのための複雑なテンプレート](https://github.com/pingcap/docs/blob/master/config-templates/complex-tiflash.yaml)

上記TiDBクラスタートポロジーファイルの構成項目の詳細な説明については、[TiUPを使用してTiDBを展開するためのトポロジ構成ファイル](/tiup/tiup-cluster-topology-reference.md) を参照してください。

### キーパラメータ

- PDの[配置ルール](/configure-placement-rules.md)機能を有効にするには、`replication.enable-placement-rules`の設定テンプレートの値を`true`に設定します。
- `tiflash_servers`のインスタンスレベルの`"-host"`構成は、ドメイン名ではなくIPのみをサポートしています。
- 詳細なTiFlashパラメータの説明については、[TiFlashの構成](/tiflash/tiflash-configuration.md) を参照してください。

> **注意:**
>
> - 構成ファイルで`tidb`ユーザを手動で作成する必要はありません。TiUPクラスターコンポーネントは自動的に対象のマシンに`tidb`ユーザを作成します。ユーザをカスタマイズしたり、制御マシンとユーザを一貫させることができます。
> - 展開ディレクトリを相対パスとして構成する場合、クラスタはユーザのホームディレクトリに展開されます。