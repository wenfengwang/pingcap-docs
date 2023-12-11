---
title: TiDB デプロイメントFAQ
summary: TiDBのデプロイメントに関するFAQについて学びます。
---

# TiDBデプロイメントFAQ

この文書は、TiDBデプロイメントに関連するFAQをまとめたものです。

## ソフトウェアおよびハードウェア要件

### TiDBはどのようなオペレーティングシステムをサポートしていますか？

TiDBでサポートされているオペレーティングシステムについては、[ソフトウェアとハードウェアの推奨事項](/hardware-and-software-requirements.md)を参照してください。

### 開発、テスト、または本番環境でのTiDBクラスタの推奨ハードウェア構成は何ですか？

開発、テスト、および本番環境用のサーバーハードウェア構成に関する要件と推奨事項については、[ソフトウェアとハードウェアの推奨事項 - サーバーの推奨事項](/hardware-and-software-requirements.md#server-recommendations)を参照してください。

### 10ギガビットの2つのネットワークカードの目的は何ですか？

TiDBは分散クラスタであり、特にPDではタイムスタンプの一意性が必要です。PDが一貫性のない時刻を持っていると、PDサーバの切り替え時に待ち時間が長くなります。2つのネットワークカードのボンディングはデータ転送の安定性を保証し、10ギガビットは転送速度を保証します。ギガビットネットワークカードはボトルネックになりやすいため、10ギガビットネットワークカードの使用を強く推奨します。

### SSDにRAIDを使用しない場合は実珵ですか？

リソースが十分であれば、SSDにRAID 10を使用することをお勧めします。リソースが不十分な場合は、SSDにRAIDを使用しないことも許容されます。

### TiDBコンポーネントの推奨構成は何ですか？

- TiDBはCPUとメモリの要求が高いです。TiDB Binlogを有効にする場合、サービスのボリューム推定とGC操作にかかる時間に基づいてローカルディスクスペースを増やす必要があります。ただし、SSDディスクは必須ではありません。
- PDはクラスタメタデータを保持し、頻繁な読み書きリクエストがあります。高I/Oディスクを要求します。性能の低いディスクはクラスタ全体の性能に影響を与えます。SSDディスクの使用をお勧めします。さらに、リージョンの数が多い場合は、CPUとメモリの要求が高くなります。
- TiKVはCPU、メモリ、およびディスクの要求が高いです。SSDを使用する必要があります。

詳細については、[ソフトウェアとハードウェアの推奨事項](/hardware-and-software-requirements.md)を参照してください。

## インストールとデプロイメント

本番環境では、TiDBクラスタをデプロイする際には、[TiUP](/tiup/tiup-overview.md)を使用することをお勧めします。[TiUPを使用したTiDBクラスタのデプロイ](/production-deployment-using-tiup.md)を参照してください。

### TiKV/PDの変更された`toml`構成が反映されない理由は？

TiKV/PDで`--config`パラメータを設定して`toml`構成を有効にする必要があります。TiKV/PDはデフォルトでは構成を読み取りません。現在、この問題はバイナリを使用して展開した場合にのみ発生します。TiKVの場合は、構成を編集してサービスを再起動します。PDの場合、構成ファイルはPDが最初に起動されたときにのみ読み取られ、その後はpd-ctlを使用して構成を変更できます。詳細については、[PDコントロールユーザーガイド](/pd-control.md)を参照してください。

### TiDBモニタリングフレームワーク（Prometheus + Grafana）は単独のマシンまたは複数のマシンでデプロイするべきですか？推奨されるCPUとメモリは何ですか？

モニタリングマシンは単独のデプロイを推奨します。8コアCPU、16GB以上のメモリ、および500GB以上のハードディスクを推奨します。

### モニターがすべてのメトリクスを表示しない理由は？

モニターのマシン時刻とクラスタ内の時刻の差を確認してください。大きい場合は時刻を修正し、モニターがすべてのメトリクスを表示します。

### supervise/svc/svstatサービスの機能は何ですか？

- supervise: デーモンプロセス、プロセスを管理する
- svc: サービスの開始と停止
- svstat: プロセスの状態を確認する

### inventory.ini変数の説明

| 変数             | 説明                       |
| ---- | ------- |
| `cluster_name` | クラスタの名前、調整可能 |
| `tidb_version` | TiDBのバージョン |
| `deployment_method` | デプロイメント方法、デフォルトではバイナリ、Dockerオプション |
| `process_supervision` | プロセスの監視方法、デフォルトでsystemd、superviseオプション |
| `timezone` | 管理対象ノードのタイムゾーン、調整可能、デフォルトは`Asia/Shanghai`、`set_timezone`変数と一緒に使用 |
| `set_timezone` | 管理対象ノードのタイムゾーンを編集するかどうか、デフォルトはTrue; Falseは閉じることを意味します |
| `enable_elk` | 現在はサポートされていません |
| `enable_firewalld` | ファイアウォールを有効にするかどうか、デフォルトは閉じる |
| `enable_ntpd` | 管理対象ノードのNTPサービスを監視するかどうか、デフォルトはTrue; 閉じないでください |
| `machine_benchmark` | 管理対象ノードのディスクIOPSを監視するかどうか、デフォルトはTrue; 閉じないでください |
| `set_hostname` | 管理対象ノードのホスト名をIPに基づいて編集するかどうか、デフォルトはFalse |
| `enable_binlog` | Pumpをデプロイし、binlogを有効にするかどうか、デフォルトはFalse、Kafkaクラスタに依存。`zookeeper_addrs`変数を参照してください |
| `zookeeper_addrs` | binlog KafkaクラスタのZooKeeperアドレス |
| `enable_slow_query_log` | TiDBの遅いクエリログを単一ファイル(`{{ deploy_dir }}/log/tidb_slow_query.log`)に記録するかどうか、デフォルトはFalse、TiDBログに記録します |
| `deploy_without_tidb` | キーと値のモード、PD、TiKV、およびモニタリングサービスのみをデプロイし、TiDBをデプロイしない。`inventory.ini`ファイルで`tidb_servers`ホストグループのIPをnullに設定してください |

### TiDBの遅いクエリログを別々に記録する方法は？遅いクエリSQLステートメントをどのように特定しますか？

1. TiDBの遅いクエリ定義はTiDBの構成ファイルにあります。`tidb_slow_log_threshold: 300`パラメータは遅いクエリの閾値値を構成するために使用されます（単位：ミリ秒）。

2. 遅いクエリが発生した場合、Grafanaを使用して遅いクエリが発生した`tidb-server`インスタンスと遅いクエリの時刻を特定し、対応するノードのログに記録されたSQLステートメント情報を見つけることができます。

3. ログに加えて、`ADMIN SHOW SLOW`コマンドを使用して遅いクエリを表示することもできます。詳細については、[`ADMIN SHOW SLOW`コマンド](/identify-slow-queries.md#admin-show-slow-command)を参照してください。

### TiKVの`label`が最初のTiDBクラスタの展開時に構成されていない場合、`label`構成を追加する方法は？

TiDBの`label`構成はクラスタの展開アーキテクチャに関連しています。重要であり、PDがグローバル管理とスケジューリングを実行するための基礎です。以前にクラスタを展開する際に`label`を構成していない場合は、PD管理ツールである`pd-ctl`を使用して手動で`location-labels`情報を追加する必要があります。たとえば、`config set location-labels "zone,rack,host"`（実際の`label`レベル名に基づいて構成してください）。

`pd-ctl`の使用方法については、[PD制御ユーザーガイド](/pd-control.md)を参照してください。

### ディスクテストの`dd`コマンドが`oflag=direct`オプションを使用している理由は？

Directモードは、書き込みリクエストをI/Oコマンドにラップし、このコマンドをディスクに送信してファイルシステムキャッシュをバイパスし、ディスクの実隵のI/O読み書きパフォーマンスを直接テストします。

### TiKVインスタンスのディスクパフォーマンスをテストするために`fio`コマンドをどのように使用しますか？

- ランダムリードテスト:

    {{< copyable "shell-regular" >}}

    ```bash
    ./fio -ioengine=psync -bs=32k -fdatasync=1 -thread -rw=randread -size=10G -filename=fio_randread_test.txt -name='fio randread test' -iodepth=4 -runtime=60 -numjobs=4 -group_reporting --output-format=json --output=fio_randread_result.json
    ```

- シーケンシャルライトとランダムリードのミックステスト:

    {{< copyable "shell-regular" >}}

    ```bash
    ./fio -ioengine=psync -bs=32k -fdatasync=1 -thread -rw=randrw -percentage_random=100,0 -size=10G -filename=fio_randread_write_test.txt -name='fio mixed randread and sequential write test' -iodepth=4 -runtime=60 -numjobs=4 -group_reporting --output-format=json --output=fio_randread_write_test.json
    ```

## 現在TiDBがサポートしているパブリッククラウドベンダーは？

TiDBは[Google Cloud GKE](https://docs.pingcap.com/tidb-in-kubernetes/stable/deploy-on-gcp-gke)、[AWS EKS](https://docs.pingcap.com/tidb-in-kubernetes/stable/deploy-on-aws-eks)、および[Alibaba Cloud ACK](https://docs.pingcap.com/tidb-in-kubernetes/stable/deploy-on-alibaba-cloud)での展開をサポートしています。

さらに、TiDBは現在JD CloudとUCloudでも利用可能です。