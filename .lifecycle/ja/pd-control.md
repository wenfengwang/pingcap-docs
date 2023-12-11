---
title: PDコントロールユーザーガイド
summary: PDコントロールを使用してクラスターの状態情報を取得し、クラスターを調整します。
aliases: ['/docs/dev/pd-control/','/docs/dev/reference/tools/pd-control/']
---

# PDコントロールユーザーガイド

PDは、PDコントロールとしてコマンドラインツールを使用してクラスターの状態情報を取得し、クラスターを調整します。

## PDコントロールのインストール

> **注意:**
>
> 使用するコントロールツールのバージョンはクラスターのバージョンと一致することを推奨します。

### TiUPコマンドの使用

PDコントロールを使用するには、`tiup ctl:v<CLUSTER_VERSION> pd -u http://<pd_ip>:<pd_port> [-i]` コマンドを実行します。

### インストールパッケージのダウンロード

最新バージョンの`pd-ctl`を取得するには、TiDBサーバーのインストールパッケージをダウンロードします。 `pd-ctl`は`ctl-{version}-linux-{arch}.tar.gz`パッケージに含まれています。

| インストールパッケージ                                                                 | OS | アーキテクチャ | SHA256チェックサム                                                    |
| :------------------------------------------------------------------------ | :------- | :---- | :--------------------------------------------------------------- |
| `https://download.pingcap.org/tidb-community-server-{version}-linux-amd64.tar.gz` (pd-ctl) | Linux | amd64 | `https://download.pingcap.org/tidb-community-server-{version}-linux-amd64.tar.gz.sha256` |
| `https://download.pingcap.org/tidb-community-server-{version}-linux-arm64.tar.gz` (pd-ctl) | Linux | arm64 | `https://download.pingcap.org/tidb-community-server-{version}-linux-arm64.tar.gz.sha256` |

> **注意:**
>
> リンク内の `{version}` はTiDBのバージョン番号を示します。例えば、`v7.4.0`の`amd64`アーキテクチャ向けのダウンロードリンクは `https://download.pingcap.org/tidb-community-server-v7.4.0-linux-amd64.tar.gz` です。

### ソースコードからのコンパイル

1. [Go](https://golang.org/) 1.21 以降が必要です。Goモジュールが使用されているためです。
2. [PDプロジェクト](https://github.com/pingcap/pd)のルートディレクトリで、`make` または `make pd-ctl` コマンドを使用して `bin/pd-ctl` をコンパイルおよび生成します。

## 使用法

シングルコマンドモード:

```bash
tiup ctl:v<CLUSTER_VERSION> pd store -u http://127.0.0.1:2379
```

対話モード:

```bash
tiup ctl:v<CLUSTER_VERSION> pd -i -u http://127.0.0.1:2379
```

環境変数の使用:

```bash
export PD_ADDR=http://127.0.0.1:2379
tiup ctl:v<CLUSTER_VERSION> pd
```

暗号化のためのTLSの使用:

```bash
tiup ctl:v<CLUSTER_VERSION> pd -u https://127.0.0.1:2379 --cacert="path/to/ca" --cert="path/to/cert" --key="path/to/key"
```

## コマンドラインフラグ

### `--cacert`

+ PEM形式の信頼されたCAの証明書ファイルへのパスを指定します
+ デフォルト: ""

### `--cert`

+ PEM形式のSSL証明書ファイルへのパスを指定します
+ デフォルト: ""

### `--detach` / `-d`

+ シングルコマンドラインモードを使用します（readlineに入力しません）
+ デフォルト: true

### `--help` / `-h`

+ ヘルプ情報を出力します
+ デフォルト: false

### `--interact` / `-i`

+ インタラクティブモードを使用します（readlineに入力します）
+ デフォルト: false

### `--key`

+ SSLのPEM形式の証明書キーファイルへのパスを指定します。これは `--cert` で指定された証明書の秘密鍵です
+ デフォルト: ""

### `--pd` / `-u`

+ PDのアドレスを指定します
+ デフォルトのアドレス: `http://127.0.0.1:2379`
+ 環境変数: `PD_ADDR`

### `--version` / `-V`

+ バージョン情報を表示して終了します
+ デフォルト: false

## コマンド

### `cluster`

このコマンドを使用してクラスターの基本情報を表示します。

使用法:

```bash
>> cluster                                     // クラスター情報を表示します
{
  "id": 6493707687106161130,
  "max_peer_count": 3
}
```

### `config [show | set <option> <value> | placement-rules]`

このコマンドを使用して、設定情報を表示または変更します。

使用法:

```bash
>> config show                                // スケジュールの設定情報を表示します
{
  "replication": {
    "enable-placement-rules": "true",
    "isolation-level": "",
    "location-labels": "",
    "max-replicas": 3,
    "strictly-match-label": "false"
  },
  "schedule": {
    "enable-cross-table-merge": "true",
    "high-space-ratio": 0.7,
    "hot-region-cache-hits-threshold": 3,
    "hot-region-schedule-limit": 4,
    "leader-schedule-limit": 4,
    "leader-schedule-policy": "count",
    "low-space-ratio": 0.8,
    "max-merge-region-keys": 200000,
    "max-merge-region-size": 20,
    "max-pending-peer-count": 64,
    "max-snapshot-count": 64,
    "max-store-down-time": "30m0s",
    "merge-schedule-limit": 8,
    "patrol-region-interval": "10ms",
    "region-schedule-limit": 2048,
    "region-score-formula-version": "v2",
    "replica-schedule-limit": 64,
    "scheduler-max-waiting-operator": 5,
    "split-merge-interval": "1h0m0s",
    "tolerant-size-ratio": 0
  }
}
>> config show all                            // すべての設定情報を表示します
>> config show replication                    // 複製の設定情報を表示します
{
  "max-replicas": 3,
  "location-labels": "",
  "isolation-level": "",
  "strictly-match-label": "false",
  "enable-placement-rules": "true"
}

>> config show cluster-version                // クラスターの現在のバージョンを表示します。これはクラスター内のTiKVノードの最小バージョンであり、バイナリバージョンには対応しません。
"5.2.2"
```

- `max-snapshot-count` は、一つのストアが同時に受信または送信する最大スナップショット数を制御します。この設定により、正常なアプリケーションのリソースを占有することを避けるため、スケジューラは制限されます。 レプリカの追加スピードまたはバランシングの速度を向上させる必要がある場合は、この値を増やします。

    ```bash
    config set max-snapshot-count 64  // スナップショットの最大数を64に設定
    ```

- `max-pending-peer-count` は、一つのストアにおけるペンディングピアの最大数を制御します。この設定により、一部のノードで最新のログがないリージョンを大量に生成することを避けるため、スケジューラは制限されます。レプリカの追加スピードまたはバランシングの速度を向上させる必要がある場合は、この値を増やします。 0に設定すると制限がないことを示します。

    ```bash
    config set max-pending-peer-count 64  // ペンディングピアの最大数を64に設定
    ```

- `max-merge-region-size` はリージョンマージの上限サイズ（単位はMiB）を制御します。 `regionSize` が指定された値を超えると、PDは隣接リージョンとマージしません。 0に設定するとリージョンマージが無効になります。

    ```bash
    config set max-merge-region-size 16 // リージョンマージの上限サイズを16 MiBに設定
    ```

- `max-merge-region-keys` はリージョンマージのキーカウントの上限を制御します。 `regionKeyCount` が指定された値を超えると、PDは隣接するリージョンとマージしません。

    ```bash
    config set max-merge-region-keys 50000 // キーカウントの上限を50000に設定
    ```

- `split-merge-interval` は同じリージョンの `split` と `merge` 操作間の間隔を制御します。これにより、新しく分割されたリージョンは一定期間内にマージされません。

    ```bash
    config set split-merge-interval 24h  // `split` と `merge` の間隔を1日に設定
    ```

- `enable-one-way-merge` はPDがリージョンを隣接リージョンとのみマージすることを制御します。 `false` に設定すると、PDはリージョンを隣接する2つのリージョンとマージできます。

    ```bash
    config set enable-one-way-merge true  // ワンウェイマージングを有効にします。
    ```

- `enable-cross-table-merge` は、異なるテーブルからのリージョンのマージを有効にするために使用されます。 `false` に設定すると、PDは異なるテーブルからのリージョンをマージしません。 このオプションは、キータイプが "table" の場合にのみ機能します。

    ```bash
    config set enable-cross-table-merge true  // クロステーブルマージを有効にします。
    ```
- `key-type`は、クラスタで使用されるキーのエンコーディングタイプを指定します。サポートされているオプションは["table", "raw", "txn"]であり、デフォルト値は"table"です。
    - クラスタにTiDBインスタンスが存在しない場合、`key-type`は"raw"または"txn"になり、PDは`enable-cross-table-merge`の設定に関係なく、テーブルをまたがる領域のマージを許可します。
    - クラスタにTiDBインスタンスが存在する場合、`key-type`は"table"である必要があります。PDがテーブルをまたがる領域をマージできるかどうかは、`enable-cross-table-merge`によって決まります。もし`key-type`が"raw"である場合、配置ルールは機能しません。

    ```bash
    config set key-type raw  // クロステーブルマージを有効にする
    ```

- `region-score-formula-version`は、リージョンスコアの計算式のバージョンを制御します。値のオプションは`v1`と`v2`です。式のバージョン2は、TiKVノードのオンライン化やオフライン化など、一部のシナリオで冗長なバランスリージョンのスケジューリングを削減するのに役立ちます。

    {{< copyable "" >}}

    ```bash
    config set region-score-formula-version v2
    ```

- `patrol-region-interval`は、`replicaChecker`がリージョンの健康状態を確認する実行頻度を制御します。短い間隔はより高い実行頻度を示します。一般的に、これを調整する必要はありません。

    ```bash
    config set patrol-region-interval 10ms // replicaCheckerの実行頻度を10msに設定
    ```

- `max-store-down-time`は、PDが指定された時間内にストアから心拍信号を受信しない場合、切断されたストアを復元できないと判断する時間を制御します。その場合、PDは他のノードにレプリカを追加します。

    ```bash
    config set max-store-down-time 30m  // PDが特定の期間内にストアから心拍信号を受信せず、その後レプリカを追加し始める時間を30分に設定
    ```

- `max-store-preparing-time`は、ストアがオンラインになるための最大待機時間を制御します。ストアのオンライン段階では、PDはストアのオンライン進行状況をクエリできます。指定された時間を超過すると、PDはストアがオンラインになり、再びストアのオンライン進捗をクエリできないと見なします。ただし、これによりリージョンが新しいオンラインストアに転送されることはありません。ほとんどのシナリオでは、このパラメータを調整する必要はありません。

    次のコマンドは、ストアがオンラインになるための最大待機時間を4時間に設定しています。

    {{< copyable "" >}}

    ```bash
    config set max-store-preparing-time 4h
    ```

- `leader-schedule-limit`は、同時にリーダーのスケジュールを行うタスクの数を制御します。この値はリーダーバランスの速度に影響します。大きな値はより高速な速度を意味し、値を0に設定するとスケジューリングが停止します。通常、リーダーのスケジュールには小さな負荷がかかりますので、必要に応じて値を増やすことができます。

    ```bash
    config set leader-schedule-limit 4         // 最大で同時に4つのリーダースケジュールタスクを行う
    ```

- `region-schedule-limit`は、同時にリージョンのスケジュールを行うタスクの数を制御します。この値により、多くのリージョンバランス演算子が作成されるのを避けることができます。デフォルト値は`2048`で、すべてのクラスタサイズに対応できる十分な値ですが、値を`0`に設定するとスケジューリングが停止します。通常、リージョンのスケジューリング速度は`store-limit`によって制限されますが、正確に理解していない限り、この値をカスタマイズしないことをお勧めします。

    ```bash
    config set region-schedule-limit 2         // 最大で同時に2つのリージョンスケジュールタスクを実行
    ```

- `replica-schedule-limit`は、同時にレプリカをスケジュールするタスクの数を制御します。ノードがダウンしたり削除されたりした場合のスケジューリング速度に影響します。大きな値はより高速な速度を意味し、値を0に設定するとスケジューリングが停止します。通常、レプリカのスケジューリングには多くの負荷がかかりますので、あまり大きな値を設定しないように注意してください。この構成項目は通常、デフォルト値のままにしておくことが推奨されます。値を変更したい場合は、実際の状況に最も適した値を見つけるためにいくつかの値を試してみる必要があります。

    ```bash
    config set replica-schedule-limit 4        // 最大で同時に4つのレプリカスケジューリングタスクを実行
    ```

- `merge-schedule-limit`は、リージョンマージスケジューリングタスクの数を制御します。値を`0`に設定するとリージョンマージを停止します。通常、マージスケジューリングには多くの負荷がかかりますので、あまり大きな値を設定しないように注意してください。この構成項目は通常、デフォルト値のままにしておくことが推奨されます。値を変更したい場合は、実際の状況に最も適した値を見つけるためにいくつかの値を試してみる必要があります。

    ```bash
    config set merge-schedule-limit 16       // 最大で同時に16のマージスケジューリングタスクを実行
    ```

- `hot-region-schedule-limit`は、同時に実行されるホットリージョンスケジューリングタスクの数を制御します。その値を`0`に設定するとスケジューリングが無効になります。あまり大きな値を設定しないことをお勧めします。そうしないと、システムのパフォーマンスに影響を及ぼす可能性があります。この構成項目は通常、デフォルト値のままにしておくことが推奨されます。値を変更したい場合は、実際の状況に最も適した値を見つけるためにいくつかの値を試してみる必要があります。

    ```bash
    config set hot-region-schedule-limit 4       // 最大で同時に4つのホットリージョンスケジューリングタスクを実行
    ```

- `hot-region-cache-hits-threshold`は、ホットリージョンを識別するために必要な分数の分単位を設定するために使用されます。PDは、リージョンがこの期間よりも長い時間ホット状態にある場合にのみホットスポットスケジューリングに参加できます。

- `tolerant-size-ratio`は、バランスバッファ領域のサイズを制御します。2つのストアのリーダーやリージョン間のスコアの違いが、リージョンサイズの指定された複数回未満の場合、PDはバランスと見なします。

    ```bash
    config set tolerant-size-ratio 20        // バッファ領域のサイズを平均リージョンサイズの約20倍に設定
    ```

- `low-space-ratio`は、不十分なストアの空き容量と見なされる閾値を制御します。ノードが占有する空間の比率が指定された値を超えると、PDは可能な限り該当ノードへのデータの移行を避けようとします。同時に、PDは残りの空間を主にスケジュールし、該当ノードのディスクスペースを使い切ることを避けます。

    ```bash
    config set low-space-ratio 0.9              // 不十分な空き容量の閾値値を0.9に設定
    ```

- `high-space-ratio`は、十分なストアの空き容量と見なされる閾値を制御します。この構成は、`region-score-formula-version`が`v1`に設定されている場合にのみ有効です。ノードが占有する空間の比率が指定された値よりも少ない場合、PDは残りの空間を無視し、主に実際のデータボリュームをスケジュールします。

    ```bash
    config set high-space-ratio 0.5             // 十分な空き容量の閾値値を0.5に設定
    ```

- `cluster-version`は、クラスタのバージョンであり、一部の機能を有効または無効にし、互換性の問題に対処するために使用されます。デフォルトでは、クラスタ内の通常稼働しているTiKVノードの最小バージョンです。必要に応じて手動で設定することができますが、以前のバージョンにロールバックする必要がある場合にのみ行うことをお勧めします。

    ```bash
    config set cluster-version 1.0.8              // クラスタのバージョンを1.0.8に設定
    ```

- `replication-mode`は、2つのデータセンターが存在するシナリオにおけるリージョンのレプリケーションモードを制御します。詳細については、[Enable the DR Auto-Sync mode](/two-data-centers-in-one-city-deployment.md#enable-the-dr-auto-sync-mode)を参照してください。

- `leader-schedule-policy`は、リーダーのスケジュール戦略を選択するために使用されます。`size`または`count`に従ってリーダーをスケジュールできます。

- `scheduler-max-waiting-operator`は、各スケジューラで待機中の演算子の数を制御します。

- `enable-remove-down-replica`は、ダウン状態のレプリカを自動的に削除する機能を有効にするために使用されます。`false`に設定すると、PDはダウンタイムレプリカを自動的にクリーンアップしません。

- `enable-replace-offline-replica`は、オフライン状態のレプリカを移行する機能を有効にするために使用されます。`false`に設定すると、PDはオフラインレプリカを移行しません。

- `enable-make-up-replica`は、レプリカを作成する機能を有効にするために使用されます。`false`に設定すると、PDは十分なレプリカのないリージョンにはレプリカを追加しません。

- `enable-remove-extra-replica`は、余分なレプリカを削除する機能を有効にするために使用されます。`false`に設定すると、PDは冗長なレプリカを持つリージョンから余分なレプリカを削除しません。

- `enable-location-replacement`は、隔離レベルのチェックを有効にするために使用されます。`false`に設定すると、PDはスケジューリングを通じてリージョンレプリカの隔離レベルを上げません。

- `enable-debug-metrics`は、デバッグ用のメトリクスを有効にするために使用されます。`true`に設定すると、PDは`balance-tolerant-size`などのいくつかのメトリクスを有効にします。

- `enable-placement-rules`は、配置ルールを有効にするために使用されます。これは、v5.0以降のバージョンではデフォルトで有効になっています。

- `store-limit-mode`は、ストアの速度を制限するモードを制御するために使用されます。オプションのモードは`auto`および`manual`です。`auto`モードでは、ストアは負荷に応じて自動的にバランスされます（非推奨）。
- `store-limit-version`はストアリミットフォーミュラのバージョンを制御します。v1モードでは、`store limit`を手動で変更して、単一のTiKVのスケジューリング速度を制限できます。v2モードは実験的な機能です。v2モードでは、TiKVスナップショットの能力に基づいて、PDが`store limit`の値を自動的に調整します。詳細については、[store limit v2の原則](/configure-store-limit.md#principles-of-store-limit-v2)を参照してください。

    ```bash
    config set store-limit-version v2       // store limit v2を使用
    ```

- PDはフローナンバーの最小桁を四捨五入し、Regionフロー情報の変更による統計情報の更新を減らします。この設定項目は、Regionフロー情報の四捨五入する最小桁数を指定するために使用されます。例えば、デフォルト値が`3`のため、フロー`100512`は`101000`に丸められます。この設定は`trace-region-flow`を置き換えます。

- 例えば、`flow-round-by-digit`の値を`4`に設定します：

    {{< copyable "" >}}

    ```bash
    config set flow-round-by-digit 4
    ```

#### `config placement-rules [disable | enable | load | save | show | rule-group]`

`config placement-rules [disable | enable | load | save | show | rule-group]`の使用法については、[配置ルールの設定](/configure-placement-rules.md#configure-rules)を参照してください。

### `health`

このコマンドを使用して、クラスタのヘルス情報を表示します。

使用法：

```bash
>> health                                // ヘルス情報を表示
[
  {
    "name": "pd",
    "member_id": 13195394291058371180,
    "client_urls": [
      "http://127.0.0.1:2379"
      ......
    ],
    "health": true
  }
  ......
]
```

### `hot [read | write | store|  history <start_time> <end_time> [<key> <value>]]`

このコマンドを使用して、クラスタのホットスポット情報を表示します。

使用法：

```bash
>> hot read                                // 読み取り操作のホットスポットを表示
>> hot write                               // 書き込み操作のホットスポットを表示
>> hot store                               // すべての読み取りおよび書き込み操作のホットスポットを表示
>> hot history 1629294000000 1631980800000 // 指定期間のヒストリカルなホットスポットを表示（ミリ秒）。1629294000000が開始時間で1631980800000が終了時間です。
{
  "history_hot_region": [
    {
      "update_time": 1630864801948,
      "region_id": 103,
      "peer_id": 1369002,
      "store_id": 3,
      "is_leader": true,
      "is_learner": false,
      "hot_region_type": "read",
      "hot_degree": 152,
      "flow_bytes": 0,
      "key_rate": 0,
      "query_rate": 305,
      "start_key": "7480000000000000FF5300000000000000F8",
      "end_key": "7480000000000000FF5600000000000000F8"
    },
    ...
  ]
}
>> hot history 1629294000000 1631980800000 hot_region_type read region_id 1,2,3 store_id 1,2,3 peer_id 1,2,3 is_leader true is_learner true // 追加の条件を指定して指定期間のヒストリカルなホットスポットを表示
{
  "history_hot_region": [
    {
      "update_time": 1630864801948,
      "region_id": 103,
      "peer_id": 1369002,
      "store_id": 3,
      "is_leader": true,
      "is_learner": false,
      "hot_region_type": "read",
      "hot_degree": 152,
      "flow_bytes": 0,
      "key_rate": 0,
      "query_rate": 305,
      "start_key": "7480000000000000FF5300000000000000F8",
      "end_key": "7480000000000000FF5600000000000000F8"
    },
    ...
  ]
}
```

### `label [store <name> <value>]`

このコマンドを使用して、クラスタのラベル情報を表示します。

使用法：

```bash
>> label                                // すべてのラベルを表示
>> label store zone cn                  // "zone":"cn"のラベルを含むすべてのストアを表示
```

### `member [delete | leader_priority | leader [show | resign | transfer <member_name>]]`

このコマンドを使用して、PDメンバーを表示し、指定されたメンバーを削除するかリーダーの優先順位を設定します。

使用法：

```bash
>> member                               // すべてのメンバーの情報を表示
{
  "header": {......},
  "members": [......],
  "leader": {......},
  "etcd_leader": {......},
}
>> member delete name pd2               // "pd2"を削除
Success!
>> member delete id 1319539429105371180 // IDを使用してノードを削除
Success!
>> member leader show                   // リーダー情報を表示
{
  "name": "pd",
  "member_id": 13155432540099656863,
  "peer_urls": [......],
  "client_urls": [......]
}
>> member leader resign // 現在のメンバーからリーダーを移動
......
>> member leader transfer pd3 // リーダーを指定されたメンバーに移行
......
```

### `operator [check | show | add | remove]`

このコマンドを使用して、スケジューリング操作を表示および制御します。

使用法：

```bash
>> operator show                                        // すべてのオペレータを表示
>> operator show admin                                  // すべての管理者オペレータを表示
>> operator show leader                                 // すべてのリーダーオペレータを表示
>> operator show region                                 // すべてのリージョンオペレータを表示
>> operator add add-peer 1 2                            // ストア2にリージョン1のレプリカを追加
>> operator add add-learner 1 2                         // ストア2にリージョン1のラーナーレプリカを追加
>> operator add remove-peer 1 2                         // ストア2のリージョン1のレプリカを削除
>> operator add transfer-leader 1 2                     // リージョン1のリーダーをストア2にスケジュール
>> operator add transfer-region 1 2 3 4                 // リージョン1をストア2、3、4にスケジュール
>> operator add transfer-peer 1 2 3                     // ストア2のリージョン1のレプリカをストア3にスケジュール
>> operator add merge-region 1 2                        // リージョン1をリージョン2とマージ
>> operator add split-region 1 --policy=approximate     // Region 1をおおよその推定値に基づいて2つに分割
>> operator add split-region 1 --policy=scan            // Region 1を正確なスキャン値に基づいて2つに分割
>> operator remove 1                                    // リージョン1のスケジューリングオペレーションを削除
>> operator check 1                                     // リージョン1に関連するオペレータのステータスをチェック
```

リージョンの分割はできるだけ中央に近い位置から開始されます。この位置を特定するには、"scan"と"approximate"の2つの戦略を使用できます。それらの違いは、前者はリージョンをスキャンして中央キーを決定し、後者はSSTファイルに記録された統計情報をチェックしておおよその位置を得ることです。一般的に、前者はより正確ですが、後者はより少ないI/Oを消費し、より速く完了できます。

### `ping`

このコマンドを使用して、`ping` PDがかかる時間を表示します。

使用法：

```bash
>> ping
time: 43.12698ms
```

### `region <region_id> [--jq="<query string>"]`

このコマンドを使用して、Regionの情報を表示します。jqフォーマットの出力については、[jq-formatted-json-output-usage](#jq-formatted-json-output-usage)を参照してください。

使用法：

```bash
>> region                               // すべてのRegionの情報を表示
{
  "count": 1,
  "regions": [......]
}

>> region 2                             // IDが2のRegionの情報を表示
{
  "id": 2,
  "start_key": "7480000000000000FF1D00000000000000F8",
  "end_key": "7480000000000000FF1F00000000000000F8",
  "epoch": {
    "conf_ver": 1,
    "version": 15
  },
  "peers": [
    {
      "id": 40,
      "store_id": 3
    }
  ],
  "leader": {
    "id": 40,
    "store_id": 3
  },
  "written_bytes": 0,
  "read_bytes": 0,
  "written_keys": 0,
  "read_keys": 0,
  "approximate_size": 1,
  "approximate_keys": 0
}
```

### `region key [--format=raw|encode|hex] <key>`

このコマンドを使用して、特定のキーが存在するRegionをクエリします。raw、encoding、hexの形式をサポートしており、エンコーディング形式でキーを使用する場合はシングルクォートを使用する必要があります。

Hex形式の使用法（デフォルト）：

```bash
```json
>> region key 7480000000000000FF1300000000000000F8
{
  "region": {
    "id": 2,
    ......
  }
}
```

Raw formatの使用方法：

```bash
>> region key --format=raw abc
{
  "region": {
    "id": 2,
    ......
  }
}
```

エンコード形式の使用方法：

```bash
>> region key --format=encode 't\200\000\000\000\000\000\000\377\035_r\200\000\000\000\000\377\017U\320\000\000\000\000\000\372'
{
  "region": {
    "id": 2,
    ......
  }
}
```

### `region scan`

すべてのリージョンを取得するには、このコマンドを使用します。

使用法：

```bash
>> region scan
{
  "count": 20,
  "regions": [......],
}
```

### `region sibling <region_id>`

特定のリージョンの隣接するリージョンをチェックするには、このコマンドを使用します。

使用法：

```bash
>> region sibling 2
{
  "count": 2,
  "regions": [......],
}
```

### `region keys [--format=raw|encode|hex] <start_key> <end_key> <limit>`

このコマンドを使用して、指定された範囲`[startkey、endkey)`内のすべてのリージョンをクエリします。 `endKey`のない範囲もサポートされています。

`limit`パラメータはキーの数を制限します。 `limit`のデフォルト値は`16`で、`-1`の値は無制限のキーを意味します。

使用法：

```bash
>> region keys --format=raw a         // キーaから始まるすべてのリージョンをデフォルトの制限数16で表示
{
  "count": 16,
  "regions": [......],
}

>> region keys --format=raw a z      // 範囲[a、z)内のすべてのリージョンをデフォルトの制限数16で表示
{
  "count": 16,
  "regions": [......],
}

>> region keys --format=raw a z -1   // 制限数なしで範囲[a、z)内のすべてのリージョンを表示
{
  "count": ...,
  "regions": [......],
}

>> region keys --format=raw a "" 20   // キーaから始まるリージョンを制限数20で表示
{
  "count": 20,
  "regions": [......],
}
```

### `region store <store_id>`

特定のストアのすべてのリージョンをリストするには、このコマンドを使用します。

使用法：

```bash
>> region store 2
{
  "count": 10,
  "regions": [......],
}
```

### `region topread [limit]`

トップの読み込みフローを持つリージョンをリストするには、このコマンドを使用します。制限のデフォルト値は16です。

使用法：

```bash
>> region topread
{
  "count": 16,
  "regions": [......],
}
```

### `region topwrite [limit]`

トップの書き込みフローを持つリージョンをリストするには、このコマンドを使用します。制限のデフォルト値は16です。

使用法：

```bash
>> region topwrite
{
  "count": 16,
  "regions": [......],
}
```

### `region topconfver [limit]`

トップの構成バージョンを持つリージョンをリストするには、このコマンドを使用します。制限のデフォルト値は16です。

使用法：

```bash
>> region topconfver
{
  "count": 16,
  "regions": [......],
}
```

### `region topversion [limit]`

トップのバージョンを持つリージョンをリストするには、このコマンドを使用します。制限のデフォルト値は16です。

使用法：

```bash
>> region topversion
{
  "count": 16,
  "regions": [......],
}
```

### `region topsize [limit]`

トップの近似サイズを持つリージョンをリストするには、このコマンドを使用します。制限のデフォルト値は16です。

使用法：

```bash
>> region topsize
{
  "count": 16,
  "regions": [......],
}

```

### `region check [miss-peer | extra-peer | down-peer | pending-peer | offline-peer | empty-region | hist-size | hist-keys] [--jq="<query string>"]`

異常な状態のリージョンをチェックするには、このコマンドを使用します。jq形式の出力については、[jq形式のJSON出力の使用法](#jq-formatted-json-output-usage)を参照してください。

さまざまなタイプの説明：

- miss-peer：十分なレプリカのないリージョン
- extra-peer：余分なレプリカを持つリージョン
- down-peer：いくつかのレプリカが停止しているリージョン
- pending-peer：いくつかのレプリカが保留中のリージョン

使用法：

```bash
>> region check miss-peer
{
  "count": 2,
  "regions": [......],
}
```

### `scheduler [show | add | remove | pause | resume | config | describe]`

スケジューリングポリシーを表示および制御するには、このコマンドを使用します。

使用法：

```bash
>> scheduler show                                 // 作成されたすべてのスケジューラを表示
>> scheduler add grant-leader-scheduler 1         // ストア1のすべてのリージョンのリーダーをストア1にスケジュールする
>> scheduler add evict-leader-scheduler 1         // ストア1のすべてのリージョンのリーダーを移動する
>> scheduler config evict-leader-scheduler        // v4.0.0以降、スケジューラが存在するストアを表示
>> scheduler add shuffle-leader-scheduler         // 異なるストアの上でリーダーをランダムに交換する
>> scheduler add shuffle-region-scheduler         // 異なるストアでリージョンをランダムにスケジュールする
>> scheduler add evict-slow-store-scheduler       // 遅いストアが1つだけある場合、そのストアのすべてのリージョンのリーダーを移動する
>> scheduler remove grant-leader-scheduler-1      // 対応するスケジューラを削除し、`-1`はストアIDに対応します
>> scheduler pause balance-region-scheduler 10    // balance-regionスケジューラを10秒間一時停止する
>> scheduler pause all 10                         // すべてのスケジューラを10秒間一時停止する
>> scheduler resume balance-region-scheduler      // balance-regionスケジューラを再開する
>> scheduler resume all                           // すべてのスケジューラを再開する
>> scheduler config balance-hot-region-scheduler  // balance-hot-regionスケジューラの構成を表示
>> scheduler describe balance-region-scheduler    // balance-regionスケジューラの実行状態と関連する診断情報を表示
```

### `scheduler describe balance-region-scheduler`

`balance-region-scheduler`の実行状態と関連する診断情報を表示するには、このコマンドを使用します。

TiDB v6.3.0以降、PDは`balance-region-scheduler`および`balance-leader-scheduler`の実行状態と簡単な診断情報を提供します。その他のスケジューラおよびチェッカーはまだサポートされていません。この機能を有効にするには、`pd-ctl`を使用して[`enable-diagnostic`](/pd-configuration-file.md#enable-diagnostic-new-in-v630)の設定項目を変更できます。

スケジューラの状態は次のいずれかです:

- `disabled`: スケジューラが利用できない、または削除されている
- `paused`: スケジューラが一時停止されている
- `scheduling`: スケジューラがスケジューリング演算子を生成している
- `pending`: スケジューラがスケジューリング演算子を生成できない。`pending`状態のスケジューラでは、簡単な診断情報が返されます。簡単な情報はストアの状態を説明し、なぜこれらのストアをスケジューリングのために選択できないかを説明します
- `normal`: スケジューリング演算子を生成する必要はありません。

### `scheduler config balance-leader-scheduler`

`balance-leader-scheduler`ポリシーを表示および制御するには、このコマンドを使用します。

TiDB v6.0.0以降、PDは`balance-leader-scheduler`で`Batch`パラメータを導入し、balance-leaderがタスクを処理する速度を制御します。このパラメータを使用するには、`pd-ctl`を使用して`balance-leader batch`の設定項目を変更できます。

v6.0.0より前のバージョンでは、PDにはこの構成項目がありません，つまり`balance-leader batch=1`です。`balance-leader batch`のデフォルト値はv6.0.0以降、`4`です。この構成項目を`4`より大きな値に設定するには、同時に[`scheduler-max-waiting-operator`](#config-show--set-option-value--placement-rules)（デフォルト値は`5`）の値を大きく設定する必要があります。両方の構成項目を変更した後に期待の加速効果を得ることができます。

```bash
scheduler config balance-leader-scheduler set batch 3 // balance-leaderスケジューラがバッチで実行できるオペレータのサイズを3に設定
```

#### `scheduler config balance-hot-region-scheduler`

`balance-hot-region-scheduler`ポリシーを表示および制御するには、このコマンドを使用します。

使用法：

```bash
>> scheduler config balance-hot-region-scheduler  // balance-hot-regionスケジューラのすべての構成を表示
{
  "min-hot-byte-rate": 100,
  "min-hot-key-rate": 10,
  "min-hot-query-rate": 10,
  "max-zombie-rounds": 3,
  "max-peer-number": 1000,
  "byte-rate-rank-step-ratio": 0.05,
  "key-rate-rank-step-ratio": 0.05,
  "query-rate-rank-step-ratio": 0.05,
  "count-rank-step-ratio": 0.01,
```yaml
"great-dec-ratio": 0.95,
"minor-dec-ratio": 0.99,
"src-tolerance-ratio": 1.05,
"dst-tolerance-ratio": 1.05,
"read-priorities": [
  "query",
  "byte"
],
"write-leader-priorities": [
  "key",
  "byte"
],
"write-peer-priorities": [
  "byte",
  "key"
],
"strict-picking-store": "true",
"enable-for-tiflash": "true",
"rank-formula-version": "v2"
}
```

- `min-hot-byte-rate` means the smallest number of bytes to be counted, which is usually 100.

    ```bash
    scheduler config balance-hot-region-scheduler set min-hot-byte-rate 100
    ```

- `min-hot-key-rate` means the smallest number of keys to be counted, which is usually 10.

    ```bash
    scheduler config balance-hot-region-scheduler set min-hot-key-rate 10
    ```

- `min-hot-query-rate` means the smallest number of queries to be counted, which is usually 10.

    ```bash
    scheduler config balance-hot-region-scheduler set min-hot-query-rate 10
    ```

- `max-zombie-rounds` means the maximum number of heartbeats with which an operator can be considered as the pending influence. If you set it to a larger value, more operators might be included in the pending influence. Usually, you do not need to adjust its value. Pending influence refers to the operator influence that is generated during scheduling but still has an effect.

    ```bash
    scheduler config balance-hot-region-scheduler set max-zombie-rounds 3
    ```

- `max-peer-number` means the maximum number of peers to be solved, which prevents the scheduler from being too slow.

    ```bash
    scheduler config balance-hot-region-scheduler set max-peer-number 1000
    ```

- `byte-rate-rank-step-ratio`, `key-rate-rank-step-ratio`, `query-rate-rank-step-ratio`, and `count-rank-step-ratio` respectively mean the step ranks of byte, key, query, and count. The rank-step-ratio decides the step when the rank is calculated. `great-dec-ratio` and `minor-dec-ratio` are used to determine the `dec` rank. Usually, you do not need to modify these items.

    ```bash
    scheduler config balance-hot-region-scheduler set byte-rate-rank-step-ratio 0.05
    ```

- `src-tolerance-ratio` and `dst-tolerance-ratio` are configuration items for the expectation scheduler. The smaller the `tolerance-ratio`, the easier it is for scheduling. When redundant scheduling occurs, you can appropriately increase this value.

    ```bash
    scheduler config balance-hot-region-scheduler set src-tolerance-ratio 1.1
    ```

- `read-priorities`, `write-leader-priorities`, and `write-peer-priorities` control which dimension the scheduler prioritizes for hot Region scheduling. Two dimensions are supported for configuration.

    - `read-priorities` and `write-leader-priorities` control which dimensions the scheduler prioritizes for scheduling hot Regions of the read and write-leader types. The dimension options are `query`, `byte`, and `key`.
    - `write-peer-priorities` controls which dimensions the scheduler prioritizes for scheduling hot Regions of the write-peer type. The dimension options are `byte` and `key`.

    > **Note:**
    >
    > If a cluster component is earlier than v5.2, the configuration of `query` dimension does not take effect. If some components are upgraded to v5.2 or later, the `byte` and `key` dimensions still by default have the priority for hot Region scheduling. After all components of the cluster are upgraded to v5.2 or later, such a configuration still takes effect for compatibility. You can view the real-time configuration using the `pd-ctl` command. Usually, you do not need to modify these configurations.

    ```bash
    scheduler config balance-hot-region-scheduler set read-priorities query,byte
    ```

- `strict-picking-store` controls the search space of hot Region scheduling. Usually, it is enabled. This configuration item only affects the behavior when `rank-formula-version` is `v1`. When it is enabled, hot Region scheduling ensures hot Region balance on the two configured dimensions. When it is disabled, hot Region scheduling only ensures the balance on the dimension with the first priority, which might reduce balance on other dimensions. Usually, you do not need to modify this configuration.

    ```bash
    scheduler config balance-hot-region-scheduler set strict-picking-store true
    ```

- `rank-formula-version` controls which scheduler algorithm version is used in hot Region scheduling. Value options are `v1` and `v2`. The default value is `v2`.

    - The `v1` algorithm is the scheduler strategy used in TiDB v6.3.0 and earlier versions. This algorithm mainly focuses on reducing load difference between stores and avoids introducing side effects in the other dimension.
    - The `v2` algorithm is an experimental scheduler strategy introduced in TiDB v6.3.0 and is in General Availability (GA) in TiDB v6.4.0. This algorithm mainly focuses on improving the rate of the equitability between stores and factors in few side effects. Compared with the `v1` algorithm with `strict-picking-store` being `true`, the `v2` algorithm pays more attention to the priority equalization of the first dimension. Compared with the `v1` algorithm with `strict-picking-store` being `false`, the `v2` algorithm considers the balance of the second dimension.
    - The `v1` algorithm with `strict-picking-store` being `true` is conservative and scheduling can only be generated when there is a store with a high load in both dimensions. In certain scenarios, it might be impossible to continue balancing due to dimensional conflicts. To achieve better balancing in the first dimension, it is necessary to set the `strict-picking-store` to `false`. The `v2` algorithm can achieve better balancing in both dimensions and reduce invalid scheduling.

  ```bash
  scheduler config balance-hot-region-scheduler set rank-formula-version v2
  ```

- `enable-for-tiflash` controls whether hot Region scheduling takes effect for TiFlash instances. Usually, it is enabled. When it is disabled, the hot Region scheduling between TiFlash instances is not performed.

    ```bash
    scheduler config balance-hot-region-scheduler set enable-for-tiflash true
    ```

### `service-gc-safepoint`

Use this command to query the current GC safepoint and service GC safepoint. The output is as follows:

```bash
{
  "service_gc_safe_points": [
    {
      "service_id": "gc_worker",
      "expired_at": 9223372036854775807,
      "safe_point": 439923410637160448
    }
  ],
  "gc_safe_point": 0
}
```

### `store [delete | cancel-delete | label | weight | remove-tombstone | limit ] <store_id> [--jq="<query string>"]`

For a jq formatted output, see [jq-formatted-json-output-usage](#jq-formatted-json-output-usage).

#### Get a store

To display the information of all stores, run the following command:

```bash
store
```

```
{
  "count": 3,
  "stores": [...]
}
```

To get the store with id of 1, run the following command:

```bash
store 1
```

```
......
```

#### Delete a store

To delete the store with id of 1, run the following command:

```bash
store delete 1
```

To cancel deleting `Offline` state stores which are deleted using `store delete`, run the `store cancel-delete` command. After canceling, the store changes from `Offline` to `Up`. Note that the `store cancel-delete` command cannot change a `Tombstone` state store to the `Up` state.

To cancel deleting the store with id of 1, run the following command:

```bash
store cancel-delete 1
```

To delete all stores in `Tombstone` state, run the following command:

```bash
store remove-tombstone
```

> **Note:**
>
> If the PD leader changes during store deletion, you need to modify the store limit manually using the [`store limit`](#configure-store-scheduling-speed) command.

#### Manage store labels

To manage the labels of a store, run the `store label` command.

- To set a label with the key being `"zone"` and value being `"cn"` to the store with id of 1, run the following command:

    ```bash
    store label 1 zone=cn
    ```

- To update the label of a store, for example, changing the value of the key `"zone"` from `"cn"` to `"us"` for the store with id of 1, run the following command:

    ```bash
    store label 1 zone=us
    ```

- ストアIDが1のすべてのラベルを書き換えるには、`--rewrite`オプションを使用します。このオプションはすべての既存のラベルを上書きします：

    ```bash
    store label 1 region=us-est-1 disk=ssd --rewrite
    ```

- ストアIDが1のストアから`"disk"`ラベルを削除するには、`--delete`オプションを使用します：

    ```bash
    store label 1 disk --delete
    ```

> **注意:**
>
> - ストアのラベルは、TiKVのラベルとPDのラベルをマージして更新されます。具体的には、TiKV構成ファイルでストアラベルを変更し、クラスタを再起動した後、PDは独自のストアラベルをTiKVのストアラベルとマージし、ラベルを更新してマージ結果を永続化します。
> - TiUPを使用してストアのラベルを管理するには、クラスタを再起動する前に`store label <id> --force`コマンドを実行して、PDに保存されたラベルを空にすることができます。

#### ストアの重みの設定

ストアIDが1のストアのリーダーの重みを5、リージョンの重みを10に設定するには、次のコマンドを実行します：

```bash
store weight 1 5 10
```

#### ストアのスケジューリング速度の設定

ストアのスケジューリング速度は`store limit`を使用して設定することができます。`store limit`の原則と使用方法の詳細については、[`store limit`](/configure-store-limit.md)を参照してください。

```bash
>> store limit                         // すべてのストアでの1分あたりのadd-peer操作の速度制限とremove-peer操作の制限を表示
>> store limit add-peer                // すべてのストアでの1分あたりのadd-peer操作の速度制限を表示
>> store limit remove-peer             // すべてのストアでの1分あたりのremove-peer操作の制限を表示
>> store limit all 5                   // すべてのストアでのadd-peer操作の制限を1分あたり5回、remove-peer操作の制限を1分あたり5回に設定
>> store limit 1 5                     // ストア1でのadd-peer操作の制限を1分あたり5回、remove-peer操作の制限を1分あたり5回に設定
>> store limit all 5 add-peer          // すべてのストアでのadd-peer操作の制限を1分あたり5回に設定
>> store limit 1 5 add-peer            // ストア1でのadd-peer操作の制限を1分あたり5回に設定
>> store limit 1 5 remove-peer         // ストア1でのremove-peer操作の制限を1分あたり5回に設定
>> store limit all 5 remove-peer       // すべてのストアでのremove-peer操作の制限を1分あたり5回に設定
```

> **注意:**
>
> `pd-ctl`を使用してTiKVストアの状態（`Up`、`Disconnect`、`Offline`、`Down`、または`Tombstone`）を確認することができます。それぞれの状態の関係については、[TiKVストアの各状態間の関係](/tidb-scheduling.md#information-collection)を参照してください。

### `log [fatal | error | warn | info | debug]`

このコマンドを使用して、PDリーダーのログレベルを設定します。

使用方法：

```bash
log warn
```

### `tso`

このコマンドを使用して、TSOの物理的および論理的な時刻を解析します。

使用法：

```bash
>> tso 395181938313123110        // TSOを解析
system:  2017-10-09 05:50:59 +0800 CST
logic:  120102
```

### `unsafe remove-failed-stores [store-ids | show]`

> **警告:**
>
> - この機能はデータの整合性とデータインデックスの整合性をTiKVが保証できないため、損失が発生します。
> - 機能に関連する操作を実行する際は、TiDBチームのサポートを受けることをお勧めします。誤操作が行われた場合、クラスタを回復することが難しいことがあります。

永続的に損傷したレプリカによってデータが利用できなくなった場合、このコマンドを使用して損失回復操作を実行します。次の例を参照してください。詳細については[オンライン・アンセーフ・リカバリ](/online-unsafe-recovery.md)を参照してください。

永続的に損傷したストアを削除するために、次のように実行します：

```bash
unsafe remove-failed-stores 101,102,103
```

```bash
成功しました！
```

現在のまたは過去のオンライン・アンセーフ・リカバリの状態を表示するには：

```bash
unsafe remove-failed-stores show
```

```bash
[
  "すべての稼働中のストアからクラスタ情報を収集中, 10/12.",
  "PDにレポートを行ったストア: 1, 2, 3, ...",
  "PDにレポートを行っていないストア: 11, 12",
]
```

## JqフォーマットJSON出力の使用法

### `store`の出力を簡略化する

```bash
>> store --jq=".stores[].store | { id, address, state_name}"
{"id":1,"address":"127.0.0.1:20161","state_name":"Up"}
{"id":30,"address":"127.0.0.1:20162","state_name":"Up"}
...
```

### ノードの残りのスペースをクエリする

```bash
>> store --jq=".stores[] | {id: .store.id, available: .status.available}"
{"id":1,"available":"10 GiB"}
{"id":30,"available":"10 GiB"}
...
```

### ステータスが`Up`でないすべてのノードをクエリする

```bash
store --jq='.stores[].store | select(.state_name!="Up") | { id, address, state_name}'
```

```
{"id":1,"address":"127.0.0.1:20161""state_name":"Offline"}
{"id":5,"address":"127.0.0.1:20162""state_name":"Offline"}
...
```

### すべてのTiFlashノードをクエリする

```bash
store --jq='.stores[].store | select(.labels | length>0 and contains([{"key":"engine","value":"tiflash"}])) | { id, address, state_name}'
```

```
{"id":1,"address":"127.0.0.1:20161""state_name":"Up"}
{"id":5,"address":"127.0.0.1:20162""state_name":"Up"}
...
```

### リージョンのレプリカの分散状況をクエリする

```bash
>> region --jq=".regions[] | {id: .id, peer_stores: [.peers[].store_id]}"
{"id":2,"peer_stores":[1,30,31]}
{"id":4,"peer_stores":[1,31,34]}
...
```

### レプリカの数に基づいてリージョンをフィルタリングする

たとえば、レプリカの数が3でないすべてのリージョンをフィルタリングするには：

```bash
>> region --jq=".regions[] | {id: .id, peer_stores: [.peers[].store_id] | select(length != 3)}"
{"id":12,"peer_stores":[30,32]}
{"id":2,"peer_stores":[1,30,31,32]}
```

### レプリカのストアIDに基づいてリージョンをフィルタリングする

たとえば、ストア30にレプリカがあるすべてのリージョンをフィルタリングするには：

```bash
>> region --jq=".regions[] | {id: .id, peer_stores: [.peers[].store_id] | select(any(.==30))}"
{"id":6,"peer_stores":[1,30,31]}
{"id":22,"peer_stores":[1,30,32]}
...
```

同様に、ストア30またはストア31にレプリカがあるすべてのリージョンを見つけることもでいます：

```bash
>> region --jq=".regions[] | {id: .id, peer_stores: [.peers[].store_id] | select(any(.==(30,31)))}"
{"id":16,"peer_stores":[1,30,34]}
{"id":28,"peer_stores":[1,30,32]}
{"id":12,"peer_stores":[30,32]}
...
```

### データを復元するときの関連リージョンを検索する

たとえば、[ストア1、ストア30、ストア31]がダウンタイムにある場合、通常のレプリカよりもダウンレプリカが多いすべてのリージョンを見つけることができます：

```bash
>> region --jq=".regions[] | {id: .id, peer_stores: [.peers[].store_id] | select(length as $total | map(if .==(1,30,31) then . else empty end) | length>=$total-length) }"
{"id":2,"peer_stores":[1,30,31,32]}
{"id":12,"peer_stores":[30,32]}
{"id":14,"peer_stores":[1,30,32]}
...
```

または、[ストア1、ストア30、ストア31]が起動しない場合、ストア1で安全にデータを手動で削除できるリージョンを見つけることができます。この方法では、ストア1にレプリカがあり、他のダウンレプリカがないすべてのリージョンをフィルタリングすることができます：

```bash
>> region --jq=".regions[] | {id: .id, peer_stores: [.peers[].store_id] | select(length>1 and any(.==1) and all(.!=(30,31)))}"
{"id":24,"peer_stores":[1,32,33]}
```
```bash
[store30、store31]がダウンしている場合、「remove-peer」オペレーターを作成して安全に処理できるすべてのリージョンを見つけます。すなわち、1つのダウンピアを持つリージョン：

```bash
>> region --jq=".regions[] | {id: .id, remove_peer: [.peers[].store_id] | select(length>1) | map(if .==(30,31) then . else empty end) | select(length==1)}"
{"id":12,"remove_peer":[30]}
{"id":4,"remove_peer":[31]}
{"id":22,"remove_peer":[30]}
...
```