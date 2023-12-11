---
title: ソフトウェアとハードウェアの推奨事項
summary: TiDBの展開および実行におけるソフトウェアとハードウェアの推奨事項を学ぶ
aliases: ['/docs/dev/hardware-and-software-requirements/','/docs/dev/how-to/deploy/hardware-recommendations/']
---

# ソフトウェアとハードウェアの推奨事項

<!-- TiDBに関するローカリゼーションノート:

- 英語: 分散SQLを使用し、HTAPを強調します.
- 中国語: "NewSQL"を使用し、ワンストップリアルタイムHTAP ("一栈式实时 HTAP")を強調します
- 日本語: NewSQLを使用する理由が認識されているため、NewSQLを使用します

-->

高性能なオープンソース分散SQLデータベースであるTiDBは、Intelアーキテクチャサーバー、ARMアーキテクチャサーバー、および主要な仮想化環境に展開でき、十分に実行できます。TiDBは、主要なハードウェアネットワークおよびLinuxオペレーティングシステムのほとんどをサポートしています。

## OSとプラットフォームの要件

<table>
<thead>
  <tr>
    <th>オペレーティングシステム</th>
    <th>サポートされるCPUアーキテクチャ</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>Red Hat Enterprise Linux 8.4 またはそれ以降の8.xバージョン</td>
    <td><ul><li>x86_64</li><li>ARM 64</li></ul></td>
  </tr>
  <tr>
    <td><ul><li>Red Hat Enterprise Linux 7.3 またはそれ以降の7.xバージョン</li><li>CentOS 7.3 またはそれ以降の7.xバージョン</li></ul></td>
    <td><ul><li>x86_64</li><li>ARM 64</li></ul></td>
  </tr>
  <tr>
    <td>Amazon Linux 2</td>
    <td><ul><li>x86_64</li><li>ARM 64</li></ul></td>
  </tr>
  <tr>
    <td>Kylin Euler V10 SP1/SP2</td>
    <td><ul><li>x86_64</li><li>ARM 64</li></ul></td>
  </tr>
  <tr>
    <td>UnionTech OS (UOS) V20</td>
    <td><ul><li>x86_64</li><li>ARM 64</li></ul></td>
  </tr>
  <tr>
    <td>openEuler 22.03 LTS SP1</td>
    <td><ul><li>x86_64</li><li>ARM 64</li></ul></td>
  </tr>
  <tr>
    <td>macOS 12（Monterey）またはそれ以降</td>
    <td><ul><li>x86_64</li><li>ARM 64</li></ul></td>
  </tr>
  <tr>
    <td>Oracle Enterprise Linux 7.3 またはそれ以降の7.xバージョン</td>
    <td>x86_64</td>
  </tr>
  <tr>
    <td>Ubuntu LTS 18.04 またはそれ以降</td>
    <td>x86_64</td>
  </tr>
  <tr>
    <td>CentOS 8 Stream</td>
    <td><ul><li>x86_64</li><li>ARM 64</li></ul></td>
  </tr>
  <tr>
    <td>Debian 9（Stretch）またはそれ以降</td>
    <td>x86_64</td>
  </tr>
  <tr>
    <td>Fedora 35 またはそれ以降</td>
    <td>x86_64</td>
  </tr>
  <tr>
    <td>openSUSE Leap v15.3以降（Tumbleweedを含まない）</td>
    <td>x86_64</td>
  </tr>
  <tr>
    <td>Rocky Linux 9.1 またはそれ以降</td>
    <td><ul><li>x86_64</li><li>ARM 64</li></ul></td>
  </tr>
  <tr>
    <td>SUSE Linux Enterprise Server 15</td>
    <td>x86_64</td>
  </tr>
</tbody>
</table>

> **注:**
>
> - Oracle Enterprise Linuxについて、TiDBはRed Hat互換カーネル(RHCK)をサポートし、Oracle Enterprise Linuxが提供するUnbreakable Enterprise Kernelはサポートされていません。
> - [CentOS Linux EOL](https://www.centos.org/centos-linux-eol/)によると、CentOS Linux 8の上流サポートは2021年12月31日に終了しました。CentOS Stream 8は引き続きCentOS組織によってサポートされます。
> - Ubuntu 16.04のサポートは将来のTiDBのバージョンで削除されます。Ubuntu 18.04以降へのアップグレードを強く推奨します。
> - 前述のテーブルに記載されているオペレーティングシステムの32ビット版を使用している場合、TiDBのコンパイル、ビルド、またはデプロイの成否は**保証されていません**し、対応するCPUアーキテクチャについても対応しない場合があります。
> - 上記に記載されていないその他のオペレーティングシステムバージョンも動作する場合がありますが、公式にはサポートされていません。

### TiDBのコンパイルおよび実行に必要なライブラリ

|  コンパイルおよび実行に必要なライブラリ |  バージョン   |
|   :---   |   :---   |
|   Golang  |  1.21 またはそれ以降 |
|   Rust    |   nightly-2022-07-31 またはそれ以降  |
|  GCC      |   7.x      |
|  LLVM     |  13.0 またはそれ以降  |

TiDBの実行に必要なライブラリ: glibc (バージョン2.28-151.el8)

### Dockerイメージの依存関係

以下のCPUアーキテクチャがサポートされています:

- x86_64。TiDB v6.6.0からは、[x84-64-v2命令セット](https://developers.redhat.com/blog/2021/01/05/building-red-hat-enterprise-linux-9-for-the-x86-64-v2-microarchitecture-level)が必要となります。
- ARM 64

## ソフトウェアの推奨事項

### 制御マシン

| ソフトウェア | バージョン |
| :--- | :--- |
| sshpass | 1.06 またはそれ以降 |
| TiUP | 1.5.0 またはそれ以降 |

> **注:**
>
> [制御マシンにTiUPを展開](/production-deployment-using-tiup.md#step-2-deploy-tiup-on-the-control-machine)することが必須です。これによりTiDBクラスターの操作と管理が可能となります。

### ターゲットマシン

| ソフトウェア | バージョン |
| :--- | :--- |
| sshpass | 1.06 またはそれ以降 |
| numa | 2.0.12 またはそれ以降 |
| tar | 任意 |

## サーバーの推奨事項

TiDBをIntel x86-64アーキテクチャの64ビット汎用ハードウェアサーバープラットフォームまたはARMアーキテクチャのハードウェアサーバープラットフォームで展開および実行できます。開発、テスト、および本番環境向けのサーバーハードウェア構成に関する要件および推奨事項（オペレーティングシステム自体により消費されるリソースを無視）は以下の通りです：

### 開発およびテスト環境

| コンポーネント | CPU     | メモリ | ローカルストレージ  | ネットワーク  | インスタンス数（最低要件） |
| :------: | :-----: | :-----: | :----------: | :------: | :----------------: |
| TiDB    | 8コア以上   | 16 GB以上  | [ディスクスペース要件](#disk-space-requirements) | ギガビットネットワークカード | 1（PDと同じマシンに展開可能）      |
| PD      | 4コア以上   | 8 GB以上  | SAS, 200 GB以上 | ギガビットネットワークカード | 1（TiDBと同じマシンに展開可能）       |
| TiKV    | 8コア以上   | 32 GB以上  | SAS, 200 GB以上 | ギガビットネットワークカード | 3       |
| TiFlash | 32コア以上  | 64 GB以上  | SSD, 200 GB以上 | ギガビットネットワークカード | 1     |
| TiCDC | 8コア以上 | 16 GB以上 | SAS, 200 GB以上 | ギガビットネットワークカード | 1 |

> **注:**
>
> - テスト環境では、TiDBおよびPDインスタンスを同じサーバーに展開できます。
> - パフォーマンスに関連するテストには、低性能なストレージおよびネットワークハードウェア構成を使用しないでください。これによりテスト結果の正確性が確保されます。
> - TiKVサーバーでは、より高速な読み書きを確保するためにNVMe SSDの使用が推奨されます。
> - 機能のテストおよび検証のみを行う場合は、[TiDBのクイックスタートガイド](/quick-start-with-tidb.md)に従ってTiDBを単一のマシンに展開してください。
> - v6.3.0以降、Linux AMD64アーキテクチャでTiFlashを展開するには、CPUがAVX2命令セットをサポートしている必要があります。`cat /proc/cpuinfo | grep avx2`に出力があることを確認してください。Linux ARM64アーキテクチャでTiFlashを展開するには、CPUがARMv8命令セットアーキテクチャをサポートしている必要があります。`cat /proc/cpuinfo | grep 'crc32' | grep 'asimd'`に出力があることを確認してください。命令セットの拡張を使用することで、TiFlashのベクタライゼーションエンジンがより良いパフォーマンスを提供します。

### 本番環境
| コンポーネント | CPU | メモリ | ハードディスクタイプ | ネットワーク | インスタンス数（最小要件） |
| :-----: | :------: | :------: | :------: | :------: | :-----: |
| TiDB | 16 コア以上 | 48 GB以上 | SSD | 10ギガビットネットワークカード（2つを希望） | 2 |
| PD | 8 コア以上 | 16 GB以上 | SSD | 10ギガビットネットワークカード（2つを希望） | 3 |
| TiKV | 16 コア以上 | 64 GB以上 | SSD | 10ギガビットネットワークカード（2つを希望） | 3 |
| TiFlash | 48 コア以上 | 128 GB以上 | SSD 1つ以上 | 10ギガビットネットワークカード（2つを希望） | 2 |
| TiCDC | 16 コア以上 | 64 GB以上 | SSD | 10ギガビットネットワークカード（2つを希望） | 2 |
| Monitor | 8 コア以上 | 16 GB以上 | SAS | ギガビットネットワークカード | 1 |

> **注記:**
>
> - 本番環境では、TiDBとPDのインスタンスを同じサーバーに展開できます。性能と信頼性に高い要件がある場合は、別々に展開してください。
> - 本番環境では、TiDB、TiKV、TiFlashをそれぞれ少なくとも8つのCPUコアで構成することを強くお勧めします。さらなるパフォーマンスを得るためには、より高い構成が推奨されます。
> - TiKVのハードディスクサイズは、PCIe SSDを使用している場合は4 TB以内、通常のSSDを使用している場合は1.5 TB以内にしてください。

TiFlashを展開する前に、以下の事項に注意してください:

- [複数のディスクに展開](/tiflash/tiflash-configuration.md#multi-disk-deployment)することができます。
- TiFlashデータディレクトリの最初のディスクに高性能SSDを使用することをお勧めします。このディスクの性能はPCIe SSDなどのTiKVと同等以上であることが望ましいです。ディスクの容量は総容量の10%未満である必要があり、そうでない場合はノードのボトルネックになる可能性があります。他のディスクには通常のSSDを展開することができますが、より優れたPCIe SSDを使用するとパフォーマンスが向上します。
- TiFlashをTiKVとは異なるノードに展開することをお勧めします。TiFlashとTiKVを同じノードに展開する必要がある場合は、CPUコア数とメモリを増やし、TiFlashとTiKVを異なるディスクに展開するようにしてください。

TiFlashディスクの総容量は次のように計算されます: `全体のTiKVクラスターのデータ容量の複製数 / TiKVの複製数 * TiFlashの複製数`。例えば、TiKVの全体の計画容量が1 TBで、TiKVの複製数が3個でTiFlashの複製数が2個の場合、TiFlashの推奨総容量は`1024 GB / 3 * 2`となります。特定のテーブルのデータのみを複製することもできます。その場合は、複製するテーブルのデータ容量に応じてTiFlashの容量を決定してください。

TiCDCを展開する前に、TiCDCをPCIe SSDディスクに500 GB以上展開することをお勧めします。

## ネットワーク要件

TiDBはオープンソースの分散SQLデータベースであり、実際の環境でのTiDB展開には、管理者が関連するポートをネットワーク側とホスト側で開く必要があります。

| コンポーネント | デフォルトポート | 説明 |
| :--:| :--: | :-- |
| TiDB |  4000  | アプリケーションとDBAツール間の通信ポート |
| TiDB | 10080  | TiDBの状態を報告する通信ポート |
| TiKV | 20160 | TiKV間の通信ポート |
| TiKV |  20180 | TiKVの状態を報告する通信ポート |
| PD | 2379 | TiDBとPD間の通信ポート |
| PD | 2380 | PDクラスター内でのノード間通信ポート |
| TiFlash | 9000 | TiFlash TCPサービスポート |
| TiFlash | 3930 | TiFlash RAFTおよびコプロセッササービスポート |
| TiFlash | 20170 | TiFlashプロキシサービスポート |
| TiFlash | 20292 | PrometheusがTiFlashプロキシメトリクスを取得するポート |
| TiFlash | 8234 | PrometheusがTiFlashメトリクスを取得するポート |
| Pump | 8250 | Pump通信ポート |
| Drainer | 8249 | Drainer通信ポート |
| TiCDC | 8300 | TiCDC通信ポート |
| Monitoring | 9090 | Prometheusサービスの通信ポート |
| Monitoring | 12020 | NgMonitoringサービスの通信ポート |
| Node_exporter | 9100 | 各TiDBクラスターノードのシステム情報を報告する通信ポート |
| Blackbox_exporter | 9115 | Blackbox_exporter通信ポート。TiDBクラスターのポートを監視するために使用されます |
| Grafana | 3000 | 外部Web監視サービスおよびクライアント（ブラウザ）アクセスのためのポート |
| Alertmanager | 9093 | アラートWebサービスのポート |
| Alertmanager | 9094 | アラート通信ポート |

## ディスク容量要件

<table>
<thead>
  <tr>
    <th>コンポーネント</th>
    <th>ディスク容量要件</th>
    <th>正常なディスク使用率</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>TiDB</td>
    <td><ul><li>ログディスク用に少なくとも30 GB</li><li>v6.5.0以降は、Fast Online DDL（<a href="https://docs.pingcap.com/tidb/dev/system-variables#tidb_ddl_enable_fast_reorg-new-in-v630">tidb_ddl_enable_fast_reorg</a>変数で制御）がデフォルトで有効になっており、インデックスの追加などのDDL操作を高速化します。アプリケーションに大きなオブジェクトを含むDDL操作が存在する場合や、<a href="https://docs.pingcap.com/tidb/dev/sql-statement-import-into">IMPORT INTO</a>を使用してデータをインポートしたい場合は、TiDBに追加のSSDディスクを準備することを強くお勧めします（100 GB以上）。詳細な構成手順については、<a href="https://docs.pingcap.com/tidb/dev/check-before-deployment#set-temporary-spaces-for-tidb-instances-recommended">TiDBインスタンスの一時スペースを設定</a>を参照してください</li></ul></td>
    <td>90%未満</td>
  </tr>
  <tr>
    <td>PD</td>
    <td>データディスクおよびログディスク用に少なくとも20 GBずつ</td>
    <td>90%未満</td>
  </tr>
  <tr>
    <td>TiKV</td>
    <td>データディスクおよびログディスク用に少なくとも100 GBずつ</td>
    <td>80%未満</td>
  </tr>
  <tr>
    <td>TiFlash</td>
    <td>データディスク用に少なくとも100 GB、ログディスク用に少なくとも30 GB</td>
    <td>80%未満</td>
  </tr>
  <tr>
    <td>TiUP</td>
    <td><ul><li>制御マシン: 単一バージョンのTiDBクラスターを展開するために1 GB以上のスペースが必要です。複数のバージョンのTiDBクラスターを展開する場合、必要なスペースが増加します。</li><li>デプロイメントサーバー（TiDBコンポーネントが実行されるマシン）: TiFlashは約700 MBのスペースを占有し、PD、TiDB、TiKVなどの他のコンポーネントはそれぞれ約200 MBのスペースを占有します。クラスターデプロイプロセス中、TiUPクラスターは一時スペース（<code>/tmp</code>ディレクトリ）に一時ファイルを保存するため、1 MB未満の一時スペースが必要です。</li></ul></td>
    <td>N/A</td>
  </tr>
  <tr>
    <td>Ngmonitoring</td>
    <td><ul><li>Conprof: 3 x 1 GB x コンポーネント数（各コンポーネントは1日あたり約1 GBのスペースを占有し、合計3日分）+ 20 GBの予約スペース</li><li>Top SQL: 30 x 50 MB x コンポーネント数（各コンポーネントは1日あたり約50 MBのスペースを占有し、合計30日分）</li><li>ConprofおよびTop SQLは予約スペースを共有します</li></ul></td>
    <td>N/A</td>
  </tr>
</tbody>
</table>

## Webブラウザの要件

TiDBは、データベースメトリクスの可視化を提供するために[Grafana](https://grafana.com/)を使用しています。JavaScriptを有効にした最新バージョンのInternet Explorer、Chrome、またはFirefoxが必要です。