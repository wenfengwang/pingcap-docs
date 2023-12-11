---
title: TiDB Binlog のアップグレード方法
summary: TiDB Binlog を最新クラスターバージョンにアップグレードする方法について学びます。
aliases: ['/docs/dev/tidb-binlog/upgrade-tidb-binlog/','/docs/dev/reference/tidb-binlog/upgrade/','/docs/dev/how-to/upgrade/tidb-binlog/']
---

# TiDB Binlog のアップグレード

本書は、手動で展開された TiDB Binlog を最新の[クラスター](/tidb-binlog/tidb-binlog-overview.md)バージョンにアップグレードする方法について紹介しています。また、以前の非互換バージョン（Kafka/Local バージョン）から最新バージョンに TiDB Binlog をアップグレードする方法についても説明しています。

> **注意:**
>
> - TiDB Binlog は TiDB v5.0 で導入された一部の機能と互換性がなく、一緒に使用することはできません。詳細については、[Notes](/tidb-binlog/tidb-binlog-overview.md#notes)を参照してください。
> - TiDB v7.5.0 から、TiDB Binlog のデータレプリケーション機能の技術サポートは提供されなくなりました。データレプリケーションの代替ソリューションとして、[TiCDC](/ticdc/ticdc-overview.md)の使用を強く推奨します。
> - TiDB v7.5.0 では引き続き TiDB Binlog のリアルタイムバックアップおよびリストア機能がサポートされていますが、このコンポーネントは将来のバージョンで完全に廃止される予定です。データの回復の代替ソリューションとして、[PITR](/br/br-pitr-guide.md)の使用を推奨します。

## 手動で展開された TiDB Binlog のアップグレード

TiDB Binlog を手動で展開した場合は、次の手順に従ってください。

### Pump のアップグレード

まず、クラスター内の各 Pump インスタンスを1つずつアップグレードします。これにより、常に TiDB から binlog を受信できる Pump インスタンスがクラスターに存在します。手順は以下の通りです。

1. 元のファイルを新しい `pump` のバージョンで置き換えます。
2. Pump プロセスを再起動します。

### Drainer のアップグレード

次に、Drainer コンポーネントをアップグレードします。

1. 元のファイルを新しい `drainer` のバージョンで置き換えます。
2. Drainer プロセスを再起動します。

## Kafka/Local バージョンからクラスターバージョンの TiDB Binlog へのアップグレード

新しい TiDB バージョン（v2.0.8-binlog、v2.1.0-rc.5 以降）は、Kafka バージョンや Local バージョンの TiDB Binlog とは互換性がありません。TiDB を新しいバージョンにアップグレードすると、TiDB Binlog のクラスターバージョンを使用する必要があります。アップグレード前に Kafka バージョンまたは Local バージョンの TiDB Binlog を使用している場合は、TiDB Binlog をクラスターバージョンにアップグレードする必要があります。

TiDB Binlog のバージョンと TiDB のバージョンの対応関係は次の表に示されています。

| TiDB Binlog バージョン | TiDB バージョン                               | ノート     |
|---------------------|-------------------------------------------|--------------------------------------------------------------------------------------------|
| Local               | TiDB 1.0 またはそれ以前のバージョン             |                                                                                            |
| Kafka               | TiDB 1.0 ～ TiDB 2.1 RC5                       | TiDB 1.0 は TiDB Binlog の Local バージョンと Kafka バージョンの両方をサポートしています。                        |
| Cluster             | TiDB v2.0.8-binlog、TiDB 2.1 RC5 またはそれ以降 | TiDB v2.0.8-binlog は TiDB Binlog のクラスターバージョンをサポートする特別な 2.0 バージョンです。          |

### アップグレードプロセス

> **注意:**
>
> 完全なデータのインポートが許容される場合は、古いバージョンを放棄し、[TiDB Binlog クラスターデプロイメント](/tidb-binlog/deploy-tidb-binlog.md)に従って TiDB Binlog を展開できます。

元のチェックポイントから複製を再開する場合は、TiDB Binlog をアップグレードするために、次の手順を実行します。

1. 新しいバージョンの Pump を展開します。
2. TiDB クラスターサービスを停止します。
3. TiDB および構成をアップグレードし、binlog データを新しい Pump クラスターに書き込みます。
4. TiDB クラスターをサービスに再接続します。
5. 古い Drainer のバージョンが古い Pump のデータをダウンストリームに完全に複製したことを確認します。

    Drainer の `status` インタフェースをクエリします。以下のコマンドを使用します。

    {{< copyable "shell-regular" >}}

    ```bash
    curl 'http://172.16.10.49:8249/status'
    ```

    ```
    {"PumpPos":{"172.16.10.49:8250":{"offset":32686}},"Synced": true ,"DepositWindow":{"Upper":398907800202772481,"Lower":398907799455662081}}
    ```

    `Synced` の返り値が True である場合、古い Drainer は古い Pump のデータをダウンストリームに完全に複製したことを意味します。

6. 新しいバージョンの Drainer を起動します。
7. 古いバージョンの Pump および Drainer および依存する Kafka および ZooKeeper を終了します。