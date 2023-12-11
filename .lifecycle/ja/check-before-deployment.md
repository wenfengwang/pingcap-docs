---
title: TiDB環境とシステム構成のチェック
summary: TiDBを展開する前に環境のチェック操作を学びます。
aliases: ['/docs/dev/check-before-deployment/']
---

# TiDB環境とシステム構成のチェック

このドキュメントでは、TiDBを展開する前の環境チェック操作について説明します。以下の手順は優先度に従って並べられています。

## TiKVを展開するターゲットマシンに対して、データディスクのext4ファイルシステムをマウントします

本番環境でTiKVデータを保存するためには、EXT4ファイルシステムのNVMe SSDの使用を推奨します。この構成は大量のオンラインシナリオでの信頼性、セキュリティ、安定性が証明されているベストプラクティスです。

`root`ユーザーアカウントを使用してターゲットマシンにログインします。

データディスクをext4ファイルシステムにフォーマットし、`nodelalloc`および`noatime`マウントオプションをファイルシステムに追加します。`nodelalloc`オプションを追加することが必須です。さもなくば、TiUPの展開が事前チェックに合格できません。`noatime`オプションはオプショナルです。

> **注記:**
>
> データディスクがext4にフォーマットされ、マウントオプションが追加されている場合、「umount /dev/nvme0n1p1」というコマンドを実行してアンインストールし、直接次に進んで`/etc/fstab`ファイルを編集し、再度マウントオプションをファイルシステムに追加します。

`/dev/nvme0n1`データディスクを対象に例を挙げます:

1. データディスクを表示します。

    {{< copyable "shell-root" >}}

    ```bash
    fdisk -l
    ```

    ```
    Disk /dev/nvme0n1: 1000 GB
    ```

2. パーティションを作成します。

    {{< copyable "shell-root" >}}

    ```bash
    parted -s -a optimal /dev/nvme0n1 mklabel gpt -- mkpart primary ext4 1 -1
    ```

    > **注記:**
    >
    > パーティションのデバイス番号を表示するには、`lsblk`コマンドを使用します。NVMeディスクの場合、生成されたデバイス番号は通常`nvme0n1p1`です。通常のディスクの場合（たとえば`/dev/sdb`）、生成されたデバイス番号は通常`sdb1`です。

3. データディスクをext4ファイルシステムにフォーマットします。

    {{< copyable "shell-root" >}}

    ```bash
    mkfs.ext4 /dev/nvme0n1p1
    ```

4. データディスクのパーティションUUIDを表示します。

    この例では、nvme0n1p1のUUIDは`c51eb23b-195c-4061-92a9-3fad812cc12f`です。

    {{< copyable "shell-root" >}}

    ```bash
    lsblk -f
    ```

    ```
    NAME    FSTYPE LABEL UUID                                 MOUNTPOINT
    sda
    ├─sda1  ext4         237b634b-a565-477b-8371-6dff0c41f5ab /boot
    ├─sda2  swap         f414c5c0-f823-4bb1-8fdf-e531173a72ed
    └─sda3  ext4         547909c1-398d-4696-94c6-03e43e317b60 /
    sr0
    nvme0n1
    └─nvme0n1p1 ext4         c51eb23b-195c-4061-92a9-3fad812cc12f
    ```

5. `/etc/fstab`ファイルを編集し、`nodelalloc`マウントオプションを追加します。

    {{< copyable "shell-root" >}}

    ```bash
    vi /etc/fstab
    ```

    ```
    UUID=c51eb23b-195c-4061-92a9-3fad812cc12f /data1 ext4 defaults,nodelalloc,noatime 0 2
    ```

6. データディスクをマウントします。

    {{< copyable "shell-root" >}}

    ```bash
    mkdir /data1 && \
    mount -a
    ```

7. 次のコマンドを使用してチェックします。

    {{< copyable "shell-root" >}}

    ```bash
    mount -t ext4
    ```

    ```
    /dev/nvme0n1p1 on /data1 type ext4 (rw,noatime,nodelalloc,data=ordered)
    ```

    ファイルシステムがext4であり、マウントオプションに`nodelalloc`が含まれている場合、データディスクのext4ファイルシステムをターゲットマシンにマウントする操作は成功です。

## システムスワップをチェックおよび無効化します

TiDBでは適切なメモリスペースが必要です。メモリが不足すると、バッファとしてスワップを使用することはパフォーマンスを低下させる可能性があります。そのため、次のコマンドを実行して、システムスワップを永久的に無効化することを推奨します:

{{< copyable "shell-regular" >}}

```bash
echo "vm.swappiness = 0">> /etc/sysctl.conf
swapoff -a && swapon -a
sysctl -p
```

> **注記:**
>
> - `swapoff -a`を実行してから`swapon -a`を実行してスワップをリフレッシュします。スワップの変更を取り消して`swapoff -a`のみを実行すると、システムを再起動するとスワップが再度有効になります。
>
> - `sysctl -p`はシステムを再起動せずに設定を有効にするためのものです。

## TiDBインスタンスの一時領域を設定します（推奨）

TiDBの一部の操作では、サーバーに一時ファイルを書き込む必要があるため、TiDBを実行するオペレーティングシステムユーザーが対象ディレクトリに読み取りおよび書き込み権限を持っていることが必要です。`root`特権でTiDBインスタンスを起動しない場合は、ディレクトリの権限をチェックして適切に設定する必要があります。

- TiDB作業領域

    ハッシュテーブル構築およびソートなどの大量のメモリを消費する操作は、メモリ消費を減らし、安定性を向上するために一時データをディスクに書き込むことがあります。書き込みのためのディスクの場所は、構成項目[`tmp-storage-path`](/tidb-configuration-file.md#tmp-storage-path)で定義されています。デフォルトの構成では、TiDBを実行するユーザーがオペレーティングシステムの一時フォルダ（通常は`/tmp`）に読み取りおよび書き込み権限を持っていることを確認してください。

- `Fast Online DDL`作業領域

    変数[`tidb_ddl_enable_fast_reorg`](/system-variables.md#tidb_ddl_enable_fast_reorg-new-in-v630)が`ON`に設定されている場合（v6.5.0および以降のバージョンでのデフォルト値）、`Fast Online DDL`が有効になり、一部のDDL操作にはファイルシステムで一時ファイルの読み書きが必要です。場所は構成項目[`temp-dir`](/tidb-configuration-file.md#temp-dir-new-in-v630)で定義されています。デフォルトのディレクトリ`/tmp/tidb`を例に挙げます:

    > **注記:**
    >
    > アプリケーションで大きなオブジェクトのDDL操作が存在する場合は、[`temp-dir`](/tidb-configuration-file.md#temp-dir-new-in-v630)に独立した大規模なファイルシステムを構成することを強く推奨します。

    ```shell
    sudo mkdir /tmp/tidb
    ```

    もし既に`/tmp/tidb`ディレクトリが存在する場合、書き込み権限が与えられていることを確認してください。

    ```shell
    sudo chmod -R 777 /tmp/tidb
    ```

    > **注記:**
    >
    > ディレクトリが存在しない場合、TiDBは起動時に自動的に作成します。ディレクトリの作成に失敗した場合やTiDBがそのディレクトリに対して読み書き権限を持っていない場合、[`Fast Online DDL`](/system-variables.md#tidb_ddl_enable_fast_reorg-new-in-v630)は実行時に予測不能な問題を経験する可能性があります。

## ターゲットマシンのファイアウォールサービスをチェックおよび停止します

TiDBクラスターでは、ノード間のアクセスポートを開放して、読み書きの要求やデータのハートビートなどの情報伝送を確保する必要があります。一般のオンラインシナリオでは、データベースとアプリケーションサービスの間、およびデータベースノード間のデータのやり取りはすべてセキュアなネットワーク内で行われます。そのため、特別なセキュリティ要件がない場合は、ターゲットマシンのファイアウォールを停止することをお勧めします。それ以外の場合は、[ポート使用状況](/hardware-and-software-requirements.md#network-requirements)を参照し、ファイアウォールサービスの許可リストに必要なポート情報を追加してください。

このセクションの残りの部分では、ターゲットマシンのファイアウォールサービスの停止方法について説明します。

1. ファイアウォールの状態を確認します。これは、CentOS Linux リリース 7.7.1908 (Core)を例にしています。

    {{< copyable "shell-regular" >}}

    ```shell
    sudo firewall-cmd --state
    sudo systemctl status firewalld.service
    ```

2. ファイアウォールサービスを停止します。

    {{< copyable "shell-regular" >}}

    ```bash
    sudo systemctl stop firewalld.service
    ```

3. ファイアウォールサービスの自動起動を無効化します。

    {{< copyable "shell-regular" >}}

    ```bash
    sudo systemctl disable firewalld.service
    ```

4. ファイアウォールの状態を確認します。

```bash
sudo systemctl status firewalld.service
```

## NTPサービスの確認とインストール

TiDBは、トランザクションのACIDモデルにおける一貫性を保証するため、ノード間でクロック同期が必要な分散データベースシステムです。

現在、クロック同期の一般的な解決策は、Network Time Protocol（NTP）サービスを使用することです。インターネット上の`pool.ntp.org`タイミングサービスを使用するか、オフライン環境で独自のNTPサービスを構築することができます。

NTPサービスがインストールされているか、NTPサーバーと正常に同期されているかを確認するには、以下の手順を実行してください:

1. 次のコマンドを実行します。`running` と戻ってきたら、NTPサービスが実行されています。

    ```bash
    sudo systemctl status ntpd.service
    ```

    ```
    ntpd.service - Network Time Service
    Loaded: loaded (/usr/lib/systemd/system/ntpd.service; disabled; vendor preset: disabled)
    Active: active (running) since 月 2017-12-18 13:13:19 CST; 3s ago
    ```

    - もし `Unit ntpd.service could not be found.` と返ってきたら、次のコマンドを実行して、システムが`ntpd`ではなく`chronyd`を使用してNTPとのクロック同期を行うように設定されているかを確認してください:

        ```bash
        sudo systemctl status chronyd.service
        ```

        ```
        chronyd.service - NTP client/server
        Loaded: loaded (/usr/lib/systemd/system/chronyd.service; enabled; vendor preset: enabled)
        Active: active (running) since 月 2021-04-05 09:55:29 EDT; 3 days ago
        ```

        もし `chronyd` もしくは `ntpd` のどちらも設定されていない場合、システムにどちらもインストールされていないことを示します。まず、`chronyd` もしくは `ntpd` をインストールして、自動的に起動できるようにしてください。デフォルトでは、`ntpd` が使用されます。

        システムが`chronyd`を使用するように構成されている場合、ステップ3に進んでください。

2. `ntpstat`コマンドを実行して、NTPサービスがNTPサーバーと同期しているかどうかを確認してください。

    > **注記:**
    >
    > Ubuntuシステムの場合は、`ntpstat`パッケージをインストールする必要があります。

    ```bash
    ntpstat
    ```

    - `synchronised to NTP server` (NTPサーバーと同期中) と返ってきた場合、同期プロセスは正常です。

        ```
        synchronised to NTP server (85.199.214.101) at stratum 2
        time correct to within 91 ms
        polling server every 1024 s
        ```

    - 以下の状況はNTPサービスが正常に同期されていないことを示します:

        ```
        unsynchronised
        ```

    - 以下の状況はNTPサービスが正常に実行されていないことを示します:

        ```
        Unable to talk to NTP daemon. Is it running?
        ```

3. `chronyc tracking`コマンドを実行して、ChronyサービスがNTPサーバーと同期しているかどうかを確認してください。

    > **注記:**
    >
    > これはNTPdの代わりにChronyを使用するシステムにのみ適用されます。

    ```bash
    chronyc tracking
    ```

    - コマンドが`Leap status : Normal`と返ってきた場合、同期プロセスは正常です。

        ```
        Reference ID    : 5EC69F0A (ntp1.time.nl)
        Stratum         : 2
        Ref time (UTC)  : Thu May 20 15:19:08 2021
        System time     : 0.000022151 seconds slow of NTP time
        Last offset     : -0.000041040 seconds
        RMS offset      : 0.000053422 seconds
        Frequency       : 2.286 ppm slow
        Residual freq   : -0.000 ppm
        Skew            : 0.012 ppm
        Root delay      : 0.012706812 seconds
        Root dispersion : 0.000430042 seconds
        Update interval : 1029.8 seconds
        Leap status     : Normal
        ```

    - コマンドが以下の結果を返した場合、同期にエラーが発生しています:

        ```
        Leap status    : Not synchronised
        ```

    - コマンドが以下の結果を返した場合、`chronyd`サービスが正常に実行されていません:

        ```
        506 Cannot talk to daemon
        ```

NTPサービスをできるだけ早く同期させるためには、以下のコマンドを実行してください。NTPサーバーとして`pool.ntp.org`を使用してください。

```bash
sudo systemctl stop ntpd.service && \
sudo ntpdate pool.ntp.org && \
sudo systemctl start ntpd.service
```

CentOS 7システムにNTPサービスを手動でインストールするには、以下のコマンドを実行してください:

```bash
sudo yum install ntp ntpdate && \
sudo systemctl start ntpd.service && \
sudo systemctl enable ntpd.service
```

## オペレーティングシステムの最適なパラメータの確認と構成

TiDBを本番環境で使用する場合、以下のようにオペレーティングシステムの構成を最適化することをお勧めします:

1. THP (Transparent Huge Pages) を無効にする。データベースのメモリアクセスパターンは連続的でなくばらばらになりがちです。高レベルのメモリ断片化が深刻な場合、THPページの割り当てによりより高い遅延が発生します。
2. ストレージメディアのI/Oスケジューラを `noop` に設定する。高速SSDストレージメディアの場合、カーネルのI/Oスケジューリング操作は性能の低下を引き起こす可能性があります。スケジューラが `noop` に設定されると、カーネルは他の操作を行わずにI/Oリクエストをハードウェアに直接送信するため、パフォーマンスが向上します。また、`noop`スケジューラの適用が良いでしょう。
3. CPU周波数を制御するcpufrequモジュールの `performance`モードを選択する。CPU周波数が最大サポートされる動作周波数に固定されるため、パフォーマンスが最大化されます。

現在のオペレーティングシステムの構成を確認して最適なパラメータを構成するには、以下の手順を実行してください:

1. THPが有効か無効かを確認するために、次のコマンドを実行してください:

    ```bash
    cat /sys/kernel/mm/transparent_hugepage/enabled
    ```

    ```
    [always] madvise never
    ```

    > **注記:**
    >
    > `[always] madvise never` と出力された場合、THPが有効です。無効にする必要があります。

2. データディレクトリが存在するディスクのI/Oスケジューラを確認するために、データディレクトリをsdbおよびsdcディスクに作成したと仮定します:

    ```bash
    cat /sys/block/sd[bc]/queue/scheduler
    ```

    ```
    noop [deadline] cfq
    noop [deadline] cfq
    ```

    > **注記:**
    >
    > `noop [deadline] cfq` が出力された場合、ディスクのI/Oスケジューラが `deadline`モードになっています。`noop`に変更する必要があります。

3. ディスクの `ID_SERIAL` を確認するために、次のコマンドを実行してください:

    ```bash
    udevadm info --name=/dev/sdb | grep ID_SERIAL
    ```

    ```
    E: ID_SERIAL=36d0946606d79f90025f3e09a0c1f9e81
    E: ID_SERIAL_SHORT=6d0946606d79f90025f3e09a0c1f9e81
    ```

    > **注記:**
    >
    > 複数のディスクにデータディレクトリが割り当てられている場合、各ディスクの`ID_SERIAL`を記録するために上記のコマンドを複数回実行する必要があります。

4. cpufreqモジュールの電源ポリシーを確認するために、次のコマンドを実行してください:

    ```bash
    cpupower frequency-info --policy
    ```

    ```
    analyzing CPU 0:
    current policy: frequency should be within 1.20 GHz and 3.10 GHz.
                  The governor "powersave" may decide which speed to use within this range.
    ```

    > **注記:**
    >
    > `The governor "powersave"` が出力された場合、cpufreqモジュールの電源ポリシーは `powersave` です。`performance`に変更する必要があります。仮想マシンまたはクラウドホストを使用している場合は、通常 `Unable to determine current policy` と出力されますが、何も変更する必要はありません。

5. オペレーティングシステムの最適なパラメータを構成するには以下を実行してください:

    + 方法1: tunedを使用する（推奨）:

        1. `tuned-adm list`コマンドを実行して、現在のオペレーティングシステムのtunedプロファイルを確認してください:

            ```bash
            tuned-adm list
            ```

            ```
            利用可能なプロファイル:
            ```
            - バランスの取れた                - 一般的で非特殊なチューニングプロファイル
            - デスクトップ                    - デスクトップ利用ケース向けに最適化
            - HPCコンピュート                 - HPCコンピュートワークロード向けに最適化
            - レイテンシーパフォーマンス      - 決定論的パフォーマンスを向上させるための最適化（消費電力増加の代償）
            - ネットワークレイテンシー        - 低レイテンシーネットワークパフォーマンスに焦点を当て、決定論的なパフォーマンスを向上させるための最適化（消費電力増加の代償）
            - ネットワークスループット        - 古いCPUや40G以上のネットワークでのみ必要な、ストリーミングネットワークスループットの最適化
            - パワーセーブ                   - 低消費電力向けに最適化
            - スループットパフォーマンス    - 一般的なサーバーワークロード全般に優れたパフォーマンスを提供するチューニング
            - 仮想ゲスト                     - 仮想ゲスト内で実行するための最適化
            - 仮想ホスト                     - KVMゲストで実行するための最適化
            現在のアクティブプロファイル: バランスの取れた
            ```

            出力 `現在のアクティブプロファイル: バランスの取れた` は、現在のオペレーティングシステムのチューニングプロファイルが「バランスの取れた」であることを意味します。現在のプロファイルに基づいてオペレーティングシステムの構成を最適化することが推奨されます。

        2. 新しいチューニングプロファイルを作成:

            {{< copyable "shell-regular" >}}

            ```bash
            mkdir /etc/tuned/balanced-tidb-optimal/
            vi /etc/tuned/balanced-tidb-optimal/tuned.conf
            ```

            ```
            [main]
            include=balanced

            [cpu]
            governor=performance

            [vm]
            transparent_hugepages=never

            [disk]
            devices_udev_regex=(ID_SERIAL=36d0946606d79f90025f3e09a0c1fc035)|(ID_SERIAL=36d0946606d79f90025f3e09a0c1f9e81)
            elevator=noop
            ```

            出力 `include=balanced` は、オペレーティングシステムの最適化構成を現在の「バランスの取れた」プロファイルに追加することを意味します。

        3. 新しいチューニングプロファイルを適用:

            {{< copyable "shell-regular" >}}

            ```bash
            tuned-adm profile balanced-tidb-optimal
            ```

    + 方法2: スクリプトを使用して構成します。既に方法1を使用している場合は、この方法はスキップしてください。

        1. デフォルトのカーネルバージョンを確認するために `grubby` コマンドを実行:

            > **注記:**
            >
            > `grubby` を実行する前に `grubby` パッケージをインストールしてください。

            {{< copyable "shell-regular" >}}

            ```bash
            grubby --default-kernel
            ```

            ```bash
            /boot/vmlinuz-3.10.0-957.el7.x86_64
            ```

        2. カーネル構成を変更するために `grubby --update-kernel` を実行:

            {{< copyable "shell-regular" >}}

            ```bash
            grubby --args="transparent_hugepage=never" --update-kernel /boot/vmlinuz-3.10.0-957.el7.x86_64
            ```

            > **注記:**
            >
            > `--update-kernel` の後に実際のデフォルトカーネルバージョンが続きます。

        3. 変更したデフォルトのカーネル構成を確認するために `grubby --info` を実行:

            {{< copyable "shell-regular" >}}

            ```bash
            grubby --info /boot/vmlinuz-3.10.0-957.el7.x86_64
            ```

            > **注記:**
            >
            > `--info` の後に実際のデフォルトカーネルバージョンが続きます。

            ```
            index=0
            kernel=/boot/vmlinuz-3.10.0-957.el7.x86_64
            args="ro crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet LANG=en_US.UTF-8 transparent_hugepage=never"
            root=/dev/mapper/centos-root
            initrd=/boot/initramfs-3.10.0-957.el7.x86_64.img
            title=CentOS Linux (3.10.0-957.el7.x86_64) 7 (Core)
            ```

        4. 現在のカーネル構成を変更して即座にTHPを無効にするために次のコマンドを実行:

            {{< copyable "shell-regular" >}}

            ```bash
            echo never > /sys/kernel/mm/transparent_hugepage/enabled
            echo never > /sys/kernel/mm/transparent_hugepage/defrag
            ```

        5. udevスクリプトにI/Oスケジューラを構成するために次のコマンドを実行:

            {{< copyable "shell-regular" >}}

            ```bash
            vi /etc/udev/rules.d/60-tidb-schedulers.rules
            ```

            ```
            ACTION=="add|change", SUBSYSTEM=="block", ENV{ID_SERIAL}=="36d0946606d79f90025f3e09a0c1fc035", ATTR{queue/scheduler}="noop"
            ACTION=="add|change", SUBSYSTEM=="block", ENV{ID_SERIAL}=="36d0946606d79f90025f3e09a0c1f9e81", ATTR{queue/scheduler}="noop"

            ```

        6. udevスクリプトを適用するために次のコマンドを実行:

            {{< copyable "shell-regular" >}}

            ```bash
            udevadm control --reload-rules
            udevadm trigger --type=devices --action=change
            ```

        7. CPUの電力ポリシーを構成するためのサービスを作成:

            {{< copyable "shell-regular" >}}

            ```bash
            cat  >> /etc/systemd/system/cpupower.service << EOF
            [Unit]
            Description=CPU performance
            [Service]
            Type=oneshot
            ExecStart=/usr/bin/cpupower frequency-set --governor performance
            [Install]
            WantedBy=multi-user.target
            EOF
            ```

        8. CPU電力ポリシー構成サービスを適用するために次のコマンドを実行:

            {{< copyable "shell-regular" >}}

            ```bash
            systemctl daemon-reload
            systemctl enable cpupower.service
            systemctl start cpupower.service
            ```

6. 次のコマンドを実行してTHPの状態を確認:

    {{< copyable "shell-regular" >}}

    ```bash
    cat /sys/kernel/mm/transparent_hugepage/enabled
    ```

    ```
    always madvise [never]
    ```

7. データディレクトリがあるディスクのI/Oスケジューラを確認するために次のコマンドを実行:

    {{< copyable "shell-regular" >}}

    ```bash
    cat /sys/block/sd[bc]/queue/scheduler
    ```

    ```
    [noop] deadline cfq
    [noop] deadline cfq
    ```

8. cpufreqモジュールの電力ポリシーを確認するために次のコマンドを実行:

    {{< copyable "shell-regular" >}}

    ```bash
    cpupower frequency-info --policy
      ```

    ```
    analyzing CPU 0:
    current policy: frequency should be within 1.20 GHz and 3.10 GHz.
                  The governor "performance" may decide which speed to use within this range.
    ```

9. `sysctl`パラメータを変更するために次のコマンドを実行:

    {{< copyable "shell-regular" >}}

    ```bash
    echo "fs.file-max = 1000000">> /etc/sysctl.conf
    echo "net.core.somaxconn = 32768">> /etc/sysctl.conf
    echo "net.ipv4.tcp_tw_recycle = 0">> /etc/sysctl.conf
    echo "net.ipv4.tcp_syncookies = 0">> /etc/sysctl.conf
    echo "vm.overcommit_memory = 1">> /etc/sysctl.conf
    sysctl -p
    ```

10. ユーザーの `limits.conf` ファイルを構成するために次のコマンドを実行:

    {{< copyable "shell-regular" >}}

    ```bash
    cat << EOF >>/etc/security/limits.conf
    tidb           soft    nofile          1000000
    tidb           hard    nofile          1000000
    tidb           soft    stack          32768
    tidb           hard    stack          32768
    EOF
    ```

## SSH相互信頼およびパスワードなしでのsudoの手動構成

このセクションでは、SSH相互信頼とパスワードなしでのsudoの手動構成方法について説明します。TiUPを使用して展開することをお勧めし、TiUPが自動的にSSH相互信頼とパスワードなしでログインを構成します。TiUPを使用してTiDBクラスターを展開する場合は、このセクションを無視してください。

1. `root` ユーザーアカウントを使用してそれぞれターゲットマシンにログインし、`tidb` ユーザーを作成し、ログインパスワードを設定します。

    {{< copyable "shell-root" >}}

    ```bash
    useradd tidb && \
    passwd tidb
    ```

2. パスワードなしでsudoを構成するには、次のコマンドを実行し、ファイルの最後に `tidb ALL=(ALL) NOPASSWD: ALL` を追加します:

    {{< copyable "shell-root" >}}

    ```bash
    visudo
    ```

    ```
    tidb ALL=(ALL) NOPASSWD: ALL
    ```
3. `tidb`ユーザーを使用して制御マシンにログインし、次のコマンドを実行してください。 `10.0.1.1` を対象マシンのIPに置き換え、対象マシンの `tidb` ユーザーのパスワードをプロンプトで入力してください。 コマンドが実行されたら、SSH相互信頼がすでに作成されます。 他のマシンにも適用されます。 新しく作成された `tidb` ユーザーには `.ssh` ディレクトリがありません。 そのようなディレクトリを作成するには、RSA鍵を生成するコマンドを実行してください。 制御マシンにTiDBコンポーネントを展開するには、制御マシンと制御マシン自体に対して相互信頼を構成してください。

    {{< copyable "shell-regular" >}}

    ```bash
    ssh-keygen -t rsa
    ssh-copy-id -i ~/.ssh/id_rsa.pub 10.0.1.1
    ```

4. `tidb`ユーザーアカウントを使用して制御マシンにログインし、`ssh`を使用して対象マシンのIPにログインしてください。 パスワードを入力する必要がなく、正常にログインできる場合、SSH相互信頼が正常に構成されています。

    {{< copyable "shell-regular" >}}

    ```bash
    ssh 10.0.1.1
    ```

    ```
    [tidb@10.0.1.1 ~]$
    ```

5. `tidb`ユーザーを使用して対象マシンにログインした後、次のコマンドを実行してください。 パスワードを入力する必要がなく、`root`ユーザーに切り替えることができる場合、`tidb`ユーザーのパスワードなしでのsudoが正常に構成されています。

    {{< copyable "shell-regular" >}}

    ```bash
    sudo -su root
    ```

    ```
    [root@10.0.1.1 tidb]#
    ```

## `numactl`ツールのインストール

このセクションでは、NUMAツールのインストール方法について説明します。 オンライン環境では、通常、ハードウェア構成が必要以上に高いため、ハードウェアリソースをよりよく計画するために、1台のマシンに複数のTiDBまたはTiKVインスタンスを展開できます。 そのようなシナリオでは、CPUリソースの競合による性能低下を防ぐために、NUMAツールを使用できます。

> **注意:**
>
> - NUMAを使用してコアをバインディングすると、CPUリソースを分離する方法であり、高構成の物理マシンに複数のインスタンスを展開するのに適しています。
> - `tiup cluster deploy`を使用してデプロイが完了したら、`exec`コマンドを使用してクラスターレベルの管理操作を実行できます。

NUMAツールをインストールするには、次の2つの方法のいずれかを選択できます。

**方法1**：対象ノードにログインしてNUMAをインストールします。 CentOS Linux release 7.7.1908 (Core)を例にとっています。

```bash
sudo yum -y install numactl
```

**方法2**：`tiup cluster exec`コマンドを実行して、既存のクラスターにNUMAを一括でインストールします。

1. [TiUPを使用してTiDBクラスターをデプロイ](/production-deployment-using-tiup.md)に従って、クラスター `tidb-test` を展開してください。 TiDBクラスターをインストール済みの場合は、この手順をスキップしてください。

    ```bash
    tiup cluster deploy tidb-test v6.1.0 ./topology.yaml --user root [-p] [-i /home/root/.ssh/gcp_rsa]
    ```

2. `sudo`権限を使用して、`tidb-test`クラスター内のすべての対象マシンにNUMAをインストールするために`tiup cluster exec`コマンドを実行してください:

    ```bash
    tiup cluster exec tidb-test --sudo --command "yum -y install numactl"
    ```

    `tiup cluster exec`コマンドのヘルプ情報を取得するには、`tiup cluster exec --help`コマンドを実行してください。