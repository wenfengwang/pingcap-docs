---
title: TiSparkの展開トポロジー
summary: 最小TiDBトポロジーに基づいてTiUPを使用してTiSparkを展開する展開トポロジーを学ぶ

# TiSparkの展開トポロジー

> **警告:**
>
> TiUPクラスターでのTiSparkサポートは非推奨です。**使用をお勧めしません**。

このドキュメントは、TiSparkの展開トポロジーと、最小クラスタートポロジーに基づいてTiSparkを展開する方法について紹介します。

TiSparkは、TiDB/TiKV上でApache Sparkを実行し、複雑なOLAPクエリに対応するために構築されたコンポーネントです。これにより、Sparkプラットフォームと分散TiKVクラスターの利点がTiDBにもたらされ、TiDBはオンライン取引と分析の両方をカバーするワンストップソリューションになります。

TiSparkアーキテクチャや使用方法の詳細については、[TiSparkユーザーガイド](/tispark-overview.md)を参照してください。

## トポロジー情報

| インスタンス | 数 | 物理マシン構成 | IP | 設定 |
| :-- | :-- | :-- | :-- | :-- |
| TiDB | 3 | 16 VCore 32GB * 1 | 10.0.1.1 <br/> 10.0.1.2 <br/> 10.0.1.3 | デフォルトポート <br/> グローバルディレクトリ設定 |
| PD | 3 | 4 VCore 8GB * 1 |10.0.1.4 <br/> 10.0.1.5 <br/> 10.0.1.6 | デフォルトポート <br/> グローバルディレクトリ設定 |
| TiKV | 3 | 16 VCore 32GB 2TB (nvme ssd) * 1 | 10.0.1.7 <br/> 10.0.1.8 <br/> 10.0.1.9 | デフォルトポート <br/> グローバルディレクトリ設定 |
| TiSpark | 3 | 8 VCore 16GB * 1 | 10.0.1.21 (master) <br/> 10.0.1.22 (worker) <br/> 10.0.1.23 (worker) | デフォルトポート <br/> グローバルディレクトリ設定 |
| モニタリング & Grafana | 1 | 4 VCore 8GB * 1 500GB (ssd) | 10.0.1.11 | デフォルトポート <br/> グローバルディレクトリ設定 |

## トポロジーテンプレート

- [シンプルTiSparkトポロジーテンプレート](https://github.com/pingcap/docs/blob/master/config-templates/simple-tispark.yaml)
- [複雑TiSparkトポロジーテンプレート](https://github.com/pingcap/docs/blob/master/config-templates/complex-tispark.yaml)

上記TiDBクラスタートポロジーファイルの構成項目の詳細については、[TiUPを使用してTiDBを展開するためのトポロジーコンフィグファイル](/tiup/tiup-cluster-topology-reference.md)を参照してください。

> **注意:**
>
> - 構成ファイルで`tidb`ユーザーを手動で作成する必要はありません。TiUPクラスターコンポーネントが対象のマシンで`tidb`ユーザーを自動的に作成します。ユーザーをカスタマイズするか、制御マシンとユーザーを一貫させることができます。
> - デプロイディレクトリを相対パスとして構成した場合、クラスターはユーザーのホームディレクトリに展開されます。

## 必要条件

TiSparkはApache Sparkクラスターに基づいているため、TiSparkを含むTiDBクラスターを開始する前に、TiSparkを展開するサーバーにJavaランタイム環境（JRE）8がインストールされていることを確認する必要があります。それ以外の場合、TiSparkを起動できません。

TiUPではJREの自動インストールはサポートされていません。自分でインストールする必要があります。詳細なインストール手順については、[プレビルトOpenJDKパッケージのダウンロードとインストール方法](https://openjdk.java.net/install/)を参照してください。

展開サーバーにJRE 8が既にインストールされているが、システムのデフォルトパッケージ管理ツールのパスに存在しない場合、`java_home`パラメータを設定することで使用するJRE環境のパスを指定できます。このパラメータは`JAVA_HOME`システム環境変数に対応しています。