---
title: DMクラスターにホットフィックスを適用する
summary: DMクラスターにホットフィックスパッチを適用する方法を学びます。
---

# DMクラスターにホットフィックスを適用する

クラスターが実行中の間にサービスのバイナリを動的に置き換える必要がある場合（つまり、置き換え中もクラスターを利用できるようにするために）、`tiup dm patch`コマンドを使用できます。このコマンドは次の動作を行います。

- 置き換えのためのバイナリパッケージを対象マシンにアップロードします。
- 関連するノードをAPIを使用してオフラインにします。
- 対象のサービスを停止します。
- バイナリパッケージを展開し、サービスを置き換えます。
- 対象のサービスを開始します。

## 構文

```shell
tiup dm patch <cluster-name> <package-path> [flags]
```

- `<cluster-name>`: 操作対象のクラスターの名前
- `<package-path>`: 置き換えに使用するバイナリパッケージのパス

### 準備

このコマンドに必要なバイナリパッケージを事前に以下の手順に従って準備する必要があります。

- 置き換えるコンポーネント（dm-master、dm-workerなど）の`${component}`の名前、コンポーネントの`${version}`（v2.0.0、v2.0.1など）、コンポーネントが実行されるオペレーティングシステム`${os}`とプラットフォーム`${arch}`を決定します。
- 次のコマンドを使用して現在のコンポーネントパッケージをダウンロードします：`wget https://tiup-mirrors.pingcap.com/${component}-${version}-${os}-${arch}.tar.gz -O /tmp/${component}-${version}-${os}-${arch}.tar.gz`。
- `mkdir -p /tmp/package && cd /tmp/package`を実行して一時ディレクトリを作成してファイルを詰めます。
- `/tmp/${component}-${version}-${os}-${arch}.tar.gz`を展開するために`tar xf /tmp/${component}-${version}-${os}-${arch}.tar.gz`を実行します。
- 一時パッケージディレクトリ内のファイル構造を表示するには`find .`を実行します。
- バイナリファイルや設定ファイルを一時ディレクトリ内の対応する場所にコピーします。
- 修正済みのファイルを再パッケージ化するには`tar czf /tmp/${component}-hotfix-${os}-${arch}.tar.gz *`を実行します。
- 最後に、`tiup dm patch`コマンドの`<package-path>`の値として`/tmp/${component}-hotfix-${os}-${arch}.tar.gz`を使用できます。

## オプション

### --overwrite

- 特定のコンポーネント（dm-workerなど）をパッチ適用した後、将来クラスターがスケールアウトするとき、tiup-dmはデフォルトで元のコンポーネントバージョンを使用します。将来のスケールアウト時にパッチ適用したバージョンを使用するには、コマンドでオプション`--overwrite`を指定する必要があります。
- データ型: `BOOLEAN`
- このオプションはデフォルトで無効（`false`値）です。このオプションを有効にするには、コマンドにこのオプションを追加し、`true`の値を渡すか値を渡さないかのいずれかです。

### -N, --node

- 置き換えるノードを指定します。このオプションの値はノードIDのコンマ区切りのリストです。ノードIDは`[tiup dm display](/tiup/tiup-component-dm-display.md)`コマンドによって返されるクラスターステータステーブルの最初の列から取得できます。
- データ型: `STRING`
- このオプションが指定されていない場合、デフォルトですべてのノードが置き換え対象になります。

> **注意：**
>
> 同時に`-R, --role`オプションが指定されている場合、TiUPは`-N, --node`と`-R, --role`の両方の要件を満たすサービスノードを置き換えます。

### -R, --role

- 置き換えるロールを指定します。このオプションの値はノードの役割のコンマ区切りのリストです。ノードの役割は`[tiup dm display](/tiup/tiup-component-dm-display.md)`コマンドによって返されるクラスターステータステーブルの2番目の列から取得できます。
- データ型: `STRING`
- このオプションが指定されていない場合、デフォルトですべてのロールが置き換え対象になります。

> **注意：**
>
> 同時に`-N, --node`オプションが指定されている場合、TiUPは`-N, --node`と`-R, --role`の両方の要件を満たすサービスノードを置き換えます。

### --offline

- 現在のクラスターがオフラインであることを宣言します。このオプションが指定されている場合、TiUP DMはサービスを再起動することなくクラスターコンポーネントのバイナリファイルをそのまま置き換えます。

### -h, --help

- ヘルプ情報を出力します。
- データ型: `BOOLEAN`
- このオプションはデフォルトで無効（`false`値）です。このオプションを有効にするには、コマンドにこのオプションを追加し、`true`の値を渡すか値を渡さないかのいずれかです。

## 例

次の例では、TiUPを使用してデプロイされた`v5.3.0`クラスターに`v5.3.0-hotfix`を適用する方法を示しています。クラスターを他の方法でデプロイした場合、操作は異なることがあります。

> **注意：**
>
> ホットフィックスは非常の修正のためにのみ使用されます。その日常的なメンテナンスは複雑です。公式版がリリースされたら、できるだけ早くDMクラスターをアップグレードすることをお勧めします。

### 準備

ホットフィックスを適用する前に、ホットフィックスパッケージ`dm-linux-amd64.tar.gz`を準備し、現在のDMソフトウェアバージョンを確認してください：

```shell
/home/tidb/dm/deploy/dm-master-8261/bin/dm-master/dm-master -V
```

出力：

```
リリースバージョン: v5.3.0

Gitコミットハッシュ: 20626babf21fc381d4364646c40dd84598533d66
Gitブランチ: heads/refs/tags/v5.3.0
UTCビルド時間: 2021-11-29 08:29:49
Goバージョン: go version go1.16.4 linux/amd64
```

### パッチパッケージを準備し、DMクラスターに適用する

1. 現在のバージョンに一致するDMソフトウェアパッケージを準備します：

    ```shell
    mkdir -p /tmp/package
    tar -zxvf /root/.tiup/storage/dm/packages/dm-master-v5.3.0-linux-amd64.tar.gz -C /tmp/package/
    tar -zxvf /root/.tiup/storage/dm/packages/dm-worker-v5.3.0-linux-amd64.tar.gz -C /tmp/package/
    ```

2. ホットフィックスパッケージでバイナリファイルを置き換えます：

    ```shell
    # ホットフィックスパッケージを展開してバイナリファイルを置き換えます。
    cd /root; tar -zxvf dm-linux-amd64.tar.gz
    cp /root/dm-linux-amd64/bin/dm-master /tmp/package/dm-master/dm-master
    cp /root/dm-linux-amd64/bin/dm-worker /tmp/package/dm-worker/dm-worker
    # 修正済みのファイルを再パッケージ化します。
    # 他のデプロイメント方法によってパッケージング方法が異なる場合がありますのでご注意ください。
    cd /tmp/package/ && tar -czvf dm-master-hotfix-linux-amd64.tar.gz dm-master/
    cd /tmp/package/ && tar -czvf dm-worker-hotfix-linux-amd64.tar.gz dm-worker/
    ```

3. ホットフィックスを適用します：

    クラスターステータスをクエリします。以下は例として`dm-test`という名前のクラスターを使用しています：

    ```shell
    tiup dm display dm-test
    ```

    出力：

    ```
    クラスタータイプ:       dm
    クラスター名:       dm-test
    クラスターバージョン:    v5.3.0
    デプロイユーザー:        tidb
    SSHタイプ:           builtin
    ID                  役割                 ホスト           ポート      OS/Arch       ステータス     データディレクトリ                              デプロイディレクトリ
    --                  ----                 ----           -----      -------       ------     --------                              ----------
    172.16.100.21:9093  alertmanager         172.16.100.21  9093/9094  linux/x86_64  Up         /home/tidb/dm/data/alertmanager-9093  /home/tidb/dm/deploy/alertmanager-9093
    172.16.100.21:8261  dm-master            172.16.100.21  8261/8291  linux/x86_64  Healthy|L  /home/tidb/dm/data/dm-master-8261     /home/tidb/dm/deploy/dm-master-8261
    172.16.100.21:8262  dm-worker            172.16.100.21  8262       linux/x86_64  Free       /home/tidb/dm/data/dm-worker-8262     /home/tidb/dm/deploy/dm-worker-8262
    172.16.100.21:3000  grafana              172.16.100.21  3000       linux/x86_64  Up         -                                     /home/tidb/dm/deploy/grafana-3000
    172.16.100.21:9090  prometheus           172.16.100.21  9090       linux/x86_64  Up         /home/tidb/dm/data/prometheus-9090    /home/tidb/dm/deploy/prometheus-9090
    Total nodes: 5
    ```

    指定されたノードまたは指定された役割にホットフィックスを適用します。`-R`と`-N`の両方が指定されている場合、共通部分が取られます。

    ```
```markdown
    # 指定されたノードにホットフィックスを適用します。
    tiup dm patch dm-test dm-master-hotfix-linux-amd64.tar.gz -N 172.16.100.21:8261
    tiup dm patch dm-test dm-worker-hotfix-linux-amd64.tar.gz -N 172.16.100.21:8262
    # 指定された役割にホットフィックスを適用します。
    tiup dm patch dm-test dm-master-hotfix-linux-amd64.tar.gz -R dm-master
    tiup dm patch dm-test dm-worker-hotfix-linux-amd64.tar.gz -R dm-worker
    ```

4. ホットフィックスの適用結果をクエリします：

    ```shell
    /home/tidb/dm/deploy/dm-master-8261/bin/dm-master/dm-master -V
    ```

    出力：

    ```
    リリースバージョン: v5.3.0-20211230
    Git Commit ハッシュ: ca7070c45013c24d34bd9c1e936071253451d707
    Git ブランチ: heads/refs/tags/v5.3.0-20211230
    UTC ビルド時間: 2022-01-05 14:19:02
    Go バージョン: go version go1.16.4 linux/amd64
    ```

    クラスタ情報が相応に変更されます：

    ```shell
    tiup dm display dm-test
    ```

    出力：

    ```
    コンポーネントを起動中: /root/.tiup/components/dm/v1.8.1/tiup-dm display dm-test
    クラスタの種類:       dm
    クラスタ名:           dm-test
    クラスタバージョン:   v5.3.0
    デプロイユーザー:      tidb
    SSH タイプ:           builtin
    ID                    役割                 ホスト           ポート      OS/Arch       ステータス     データディレクトリ                           デプロイディレクトリ
    --                    ----                 ----           -----      -------       ------     --------                              ----------
    172.16.100.21:9093    alertmanager         172.16.100.21  9093/9094  linux/x86_64  Up         /home/tidb/dm/data/alertmanager-9093  /home/tidb/dm/deploy/alertmanager-9093
    172.16.100.21:8261    dm-master (patched)  172.16.100.21  8261/8291  linux/x86_64  Healthy|L  /home/tidb/dm/data/dm-master-8261     /home/tidb/dm/deploy/dm-master-8261
    172.16.100.21:8262    dm-worker (patched)  172.16.100.21  8262       linux/x86_64  Free       /home/tidb/dm/data/dm-worker-8262     /home/tidb/dm/deploy/dm-worker-8262
    172.16.100.21:3000    grafana              172.16.100.21  3000       linux/x86_64  Up         -                                     /home/tidb/dm/deploy/grafana-3000
    172.16.100.21:9090    prometheus           172.16.100.21  9090       linux/x86_64  Up         /home/tidb/dm/data/prometheus-9090    /home/tidb/dm/deploy/prometheus-9090
    ノード合計: 5
    ```

[<< 前のページに戻る - TiUP DM コマンドリスト](/tiup/tiup-component-dm.md#command-list)
```