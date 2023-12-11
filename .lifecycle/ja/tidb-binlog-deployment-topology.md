---
title: TiDB バイナリログ配置トポロジー
summary: TiDB トポロジーの最小構成に基づいた TiDB バイナリログの配置トポロジーを学びます。
aliases: ['/docs/dev/tidb-binlog-deployment-topology/']
---

# TiDB バイナリログ配置トポロジー

このドキュメントでは、最小 TiDB トポロジーに基づいた [TiDB バイナリログ](/tidb-binlog/tidb-binlog-overview.md) の配置トポロジーについて説明します。

TiDB バイナリログは、増分データのレプリケーションに広く使用されています。ほぼリアルタイムのバックアップとレプリケーションを提供します。

## トポロジー情報

| インスタンス | カウント | 物理マシン構成 | IP | 設定 |
| :-- | :-- | :-- | :-- | :-- |
| TiDB | 3 | 16 VCore 32 GB | 10.0.1.1 <br/> 10.0.1.2 <br/> 10.0.1.3 | デフォルトポート構成; <br/> `enable_binlog` を有効にする; <br/> `ignore-error` を有効にする |
| PD | 3 | 4 VCore 8 GB | 10.0.1.4 <br/> 10.0.1.5 <br/> 10.0.1.6 | デフォルトポート構成 |
| TiKV | 3 | 16 VCore 32 GB | 10.0.1.7 <br/> 10.0.1.8 <br/> 10.0.1.9 | デフォルトポート構成 |
| Pump| 3 | 8 VCore 16GB | 10.0.1.1 <br/> 10.0.1.7 <br/> 10.0.1.8 | デフォルトポート構成; <br/> GC 時間を 7 日に設定 |
| Drainer | 1 | 8 VCore 16GB | 10.0.1.12 | デフォルトポート構成; <br/> デフォルトの初期化 commitTS を最新のタイムスタンプとして -1 に設定; <br/> ダウンストリームのターゲット TiDB を `10.0.1.12:4000` に構成する |

### トポロジーテンプレート

- [ `mysql` をダウンストリームタイプとする TiDB バイナリログトポロジーの簡易テンプレート](https://github.com/pingcap/docs/blob/master/config-templates/simple-tidb-binlog.yaml)
- [ `file` をダウンストリームタイプとする TiDB バイナリログトポロジーの簡易テンプレート](https://github.com/pingcap/docs/blob/master/config-templates/simple-file-binlog.yaml)
- [ TiDB バイナリログトポロジーの複雑なテンプレート](https://github.com/pingcap/docs/blob/master/config-templates/complex-tidb-binlog.yaml)

上記TiDBクラスタトポロジーファイルの設定項目の詳細な説明については、「TiUPを使用してTiDBをデプロイするためのトポロジー構成ファイル」を参照してください、[/tiup/tiup-cluster-topology-reference.md](/tiup/tiup-cluster-topology-reference.md)。

### キーパラメータ

トポロジー構成テンプレートのキーパラメータは次のとおりです:

- `server_configs.tidb.binlog.enable: true`

    - バイナリログサービスを有効にします。
    - デフォルト値: `false`。

- `server_configs.tidb.binlog.ignore-error: true`

    - 高可用性シナリオでは、この構成を有効にすることをお勧めします。
    - `true`に設定すると、エラーが発生した場合、TiDBはバイナリログへのデータ書き込みを停止し、「tidb_server_critical_error_total」監視メトリクスの値に`1`を追加します。
    - `false`に設定すると、TiDBはバイナリログへのデータ書き込みに失敗した場合、TiDBサービス全体が停止します。

- `drainer_servers.config.syncer.db-type`

    TiDBバイナリログのダウンストリームタイプ。現在、`mysql`、`tidb`、`kafka`、および`file`がサポートされています。

- `drainer_servers.config.syncer.to`

    TiDBバイナリログのダウンストリーム構成。異なる `db-type` に応じて、この構成項目を使用してダウンストリームデータベースの接続パラメータ、Kafkaの接続パラメータ、ファイルの保存パスを構成できます。詳細については、「[TiDBバイナリログ構成ファイル](/tidb-binlog/tidb-binlog-configuration-file.md#syncerto)」を参照してください。

> **注意:**
>
> - 構成ファイルテンプレートを編集する際は、カスタムポートやディレクトリを必要としない場合は、IP のみを変更します。
> - 構成ファイルで `tidb` ユーザを手動で作成する必要はありません。TiUP クラスタコンポーネントは自動的に対象マシンに `tidb` ユーザを作成します。ユーザをカスタマイズするか、ユーザを制御マシンと一貫させることができます。
> - デプロイディレクトリを相対パスとして構成する場合、クラスタはユーザのホームディレクトリにデプロイされます。