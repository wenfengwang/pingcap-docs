---
title: TiDBでHAProxyを使用するためのベストプラクティス
summary: このドキュメントでは、TiDBでのHAProxyの構成および使用のベストプラクティスについて説明します。
aliases: ['/docs/dev/best-practices/haproxy-best-practices/', '/docs/dev/reference/best-practices/haproxy/']
---

# TiDBでHAProxyを使用するためのベストプラクティス

このドキュメントでは、TiDBでの[HAProxy](https://github.com/haproxy/haproxy)の構成および使用のベストプラクティスについて説明します。HAProxyはTCPベースのアプリケーションのロードバランシングを提供します。TiDBのクライアントからは、HAProxyが提供する浮動仮想IPアドレスに接続することでデータを操作でき、TiDBサーバーレイヤーでの負荷分散を実現できます。

![TiDBでのHAProxyベストプラクティス](/media/haproxy.jpg)

> **注:**
>
> すべてのTiDBのバージョンで動作するHAProxyの最小バージョンはv1.5です。v1.5からv2.1の間では、`mysql-check`で`post-41`オプションを設定する必要があります。HAProxy v2.2以降を使用することを推奨します。

## HAProxyの概要

HAProxyは、C言語で書かれた無料のオープンソースソフトウェアであり、TCPおよびHTTPベースのアプリケーションのための高可用性のロードバランサーおよびプロキシサーバーを提供します。CPUとメモリの効率的な使用により、HAProxyは現在、GitHub、Bitbucket、Stack Overflow、Reddit、Tumblr、Twitter、Tuenti、AWS（Amazon Web Services）など多くの有名なウェブサイトによって広く使用されています。

HAProxyは、Linuxカーネルのコア貢献者であるWilly Tarreauによって2000年に作成され、現在もプロジェクトのメンテナンスを担当し、オープンソースコミュニティで無料のソフトウェア更新を提供しています。このガイドでは、HAProxy [2.6](https://www.haproxy.com/blog/announcing-haproxy-2-6/) を使用しています。最新の安定版を使用することをお勧めします。詳細については、[HAProxyのリリースバージョン](http://www.haproxy.org/)を参照してください。

## 基本的な機能

- [高可用性](http://cbonte.github.io/haproxy-dconv/2.6/intro.html#3.3.4): HAProxyは優雅なシャットダウンとシームレスな切り替えをサポートして高可用性を提供します。
- [ロードバランシング](http://cbonte.github.io/haproxy-dconv/2.6/configuration.html#4.2-balance): 2つの主要なプロキシモードがサポートされています。TCP（レイヤー4とも呼ばれる）とHTTP（レイヤー7とも呼ばれる）。roundrobin、leastconn、randomなど9つ以上のロードバランシングアルゴリズムがサポートされています。
- [ヘルスチェック](http://cbonte.github.io/haproxy-dconv/2.6/configuration.html#5.2-check): HAProxyは定期的にサーバーのHTTPまたはTCPモードの状態をチェックします。
- [スティッキーセッション](http://cbonte.github.io/haproxy-dconv/2.6/intro.html#3.3.6): アプリケーションがスティッキーセッションをサポートしていない場合、HAProxyはクライアントを特定のサーバーに固定します。
- [SSL](http://cbonte.github.io/haproxy-dconv/2.6/intro.html#3.3.2): HTTPS通信と解決がサポートされています。
- [監視および統計情報](http://cbonte.github.io/haproxy-dconv/2.6/intro.html#3.3.3): webページを通じて、サービスの状態とトラフィックフローをリアルタイムで監視できます。

## 開始する前に

HAProxyを展開する前に、ハードウェアおよびソフトウェアの要件を満たしていることを確認してください。

### ハードウェア要件

サーバーについて、以下のハードウェア要件を満たすことを推奨しています。負荷分散環境に応じて、サーバースペックを向上させることもできます。

| ハードウェアリソース | 最小の仕様      |
| :--------------------- | :-------------------- |
| CPU                    | 2つのコア、3.5 GHz      |
| メモリ                 | 16 GB                 |
| ストレージ              | 50 GB（SATA）          |
| ネットワークインターフェイスカード | 10G ネットワークカード      |

### ソフトウェア要件

以下のオペレーティングシステムを使用し、必要な依存関係がインストールされていることを確認します。HAProxyをインストールするためにyumを使用する場合、依存関係はそれと一緒にインストールされるため、別途インストールする必要はありません。

#### オペレーティングシステム

| Linuxディストリビューション       | バージョン         |
| :----------------------- | :----------- |
| Red Hat Enterprise Linux | 7または8   |
| CentOS                   | 7または8   |
| Oracle Enterprise Linux  | 7または8   |
| Ubuntu LTS               | 18.04またはそれ以降のバージョン |

> **注:**
>
> - 他のサポートされているオペレーティングシステムの詳細については、[HAProxyのドキュメント](https://github.com/haproxy/haproxy/blob/master/INSTALL)を参照してください。

#### 依存関係

- epel-release
- gcc
- systemd-devel

上記の依存関係をインストールするには、次のコマンドを実行します：

{{< copyable "shell-regular" >}}

```bash
yum -y install epel-release gcc systemd-devel
```

## HAProxyの展開

HAProxyを使用して、簡単にロードバランスされたデータベース環境を構成およびセットアップできます。このセクションでは、一般的な展開操作を示します。実際のシナリオに基づいて[構成ファイル](http://cbonte.github.io/haproxy-dconv/2.6/configuration.html)をカスタマイズできます。

### HAProxyのインストール

1. HAProxy 2.6.2のソースコードパッケージをダウンロードします：

    {{< copyable "shell-regular" >}}

    ```bash
    wget https://www.haproxy.org/download/2.6/src/haproxy-2.6.2.tar.gz
    ```

2. パッケージを展開します：

    {{< copyable "shell-regular" >}}

    ```bash
    tar zxf haproxy-2.6.2.tar.gz
    ```

3. ソースコードからアプリケーションをコンパイルします：

    {{< copyable "shell-regular" >}}

    ```bash
    cd haproxy-2.6.2
    make clean
    make -j 8 TARGET=linux-glibc USE_THREAD=1
    make PREFIX=${/app/haproxy} SBINDIR=${/app/haproxy/bin} install  # `${/app/haproxy}`および`${/app/haproxy/bin}`をカスタムディレクトリに置き換えてください。
    ```

4. プロファイルを再構成します：

    {{< copyable "shell-regular" >}}

    ```bash
    echo 'export PATH=/app/haproxy/bin:$PATH' >> /etc/profile
    . /etc/profile
    ```

5. インストールが成功したかどうかを確認します：

    {{< copyable "shell-regular" >}}

    ```bash
    which haproxy
    ```

#### HAProxyコマンド

キーワードとその基本的な使用法の一覧を表示するには、次のコマンドを実行します：

{{< copyable "shell-regular" >}}

```bash
haproxy --help
```

| オプション | 説明 |
| :-------| :---------|
| `-v` | バージョンとビルド日付を報告します。 |
| `-vv` | バージョン、ビルドオプション、ライブラリのバージョン、使用可能なポーラーなどを表示します。 |
| `-d` | デバッグモードを有効にします。 |
| `-db` | バックグラウンドモードおよびマルチプロセスモードを無効にします。 |
| `-dM [<バイト>]` | `<バイト>`を使用してメモリ毒剤を課します。これは、malloc()またはpool_alloc2()で割り当てられたすべてのメモリ領域が呼び出し元に渡される前に、`<バイト>`で埋められることを意味します。 |
| `-V` | 冗長モードを有効にします（クワイエットモードを無効にします）。 |
| `-D` | デーモンとして開始します。|
| `-C <ディレクトリ>` | 設定ファイルを読み込む前にディレクトリ`<ディレクトリ>`に変更します。 |
| `-W` | マスターワーカーモード。 |
| `-q` | 「クワイエット」モードを設定します：これにより、構成解析および起動時の一部のメッセージが無効になります。 |
| `-c` | 構成ファイルのチェックのみを行い、バインドを試みる前に終了します。 |
| `-n <制限値>` | プロセスごとの接続制限を`<制限値>`に制限します。 |
| `-m <制限値>` | すべてのプロセス全体で割り当て可能なメモリを`<制限値>`メガバイトに制限します。 |
| `-N <制限値>` | デフォルトのプロキシごとのmaxconnを`<制限値>`に設定します（通常は2000）。 |
| `-L <名前>` | ローカルピア名を`<名前>`に変更します。これはデフォルトでローカルホスト名になります。 |
| `-p <ファイル>` | 起動時にすべてのプロセスのPIDを`<ファイル>`に書き込みます。 |
| `-de` | epoll(7)の使用を無効にします。epoll(7)はLinux 2.6および一部のカスタムLinux 2.4システムでのみ利用可能です。 |
| `-dp` | poll(2)の使用を無効にします。代わりにselect(2)が使用される場合があります。 |
| `-dS` | 古いカーネルで壊れているsplice(2)の使用を無効にします。 |
| `-dR` | SO_REUSEPORTの使用を無効にします。 |
| `-dr` | サーバーアドレスの解決に失敗しても無視します。 |
| `-dV` | サーバーサイドでSSLの検証を無効にします。 |
| `-sf <pidlist>` | 起動後にpidlistのPIDに「終了」シグナルを送信します。このシグナルを受け取ったプロセスは終了前にすべてのセッションが終わるのを待ちます。このオプションは最後に指定する必要があり、その後に任意の数のPIDを続けます。技術的には、SIGTTOUおよびSIGUSR1が送信されます。 |
| `-st <pidlist>` | 「terminate」シグナルを起動後にpidlist内のPIDに送信します。このシグナルを受け取ったプロセスは即座に終了し、すべてのアクティブなセッションを閉じます。このオプションは最後に指定する必要があり、任意の数のPIDが続きます。技術的には、SIGTTOUとSIGTERMが送信されます。 |
| `-x <unix_socket>` | 指定したソケットに接続し、古いプロセスからすべてのリスニングソケットを取得します。その後、これらのソケットは新しいものをバインドする代わりに使用されます。 |
| `-S <bind>[,<bind_options>...]` | マスター-ワーカーモードでは、マスターCLIを作成します。このCLIはすべてのワーカーのCLIにアクセスを有効にします。デバッグに役立ち、終了するプロセスにアクセスする便利な方法です。 |

HAProxyコマンドラインオプションの詳細については、[HAProxyのManagement Guide](http://cbonte.github.io/haproxy-dconv/2.6/management.html) および[HAProxyのGeneral Commands Manual](https://manpages.debian.org/buster-backports/haproxy/haproxy.1.en.html) を参照してください。

### HAProxyの設定

HAProxyをインストールする際にyumを使用すると、設定テンプレートが生成されます。また、以下の構成項目をシナリオに応じてカスタマイズすることもできます。

```yaml
global                                     # グローバル構成。
   log         127.0.0.1 local2            # グローバルシスログサーバー（最大2つまで）。
   chroot      /var/lib/haproxy            # 現在のディレクトリを変更し、スタートアッププロセスのスーパーユーザー特権を設定してセキュリティを向上させます。
   pidfile     /var/run/haproxy.pid        # HAProxyプロセスのPIDをこのファイルに書き込みます。
   maxconn     4096                        # 単一のHAProxyプロセスの同時接続の最大数。コマンドライン引数「-n」と同等です。
   nbthread    48                          # スレッドの最大数。 （上限はCPUの数と同じです）
   user        haproxy                     # UIDパラメータと同じです。
   group       haproxy                     # GIDパラメータと同じです。専用のユーザーグループが推奨されます。
   daemon                                  # プロセスをバックグラウンドでフォークします。これはコマンドライン「-D」引数と等価です。コマンドライン「-db」の引数によって無効にすることもできます。
   stats socket /var/lib/haproxy/stats     # 統計情報出力が保存されるディレクトリ。

defaults                                   # デフォルト構成。
   log global                              # グローバル構成の設定を継承します。
   retries 2                               # アップストリームサーバーに接続する最大リトライ回数。接続試行回数がこの値を超えると、バックエンドサーバーは利用不可と見なされます。
   timeout connect  2s                     # バックエンドサーバーへの接続試行が成功するまでの最大待ち時間。サーバーがHAProxyと同じLANにある場合は、より短い時間に設定する必要があります。
   timeout client 30000s                   # クライアント側の最大非アクティブ時間。
   timeout server 30000s                   # サーバー側の最大非アクティブ時間。

listen admin_stats                         # フロントエンドとバックエンドからの情報を報告するStatsページの名前。必要に応じて名前をカスタマイズできます。
   bind 0.0.0.0:8080                       # リスニングポート。
   mode http                               # モニタリングモード。
   option httplog                          # HTTPロギングを有効にします。
   maxconn 10                              # 最大同時接続数。
   stats refresh 30s                       # 30秒ごとに自動的にStatsページを更新します。
   stats uri /haproxy                      # StatsページのURL。
   stats realm HAProxy                     # Statsページの認証領域。
   stats auth admin:pingcap123             # Statsページのユーザー名とパスワード。複数のユーザー名を持つことができます。
   stats hide-version                      # StatsページでのHAProxyのバージョン情報を非表示にします。
   stats admin if TRUE                     # バックエンドサーバーを手動で有効または無効にします（HAProxy 1.4.9以降のバージョンでサポート）。

listen tidb-cluster                        # データベース負荷分散。
   bind 0.0.0.0:3390                       # 浮動IPアドレスとリスニングポート。
   mode tcp                                # HAProxyはレイヤー4、トランスポートレイヤーを使用します。
   balance leastconn                       # 接続を受信する最も少ない接続数を持つサーバーが接続を受けます。長いセッションを予想されるLDAP、SQL、TSEなどのプロトコルには「leastconn」が推奨されます。アルゴリズムは動的です。つまり、サーバーのウェイトは遅い開始のためにフライで調整される場合があります。
   server tidb-1 10.9.18.229:4000 check inter 2000 rise 2 fall 3       # 2000ミリ秒ごとにポート4000を検出します。2回の成功と3回の失敗でサーバーが利用可能または利用不可と見なされます。
   server tidb-2 10.9.39.208:4000 check inter 2000 rise 2 fall 3
   server tidb-3 10.9.64.166:4000 check inter 2000 rise 2 fall 3
```

`SHOW PROCESSLIST`を使用してソースIPアドレスを確認するには、TiDBに接続するために[PROXYプロトコル](https://www.haproxy.org/download/1.8/doc/proxy-protocol.txt)を構成する必要があります。

```yaml
   server tidb-1 10.9.18.229:4000 send-proxy check inter 2000 rise 2 fall 3
   server tidb-2 10.9.39.208:4000 send-proxy check inter 2000 rise 2 fall 3
   server tidb-3 10.9.64.166:4000 send-proxy check inter 2000 rise 2 fall 3
```

> **注記:**
>
> PROXYプロトコルを使用する前に、TiDBサーバーの構成ファイルで[`proxy-protocol.networks`](/tidb-configuration-file.md#networks)を構成する必要があります。

### HAProxyの起動

HAProxyを起動するには、`haproxy`を実行してください。デフォルトで`/etc/haproxy/haproxy.cfg`が読み込まれます（推奨）。

{{< copyable "shell-regular" >}}

```bash
haproxy -f /etc/haproxy/haproxy.cfg
```

### HAProxyの停止

HAProxyを停止するには、`kill -9`コマンドを使用します。

1. 以下のコマンドを実行してください：

    {{< copyable "shell-regular" >}}

    ```bash
    ps -ef | grep haproxy
    ```

2. HAProxyのプロセスを終了してください：

    {{< copyable "shell-regular" >}}

    ```bash
    kill -9 ${haproxy.pid}
    ```